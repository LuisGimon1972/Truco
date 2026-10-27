<template>
  <q-page class="flex flex-center text-white q-pa-sm truco-page">
    <div
      class="game-container column items-center q-gutter-xs q-pa-md"
      style="max-width: 950px; width: 100%"
    >
      <!-- Cabeçalho Elegante / Placar -->
      <div class="column items-center q-gutter-xs w-full q-px-sm" style="width: 100%">
        <div class="text-h5 text-weight-bold text-white tracking-wide text-shadow">
          Truco Game Venezuelan
        </div>

        <!-- Placar e Apostas Estilizados -->
        <div class="row justify-center items-center q-gutter-sm q-my-none">
          <q-chip class="game-chip dense" icon="person">Você: {{ pontosHumano }}</q-chip>
          <q-chip class="game-chip dense" icon="smart_toy">IA: {{ pontosIA }}</q-chip>
          <q-chip class="game-chip-gold dense" icon="local_fire_department"
            >Aposta: {{ apostaAtual }} pts</q-chip
          >
        </div>

        <!-- Botões de Ação Principal -->
        <div class="row justify-center items-center q-gutter-xs flex-wrap q-mt-xs">
          <q-btn
            color="amber-9"
            text-color="dark"
            icon="refresh"
            label="Nova Partida"
            @click="resetaJogo"
            class="action-btn text-weight-bold"
            dense
            unelevated
          />
          <q-btn
            color="negative"
            icon="exit_to_app"
            label="Sair"
            @click="exitGame"
            class="action-btn text-weight-bold"
            dense
            unelevated
          />
        </div>
      </div>

      <!-- Estado e Mensagens da Rodada -->
      <div class="status-banner text-amber-3 text-center q-px-sm q-py-xs shadow-1">
        {{ mensagemStatus }}
      </div>

      <!-- Layout Principal da Mesa com Ações Laterais -->
      <div class="row justify-between items-center w-full q-gutter-xs" style="width: 100%">
        <!-- Painel Esquerdo: Truco e Progressão -->
        <div class="column q-gutter-xs items-center side-panel">
          <div class="text-caption text-amber-4 text-weight-bold">Apostas</div>
          <q-btn
            color="amber-8"
            text-color="dark"
            label="Truco (3)"
            class="side-action-btn text-weight-bold"
            :disabled="
              estadoJogo !== 'JOGANDO' || apostaAtual !== 1 || ultimoApostador === 'humano'
            "
            @click="cantarAcao('Truco')"
            dense
            unelevated
          />
          <q-btn
            color="amber-8"
            text-color="dark"
            label="Retruco (6)"
            class="side-action-btn text-weight-bold"
            :disabled="
              estadoJogo !== 'JOGANDO' || apostaAtual !== 3 || ultimoApostador === 'humano'
            "
            @click="cantarAcao('Retruco')"
            dense
            unelevated
          />
          <q-btn
            color="orange-9"
            label="Vale 9 (9)"
            class="side-action-btn text-weight-bold"
            :disabled="
              estadoJogo !== 'JOGANDO' || apostaAtual !== 6 || ultimoApostador === 'humano'
            "
            @click="cantarAcao('Vale 9')"
            dense
            unelevated
          />
          <q-btn
            color="deep-orange-9"
            label="Vale 12 (12)"
            class="side-action-btn text-weight-bold"
            :disabled="
              estadoJogo !== 'JOGANDO' || apostaAtual !== 9 || ultimoApostador === 'humano'
            "
            @click="cantarAcao('Vale 12')"
            dense
            unelevated
          />
        </div>

        <!-- Tapete de Jogo Luxuoso (Centro) -->
        <div class="game-table-felt column items-center q-gutter-sm q-pa-sm shadow-10 col">
          <!-- 1. Linha Superior: Oponente (IA) -->
          <div class="column items-center">
            <div class="text-caption text-blue-2 q-mb-none tracking-wider">Oponente (IA)</div>
            <div class="row q-gutter-sm justify-center">
              <div
                v-for="(cartaAI, idx) in maoAI"
                :key="'ai-' + idx"
                class="table-card-back flex flex-center"
              >
                <span v-if="cartaAI !== 'vazio'" class="text-subtitle1 text-amber-4">🂠</span>
                <span v-else class="text-grey-6 text-caption">-</span>
              </div>
            </div>
          </div>

          <!-- 2. Linha Central: Jogadas Oponente | Vira | Jogadas Jogador -->
          <div class="row justify-around items-center w-full q-py-none" style="width: 100%">
            <div class="column items-center">
              <div class="text-caption text-blue-2">Oponente</div>
              <div class="table-card flex flex-center">
                <img
                  v-if="cartaJogadaIA"
                  src="images/cartasTruco.png"
                  class="atlas-img"
                  :style="getSpriteStyle(cartaJogadaIA)"
                />
                <span v-else class="text-grey-5 text-caption">Mesa</span>
              </div>
            </div>

            <div class="column items-center">
              <div class="text-caption text-amber-3 text-weight-bold">Vira</div>
              <div class="table-card vira-glow flex flex-center">
                <img
                  v-if="cartaPinta"
                  src="images/cartasTruco.png"
                  class="atlas-img"
                  :style="getSpriteStyle(cartaPinta)"
                />
                <span v-else class="text-grey-6 text-caption">Vazio</span>
              </div>
            </div>

            <div class="column items-center">
              <div class="text-caption text-blue-2">Jogador</div>
              <div class="table-card flex flex-center">
                <img
                  v-if="cartaJogadaHumano"
                  src="images/cartasTruco.png"
                  class="atlas-img"
                  :style="getSpriteStyle(cartaJogadaHumano)"
                />
                <span v-else class="text-grey-5 text-caption">Mesa</span>
              </div>
            </div>
          </div>

          <!-- 3. Linha Inferior: Sua Mão -->
          <div class="column items-center">
            <div class="text-caption text-blue-2 q-mb-none tracking-wider">Sua Mão</div>
            <div class="row q-gutter-sm justify-center">
              <div
                v-for="(carta, index) in maoHumano"
                :key="'humano-' + index"
                class="player-card flex flex-center"
                :class="{
                  'disabled-card':
                    carta === 'vazio' || estadoJogo !== 'JOGANDO' || estadoDesafioAI !== '',
                }"
                @click="jogarCartaHumano(index)"
              >
                <img
                  v-if="carta !== 'vazio'"
                  src="images/cartasTruco.png"
                  class="atlas-img"
                  :style="getSpriteStyle(carta)"
                />
                <span v-else class="text-grey-5 text-caption">Jogado</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Painel Direito: Envido e Flor -->
        <div class="column q-gutter-xs items-center side-panel">
          <div class="text-caption text-amber-4 text-weight-bold">Tantos</div>
          <q-btn
            color="primary"
            label="Envido (2)"
            class="side-action-btn text-weight-bold"
            :disabled="estadoJogo !== 'JOGANDO' || envidoCantadoRodada || vazaAtual > 1"
            @click="cantarAcao('Envido')"
            dense
            unelevated
          />
          <q-btn
            color="primary"
            label="Real Envido (3)"
            class="side-action-btn text-weight-bold"
            :disabled="estadoJogo !== 'JOGANDO' || envidoCantadoRodada || vazaAtual > 1"
            @click="cantarAcao('Real Envido')"
            dense
            unelevated
          />
          <q-btn
            color="accent"
            label="Flor (3)"
            class="side-action-btn text-weight-bold"
            :disabled="estadoJogo !== 'JOGANDO' || florCantadaRodada || vazaAtual > 1"
            @click="cantarAcao('Flor')"
            dense
            unelevated
          />
        </div>
      </div>

      <!-- Diálogo de Desafios da IA (Envido / Truco) -->
      <q-dialog v-model="mostrarDialogoDesafio" persistent>
        <q-card class="bg-dark text-white q-pa-md text-center dialog-effect">
          <q-card-section>
            <q-icon
              name="local_fire_department"
              color="amber-7"
              size="48px"
              class="q-mb-xs animate-pulse"
            />
            <div class="text-h6 text-amber-5 text-weight-bold">Desafio da IA!</div>
            <div class="text-body2 q-mt-sm text-grey-3">{{ textoDesafioAI }}</div>
          </q-card-section>
          <q-card-actions align="center" class="q-gutter-sm q-mt-xs">
            <q-btn
              color="positive"
              label="Aceitar"
              class="text-weight-bold"
              @click="responderDesafio('Aceitar')"
              dense
              v-close-popup
              unelevated
            />
            <q-btn
              v-if="estadoDesafioAI === 'TRUCO'"
              color="amber-7"
              text-color="dark"
              label="Retruco"
              class="text-weight-bold"
              @click="responderDesafio('Retruco')"
              dense
              v-close-popup
              unelevated
            />
            <q-btn
              color="negative"
              label="Rejeitar"
              class="text-weight-bold"
              @click="responderDesafio('Rejeitar')"
              dense
              v-close-popup
              unelevated
            />
          </q-card-actions>
        </q-card>
      </q-dialog>

      <!-- Diálogo de Resposta da IA ao seu Desafio -->
      <q-dialog v-model="mostrarDialogoRespostaIA" persistent>
        <q-card class="bg-dark text-white q-pa-md text-center dialog-effect">
          <q-card-section>
            <q-icon name="smart_toy" color="cyan-4" size="48px" class="q-mb-xs" />
            <div class="text-h6 text-cyan-4 text-weight-bold">Resposta da IA</div>
            <div class="text-body2 q-mt-sm text-grey-3" v-html="textoRespostaIA"></div>
          </q-card-section>
          <q-card-actions align="center" class="q-mt-xs">
            <q-btn
              color="amber-5"
              text-color="dark"
              label="Continuar"
              dense
              v-close-popup
              unelevated
            />
          </q-card-actions>
        </q-card>
      </q-dialog>

      <!-- Ecrã de Fim de Partida -->
      <q-dialog v-model="fimPartida" persistent>
        <q-card class="bg-dark text-white q-pa-lg text-center dialog-effect">
          <q-card-section>
            <q-icon name="military_tech" color="amber-4" size="56px" class="q-mb-xs" />
            <div class="text-h5 text-amber-4 text-weight-bold">Fim da Partida!</div>
            <div class="text-body1 q-mt-sm text-grey-2">{{ mensagemFimPartida }}</div>
          </q-card-section>
          <q-card-actions align="center" class="q-gutter-sm q-mt-sm">
            <q-btn
              color="amber-5"
              text-color="dark"
              icon="replay"
              label="Jogar Novamente"
              class="text-weight-bold"
              @click="resetaJogo"
              dense
              v-close-popup
              unelevated
            />
            <q-btn
              color="grey-8"
              label="Menu Principal"
              class="text-weight-bold"
              @click="exitGame"
              dense
              v-close-popup
              unelevated
            />
          </q-card-actions>
        </q-card>
      </q-dialog>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();

