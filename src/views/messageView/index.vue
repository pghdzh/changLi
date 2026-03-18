<template>
  <div class="changli-message-board" aria-live="polite">
    <!-- ========== 背景层：保留原有轮播，新增装饰 ========== -->
    <!-- 原有轮播背景（保留） -->
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



    <!-- ========== 顶部标题（长离风格增强） ========== -->
    <header class="board-header" role="banner">

      <div class="title-wrap">
        
        <h1>焚愿留言板</h1>
        <span class="title-count">（共{{ count }}条）</span>
   
      </div>

      <p class="subtitle">
        <span class="quote-left">“</span>
        寄一纸焚愿，留一句余烬暖言
        <span class="quote-right">”</span>
      </p>

      <!-- 新增：墨迹装饰 [citation:8] -->
      <div class="ink-splash"></div>
    </header>

    <!-- ========== 留言展示区（卡片增强） ========== -->
    <section class="message-list">
      <transition-group name="msg" tag="div" class="message-list-inner">
        <div v-if="loading" class="skeleton-wrap" key="skeleton">
          <div class="skeleton" v-for="i in 3" :key="i">
            <div class="sk-avatar"></div>
            <div class="sk-lines">
              <div class="sk-line short"></div>
              <div class="sk-line"></div>
            </div>
          </div>
        </div>
        <div
          v-for="(msg, idx) in messages"
          :key="msg.id || msg._tempId || idx"
          class="message-card"
          :data-index="idx"
          tabindex="0"
          role="article"
          :aria-label="`留言来自 ${msg.name || '匿名'}，内容：${msg.content}`"
          :class="'flame-level-' + ((idx % 4) + 1)"
        >
          <!-- 新增：卡片火焰背景（随离火层数变化） -->
          <div class="card-flame-bg" :class="'level-' + ((idx % 4) + 1)"></div>

          <div class="message-meta">
            <div class="left-meta">
              <div class="name-avatar" :title="msg.name || '匿名'">
                {{ getInitial(msg.name) }}
                <!-- 新增：头像周围小火苗 -->
                <span class="avatar-spark"></span>
              </div>
              <div class="meta-texts">
                <div class="message-name">{{ msg.name || "匿名" }}</div>
                <div class="message-time">{{ formatTime(msg.created_at) }}</div>
              </div>
            </div>

            <!-- 心眼图标（增强版）[citation:1] -->
            <div
              class="shinzo-icon"
              role="button"
              tabindex="0"
              aria-label="心眼·离火"
            >
              <svg
                viewBox="0 0 48 48"
                width="40"
                height="40"
                aria-hidden="true"
                focusable="false"
              >
                <!-- 四层火焰核心（根据索引变化颜色） -->
                <g class="flame-core" :class="'flame-level-1'">
                  <path
                    d="M24 8 C20 14, 16 20, 20 26 C24 32, 30 34, 28 38 C22 36, 18 30, 22 24 C24 20, 26 16, 24 8 Z"
                    fill="currentColor"
                  />
                  <path
                    d="M28 12 C30 16, 34 18, 32 24 C30 30, 26 32, 28 36 C24 34, 22 28, 26 22 C28 18, 29 16, 28 12 Z"
                    fill="currentColor"
                    opacity="0.7"
                  />
                </g>
             
                <!-- 飞烬火星（动态） -->
                <g class="sparks">
                  <circle cx="8" cy="12" r="1.5" fill="#ffb47b" />
                  <circle cx="40" cy="16" r="1.2" fill="#ffb47b" />
                  <circle cx="36" cy="36" r="1.8" fill="#ffb47b" />
                  <circle cx="12" cy="38" r="1.1" fill="#ffb47b" />
                </g>
              </svg>
              <!-- 新增：悬停时出现的墨迹光圈 [citation:8] -->
              <div class="icon-ink-ring"></div>
            </div>
          </div>

          <p class="message-content">{{ msg.content }}</p>

          <!-- 新增：留言底部火焰装饰条（随离火层数） -->
          <div class="card-footer-flame">
            <span class="flame-segment" v-for="seg in 4" :key="seg"></span>
          </div>
        </div>
      </transition-group>
    </section>

    <!-- ========== 底部发送区（增强版） ========== -->
    <section class="message-form" aria-label="写下你的留言">
      <!-- 新增：四层动态火焰边框 [citation:2] -->
      <div class="form-flame-border">
        <div class="flame-layer layer1"></div>
        <div class="flame-layer layer2"></div>
        <div class="flame-layer layer3"></div>
        <div class="flame-layer layer4"></div>
      </div>

      <!-- 新增：焚香烟雾效果 -->
      <div class="incense-smoke"></div>

      <label class="sr-only" for="mb-name">你的昵称</label>
      <input
        id="mb-name"
        v-model="name"
        type="text"
        placeholder="你的昵称"
        @keydown.enter.prevent
      />

      <label class="sr-only" for="mb-content">留言内容</label>
      <textarea
        id="mb-content"
        v-model="content"
        placeholder="写下你的留言..."
        @keydown.ctrl.enter.prevent="submitMessage"
        @input="autoGrow"
        ref="textareaRef"
      />

      <div class="form-row">
        <div class="hint">
          <span class="hint-flame"></span>
          <span class="hint-text"
            >按 <kbd>Ctrl</kbd> + <kbd>Enter</kbd> 焚音传讯</span
          >
        </div>
        <button @click="submitMessage" :disabled="isSending || !content.trim()">
          <span v-if="!isSending">· 焚音寄愿 ·</span>
          <span v-else>· 余烬燃信中 ···</span>
          <!-- 新增：按钮火星 -->
          <span class="button-spark"></span>
        </button>
      </div>

      <!-- 新增：底部棋局纹样 [citation:1][citation:8] -->
      <div class="form-chess-pattern"></div>
    </section>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick, onBeforeUnmount } from "vue";
