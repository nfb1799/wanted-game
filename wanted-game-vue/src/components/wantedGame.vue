<template>
  <div class="wanted-game">
    <h2>WANTED!</h2>
    <div class="target-face">
      Find: <span>{{ targetEmoji }}</span>
    </div>
    <div class="score-display">
      Score: <b>{{ score }}</b>
    </div>
    <div v-if="gameOver" class="game-over">
      <h3>Game Over!</h3>
      <button @click="restart">Restart</button>
    </div>
    <div v-else>
      <div class="game-area" ref="gameArea">
        <div
          v-for="face in faces"
          :key="face.id"
          class="face"
          :style="{
            left: face.x + 'px',
            top: face.y + 'px'
          }"
          @click="selectFace(face)"
        >
          {{ face.emoji }}
        </div>
      </div>
      <div class="timer-container">
        <div class="rope-timer-wrapper">
          <div class="rope-bg"></div>
          <div class="rope-burn" :style="{ width: ropeWidth + '%' }"></div>
          <div class="rope-fire" :style="{ left: ropeWidth + '%' }">🔥</div>
        </div>
        <div class="timer-label">
          Time left: {{ timeLeft.toFixed(1) }}s
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, nextTick, watch } from 'vue';

const EMOJIS = [
  "😀", "😎", "😡", "😱", "😈", "😺", "🤠", "🥸",
  "👻", "🤖", "🧙", "🧑", "🐵", "🐸", "🐼", "🦁", "🐻", "🦊", "🐶"
];
const AREA_WIDTH = 400;
const AREA_HEIGHT = 270;
const FACE_SIZE = 40;
const START_DISTRACTORS = 8;
const DISTRACTOR_INCREASE = 2;

// Timer settings
const START_TIME = 7; // seconds for first round
const MIN_TIME = 2.2; // fastest possible round
const TIME_DECREASE_PER_ROUND = 0.5; // seconds less per round, min capped

const faces = ref([]);
const targetEmoji = ref("");
const score = ref(0);
const round = ref(1);
const gameOver = ref(false);

const timer = ref(null);
const timeLeft = ref(START_TIME);
const roundMaxTime = ref(START_TIME);
const ropeWidth = ref(100); // percent, for animation

let animationId = null;

function getRandomEmoji(excludeList = []) {
  let available = EMOJIS.filter(e => !excludeList.includes(e));
  return available[Math.floor(Math.random() * available.length)];
}

function randomPosition() {
  return {
    x: Math.random() * (AREA_WIDTH - FACE_SIZE),
    y: Math.random() * (AREA_HEIGHT - FACE_SIZE)
  };
}

function randomVelocity() {
  const speed = 1.5 + Math.random() * 2;
  const angle = Math.random() * Math.PI * 2;
  return {
    dx: Math.cos(angle) * speed,
    dy: Math.sin(angle) * speed
  };
}

function getRoundTime() {
  return Math.max(START_TIME - (round.value - 1) * TIME_DECREASE_PER_ROUND, MIN_TIME);
}

function setupRound() {
  faces.value = [];
  // Choose target emoji
  targetEmoji.value = getRandomEmoji();
  // Distractor pool (exclude target, no duplicates)
  let distractorCount = START_DISTRACTORS + (round.value - 1) * DISTRACTOR_INCREASE;
  let available = EMOJIS.filter(e => e !== targetEmoji.value);
  distractorCount = Math.min(distractorCount, available.length);
  let distractors = [];
  while (distractors.length < distractorCount) {
    let next = getRandomEmoji([targetEmoji.value, ...distractors]);
    distractors.push(next);
  }
  // Place target
  const targetPos = randomPosition();
  const targetVel = randomVelocity();
  faces.value.push({
    id: 0,
    emoji: targetEmoji.value,
    x: targetPos.x,
    y: targetPos.y,
    dx: targetVel.dx,
    dy: targetVel.dy,
    isTarget: true
  });
  // Place each distractor (no duplicates)
  distractors.forEach((emoji, i) => {
    const pos = randomPosition();
    const vel = randomVelocity();
    faces.value.push({
      id: i + 1,
      emoji,
      x: pos.x,
      y: pos.y,
      dx: vel.dx,
      dy: vel.dy,
      isTarget: false
    });
  });
  // Reset timer for this round
  roundMaxTime.value = getRoundTime();
  timeLeft.value = roundMaxTime.value;
  ropeWidth.value = 100;
}