const limitePartida = 12;
const pontosHumano = ref(0);
const pontosIA = ref(0);
const apostaAtual = ref(1);
const estadoJogo = ref('AGUARDANDO');
const mensagemStatus = ref('Clique em "Nova Partida" para começar!');

const cartaPinta = ref('');
const maoHumano = ref<string[]>([]);
const maoAI = ref<string[]>([]);

const cartaJogadaHumano = ref('');
const cartaJogadaIA = ref('');

const vazaAtual = ref(1);
const vitoriasHumano = ref(0);
const vitoriasIA = ref(0);
const vencedoresVazas = ref<number[]>([]);

const florCantadaRodada = ref(false);
const envidoCantadoRodada = ref(false);
const estadoDesafioAI = ref('');
const valorDesafioAI = ref(0);

const ultimoApostador = ref<'humano' | 'ai' | null>(null);

const mostrarDialogoDesafio = ref(false);
const textoDesafioAI = ref('');

const mostrarDialogoRespostaIA = ref(false);
const textoRespostaIA = ref('');

const fimPartida = ref(false);
const mensagemFimPartida = ref('');

const naipes = ['copa', 'oro', 'espada', 'basto'];

const getSpriteStyle = (carta: string) => {
  if (!carta || carta === 'vazio' || !carta.includes('_')) return {};
  const partes = carta.split('_');
  const num = parseInt(partes[0] ?? '0', 10);
  const naipe = (partes[1] ?? '').toLowerCase();

  let linha = 0;
  if (naipe === 'copa') linha = 0;
  else if (naipe === 'oro' || naipe === 'ouro') linha = 1;
  else if (naipe === 'espada') linha = 2;
  else if (naipe === 'basto') linha = 3;

  let coluna = num - 1;
  if (num >= 10) {
    coluna = num - 3;
  }

  return {
    transform: `translate(${-coluna * 10}%, ${-linha * 25}%)`,
  };
};

