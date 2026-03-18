<template>
  <div class="yuzuki-resources">
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
    <header class="hero">
      <div class="hero-inner">
        <h1>资源分享</h1>
        <p class="subtitle">可自由上传关于长离的相关链接</p>
      </div>
    </header>

    <main class="container">
      <section class="uploader" :class="{ collapsed: uploaderCollapsed }">
        <div class="uploader-head">
          <button
            class="toggle"
            @click="toggleUploader"
            :aria-expanded="!uploaderCollapsed"
          >
            <span v-if="uploaderCollapsed">展开上传区</span>
            <span v-else>收起上传区</span>
          </button>
        </div>

        <form
          @submit.prevent="addResource"
          class="upload-form"
          :aria-hidden="uploaderCollapsed"
        >
          <div class="row">
            <input
              v-model="form.title"
              type="text"
              placeholder="标题（必填，如果有解压码之类的也写这里吧）"
              aria-label="标题"
            />
            <input
              v-model="form.type"
              type="text"
              placeholder="链接类型(网页链接、b站视频、网盘链接等等)"
              aria-label="来源"
            />
          </div>

          <div class="row">
            <input
              v-model="form.uploader"
              type="text"
              placeholder="上传人（可选）"
              aria-label="上传人"
            />
            <input
              v-model="form.link"
              type="url"
              placeholder="链接(只输入网址不能有中文)"
              aria-label="链接"
            />
          </div>

          <div class="actions">
            <button type="submit" class="btn primary">上传</button>
          </div>
        </form>
      </section>

      <section class="list">
        <div class="list-header">
          <h2>资源列表（{{ resources.length }}）</h2>
          <div class="sort">
            <label>
              排序：
              <select v-model="sortBy">
                <option value="time">按时间（新→旧）</option>
                <option value="likes">按点赞（高→低）</option>
              </select>
            </label>
          </div>
        </div>

        <ul class="items">
          <li v-for="item in sortedResources" :key="item.id" class="item">
            <a
              :href="item.link"
              target="_blank"
              rel="noopener noreferrer"
              class="title"
              >{{ item.title }}</a
            >

            <div class="meta">
              <div class="left">
                <span class="uploader">{{ item.uploader || "匿名" }}</span>
                <span class="dot">•</span>
                <time :datetime="item.time">{{ formatTime(item.time) }}</time>
              </div>

              <div class="right">
                <button
                  @click.prevent="handleLike(item)"
                  :aria-pressed="likedIds.has(String(item.id))"
                  class="like-btn"
                  :class="{ active: likedIds.has(String(item.id)) }"
                >
                  <img
                    :src="
                      likedIds.has(String(item.id))
                        ? '/icons/heart-red-filled.svg'
                        : '/icons/heart-red-outline.svg'
                    "
                    class="heart-icon"
                    alt="heart"
                  />
                  <span class="count">{{ item.likes }}</span>
                </button>

                <span class="badge">{{ item.type }}</span>
              </div>
            </div>
          </li>
        </ul>

        <p v-if="resources.length === 0" class="empty">
          目前没有资源，快来上传第一条吧！
        </p>
      </section>
    </main>

    <footer class="foot">提示：点击标题将直接跳转</footer>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount } from "vue";
// 如果你的工程使用 ts 路径别名 @ 指向 src，可以用 '@/api/resource'，否则根据实际路径调整
import {
  getResourceList,
  createResource,
  likeResource,
} from "@/api/modules/resource";
import { ElMessage } from "element-plus";

interface Resource {
  id: number | string;
  title: string;
  uploader?: string;
  time: string; // ISO 或 created_at
  likes: number;
  link: string;
  type: string;
  role_key?: string;
}

const STORAGE_KEY = "changLi_resources_v1";
const DEFAULT_ROLE = "changLi";

const form = ref<{
  title: string;
  uploader: string;
  link: string;
  type: string;
}>({
  title: "",
  uploader: "",
  link: "",
  type: "",
});

