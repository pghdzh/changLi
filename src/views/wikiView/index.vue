<template>
  <div class="wiki-page">
    <!-- 背景轮播放在最底层 -->
    <div class="carousel">
      <img
        v-for="(src, idx) in randomFive"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>
    <header class="wiki-header">
      <div class="title">
        <h1>长离文本分享</h1>
        <p class="subtitle">焚言寄愿，心焰长明</p>
      </div>
      <div class="actions">
        <input
          v-model="search"
          class="search"
          placeholder="搜索标题或者标签..."
        />
        <button class="btn btn-new" @click="openCreate">新建词条</button>
      </div>
    </header>

    <main class="wiki-body">
      <div v-if="filteredEntries.length === 0" class="empty">
        没有找到匹配的词条 ✨
      </div>

      <ul class="entry-list">
        <li v-for="entry in filteredEntries" :key="entry.id" class="entry-card">
          <div class="entry-head">
            <div class="entry-meta" @click="openDetail(entry)">
              <div class="entry-title-wrap">
                <h2 class="entry-title">{{ entry.title }}</h2>
                <span class="entry-badge">#{{ entry.slug || "未设置" }}</span>
              </div>
              <div class="entry-info">
                <span class="meta-item">作者：{{ entry.author }}</span>
                <span class="meta-item"
                  >时间：{{ formatTime(entry.updatedAt) }}</span
                >
              </div>
            </div>

            <div class="entry-actions">
              <button
                class="like"
                :class="{ active: isLiked(entry.id) }"
                :aria-pressed="isLiked(entry.id)"
                @click.stop="toggleLike(entry.id)"
              >
                <img
                  :src="
                    isLiked(entry.id)
                      ? '/icons/heart-red-filled.svg'
                      : '/icons/heart-red-outline.svg'
                  "
                  alt="like"
                />
                <span class="like-count">{{ entry.likes }}</span>
              </button>
              <div class="edit-delete" v-if="canEdit(entry.id)">
                <button class="small" @click="openEdit(entry)">编辑</button>
                <button class="small danger" @click="remove(entry.id)">
                  删除
                </button>
              </div>
            </div>
          </div>
        </li>
      </ul>
    </main>

    <!-- Edit/Create Modal -->
    <transition name="fade-zoom">
      <div class="modal-overlay" v-if="showModal">
        <div class="modal">
          <header class="modal-header">
            <h3>{{ editing ? "编辑词条" : "新建词条" }}</h3>
            <button class="close" @click="closeModal">✕</button>
          </header>
          <section class="modal-body">
            <label>
              标题
              <input v-model="form.title" placeholder="输入标题" />
            </label>

            <label>
              词条（短标签）
              <input
                v-model="form.slug"
                placeholder="比如：彩蛋、考据、点位等等"
              />
            </label>

            <label>
              作者
              <input v-model="form.author" placeholder="作者昵称" />
            </label>

            <label>
              内容
              <textarea
                v-model="form.content"
                rows="8"
                placeholder="在这里输入词条内容"
              ></textarea>
            </label>
          </section>
          <footer class="modal-footer">
            <button class="btn ghost" @click="closeModal">取消</button>
            <button class="btn" @click="submit">
              {{ editing ? "保存" : "创建" }}
            </button>
          </footer>
        </div>
      </div>
    </transition>

    <!-- Detail Modal -->
    <transition name="fade-zoom">
      <div class="modal-overlay" v-if="detailEntry">
        <div class="modal detail-modal">
          <header class="modal-header">
            <h3>{{ detailEntry.title }}</h3>
            <button class="close" @click="detailEntry = null">✕</button>
          </header>
          <section class="modal-body">
            <div class="detail-content">{{ detailEntry.content }}</div>
          </section>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed, onMounted, onUnmounted } from "vue";
import { ElMessage } from "element-plus";
import {
  getWikiList,
  createWiki,
  updateWiki,
  deleteWiki,
  likeWiki,
} from "@/api/modules/wiki";

// 本地存储自己创建的词条 ID
const LS_MY_WIKI_IDS = "yuzuriha:wiki:my_ids";
const myWikiIds: string[] = JSON.parse(
  localStorage.getItem(LS_MY_WIKI_IDS) || "[]"
);
const markAsMine = (id: string | number) => {
  if (!myWikiIds.includes(String(id))) {
    myWikiIds.push(String(id));
    localStorage.setItem(LS_MY_WIKI_IDS, JSON.stringify(myWikiIds));
  }
};
const canEdit = (id: string | number) => myWikiIds.includes(String(id));