const criarBaralho = () => {
  let baralho: string[] = [];
  for (let i = 0; i < 4; i++) {
    const naipeAtual = naipes[i];
    for (let j = 1; j <= 12; j++) {
      if (j !== 8 && j !== 9) {
        baralho.push(`${j}_${naipeAtual}`);
      }
    }
  }
  return baralho.sort(() => 0.5 - Math.random());
};

const normalizarNaipe = (naipe: string) => {
  if (naipe === 'oro' || naipe === 'ouro') return 'oro';
  return naipe;
};

const alvosPinta = () => {
  if (!cartaPinta.value) return { alvoPerico: 11, alvoPerica: 10, naipePinta: 'copa' };
  const dados = cartaPinta.value.split('_');
  const numPinta = parseInt(dados[0] ?? '0', 10);
  const naipePinta = normalizarNaipe(dados[1] ?? 'copa');

  let alvoPerico = 11;
  let alvoPerica = 10;
  if (numPinta === 11) alvoPerico = 12;
  else if (numPinta === 10) alvoPerica = 12;

  return { alvoPerico, alvoPerica, naipePinta };
};

const obterForcaCarta = (carta: string) => {
  if (!carta || carta === 'vazio') return 0;
  const partes = carta.split('_');
  const num = parseInt(partes[0] ?? '0', 10);
  const naipe = normalizarNaipe(partes[1] ?? '');
  const pinta = alvosPinta();

  if (num === pinta.alvoPerico && naipe === pinta.naipePinta) return 16;
  if (num === pinta.alvoPerica && naipe === pinta.naipePinta) return 15;

  if (num === 1 && naipe === 'espada') return 14;
  if (num === 1 && naipe === 'basto') return 13;
  if (num === 7 && naipe === 'espada') return 12;
  if (num === 7 && naipe === 'oro') return 11;

  if (num === 3) return 10;
  if (num === 2) return 9;
  if (num === 1) return 8;
  if (num === 12) return 7;
  if (num === 11) return 6;
  if (num === 10) return 5;
  if (num === 7) return 4;
  if (num === 6) return 3;
  if (num === 5) return 2;
  if (num === 4) return 1;

  return 0;
};

