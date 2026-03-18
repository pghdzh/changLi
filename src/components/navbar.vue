<template>
  <header class="app-header">
    <!-- 火焰粒子背景 (装饰动效) -->
    <div class="flame-particles">
      <span v-for="i in 16" :key="i" :style="getParticleStyle(i)"></span>
    </div>

    <!-- 新增：飘浮的细小星火 (提升精致感) -->
    <div class="sparks">
      <span v-for="j in 8" :key="'spark' + j" :style="getSparkStyle(j)"></span>
    </div>

    <!-- 标题区：加入动态拖尾光效 & 星火环绕 -->
    <h1 class="title">
      <span class="flame-ring"></span>
      长离·丹煌离火
   
    </h1>

    <!-- 控制区 (在线人数 + BGM) - 始终显示在顶部 -->
    <div class="header-controls">
      <div class="online-count" v-if="onlineCount !== null">
        <span class="flame-dot"></span>
        <span class="count">{{ onlineCount }}</span>
        <span class="label">在线弈客</span>
      </div>
      <button
        class="bgm-btn"
        :class="{ playing: isBgmPlaying }"
        @click="toggleBgm"
        aria-label="BGM播放控制"
      >
        <span class="btn-flame" v-if="isBgmPlaying"></span>
        <svg v-if="!isBgmPlaying" viewBox="0 0 24 24" width="20" height="20">
          <path d="M8 5v14l11-7z" fill="currentColor" />
        </svg>
        <svg v-else viewBox="0 0 24 24" width="20" height="20">
          <path d="M6 19h4V5H6v14zm8-14v14h4V5h-4z" fill="currentColor" />
        </svg>
        <span class="bgm-tooltip">山霁浮古今</span>
      </button>
    </div>

    <!-- 移动端汉堡按钮 -->
    <button
      class="hamburger"
      @click="toggleMobileNav"
      :class="{ active: mobileNavOpen }"
      aria-label="Toggle navigation"
    >
      <span></span>
      <span></span>
      <span></span>
    </button>

    <!-- 导航区域 -->
    <nav :class="['nav-links', { 'mobile-open': mobileNavOpen }]">
      <!-- 主导航 (前6个) -->
      <RouterLink
        v-for="item in mainNavItems"
        :key="item.name"
        :to="item.path"
        class="nav-item"
        active-class="active-link"
        @click="mobileNavOpen = false"
      >
        <span class="item-flame"></span>
        <span class="item-glow"></span>
        <span class="item-ripple"></span>
        <!-- 新增涟漪效果 -->
        {{ item.name }}
      </RouterLink>

      <!-- 更多下拉菜单 (桌面端) -->
      <div class="dropdown" v-if="dropdownItems.length > 0">
        <button class="dropdown-trigger nav-item">
          更多棋局
          <svg
            class="dropdown-arrow"
            viewBox="0 0 24 24"
            width="16"
            height="16"
          >
            <path d="M7 10l5 5 5-5z" fill="currentColor" />
          </svg>
        </button>
        <div class="dropdown-menu">
          <RouterLink
            v-for="item in dropdownItems"
            :key="item.name"
            :to="item.path"
            class="dropdown-item"
            active-class="active-link"
            @click="mobileNavOpen = false"
          >
            <span class="item-flame"></span>
            {{ item.name }}
          </RouterLink>
          <a
            href="https://slty.site/#/redirector"
            target="_blank"
            rel="noopener"
            class="dropdown-item"
            @click="mobileNavOpen = false"
          >
            <span class="item-flame"></span>
            霜落映界
          </a>
        </div>
      </div>

      <!-- 移动端显示全部下拉菜单项 (平铺) -->
      <template v-if="mobileNavOpen">
        <RouterLink
          v-for="item in dropdownItems"
          :key="'mobile-' + item.name"
          :to="item.path"
          class="nav-item mobile-sub"
          active-class="active-link"
          @click="mobileNavOpen = false"
        >
          {{ item.name }}
        </RouterLink>
        <a
          href="https://slty.site/#/redirector"
          target="_blank"
          rel="noopener"
          class="nav-item mobile-sub"
          @click="mobileNavOpen = false"
        >
          霜落映界
        </a>
      </template>
    </nav>

    <!-- 音频元素 -->
    <audio
      ref="bgmAudio"
      loop
      preload="metadata"
      :src="bgmUrl"
      @ended="isBgmPlaying = false"
      @pause="isBgmPlaying = false"
      @play="isBgmPlaying = true"
    ></audio>
  </header>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, computed } from "vue";
