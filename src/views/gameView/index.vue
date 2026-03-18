<template>
  <div class="gobang-container">
    <!-- 背景装饰层 -->
    <div class="bg-particles">
      <span
        v-for="i in 24"
        :key="i"
        class="particle"
        :style="particleStyle(i)"
      ></span>
    </div>
    <div class="board-bg"></div>
    <div class="page-glow"></div>

    <div class="game-wrapper">
      <!-- 标题区 -->
      <header class="game-header">
        <h1 class="title">
          <span class="title-flame"></span>
          长离 · 焚棋
          <span class="title-flame"></span>
        </h1>
        <p class="subtitle">对弈者，当如离火，落子无悔</p>
      </header>

      <!-- 棋盘区 -->
      <div class="board-wrapper">
        <canvas
          ref="canvasRef"
          class="board-canvas"
          :width="boardSize"
          :height="boardSize"
          @click="handleCanvasClick"
        ></canvas>
        <div class="turn-indicator" :class="{ 'ai-turn': currentPlayer === 2 }">
          <span class="indicator-text">{{
            currentPlayer === 1 ? "你的回合" : "长离·谋棋"
          }}</span>
          <span class="indicator-flame"></span>
        </div>
      </div>

      <!-- 状态和按钮区 -->
      <div class="game-footer">
        <div class="status-panel">
          <div class="status-item">
            <span class="status-label">局数</span>
            <span class="status-value">{{ gameCount }}</span>
          </div>
          <div class="status-item">
            <span class="status-label">胜者</span>
            <span class="status-value winner-text">{{ winnerText }}</span>
          </div>
        </div>
        <button class="restart-btn" @click="restartGame">
          <span class="btn-flame"></span>
          焚局重启
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, computed } from "vue";
import changliImg from "@/assets/avatar/changli.png";
import manImg from "@/assets/avatar/man.png";

// ========== 游戏配置 ==========
const BOARD_SIZE = 15;
const CELL_SIZE = 40;
const PADDING = 40;
const boardSize = (BOARD_SIZE - 1) * CELL_SIZE + PADDING * 2;

// 游戏状态
const board = ref<number[][]>(
  Array(BOARD_SIZE)
    .fill(0)
    .map(() => Array(BOARD_SIZE).fill(0))
);
const currentPlayer = ref<1 | 2>(1);
const gameOver = ref(false);
const winner = ref<0 | 1 | 2>(0);
const gameCount = ref(0);
const winningStones = ref<[number, number][] | null>(null); // 存储胜利的五子坐标

// 图片资源
const changliImage = ref<HTMLImageElement | null>(null);
const manImage = ref<HTMLImageElement | null>(null);
const imagesLoaded = ref(false);
const canvasRef = ref<HTMLCanvasElement | null>(null);

const winnerText = computed(() => {
  if (!gameOver.value) return "进行中";
  if (winner.value === 1) return "你胜";
  if (winner.value === 2) return "长离胜";
  return "平局";
});

// ========== 初始化图片 ==========
function loadImages() {
  const img1 = new Image();
  img1.src = changliImg;
  img1.onload = () => {
    changliImage.value = img1;
    checkImagesLoaded();
  };
  const img2 = new Image();
  img2.src = manImg;
  img2.onload = () => {
    manImage.value = img2;
    checkImagesLoaded();
  };
}

function checkImagesLoaded() {
  if (changliImage.value && manImage.value) {
    imagesLoaded.value = true;
    drawBoard();
  }
}