const resources = ref<Resource[]>([]);
const likedIds = ref(new Set<string>());
const sortBy = ref<"time" | "likes">("time");
const uploaderCollapsed = ref(false);

function mapServerToLocal(row: any): Resource {
  return {
    id: row.id,
    title: row.title,
    uploader: row.uploader || "匿名",
    time: row.created_at || row.time || new Date().toISOString(),
    likes: row.likes ?? 0,
    link: row.link,
    type: row.storage_type || row.type || "other",
    role_key: row.role_key,
  };
}

async function loadResources() {
  try {
    // 尝试从后端拉取（分页可扩展，这里一次拉足够 demo）
    const res: any = await getResourceList({
      role_key: DEFAULT_ROLE,
      page: 1,
      pageSize: 100,
    });
    if (res && res.success && Array.isArray(res.data)) {
      resources.value = res.data.map(mapServerToLocal);
      // 可恢复本地点赞状态（仅 UI 记忆）
      const raw = localStorage.getItem(STORAGE_KEY);
      if (raw) {
        try {
          const parsed = JSON.parse(raw);
          if (parsed.liked && Array.isArray(parsed.liked)) {
            parsed.liked.forEach((id: string) => likedIds.value.add(id));
          }
        } catch (e) {
          /* ignore */
        }
      }
      return;
    }
  } catch (err) {
    console.warn("拉取资源失败，使用本地缓存", err);
  }
  // 回退：本地缓存（仅恢复点赞状态）
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (raw) {
      const parsed = JSON.parse(raw);
      if (parsed.liked && Array.isArray(parsed.liked)) {
        parsed.liked.forEach((id: string) => likedIds.value.add(id));
      }
    }
  } catch (e) {
    console.warn("本地加载失败", e);
  }
}

function saveLocalCache() {
  try {
    const liked = Array.from(likedIds.value);
    localStorage.setItem(STORAGE_KEY, JSON.stringify({ liked }));
  } catch (e) {
    console.warn("保存本地缓存失败", e);
  }
}

// ========== 背景图片导入与轮播 ==========
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
let Imgtimer: number | undefined;

onMounted(() => {
  loadResources();
  Imgtimer = window.setInterval(() => {
    currentIndex.value =
      (currentIndex.value + 1) % Math.max(1, randomFive.value.length);
  }, 5200);
  uploaderCollapsed.value = window.innerWidth <= 640;
});
function toggleUploader() {
  uploaderCollapsed.value = !uploaderCollapsed.value;
}
onBeforeUnmount(() => {
  if (Imgtimer) clearInterval(Imgtimer);
});

async function addResource() {
  const t = form.value.title.trim();
  const l = form.value.link.trim();
  if (!form.value.title.trim() || !form.value.link.trim()) {
    return ElMessage.warning("请填写完整信息");
  }
  if (!/^https?:\/\//i.test(l)) {
    return ElMessage.error("请输入正确的链接(https开头)");
  }
  // 尝试调用后端接口
  try {
    const payload = {
      title: t,
      uploader: form.value.uploader.trim() || "匿名",
      link: l,
      storage_type: form.value.type,
      role_key: DEFAULT_ROLE,
    };
    const res: any = await createResource(payload);
    if (res && res.success && res.data) {
      const added = mapServerToLocal(res.data);
      resources.value.unshift(added);
      // 自动展开到顶部展示（可选）
      saveLocalCache();
      resetForm();
      ElMessage.success("上传成功");
      return;
    }
    ElMessage.error("上传失败");
  } catch (err) {
    console.warn("创建资源失败", err);
  }
}

function resetForm() {
  form.value.title = "";
  form.value.uploader = "";
  form.value.link = "";
  form.value.type = "";
}