import { io } from "socket.io-client";

// 导航项定义
const allNavItems = [
  { name: "离焰守棋", path: "/" },
  { name: "岁轮纪事", path: "/timeLine" },
  { name: "寄语留札", path: "/message" },
  { name: "棋局典藏", path: "/gallery" },
  { name: "五子棋", path: "/game" },
  { name: "镜弈心语", path: "/talk" },
  { name: "流焰低语", path: "/voice" },
  { name: "曲库", path: "/music" },
  { name: "匣中典籍", path: "/resources" },
  { name: "wiki", path: "/wiki" },
];

const mainNavItems = computed(() => allNavItems.slice(0, 6));
const dropdownItems = computed(() => allNavItems.slice(6));

// 移动端导航开关
const mobileNavOpen = ref(false);
function toggleMobileNav() {
  mobileNavOpen.value = !mobileNavOpen.value;
}

// 在线人数
const siteId = "changLi";
const onlineCount = ref<number | null>(null);
const socket: any = io(import.meta.env.VITE_API_BASE_URL, {
  query: { siteId },
});

onMounted(() => {
  socket.on("onlineCount", (count: number) => {
    onlineCount.value = count;
  });
});

onBeforeUnmount(() => {
  socket.disconnect();
});

// BGM 控制
const bgmAudio = ref<HTMLAudioElement | null>(null);
const isBgmPlaying = ref(false);
const bgmUrl = import.meta.env.VITE_API_BASE_URL + `/music/山霁浮古今.mp3`;

const toggleBgm = async () => {
  if (!bgmAudio.value) return;
  try {
    if (isBgmPlaying.value) {
      bgmAudio.value.pause();
    } else {
      await bgmAudio.value.play();
    }
  } catch (error) {
    console.warn("BGM播放失败:", error);
    isBgmPlaying.value = false;
  }
};

onBeforeUnmount(() => {
  if (bgmAudio.value) {
    bgmAudio.value.pause();
    bgmAudio.value.currentTime = 0;
  }
});

