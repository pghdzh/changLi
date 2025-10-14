<template>
  <header class="app-header">
    <h1 class="title">长离电子设定集</h1>
    <!-- 在线人数展示 -->
    <div class="online-count" v-if="onlineCount !== null">
      当前在线：<span class="count">{{ onlineCount }}人</span>
    </div>
    <!-- 移动端汉堡按钮 -->
    <button class="hamburger" @click="toggleMobileNav" aria-label="Toggle navigation">
      <span :class="{ open: mobileNavOpen }"></span>
      <span :class="{ open: mobileNavOpen }"></span>
      <span :class="{ open: mobileNavOpen }"></span>
    </button>

    <!-- 普通导航 & 移动端下拉导航 -->
    <nav :class="['nav-links', { 'mobile-open': mobileNavOpen }]">
      <RouterLink v-for="item in navItems" :key="item.name" :to="item.path" class="nav-item" active-class="active-link"
        @click="mobileNavOpen = false">
        {{ item.name }}
      </RouterLink>

      <a href="http://slty.site/#/redirector" target="_blank" rel="noopener" class="nav-item" active-class="active-link"
        @click="mobileNavOpen = false">
        霜落映界
      </a>
    </nav>
  </header>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, computed } from "vue";
import { io } from "socket.io-client";

const navItems = [
  { name: "离焰守棋", path: "/" },     // 首页 — 策士气质：棋局与余焰的开场
  { name: "岁轮纪事", path: "/timeLine" }, // 年谱 — 年代与抉择的时间轴
  { name: "寄语留札", path: "/message" }, // 留言板 — 他人的愿望与寄语存放处
  { name: "棋局典藏", path: "/gallery" }, // 图集 — 立绘、草稿与象征物的典藏
  { name: "匣中典籍", path: "/resources" },// 资料库 — 战术笔记、手稿与素材集合
  // { name: "焚愿祈引", path: "/wish" },    // 祈愿页 — 仪式化的祈愿与愿望寄托
  // { name: "回音泠语", path: "/voice" },   // 语音馆 — 低语、回响与短语录音
  // { name: "焚铭录", path: "/thanks" }     // 致谢/纪念 — 铭记与仪式感的结尾页
];


const mobileNavOpen = ref(false);
function toggleMobileNav() {
  mobileNavOpen.value = !mobileNavOpen.value;
}

const siteId = "changLi";

const onlineCount = ref<number | null>(null);

