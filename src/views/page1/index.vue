<template>
  <main class="home">
    <!-- 火焰粒子画布 (替代原有的玫瑰花瓣) -->
    <canvas ref="canvasEl" class="flame-canvas" aria-hidden="true"></canvas>

    <!-- 背景轮播（两组用于桌面/移动不同裁切） -->
    <div class="carousel carousel1" aria-hidden="true">
      <img
        v-for="(src, idx) in randomFive"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>
    <div class="carousel carousel2" aria-hidden="true">
      <img
        v-for="(src, idx) in randomFive2"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>

    <!-- 主内容区 -->
    <section class="center" role="main">
      <h1 class="title">
        <span class="flame-ring"></span>
        焚身以火 · 长离
      </h1>

      <div class="subtitle" aria-live="polite">
        <span class="typed">{{ typed }}</span
        ><span class="cursor" aria-hidden="true">|</span>
      </div>

      <!-- 棋盘装饰（浮影） -->
      <div class="chess-overlay"></div>
    </section>

    <!-- 页脚：增加火焰流光条 -->
    <footer class="flame-footer" role="contentinfo" aria-label="页面页脚">
      <div class="footer-inner">
        <div class="slogan">落子为誓 · 余焰为证</div>
        <div class="meta">
          © <span>{{ year }}</span> 长离电子设定集 · 制作：霜落天亦
        </div>
      </div>
      <div class="footer-flame"></div>
    </footer>
  </main>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";

const year = new Date().getFullYear();

// ========== 火焰粒子系统 ==========
const canvasEl = ref<HTMLCanvasElement | null>(null);
let ctx: CanvasRenderingContext2D;
let animationId = 0;
let lastTime = 0;
let elapsed = 0;

interface FlameParticle {
  x: number;
  y: number;
  size: number;
  speedY: number;
  speedX: number;
  opacity: number;
  color: string;
  life: number;
  maxLife: number;
}

const particles: FlameParticle[] = [];
const PARTICLE_COUNT = 80; // 粒子数量

// 初始化火焰粒子
function initParticles(width: number, height: number) {
  particles.length = 0;
  for (let i = 0; i < PARTICLE_COUNT; i++) {
    particles.push(createParticle(width, height));
  }
}

function createParticle(width: number, height: number): FlameParticle {
  const maxLife = 80 + Math.random() * 50;
  return {
    x: Math.random() * width,
    y: Math.random() * height,
    size: 4 + Math.random() * 12,
    speedY: -0.5 - Math.random() * 1.5, // 向上飘
    speedX: (Math.random() - 0.5) * 0.4,
    opacity: 0.3 + Math.random() * 0.5,
    color: `hsl(${20 + Math.random() * 20}, 90%, 60%)`, // 橙红范围
    life: Math.random() * maxLife,
    maxLife,
  };
}

// 更新粒子
function updateParticles(width: number, height: number) {
  for (let i = particles.length - 1; i >= 0; i--) {
    const p = particles[i];
    p.x += p.speedX;
    p.y += p.speedY;
    p.life += 1;

    // 超出边界或寿命结束则重置
    if (p.y < -20 || p.x < -20 || p.x > width + 20 || p.life >= p.maxLife) {
      // 重置粒子
      particles[i] = createParticle(width, height);
    }
  }
}