// 火焰粒子随机样式
function getParticleStyle(index: number) {
  const size = Math.floor(Math.random() * 40 + 15); // 15-55px
  const left = Math.random() * 100;
  const top = Math.random() * 100;
  const delay = Math.random() * 5;
  const duration = Math.random() * 10 + 8;
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

// 星火随机样式 (更小、更亮)
function getSparkStyle(index: number) {
  const size = Math.floor(Math.random() * 6 + 2); // 2-8px
  const left = Math.random() * 100;
  const top = Math.random() * 100;
  const delay = Math.random() * 3;
  const duration = Math.random() * 4 + 3;
  return {
    width: `${size}px`,
    height: `${size}px`,
    left: `${left}%`,
    top: `${top}%`,
    animationDelay: `${delay}s`,
    animationDuration: `${duration}s`,
    backgroundColor: `rgba(255, ${150 + Math.floor(Math.random() * 100)}, 80, ${
      Math.random() * 0.8 + 0.2
    })`,
    boxShadow: `0 0 ${Math.floor(Math.random() * 10 + 5)}px currentColor`,
  };
}
</script>

<style scoped lang="scss">
/* 长离子火主题 — 余焰、熔金、古卷 */
.app-header {
  --flame-core: #ff6b35;
  --flame-outer: #ff9a66;
  --flame-ember: #b33e1c;
  --deep-bg: rgba(8, 4, 3, 0.96);
  --glass-bg: rgba(30, 16, 12, 0.7);
  --muted-paper: #f5e4d4;
  --ink-gold: #d9b48f;

  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  height: 72px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 48px;
  background: linear-gradient(180deg, var(--deep-bg) 0%, #130a07 100%);
  backdrop-filter: blur(18px) saturate(200%);
  border-bottom: 1px solid rgba(255, 140, 70, 0.2);
  box-shadow: 0 10px 40px -5px rgba(0, 0, 0, 0.8),
    0 0 0 1px rgba(255, 100, 50, 0.1) inset;
  transition: height 0.2s;

  /* 火焰粒子背景 (较大) */
  .flame-particles {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    pointer-events: none;
    overflow: hidden;
    span {
      position: absolute;
      background: radial-gradient(circle, #ff8a5c, transparent);
      border-radius: 50%;
      filter: blur(16px);
      animation: flameFloat linear infinite;
      will-change: transform;
    }
  }

  /* 新增：细小星火 (飘浮闪烁) */
  .sparks {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    pointer-events: none;
    span {
      position: absolute;
      border-radius: 50%;
      background: #ffb485;
      filter: blur(1px);
      animation: sparkTwinkle ease-in-out infinite alternate;
      color: #ffb485;
    }
  }

  /* 底部流动光带 (新增装饰) */
  &::after {
    content: "";
    position: absolute;
    bottom: -2px;
    left: 0;
    right: 0;
    height: 2px;
    background: linear-gradient(
      90deg,
      transparent,
      var(--flame-core),
      var(--flame-outer),
      transparent
    );
    filter: blur(4px);
    opacity: 0.6;
    animation: flowGlow 4s linear infinite;
  }

  /* 标题 */
  .title {
    position: relative;
    font-size: 30px;
    font-weight: 800;
    letter-spacing: 2px;
    background: linear-gradient(135deg, #fcd9b8, #ffb38b, #ff8655);
    background-clip: text;
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    text-shadow: 0 0 30px rgba(255, 90, 30, 0.5);
    display: flex;
    align-items: baseline;
    gap: 10px;
    z-index: 2;

    .flame-ring {
      position: relative;
      width: 14px;
      height: 14px;
      border-radius: 50%;
      background: var(--flame-core);
      box-shadow: 0 0 25px var(--flame-core);
      &::before {
        content: "";
        position: absolute;
        width: 34px;
        height: 34px;
        top: -10px;
        left: -10px;
        background: radial-gradient(
          circle,
          rgba(255, 150, 80, 0.5) 0%,
          transparent 70%
        );
        animation: flamePulse 2.2s infinite;
      }
    }
    .title-sub {
      font-size: 16px;
      font-weight: 400;
      -webkit-text-fill-color: rgba(245, 228, 220, 0.7);
      letter-spacing: 1px;
      font-style: italic;
    }
    &:hover {
      filter: brightness(1.15);
    }
  }

  /* 控制区 (始终显示) */
  .header-controls {
    display: flex;
    align-items: center;
    gap: 16px;
    z-index: 3;
  }

  /* 在线人数 */
  .online-count {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 6px 18px 6px 12px;
    background: rgba(30, 14, 8, 0.7);
    border: 1px solid rgba(255, 120, 60, 0.3);
    border-radius: 40px;
    backdrop-filter: blur(4px);
    font-size: 15px;
    color: var(--muted-paper);
    box-shadow: 0 4px 14px rgba(0, 0, 0, 0.5);
    transition: border-color 0.2s, box-shadow 0.2s, transform 0.2s;

    .flame-dot {
      width: 10px;
      height: 10px;
      background: var(--flame-core);
      border-radius: 50%;
      box-shadow: 0 0 16px var(--flame-core);
      animation: flicker 1.8s infinite;
    }
    .count {
      font-weight: 700;
      color: var(--flame-outer);
      font-size: 18px;
      text-shadow: 0 0 10px #ff6b35;
    }
    .label {
      opacity: 0.8;
    }
    &:hover {
      border-color: #ffb485;
      box-shadow: 0 4px 20px #ff6b3540;
      transform: translateY(-2px);
    }
  }

  /* BGM按钮 */
  .bgm-btn {
    position: relative;
    width: 46px;
    height: 46px;
    border-radius: 50%;
    background: radial-gradient(circle at 30% 30%, #4a2c1e, #26160e);
    border: 1px solid rgba(255, 140, 70, 0.6);
    display: flex;
    align-items: center;
    justify-content: center;
    color: #f5bba4;
    cursor: pointer;
    transition: all 0.3s cubic-bezier(0.2, 0.9, 0.3, 1.2);
    box-shadow: 0 6px 16px rgba(0, 0, 0, 0.7),
      0 0 0 1px rgba(255, 150, 80, 0.2) inset;

    svg {
      transition: transform 0.2s;
    }
    .btn-flame {
      position: absolute;
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background: radial-gradient(circle, #ff8a5c 0%, transparent 70%);
      filter: blur(8px);
      opacity: 0.8;
      animation: flameRotate 5s linear infinite;
    }
    .bgm-tooltip {
      position: absolute;
      bottom: -36px;
      left: 50%;
      transform: translateX(-50%);
      background: #1f0e08;
      color: var(--flame-outer);
      padding: 4px 12px;
      border-radius: 30px;
      font-size: 12px;
      white-space: nowrap;
      border: 1px solid #ff6b3540;
      opacity: 0;
      pointer-events: none;
      transition: opacity 0.2s;
    }
    &:hover {
      transform: scale(1.15);
      border-color: var(--flame-core);
      box-shadow: 0 0 30px #ff6b35;
      .bgm-tooltip {
        opacity: 1;
      }
    }
    &.playing {
      color: #ffd8c0;
      background: radial-gradient(circle at 70% 30%, #8f4a28, #3b1f12);
      border-color: #ffb485;
      box-shadow: 0 0 30px #ff9f70;
    }
  }

  /* 导航区域 */
  .nav-links {
    display: flex;
    align-items: center;
    gap: 6px;
    z-index: 2;

    .nav-item {
      position: relative;
      padding: 8px 20px;
      color: #edd6c5;
      font-weight: 500;
      text-decoration: none;
      border-radius: 40px;
      transition: color 0.2s, background 0.2s, transform 0.15s;
      letter-spacing: 0.5px;
      overflow: hidden;
      z-index: 1;
      white-space: nowrap;

      .item-flame,
      .item-glow,
      .item-ripple {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        border-radius: 40px;
        pointer-events: none;
      }
      .item-flame {
        background: radial-gradient(
          ellipse at center,
          rgba(255, 120, 40, 0.25) 0%,
          transparent 70%
        );
        opacity: 0;
        transition: opacity 0.2s;
        z-index: -1;
      }
      .item-glow {
        background: radial-gradient(
          circle at var(--x, 50%) var(--y, 50%),
          rgba(255, 150, 80, 0.35) 0%,
          transparent 60%
        );
        opacity: 0;
        mix-blend-mode: screen;
      }
      .item-ripple {
        background: transparent;
        box-shadow: 0 0 0 0 rgba(255, 150, 80, 0.3);
        transition: box-shadow 0.3s;
        border-radius: 40px;
      }

      &:hover {
        color: #ffc4a2;
        background: rgba(255, 100, 40, 0.08);
        transform: translateY(-2px);
        .item-flame {
          opacity: 1;
        }
        .item-glow {
          opacity: 0.4;
        }
        .item-ripple {
          animation: ripple 0.8s ease-out;
        }
      }

      &::after {
        content: "";
        position: absolute;
        bottom: 4px;
        left: 50%;
        width: 0;
        height: 3px;
        background: linear-gradient(
          90deg,
          var(--flame-core),
          var(--flame-outer)
        );
        transform: translateX(-50%);
        transition: width 0.25s ease;
        border-radius: 3px;
        filter: blur(2px);
      }

      &.active-link {
        color: #ffbe8c;
        font-weight: 600;
        &::after {
          width: 60%;
        }
      }
    }

    /* 更多下拉 (桌面端) */
    .dropdown {
      position: relative;
      .dropdown-trigger {
        display: flex;
        align-items: center;
        gap: 4px;
        background: none;
        border: none;
        cursor: pointer;
        font-size: 16px;
        .dropdown-arrow {
          transition: transform 0.2s;
        }
      }
      &:hover {
        .dropdown-menu {
          opacity: 1;
          visibility: visible;
          transform: translateY(0);
        }
        .dropdown-arrow {
          transform: rotate(180deg);
        }
      }
      .dropdown-menu {
        position: absolute;
        top: 100%;
        right: 0;
        margin-top: 12px;
        min-width: 180px;
        background: rgba(18, 8, 5, 0.98);
        backdrop-filter: blur(16px);
        border: 1px solid rgba(255, 110, 50, 0.4);
        border-radius: 24px;
        padding: 8px 0;
        box-shadow: 0 20px 40px rgba(0, 0, 0, 0.9), 0 0 0 1px #ff6b3520 inset,
          0 0 30px #ff6b3520;
        opacity: 0;
        visibility: hidden;
        transform: translateY(-10px);
        transition: opacity 0.3s, transform 0.3s, visibility 0.3s;
        z-index: 1001;

        &::before {
          content: "";
          position: absolute;
          top: -6px;
          right: 30px;
          width: 16px;
          height: 16px;
          background: rgba(18, 8, 5, 0.98);
          border-left: 1px solid #ff6b3540;
          border-top: 1px solid #ff6b3540;
          transform: rotate(45deg);
          backdrop-filter: blur(16px);
        }

        .dropdown-item {
          display: flex;
          align-items: center;
          gap: 8px;
          padding: 10px 24px;
          color: #e8cfbc;
          text-decoration: none;
          font-size: 15px;
          transition: background 0.2s, color 0.2s, padding-left 0.2s;
          white-space: nowrap;
          position: relative;
          .item-flame {
            width: 4px;
            height: 4px;
            background: #ff9f70;
            border-radius: 50%;
            box-shadow: 0 0 8px #ff6b35;
            opacity: 0;
            transition: opacity 0.2s;
          }
          &:hover {
            background: linear-gradient(
              90deg,
              rgba(255, 120, 40, 0.2),
              transparent
            );
            color: #ffbc8c;
            padding-left: 30px;
            .item-flame {
              opacity: 1;
            }
          }
          &.active-link {
            background: rgba(255, 100, 30, 0.2);
            color: #ffb485;
          }
        }
      }
    }
  }

  /* 汉堡按钮 */
  .hamburger {
    display: none;
    flex-direction: column;
    justify-content: space-around;
    width: 36px;
    height: 36px;
    background: rgba(255, 120, 60, 0.15);
    border: 1px solid #ff8a5c60;
    border-radius: 12px;
    cursor: pointer;
    padding: 8px;
    z-index: 5;

    span {
      display: block;
      width: 100%;
      height: 2px;
      background: #f5b392;
      border-radius: 2px;
      transition: transform 0.25s, opacity 0.2s;
    }
    &.active {
      span:nth-child(1) {
        transform: translateY(8px) rotate(45deg);
      }
      span:nth-child(2) {
        opacity: 0;
      }
      span:nth-child(3) {
        transform: translateY(-8px) rotate(-45deg);
      }
    }
  }

  /* 响应式：移动端 (宽度 < 960px) */
  @media (max-width: 960px) {
    padding: 0 24px;
    .title .title-sub {
      display: none;
    }
  }

  @media (max-width: 860px) {
    .hamburger {
      display: flex;
    }

    /* 调整标题大小，给控制区留空间 */
    .title {
      font-size: 24px;
      margin-right: 8px;
    }

    .nav-links {
      position: absolute;
      top: 72px;
      left: 0;
      right: 0;
      flex-direction: column;
      background: linear-gradient(180deg, #1c0e09 0%, #0f0704 100%);
      backdrop-filter: blur(24px);
      gap: 0;
      max-height: 0;
      overflow: hidden;
      transition: max-height 0.4s ease;
      border-bottom: 2px solid #ffa05530;
      box-shadow: 0 30px 40px -10px black;
      align-items: stretch;

      &.mobile-open {
        max-height: 700px;
      }

      .nav-item,
      .mobile-sub {
        padding: 16px 28px;
        border-bottom: 1px solid #2f1c1420;
        width: 100%;
        text-align: left;
        border-radius: 0;
        &.active-link {
          &::after {
            width: 0;
          }
        }
      }

      .dropdown {
        display: none;
      } // 桌面下拉隐藏
    }

    /* 确保控制区在移动端可见且不重叠 */
    .header-controls {
      .online-count .label {
        display: none;
      } // 移动端隐藏“在线弈客”文字，只留数字和点
      .bgm-btn {
        width: 40px;
        height: 40px;
      }
    }
  }

  @media (max-width: 600px) {
    .title {
      font-size: 20px;
    }
    .header-controls {
      gap: 8px;
    }
    .online-count {
      padding: 4px 10px;
      .count {
        font-size: 16px;
      }
    }
  }
}

/* 动画库 */
@keyframes flamePulse {
  0% {
    transform: scale(1);
    opacity: 0.6;
  }
  50% {
    transform: scale(1.8);
    opacity: 0.2;
  }
  100% {
    transform: scale(1);
    opacity: 0.6;
  }
}
@keyframes flicker {
  0% {
    opacity: 1;
    box-shadow: 0 0 12px #ff6b35;
  }
  50% {
    opacity: 0.5;
    box-shadow: 0 0 25px #ff9f70;
  }
  100% {
    opacity: 1;
    box-shadow: 0 0 12px #ff6b35;
  }
}
@keyframes flameRotate {
  0% {
    transform: rotate(0deg);
    filter: blur(10px);
  }
  100% {
    transform: rotate(360deg);
    filter: blur(8px);
  }
}
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
    transform: translateY(-120px) scale(1.8);
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
    transform: scale(1.2);
  }
}
@keyframes flowGlow {
  0% {
    opacity: 0.3;
    filter: blur(4px) hue-rotate(0deg);
  }
  50% {
    opacity: 0.8;
    filter: blur(6px) hue-rotate(10deg);
  }
  100% {
    opacity: 0.3;
    filter: blur(4px) hue-rotate(0deg);
  }
}
@keyframes ripple {
  0% {
    box-shadow: 0 0 0 0 rgba(255, 150, 80, 0.5);
  }
  100% {
    box-shadow: 0 0 0 20px rgba(255, 150, 80, 0);
  }
}
</style>
