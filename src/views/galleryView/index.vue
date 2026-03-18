<template>
  <div class="gallery-container">
    <!-- 装饰性背景元素（新增） -->
    <div class="bg-ornament"></div>
    <div class="bg-grid"></div>
   

    <!-- 上传按钮（样式优化） -->
    <button class="upload-btn" @click="openUploadModal">
      <span class="btn-icon">+</span>
      <span class="btn-text">上传图片</span>
    </button>

    <!-- 主内容区域：使用flex布局，左侧画廊右侧排行榜（桌面） -->
    <div class="gallery-layout">
      <section class="gallery section">
        <div class="section-header">
          <h2 class="section-title">焚愿画廊</h2>
          <div class="sort-controls">
            <button @click="toggleSort" class="sort-btn">
              <span class="sort-icon"></span>
              <span class="sort-text"
                >按
                {{ sortBy === "like_count" ? "点赞量" : "最新上传" }} 排序</span
              >
            </button>
          </div>
        </div>

        <div class="gallery-grid">
          <div
            v-for="(img, index) in images"
            :key="img.id"
            class="card"
            @click="openLightbox(index)"
            ref="cards"
          >
            <div class="card-inner">
              <img
                :src="img.src"
                :alt="img.alt"
                loading="lazy"
                @load="onImageLoad($event)"
              />
              <div class="overlay">
                <span>查看大图</span>
              </div>
              <button class="like-btn" @click.stop="handleLike(img)">
                <i class="heart" :class="{ liked: img.liked }"></i>
                <span class="like-count">{{ img.likeCount }}</span>
              </button>
            </div>
          </div>
        </div>

        <!-- sentinel：用于触发无限滚动 -->
        <div ref="sentinel" class="sentinel"></div>

        <!-- 加载状态 -->
        <div class="loading-indicator" v-if="loading">
          <span class="spinner"></span>
          <span>加载中...</span>
        </div>
        <div class="finished-message" v-if="finished">—— 已全部加载 ——</div>
      </section>

      <!-- 排行榜侧边栏（桌面固定，移动端作为底部抽屉） -->
      <aside class="ranking-panel" :class="{ expanded }">
        <div class="panel-header" @click="expanded = !expanded">
          <div class="header-left">
            <h3 class="ranking-title">排行榜</h3>
            <span class="total-badge">共{{ imgTotal }}张</span>
          </div>
          <button class="toggle-btn" :aria-label="expanded ? '收起' : '展开'">
            <span class="toggle-icon">{{ expanded ? "▾" : "▸" }}</span>
          </button>
        </div>

        <transition name="fade">
          <ul v-if="expanded" class="ranking-list">
            <li
              v-for="(item, idx) in rankingList"
              :key="idx"
              class="ranking-item"
              :class="`rank-${idx + 1}`"
            >
              <span class="rank">{{ idx + 1 }}</span>
              <span class="name">{{ item.nickname }}</span>
              <span class="count">{{ item.count }} 张</span>
            </li>
          </ul>
        </transition>
      </aside>
    </div>

    <!-- Lightbox Modal（保持原样，稍作样式调整） -->
    <div v-if="lightboxOpen" class="lightbox" @click.self="closeLightbox">
      <button class="close" @click="closeLightbox">✕</button>
      <button class="prev" @click.stop="prevImage">‹</button>
      <img :src="images[currentIndex].src" :alt="images[currentIndex].alt" />
      <button class="next" @click.stop="nextImage">›</button>
    </div>

    <!-- 上传弹窗（保持原样，稍作样式调整） -->
    <div
      v-if="uploadModalOpen"
      class="upload-modal-overlay"
      @click.self="closeUploadModal"
    >
      <div class="upload-modal">
        <h3>批量上传图片</h3>
        <div class="tip-container">
          <ul class="tips-list">
            <li>
              审核规则： 1.不要色情倾向（不要露三点，我怕被封）
              2.要我能认出是长离。
            </li>
            <li>
              由于没有用户系统，我这边不好做审核反馈，但只要显示上传成功，我这边肯定能收到。
            </li>
            <li>
              如果图片数量较多请在b站私信联系我给我网盘链接，因为我云服务器比较小一次性上传太多图片可能会导致上传不上，感谢理解。
            </li>
            <li>
              因为审核上传一次比较麻烦，所以审核时间不定，最晚一周，感谢谅解。
            </li>
          </ul>
        </div>
        <p class="stats">
          今日已上传：<strong>{{ uploadedToday }}</strong> 张，
          剩余可上传：<strong>{{ remaining }}</strong> 张
        </p>
        <label>
          昵称：
          <input v-model="nickname" type="text" placeholder="请输入昵称" />
        </label>
        <label>
          选择图片（最多 {{ remaining }} 张）：
          <input
            ref="fileInput"
            type="file"
            multiple
            accept="image/*"
            @change="handleFileSelect"
          />
        </label>
        <p class="tip" v-if="selectedFiles.length">
          已选 {{ selectedFiles.length }} 张
        </p>
        <div class="modal-actions">
          <button :disabled="!canSubmit || isUploading" @click="submitUpload">
            {{ isUploading ? "上传中..." : "立即上传" }}
          </button>
          <button class="cancel" @click="closeUploadModal">取消</button>
        </div>
      </div>
    </div>

    <!-- 浮动小人（保持原逻辑，位置样式微调） -->
    <div class="floating-chibis">
      <img
        v-for="(pet, i) in chibiList"
        :key="i"
        :src="pet.src"
        :style="{ top: pet.top + 'px', left: pet.left + 'px' }"
        class="chibi-img"
      />
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, computed, nextTick, onBeforeUnmount } from "vue";
import { uploadImages } from "@/api/modules/images"; // 前面封装的上传接口
import { getRankingList } from "@/api/modules/ranking"; // 根据你的实际路径调整
import { gsap } from "gsap"; // ← 本地引入
import { getImagesLikesList, likeImage } from "@/api/modules/imagesLikes";
import { debounce } from "lodash";