// 数据状态
const entries = ref<any[]>([]);

// 本地存储键
const LS_LIKED_IDS = "yuzuriha:wiki:liked_ids";
// 从 localStorage 读取已点赞 id 列表（字符串形式）
const likedIds = ref<string[]>(
  JSON.parse(localStorage.getItem(LS_LIKED_IDS) || "[]")
);

const showModal = ref(false);
const editing = ref(false);
const editingId = ref<string | number | null>(null);
const detailEntry = ref<any>(null);
const form = reactive({ title: "", slug: "", author: "", content: "" });
const search = ref("");

// 时间格式化
function formatTime(ts: string | number | null | undefined) {
  if (!ts) return "未知时间";
  const date = new Date(ts);
  if (isNaN(date.getTime())) return "未知时间";
  return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(
    2,
    "0"
  )}-${String(date.getDate()).padStart(2, "0")}`;
}

// 加载词条列表
async function loadEntries() {
  try {
    const res: any = await getWikiList();
    entries.value = res.data.map((e: any) => ({
      ...e,
      createdAt: e.createdAt || e.created_at,
      updatedAt: e.updatedAt || e.updated_at,
    }));
  } catch (err) {
    console.error(err);
    ElMessage.error("加载词条失败");
  }
}

// 打开/关闭弹窗
function openCreate() {
  editing.value = false;
  editingId.value = null;
  form.title = "";
  form.slug = "";

  form.content = "";
  showModal.value = true;
}
function openEdit(entry: any) {
  if (!canEdit(entry.id)) {
    ElMessage.warning("只有创建者可以编辑");
    return;
  }
  editing.value = true;
  editingId.value = entry.id;
  form.title = entry.title;
  form.slug = entry.slug;
  form.author = entry.author;
  form.content = entry.content;
  showModal.value = true;
}
function closeModal() {
  showModal.value = false;
}

const canSubmit = computed(() => form.title.trim() && form.content.trim());

// 提交
async function submit() {
  if (!canSubmit.value) {
    ElMessage.warning("请填写标题和内容");
    return;
  }
  const payload = {
    title: form.title.trim(),
    author: form.author.trim() || "匿名",
    content: form.content.trim(),
    slug: null,
  };
  if (form.slug.trim()) payload.slug = form.slug.trim();
  try {
    if (editing.value && editingId.value) {
      await updateWiki(editingId.value, payload);
      ElMessage.success("编辑成功");
    } else {
      const res: any = await createWiki(payload);
      markAsMine(res.data.id);
      editingId.value = res.data.id;
      ElMessage.success("创建成功");
    }
    showModal.value = false;
    loadEntries();
  } catch (err) {
    console.error(err);
    ElMessage.error("提交失败");
  }
}

// 删除
async function remove(id: string | number) {
  if (!canEdit(id)) {
    ElMessage.warning("只有创建者可以删除");
    return;
  }
  if (!confirm("确认删除该词条？此操作不可撤销")) return;
  try {
    await deleteWiki(id);
    const index = myWikiIds.indexOf(String(id));
    if (index !== -1) myWikiIds.splice(index, 1);
    localStorage.setItem(LS_MY_WIKI_IDS, JSON.stringify(myWikiIds));
    ElMessage.success("删除成功");
    loadEntries();
  } catch (err) {
    console.error(err);
    ElMessage.error("删除失败");
  }
}

// 点赞
function persistLikedIds() {
  try {
    localStorage.setItem(LS_LIKED_IDS, JSON.stringify(likedIds.value));
  } catch (e) {
    console.warn("保存 likedIds 失败", e);
  }
}

// 判断是否已点赞（供模板绑定 class/aria-pressed）
function isLiked(id: string | number) {
  return likedIds.value.includes(String(id));
}

// 点赞 / 取消点赞（乐观更新，本地仅存 id，点赞数使用 entry.likes）
async function toggleLike(id: string | number) {
  const entry = entries.value.find((e) => e.id === id);
  if (!entry) return;

  const idStr = String(id);
  const wasLiked = likedIds.value.includes(idStr);

  // 乐观更新 UI（立即反映）
  if (wasLiked) {
    // 取消点赞：保证不低于 0
    entry.likes = Math.max(0, (entry.likes || 0) - 1);
    likedIds.value = likedIds.value.filter((x) => x !== idStr);
  } else {
    // 点赞
    entry.likes = (entry.likes || 0) + 1;
    likedIds.value.push(idStr);
  }
  persistLikedIds();

  try {
    // 调用后端（action: 'like' | 'unlike' | 'toggle'）
    // 我们明确传 'like' 或 'unlike'
    const action = wasLiked ? "unlike" : "like";
    await likeWiki(id, action);

    // 可选：如果后端在响应中返回了最新的 likes 数（res.data.likes），
    // 你可以在这里用后端值覆盖本地（示例注释）
    // const res = await likeWiki(id, action)
    // if (res?.data?.likes !== undefined) entry.likes = res.data.likes
  } catch (err) {
    // 出错则回滚乐观更新
    console.error("toggleLike error", err);
    if (wasLiked) {
      // 取消点赞失败 -> 重新标记为已点赞
      entry.likes = (entry.likes || 0) + 1;
      if (!likedIds.value.includes(idStr)) likedIds.value.push(idStr);
    } else {
      // 点赞失败 -> 取消之前增加的 count
      entry.likes = Math.max(0, (entry.likes || 0) - 1);
      likedIds.value = likedIds.value.filter((x) => x !== idStr);
    }
    persistLikedIds();
    ElMessage.error("点赞失败，请稍后重试");
  }
}

// 详情弹窗
async function openDetail(entry: any) {
  detailEntry.value = entry;
}

// 搜索过滤
const filteredEntries = computed(() => {
  const q = String(search.value || "")
    .trim()
    .toLowerCase();
  let list = entries.value;

  // 搜索过滤
  if (q) {
    list = list.filter(
      (e) =>
        (e.title || "").toLowerCase().includes(q) ||
        (e.slug || "").toLowerCase().includes(q)
    );
  }

  // 按点赞数排序（默认降序：点赞多的在前）
  return [...list].sort((a, b) => (b.likes || 0) - (a.likes || 0));
});

// 1. 分别导入两套图
const pcModules = import.meta.glob("@/assets/images1/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const mobileModules = import.meta.glob(
  "@/assets/images2/*.{jpg,png,jpeg,webp}",
  { eager: true }
);

const pcSrcs: string[] = Object.values(pcModules).map((m: any) => m.default);
const mobileSrcs: string[] = Object.values(mobileModules).map(
  (m: any) => m.default
);

// 洗牌函数
function shuffle<T>(arr: T[]): T[] {
  const a = arr.slice();
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}

const randomFive = ref<string[]>([]);
const currentIndex = ref(0);
let timer: number;

function pickImages() {
  const isMobile = window.innerWidth < 768;
  const all = isMobile ? mobileSrcs : pcSrcs;
  randomFive.value = shuffle(all).slice(0, 5);
}

onMounted(() => {
  loadEntries();
  pickImages(); // 首次判断
  // 监听窗口大小变化
  window.addEventListener("resize", pickImages);

  // 轮播
  timer = window.setInterval(() => {
    if (randomFive.value.length > 0) {
      currentIndex.value = (currentIndex.value + 1) % randomFive.value.length;
    }
  }, 5000);
});

onUnmounted(() => {
  clearInterval(timer);
  window.removeEventListener("resize", pickImages);
});
</script>

<style scoped lang="scss">
/* ========== 长离主题色系 ========== */
:root {
  --deep-bg: #0a0503;
  --accent: #ff4f1e;
  --accent-2: #ff974f;
  --accent-3: #ffcb9a;
  --text-light: #f7e4d4;
  --glass-bg: rgba(10, 5, 3, 0.7);
  --flame-glow: rgba(255, 80, 20, 0.3);
  --card-bg: rgba(15, 8, 4, 0.6);
  --border-glow: rgba(255, 140, 60, 0.2);
}

/* 覆盖原变量（保留原变量名以兼容原样式，但重新赋值） */
$bg-deep: #0a0503;
$deep-2: #1a0c06;
$accent-1: #ff4f1e;
$accent-2: #ff974f;
$text-main: #f7e4d4;
$muted: #f7e4d4;
$card-0: #1a0c06;
$card-1: #0a0503;
$gap-overlay-1: rgba(10, 5, 3, 0.7);
$gap-overlay-2: rgba(0, 0, 0, 0.85);
$border-weak: rgba(255, 140, 60, 0.2);
$accent-glow: rgba(255, 100, 30, 0.2);
$accent-glow-2: rgba(255, 100, 30, 0.1);
$danger: #ff6b6b;
$shadow-dark: rgba(0, 0, 0, 0.75);
$inset-glow: rgba(255, 140, 60, 0.1);

.wiki-page {
  position: relative;
  min-height: 100vh;
  color: $muted;
  padding: 16px;
  box-sizing: border-box;
  padding-top: 80px;
  background: linear-gradient(135deg, #1a0c06 0%, #0a0503 50%, #1f0e07 100%);

  /* 棋盘背景（固定） */
  &::before {
    content: "";
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-image: linear-gradient(
        rgba(255, 80, 30, 0.02) 1px,
        transparent 1px
      ),
      linear-gradient(90deg, rgba(255, 80, 30, 0.02) 1px, transparent 1px);
    background-size: 60px 60px;
    pointer-events: none;
    z-index: 0;
  }

  /* 底部火焰光带 */
  &::after {
    content: "";
    position: fixed;
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

  .carousel {
    position: absolute;
    inset: 0;
    z-index: -9;

    .carousel-image {
      position: absolute;
      width: 100%;
      height: 100%;
      object-fit: cover;
      opacity: 0;
      transition: opacity 1s ease;
      filter: blur(1.5px) grayscale(6%);
      background-blend-mode: overlay;
    }
    .carousel-image.active {
      opacity: 1;
    }

    &::before {
      content: "";
      position: absolute;
      inset: 0;
      background: linear-gradient(180deg, $gap-overlay-1, $gap-overlay-2);
      pointer-events: none;
      z-index: 1;
      mix-blend-mode: multiply;
    }
  }

  .wiki-header {
    position: relative;
    z-index: 10;
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 12px;
    padding: 12px;
    background: rgba(10, 5, 3, 0.6);
    backdrop-filter: blur(12px);
    border-radius: 12px;
    box-shadow: 0 6px 18px rgba(2, 6, 8, 0.6), 0 0 28px $inset-glow inset;
    flex-wrap: wrap;
    border: 1px solid $border-weak;
    overflow: hidden;

    &::after {
      content: "";
      position: absolute;
      right: 6%;
      top: -8%;
      width: 320px;
      height: 120px;
      pointer-events: none;
      background: radial-gradient(
          40px 12px at 10% 30%,
          rgba(255, 140, 60, 0.12),
          transparent 28%
        ),
        radial-gradient(
          80px 26px at 60% 50%,
          rgba(255, 100, 30, 0.06),
          transparent 46%
        ),
        repeating-linear-gradient(
          -25deg,
          rgba(0, 0, 0, 0.04) 0 4px,
          transparent 4px 12px
        );
      filter: blur(6px) contrast(1.02);
      transform: rotate(-6deg);
      opacity: 0.95;
      mix-blend-mode: screen;
      animation: arc-drift 8s ease-in-out infinite;
    }

    .title {
      h1 {
        margin: 0;
        font-size: 18px;
        font-weight: 700;
        background: linear-gradient(135deg, #ffdbb0, #ffb47b, #ff6324);
        -webkit-background-clip: text;
        background-clip: text;
        color: transparent;
        -webkit-text-fill-color: transparent;
        letter-spacing: 1px;
      }
      .subtitle {
        font-size: 12px;
        color: #ffb47b;
      }
    }

    .actions {
      display: flex;
      gap: 8px;
      align-items: center;
      flex-wrap: wrap;
    }

    .search {
      padding: 6px 10px;
      border-radius: 8px;
      border: 1px solid rgba(255, 140, 60, 0.2);
      background: rgba(0, 0, 0, 0.3);
      color: $text-main;
      font-size: 13px;
      transition: all 0.25s;

      &:focus {
        border-color: $accent-2;
        box-shadow: 0 0 0 3px rgba(255, 100, 30, 0.2);
        outline: none;
      }
    }

    .btn-new {
      background: linear-gradient(145deg, #ff974f, #ff4f1e);
      color: #1a0c06;
      border: none;
      border-radius: 12px;
      padding: 8px 16px;
      font-size: 14px;
      font-weight: 600;
      cursor: pointer;
      box-shadow: 0 6px 18px rgba(255, 80, 20, 0.4);
      position: relative;
      overflow: hidden;
      transition: all 0.22s ease;

      &:hover {
        transform: translateY(-3px);
        box-shadow: 0 15px 35px rgba(255, 100, 30, 0.6);
      }
      &:active {
        transform: translateY(0);
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.24);
      }
      &:after {
        content: "";
        position: absolute;
        inset: 0;
        background: radial-gradient(
          circle,
          rgba(255, 255, 255, 0.06) 0%,
          transparent 60%
        );
        opacity: 0;
        transition: all 0.32s;
      }
      &:hover:after {
        opacity: 1;
      }
    }
  }

  .wiki-body {
    position: relative;
    z-index: 10;
    margin-top: 12px;

    .empty {
      text-align: center;
      padding: 40px 16px;
      color: #ffb47b;
    }

    .entry-list {
      list-style: none;
      padding: 0;
      margin: 0;
      display: grid;
      gap: 16px;

      .entry-card {
        background: rgba(10, 5, 3, 0.5);
        backdrop-filter: blur(8px);
        border-radius: 12px;
        padding: 12px;
        box-shadow: 0 12px 28px $shadow-dark;
        transition: all 0.25s ease;
        border: 1px solid rgba(255, 140, 60, 0.15);
        overflow: hidden;

        &:hover {
          background: rgba(20, 10, 5, 0.6);
          border-color: rgba(255, 140, 60, 0.4);
          box-shadow: 0 18px 36px rgba(255, 100, 30, 0.2);
          transform: translateY(-4px);
        }

        .entry-head {
          display: flex;
          justify-content: space-between;
          align-items: flex-start;
          gap: 8px;
          flex-wrap: wrap;

          .entry-meta {
            cursor: pointer;

            .entry-title-wrap {
              display: flex;
              align-items: center;
              gap: 8px;
            }

            .entry-title {
              font-size: 16px;
              margin: 0;
              color: #ffb47b;
              font-weight: 700;
            }

            .entry-badge {
              display: inline-block;
              padding: 2px 8px;
              border-radius: 999px;
              background: rgba(255, 100, 30, 0.15);
              color: #ffb47b;
              font-size: 12px;
              border: 1px solid rgba(255, 140, 60, 0.3);
            }

            .entry-info {
              display: flex;
              flex-wrap: wrap;
              gap: 8px;
              margin-top: 6px;

              .meta-item {
                font-size: 12px;
                color: #ffd9b0;
                background: rgba(255, 100, 30, 0.1);
                border-radius: 6px;
                padding: 2px 8px;
                display: flex;
                align-items: center;
              }
            }
          }

          .entry-actions {
            display: flex;
            gap: 8px;
            align-items: center;
            flex-wrap: wrap;

            .like {
              background: rgba(30, 15, 8, 0.6);
              border: 1px solid rgba(255, 140, 60, 0.2);
              border-radius: 30px;
              padding: 4px 12px;
              display: flex;
              align-items: center;
              gap: 6px;
              cursor: pointer;
              transition: all 0.18s cubic-bezier(0.2, 0.9, 0.3, 1);

              img {
                width: 20px;
                height: 20px;
                filter: drop-shadow(0 1px 0 rgba(0, 0, 0, 0.3));
              }
              .like-count {
                font-size: 13px;
                color: #ffb47b;
              }
              &.active {
                transform: scale(1.05);
                border-color: $accent-2;
                box-shadow: 0 6px 14px rgba(255, 100, 30, 0.2);
              }
            }

            .edit-delete {
              display: flex;
              gap: 6px;
            }

            .small {
              padding: 8px 10px;
              border-radius: 8px;
              background: rgba(30, 15, 8, 0.6);
              border: 1px solid rgba(255, 140, 60, 0.2);
              color: #ffb47b;
              font-size: 13px;
              transition: all 0.3s;

              &:hover {
                background: rgba(50, 25, 15, 0.8);
                border-color: $accent-2;
              }
            }

            .danger {
              color: $danger;
              border-color: rgba(255, 100, 100, 0.3);
              &:hover {
                background: rgba(255, 80, 80, 0.1);
              }
            }
          }
        }
      }
    }
  }

  /* 弹窗 */
  .modal-overlay {
    position: fixed;
    inset: 0;
    background: rgba(10, 5, 3, 0.8);
    backdrop-filter: blur(12px);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 100;

    .modal {
      width: min(720px, 94%);
      max-height: 90vh;
      overflow-y: auto;
      background: rgba(10, 5, 3, 0.9);
      backdrop-filter: blur(20px);
      border-radius: 28px;
      padding: 20px;
      box-shadow: 0 30px 60px $shadow-dark;
      border: 1px solid rgba(255, 140, 60, 0.3);

      .modal-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding-bottom: 8px;
        border-bottom: 1px solid rgba(255, 140, 60, 0.2);

        h3 {
          margin: 0;
          color: #ffb47b;
          font-size: 1.3rem;
        }
        .close {
          background: transparent;
          border: none;
          font-size: 18px;
          color: #ffb47b;
          cursor: pointer;
          transition: all 0.3s;

          &:hover {
            transform: scale(1.2);
            color: $accent-2;
          }
        }
      }

      .modal-body {
        color: $text-main;
        font-size: 14px;
        line-height: 1.6;
        display: flex;
        flex-direction: column;
        gap: 12px;
        padding: 16px 0;

        .detail-content {
          white-space: pre-wrap;
        }

        label {
          display: flex;
          flex-direction: column;
          gap: 4px;
          color: #ffb47b;

          input,
          textarea {
            background: rgba(0, 0, 0, 0.4);
            color: $text-main;
            border: 1px solid rgba(255, 140, 60, 0.2);
            border-radius: 40px;
            padding: 10px 14px;
            outline: none;
            transition: all 0.3s;

            &:focus {
              border-color: $accent-2;
              box-shadow: 0 0 0 3px rgba(255, 100, 30, 0.2);
            }
          }

          textarea {
            border-radius: 20px;
            resize: vertical;
          }
        }
      }

      .modal-footer {
        display: flex;
        justify-content: flex-end;
        gap: 12px;
        margin-top: 12px;
        padding-top: 12px;
        border-top: 1px solid rgba(255, 140, 60, 0.2);

        .btn {
          background: linear-gradient(145deg, #ff974f, #ff4f1e);
          color: #1a0c06;
          padding: 10px 24px;
          border-radius: 40px;
          border: none;
          cursor: pointer;
          font-weight: 700;
          box-shadow: 0 10px 25px rgba(255, 80, 20, 0.4);
          transition: all 0.3s;

          &:hover {
            transform: translateY(-3px);
            box-shadow: 0 15px 35px rgba(255, 100, 30, 0.6);
          }
        }
        .btn.ghost {
          background: transparent;
          border: 1px solid rgba(255, 140, 60, 0.3);
          color: #ffb47b;
          box-shadow: none;

          &:hover {
            background: rgba(255, 100, 30, 0.1);
            transform: translateY(-2px);
          }
        }
      }
    }
  }

  /* 过渡动画 */
  .fade-zoom-enter-active,
  .fade-zoom-leave-active {
    transition: all 0.3s ease;
  }
  .fade-zoom-enter-from,
  .fade-zoom-leave-to {
    opacity: 0;
    transform: scale(0.96);
  }

  /* 移动端响应式 */
  @media (max-width: 720px) {
    .wiki-header {
      flex-direction: column;
      align-items: stretch;

      .actions {
        flex-direction: column;
      }
      .search {
        width: 100%;
      }
      .btn-new {
        width: 100%;
      }
    }

    .entry-list {
      display: flex;
      flex-direction: column;
    }

    .modal {
      width: 94%;
      max-height: 94vh;
    }

    .entry-actions {
      flex-direction: column;
      align-items: stretch;
      gap: 8px;
      margin-right: 0 !important;

      .like,
      .edit-delete {
        width: 100%;
        display: flex;
        align-items: center;
      }

      .like {
        justify-content: flex-start;
        padding: 8px 10px;
        img {
          margin-right: 8px;
        }
      }

      .edit-delete {
        flex-direction: column;
        gap: 8px;
      }

      .small {
        width: 100%;
        box-sizing: border-box;
        justify-content: center;
      }
    }
  }

  /* 动画定义 */
  @keyframes arc-drift {
    0% {
      transform: translateY(0) rotate(-6deg) translateX(0);
      opacity: 0.9;
      filter: blur(6px);
    }
    50% {
      transform: translateY(-6px) rotate(3deg) translateX(-6px);
      opacity: 1;
      filter: blur(4px) saturate(1.02);
    }
    100% {
      transform: translateY(0) rotate(-6deg) translateX(0);
      opacity: 0.9;
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
}
</style>