// 绘制粒子
function drawParticles() {
  if (!ctx || !canvasEl.value) return;
  const canvas = canvasEl.value;
  const w = canvas.width / (window.devicePixelRatio || 1);
  const h = canvas.height / (window.devicePixelRatio || 1);

  ctx.clearRect(0, 0, w, h);

  // 叠加一层暗色半透明，制造拖尾效果
  ctx.fillStyle = "rgba(10, 5, 3, 0.15)";
  ctx.fillRect(0, 0, w, h);

  particles.forEach((p) => {
    const lifeRatio = 1 - p.life / p.maxLife; // 寿命越短越透明
    const opacity = p.opacity * lifeRatio * 0.8;

    // 绘制火焰粒子（圆形 + 光晕）
    ctx.save();
    ctx.globalCompositeOperation = "lighter"; // 叠加混合，增强火光

    // 主焰
    ctx.beginPath();
    ctx.arc(p.x, p.y, p.size * 0.8, 0, Math.PI * 2);
    const gradient = ctx.createRadialGradient(p.x, p.y, 0, p.x, p.y, p.size);
    gradient.addColorStop(0, `rgba(255, 150, 80, ${opacity})`);
    gradient.addColorStop(0.6, `rgba(200, 70, 20, ${opacity * 0.5})`);
    gradient.addColorStop(1, `rgba(100, 30, 5, 0)`);
    ctx.fillStyle = gradient;
    ctx.fill();

    // 内核
    ctx.beginPath();
    ctx.arc(
      p.x - p.size * 0.1,
      p.y - p.size * 0.1,
      p.size * 0.3,
      0,
      Math.PI * 2
    );
    ctx.fillStyle = `rgba(255, 220, 160, ${opacity})`;
    ctx.fill();

    ctx.restore();
  });
}

// 动画循环
function tickCanvas(now: number) {
  if (!canvasEl.value) return;
  const canvas = canvasEl.value;
  const w = canvas.clientWidth;
  const h = canvas.clientHeight;

  if (!lastTime) lastTime = now;
  const dt = (now - lastTime) / 1000;
  lastTime = now;
  elapsed += dt;

  updateParticles(w, h);
  drawParticles();

  animationId = requestAnimationFrame(tickCanvas);
}

// 响应式调整canvas
let resizeTimeout: number;
function resizeCanvas() {
  window.clearTimeout(resizeTimeout);
  resizeTimeout = window.setTimeout(() => {
    const canvas = canvasEl.value!;
    const parent = canvas.parentElement!;
    const dpr = window.devicePixelRatio || 1;
    const w = parent.clientWidth;
    const h = Math.max(parent.clientHeight, 500);

    canvas.style.width = w + "px";
    canvas.style.height = h + "px";
    canvas.width = w * dpr;
    canvas.height = h * dpr;

    ctx.setTransform(1, 0, 0, 1, 0, 0);
    ctx.scale(dpr, dpr);

    initParticles(w, h);
    lastTime = 0;
    cancelAnimationFrame(animationId);
    animationId = requestAnimationFrame(tickCanvas);
  }, 100);
}

// ========== 打字机文案（长离风格短句） ==========
const lines = [
  "生灭如常，丹煌离火永不熄。", 
  "以我残火，照你长夜，落子为誓。", 
  "弈棋布势，万物运转皆在变易之中。", 
  "谁才是樊笼之鸟呢？", 
  "海晏河清，是我此生夙愿。",
  "师父的剑像你，一切自有因果。", 
  "入局之刻，当谋定后动。", 
  "离火灼身，终有一日燃尽，但棋局未终。", 
  "逢危须弃，但心中所念，绝不弃子。",
  "丹煌行令，为迷途者点亮方向。", 
];
const typed = ref("");
let lineIndex = 0;
let charIndex = 0;
let deleting = false;
let timer: number | null = null;

const TYPING = 100;
const DELETING = 40;
const PAUSE = 2000;

function tick() {
  const cur = lines[lineIndex];
  if (!deleting) {
    typed.value = cur.slice(0, charIndex + 1);
    charIndex++;
    if (charIndex >= cur.length) {
      timer = window.setTimeout(() => {
        deleting = true;
        tick();
      }, PAUSE);
      return;
    }
    timer = window.setTimeout(tick, TYPING);
  } else {
    typed.value = cur.slice(0, charIndex - 1);
    charIndex--;
    if (charIndex <= 0) {
      deleting = false;
      lineIndex = (lineIndex + 1) % lines.length;
      timer = window.setTimeout(tick, 300);
      return;
    }
    timer = window.setTimeout(tick, DELETING);
  }
}

