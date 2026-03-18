<template>
  <div id="app">
    <transition name="flame-fade" v-if="showIntro">
      <div class="intro-container" @click="showIntro = false">
        <!-- 背景视频（原有逻辑保留） -->
        <video
          class="video-background"
          :src="videoSrc"
          autoplay
          muted
          loop
          playsinline
        ></video>

        <!-- 火焰色调叠加层 -->
        <div class="flame-overlay"></div>

        <!-- 动态火焰粒子（增强氛围） -->
        <div class="flame-particles">
          <span v-for="i in 24" :key="i" :style="getParticleStyle(i)"></span>
        </div>

        <!-- 飘浮的星火（细小光点） -->
        <div class="sparks">
          <span
            v-for="j in 12"
            :key="'spark' + j"
            :style="getSparkStyle(j)"
          ></span>
        </div>

        <div class="intro-content" aria-live="polite">
          <div class="intro-inner">
            <!-- 标题：长离 · 丹煌离火 -->
            <h1 class="flame-title">
              <span class="flame-ring"></span>
              长离
              <span class="title-sub">丹煌离火</span>
            </h1>

            <!-- 打字机语句（随机语录） -->
            <div class="typewriter-wrap">
              <p class="typewriter">
                {{ displayText }}
                <span class="cursor" :class="{ typing: isTyping }">|</span>
              </p>
            </div>
          </div>
        </div>

        <!-- 底部装饰火焰条 -->
        <div class="flame-strip"></div>
      </div>
    </transition>
    <div v-else>
      <navbar />
      <main class="main-content">
        <RouterView />
      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";
import { RouterView } from "vue-router";
import navbar from "./components/navbar.vue";

const showIntro = ref(true);
const videoSrc = ref("");
const isTyping = ref(true); // 用于控制光标闪烁

// 长离风格语录（更多火焰与棋局意象）
const lines = [
  "绯焰舞尽，长离不散。",
  "焰影摇红，照见人间。",
  "一缕离火，一曲清歌，一梦长离。",
  "长离之焰，燃尽彷徨，照亮前路。",
  "以我残火，映你长夜。",
  "我的过去，在火焰中沉默。",
  "离火灼灼，弈局未终。",
  "焚烬过往，方见真我。",
] as const;

const displayText = ref("");

let typingTimer: number | null = null;
let hideTimer: number | null = null;

/* 打字机参数 */
const typingSpeed = 120; // 每字符间隔 ms（稍快一点更优雅）

function pickRandomLine(): string {
  const idx = Math.floor(Math.random() * lines.length);
  return lines[idx];
}

/* 逐字打字 */
function startTyping(line: string) {
  const reduce =
    window.matchMedia &&
    window.matchMedia("(prefers-reduced-motion: reduce)").matches;
  if (reduce) {
    displayText.value = line;
    isTyping.value = false;
    return;
  }

  let i = 0;
  isTyping.value = true;
  typingTimer = window.setInterval(() => {
    i++;
    displayText.value = line.slice(0, i);
    if (i >= line.length) {
      if (typingTimer) {
        clearInterval(typingTimer);
        typingTimer = null;
        isTyping.value = false; // 打字结束，光标停止闪烁或变静态
      }
    }
  }, typingSpeed);
}

onMounted(() => {
  // 根据设备选择视频文件夹（保留原有逻辑）
  const isMobile = window.innerWidth <= 768;
  const folder = isMobile ? "/mp2" : "/mp1";
  const index = Math.floor(Math.random() * 4) + 1;
  videoSrc.value = `${folder}/1 (${index}).mp4`;

  // 自动隐藏（5秒后）
  hideTimer = window.setTimeout(() => {
    showIntro.value = false;
  }, 5800); // 稍长一点，让打字和动画更从容

  const line = pickRandomLine();
  // 延迟一点开始打字，等待页面渲染
  setTimeout(() => startTyping(line), 300);
});

onBeforeUnmount(() => {
  if (typingTimer) clearInterval(typingTimer);
  if (hideTimer) clearTimeout(hideTimer);
});

// 火焰粒子随机样式
function getParticleStyle(index: number) {
  const size = Math.floor(Math.random() * 60 + 20); // 20-80px
  const left = Math.random() * 100;
  const top = Math.random() * 100;
  const delay = Math.random() * 5;
  const duration = Math.random() * 12 + 8; // 8-20s
  return {
    width: `${size}px`,
    height: `${size}px`,
    left: `${left}%`,
    top: `${top}%`,
    animationDelay: `${delay}s`,
    animationDuration: `${duration}s`,
    opacity: Math.random() * 0.2 + 0.1,
  };
}

// 星火随机样式 (更小)
function getSparkStyle(index: number) {
  const size = Math.floor(Math.random() * 8 + 2); // 2-10px
  const left = Math.random() * 100;
  const top = Math.random() * 100;
  const delay = Math.random() * 3;
  const duration = Math.random() * 5 + 3;
  return {
    width: `${size}px`,
    height: `${size}px`,
    left: `${left}%`,
    top: `${top}%`,
    animationDelay: `${delay}s`,
    animationDuration: `${duration}s`,
    backgroundColor: `rgba(255, ${150 + Math.floor(Math.random() * 100)}, 80, ${
      Math.random() * 0.6 + 0.2
    })`,
    boxShadow: `0 0 ${Math.floor(Math.random() * 10 + 5)}px currentColor`,
  };
}
</script>

<style scoped lang="scss">
#app {
  position: relative;
  min-height: 100vh;
  animation: cursorAnimation 1s infinite step-start;
}