// ========== 绘制棋盘 ==========
function drawBoard() {
  const canvas = canvasRef.value;
  if (!canvas) return;
  const ctx = canvas.getContext("2d");
  if (!ctx) return;

  ctx.clearRect(0, 0, boardSize, boardSize);

  // 木质背景纹理
  ctx.fillStyle = "#2b1a0e";
  ctx.fillRect(0, 0, boardSize, boardSize);
  // 添加细微木纹
  ctx.globalAlpha = 0.05;
  for (let i = 0; i < 20; i++) {
    ctx.beginPath();
    ctx.moveTo(0, Math.random() * boardSize);
    ctx.lineTo(boardSize, Math.random() * boardSize);
    ctx.strokeStyle = "#d4a373";
    ctx.lineWidth = 1;
    ctx.stroke();
  }
  ctx.globalAlpha = 1;

  // 绘制网格线
  ctx.lineWidth = 1.5;
  ctx.strokeStyle = "#c98b5e";
  ctx.shadowColor = "#ff974f";
  ctx.shadowBlur = 4;

  for (let i = 0; i < BOARD_SIZE; i++) {
    const pos = PADDING + i * CELL_SIZE;
    ctx.beginPath();
    ctx.moveTo(PADDING, pos);
    ctx.lineTo(boardSize - PADDING, pos);
    ctx.stroke();

    ctx.beginPath();
    ctx.moveTo(pos, PADDING);
    ctx.lineTo(pos, boardSize - PADDING);
    ctx.stroke();
  }

  // 绘制星位
  ctx.shadowBlur = 8;
  const starPositions = [3, 7, 11];
  for (let r of starPositions) {
    for (let c of starPositions) {
      if (r < BOARD_SIZE && c < BOARD_SIZE) {
        ctx.beginPath();
        ctx.arc(
          PADDING + r * CELL_SIZE,
          PADDING + c * CELL_SIZE,
          5,
          0,
          2 * Math.PI
        );
        ctx.fillStyle = "#d4a373";
        ctx.fill();
        ctx.shadowBlur = 12;
        ctx.fillStyle = "#ffb07c";
        ctx.fill();
      }
    }
  }
  ctx.shadowBlur = 0;

  // 先绘制胜利光晕（在棋子下方）
  if (winningStones.value) {
    ctx.shadowColor = "#ffd700";
    ctx.shadowBlur = 25;
    ctx.globalAlpha = 0.6;
    for (const [row, col] of winningStones.value) {
      const x = PADDING + row * CELL_SIZE;
      const y = PADDING + col * CELL_SIZE;
      ctx.beginPath();
      ctx.arc(x, y, 18, 0, 2 * Math.PI);
      ctx.fillStyle = "#ffaa33";
      ctx.fill();
    }
    ctx.shadowBlur = 0;
    ctx.globalAlpha = 1;
  }

  // 绘制所有棋子
  for (let row = 0; row < BOARD_SIZE; row++) {
    for (let col = 0; col < BOARD_SIZE; col++) {
      const val = board.value[row][col];
      if (val === 0) continue;

      const x = PADDING + row * CELL_SIZE;
      const y = PADDING + col * CELL_SIZE;

      // 根据当前回合增加光晕
      ctx.shadowColor = val === 1 ? "#ffb47b" : "#ff4f1e";
      ctx.shadowBlur = currentPlayer.value === val && !gameOver.value ? 25 : 15;

      if (val === 1 && manImage.value) {
        ctx.drawImage(manImage.value, x - 15, y - 15, 30, 30);
      } else if (val === 2 && changliImage.value) {
        ctx.drawImage(changliImage.value, x - 15, y - 15, 30, 30);
      } else {
        ctx.beginPath();
        ctx.arc(x, y, 14, 0, 2 * Math.PI);
        ctx.fillStyle = val === 1 ? "#ffb47b" : "#ff4f1e";
        ctx.fill();
        ctx.strokeStyle = "#fff";
        ctx.lineWidth = 2;
        ctx.stroke();
      }
    }
  }
  ctx.shadowBlur = 0;
}

// ========== 胜负与平局检测 ==========
function checkWinner(
  row: number,
  col: number,
  player: number
): [number, number][] | null {
  const directions = [
    [1, 0],
    [0, 1],
    [1, 1],
    [1, -1],
  ];
  for (let [dx, dy] of directions) {
    const stones: [number, number][] = [];
    stones.push([row, col]);

    // 正方向
    for (let step = 1; step < 5; step++) {
      const nr = row + dx * step;
      const nc = col + dy * step;
      if (
        nr < 0 ||
        nr >= BOARD_SIZE ||
        nc < 0 ||
        nc >= BOARD_SIZE ||
        board.value[nr][nc] !== player
      )
        break;
      stones.push([nr, nc]);
    }
    // 负方向
    for (let step = 1; step < 5; step++) {
      const nr = row - dx * step;
      const nc = col - dy * step;
      if (
        nr < 0 ||
        nr >= BOARD_SIZE ||
        nc < 0 ||
        nc >= BOARD_SIZE ||
        board.value[nr][nc] !== player
      )
        break;
      stones.unshift([nr, nc]); // 插入前面，保持顺序
    }
    if (stones.length >= 5) {
      return stones.slice(0, 5); // 返回正好五颗
    }
  }
  return null;
}