const melhorForcaIA = () => {
  let melhor = 0;
  for (const c of maoAI.value) {
    if (c !== 'vazio') {
      const f = obterForcaCarta(c);
      if (f > melhor) melhor = f;
    }
  }
  return melhor;
};

const verificarTemFlor = (mao: string[]) => {
  if (!mao[0] || !mao[1] || !mao[2]) return false;
  const pinta = alvosPinta();

  let temPerico = false;
  let temPerica = false;
  const naipesMao: string[] = [];

  for (const carta of mao) {
    if (!carta || carta === 'vazio') continue;
    const partes = carta.split('_');
    const num = parseInt(partes[0] ?? '0', 10);
    const naipe = normalizarNaipe(partes[1] ?? '');

    if (num === pinta.alvoPerico && naipe === pinta.naipePinta) temPerico = true;
    if (num === pinta.alvoPerica && naipe === pinta.naipePinta) temPerica = true;
    naipesMao.push(naipe);
  }

  // Flor Reservada: tem perico e perica ao mesmo tempo
  if (temPerico && temPerica) return true;

  // 3 cartas do mesmo naipe
  if (naipesMao[0] === naipesMao[1] && naipesMao[1] === naipesMao[2]) return true;

  // 2 cartas do mesmo naipe + Perico ou Perica
  if (temPerico || temPerica) {
    const cartasFiltradas = mao.filter((c) => {
      const partes = c.split('_');
      const num = parseInt(partes[0] ?? '0', 10);
      const naipe = normalizarNaipe(partes[1] ?? '');
      const ehPeca =
        (num === pinta.alvoPerico && naipe === pinta.naipePinta) ||
        (num === pinta.alvoPerica && naipe === pinta.naipePinta);
      return !ehPeca;
    });

    if (cartasFiltradas.length === 2) {
      const n1 = normalizarNaipe(cartasFiltradas[0]!.split('_')[1] ?? '');
      const n2 = normalizarNaipe(cartasFiltradas[1]!.split('_')[1] ?? '');
      if (n1 === n2) return true;
    }
  }

  return false;
};