/* 火焰淡入淡出动画 */
.flame-fade-enter-active,
.flame-fade-leave-active {
  transition: opacity 1.2s cubic-bezier(0.2, 0.9, 0.3, 1);
}
.flame-fade-enter-from,
.flame-fade-leave-to {
  opacity: 0;
}

/* 欢迎页容器 */
.intro-container {
  position: fixed;
  inset: 0;
  background: #0a0503; /* 深色底 */
  z-index: 9999;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
}

/* 视频背景 */
.video-background {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0.45;
  z-index: 1;
  pointer-events: none;
}

/* 火焰色调叠加层 */
.flame-overlay {
  position: absolute;
  inset: 0;
  background: radial-gradient(
    circle at 30% 40%,
    rgba(255, 90, 30, 0.25) 0%,
    rgba(10, 5, 3, 0.8) 80%
  );
  mix-blend-mode: multiply;
  z-index: 2;
  pointer-events: none;
}

/* 火焰粒子（大） */
.flame-particles {
  position: absolute;
  inset: 0;
  pointer-events: none;
  z-index: 3;
  span {
    position: absolute;
    background: radial-gradient(circle, #ff8a5c, transparent);
    border-radius: 50%;
    filter: blur(20px);
    animation: flameFloat linear infinite;
    will-change: transform;
  }
}

/* 星火（小） */
.sparks {
  position: absolute;
  inset: 0;
  pointer-events: none;
  z-index: 4;
  span {
    position: absolute;
    border-radius: 50%;
    background: #ffb485;
    filter: blur(1.5px);
    animation: sparkTwinkle ease-in-out infinite alternate;
    color: #ffb485;
  }
}

/* 底部火焰装饰条 */
.flame-strip {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: linear-gradient(
    90deg,
    transparent,
    #ff6b35,
    #ff9a66,
    #ff6b35,
    transparent
  );
  filter: blur(4px);
  opacity: 0.7;
  z-index: 5;
  animation: stripFlow 6s linear infinite;
}

/* 主要内容区 */
.intro-content {
  position: relative;
  z-index: 10;
  width: 100%;
  padding: 24px;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #f5e4d4;
}

.intro-inner {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 32px;
  max-width: 900px;
  text-align: center;
  animation: zoomIn 1.2s ease-out both;
}

/* 火焰标题 */
.flame-title {
  position: relative;
  font-size: clamp(48px, 12vw, 88px);
  font-weight: 800;
  font-family: "Cinzel Decorative", "Times New Roman", serif;
  background: linear-gradient(135deg, #fcd9b8, #ffb38b, #ff8655);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 0 0 40px rgba(255, 100, 30, 0.6);
  margin: 0;
  line-height: 1.1;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 10px 20px;

  .flame-ring {
    position: relative;
    display: inline-block;
    width: 0.6em;
    height: 0.6em;
    border-radius: 50%;
    background: #ff6b35;
    box-shadow: 0 0 40px #ff9a66;
    margin-right: 5px;
    align-self: center;
    &::before {
      content: "";
      position: absolute;
      top: -10px;
      left: -10px;
      right: -10px;
      bottom: -10px;
      background: radial-gradient(
        circle,
        rgba(255, 150, 80, 0.6) 0%,
        transparent 70%
      );
      animation: flamePulse 2.4s infinite;
    }
  }

  .title-sub {
    font-size: 0.5em;
    font-weight: 400;
    background: none;
    -webkit-text-fill-color: rgba(245, 228, 220, 0.85);
    letter-spacing: 4px;
    font-style: italic;
    align-self: flex-end;
    margin-bottom: 0.1em;
  }
}

/* 打字机区域 */
.typewriter-wrap {
  display: inline-block;
  padding: 8px 24px;
  background: rgba(20, 8, 4, 0.4);
  backdrop-filter: blur(8px);
  border: 1px solid rgba(255, 140, 70, 0.3);
  border-radius: 60px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5),
    0 0 0 1px rgba(255, 150, 80, 0.2) inset;
}

.typewriter {
  margin: 0;
  font-size: clamp(24px, 6vw, 42px);
  font-weight: 500;
  font-family: "Georgia", "楷体", "KaiTi", serif;
  color: #ffe6d5;
  text-shadow: 0 2px 10px #ff6b3540;
  display: flex;
  align-items: center;
  white-space: nowrap;

  @media (max-width: 600px) {
    white-space: normal;
    word-break: break-word;
  }

  .cursor {
    display: inline-block;
    width: 0.1em;
    margin-left: 4px;
    color: #ff9a66;
    font-weight: 300;
    animation: blink 1s step-end infinite;
    &.typing {
      animation: blink 0.7s step-end infinite;
    }
  }
}

/* 动画定义 */
@keyframes flameFloat {
  0% {
    transform: translateY(0) scale(1);
    opacity: 0;
  }
  20% {
    opacity: 0.3;
  }
  80% {
    opacity: 0.2;
  }
  100% {
    transform: translateY(-150px) scale(2);
    opacity: 0;
  }
}

@keyframes sparkTwinkle {
  0% {
    opacity: 0.2;
    transform: scale(0.8);
  }
  100% {
    opacity: 1;
    transform: scale(1.5);
  }
}

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

@keyframes zoomIn {
  0% {
    transform: scale(0.9);
    opacity: 0;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

/* 移动端适配 */
@media (max-width: 768px) {
  .intro-inner {
    gap: 20px;
  }
  .flame-title {
    flex-direction: column;
    gap: 5px;
    .title-sub {
      font-size: 0.4em;
      letter-spacing: 2px;
    }
  }
  .typewriter-wrap {
    padding: 6px 16px;
  }
  .typewriter {
    font-size: clamp(18px, 5vw, 28px);
  }
}
</style>