// 连接时带上 query.siteId
const socket: any = io("http://1.94.189.79:3000", {
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
</script>

<style scoped lang="scss">
.app-header {
  /* 长离主题色：暗沉底与余焰光 */
  --deep-bg: rgba(12, 6, 4, 0.84); // 更深的暗底（近乎夜色）
  --glass-accent: rgba(40, 18, 12, 0.48); // 半透明匣中玻璃感
  --accent: #ff6b35; // 主点光 - 余焰橙
  --accent-2: #ff9a66; // 次级暖光 - 柔和橙
  --muted-text: #f5e8dc; // 暖米色文字
  --faint: rgba(255, 150, 80, 0.06); // 微弱暖光背景点

  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  height: 64px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 40px;
  background: linear-gradient(180deg, var(--deep-bg), rgba(18, 8, 6, 0.86));
  backdrop-filter: blur(14px) saturate(1.05);
  box-shadow: 0 6px 30px rgba(18, 8, 6, 0.6), 0 0 18px rgba(255, 107, 53, 0.04) inset;
  border-bottom: 1px solid rgba(255, 150, 80, 0.06);
  animation: fadeInDown 0.6s ease-out both;

  /* 标题 - 棋士与余焰感 */
  .title {
    font-size: 26px;
    font-weight: 700;
    color: var(--muted-text);
    background: linear-gradient(90deg, var(--accent), var(--accent-2));
    background-clip: text;
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    text-shadow: 0 4px 20px rgba(60, 22, 12, 0.38);
    transition: transform 0.28s ease, text-shadow 0.28s ease;
    letter-spacing: 0.6px;
    animation: float-slow 8s ease-in-out infinite;

    &:hover {
      transform: translateY(-2px) scale(1.03);
      text-shadow: 0 6px 28px rgba(255, 107, 53, 0.12);
    }
  }

  /* 在线人数 / 状态显示 — 匣中铭牌感 */
  .online-count {
    position: relative;
    margin-left: 16px;
    padding: 6px 14px;
    font-family: "Cinzel Decorative", serif;
    font-size: 1rem;
    color: var(--muted-text);
    background: linear-gradient(135deg, rgba(255, 107, 53, 0.04), rgba(255, 150, 80, 0.03));
    border: 1px solid rgba(255, 107, 53, 0.12);
    border-radius: 24px;
    backdrop-filter: blur(6px);
    box-shadow: 0 6px 20px rgba(12, 6, 6, 0.55), 0 0 10px rgba(255, 107, 53, 0.05);
    overflow: hidden;
    cursor: default;
    transition: transform 0.22s ease, box-shadow 0.22s ease;

    &::before {
      content: "";
      position: absolute;
      bottom: -2px;
      left: -40%;
      width: 180%;
      height: 2px;
      background: linear-gradient(90deg, transparent, var(--accent-2), transparent);
      opacity: 0.95;
      filter: blur(6px);
      transform: translateZ(0);
    }

    &:hover {
      transform: translateY(-2px);
      box-shadow: 0 10px 30px rgba(12, 6, 6, 0.6), 0 0 20px rgba(255, 107, 53, 0.09);
    }

    .count {
      display: inline-block;
      margin-left: 6px;
      font-weight: 700;
      color: var(--accent-2);
      background: linear-gradient(90deg, var(--accent), var(--accent-2));
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      text-shadow: 0 0 8px rgba(255, 107, 53, 0.07);
    }
  }

  /* 导航链接 */
  .nav-links {
    display: flex;
    gap: 22px;
    align-items: center;

    .nav-item {
      position: relative;
      color: var(--muted-text);
      font-weight: 500;
      text-decoration: none;
      transition: color 0.25s ease, transform 0.18s ease;
      padding: 6px 4px;
      border-radius: 6px;

      /* 下划线：余焰流光 + 缓动展开（保留流动感） */
      &::after {
        content: "";
        position: absolute;
        left: 50%;
        bottom: -6px;
        width: 0;
        height: 3px;
        background: linear-gradient(90deg, transparent, var(--accent), var(--accent-2), transparent);
        transition: width 0.36s cubic-bezier(0.2, 0.9, 0.2, 1), left 0.36s cubic-bezier(0.2, 0.9, 0.2, 1), opacity 0.28s;
        transform: translateX(-50%);
        opacity: 0.9;
        border-radius: 3px;
        filter: blur(0.6px);
        background-size: 200% 100%;
        animation: flow 6s linear infinite;
      }

      &:hover {
        color: var(--accent-2);
        transform: translateY(-2px);
        text-shadow: 0 0 8px rgba(255, 150, 80, 0.04);
      }

      &:hover::after {
        width: 70%;
        left: 50%;
        opacity: 1;
      }
    }

    /* 激活态 */
    .active-link {
      color: var(--accent);
      font-weight: 600;

      &::after {
        width: 100%;
        opacity: 1;
      }
    }
  }

  /* 汉堡按钮（移动） */
  .hamburger {
    display: none;
    flex-direction: column;
    justify-content: space-around;
    width: 28px;
    height: 24px;
    background: none;
    border: none;
    cursor: pointer;
    padding: 0;

    span {
      display: block;
      width: 100%;
      height: 3px;
      background: rgba(250, 240, 235, 0.92);
      border-radius: 2px;
      transition: transform 0.28s ease, opacity 0.28s ease, background 0.28s;
      box-shadow: 0 2px 8px rgba(20, 8, 8, 0.36);
    }

    span.open:nth-child(1) {
      transform: translateY(10px) rotate(45deg);
    }

    span.open:nth-child(2) {
      opacity: 0;
    }

    span.open:nth-child(3) {
      transform: translateY(-10px) rotate(-45deg);
    }
  }

  /* 响应式：小屏折叠导航 */
  @media (max-width: 768px) {
    padding: 0 20px;

    .title {
      display: none;
    }

    .hamburger {
      display: flex;
    }

    .nav-links {
      position: absolute;
      top: 64px;
      left: 0;
      right: 0;
      flex-direction: column;
      background: linear-gradient(180deg, rgba(18, 8, 6, 0.98), rgba(12, 6, 4, 0.99));
      backdrop-filter: blur(12px);
      gap: 0;
      overflow: hidden;
      max-height: 0;
      transition: max-height 0.34s ease;
      border-top: 1px solid rgba(255, 150, 80, 0.04);

      &.mobile-open {
        max-height: 520px;
      }

      .nav-item {
        padding: 14px 20px;
        border-bottom: 1px solid rgba(255, 255, 255, 0.03);
      }
    }
  }
}

/* 将动画与关键帧置于顶部级别 */
@keyframes flow {
  0% {
    background-position: 0% 50%;
  }

  50% {
    background-position: 100% 50%;
  }

  100% {
    background-position: 0% 50%;
  }
}

@keyframes float-slow {
  0% {
    transform: translateY(0);
  }

  50% {
    transform: translateY(-4px);
  }

  100% {
    transform: translateY(0);
  }
}

/* 入口淡入下落（保持原名） */
@keyframes fadeInDown {
  0% {
    opacity: 0;
    transform: translateY(-8px);
  }

  100% {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