// Cálculo do Envido atualizado com Perico (30 + carta = 35 ex) e Perica (29 + carta = 34 ex)
const calcularPontosEnvido = (mao: string[]) => {
  const pinta = alvosPinta();
  let melhorEnvido = 0;
  let melhorSola = 0;

  let pericoPresente = false;
  let pericaPresente = false;
  let cartasDoNaipePinta: number[] = [];

  for (const carta of mao) {
    if (!carta || carta === 'vazio') continue;
    const partes = carta.split('_');
    const num = parseInt(partes[0] ?? '0', 10);
    const naipe = normalizarNaipe(partes[1] ?? '');

    let valorEnv = num >= 10 ? 0 : num;
    if (valorEnv > melhorSola) melhorSola = valorEnv;

    if (naipe === pinta.naipePinta) {
      if (num === pinta.alvoPerico) {
        pericoPresente = true;
      } else if (num === pinta.alvoPerica) {
        pericaPresente = true;
      } else {
        cartasDoNaipePinta.push(valorEnv);
      }
    }
  }

  // Perico vale 30 base + valor da carta da mesma pinta (ex: 30 + 5 = 35)
  if (pericoPresente) {
    let maxCombo = 30;
    for (const val of cartasDoNaipePinta) {
      const soma = 30 + val;
      if (soma > maxCombo) maxCombo = soma;
    }
    if (maxCombo > melhorEnvido) melhorEnvido = maxCombo;
  }

  // Perica vale 29 base + valor da carta da mesma pinta (ex: 29 + 5 = 34)
  if (pericaPresente) {
    let maxCombo = 29;
    for (const val of cartasDoNaipePinta) {
      const soma = 29 + val;
      if (soma > maxCombo) maxCombo = soma;
    }
    if (maxCombo > melhorEnvido) melhorEnvido = maxCombo;
  }

  // Pares normais do mesmo naipe
  for (let i = 0; i < mao.length; i++) {
    const c1 = mao[i];
    if (!c1 || c1 === 'vazio') continue;
    const [n1Str, s1] = c1.split('_');
    const num1 = parseInt(n1Str ?? '0', 10);
    const naipe1 = normalizarNaipe(s1 ?? '');
    const v1 = num1 >= 10 ? 0 : num1;

    for (let j = i + 1; j < mao.length; j++) {
      const c2 = mao[j];
      if (!c2 || c2 === 'vazio') continue;
      const [n2Str, s2] = c2.split('_');
      const num2 = parseInt(n2Str ?? '0', 10);
      const naipe2 = normalizarNaipe(s2 ?? '');
      const v2 = num2 >= 10 ? 0 : num2;

      if (naipe1 === naipe2) {
        const isP1Perico = num1 === pinta.alvoPerico && naipe1 === pinta.naipePinta;
        const isP1Perica = num1 === pinta.alvoPerica && naipe1 === pinta.naipePinta;
        const isP2Perico = num2 === pinta.alvoPerico && naipe2 === pinta.naipePinta;
        const isP2Perica = num2 === pinta.alvoPerica && naipe2 === pinta.naipePinta;

        let soma = 0;
        if (isP1Perico || isP2Perico) {
          soma = 30 + (isP1Perico ? v2 : v1);
        } else if (isP1Perica || isP2Perica) {
          soma = 29 + (isP1Perica ? v2 : v1);
        } else {
          soma = v1 + v2 + 20;
        }
        if (soma > melhorEnvido) melhorEnvido = soma;
      }
    }
  }

  if (pericoPresente && 30 > melhorEnvido) melhorEnvido = 30;
  if (pericaPresente && 29 > melhorEnvido) melhorEnvido = 29;

  if (melhorEnvido === 0) melhorEnvido = melhorSola;
  return melhorEnvido;
};

const resetaJogo = () => {
  estadoJogo.value = 'AGUARDANDO';
  pontosHumano.value = 0;
  pontosIA.value = 0;
  apostaAtual.value = 1;
  vazaAtual.value = 1;
  vitoriasHumano.value = 0;
  vitoriasIA.value = 0;
  vencedoresVazas.value = [];
  florCantadaRodada.value = false;
  envidoCantadoRodada.value = false;
  estadoDesafioAI.value = '';
  ultimoApostador.value = null;
  cartaJogadaHumano.value = '';
  cartaJogadaIA.value = '';
  maoHumano.value = [];
  maoAI.value = [];
  fimPartida.value = false;
  mensagemStatus.value = 'Nova partida pronta. Clique em "Nova Partida" para iniciar.';
  iniciarPartida();
};

const iniciarPartida = () => {
  apostaAtual.value = 1;
  vazaAtual.value = 1;
  vitoriasHumano.value = 0;
  vitoriasIA.value = 0;
  vencedoresVazas.value = [];
  florCantadaRodada.value = false;
  envidoCantadoRodada.value = false;
  estadoDesafioAI.value = '';
  ultimoApostador.value = null;
  cartaJogadaHumano.value = '';
  cartaJogadaIA.value = '';
  estadoJogo.value = 'JOGANDO';

  const baralho = criarBaralho();
  cartaPinta.value = baralho[0] ?? '';
  baralho.shift();

  maoHumano.value = [baralho[0] ?? '', baralho[1] ?? '', baralho[2] ?? ''];
  maoAI.value = [baralho[3] ?? '', baralho[4] ?? '', baralho[5] ?? ''];

  mensagemStatus.value = 'Rodada iniciada! Jogue uma carta ou utilize os botões laterais.';

  if (verificarTemFlor(maoAI.value)) {
    florCantadaRodada.value = true;
    pontosIA.value += 3;
    mensagemStatus.value = '🌸 A IA cantou Flor e anota 3 pontos!';
    verificarFimPartida();
  }

  if (estadoJogo.value === 'JOGANDO' && !envidoCantadoRodada.value) {
    const ptsAI = calcularPontosEnvido(maoAI.value);
    if (ptsAI >= 28) {
      envidoCantadoRodada.value = true;
      estadoDesafioAI.value = 'ENVIDO';
      ultimoApostador.value = 'ai';
      valorDesafioAI.value = ptsAI >= 31 ? 3 : 2;
      textoDesafioAI.value = `A IA canta ${valorDesafioAI.value === 3 ? 'Real Envido' : 'Envido'}! Deseja aceitar?`;
      mostrarDialogoDesafio.value = true;
    }
  }
};