function isBoardFull(): boolean {
  for (let row = 0; row < BOARD_SIZE; row++) {
    for (let col = 0; col < BOARD_SIZE; col++) {
      if (board.value[row][col] === 0) return false;
    }
  }
  return true;
}

// ========== 玩家落子 ==========
function placePiece(row: number, col: number) {
  if (
    gameOver.value ||
    board.value[row][col] !== 0 ||
    currentPlayer.value !== 1
  )
    return false;

  board.value[row][col] = 1;
  drawBoard();

  const win = checkWinner(row, col, 1);
  if (win) {
    winningStones.value = win;
    gameOver.value = true;
    winner.value = 1;
    gameCount.value++;
    drawBoard(); // 重绘显示高亮
  } else if (isBoardFull()) {
    winningStones.value = null;
    gameOver.value = true;
    winner.value = 0;
    gameCount.value++;
  } else {
    currentPlayer.value = 2;
    setTimeout(() => aiMove(), 100);
  }
  return true;
}

// ========== AI 决策 ==========
function evaluatePosition(row: number, col: number, player: number): number {
  let score = 0;
  const directions = [
    [1, 0],
    [0, 1],
    [1, 1],
    [1, -1],
  ];
  for (let [dx, dy] of directions) {
    let count = 1;
    for (let step = 1; step < 5; step++) {
      const nr = row + dx * step;
      const nc = col + dy * step;
      if (
        nr < 0 ||
        nr >= BOARD_SIZE ||
        nc < 0 ||
        nc >= BOARD_SIZE ||
        board.value[nr][nc] !== player
      )
        break;
      count++;
    }
    for (let step = 1; step < 5; step++) {
      const nr = row - dx * step;
      const nc = col - dy * step;
      if (
        nr < 0 ||
        nr >= BOARD_SIZE ||
        nc < 0 ||
        nc >= BOARD_SIZE ||
        board.value[nr][nc] !== player
      )
        break;
      count++;
    }
    if (count >= 5) score += 10000;
    else if (count === 4) score += 1000;
    else if (count === 3) score += 100;
    else if (count === 2) score += 10;
    else if (count === 1) score += 1;
  }
  return score;
}

function aiMove() {
  if (gameOver.value || currentPlayer.value !== 2) return;

  let bestScore = -1;
  let bestMove: [number, number] | null = null;

  for (let row = 0; row < BOARD_SIZE; row++) {
    for (let col = 0; col < BOARD_SIZE; col++) {
      if (board.value[row][col] !== 0) continue;
      const aiScore = evaluatePosition(row, col, 2);
      const playerScore = evaluatePosition(row, col, 1);
      const totalScore = aiScore * 0.8 + playerScore * 1.2;
      if (totalScore > bestScore) {
        bestScore = totalScore;
        bestMove = [row, col];
      }
    }
  }

  if (bestMove) {
    const [row, col] = bestMove;
    board.value[row][col] = 2;
    drawBoard();

    const win = checkWinner(row, col, 2);
    if (win) {
      winningStones.value = win;
      gameOver.value = true;
      winner.value = 2;
      gameCount.value++;
      drawBoard();
    } else if (isBoardFull()) {
      winningStones.value = null;
      gameOver.value = true;
      winner.value = 0;
      gameCount.value++;
    } else {
      currentPlayer.value = 1;
    }
  }
}

// ========== 事件处理 ==========
function handleCanvasClick(e: MouseEvent) {
  if (!canvasRef.value) return;
  const rect = canvasRef.value.getBoundingClientRect();
  const scaleX = canvasRef.value.width / rect.width;
  const scaleY = canvasRef.value.height / rect.height;
  const mouseX = (e.clientX - rect.left) * scaleX;
  const mouseY = (e.clientY - rect.top) * scaleY;

  let minDist = Infinity;
  let targetRow = -1,
    targetCol = -1;
  for (let r = 0; r < BOARD_SIZE; r++) {
    for (let c = 0; c < BOARD_SIZE; c++) {
      const x = PADDING + r * CELL_SIZE;
      const y = PADDING + c * CELL_SIZE;
      const dist = Math.hypot(mouseX - x, mouseY - y);
      if (dist < minDist && dist < CELL_SIZE / 2) {
        minDist = dist;
        targetRow = r;
        targetCol = c;
      }
    }
  }
  if (targetRow !== -1 && targetCol !== -1) placePiece(targetRow, targetCol);
}