const sortBy = ref<"uploaded_at" | "like_count">("like_count");
const order = ref<"asc" | "desc">("desc");
function toggleSort() {
  if (sortBy.value === "uploaded_at") {
    sortBy.value = "like_count";
    order.value = "desc";
  } else {
    sortBy.value = "uploaded_at";
    order.value = "desc";
  }
  pageImage.value = 1;
  images.value = [];
  finished.value = false;
  window.scrollTo(0, 0);
  loadNextPage();
}
// 获取已点赞 ID 数组
function getLikedIds(): number[] {
  const data = localStorage.getItem("likedImageIds");
  return data ? JSON.parse(data) : [];
}

// 保存已点赞 ID 数组
function setLikedIds(ids: number[]) {
  localStorage.setItem("likedImageIds", JSON.stringify(ids));
}

async function handleLike(img: ImageItem) {
  if (img.liked) return; // 已点过就不重复调用

  try {
    await likeImage(img.id); // 调用后端接口
    img.likeCount += 1; // 本地更新点赞数
    img.liked = true; // 标记已点赞

    // 更新 localStorage
    const likedIds = getLikedIds();
    likedIds.push(img.id);
    setLikedIds(likedIds);
  } catch (error) {
    console.error("点赞失败", error);
    alert("点赞失败，请稍后重试");
  }
}

interface ImageItem {
  src: string;
  alt: string;
  likeCount: number;
  id: number;
  liked: Boolean;
}

interface RankingItem {
  id?: number; // 如果接口返回有 id，可加上
  nickname: string;
  count: number;
}
const rankingList = ref<RankingItem[]>([]);
const expanded = ref(true);

// 默认分页参数（如不分页可省略）
const page = 1;
const pageSize = 99;

const fetchRanking = async () => {
  const res = await getRankingList({
    page,
    pageSize,
    character_key: "changLi",
  });
  if (res.success) {
    rankingList.value = res.data;
  } else {
    console.error("获取排行榜失败", res.message);
  }
};

// 响应式存放最终图片列表
const images = ref<ImageItem[]>([]);

const pageImage = ref(1);
const limit = ref(10);
const loading = ref(false);
const finished = ref(false);

const sentinel = ref<HTMLElement | null>(null);

// 1. 在外层创建一个单例 observerCard
const observerCard = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        entry.target.classList.add("visible");
        observerCard.unobserve(entry.target);
      }
    });
  },
  { threshold: 0.1 }
);
// 2. 每次有新卡片时，都调用这个方法去挂载观察
async function observeNewCards(startIndex = 0) {
  await nextTick();
  const cards = document.querySelectorAll<HTMLElement>(".card");
  for (let i = startIndex; i < cards.length; i++) {
    observerCard.observe(cards[i]);
  }
}
const fixImageUrl = (url: string): string => {
  if (url.includes("127.0.0.1")) {
    // 将 127.0.0.1 替换为当前页面的完整源（协议+域名）
    return url.replace("http://127.0.0.1", window.location.origin);
  }
  return url;
};

