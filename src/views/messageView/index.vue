<template>
  <div class="megumi-message-board" aria-live="polite">

    <!-- 背景轮播（两组用于桌面/移动不同裁切） -->
    <div class="carousel carousel1" aria-hidden="true">
      <img v-for="(src, idx) in randomFive" :key="idx" :src="src" class="carousel-image"
        :class="{ active: idx === currentIndex }" />
    </div>
    <div class="carousel carousel2" aria-hidden="true">
      <img v-for="(src, idx) in randomFive2" :key="idx" :src="src" class="carousel-image"
        :class="{ active: idx === currentIndex }" />
    </div>
    <!-- 半透明顶部标题 -->
    <header class="board-header" role="banner">
      <div class="title-wrap">
        <h1>留言板</h1>
        <span class="title-count">（共{{ count }}条）</span>

        <p class="subtitle">寄一纸焚愿，留一句余烬暖言</p>
      </div>
    </header>

    <!-- 留言展示区 -->
    <section class="message-list">
      <transition-group name="msg" tag="div" class="message-list-inner">
        <div v-if="loading" class="skeleton-wrap" key="skeleton">
          <div class="skeleton" v-for="i in 1" :key="i">
            <div class="sk-avatar"></div>
            <div class="sk-lines">
              <div class="sk-line short"></div>
              <div class="sk-line"></div>
            </div>
          </div>
        </div>
        <div v-for="(msg, idx) in messages" :key="msg.id || msg._tempId || idx" class="message-card" :data-index="idx"
          tabindex="0" role="article" :aria-label="`留言来自 ${msg.name || '匿名'}，内容：${msg.content}`">
          <div class="message-meta">
            <div class="left-meta">
              <div class="name-avatar" :title="msg.name || '匿名'">
                {{ getInitial(msg.name) }}
              </div>
              <div class="meta-texts">
                <div class="message-name">{{ msg.name || "匿名" }}</div>
                <div class="message-time">{{ formatTime(msg.created_at) }}</div>
              </div>
            </div>
            <!-- HTML：把 id/class 复制到对应位置 -->
            <div class="shouan-icon" role="button" tabindex="0" aria-label="共鸣之晶">
              <!-- 替换用：焰棋徽（aria-hidden） -->
              <svg viewBox="0 0 48 48" width="36" height="36" aria-hidden="true" focusable="false">

                <!-- 核心：抽象火焰 / 棋子意象（兼具策略与焰） -->
                <g class="ember-core" transform="translate(0,0)">
                  <path
                    d="M24 14 C26 18, 30 20, 28 26 C26 32, 22 34, 24 38 C20 34, 18 30, 20 26 C22 22, 24 20, 24 14 Z" />
                </g>

                <!-- 飞烬 / 星屑 -->
                <g class="ember-sparks" aria-hidden="true">
                  <circle cx="6" cy="10" r="0.95" />
                  <circle cx="42" cy="14" r="0.8" />
                  <circle cx="38" cy="36" r="0.75" />
                  <circle cx="10" cy="34" r="0.7" />
                </g>
              </svg>

            </div>
          </div>

          <p class="message-content">{{ msg.content }}</p>
        </div>
      </transition-group>
    </section>

    <!-- 底部发送区 -->
    <section class="message-form" aria-label="写下你的留言">
      <label class="sr-only" for="mb-name">你的昵称</label>
      <input id="mb-name" v-model="name" type="text" placeholder="你的昵称" @keydown.enter.prevent />

      <label class="sr-only" for="mb-content">留言内容</label>
      <textarea id="mb-content" v-model="content" placeholder="写下你的留言..." @keydown.ctrl.enter.prevent="submitMessage"
        @input="autoGrow" ref="textareaRef" />

      <div class="form-row">
        <div class="hint">按 <kbd>Ctrl</kbd> + <kbd>Enter</kbd> 快捷发送</div>
        <button @click="submitMessage" :disabled="isSending || !content.trim()">
          <span v-if="!isSending">发送</span>
          <span v-else>发送中…</span>
        </button>
      </div>
    </section>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick, onBeforeUnmount } from "vue";