const verificarFimPartida = () => {
  if (pontosHumano.value >= limitePartida) {
    mensagemFimPartida.value = `🎉 Parabéns! Venceu a partida por ${pontosHumano.value} a ${pontosIA.value}!`;
    fimPartida.value = true;
    estadoJogo.value = 'FIM';
  } else if (pontosIA.value >= limitePartida) {
    mensagemFimPartida.value = `🤖 Fim do jogo! A IA venceu a partida por ${pontosIA.value} a ${pontosHumano.value}.`;
    fimPartida.value = true;
    estadoJogo.value = 'FIM';
  }
};

const encerrarRodada = (vencedor: number) => {
  if (vencedor === 1) {
    pontosHumano.value += apostaAtual.value;
    mensagemStatus.value = `🏆 Você venceu a rodada e anota ${apostaAtual.value} ponto(s)!`;
  } else if (vencedor === 2) {
    pontosIA.value += apostaAtual.value;
    mensagemStatus.value = `🤖 A IA venceu a rodada e anota ${apostaAtual.value} ponto(s)!`;
  } else {
    mensagemStatus.value = '🤝 Rodada empatada — ninguém anota pontos.';
  }

  verificarFimPartida();
  if (estadoJogo.value !== 'FIM') {
    setTimeout(() => {
      iniciarPartida();
    }, 2000);
  }
};

const ladoVencedorRodada = (): number => {
  if (vencedoresVazas.value.length >= 1 && vencedoresVazas.value[0] === 0) {
    if (vencedoresVazas.value.length >= 3) {
      const terceira = vencedoresVazas.value[2];
      if (terceira !== undefined && terceira !== 0) return terceira;
    }
    return 1;
  }
  if (vitoriasHumano.value > vitoriasIA.value) return 1;
  if (vitoriasIA.value > vitoriasHumano.value) return 2;
  for (const r of vencedoresVazas.value) {
    if (r !== 0) return r;
  }
  return 1;
};

const jogarCartaHumano = (index: number) => {
  if (estadoJogo.value !== 'JOGANDO' || estadoDesafioAI.value !== '') return;
  const carta = maoHumano.value[index];
  if (!carta || carta === 'vazio') return;

  maoHumano.value[index] = 'vazio';
  cartaJogadaHumano.value = carta;
  mensagemStatus.value = 'Você jogou sua carta.';

  const cartasValidasAI = maoAI.value.filter((c) => c !== 'vazio');
  if (cartasValidasAI.length === 0) return;

  const idxAI = Math.floor(Math.random() * cartasValidasAI.length);
  const cartaAI = cartasValidasAI[idxAI];
  if (!cartaAI) return;
  const realIdxAI = maoAI.value.indexOf(cartaAI);
  maoAI.value[realIdxAI] = 'vazio';
  cartaJogadaIA.value = cartaAI;

  if (apostaAtual.value === 1 && melhorForcaIA() >= 10 && Math.random() < 0.5) {
    estadoDesafioAI.value = 'TRUCO';
    ultimoApostador.value = 'ai';
    valorDesafioAI.value = 3;
    textoDesafioAI.value = 'A IA cantou TRUCO! Deseja aceitar?';
    mostrarDialogoDesafio.value = true;
    return;
  }

  processarResultadoVaza(carta, cartaAI);
};

const processarResultadoVaza = (cHumano: string, cAI: string) => {
  const fHumano = obterForcaCarta(cHumano);
  const fAI = obterForcaCarta(cAI);

  let resVaza = 0;
  if (fHumano > fAI) {
    vitoriasHumano.value++;
    resVaza = 1;
    mensagemStatus.value += ` | Você venceu a vaza ${vazaAtual.value}!`;
  } else if (fAI > fHumano) {
    vitoriasIA.value++;
    resVaza = 2;
    mensagemStatus.value += ` | A IA venceu a vaza ${vazaAtual.value}!`;
  } else {
    mensagemStatus.value += ` | Vaza empatada!`;
  }

  vencedoresVazas.value.push(resVaza);
  vazaAtual.value++;

  if (
    vitoriasHumano.value >= 2 ||
    vitoriasIA.value >= 2 ||
    vazaAtual.value > 3 ||
    (vencedoresVazas.value.length === 2 && vencedoresVazas.value[0] === 0)
  ) {
    const vencedor = ladoVencedorRodada();
    encerrarRodada(vencedor);
  }
};