async function handleLike(item: Resource) {
  // UI 乐观更新
  const id = item.id;
  const wasLiked = likedIds.value.has(String(id));
  if (wasLiked) {
    likedIds.value.delete(String(id));
    item.likes = Math.max(0, item.likes - 1);
  } else {
    likedIds.value.add(String(id));
    item.likes++;
  }
  saveLocalCache();

  // 同步后端（不依赖返回值进行 UI 回滚，简单处理：若失败则回退）
  try {
    const action = wasLiked ? "unlike" : "like";
    const res: any = await likeResource(id, action);
    if (
      res &&
      res.success &&
      res.data &&
      typeof res.data.likes !== "undefined"
    ) {
      item.likes = res.data.likes;
    }
  } catch (err) {
    console.warn("点赞接口调用失败，回滚本地状态", err);
    // 回滚
    if (wasLiked) {
      // 本来是已赞，取消失败 -> 重新添加
      likedIds.value.add(String(id));
      item.likes++;
    } else {
      likedIds.value.delete(String(id));
      item.likes = Math.max(0, item.likes - 1);
    }
    saveLocalCache();
  }
}

const sortedResources = computed(() => {
  const arr = [...resources.value];
  if (sortBy.value === "time") {
    arr.sort((a, b) => +new Date(b.time) - +new Date(a.time));
  } else {
    arr.sort((a, b) => b.likes - a.likes);
  }
  return arr;
});

function formatTime(iso: string) {
  try {
    const d = new Date(iso);
    return new Intl.DateTimeFormat("zh-CN", {
      month: "2-digit",
      day: "2-digit",
      hour: "2-digit",
      minute: "2-digit",
    }).format(d);
  } catch (e) {
    return iso;
  }
}
</script>

<style scoped lang="scss">
/* ========== 长离主题色系 ========== */