function restartGame() {
  board.value = Array(BOARD_SIZE)
    .fill(0)
    .map(() => Array(BOARD_SIZE).fill(0));
  currentPlayer.value = 1;
  gameOver.value = false;
  winner.value = 0;
  winningStones.value = null;
  drawBoard();
}

// ========== 粒子装饰 ==========
function particleStyle(index: number) {
  const left = Math.random() * 100;
  const width = 2 + Math.random() * 8;
  const height = 10 + Math.random() * 30;
  const duration = 5 + Math.random() * 15;
  const delay = Math.random() * 10;
  const color = Math.random() > 0.5 ? "255, 140, 60" : "255, 80, 30";
  return {
    left: left + "%",
    width: width + "px",
    height: height + "px",
    animationDuration: duration + "s",
    animationDelay: delay + "s",
    backgroundColor: `rgba(${color}, 0.25)`,
  };
}

// ========== 生命周期 ==========
onMounted(() => {
  loadImages();
  drawBoard();
});

watch(
  [board, imagesLoaded],
  () => {
    if (imagesLoaded.value) drawBoard();
  },
  { deep: true }
);
</script>

<style scoped lang="scss">
.gobang-container {
  position: relative;
  min-height: 100vh;
  background: linear-gradient(135deg, #1a0c06 0%, #0a0503 50%, #1f0e07 100%);
  color: #f7e4d4;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
  font-family: "Noto Sans SC", system-ui, sans-serif;
  overflow: hidden;

  /* 背景粒子 */
  .bg-particles {
    position: absolute;
    inset: 0;
    pointer-events: none;
    z-index: 0;
    .particle {
      position: absolute;
      bottom: -20px;
      border-radius: 50%;
      filter: blur(8px);
      opacity: 0.2;
      animation: float-up 12s infinite ease-in-out;
    }
  }

  .board-bg {
    position: absolute;
    inset: 0;
    background-image: linear-gradient(
        rgba(255, 80, 30, 0.02) 1px,
        transparent 1px
      ),
      linear-gradient(90deg, rgba(255, 80, 30, 0.02) 1px, transparent 1px);
    background-size: 60px 60px;
    pointer-events: none;
    z-index: 0;
  }

  .page-glow {
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 4px;
    background: linear-gradient(
      90deg,
      transparent,
      #ff4f1e,
      #ff974f,
      #ff4f1e,
      transparent
    );
    filter: blur(6px);
    opacity: 0.5;
    animation: glow-pulse 4s infinite alternate;
    z-index: 1;
    pointer-events: none;
  }

  .game-wrapper {
    position: relative;
    z-index: 10;
    max-width: 900px;
    width: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 24px;
    padding: 20px;
    background: rgba(10, 5, 3, 0.6);
    backdrop-filter: blur(12px);
    border-radius: 48px;
    border: 1px solid rgba(255, 140, 60, 0.3);
    box-shadow: 0 30px 60px -10px rgba(0, 0, 0, 0.8);
  }

  .game-header {
    text-align: center;

    .title {
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 12px;
      font-size: 2rem;
      font-weight: 700;
      background: linear-gradient(135deg, #ffdbb0, #ffb47b, #ff6324);
      -webkit-background-clip: text;
      background-clip: text;
      color: transparent;
      margin: 0;

      .title-flame {
        display: inline-block;
        width: 30px;
        height: 30px;
        background: radial-gradient(circle, #ff974f, #ff4f1e);
        border-radius: 50%;
        filter: blur(10px);
        animation: flame-pulse 2s infinite alternate;
      }
    }

    .subtitle {
      color: #ffb47b;
      font-style: italic;
      margin: 4px 0 0;
    }
  }

  .board-wrapper {
    position: relative;
    display: flex;
    justify-content: center;

    .board-canvas {
      width: 100%;
      height: auto;
      max-width: 600px;
      border-radius: 32px;
      box-shadow: 0 20px 40px -10px rgba(0, 0, 0, 0.8),
        0 0 0 1px rgba(255, 140, 60, 0.3) inset;
      cursor: pointer;
    }

    .turn-indicator {
      position: absolute;
      top: -16px;
      right: 0;
      background: rgba(20, 10, 5, 0.8);
      backdrop-filter: blur(8px);
      padding: 6px 16px;
      border-radius: 40px;
      border: 1px solid #ff974f;
      display: flex;
      align-items: center;
      gap: 8px;
      animation: indicator-float 2s infinite alternate;

      .indicator-text {
        font-weight: 700;
        color: #ffd9b0;
      }

      .indicator-flame {
        width: 12px;
        height: 18px;
        background: linear-gradient(180deg, #ff974f, #ff4f1e);
        border-radius: 4px;
        transform: skewX(-5deg);
        box-shadow: 0 0 15px #ff712b;
      }

      &.ai-turn .indicator-text {
        color: #ffb47b;
      }

      @media (max-width: 768px) {
        top: -10px;
        right: 5px;
        padding: 4px 12px;
        font-size: 0.8rem;
      }
    }
  }

  .game-footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    max-width: 600px;

    .status-panel {
      display: flex;
      gap: 24px;
      background: rgba(0, 0, 0, 0.3);
      padding: 8px 16px;
      border-radius: 40px;
      border: 1px solid rgba(255, 140, 60, 0.2);

      .status-item {
        display: flex;
        align-items: center;
        gap: 8px;

        .status-label {
          color: #b99f8b;
          font-size: 0.9rem;
        }
        .status-value {
          color: #ffb47b;
          font-weight: 700;
          font-size: 1.1rem;

          &.winner-text {
            min-width: 60px;
            text-align: center;
          }
        }
      }
    }

    .restart-btn {
      position: relative;
      padding: 10px 24px;
      background: linear-gradient(145deg, #ff974f, #ff4f1e);
      border: none;
      border-radius: 40px;
      color: #1a0c06;
      font-weight: 700;
      font-size: 1rem;
      cursor: pointer;
      overflow: hidden;
      transition: all 0.3s;
      box-shadow: 0 10px 25px rgba(255, 80, 20, 0.4);
      display: flex;
      align-items: center;
      gap: 8px;

      .btn-flame {
        width: 16px;
        height: 16px;
        background: radial-gradient(circle, #ffd9b0, transparent);
        filter: blur(4px);
        animation: btn-flame 1.2s infinite alternate;
      }

      &:hover {
        transform: translateY(-3px);
        box-shadow: 0 15px 35px rgba(255, 100, 30, 0.6);
      }
    }
  }
}

/* 动画 */
@keyframes float-up {
  0% {
    transform: translateY(0) scale(1);
    opacity: 0;
  }
  20% {
    opacity: 0.2;
  }
  80% {
    opacity: 0.15;
  }
  100% {
    transform: translateY(-400px) scale(2.5);
    opacity: 0;
  }
}

@keyframes glow-pulse {
  0% {
    opacity: 0.3;
    filter: blur(4px);
  }
  100% {
    opacity: 0.8;
    filter: blur(10px);
  }
}

@keyframes flame-pulse {
  0% {
    opacity: 0.5;
    filter: blur(8px);
    transform: scale(1);
  }
  100% {
    opacity: 1;
    filter: blur(15px);
    transform: scale(1.2);
  }
}

@keyframes indicator-float {
  0% {
    transform: translateY(0);
  }
  100% {
    transform: translateY(-4px);
  }
}

@keyframes btn-flame {
  0% {
    opacity: 0.3;
    filter: blur(2px);
    transform: scale(1);
  }
  100% {
    opacity: 1;
    filter: blur(6px);
    transform: scale(1.2);
  }
}

/* 响应式 */
@media (max-width: 768px) {
  .game-wrapper {
    padding: 16px;
  }

  .game-header .title {
    font-size: 1.6rem;
  }

  .game-footer {
    flex-direction: column;
    gap: 16px;

    .status-panel {
      width: 100%;
      justify-content: center;
    }

    .restart-btn {
      width: 100%;
      justify-content: center;
    }
  }

  .bg-particles {
    display: block;
    opacity: 0.3;
  }
}
</style>