import { getMessageList, createMessage } from "@/api/modules/message";

// 原有数据逻辑保持不变
const messages = ref<any[]>([]);
const count = ref(0);
const name = ref(localStorage.getItem("shou_name") || "");
const content = ref("");
const loading = ref(true);
const isSending = ref(false);
const textareaRef = ref<HTMLTextAreaElement | null>(null);


// 原有轮播相关变量
const currentIndex = ref(0);
let Imgtimer: number | undefined;

// 轮播图片导入（保持不变）
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



// 原有方法保持不变
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
    await createMessage(payload);
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

onMounted(() => {
  fetchMessages();
  Imgtimer = window.setInterval(() => {
    currentIndex.value =
      (currentIndex.value + 1) % Math.max(1, randomFive.value.length);
  }, 5200);
  nextTick(() => autoGrow());
});

onBeforeUnmount(() => {
  if (Imgtimer) clearInterval(Imgtimer);
});
</script>

<style scoped lang="scss">
.changli-message-board {
  /* 长离主题色系 */
  --deep-bg: #0a0503;
  --accent: #ff4f1e;
  --accent-2: #ff974f;
  --accent-3: #ffcb9a;
  --text-light: #f7e4d4;
  --glass-bg: rgba(10, 5, 3, 0.7);
  --flame-glow: rgba(255, 80, 20, 0.3);

  position: relative;
  min-height: 100vh;
  padding-top: 140px;
  display: flex;
  flex-direction: column;
  background: linear-gradient(135deg, #1a0c06 0%, #0a0503 50%, #1f0e07 100%);
  font-family: "Noto Sans SC", system-ui, -apple-system, sans-serif;
  color: var(--text-light);
  overflow-x: hidden;

  /* ========== 背景层（轮播 + 新增装饰） ========== */
  .carousel {
    position: fixed;
    inset: 0;
    z-index: 0;
    pointer-events: none;

    &::before {
      content: "";
      position: absolute;
      inset: 0;
      background: linear-gradient(
        180deg,
        rgba(10, 5, 3, 0.4),
        rgba(10, 5, 3, 0.3)
      );
      pointer-events: none;
      z-index: 1;
    }

    .carousel-image {
      position: absolute;
      width: 100%;
      height: 100%;
      object-fit: cover;
      opacity: 0;
      transition: opacity 1.5s ease;
  

      &.active {
        opacity: 1;
      }
    }
  }

  .carousel2 {
    display: none;
  }


  /* ========== 顶部标题（增强版） ========== */
  .board-header {
    position: absolute;
    top: 60px;
    left: 50%;
    transform: translateX(-50%);
    width: calc(100% - 32px);
    max-width: 1000px;
    background: rgba(10, 5, 3, 0.7);
    backdrop-filter: blur(1px);
    padding: 18px 30px;
    border-radius: 60px;
    border: 1px solid rgba(255, 140, 60, 0.3);
    box-shadow: 0 25px 50px -10px rgba(0, 0, 0, 0.8),
      0 0 40px rgba(255, 80, 20, 0.2);
    z-index: 10;
    overflow: hidden;

    &::before {
      content: "";
      position: absolute;
      top: 0;
      left: -100%;
      width: 100%;
      height: 2px;
      background: linear-gradient(
        90deg,
        transparent,
        var(--accent),
        var(--accent-2),
        transparent
      );
      animation: border-flow 6s infinite;
    }



    .title-wrap {
      display: flex;
      align-items: center;
      gap: 12px;
      margin-left: 140px;

 

      h1 {
        margin: 0;
        font-size: 2rem;
        font-weight: 700;
        background: linear-gradient(135deg, #ffdbb0, #ffb47b, #ff6324);
        -webkit-background-clip: text;
        background-clip: text;
        color: transparent;
        letter-spacing: 2px;
      }

      .title-count {
        font-size: 1rem;
        color: #ffb47b;
      }

  
    }

    .subtitle {
      margin: 8px 0 0 140px;
      color: #ffd9b0;
      font-size: 1rem;
      font-style: italic;

      .quote-left,
      .quote-right {
        color: #ff974f;
        font-size: 1.2rem;
        opacity: 0.6;
      }
    }

    /* 墨迹装饰 [citation:8] */
    .ink-splash {
      position: absolute;
      bottom: -10px;
      right: 20px;
      width: 100px;
      height: 40px;
      background: radial-gradient(
        ellipse,
        rgba(255, 100, 30, 0.1),
        transparent
      );
      filter: blur(8px);
      border-radius: 50%;
    }
  }

  /* ========== 留言列表 ========== */
  .message-list {
    z-index: 5;
    position: relative;
    flex: 1;
    overflow-y: auto;
    padding: 28px 20px 300px;

    .message-list-inner {
      max-width: 1000px;
      margin: 0 auto;
      display: flex;
      flex-direction: column;
      gap: 20px;
    }

    /* skeleton 加载 */
    .skeleton-wrap {
      display: flex;
      flex-direction: column;
      gap: 20px;

      .skeleton {
        display: flex;
        gap: 14px;
        padding: 20px;
        background: rgba(20, 10, 5, 0.5);
        border-radius: 24px;
        border: 1px solid rgba(255, 140, 60, 0.1);
      }

      .sk-avatar {
        width: 52px;
        height: 52px;
        border-radius: 16px;
        background: linear-gradient(90deg, #ff974f40, #ff4f1e40);
        animation: skeleton-pulse 1.5s infinite alternate;
      }

      .sk-lines {
        flex: 1;

        .sk-line {
          height: 14px;
          border-radius: 7px;
          background: rgba(255, 140, 60, 0.2);
          margin-bottom: 10px;
          animation: skeleton-pulse 1.5s infinite alternate;

          &.short {
            width: 40%;
          }
        }
      }
    }
  }

  /* 留言卡片（增强版） */
  .message-card {
    position: relative;
    background: rgba(15, 8, 4, 0.6);
    backdrop-filter: blur(2px);
    border-radius: 28px;
    padding: 22px 26px;
    border: 1px solid rgba(255, 140, 60, 0.2);
    box-shadow: 0 20px 40px -10px rgba(0, 0, 0, 0.6);
    transition: all 0.4s cubic-bezier(0.2, 0.9, 0.3, 1);
    overflow: hidden;

    /* 四层离火卡片背景 [citation:2] */
    .card-flame-bg {
      position: absolute;
      top: 0;
      right: 0;
      width: 200px;
      height: 100%;
      background: radial-gradient(
        circle at 70% 50%,
        transparent 30%,
        rgba(255, 140, 60, 0.1)
      );
      filter: blur(30px);
      opacity: 0;
      transition: opacity 0.4s;
      pointer-events: none;

      &.level-1 {
        background: radial-gradient(circle at 70% 50%, #ff974f20, transparent);
      }
      &.level-2 {
        background: radial-gradient(circle at 70% 50%, #ffb47b30, transparent);
      }
      &.level-3 {
        background: radial-gradient(circle at 70% 50%, #ffc99e40, transparent);
      }
      &.level-4 {
        background: radial-gradient(circle at 70% 50%, #ffe0ba50, transparent);
      }
    }

    &:hover {
      transform: translateY(-6px);
      border-color: rgba(255, 140, 60, 0.5);
      box-shadow: 0 30px 60px -10px rgba(255, 100, 30, 0.3);

      .card-flame-bg {
        opacity: 0.6;
      }
    }

    .message-meta {
      display: flex;
      justify-content: space-between;
      align-items: flex-start;
      gap: 14px;
      margin-bottom: 14px;

      .left-meta {
        display: flex;
        gap: 16px;
        align-items: center;

        .name-avatar {
          position: relative;
          width: 56px;
          height: 56px;
          border-radius: 18px;
          display: flex;
          align-items: center;
          justify-content: center;
          font-weight: 700;
          font-size: 1.3rem;
          color: #2b1208;
          background: linear-gradient(145deg, #ffd9b0, #ff974f);
          border: 2px solid rgba(255, 160, 110, 0.4);
          box-shadow: 0 10px 25px rgba(255, 100, 30, 0.3);

          .avatar-spark {
            position: absolute;
            top: -4px;
            right: -4px;
            width: 12px;
            height: 12px;
            background: radial-gradient(circle, #ffb47b, #ff4f1e);
            border-radius: 50%;
            filter: blur(3px);
            animation: spark-pulse 2s infinite alternate;
          }
        }

        .meta-texts {
          .message-name {
            font-size: 1.2rem;
            font-weight: 600;
            color: #ffd9b0;
            margin-bottom: 4px;
          }
          .message-time {
            font-size: 0.85rem;
            color: #b99f8b;
          }
        }
      }

      /* 心眼图标（增强版） */
      .shinzo-icon {
        position: relative;
        width: 56px;
        height: 56px;
        display: flex;
        align-items: center;
        justify-content: center;
        background: rgba(20, 10, 5, 0.7);
        border-radius: 18px;
        border: 1px solid rgba(255, 140, 60, 0.3);
        cursor: pointer;
        transition: all 0.3s;

        svg {
          width: 44px;
          height: 44px;
          filter: drop-shadow(0 0 15px rgba(255, 100, 30, 0.4));
          z-index: 2;
        }

        /* 四层火焰颜色 */
        .flame-core.flame-level-1 {
          color: #ff974f;
        }
        .flame-core.flame-level-2 {
          color: #ffb47b;
        }
        .flame-core.flame-level-3 {
          color: #ffc99e;
        }
        .flame-core.flame-level-4 {
          color: #ffe0ba;
        }

        .sparks circle {
          animation: spark-float 2.5s infinite alternate;
          &:nth-child(2) {
            animation-delay: 0.3s;
          }
          &:nth-child(3) {
            animation-delay: 0.6s;
          }
          &:nth-child(4) {
            animation-delay: 0.9s;
          }
        }

        /* 墨迹光圈 [citation:8] */
        .icon-ink-ring {
          position: absolute;
          width: 80px;
          height: 80px;
          border: 1px solid rgba(255, 140, 60, 0.2);
          border-radius: 50%;
          opacity: 0;
          transition: all 0.4s;
          pointer-events: none;
        }

        &:hover {
          transform: translateY(-4px) scale(1.05);
          border-color: rgba(255, 140, 60, 0.8);
          box-shadow: 0 15px 30px rgba(255, 100, 30, 0.4);

          .icon-ink-ring {
            opacity: 0.5;
            transform: scale(1.2);
          }
        }
      }
    }

    .message-content {
      font-size: 1rem;
      line-height: 1.7;
      color: #ecdacb;
      margin: 0;
      padding-left: 72px;
      white-space: pre-wrap;
      word-break: break-word;
      position: relative;
      z-index: 2;
    }

    /* 卡片底部火焰装饰条 */
    .card-footer-flame {
      position: absolute;
      bottom: 0;
      left: 10%;
      width: 80%;
      height: 4px;
      display: flex;
      gap: 2px;
      opacity: 0.3;
      transition: opacity 0.3s;

      .flame-segment {
        flex: 1;
        height: 100%;
        background: linear-gradient(90deg, var(--accent), var(--accent-2));
        border-radius: 2px;
        animation: segment-flicker 2s infinite alternate;

        &:nth-child(1) {
          animation-delay: 0s;
        }
        &:nth-child(2) {
          animation-delay: 0.2s;
        }
        &:nth-child(3) {
          animation-delay: 0.4s;
        }
        &:nth-child(4) {
          animation-delay: 0.6s;
        }
      }
    }

    &:hover .card-footer-flame {
      opacity: 0.8;
    }
  }

  /* ========== 底部发送区（增强版） ========== */
  .message-form {
    position: fixed;
    left: 50%;
    transform: translateX(-50%);
    bottom: 24px;
    width: calc(100% - 32px);
    max-width: 1000px;
    background: rgba(10, 5, 3, 0.85);
    backdrop-filter: blur(20px);
    padding: 24px 30px;
    border-radius: 60px;
    border: 1px solid rgba(255, 140, 60, 0.3);
    box-shadow: 0 30px 60px -10px rgba(0, 0, 0, 0.9),
      0 0 50px rgba(255, 80, 20, 0.2);
    z-index: 20;
    overflow: hidden;

    /* 四层动态火焰边框 [citation:2] */
    .form-flame-border {
      position: absolute;
      inset: -2px;
      border-radius: 62px;
      overflow: hidden;
      pointer-events: none;

      .flame-layer {
        position: absolute;
        inset: 0;
        border: 2px solid transparent;
        border-radius: 62px;
        background: linear-gradient(
            90deg,
            transparent,
            var(--accent),
            transparent
          )
          border-box;
        -webkit-mask: linear-gradient(#fff 0 0) padding-box,
          linear-gradient(#fff 0 0);
        -webkit-mask-composite: xor;
        mask-composite: exclude;
        opacity: 0.3;
        animation: flame-border-rotate 8s linear infinite;

        &.layer1 {
          filter: blur(2px);
          opacity: 0.6;
        }
        &.layer2 {
          filter: blur(4px);
          animation-duration: 6s;
        }
        &.layer3 {
          filter: blur(6px);
          animation-duration: 10s;
        }
        &.layer4 {
          filter: blur(8px);
          animation-duration: 12s;
        }
      }
    }

    /* 焚香烟雾效果 */
    .incense-smoke {
      position: absolute;
      bottom: 100%;
      left: 20%;
      width: 100px;
      height: 100px;
      background: radial-gradient(
        circle,
        rgba(255, 200, 150, 0.1),
        transparent
      );
      filter: blur(20px);
      animation: smoke-rise 8s infinite ease-out;
      pointer-events: none;
    }

    input,
    textarea {
      width: 100%;
      padding: 16px 22px;
      border-radius: 40px;
      border: 1px solid rgba(255, 140, 60, 0.2);
      background: rgba(0, 0, 0, 0.4);
      color: #f0dac8;
      font-size: 1rem;
      transition: all 0.3s;
      margin-bottom: 16px;
      position: relative;
      z-index: 2;

      &::placeholder {
        color: #b99f8b;
      }

      &:focus {
        outline: none;
        border-color: rgba(255, 140, 60, 0.8);
        box-shadow: 0 0 0 4px rgba(255, 100, 30, 0.15);
        background: rgba(30, 15, 8, 0.6);
        transform: translateY(-2px);
      }
    }

    textarea {
      min-height: 90px;
      resize: none;
    }

    .form-row {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 20px;

      .hint {
        display: flex;
        align-items: center;
        gap: 10px;
        color: #ffb47b;

        .hint-flame {
          width: 20px;
          height: 20px;
          background: radial-gradient(circle, #ff974f, #ff4f1e);
          border-radius: 50%;
          filter: blur(5px);
          animation: flame-pulse 1.5s infinite alternate;
        }

        .hint-text {
          font-size: 0.9rem;
        }

        kbd {
          background: rgba(30, 15, 8, 0.8);
          border-radius: 8px;
          padding: 4px 10px;
          border: 1px solid #ff6324;
          color: #ffd9b0;
          font-size: 0.8rem;
          margin: 0 2px;
        }
      }

      button {
        position: relative;
        padding: 14px 36px;
        background: linear-gradient(145deg, #ff974f, #ff4f1e);
        border: none;
        border-radius: 50px;
        color: #1a0c06;
        font-weight: 700;
        font-size: 1rem;
        cursor: pointer;
        transition: all 0.3s;
        box-shadow: 0 15px 30px rgba(255, 80, 20, 0.5);
        overflow: hidden;

        &::before {
          content: "";
          position: absolute;
          top: 0;
          left: -100%;
          width: 100%;
          height: 100%;
          background: linear-gradient(
            90deg,
            transparent,
            rgba(255, 255, 255, 0.2),
            transparent
          );
          transition: left 0.6s;
        }

        .button-spark {
          position: absolute;
          top: -10px;
          right: -10px;
          width: 30px;
          height: 30px;
          background: radial-gradient(circle, #ffd9b0, transparent);
          filter: blur(8px);
          animation: spark-rotate 3s infinite linear;
        }

        &:hover:not(:disabled) {
          transform: translateY(-4px) scale(1.02);
          box-shadow: 0 25px 40px rgba(255, 100, 30, 0.7);

          &::before {
            left: 100%;
          }
        }

        &:disabled {
          opacity: 0.5;
          cursor: not-allowed;
          background: linear-gradient(145deg, #664433, #332211);
        }
      }
    }

    /* 底部棋局纹样 [citation:1][citation:8] */
    .form-chess-pattern {
      position: absolute;
      bottom: -20px;
      left: 10%;
      width: 80%;
      height: 20px;
      background: repeating-linear-gradient(
        90deg,
        transparent,
        transparent 20px,
        rgba(255, 140, 60, 0.1) 20px,
        rgba(255, 140, 60, 0.1) 40px
      );
      opacity: 0.3;
      pointer-events: none;
    }
  }

  /* 响应式 */
  @media (max-width: 980px) {
    .carousel1 {
      display: none;
    }
    .carousel2 {
      display: block;
    }

    .board-header {
      .title-wrap {
        margin-left: 0;
      }
 
    }

    .talisman {
      display: none;
    }
  }

  @media (max-width: 768px) {
    padding-top: 120px;

    .board-header {
      padding: 16px 20px;

      .title-wrap h1 {
        font-size: 1.5rem;
      }
      .subtitle {
        margin-left: 0;
      }
    }

    .message-card {
      padding: 18px 20px;

      .message-content {
        padding-left: 0;
        margin-top: 12px;
      }
    }

    .message-form {
      padding: 20px;

      .form-row {
        flex-direction: column;
        align-items: stretch;

        button {
          width: 100%;
        }
        .hint{
          display: none;
        }
      }
    }

    .feather-fall {
      display: none;
    }
  }
}

/* ========== 动画定义 ========== */
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

@keyframes feather-fall {
  0% {
    transform: translateY(0) rotate(0deg);
    opacity: 0;
  }
  10% {
    opacity: 0.3;
  }
  90% {
    opacity: 0.2;
  }
  100% {
    transform: translateY(100vh) rotate(360deg);
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
    filter: blur(12px);
  }
}

@keyframes glow-pulse-secondary {
  0% {
    opacity: 0.2;
    filter: blur(3px);
  }
  100% {
    opacity: 0.5;
    filter: blur(10px);
  }
}

@keyframes border-flow {
  0% {
    left: -100%;
  }
  50% {
    left: 100%;
  }
  100% {
    left: 200%;
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

@keyframes feather-glow {
  0% {
    opacity: 0.3;
  }
  100% {
    opacity: 0.8;
  }
}

@keyframes skeleton-pulse {
  0% {
    opacity: 0.4;
  }
  100% {
    opacity: 0.8;
  }
}

@keyframes spark-pulse {
  0% {
    opacity: 0.5;
    transform: scale(1);
  }
  100% {
    opacity: 1;
    transform: scale(1.3);
  }
}

@keyframes spark-float {
  0% {
    transform: translate(0, 0) scale(1);
    opacity: 0.3;
  }
  100% {
    transform: translate(-5px, -8px) scale(1.8);
    opacity: 1;
  }
}

@keyframes segment-flicker {
  0% {
    opacity: 0.3;
    height: 2px;
  }
  100% {
    opacity: 1;
    height: 6px;
  }
}

@keyframes flame-border-rotate {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

@keyframes smoke-rise {
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
    transform: translateY(-100px) scale(2);
    opacity: 0;
  }
}

@keyframes spark-rotate {
  0% {
    transform: rotate(0deg) scale(1);
  }
  50% {
    transform: rotate(180deg) scale(1.2);
  }
  100% {
    transform: rotate(360deg) scale(1);
  }
}

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
</style>