.yuzuki-resources {
  --deep-bg: #0a0503;
  --accent: #ff4f1e;
  --accent-2: #ff974f;
  --accent-3: #ffcb9a;
  --text-light: #f7e4d4;
  --glass-bg: rgba(10, 5, 3, 0.7);
  --flame-glow: rgba(255, 80, 20, 0.3);
  --card-bg: rgba(15, 8, 4, 0.6);
  --border-glow: rgba(255, 140, 60, 0.2);
  position: relative;
  min-height: 100vh;
  padding-top: 60px;
  background: linear-gradient(135deg, #1a0c06 0%, #0a0503 50%, #1f0e07 100%);
  color: var(--text-light);
  font-family: "Noto Sans SC", system-ui, -apple-system, sans-serif;
  overflow-x: hidden;

  /* ========== 轮播背景（保留原有，调整遮罩颜色） ========== */
  .carousel {
    position: absolute;
    inset: 0;
    z-index: 0;
    pointer-events: none;

    &::before {
      content: "";
      position: absolute;
      inset: 0;
      background: linear-gradient(
        180deg,
        rgba(10, 5, 3, 0.3),
        rgba(10, 5, 3, 0.45)
      );
      z-index: 1;
    }

    .carousel-image {
      position: absolute;
      width: 100%;
      height: 100%;
      object-fit: cover;
      opacity: 0;
      filter: blur(1px) saturate(0.8);
      transition: opacity 1.5s ease;

      &.active {
        opacity: 1;
      }
    }
  }

  .carousel2 {
    display: none;
  }

  /* ========== 顶部 HERO ========== */
  .hero {
    padding: 18px 12px;
    background: rgba(10, 5, 3, 0.6);
    backdrop-filter: blur(2px);
    border-bottom: 1px solid rgba(255, 140, 60, 0.2);
    position: relative;
    z-index: 10;

    .hero-inner {
      max-width: 1000px;
      margin: 0 auto;
      display: flex;
      flex-direction: column;
      gap: 6px;

      h1 {
        margin: 0;
        font-size: 20px;
        font-weight: 900;
        background: linear-gradient(135deg, #ffdbb0, #ffb47b, #ff6324);
        -webkit-background-clip: text;
        background-clip: text;
        color: transparent;
        letter-spacing: 1px;
      }

      .subtitle {
        margin-top: 6px;
        color: #ffb47b;
        font-size: 13px;
        font-style: italic;
      }
    }
  }

  /* ========== 主容器 ========== */
  .container {
    position: relative;
    z-index: 10;
    max-width: 1000px;
    margin: 16px auto;
    padding: 0 12px;
    width: 100%;
    box-sizing: border-box;
  }

  /* ========== 上传区（毛玻璃火焰） ========== */
  .uploader {
    border-radius: 28px;
    padding: 0;
    background: rgba(10, 5, 3, 0.6);
    backdrop-filter: blur(1px);
    border: 1px solid rgba(255, 140, 60, 0.2);
    box-shadow: 0 20px 40px -10px rgba(0, 0, 0, 0.6);
    transition: all 0.3s;

    &:hover {
      border-color: rgba(255, 140, 60, 0.4);
      box-shadow: 0 25px 50px -10px rgba(255, 100, 30, 0.2);
    }

    .uploader-head {
      display: flex;
      justify-content: flex-end;
      padding: 10px 16px;

      .toggle {
        background: rgba(30, 15, 8, 0.6);
        border: 1px solid rgba(255, 140, 60, 0.3);
        color: #ffb47b;
        padding: 6px 16px;
        border-radius: 30px;
        cursor: pointer;
        font-weight: 700;
        transition: all 0.3s;

        &:hover {
          background: rgba(50, 25, 15, 0.8);
          border-color: var(--accent-2);
          transform: translateY(-2px);
        }
      }
    }

    .upload-form {
      padding: 16px 20px 24px;
      max-height: 1600px;
      overflow: hidden;
      transition: max-height 0.3s ease, padding 0.3s ease;

      .row {
        display: flex;
        gap: 12px;
        margin-bottom: 12px;

        input {
          flex: 1;
          padding: 12px 16px;
          border-radius: 40px;
          border: 1px solid rgba(255, 140, 60, 0.2);
          background: rgba(0, 0, 0, 0.3);
          color: var(--text-light);
          font-size: 14px;
          outline: none;
          transition: all 0.3s;

          &::placeholder {
            color: #b99f8b;
          }

          &:focus {
            border-color: var(--accent-2);
            box-shadow: 0 0 0 3px rgba(255, 100, 30, 0.2);
            background: rgba(20, 10, 5, 0.5);
          }
        }

        select {
          max-width: 140px;
          padding: 12px 16px;
          border-radius: 40px;
          border: 1px solid rgba(255, 140, 60, 0.2);
          background: rgba(0, 0, 0, 0.3);
          color: #ffb47b;
          outline: none;
        }
      }

      .actions {
        display: flex;
        gap: 12px;
        align-items: center;

        .btn {
          padding: 10px 24px;
          border-radius: 40px;
          border: none;
          font-weight: 700;
          cursor: pointer;
          transition: all 0.3s;

          &.primary {
            background: linear-gradient(145deg, #ff974f, #ff4f1e);
            color: #1a0c06;
            box-shadow: 0 10px 25px rgba(255, 80, 20, 0.4);

            &:hover:not(:disabled) {
              transform: translateY(-3px);
              box-shadow: 0 15px 35px rgba(255, 100, 30, 0.6);
            }
          }

          &.secondary {
            background: transparent;
            border: 1px solid rgba(255, 140, 60, 0.3);
            color: #ffb47b;

            &:hover {
              background: rgba(255, 100, 30, 0.1);
            }
          }
        }
      }
    }

    &.collapsed .upload-form {
      max-height: 0;
      padding-top: 0;
      padding-bottom: 0;
    }
  }

  /* ========== 资源列表 ========== */
  .list {
    margin-top: 24px;

    .list-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 16px;
      padding: 0 4px;

      h2 {
        font-size: 1.2rem;
        margin: 0;
        color: #ffb47b;
        font-weight: 700;
      }

      .sort select {
        padding: 8px 16px;
        border-radius: 30px;
        border: 1px solid rgba(255, 140, 60, 0.2);
        background: rgba(0, 0, 0, 0.3);
        color: #ffb47b;
        outline: none;
        cursor: pointer;

        &:focus {
          border-color: var(--accent-2);
        }
      }
    }

    .items {
      list-style: none;
      padding: 0;
      margin: 0;

      .item {
        background: rgba(10, 5, 3, 0.5);
        backdrop-filter: blur(1px);
        border-radius: 24px;
        padding: 18px 22px;
        margin-bottom: 14px;
        border: 1px solid rgba(255, 140, 60, 0.15);
        box-shadow: 0 15px 35px -10px rgba(0, 0, 0, 0.5);
        transition: all 0.3s cubic-bezier(0.2, 0.9, 0.3, 1);
        position: relative;
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
            var(--accent-2),
            transparent
          );
          transition: left 0.6s;
        }

        &:hover {
          transform: translateY(-6px);
          border-color: rgba(255, 140, 60, 0.4);
          box-shadow: 0 25px 50px -10px rgba(255, 100, 30, 0.2);

          &::before {
            left: 100%;
          }
        }

        .title {
          display: block;
          color: #ffd9b0;
          font-weight: 700;
          text-decoration: none;
          margin-bottom: 12px;
          font-size: 1.1rem;
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
          transition: color 0.3s;

          &:hover {
            color: var(--accent-2);
          }
        }

        .meta {
          display: flex;
          justify-content: space-between;
          align-items: center;
          font-size: 0.9rem;

          .left {
            display: flex;
            align-items: center;
            gap: 8px;

            .uploader {
              color: #ffb47b;
              font-weight: 600;
            }

            .dot {
              color: #b99f8b;
            }

            time {
              color: #b99f8b;
            }
          }

          .right {
            display: flex;
            align-items: center;
            gap: 12px;

            .like-btn {
              display: flex;
              align-items: center;
              gap: 6px;
              background: rgba(30, 15, 8, 0.6);
              border: 1px solid rgba(255, 140, 60, 0.2);
              border-radius: 30px;
              padding: 4px 12px;
              cursor: pointer;
              transition: all 0.3s;

              &:hover {
                background: rgba(50, 25, 15, 0.8);
                border-color: var(--accent-2);
                transform: translateY(-2px);
              }

              .heart-icon {
                width: 18px;
                height: 18px;
                filter: drop-shadow(0 0 4px rgba(255, 0, 0, 0.5));
              }

              .count {
                color: #ffb47b;
                font-weight: 600;
              }

              &.active .heart-icon {
                filter: drop-shadow(0 0 8px #ff4f1e);
              }
            }

            .badge {
              padding: 4px 12px;
              border-radius: 30px;
              font-size: 0.8rem;
              font-weight: 600;
              background: rgba(255, 100, 30, 0.15);
              border: 1px solid rgba(255, 140, 60, 0.3);
              color: #ffb47b;
            }
          }
        }
      }
    }

    .empty {
      text-align: center;
      color: #b99f8b;
      padding: 40px 0;
      font-style: italic;
    }
  }

  /* 底部提示 */
  .foot {
    position: relative;
    z-index: 10;
    text-align: center;
    color: #b99f8b;
    font-size: 0.85rem;
    margin: 30px 0 40px;
  }

  /* ========== 动画 ========== */
  @keyframes feather-float {
    0% {
      transform: rotate(8deg) scale(1);
      opacity: 0.2;
    }
    100% {
      transform: rotate(25deg) scale(1.3);
      opacity: 0.4;
    }
  }

  /* ========== 响应式 ========== */
  @media (max-width: 1024px) {
    .carousel1 {
      display: none;
    }
    .carousel2 {
      display: block;
    }
  }

  @media (max-width: 768px) {
    padding-top: 80px;

    .hero {
      padding: 12px 10px;
    }

    .upload-form .row {
      flex-direction: column;
    }

    .actions {
      flex-direction: column;
      align-items: stretch;
    }

    .items .item .title {
      white-space: normal;
    }
  }
}
</style>