// ========== 背景图片轮播 ==========
const modules = import.meta.glob("@/assets/images1/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const allSrcs: string[] = Object.values(modules).map((mod: any) => mod.default);

const modules2 = import.meta.glob("@/assets/images2/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const allSrcs2: string[] = Object.values(modules2).map(
  (mod: any) => mod.default
);

function shuffle<T>(arr: T[]): T[] {
  const a = arr.slice();
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}

const randomFive = ref<string[]>(shuffle(allSrcs).slice(0, 5));
const randomFive2 = ref<string[]>(shuffle(allSrcs2).slice(0, 5));
const currentIndex = ref(0);
let imgTimer: number | undefined;

onMounted(() => {
  // 启动打字机
  timer = window.setTimeout(tick, 500);

  // 轮播切换
  imgTimer = window.setInterval(() => {
    currentIndex.value = (currentIndex.value + 1) % randomFive.value.length;
  }, 5200);

  // 初始化canvas
  const canvas = canvasEl.value!;
  ctx = canvas.getContext("2d")!;
  resizeCanvas();
  window.addEventListener("resize", resizeCanvas);
});

onBeforeUnmount(() => {
  if (imgTimer) clearInterval(imgTimer);
  if (timer) clearTimeout(timer);
  cancelAnimationFrame(animationId);
  window.removeEventListener("resize", resizeCanvas);
});
</script>

<style scoped lang="scss">
/* 长离主题色调：深红底、橙焰、暖金 */
.home {
  --flame-core: #ff6b35;
  --flame-outer: #ff9a66;
  --flame-ember: #b33e1c;
  --deep-bg: #1a0c08;
  --deep-bg2: #2f140e;
  --muted-paper: #f5e4d4;

  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  background: linear-gradient(180deg, var(--deep-bg) 0%, var(--deep-bg2) 100%);
  font-family: "Georgia", "Times New Roman", "楷体", serif;
  position: relative;
  overflow: hidden;

  .flame-canvas {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 2;
    pointer-events: none;
  }

  /* 背景轮播 */
  .carousel {
    position: absolute;
    inset: 0;
    z-index: 1;
    pointer-events: none;

    &::before {
      content: "";
      position: absolute;
      inset: 0;
      background: radial-gradient(
        circle at 30% 40%,
        rgba(200, 70, 20, 0.25) 0%,
        rgba(10, 5, 3, 0.8) 90%
      );
      z-index: 1;
      mix-blend-mode: multiply;
    }

    .carousel-image {
      position: absolute;
      width: 100%;
      height: 100%;
      object-fit: cover;
      opacity: 0;
      transition: opacity 1.5s ease, transform 10s linear;
      filter: blur(1px) saturate(1.1);
      transform: scale(1.05);

      &.active {
        opacity: 1;
        transform: scale(1);
      }
    }
  }

  .carousel2 {
    display: none;
  }

  /* 主内容区 */
  .center {
    position: relative;
    flex: 1 0 auto;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
    padding: 34px 20px;
    gap: 20px;
    z-index: 10;

    /* 棋盘虚影装饰 */
    .chess-overlay {
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background-image: linear-gradient(
          rgba(255, 100, 50, 0.03) 1px,
          transparent 1px
        ),
        linear-gradient(90deg, rgba(255, 100, 50, 0.03) 1px, transparent 1px);
      background-size: 60px 60px;
      pointer-events: none;
      z-index: -1;
      opacity: 0.3;
    }

    .title {
      position: relative;
      font-size: clamp(36px, 10vw, 72px);
      font-weight: 800;
      font-family: "Cinzel Decorative", serif;
      background: linear-gradient(135deg, #fcd9b8, #ffb38b, #ff8655);
      background-clip: text;
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      text-shadow: 0 0 40px rgba(255, 90, 30, 0.5);
      margin: 0;
      line-height: 1.2;
      display: flex;
      flex-wrap: wrap;
      align-items: center;
      justify-content: center;
      gap: 10px 20px;

      .flame-ring {
        position: relative;
        width: 0.5em;
        height: 0.5em;
        border-radius: 50%;
        background: var(--flame-core);
        box-shadow: 0 0 30px var(--flame-core);
        &::before {
          content: "";
          position: absolute;
          top: -8px;
          left: -8px;
          right: -8px;
          bottom: -8px;
          background: radial-gradient(
            circle,
            rgba(255, 150, 80, 0.5) 0%,
            transparent 70%
          );
          animation: flamePulse 2.4s infinite;
        }
      }

      .title-sub {
        font-size: 0.4em;
        font-weight: 400;
        background: none;
        -webkit-text-fill-color: rgba(245, 228, 220, 0.85);
        letter-spacing: 4px;
        font-style: italic;
        align-self: flex-end;
        margin-bottom: 0.2em;
      }
    }

    .subtitle {
      font-size: clamp(18px, 4vw, 28px);
      min-height: 2.2em;
      color: var(--muted-paper);
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 8px;
      padding: 8px 24px;
      background: rgba(30, 12, 8, 0.4);
      backdrop-filter: blur(8px);
      border: 1px solid rgba(255, 120, 60, 0.3);
      border-radius: 60px;
      box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5),
        0 0 0 1px rgba(255, 150, 80, 0.2) inset;

      .typed {
        font-weight: 500;
        letter-spacing: 1px;
        text-shadow: 0 0 10px #ff9f70;
      }

      .cursor {
        display: inline-block;
        width: 0.1em;
        margin-left: 2px;
        color: #ff9a66;
        font-weight: 300;
        animation: blink 1s step-end infinite;
      }
    }
  }

  /* 页脚 */
  .flame-footer {
    position: relative;
    background: linear-gradient(
      180deg,
      rgba(20, 8, 4, 0.9),
      rgba(10, 4, 2, 0.98)
    );
    border-top: 2px solid rgba(255, 100, 50, 0.2);
    color: var(--muted-paper);
    font-size: 14px;
    padding: 16px 0;
    text-align: center;
    z-index: 10;

    .footer-inner {
      position: relative;
      z-index: 2;
    }

    .slogan {
      background: linear-gradient(90deg, #ff6b35, #ff9a66);
      background-clip: text;
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      font-size: 16px;
      font-weight: 600;
      letter-spacing: 2px;
      margin-bottom: 6px;
      text-shadow: 0 0 15px #ff6b3540;
    }

    .meta {
      color: rgba(245, 232, 220, 0.65);
      font-size: 12px;
    }

    .footer-flame {
      position: absolute;
      bottom: 0;
      left: 0;
      right: 0;
      height: 3px;
      background: linear-gradient(
        90deg,
        transparent,
        #ff6b35,
        #ff9a66,
        #ff6b35,
        transparent
      );
      filter: blur(4px);
      opacity: 0.8;
      animation: stripFlow 6s linear infinite;
    }
  }
}

/* 动画 */
@keyframes flamePulse {
  0% {
    transform: scale(1);
    opacity: 0.6;
  }
  50% {
    transform: scale(1.5);
    opacity: 0.2;
  }
  100% {
    transform: scale(1);
    opacity: 0.6;
  }
}

@keyframes blink {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0;
  }
}

@keyframes stripFlow {
  0% {
    background-position: 0% 0%;
    opacity: 0.5;
  }
  50% {
    background-position: 100% 0%;
    opacity: 1;
  }
  100% {
    background-position: 200% 0%;
    opacity: 0.5;
  }
}

/* 移动端适配 */
@media (max-width: 720px) {
  .home {
    .carousel1 {
      display: none;
    }
    .carousel2 {
      display: block;
    }

    .center {
      .title {
        flex-direction: column;
        gap: 5px;
        .title-sub {
          font-size: 0.3em;
        }
      }
      .subtitle {
        font-size: 16px;
        padding: 6px 16px;
      }
    }

    .flame-footer {
      .slogan {
        font-size: 14px;
      }
    }
  }
}
</style>