const imgTotal = ref(0);
async function loadNextPage() {
  if (loading.value || finished.value) return;
  loading.value = true;
  try {
    const res = await getImagesLikesList({
      page: pageImage.value,
      limit: limit.value,
      sortBy: sortBy.value,
      character_key: "changLi",
      order: order.value,
    });
    imgTotal.value = res.total;
    const likedIds = getLikedIds();
    const list = (
      res.images as Array<{ url: string; like_count: number; id: number }>
    ).map((item) => ({
      src: fixImageUrl(item.url),
      alt: "",
      likeCount: item.like_count,
      id: item.id, // 如果需要的话，方便点赞用
      liked: likedIds.includes(item.id),
    }));
    if (list.length === 0) {
      finished.value = true;
      return;
    }
    // 记录加载前的长度，方便后面找出“新增”节点
    const oldLength = images.value.length;
    const existingIds = new Set(images.value.map((i) => i.id));
    const filtered = list.filter((item) => !existingIds.has(item.id));
    images.value.push(...filtered);
    pageImage.value++;

    observeNewCards(oldLength);
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
}

// 3. 给 loadNextPage 包装一个防抖版
const debouncedLoad = debounce(
  () => {
    loadNextPage();
  },
  200,
  { leading: true, trailing: false }
);

const lightboxOpen = ref(false);
const currentIndex = ref(0);

function openLightbox(index: number) {
  currentIndex.value = index;
  lightboxOpen.value = true;
}
function closeLightbox() {
  lightboxOpen.value = false;
}
function prevImage() {
  currentIndex.value =
    (currentIndex.value + images.value.length - 1) % images.value.length;
}
function nextImage() {
  currentIndex.value = (currentIndex.value + 1) % images.value.length;
}

// 渐显＆Blur‑Up 效果
function onImageLoad(e: Event) {
  const img = e.target as HTMLImageElement;
  const card = img.closest(".card");
  card?.classList.add("loaded");
}

// 上传弹窗逻辑

const uploadModalOpen = ref(false);
const nickname = ref("");
const fileInput = ref<HTMLInputElement>();
const selectedFiles = ref<File[]>([]);

// 从 localStorage 读取“今天”已上传数量
function getTodayKey() {
  return `uploaded_${new Date().toISOString().slice(0, 10)}`;
}
const uploadedToday = ref<number>(
  Number(localStorage.getItem(getTodayKey()) || 0)
);
const remaining = computed(() => Math.max(27 - uploadedToday.value, 0));

// 控制提交按钮
const canSubmit = computed(() => {
  return (
    nickname.value.trim().length > 0 &&
    selectedFiles.value.length > 0 &&
    selectedFiles.value.length <= remaining.value
  );
});

// 放在 script 顶部，或者 utils 里
function clearOldUploadRecords() {
  const today = new Date();
  const storage = window.localStorage;
  for (const key of Object.keys(storage)) {
    if (!key.startsWith("uploaded_")) continue;

    // key 格式 uploaded_YYYY-MM-DD
    const dateStr = key.slice("uploaded_".length);
    const recordDate = new Date(dateStr);
    if (isNaN(recordDate.getTime())) continue;

    // 计算相差天数
    const diffMs = today.getTime() - recordDate.getTime();
    const diffDays = diffMs / (1000 * 60 * 60 * 24);

    // 如果超过 2 天，就删掉
    if (diffDays > 2) {
      storage.removeItem(key);
    }
  }
}

function openUploadModal() {
  clearOldUploadRecords();
  nickname.value = "";
  selectedFiles.value = [];
  if (fileInput.value) fileInput.value.value = "";
  // 每次打开重新刷新已上传数
  uploadedToday.value = Number(localStorage.getItem(getTodayKey()) || 0);
  uploadModalOpen.value = true;
}
function closeUploadModal() {
  uploadModalOpen.value = false;
}

// 本地截断到剩余数量
function handleFileSelect(e: Event) {
  const files = Array.from((e.target as HTMLInputElement).files || []);

  if (!files) return;

  const validFiles: File[] = [];
  for (const file of files) {
    if (file.size > 20 * 1024 * 1024) {
      alert(`文件太大：${file.name}，请控制在 20MB 内`);
      continue;
    }
    validFiles.push(file);
  }

  if (validFiles.length === 0) return;

  if (validFiles.length > remaining.value) {
    alert(
      `今天最多还能上传 ${remaining.value} 张，已为你截取前 ${remaining.value} 张`
    );
    selectedFiles.value = files.slice(0, remaining.value);
  } else {
    selectedFiles.value = files;
  }
}
const isUploading = ref(false);
async function submitUpload() {
  if (!canSubmit.value) return;
  isUploading.value = true;
  try {
    const res = await uploadImages(
      selectedFiles.value,
      nickname.value.trim(),
      "changLi"
    );
    const uploadedCount = res.data.length;
    // 更新 localStorage
    uploadedToday.value += uploadedCount;
    localStorage.setItem(getTodayKey(), String(uploadedToday.value));

    alert(`成功上传 ${uploadedCount} 张图片`);
    closeUploadModal();
    // …可选：刷新画廊列表或把新图片追加到 images …
  } catch (err: any) {
    console.error(err);
    alert(err.message || "上传失败");
  } finally {
    isUploading.value = false;
  }
}

interface Chibi {
  src: string;
  top: number;
  left: number;
}

const chibiList = ref<Chibi[]>([]);
let sentinelObserver: IntersectionObserver;
// Scroll-triggered lazy animation
onMounted(async () => {
  // 1. 拉排行榜
  await fetchRanking();

  // 2. 拉第一页图片并挂载动画 observer
  await loadNextPage(); // 内部会调用 observeNewCards(oldLen)
  // 对首次卡片做一次完整 observe
  observeNewCards(0);

  // 3. 初始化 sentinelObserver，再 observe
  sentinelObserver = new IntersectionObserver(
    (entries) => {
      if (entries[0].isIntersecting) debouncedLoad();
    },
    { rootMargin: "0px", threshold: 0.1 }
  );
  if (sentinel.value) {
    sentinelObserver.observe(sentinel.value);
  }
  // 1. 基础配置信息
  const total = 5;
  let pickCount = 3; // 每次抽取 3 张
  const vw = window.innerWidth;
  const vh = window.innerHeight;
  const isMobile = window.innerWidth <= 768;
  // 如果已知单张小人图片的宽高，可避免超出边界；
  // 假设小人图片宽 100px、高 100px，按需替换：
  const imgWidth = 100;
  const imgHeight = 100;

  // 2. Fisher–Yates 洗牌函数
  function shuffle(array) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
  }

  // 3. 随机选出 3 个编号
  if (isMobile) {
    pickCount = 1;
  }
  const nums = shuffle(Array.from({ length: total }, (_, k) => k + 1));
  const picks = nums.slice(0, pickCount);

  // 4. 生成随机位置并填充 chibiList
  chibiList.value = []; // 先清空
  picks.forEach((i) => {
    chibiList.value.push({
      src: `/QImages/1 (${i}).png`,
      left: Math.random() * (vw - imgWidth), // 保证不超出左右边界
      top: Math.random() * (vh - imgHeight), // 保证不超出上下边界
    });
  });

  // 2. 等 img 渲染到 DOM
  await nextTick();

  // 3. 给每个小人绑定 GSAP 动画
  const imgs = document.querySelectorAll<HTMLImageElement>(".chibi-img");
  imgs.forEach((img, index) => {
    const padding = 200; // 边缘预留空间
    // ✅ 初始出场动画（闪现）
    gsap.fromTo(
      img,
      { opacity: 0, scale: 0.5 },
      {
        opacity: 1,
        scale: 1,
        duration: 0.8,
        ease: "back.out(2)",
        delay: 0.2 * index,
      }
    );

    // ✅ 鼠标靠近闪避
    img.addEventListener("mouseenter", () => {
      gsap.killTweensOf(img);

      gsap.to(img, {
        x: "+=" + ((Math.random() - 0.5) * 400).toFixed(0),
        y: "+=" + ((Math.random() - 0.5) * 400).toFixed(0),
        duration: 1.2,
        ease: "back.out(2)",
        onComplete: () => {
          // 闪避完成后，再重新启用动画
          animate(img);
        },
      });
    });

    const animate = (img: HTMLImageElement) => {
      let { x, y } = img.getBoundingClientRect();
      let deltaX = (Math.random() - 0.5) * 200;
      let deltaY = (Math.random() - 0.5) * 200;

      // 预测一下偏移后的位置
      let nextX = x + deltaX;
      let nextY = y + deltaY;

      // 校正：防漂出左、右、上、下边界
      if (nextX < padding) deltaX = padding - x;
      if (nextX + img.width > window.innerWidth - padding)
        deltaX = window.innerWidth - padding - (x + img.width);
      if (nextY < padding) deltaY = padding - y;
      if (nextY + img.height > window.innerHeight - padding)
        deltaY = window.innerHeight - padding - (y + img.height);

      gsap.to(img, {
        x: `+=${deltaX.toFixed(0)}`,
        y: `+=${deltaY.toFixed(0)}`,
        rotation: `+=${((Math.random() - 0.5) * 60).toFixed(0)}`,
        duration: 2 + Math.random() * 2,
        ease: "power1.inOut",
        onComplete: () => animate(img),
      });
    };
    animate(img);
  });
});