const responderDesafio = (resposta: string) => {
  if (estadoDesafioAI.value === 'ENVIDO') {
    if (resposta === 'Aceitar') {
      const ptsHumano = calcularPontosEnvido(maoHumano.value);
      const ptsAI = calcularPontosEnvido(maoAI.value);
      if (ptsHumano >= ptsAI) {
        pontosHumano.value += valorDesafioAI.value;
        mensagemStatus.value = `🎯 Envido aceito (${ptsHumano} x ${ptsAI}) — Você anota ${valorDesafioAI.value} pts!`;
      } else {
        pontosIA.value += valorDesafioAI.value;
        mensagemStatus.value = `🎯 Envido aceito (${ptsHumano} x ${ptsAI}) — A IA anota ${valorDesafioAI.value} pts!`;
      }
      verificarFimPartida();
    } else {
      pontosIA.value += 1;
      mensagemStatus.value = '🤖 Você rejeitou o Envido. A IA anota 1 ponto.';
      verificarFimPartida();
    }
    estadoDesafioAI.value = '';
  } else if (estadoDesafioAI.value === 'TRUCO') {
    if (resposta === 'Aceitar') {
      apostaAtual.value = 3;
      ultimoApostador.value = 'ai';
      mensagemStatus.value = '✨ Você aceitou o Truco! A rodada vale 3 pontos.';
      estadoDesafioAI.value = '';
      if (cartaJogadaHumano.value && cartaJogadaIA.value) {
        processarResultadoVaza(cartaJogadaHumano.value, cartaJogadaIA.value);
      }
      return;
    } else if (resposta === 'Retruco') {
      apostaAtual.value = 6;
      ultimoApostador.value = 'humano';
      mensagemStatus.value = '🔥 Você retrucou! A rodada vale 6 pontos.';
      estadoDesafioAI.value = '';
      if (cartaJogadaHumano.value && cartaJogadaIA.value) {
        processarResultadoVaza(cartaJogadaHumano.value, cartaJogadaIA.value);
      }
      return;
    } else {
      pontosIA.value += 1;
      mensagemStatus.value = '🤖 Você rejeitou o Truco. A IA anota 1 ponto.';
      encerrarRodada(2);
    }
    estadoDesafioAI.value = '';
  }
};

const cantarAcao = (acao: string) => {
  if (acao === 'Flor') {
    if (vazaAtual.value > 1 || florCantadaRodada.value) {
      mensagemStatus.value = '❌ A Flor não pode ser cantada agora.';
      return;
    }
    if (!verificarTemFlor(maoHumano.value)) {
      mensagemStatus.value = '❌ Você não tem Flor na mão!';
      return;
    }
    florCantadaRodada.value = true;
    pontosHumano.value += 3;
    mensagemStatus.value = '🌸 Você cantou Flor e anota 3 pontos!';
    verificarFimPartida();
  } else if (acao === 'Envido' || acao === 'Real Envido') {
    if (vazaAtual.value > 1 || envidoCantadoRodada.value) {
      mensagemStatus.value = '❌ O Envido não pode ser cantado agora.';
      return;
    }
    envidoCantadoRodada.value = true;
    ultimoApostador.value = 'humano';
    const valorEnvido = acao === 'Real Envido' ? 3 : 2;
    const ptsH = calcularPontosEnvido(maoHumano.value);
    const ptsA = calcularPontosEnvido(maoAI.value);

    if (ptsA >= 26 || Math.random() < 0.65) {
      if (ptsH >= ptsA) {
        pontosHumano.value += valorEnvido;
        textoRespostaIA.value = `A IA <b>ACEITOU</b> o ${acao}!<br>Seus pontos: ${ptsH} | Pontos da IA: ${ptsA}<br><br>🏆 <b>Você venceu o Envido e anota ${valorEnvido} pontos!</b>`;
      } else {
        pontosIA.value += valorEnvido;
        textoRespostaIA.value = `A IA <b>ACEITOU</b> o ${acao}!<br>Seus pontos: ${ptsH} | Pontos da IA: ${ptsA}<br><br>🤖 <b>A IA venceu o Envido e anota ${valorEnvido} pontos.</b>`;
      }
    } else {
      pontosHumano.value += 1;
      textoRespostaIA.value = `A IA <b>REJEITOU</b> o ${acao}!<br><br>🏆 <b>Você anota 1 ponto por desistência da IA.</b>`;
    }
    mostrarDialogoRespostaIA.value = true;
    verificarFimPartida();
  } else if (acao === 'Truco' || acao === 'Retruco' || acao === 'Vale 9' || acao === 'Vale 12') {
    const novoValor = acao === 'Truco' ? 3 : acao === 'Retruco' ? 6 : acao === 'Vale 9' ? 9 : 12;
    const forcaAI = melhorForcaIA();

    ultimoApostador.value = 'humano';

    if (forcaAI >= 7 || Math.random() < 0.7) {
      apostaAtual.value = novoValor;
      textoRespostaIA.value = `A IA <b>ACEITOU</b> o seu ${acao}!<br>A rodada passa a valer <b>${novoValor} pontos</b>.`;
      ultimoApostador.value = 'ai';

      if (forcaAI >= 12 && novoValor < 12 && Math.random() < 0.45) {
        const retrucoValor = novoValor === 3 ? 6 : novoValor === 6 ? 9 : 12;
        apostaAtual.value = retrucoValor;
        ultimoApostador.value = 'ai';
        textoRespostaIA.value = `A IA aceitou e <b>RETRUCOU</b>!<br>A rodada sobe para <b>${retrucoValor} pontos</b>!`;
      }
    } else {
      const pontosGanho = apostaAtual.value === 1 ? 1 : Math.floor(apostaAtual.value / 2);
      pontosHumano.value += pontosGanho;
      textoRespostaIA.value = `A IA <b>REJEITOU</b> o seu ${acao}!<br><br>🏆 <b>Você anota ${pontosGanho} ponto(s).</b>`;
      mostrarDialogoRespostaIA.value = true;
      verificarFimPartida();
      if (estadoJogo.value !== 'FIM') {
        setTimeout(() => {
          iniciarPartida();
        }, 2000);
      }
      return;
    }
    mostrarDialogoRespostaIA.value = true;
  }
};