import { getMessageList, createMessage } from "@/api/modules/message";

const messages = ref<any[]>([]);
const count = ref(0);
const name = ref(localStorage.getItem("shou_name") || "");
const content = ref("");
const loading = ref(true);
const isSending = ref(false);

const textareaRef = ref<HTMLTextAreaElement | null>(null);

const fetchMessages = async () => {
  loading.value = true;
  try {
    const res = await getMessageList({ page: 1, pageSize: 9999 });
    messages.value = res.data || [];
    count.value = res.pagination.total;
    await nextTick();
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const submitMessage = async () => {
  if (!content.value.trim() || isSending.value) return;
  isSending.value = true;
  const payload = { name: name.value || "匿名", content: content.value };
  try {
    localStorage.setItem("shou_name", name.value);
    content.value = "";
    await nextTick();
    // 发送请求
    await createMessage(payload);
    // 重新同步列表（更可靠）
    await fetchMessages();
  } catch (err) {
    console.error(err);
  } finally {
    isSending.value = false;
  }
};

const formatTime = (time: string) => {
  if (!time) return "";
  const d = new Date(time);
  // 例如：2025-08-11 15:30
  const y = d.getFullYear();
  const m = String(d.getMonth() + 1).padStart(2, "0");
  const day = String(d.getDate()).padStart(2, "0");
  const hh = String(d.getHours()).padStart(2, "0");
  const mm = String(d.getMinutes()).padStart(2, "0");
  return `${y}-${m}-${day} ${hh}:${mm}`;
};

const getInitial = (n?: string) => {
  if (!n) return "匿";
  return n.trim().slice(0, 1).toUpperCase();
};

const autoGrow = (e?: Event) => {
  const ta = textareaRef.value;
  if (!ta) return;
  ta.style.height = "auto";
  const h = Math.min(ta.scrollHeight, 220);
  ta.style.height = h + "px";
};



// ========== 背景图片导入与轮播 ==========
const modules = import.meta.glob('@/assets/images1/*.{jpg,png,jpeg,webp}', { eager: true })
const allSrcs: string[] = Object.values(modules).map((mod: any) => mod.default)

const modules2 = import.meta.glob('@/assets/images2/*.{jpg,png,jpeg,webp}', { eager: true })
const allSrcs2: string[] = Object.values(modules2).map((mod: any) => mod.default)

function shuffle<T>(arr: T[]): T[] {
  const a = arr.slice()
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
      ;[a[i], a[j]] = [a[j], a[i]]
  }
  return a
}
const randomFive = ref<string[]>(shuffle(allSrcs).slice(0, 5))
const randomFive2 = ref<string[]>(shuffle(allSrcs2).slice(0, 5))

const currentIndex = ref(0)
let Imgtimer: number | undefined


onMounted(() => {
  fetchMessages();
  Imgtimer = window.setInterval(() => {
    currentIndex.value = (currentIndex.value + 1) % (Math.max(1, randomFive.value.length))
  }, 5200)
  nextTick(() => autoGrow());
});

onBeforeUnmount(() => {
  if (Imgtimer) clearInterval(Imgtimer)
})
</script>

<style scoped lang="scss">
/* ---------------- 守岸人 主题 - 替换版 .megumi-message-board ---------------- */
.megumi-message-board {
  position: relative;
  min-height: 100vh;
  padding-top: 110px;
  display: flex;
  flex-direction: column;
  /* 深海 / 黑海岸色调：深蓝 -> 海蓝 -> 冰蓝（带微弱星尘） */
  background: linear-gradient(180deg,
      #000 0%,
      #fff 100%);

  font-family: "Noto Sans SC", "Noto Sans", system-ui, -apple-system, "Segoe UI",
    Roboto, Arial;
  color: #e6f7fb;
  overflow: hidden;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;


  .carousel {
    position: absolute;
    inset: 0;
    z-index: 0;
    pointer-events: none;

    &::before {
      content: "";
      position: absolute;
      inset: 0;
      background: linear-gradient(180deg, rgba(2, 8, 14, 0.15), rgba(2, 8, 14, 0.25));
      pointer-events: none;
      z-index: 1;
    }

    .carousel-image {
      position: absolute;
      width: 100%;
      height: 100%;
      object-fit: cover;
      opacity: 0;
      transition: opacity 1s ease, transform 10s linear;
      filter: blur(0.8px) saturate(0.92);
      transform: scale(1.05);

      &.active {
        opacity: 1;
        transform: scale(1);
      }
    }
  }

  /* 可在小屏使用第二组竖图，避免裁切失衡 */
  .carousel2 {
    display: none;
  }

  /* ---------- 顶部标题（毛玻璃 + 晶体装饰） ---------- */
  .board-header {
    position: absolute;
    top: 72px;
    left: 50%;
    transform: translateX(-50%);
    width: calc(100% - 32px);
    max-width: 960px;
    /* 长离主题：暗底 + 微暖余焰光 */

    padding: 14px 18px;
    border-radius: 14px;
    box-shadow: 0 14px 44px rgba(6, 4, 4, 0.6), inset 0 1px 0 rgba(255, 160, 110, 0.02);
    backdrop-filter: blur(2px) saturate(110%);
    z-index: 6;
    border: 1px solid rgba(255, 120, 70, 0.06);

    .title-wrap {
      display: flex;
      align-items: center;
      gap: 12px;

      h1 {
        margin: 0;
        font-size: 18px;
        color: #ffb890;
        /* 暖金色标题 */
        letter-spacing: 0.4px;
        font-weight: 900;
        text-shadow: 0 2px 14px rgba(80, 30, 18, 0.28);
      }

      .title-count {
        color: #ffdebe;
        /* 暖米色偏亮 */
        font-size: 12px;
        font-weight: 700;
        margin-left: 6px;
      }

      .subtitle {
        margin: 0;
        margin-left: auto;
        color: rgba(245, 232, 220, 0.92);
        /* 暖米色文本 */
        font-size: 13px;
        font-weight: 600;
        /* 微光下沉效果，契合仪式感 */
        text-shadow: 0 1px 8px rgba(255, 120, 70, 0.04);
      }
    }

    /* 角落的余焰光斑装饰（视觉） */
    &::before {
      content: "";
      position: absolute;
      right: 12px;
      top: 8px;
      width: 84px;
      height: 48px;
      background:
        radial-gradient(34px 16px at 20% 40%, rgba(255, 180, 120, 0.12), transparent 30%),
        radial-gradient(40px 20px at 60% 60%, rgba(255, 120, 70, 0.08), transparent 30%);
      filter: blur(6px);
      pointer-events: none;
      z-index: -1;
    }


  }

  /* ---------- 留言列表 ---------- */
  .message-list {
    z-index: 2;
    position: relative;
    flex: 1;
    overflow-y: auto;
    padding: 28px 20px 340px;
    /* 给底部输入更多空间 */
    margin-top: 18px;

    .message-list-inner {
      max-width: 960px;
      max-height: 80vh;
      margin: 0 auto;
      display: flex;
      flex-direction: column;
      gap: 16px;
      position: relative;
      z-index: 2;
      overflow-y: auto;
    }

    /* skeleton 样式替换成守岸人色系 */
    .skeleton-wrap {
      display: flex;
      flex-direction: column;
      gap: 12px;

      .skeleton {
        display: flex;
        gap: 12px;
        align-items: center;
        padding: 12px;
        background: linear-gradient(180deg,
            rgba(8, 36, 48, 0.36),
            rgba(6, 34, 44, 0.28));
        border-radius: 12px;
        box-shadow: 0 6px 18px rgba(2, 18, 26, 0.5);
        border: 1px solid rgba(80, 160, 200, 0.03);
      }

      .sk-avatar {
        width: 44px;
        height: 44px;
        border-radius: 10px;
        background: linear-gradient(90deg, #d16711, #f0831e);
      }

      .sk-lines {
        flex: 1;

        .sk-line {
          height: 10px;
          border-radius: 6px;
          background: linear-gradient(90deg,
              rgba(255, 255, 255, 0.06),
              rgba(150, 220, 250, 0.03));
          margin-bottom: 8px;
        }

        .sk-line.short {
          width: 40%;
        }
      }
    }
  }

  /* ---------- 单条消息卡片（ ---------- */
  /* 长离风格：留言卡（暗焰匣） */
  .message-card {
    /* 暗底 + 匣中余焰轻映 */
    background: linear-gradient(180deg, rgba(10, 6, 4, 0.16), rgba(8, 4, 3, 0.22));
    border-radius: 14px;
    padding: 14px 16px;
    margin: 6px auto;
    width: calc(100% - 48px);
    max-width: 960px;

    border: 1px solid rgba(255, 120, 70, 0.04);
    transition: transform 0.32s cubic-bezier(.2, .9, .3, 1), box-shadow 0.32s, border-color 0.32s;
    transform-origin: center;
    position: relative;
    z-index: 3;
    overflow: visible;

    /* 右上角保留（可做更淡的余焰光斑） */
    &::before {
      content: "";
      position: absolute;
      right: 10px;
      top: 8px;
      width: 44px;
      height: 44px;
      background:
        radial-gradient(circle at 30% 30%, rgba(255, 180, 120, 0.08), transparent 30%),
        radial-gradient(circle at 70% 70%, rgba(255, 120, 70, 0.04), transparent 30%);
      filter: blur(3px);
      pointer-events: none;
      border-radius: 8px;
      z-index: 2;
    }

    &:hover {
      transform: translateY(-8px) scale(1.01);
      box-shadow: 0 34px 100px rgba(6, 4, 4, 0.8);
      border-color: rgba(255, 140, 90, 0.12);
      background: linear-gradient(180deg, rgba(10, 6, 4, 0.56), rgba(8, 4, 3, 0.72));
    }

    .message-meta {
      display: flex;
      justify-content: space-between;
      align-items: flex-start;
      gap: 12px;
      margin-bottom: 8px;

      .left-meta {
        display: flex;
        gap: 12px;
        align-items: center;

        .name-avatar {
          width: 48px;
          height: 48px;
          border-radius: 10px;
          display: inline-flex;
          align-items: center;
          justify-content: center;
          font-weight: 900;
          color: #2b1208;
          background: linear-gradient(180deg, #ffd9b8 0%, #ffb890 60%);
          border: 2px solid rgba(255, 160, 110, 0.06);
          box-shadow: inset 0 -6px 18px rgba(20, 8, 6, 0.06);
          font-size: 16px;
          flex-shrink: 0;
        }

        .meta-texts {
          .message-name {
            font-size: 15px;
            color: #ffb890;
            /* 暖金色 */
            font-weight: 800;
            line-height: 1;
            text-shadow: 0 3px 8px rgba(40, 16, 12, 0.35);
          }

          .message-time {
            font-size: 12px;
            color: rgba(245, 232, 220, 0.78);
            margin-top: 2px;
          }
        }
      }

      /* 右侧：焰棋徽（替代晶格） */
      .shouan-icon {
        display: inline-grid;
        place-items: center;
        width: 52px;
        height: 52px;
        border-radius: 12px;
        cursor: pointer;
        user-select: none;
        position: relative;
        z-index: 4;

        background: linear-gradient(180deg, rgba(10, 6, 4, 0.92), rgba(12, 8, 6, 0.94));
        border: 1px solid rgba(255, 120, 70, 0.06);
        box-shadow: 0 8px 30px rgba(6, 4, 4, 0.64), inset 0 1px 0 rgba(255, 160, 110, 0.02);
        transition: transform 260ms cubic-bezier(.2, .9, .3, 1), box-shadow 260ms, background 260ms;
        -webkit-tap-highlight-color: transparent;
        will-change: transform, box-shadow, opacity;

        svg {
          width: 36px;
          height: 36px;
          display: block;
          overflow: visible;
        }



        .ember-core path {
          fill: #ff9a66;
          opacity: 0.14;
          transition: fill 260ms, opacity 260ms, transform 260ms, filter 260ms;
          filter: drop-shadow(0 8px 20px rgba(255, 120, 70, 0.06));
        }

        .ember-sparks circle {
          fill: rgba(255, 200, 150, 0.95);
          opacity: 0;
          transition: opacity 240ms, transform 360ms;
        }

        &:hover,
        &:focus {
          transform: translateY(-6px) scale(1.04);
          box-shadow: 0 28px 86px rgba(8, 4, 4, 0.72), inset 0 1px 0 rgba(255, 160, 110, 0.02);
          background: linear-gradient(180deg, rgba(14, 8, 6, 0.96), rgba(16, 10, 8, 0.98));



          .ember-core path {
            opacity: 1;
            transform: scale(1.03);
            fill: #ff9a66;
            filter: drop-shadow(0 18px 46px rgba(255, 120, 70, 0.12));
          }

          .ember-sparks circle {
            opacity: 1;

            &:nth-child(1) {
              transform: translate(-4px, -6px) scale(1.4);
            }

            &:nth-child(2) {
              transform: translate(6px, -4px) scale(1.2);
            }

            &:nth-child(3) {
              transform: translate(4px, 6px) scale(1.1);
            }

            &:nth-child(4) {
              transform: translate(-6px, 4px) scale(1.15);
            }
          }
        }

        &:active {
          transform: translateY(-2px) scale(0.99);
        }

        &:focus {
          outline: none;
          box-shadow: 0 28px 86px rgba(8, 4, 4, 0.72), 0 0 0 6px rgba(255, 120, 70, 0.06);
        }

        &.active {
          .ember-core path {
            opacity: 1;
            animation: emberCorePulse 2000ms ease-in-out infinite;
          }


        }

        /* 动画：浮动 / 框体摆动 / 核心呼吸 / 火星上浮 */
        animation: emberFloat 8s ease-in-out infinite;



        .ember-core path {
          animation: emberCoreBreathe 4.6s ease-in-out infinite;
          transform-origin: 50% 50%;
        }

        .ember-sparks circle {
          animation: emberSparkFloat 1800ms ease-in-out infinite;
        }

        @media (max-width: 480px) {
          width: 44px;
          height: 44px;

          svg {
            width: 30px;
            height: 30px;
          }
        }
      }

      /* ============ keyframes ============ */
      @keyframes emberFloat {
        0% {
          transform: translateY(0) scale(1);
        }

        40% {
          transform: translateY(-6px) scale(1.015);
        }

        70% {
          transform: translateY(-3px) scale(1.008);
        }

        100% {
          transform: translateY(0) scale(1);
        }
      }

      @keyframes emberFrameSway {
        0% {
          transform: rotate(-0.6deg) translateY(0);
        }

        50% {
          transform: rotate(0.6deg) translateY(-2px);
        }

        100% {
          transform: rotate(-0.6deg) translateY(0);
        }
      }

      @keyframes emberCoreBreathe {
        0% {
          transform: scale(1);
          opacity: 0.9;
          filter: drop-shadow(0 6px 18px rgba(255, 120, 70, 0.06));
        }

        50% {
          transform: scale(1.04);
          opacity: 1;
          filter: drop-shadow(0 18px 46px rgba(255, 140, 80, 0.12));
        }

        100% {
          transform: scale(1);
          opacity: 0.9;
          filter: drop-shadow(0 6px 18px rgba(255, 120, 70, 0.06));
        }
      }

      @keyframes emberSparkFloat {
        0% {
          opacity: 0;
          transform: translateY(0) scale(0.8);
          filter: blur(0);
        }

        35% {
          opacity: 1;
          transform: translateY(-6px) scale(1.15);
          filter: blur(.2px);
        }

        70% {
          opacity: 0.6;
          transform: translateY(-10px) scale(1.25);
          filter: blur(.8px);
        }

        100% {
          opacity: 0;
          transform: translateY(-14px) scale(1.35);
          filter: blur(1.6px);
        }
      }

      @keyframes emberCorePulse {
        0% {
          transform: scale(1);
          filter: drop-shadow(0 6px 18px rgba(255, 120, 70, 0.06));
        }

        50% {
          transform: scale(1.06);
          filter: drop-shadow(0 18px 46px rgba(255, 140, 80, 0.12));
        }

        100% {
          transform: scale(1);
          filter: drop-shadow(0 6px 18px rgba(255, 120, 70, 0.06));
        }
      }
    }

    /* .message-meta end */

    .message-content {
      font-size: 15px;
      color: rgba(245, 232, 220, 0.96);
      line-height: 1.7;
      white-space: pre-wrap;
      word-break: break-word;
      margin: 0;
      padding-bottom: 2px;
      letter-spacing: 0.2px;
    }
  }


  /* ---------- 固定底部输入区（守岸人风格） ---------- */
  .message-form {
    position: fixed;
    left: 50%;
    transform: translateX(-50%);
    bottom: 18px;
    width: calc(100% - 32px);
    max-width: 960px;

    /* 暗匣背景与微弱余焰内光（写死颜色） */
    background: linear-gradient(180deg, rgba(10, 6, 4, 0.92), rgba(8, 4, 3, 0.96));
    padding: 14px;
    border-radius: 14px;
    display: flex;
    flex-direction: column;
    gap: 10px;
    box-shadow: 0 22px 64px rgba(6, 4, 4, 0.78), inset 0 1px 0 rgba(255, 160, 110, 0.02);
    z-index: 6;
    border: 1px solid rgba(255, 120, 70, 0.04);
    will-change: transform, opacity;

    /* 表单字段（输入/文本区） */
    input,
    textarea {
      padding: 12px 14px;
      border-radius: 12px;
      border: 1px solid rgba(255, 120, 70, 0.06);
      font-size: 14px;
      outline: none;
      transition: box-shadow 0.18s, border-color 0.18s, background 0.18s, transform 0.12s;
      background: linear-gradient(180deg, rgba(14, 8, 6, 0.60), rgba(10, 6, 4, 0.64));
      box-shadow: inset 0 -4px 10px rgba(0, 0, 0, 0.48);
      color: #f5e8dc;
      /* 暖米文字 */
      resize: none;
      -webkit-appearance: none;
      -moz-appearance: none;
    }

    input::placeholder,
    textarea::placeholder {
      color: rgba(245, 230, 216, 0.46);
    }

    input:focus,
    textarea:focus {
      border-color: rgba(255, 140, 90, 0.22);
      box-shadow: 0 10px 30px rgba(255, 120, 70, 0.06), inset 0 -6px 12px rgba(0, 0, 0, 0.4);
      background: linear-gradient(180deg, rgba(18, 10, 8, 0.72), rgba(12, 8, 6, 0.70));
      transform: translateY(-1px);
    }

    textarea {
      min-height: 64px;
      max-height: 220px;
      line-height: 1.6;
    }

    /* 行布局：提示 + 提交 */
    .form-row {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 12px;

      .hint {
        color: rgba(245, 232, 220, 0.86);
        font-size: 13px;
        display: flex;
        align-items: center;
        gap: 8px;

        kbd {
          background: rgba(18, 10, 8, 0.78);
          border-radius: 6px;
          padding: 3px 7px;
          border: 1px solid rgba(255, 120, 70, 0.06);
          font-size: 12px;
          box-shadow: inset 0 -2px 6px rgba(0, 0, 0, 0.36);
          color: #ffd9b8;
          font-weight: 700;
          letter-spacing: 0.6px;
        }
      }

      /* 提交按钮：余焰渐变 + 微光呼吸 */
      button {
        padding: 10px 18px;
        background: linear-gradient(180deg, #ff8a4a 0%, #ff6b35 60%);
        color: #2b1208;
        border: none;
        border-radius: 12px;
        cursor: pointer;
        font-weight: 800;
        box-shadow: 0 14px 44px rgba(255, 120, 70, 0.12), inset 0 1px 0 rgba(255, 255, 255, 0.02);
        transition: transform 0.14s ease, box-shadow 0.14s ease, opacity 0.14s ease;
        will-change: transform, box-shadow;
        display: inline-flex;
        align-items: center;
        gap: 8px;

        /* 小光晕装饰（伪元素） */
        &::after {
          content: "";
          display: inline-block;
          width: 6px;
          height: 6px;
          border-radius: 50%;
          background: radial-gradient(circle at 40% 40%, #ffd9b8 0%, #ff9a66 40%, transparent 60%);
          box-shadow: 0 6px 18px rgba(255, 140, 80, 0.18);
        }

        &:hover {
          transform: translateY(-3px) scale(1.02);
          box-shadow: 0 22px 58px rgba(255, 120, 70, 0.14);
        }

        &:active {
          transform: translateY(-1px) scale(0.995);
        }
      }

      /* 禁用态 */
      button:disabled {
        opacity: 0.52;
        cursor: not-allowed;
        transform: none;
        box-shadow: none;
        background: linear-gradient(180deg, rgba(14, 8, 6, 0.6), rgba(12, 8, 6, 0.6));
        color: rgba(220, 200, 180, 0.6);
      }
    }



    /* 轻微脉冲（可作为调用高亮或新留言提示时短暂加上 .pulse） */
    &.pulse {
      animation: formPulse 1200ms ease-in-out 1;
    }
  }

  /* 辅助关键帧（按钮/表单轻微呼吸与脉冲） */
  @keyframes formPulse {
    0% {
      transform: translateX(-50%) scale(1);
      box-shadow: 0 22px 64px rgba(6, 4, 4, 0.78);
    }

    40% {
      transform: translateX(-50%) scale(1.01);
      box-shadow: 0 28px 84px rgba(255, 120, 70, 0.12);
    }

    100% {
      transform: translateX(-50%) scale(1);
      box-shadow: 0 22px 64px rgba(6, 4, 4, 0.78);
    }
  }

  /* ---------- 响应式：移动端收敛（守岸人样式） ---------- */
  @media (max-width: 980px) {
    padding-top: 90px;


    .carousel1 {
      display: none;
    }

    .carousel2 {
      display: block;
    }



    .board-header {
      left: 12px;
      transform: none;
      width: calc(100% - 24px);
    }

    .message-list {
      padding: 18px 12px 260px;

      .message-list-inner {
        gap: 12px;
      }
    }

    .message-card {
      width: calc(100% - 28px);
      border-radius: 12px;
      padding: 12px;

      .name-avatar {
        width: 44px;
        height: 44px;
      }
    }

    .message-form {
      left: 12px;
      transform: none;
      width: calc(100% - 24px);
      bottom: 12px;
      padding: 12px;
    }
  }

  /* sr-only（无障碍隐藏） */
  .sr-only {
    position: absolute !important;
    width: 1px !important;
    height: 1px !important;
    padding: 0 !important;
    margin: -1px !important;
    overflow: hidden !important;
    clip: rect(0, 0, 0, 0) !important;
    white-space: nowrap !important;
    border: 0 !important;
  }
}
</style>