onBeforeUnmount(() => {
  observerCard.disconnect();
  sentinelObserver.disconnect();
  // 以及你在 onMounted 里新建的其它 Observer
});
</script>

<style lang="scss" scoped>
// 主题变量（统一为长离风格）
:root {
  --bg-deep: #0a0503;
  --accent: #ff4f1e;
  --accent-2: #ff974f;
  --accent-3: #ffcb9a;
  --text-light: #f7e4d4;
  --glass-bg: rgba(10, 5, 3, 0.7);
  --flame-glow: rgba(255, 80, 20, 0.3);
  --card-bg: rgba(15, 8, 4, 0.6);
  --border-glow: rgba(255, 140, 60, 0.2);
}

.gallery-container {
  position: relative;
  min-height: 100vh;
  background: linear-gradient(135deg, #1a0c06 0%, #0a0503 50%, #1f0e07 100%);
  color: var(--text-light);
  font-family: "Noto Sans SC", system-ui, -apple-system, sans-serif;
  overflow-x: hidden;

  // 装饰性背景元素
  .bg-ornament {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: radial-gradient(
        circle at 30% 30%,
        rgba(255, 140, 60, 0.05) 0%,
        transparent 50%
      ),
      radial-gradient(
        circle at 70% 70%,
        rgba(255, 80, 30, 0.05) 0%,
        transparent 50%
      );
    pointer-events: none;
    z-index: 0;
  }

  .bg-grid {
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



  // 上传按钮（优化）
  .upload-btn {
    position: fixed;
    bottom: 32px;
    left: 32px;
    z-index: 100;
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 12px 24px;
    background: linear-gradient(145deg, #ff974f, #ff4f1e);
    border: none;
    border-radius: 40px;
    color: #1a0c06;
    font-weight: 700;
    font-size: 1rem;
    cursor: pointer;
    box-shadow: 0 15px 30px rgba(255, 80, 20, 0.4),
      0 0 0 1px rgba(255, 140, 60, 0.3);
    transition: all 0.3s cubic-bezier(0.2, 0.9, 0.3, 1);

    .btn-icon {
      font-size: 1.4rem;
      line-height: 1;
    }

    &:hover {
      transform: translateY(-4px);
      box-shadow: 0 25px 40px rgba(255, 100, 30, 0.5),
        0 0 0 1px rgba(255, 140, 60, 0.5);
    }

    &:active {
      transform: translateY(0);
    }

    @media (max-width: 768px) {
      bottom: 20px;
      left: 20px;
      padding: 10px 20px;
      font-size: 0.9rem;
    }
  }

  // 主布局：flex 左右结构
  .gallery-layout {
    position: relative;
    z-index: 10;
    max-width: 1400px;
    margin: 0 auto;
    padding: 100px 24px 40px;
    display: flex;
    gap: 32px;

    @media (max-width: 1024px) {
      flex-direction: column;
      padding-top: 80px;
    }

    @media (max-width: 768px) {
      padding: 70px 16px 30px;
    }
  }

  // 画廊主区域
  .gallery {
    flex: 1;
    min-width: 0; // 防止溢出

    .section-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 28px;

      @media (max-width: 768px) {
        flex-direction: column;
        align-items: flex-start;
        gap: 12px;
      }
    }

    .section-title {
      font-size: 2rem;
      font-weight: 700;
      background: linear-gradient(135deg, #ffdbb0, #ffb47b, #ff6324);
      -webkit-background-clip: text;
      background-clip: text;
      color: transparent;
      margin: 0;
      letter-spacing: 2px;

      @media (max-width: 768px) {
        font-size: 1.6rem;
      }
    }

    .sort-controls {
      .sort-btn {
        display: flex;
        align-items: center;
        gap: 10px;
        padding: 10px 24px;
        background: rgba(10, 5, 3, 0.7);
        backdrop-filter: blur(10px);
        border: 1px solid rgba(255, 140, 60, 0.2);
        border-radius: 40px;
        color: #ffb890;
        font-size: 0.95rem;
        cursor: pointer;
        transition: all 0.3s;

        .sort-icon {
          width: 20px;
          height: 20px;
          background: radial-gradient(circle, #ff974f, #ff4f1e);
          border-radius: 50%;
          filter: blur(4px);
        }

        &:hover {
          background: rgba(20, 10, 5, 0.8);
          border-color: rgba(255, 140, 60, 0.4);
          transform: translateY(-2px);
        }
      }
    }
  }

  // 画廊网格
  .gallery-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 24px;
    margin-bottom: 32px;

    @media (max-width: 768px) {
      gap: 16px;
    }

    .card {
      perspective: 1200px;
      opacity: 0;
      transform: translateY(20px);
      transition: opacity 0.6s ease, transform 0.6s ease;

      &.visible {
        opacity: 1;
        transform: translateY(0);
      }

      .card-inner {
        position: relative;
        border-radius: 20px;
        overflow: hidden;
        background: var(--card-bg);
        backdrop-filter: blur(8px);
        border: 1px solid rgba(255, 140, 60, 0.15);
        box-shadow: 0 20px 40px -10px rgba(0, 0, 0, 0.6);
        transition: all 0.4s cubic-bezier(0.2, 0.9, 0.3, 1);

        &:hover {
          transform: translateY(-6px) rotateX(2deg);
          box-shadow: 0 30px 60px -10px rgba(255, 100, 30, 0.3);
          border-color: rgba(255, 140, 60, 0.4);

          .overlay {
            opacity: 1;
          }
        }

        img {
          width: 100%;
          height: 260px;
          object-fit: cover;
          display: block;
          filter: blur(0) grayscale(0);
          transition: transform 0.5s;
        }

        .overlay {
          position: absolute;
          bottom: 0;
          left: 0;
          width: 100%;
          padding: 16px 0;
          background: linear-gradient(to top, rgba(0, 0, 0, 0.8), transparent);
          text-align: center;
          opacity: 0;
          transition: opacity 0.3s;

          span {
            display: inline-block;
            padding: 6px 16px;
            background: rgba(255, 100, 30, 0.2);
            backdrop-filter: blur(4px);
            border: 1px solid rgba(255, 140, 60, 0.3);
            border-radius: 30px;
            color: #ffd9b0;
            font-size: 0.9rem;
            letter-spacing: 1px;
          }
        }

        .like-btn {
          position: absolute;
          bottom: 16px;
          right: 16px;
          display: flex;
          align-items: center;
          gap: 6px;
          background: rgba(10, 5, 3, 0.6);
          backdrop-filter: blur(4px);
          border: 1px solid rgba(255, 140, 60, 0.3);
          border-radius: 30px;
          padding: 6px 12px;
          cursor: pointer;
          transition: all 0.3s;

          &:hover {
            background: rgba(20, 10, 5, 0.8);
            transform: scale(1.05);
          }

          .heart {
            width: 20px;
            height: 20px;
            background: url("/icons/heart-red-outline.svg") no-repeat center;
            background-size: contain;
            filter: drop-shadow(0 0 4px rgba(255, 0, 0, 0.5));

            &.liked {
              background: url("/icons/heart-red-filled.svg") no-repeat center;
              animation: pop 0.3s ease;
            }
          }

          .like-count {
            color: #ffb47b;
            font-weight: 700;
            font-size: 0.9rem;
          }
        }
      }
    }
  }

  // 加载指示器
  .loading-indicator {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 12px;
    padding: 20px;
    color: #ffb47b;

    .spinner {
      width: 24px;
      height: 24px;
      border: 2px solid rgba(255, 140, 60, 0.3);
      border-top-color: #ff974f;
      border-radius: 50%;
      animation: spin 1s linear infinite;
    }
  }

  .finished-message {
    text-align: center;
    padding: 20px;
    color: #b99f8b;
    font-style: italic;
  }

  // 排行榜侧边栏
  .ranking-panel {
    width: 280px;
    flex-shrink: 0;
    background: rgba(10, 5, 3, 0.7);
    backdrop-filter: blur(16px);
    border: 1px solid rgba(255, 140, 60, 0.2);
    border-radius: 28px;
    padding: 20px;
    height: fit-content;
    transition: all 0.3s;

    @media (max-width: 1024px) {
      width: 100%;
      margin-top: 20px;
    }

    .panel-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      cursor: pointer;
      padding-bottom: 12px;
      border-bottom: 1px solid rgba(255, 140, 60, 0.2);

      .header-left {
        display: flex;
        align-items: center;
        gap: 12px;
      }

      .ranking-title {
        font-size: 1.2rem;
        font-weight: 700;
        color: #ffb890;
        margin: 0;
      }

      .total-badge {
        font-size: 0.8rem;
        color: #b99f8b;
        background: rgba(0, 0, 0, 0.3);
        padding: 2px 8px;
        border-radius: 20px;
      }

      .toggle-btn {
        background: rgba(30, 15, 8, 0.5);
        border: 1px solid rgba(255, 140, 60, 0.2);
        border-radius: 30px;
        padding: 4px 12px;
        color: #ffb47b;
        cursor: pointer;
        transition: all 0.3s;

        &:hover {
          background: rgba(50, 25, 15, 0.7);
          border-color: rgba(255, 140, 60, 0.4);
        }
      }
    }

    .ranking-list {
      list-style: none;
      padding: 0;
      margin: 16px 0 0;
      max-height: 400px;
      overflow-y: auto;

      &::-webkit-scrollbar {
        width: 6px;
      }
      &::-webkit-scrollbar-track {
        background: rgba(255, 140, 60, 0.1);
        border-radius: 10px;
      }
      &::-webkit-scrollbar-thumb {
        background: linear-gradient(180deg, #ff974f, #ff4f1e);
        border-radius: 10px;
      }
    }

    .ranking-item {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 12px 16px;
      margin-bottom: 8px;
      border-radius: 40px;
      background: rgba(0, 0, 0, 0.2);
      border: 1px solid transparent;
      transition: all 0.3s;

      &:hover {
        background: rgba(30, 15, 8, 0.5);
        border-color: rgba(255, 140, 60, 0.2);
        transform: translateX(-4px);
      }

      .rank {
        width: 30px;
        font-weight: 800;
        color: #ffb47b;
      }

      .name {
        flex: 1;
        font-size: 0.95rem;
        color: #ffd9b0;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
        padding: 0 8px;
      }

      .count {
        font-size: 0.9rem;
        color: #ffb47b;
        font-weight: 600;
      }

      // 前三名特殊样式
      &.rank-1 {
        background: linear-gradient(
          90deg,
          rgba(255, 140, 60, 0.2),
          transparent
        );
        border-color: #ff974f;

        .rank,
        .name,
        .count {
          color: #ffd9b0;
          font-weight: 700;
        }
      }

      &.rank-2 {
        background: linear-gradient(
          90deg,
          rgba(255, 180, 100, 0.15),
          transparent
        );
        border-color: #ffb47b;
      }

      &.rank-3 {
        background: linear-gradient(
          90deg,
          rgba(255, 160, 80, 0.1),
          transparent
        );
        border-color: #ff9a66;
      }
    }

    // 移动端排行榜自动收起
    @media (max-width: 1024px) {
      &.expanded .ranking-list {
        display: block;
      }
      &:not(.expanded) .ranking-list {
        display: none;
      }
    }
  }

  // Lightbox 优化
  .lightbox {
    position: fixed;
    inset: 0;
    z-index: 2000;
    background: rgba(0, 0, 0, 0.95);
    display: flex;
    align-items: center;
    justify-content: center;

    img {
      max-width: 85%;
      max-height: 85%;
      border-radius: 20px;
      border: 2px solid rgba(255, 140, 60, 0.3);
      box-shadow: 0 30px 60px rgba(0, 0, 0, 0.8);
    }

    button {
      position: absolute;
      width: 48px;
      height: 48px;
      background: rgba(20, 10, 5, 0.7);
      backdrop-filter: blur(8px);
      border: 1px solid rgba(255, 140, 60, 0.3);
      border-radius: 50%;
      color: #ffb890;
      font-size: 2rem;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      transition: all 0.3s;

      &:hover {
        background: rgba(40, 20, 10, 0.9);
        border-color: #ff974f;
        transform: scale(1.1);
      }

      &.close {
        top: 20px;
        right: 20px;
      }

      &.prev {
        left: 20px;
        top: 50%;
        transform: translateY(-50%);
      }

      &.next {
        right: 20px;
        top: 50%;
        transform: translateY(-50%);
      }
    }
  }

  // 上传弹窗优化
  .upload-modal-overlay {
    position: fixed;
    inset: 0;
    z-index: 2500;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(10, 5, 3, 0.8);
    backdrop-filter: blur(12px);
  }

  .upload-modal {
    width: 600px;
    max-width: 90%;
    max-height: 85vh;
    overflow-y: auto;
    background: rgba(15, 8, 4, 0.9);
    backdrop-filter: blur(20px);
    border: 1px solid rgba(255, 140, 60, 0.3);
    border-radius: 32px;
    padding: 32px;
    box-shadow: 0 40px 80px rgba(0, 0, 0, 0.8);
    color: var(--text-light);

    h3 {
      margin: 0 0 24px;
      font-size: 1.8rem;
      background: linear-gradient(135deg, #ffdbb0, #ffb47b);
      -webkit-background-clip: text;
      background-clip: text;
      color: transparent;
      text-align: center;
    }

    .tip-container {
      background: rgba(0, 0, 0, 0.3);
      border-left: 4px solid #ff974f;
      border-radius: 12px;
      padding: 16px 20px;
      margin-bottom: 20px;

      .tips-list {
        list-style: none;
        padding: 0;
        margin: 0;

        li {
          position: relative;
          padding-left: 24px;
          margin-bottom: 12px;
          font-size: 0.95rem;
          color: #ecdacb;

          &::before {
            content: "";
            position: absolute;
            left: 0;
            top: 8px;
            width: 8px;
            height: 8px;
            background: #ff974f;
            border-radius: 50%;
            box-shadow: 0 0 10px #ff974f;
          }
        }
      }
    }

    .stats {
      text-align: center;
      font-size: 1rem;
      margin: 20px 0;
      color: #ffb47b;
      strong {
        color: #ffb47b;
        font-size: 1.2rem;
      }
    }

    label {
      display: block;
      margin-bottom: 20px;
      font-size: 0.95rem;
      color: #ffd9b0;

      input {
        width: 100%;
        margin-top: 8px;
        padding: 12px 16px;
        background: rgba(0, 0, 0, 0.4);
        border: 1px solid rgba(255, 140, 60, 0.2);
        border-radius: 30px;
        color: var(--text-light);
        font-size: 1rem;
        transition: all 0.3s;

        &:focus {
          outline: none;
          border-color: #ff974f;
          box-shadow: 0 0 0 3px rgba(255, 100, 30, 0.2);
        }
      }
    }

    .modal-actions {
      display: flex;
      justify-content: flex-end;
      gap: 16px;
      margin-top: 32px;

      button {
        padding: 12px 32px;
        border: none;
        border-radius: 40px;
        font-weight: 700;
        cursor: pointer;
        transition: all 0.3s;

        &:not(.cancel) {
          background: linear-gradient(145deg, #ff974f, #ff4f1e);
          color: #1a0c06;
          box-shadow: 0 15px 30px rgba(255, 80, 20, 0.3);

          &:hover:not(:disabled) {
            transform: translateY(-3px);
            box-shadow: 0 20px 40px rgba(255, 100, 30, 0.4);
          }

          &:disabled {
            opacity: 0.5;
            animation: cursorAnimation_disabled 1s infinite step-start;
          }
        }

        &.cancel {
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

  // 浮动小人（位置微调）
  .floating-chibis {
    position: fixed;
    inset: 0;
    pointer-events: none;
    z-index: 99;

    .chibi-img {
      position: absolute;
      width: 80px;
      height: auto;
      pointer-events: auto;
      filter: drop-shadow(0 10px 20px rgba(0, 0, 0, 0.5));
      transition: transform 0.2s;
      z-index: 5;

      &:hover {
        transform: scale(1.1) rotate(5deg);
      }
    }
  }
}

// 动画定义
@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

@keyframes pop {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.3);
  }
  100% {
    transform: scale(1);
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