const exitGame = () => {
  router.push('/');
};

onMounted(() => {
  resetaJogo();
});
</script>

<style scoped>
.truco-page {
  background: radial-gradient(circle at center, #0e2a6b 0%, #061336 100%);
  min-height: 100vh;
  width: 100%;
}

.game-container {
  background: rgba(15, 35, 95, 0.85);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 20px;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.6);
}

.status-banner {
  width: 100%;
  background: rgba(0, 0, 0, 0.35);
  border-left: 4px solid #ffb300;
  border-radius: 0 8px 8px 0;
  font-weight: 500;
  font-size: 13px;
  letter-spacing: 0.3px;
}

.side-panel {
  width: 105px;
  background: rgba(0, 0, 0, 0.25);
  padding: 8px;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.side-action-btn {
  width: 100%;
  font-size: 9px;
  padding: 2px 0;
  border-radius: 6px;
  letter-spacing: 0.2px;
}

.game-table-felt {
  background: radial-gradient(circle, #105c47 0%, #063024 100%);
  border: 3px solid rgba(212, 175, 55, 0.35);
  border-radius: 16px;
  box-shadow:
    inset 0 4px 15px rgba(0, 0, 0, 0.8),
    0 6px 20px rgba(0, 0, 0, 0.4);
}

.table-card {
  position: relative;
  width: 75px;
  height: 95px;
  background-color: #ffffff;
  border-radius: 8px;
  overflow: hidden;
  border: 2px solid rgba(255, 255, 255, 0.8);
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
}

.table-card-back {
  width: 75px;
  height: 95px;
  background: linear-gradient(135deg, #1565c0 0%, #0d47a1 100%);
  border-radius: 8px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.4);
}

.vira-glow {
  box-shadow: 0 0 12px rgba(255, 193, 7, 0.6);
  border-color: #ffc107 !important;
}

.player-card {
  position: relative;
  width: 85px;
  height: 110px;
  background-color: #ffffff;
  border-radius: 10px;
  cursor: pointer;
  overflow: hidden;
  box-shadow: 0 6px 14px rgba(0, 0, 0, 0.5);
  border: 2px solid #ffffff;
  transition:
    transform 0.25s cubic-bezier(0.4, 0, 0.2, 1),
    box-shadow 0.25s ease;
}

.player-card:hover:not(.disabled-card) {
  transform: translateY(-8px) scale(1.03);
  box-shadow: 0 10px 22px rgba(0, 0, 0, 0.7);
  border-color: #ffc107;
}

.disabled-card {
  opacity: 0.45;
  cursor: not-allowed;
  filter: grayscale(70%);
}

.atlas-img {
  position: absolute;
  top: 0;
  left: 0;
  width: 1000%;
  height: 400%;
  max-width: none;
  pointer-events: none;
}

.game-chip {
  background: rgba(0, 0, 0, 0.4) !important;
  color: #fff !important;
  border: 1px solid rgba(255, 255, 255, 0.15);
  font-weight: 500;
}

.game-chip-gold {
  background: linear-gradient(135deg, #ffa000 0%, #ff6f00 100%) !important;
  color: #fff !important;
  font-weight: bold;
  box-shadow: 0 2px 6px rgba(255, 111, 0, 0.4);
}

.action-btn {
  width: 120px;
  font-size: 10px;
  border-radius: 6px;
  letter-spacing: 0.4px;
}

.dialog-effect {
  background: rgba(13, 27, 75, 0.95) !important;
  backdrop-filter: blur(14px);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.15);
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.7);
  min-width: 300px;
}
</style>