function moveFaces() {
  faces.value.forEach(face => {
    face.x += face.dx;
    face.y += face.dy;
    // Bounce off walls
    if (face.x <= 0 || face.x >= AREA_WIDTH - FACE_SIZE) face.dx *= -1;
    if (face.y <= 0 || face.y >= AREA_HEIGHT - FACE_SIZE) face.dy *= -1;
    face.x = Math.max(0, Math.min(face.x, AREA_WIDTH - FACE_SIZE));
    face.y = Math.max(0, Math.min(face.y, AREA_HEIGHT - FACE_SIZE));
  });
}

function animate() {
  moveFaces();
  animationId = requestAnimationFrame(animate);
}

function selectFace(face) {
  if (gameOver.value) return;
  if (face.isTarget) {
    score.value += 1;
    round.value += 1;
    setupRound();
  } else {
    endGame();
  }
}

function endGame() {
  gameOver.value = true;
  cancelAnimationFrame(animationId);
  clearTimer();
}

function restart() {
  score.value = 0;
  round.value = 1;
  gameOver.value = false;
  setupRound();
  nextTick(() => {
    animate();
    startTimer();
  });
}

function startTimer() {
  clearTimer();
  let lastTime = performance.now();
  function tick(now) {
    const elapsed = (now - lastTime) / 1000;
    lastTime = now;
    if (!gameOver.value) {
      timeLeft.value = Math.max(0, timeLeft.value - elapsed);
      ropeWidth.value = Math.max(0, 100 * (timeLeft.value / roundMaxTime.value));
      if (timeLeft.value > 0) {
        timer.value = requestAnimationFrame(tick);
      } else {
        endGame();
      }
    }
  }
  timer.value = requestAnimationFrame(tick);
}

function clearTimer() {
  if (timer.value) {
    cancelAnimationFrame(timer.value);
    timer.value = null;
  }
}

onMounted(() => {
  setupRound();
  animate();
  startTimer();
});

onUnmounted(() => {
  if (animationId) cancelAnimationFrame(animationId);
  clearTimer();
});

watch([score, round], ([s, r], [ps, pr]) => {
  // When round/score changes (after correct click), reset timer for new round
  if (!gameOver.value) {
    clearTimer();
    timeLeft.value = roundMaxTime.value;
    ropeWidth.value = 100;
    startTimer();
  }
});
</script>

<style scoped>
.wanted-game {
  max-width: 430px;
  margin: 24px auto;
  text-align: center;
  font-family: sans-serif;
}
.target-face {
  font-size: 32px;
  margin: 16px 0;
}
.score-display {
  font-size: 24px;
  margin-bottom: 10px;
}
.game-area {
  position: relative;
  width: 400px;
  height: 270px;
  border: 4px solid #444;
  margin: 0 auto;
  background: #e3f2fd;
  overflow: hidden;
  border-radius: 12px;
}
.face {
  position: absolute;
  font-size: 38px;
  width: 40px;
  height: 40px;
  cursor: pointer;
  user-select: none;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: filter 0.1s;
}
.face:active {
  filter: brightness(0.8);
}
.timer-container {
  margin-top: 16px;
  width: 400px;
  margin-left: auto;
  margin-right: auto;
}
.rope-timer-wrapper {
  position: relative;
  width: 380px;
  height: 24px;
  margin: 0 auto;
}
.rope-bg {
  position: absolute;
  left: 0;
  top: 8px;
  width: 100%;
  height: 8px;
  background: repeating-linear-gradient(
    90deg,
    #bbb 0 12px,
    #d8b08c 12px 24px
  );
  border-radius: 4px;
  box-shadow: 0 1px 2px #8888 inset;
}
.rope-burn {
  position: absolute;
  left: 0;
  top: 8px;
  height: 8px;
  background: linear-gradient(to right, #e25822, #f7cf7c 60%, #fff2ac 100%);
  border-radius: 4px 0 0 4px;
  box-shadow: 0 1px 2px #f7cf7c inset;
  transition: width 0.08s linear;
  z-index: 1;
}
.rope-fire {
  position: absolute;
  top: 0px;
  transform: translateX(-50%);
  font-size: 24px;
  transition: left 0.08s linear;
  filter: drop-shadow(0 0 4px #f90);
  pointer-events: none;
}
.timer-label {
  margin-top: 6px;
  font-size: 18px;
  font-family: monospace;
}
.game-over {
  margin-top: 30px;
}
</style>