<template>
  <section
    class="changli-player"
    @keydown.space.prevent="onSpace"
    tabindex="0"
    ref="rootEl"
    aria-label="长离·丹煌离火 音乐播放器"
  >
    <!-- 装饰层：火焰粒子、翎羽、棋局背景 -->
    <div class="bg-particles">
      <span
        v-for="i in 48"
        :key="i"
        class="particle"
        :style="particleStyle(i)"
      ></span>
    </div>
    <div class="board-bg"></div>
    <div class="feather-decoration left"></div>
    <div class="feather-decoration right"></div>
    <div class="page-glow"></div>

    <div class="stage">
      <!-- 左侧：封面与控制 -->
      <div class="left" role="region" aria-label="播放器控制区">
        <div class="cover" :class="videoClass" :style="coverStyle">
          <video
            v-if="videoSrc"
            class="video-background"
            :src="videoSrc"
            autoplay
            muted
            loop
            playsinline
            aria-hidden="true"
            tabindex="-1"
          ></video>
          <div class="flame-overlay"></div>

          <div v-if="loadingAudio" class="loading-overlay" aria-hidden="true">
            <div class="spinner" />
            <div class="loading-text">余焰重燃···</div>
          </div>
        </div>

        <div class="controls">
          <div class="title" :title="current?.title || '未选择曲目'">
            {{ current?.title || "未选择曲目" }}
          </div>

          <div class="meta">
            <span class="time">{{ formatTime(currentTime) }}</span>
            <span class="divider">/</span>
            <span class="time">{{ formatTime(duration) }}</span>
          </div>

          <!-- 进度条（优化） -->
          <div
            class="progress-wrap"
            ref="progressWrap"
            @click="seekByClick"
            @pointerdown.prevent="onPointerDownProgress"
            role="slider"
            :aria-valuemin="0"
            :aria-valuemax="duration"
            :aria-valuenow="currentTime"
            aria-label="进度条"
          >
            <div class="progress-bar">
              <div
                class="progress"
                :style="{ width: progressPercent + '%' }"
              ></div>
            </div>
            <div
              class="progress-handle"
              :style="{ left: progressPercent + '%' }"
              aria-hidden="true"
            ></div>
          </div>

          <!-- 控件行 -->
          <div class="btns">
            <button class="icon" @click="prev" aria-label="上一首">⟵</button>

            <button
              class="play"
              @click="togglePlay"
              :aria-pressed="playing"
              :aria-label="playing ? '暂停' : '播放'"
            >
              <span v-if="!playing">▶</span>
              <span v-else>▌▌</span>
            </button>

            <button class="icon" @click="next" aria-label="下一首">⟶</button>

            <div class="modes" role="group" aria-label="播放模式">
              <button
                :class="{ active: shuffle }"
                @click="toggleShuffle"
                title="随机"
              >
                🔀
              </button>
              <button
                :class="{ active: repeatMode !== 'off' }"
                @click="toggleRepeat"
                title="循环"
              >
                🔁
              </button>
            </div>

            <div class="volume" aria-label="音量控制">
              <input
                type="range"
                min="0"
                max="1"
                step="0.01"
                v-model.number="volume"
                aria-label="音量"
              />
            </div>
          </div>

          <div v-if="errorMessage" class="error-msg" role="status">
            {{ errorMessage }}
          </div>
        </div>
      </div>

      <!-- 右侧：播放列表（优化布局） -->
      <div
        class="right"
        :class="{ collapsed: !playlistOpen && isMobile }"
        role="region"
        aria-label="播放列表"
      >
        <div class="playlist-header">
          <div class="left-head">
            <h3>焚音曲簿</h3>
            <button
              class="toggle-list-text"
              @click="togglePlaylist"
              :title="playlistOpen ? '收起' : '展开'"
            >
              {{ playlistOpen ? "收起" : "展开" }}
            </button>
            <div class="api-hint">
              {{ loading ? "加载中…" : list.length ? "" : "目录为空" }}
            </div>
          </div>

          <div class="search-wrap">
            <input
              v-model="searchTerm"
              @input="onSearchInput"
              placeholder="搜索曲名..."
              aria-label="搜索曲目"
            />
            <button
              v-if="searchTerm"
              class="clear"
              @click="clearSearch"
              aria-label="清除搜索"
            >
              ✕
            </button>
          </div>
        </div>

        <div class="list-area">
          <div v-if="loading" class="list-loading">
            <div class="small-spinner" />
            加载曲簿...
          </div>

          <ul class="playlist" role="list">
            <li
              v-for="(item, idx) in filteredList"
              :key="item.name || idx"
              :class="{ active: current && item.name === current.name }"
              @click="selectTrack(idx)"
              tabindex="0"
              @keyup.enter="selectTrack(idx)"
              role="listitem"
              :aria-current="idx === index ? 'true' : 'false'"
            >
              <div class="left-col">
                <div
                  class="flame-dot"
                  :class="'level-' + ((idx % 4) + 1)"
                ></div>
                <div class="title" :title="item.title">{{ item.title }}</div>
              </div>
              <div class="right-col">
                <div class="len">
                  {{ item.duration ? formatTime(item.duration) : "--:--" }}
                </div>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>

    <!-- audio 元素 -->
    <audio
      ref="audioRef"
      @timeupdate="onTimeUpdate"
      @ended="onEnded"
      @loadedmetadata="onLoadedMetadata"
      @error="onAudioError"
      preload="metadata"
    ></audio>
  </section>
</template>

<script setup lang="ts">
import {
  ref,
  computed,
  onMounted,
  onBeforeUnmount,
  watch,
  nextTick,
} from "vue";
import { getMusicList, getMusicUrl } from "@/api/modules/music"; // 请确认路径

// 粒子样式生成函数（与留言板一致）
function particleStyle(index: number) {
  const left = Math.random() * 100;
  const width = 2 + Math.random() * 10;
  const height = 15 + Math.random() * 40;
  const duration = 6 + Math.random() * 20;
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
type MusicItem = {
  name: string;
  title: string;
  url?: string;
  duration?: number | null;
};

const list = ref<MusicItem[]>([]);
const loading = ref(false);
const index = ref<number>(-1);
const playing = ref(false);
const audioRef = ref<HTMLAudioElement | null>(null);
const currentTime = ref<number>(0);
const duration = ref<number>(0);
const volume = ref<number>(
  Number(localStorage.getItem("cantarella_volume") ?? 0.8)
);
const shuffle = ref<boolean>(false);
const repeatMode = ref<"off" | "one" | "all">("off");

const rootEl = ref<HTMLElement | null>(null);
const progressWrap = ref<HTMLElement | null>(null);
const dragging = ref(false);
const playlistOpen = ref(true);
const errorMessage = ref<string | null>(null);
const loadingAudio = ref(false);

// 根据窗口宽度判断移动端
const isMobile = ref<boolean>(window.innerWidth <= 920);
window.addEventListener("resize", () => {
  isMobile.value = window.innerWidth <= 920;
});

// 随机选择视频源（mp1/mp2 里）
const videoSrc = ref("");
const videoClass = ref(""); // "landscape" or "portrait"

// 搜索相关
const searchTerm = ref("");
let searchTimer: any = null;
const searchDebounceMs = 240;

// 当前曲目与进度计算
const current = computed(() =>
  index.value >= 0 && list.value[index.value] ? list.value[index.value] : null
);
const progressPercent = computed(() =>
  duration.value
    ? Math.min(100, Math.max(0, (currentTime.value / duration.value) * 100))
    : 0
);

// 封面渐变（长离风格）
const coverStyle = computed(() => {
  const t = current.value?.title || "cantarella";
  let hash = 0;
  for (let i = 0; i < t.length; i++)
    hash = (hash << 5) - hash + t.charCodeAt(i);
  const r1 = (Math.abs(hash) % 120) + 40;
  const r2 = (Math.abs(hash * 3) % 120) + 40;
  return {
    background: `radial-gradient(circle at 28% 28%, rgba(95,224,255,0.06), transparent 12%), linear-gradient(135deg, rgba(${r1},${r2},255,0.08), rgba(111,92,230,0.08))`,
  };
});

// 过滤后的列表（基于搜索）
const filteredList = computed(() => {
  const term = (searchTerm.value || "").trim().toLowerCase();
  if (!term) return list.value;
  return list.value.filter((i) => (i.title || "").toLowerCase().includes(term));
});

// 获取列表
async function fetchList() {
  loading.value = true;
  try {
    const res = await getMusicList();
    const items =
      res?.ok && Array.isArray(res.list)
        ? res.list
        : Array.isArray(res)
        ? res
        : res?.list ?? [];
    list.value = items.map((it: any) => ({
      name: it.name,
      title: it.title ?? (it.name ? it.name.replace(/\.mp3$/i, "") : "未知"),
      url: getMusicUrl(it.name),
      duration: null,
    }));
  } catch (e) {
    console.error("获取音乐列表失败", e);
    list.value = [];
    errorMessage.value = "加载目录失败";
  } finally {
    loading.value = false;
  }
}

// 设置并预检音源
async function safeSetSrc(url: string) {
  const a = audioRef.value!;
  errorMessage.value = null;
  loadingAudio.value = true;
  try {
    try {
      const head = await fetch(url, { method: "HEAD" });
      if (!head.ok) throw new Error(`资源响应 ${head.status}`);
      const ct = head.headers.get("content-type") || "";
      if (!ct.includes("audio")) {
        console.warn("content-type 不是 audio:", ct);
      }
    } catch (e) {
      // 忽略 HEAD 失败
    }
    a.src = url;
    a.load();
  } catch (err) {
    console.error("设置音源失败", err);
    errorMessage.value = "无法加载音频资源";
    throw err;
  } finally {
    // loadingAudio 会在 loadedmetadata 或 error 中被关闭
  }
}

async function loadCurrent(doPlay = false) {
  const a = audioRef.value;
  const curr = current.value;
  if (!a || !curr) return;
  a.pause();
  duration.value = 0;
  currentTime.value = 0;
  try {
    await safeSetSrc(curr.url || getMusicUrl(curr.name));
    if (doPlay) await play();
  } catch {
    playing.value = false;
    loadingAudio.value = false;
  }
}

async function play() {
  const a = audioRef.value;
  if (!a) return;
  try {
    await a.play();
    playing.value = true;
    errorMessage.value = null;
  } catch (e: any) {
    console.warn("播放失败", e);
    playing.value = false;
    errorMessage.value = "播放被浏览器阻止或资源不可用";
  }
}
function pause() {
  audioRef.value?.pause();
  playing.value = false;
}
function togglePlay() {
  if (!audioRef.value) return;
  if (playing.value) pause();
  else play();
}
function selectTrack(idxInFiltered: number) {
  const item = filteredList.value[idxInFiltered];
  if (!item) return;
  // 用唯一字段（如 name）在原始列表中找到正确索引
  const originalIndex = list.value.findIndex((it) => it.name === item.name);
  if (originalIndex === -1) return;
  index.value = originalIndex;
  loadCurrent(true);
}

// 音频事件
function onTimeUpdate(e: Event) {
  const t = e.target as HTMLAudioElement;
  currentTime.value = t.currentTime || 0;
}
function onLoadedMetadata(e: Event) {
  const t = e.target as HTMLAudioElement;
  duration.value = isFinite(t.duration) ? t.duration : 0;
  if (current.value && !current.value.duration)
    current.value.duration = duration.value;
  loadingAudio.value = false;
}
function onEnded() {
  loadingAudio.value = false;
  if (repeatMode.value === "one") {
    if (audioRef.value) {
      audioRef.value.currentTime = 0;
      play();
    }
    return;
  }
  if (shuffle.value) {
    playRandom();
    return;
  }
  if (index.value < list.value.length - 1) selectTrack(index.value + 1);
  else {
    if (repeatMode.value === "all") selectTrack(0);
    else playing.value = false;
  }
}
function onAudioError(e: Event) {
  const a = audioRef.value;
  console.error("audio error", a?.error);
  errorMessage.value = "音频播放出错";
  playing.value = false;
  loadingAudio.value = false;
}

// 上一/下一/随机
function next() {
  if (!list.value.length) return;
  if (shuffle.value) {
    playRandom();
    return;
  }
  if (index.value < list.value.length - 1) selectTrack(index.value + 1);
  else if (repeatMode.value === "all") selectTrack(0);
}
function prev() {
  if (!audioRef.value) return;
  if (audioRef.value.currentTime > 4) {
    audioRef.value.currentTime = 0;
    return;
  }
  if (index.value > 0) selectTrack(index.value - 1);
  else if (repeatMode.value === "all") selectTrack(list.value.length - 1);
}
function playRandom() {
  if (!list.value.length) return;
  if (list.value.length === 1) {
    selectTrack(0);
    return;
  }
  let i = index.value;
  while (i === index.value) i = Math.floor(Math.random() * list.value.length);
  selectTrack(i);
}

// 进度条：点击 & 拖拽
function seekByClick(e: MouseEvent | TouchEvent) {
  if (!progressWrap.value || !duration.value || !audioRef.value) return;
  const rect = progressWrap.value.getBoundingClientRect();
  const clientX =
    (e as MouseEvent).clientX ?? (e as TouchEvent).touches?.[0]?.clientX;
  if (clientX == null) return;
  const x = Math.min(Math.max(0, clientX - rect.left), rect.width);
  const ratio = x / rect.width;
  audioRef.value.currentTime = ratio * duration.value;
  currentTime.value = audioRef.value.currentTime;
}
function onPointerDownProgress(e: PointerEvent) {
  if (!progressWrap.value || !audioRef.value || !duration.value) return;
  dragging.value = true;
  (e.target as Element).setPointerCapture?.(e.pointerId);
  window.addEventListener("pointermove", onPointerMoveProgress);
  window.addEventListener("pointerup", onPointerUpProgress);
  handlePointer(e);
}
function onPointerMoveProgress(e: PointerEvent) {
  handlePointer(e);
}
function onPointerUpProgress(e: PointerEvent) {
  dragging.value = false;
  window.removeEventListener("pointermove", onPointerMoveProgress);
  window.removeEventListener("pointerup", onPointerUpProgress);
}
function handlePointer(e: PointerEvent) {
  if (!progressWrap.value || !audioRef.value || !duration.value) return;
  const rect = progressWrap.value.getBoundingClientRect();
  const x = Math.min(Math.max(0, e.clientX - rect.left), rect.width);
  const ratio = x / rect.width;
  audioRef.value.currentTime = ratio * duration.value;
  currentTime.value = audioRef.value.currentTime;
}

// 音量持久化
watch(volume, (v) => {
  if (audioRef.value) audioRef.value.volume = v;
  localStorage.setItem("cantarella_volume", String(v));
});

// 模式切换
function toggleShuffle() {
  shuffle.value = !shuffle.value;
}
function toggleRepeat() {
  if (repeatMode.value === "off") repeatMode.value = "all";
  else if (repeatMode.value === "all") repeatMode.value = "one";
  else repeatMode.value = "off";
}

// 键盘空格
function onSpace() {
  if (
    document.activeElement &&
    ["INPUT", "TEXTAREA"].includes(document.activeElement.tagName)
  )
    return;
  togglePlay();
}

// 切换播放列表（文字）
function togglePlaylist() {
  playlistOpen.value = !playlistOpen.value;
  if (playlistOpen.value)
    nextTick(() => {
      const el = document.querySelector(".playlist li.active");
      el?.scrollIntoView({ block: "center", behavior: "smooth" });
    });
}

// 搜索防抖
function onSearchInput() {
  if (searchTimer) clearTimeout(searchTimer);
  searchTimer = setTimeout(() => {
    // 过滤由 computed filteredList 完成
    searchTimer = null;
  }, searchDebounceMs);
}
function clearSearch() {
  searchTerm.value = "";
}

// 格式化时间
function formatTime(sec?: number) {
  if (!sec || !isFinite(sec)) return "--:--";
  const s = Math.floor(sec % 60);
  const m = Math.floor(sec / 60);
  return `${String(m).padStart(2, "0")}:${String(s).padStart(2, "0")}`;
}

// 生命周期：绑定 audio ref、初始化列表和 videoSrc
onMounted(async () => {
  audioRef.value =
    (document.querySelector(".cantarella-player audio") as HTMLAudioElement) ??
    null;
  if (audioRef.value) audioRef.value.volume = volume.value;

  // 设置 videoSrc 与 class：桌面优先横屏(mp1), 移动优先竖屏(mp2)
  const isM = isMobile.value;
  const folder = isM ? "/mp1" : "/mp2";
  // 随机选择 1..4，如果没有资源请保证路径存在
  const idx = Math.floor(Math.random() * 4) + 1;
  videoSrc.value = `${folder}/1 (${idx}).mp4`;
  videoClass.value = isM ? "landscape" : "portrait";

  await fetchList();

  window.addEventListener("keydown", globalKeydown);
});

onBeforeUnmount(() => {
  window.removeEventListener("keydown", globalKeydown);
});

// 全局键盘
function globalKeydown(e: KeyboardEvent) {
  if (e.code === "Space") {
    if (
      document.activeElement &&
      ["INPUT", "TEXTAREA"].includes(document.activeElement.tagName)
    )
      return;
    e.preventDefault();
    togglePlay();
  } else if (e.code === "Escape") {
    pause();
  }
}
</script>

<style scoped lang="scss">
/* ========== 长离主题色系 ========== */

.changli-player {
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
  padding: 20px;
  padding-top: 80px;
  background: linear-gradient(135deg, #1a0c06 0%, #0a0503 50%, #1f0e07 100%);
  color: var(--text-light);
  font-family: "Noto Sans SC", system-ui, -apple-system, sans-serif;
  outline: none;
  overflow-x: hidden;

  /* ========== 装饰层 ========== */
  .bg-particles {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    z-index: 1;

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
    z-index: 1;
  }

  .feather-decoration {
    position: fixed;
    width: 250px;
    height: 500px;
    background: radial-gradient(
      ellipse at 50% 50%,
      rgba(255, 140, 60, 0.06),
      transparent 70%
    );
    filter: blur(50px);
    pointer-events: none;
    z-index: 1;
    animation: feather-float 16s infinite alternate;

    &.left {
      left: -120px;
      top: 10%;
      transform: rotate(15deg);
    }
    &.right {
      right: -120px;
      bottom: 10%;
      transform: rotate(-15deg);
    }
  }

  .page-glow {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 4px;
    background: linear-gradient(
      90deg,
      transparent,
      var(--accent),
      var(--accent-2),
      var(--accent),
      transparent
    );
    filter: blur(6px);
    opacity: 0.5;
    animation: glow-pulse 4s infinite alternate;
    z-index: 2;
    pointer-events: none;
  }

  /* ========== 主布局 ========== */
  .stage {
    display: flex;
    gap: 24px;
    max-width: 1200px;
    margin: 0 auto;
    align-items: flex-start;
    position: relative;
    z-index: 10;

    @media (max-width: 1024px) {
      flex-direction: column;
    }
  }

  /* ========== 左侧播放器 ========== */
  .left {
    width: 420px;
    background: rgba(10, 5, 3, 0.6);
    backdrop-filter: blur(12px);
    border: 1px solid rgba(255, 140, 60, 0.2);
    border-radius: 32px;
    padding: 24px;
    box-shadow: 0 25px 50px -10px rgba(0, 0, 0, 0.6);
    transition: transform 0.3s, box-shadow 0.3s;

    &:hover {
      box-shadow: 0 30px 60px -10px rgba(255, 100, 30, 0.2);
      border-color: rgba(255, 140, 60, 0.4);
    }

    @media (max-width: 1024px) {
      width: 100%;
    }
  }

  .cover {
    width: 100%;
    border-radius: 24px;
    overflow: hidden;
    position: relative;
    box-shadow: 0 20px 40px -10px rgba(0, 0, 0, 0.7),
      0 0 0 1px rgba(255, 140, 60, 0.2) inset;
    max-height: 70vh; /* 防止过高，可根据需要调整 */
  }

  /* 横屏视频容器比例 */
  .cover.landscape {
    aspect-ratio: 16 / 9;
  }

  /* 竖屏视频容器比例 */
  .cover.portrait {
    aspect-ratio: 9 / 16;
  }

  .video-background {
    width: 100%;
    height: 100%;
    object-fit: cover;
    opacity: 0.7;
    filter: saturate(1.2) contrast(1.1) hue-rotate(-10deg);
  }

  /* 移动端覆盖（可调整最大高度） */
  @media (max-width: 768px) {
    .cover {
      max-height: 50vh; /* 移动端适当降低 */
    }
  }

  /* 加载遮罩 */
  .loading-overlay {
    position: absolute;
    inset: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background: rgba(10, 5, 3, 0.7);
    backdrop-filter: blur(4px);
    z-index: 5;

    .spinner {
      width: 48px;
      height: 48px;
      border-radius: 50%;
      border: 4px solid rgba(255, 140, 60, 0.2);
      border-top-color: var(--accent-2);
      animation: spin 1s linear infinite;
    }

    .loading-text {
      margin-top: 12px;
      color: #ffb47b;
      font-weight: 600;
    }
  }

  .controls {
    margin-top: 20px;

    .title {
      font-size: 1.3rem;
      font-weight: 700;
      color: #ffd9b0;
      margin-bottom: 8px;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }

    .meta {
      display: flex;
      gap: 8px;
      color: #b99f8b;
      font-size: 0.9rem;
      margin-bottom: 16px;
    }
  }

  /* 进度条 */
  .progress-wrap {
    margin-bottom: 20px;
    
    position: relative;
    touch-action: none;

    .progress-bar {
      height: 8px;
      background: rgba(255, 255, 255, 0.1);
      border-radius: 4px;
      overflow: hidden;
    }

    .progress {
      height: 100%;
      background: linear-gradient(90deg, var(--accent-2), var(--accent));
      border-radius: 4px;
      position: relative;
      box-shadow: 0 0 10px var(--accent);
    }

    .progress-handle {
      position: absolute;
      top: -4px;
      width: 16px;
      height: 16px;
      background: #ffd9b0;
      border-radius: 50%;
      transform: translateX(-50%);
      box-shadow: 0 0 15px var(--accent-2);
      transition: transform 0.2s;

      &:hover {
        transform: translateX(-50%) scale(1.2);
      }
    }
  }

  /* 按钮行 */
  .btns {
    display: flex;
    align-items: center;
    gap: 12px;
    flex-wrap: wrap;

    .icon,
    .play {
      background: rgba(30, 15, 8, 0.6);
      border: 1px solid rgba(255, 140, 60, 0.2);
      border-radius: 40px;
      padding: 8px 16px;
      color: #ffb47b;
      font-size: 1.2rem;
      
      transition: all 0.3s;

      &:hover {
        background: rgba(50, 25, 15, 0.8);
        border-color: var(--accent-2);
        transform: translateY(-3px);
      }
    }

    .play {
      background: linear-gradient(145deg, #ff974f, #ff4f1e);
      color: #1a0c06;
      font-weight: 700;
      border: none;
      padding: 8px 24px;

      &:hover {
        transform: translateY(-4px);
        box-shadow: 0 15px 30px rgba(255, 100, 30, 0.4);
      }
    }

    .modes {
      display: flex;
      gap: 4px;

      button {
        background: rgba(20, 10, 5, 0.6);
        border: 1px solid rgba(255, 140, 60, 0.2);
        border-radius: 30px;
        padding: 6px 12px;
        color: #b99f8b;
        
        transition: all 0.3s;

        &.active {
          background: rgba(255, 100, 30, 0.2);
          border-color: var(--accent-2);
          color: #ffb47b;
        }

        &:hover {
          border-color: var(--accent-2);
        }
      }
    }

    .volume {
      input[type="range"] {
        width: 80px;
        height: 4px;
        background: rgba(255, 140, 60, 0.2);
        border-radius: 2px;
        outline: none;

        &::-webkit-slider-thumb {
          -webkit-appearance: none;
          width: 16px;
          height: 16px;
          background: #ffd9b0;
          border-radius: 50%;
          box-shadow: 0 0 10px var(--accent-2);
          
          transition: transform 0.2s;

          &:hover {
            transform: scale(1.2);
          }
        }
      }
    }
  }

  .error-msg {
    margin-top: 12px;
    color: #ff6b6b;
    font-size: 0.9rem;
  }

  /* ========== 右侧播放列表（优化） ========== */
  .right {
    flex: 1;
    background: rgba(10, 5, 3, 0.6);
    backdrop-filter: blur(12px);
    border: 1px solid rgba(255, 140, 60, 0.2);
    border-radius: 32px;
    padding: 24px;
    box-shadow: 0 25px 50px -10px rgba(0, 0, 0, 0.6);
    transition: all 0.3s;
    max-height: 600px;
    overflow: hidden;
    display: flex;
    flex-direction: column;

    @media (max-width: 1024px) {
      width: 100%;
      max-height: 400px;
    }

    &.collapsed {
      max-height: 80px;
    }
  }

  .playlist-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
    padding-bottom: 12px;
    border-bottom: 1px solid rgba(255, 140, 60, 0.2);

    .left-head {
      display: flex;
      align-items: center;
      gap: 12px;

      h3 {
        margin: 0;
        font-size: 1.3rem;
        color: #ffb47b;
        font-weight: 700;
      }

      .toggle-list-text {
        background: rgba(30, 15, 8, 0.6);
        border: 1px solid rgba(255, 140, 60, 0.2);
        border-radius: 30px;
        padding: 4px 12px;
        color: #ffb47b;
        
        transition: all 0.3s;

        &:hover {
          background: rgba(50, 25, 15, 0.8);
          border-color: var(--accent-2);
        }
      }

      .api-hint {
        color: #b99f8b;
        font-size: 0.85rem;
      }
    }

    .search-wrap {
      display: flex;
      align-items: center;
      gap: 8px;

      input {
        padding: 8px 16px;
        background: rgba(0, 0, 0, 0.3);
        border: 1px solid rgba(255, 140, 60, 0.2);
        border-radius: 30px;
        color: var(--text-light);
        outline: none;
        transition: all 0.3s;

        &:focus {
          border-color: var(--accent-2);
          box-shadow: 0 0 0 3px rgba(255, 100, 30, 0.2);
        }
      }

      .clear {
        background: transparent;
        border: none;
        color: #b99f8b;
        
        padding: 4px 8px;

        &:hover {
          color: #ffb47b;
        }
      }
    }
  }

  .list-area {
    flex: 1;
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

  .list-loading {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 20px;
    color: #b99f8b;

    .small-spinner {
      width: 20px;
      height: 20px;
      border-radius: 50%;
      border: 2px solid rgba(255, 140, 60, 0.2);
      border-top-color: var(--accent-2);
      animation: spin 1s linear infinite;
    }
  }

  .playlist {
    list-style: none;
    padding: 0;
    margin: 0;

    li {
      display: grid;
      grid-template-columns: 1fr auto;
      gap: 12px;
      align-items: center;
      padding: 12px 16px;
      margin-bottom: 6px;
      border-radius: 40px;
      background: rgba(0, 0, 0, 0.2);
      border: 1px solid transparent;
      
      transition: all 0.3s;

      &:hover {
        background: rgba(30, 15, 8, 0.5);
        border-color: rgba(255, 140, 60, 0.3);
        transform: translateX(-6px);
      }

      &.active {
        background: rgba(255, 100, 30, 0.15);
        border-color: var(--accent-2);
        box-shadow: 0 0 20px rgba(255, 100, 30, 0.2);
      }

      .left-col {
        display: flex;
        align-items: center;
        gap: 12px;
        min-width: 0; /* 确保文字可收缩 */

        .flame-dot {
          width: 12px;
          height: 20px;
          border-radius: 4px;
          transform: skewX(-5deg);
          flex-shrink: 0;

          &.level-1 {
            background: linear-gradient(180deg, #ff974f, #ff4f1e);
          }
          &.level-2 {
            background: linear-gradient(180deg, #ffb47b, #ff6324);
          }
          &.level-3 {
            background: linear-gradient(180deg, #ffc99e, #ff7b3a);
          }
          &.level-4 {
            background: linear-gradient(180deg, #ffe0ba, #ff974f);
            box-shadow: 0 0 10px #ffb47b;
          }
        }

        .title {
          font-size: 0.95rem;
          color: var(--text-light);
          white-space: nowrap;
          overflow: hidden;
          text-overflow: ellipsis;
          flex: 1; /* 占用剩余空间 */
          min-width: 0; /* 允许收缩 */
        }
      }

      .right-col .len {
        color: #b99f8b;
        font-size: 0.85rem;
        font-family: monospace;
      }
    }
  }

  /* ========== 动画 ========== */
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

  @keyframes spin {
    to {
      transform: rotate(360deg);
    }
  }

  /* ========== 响应式优化 ========== */
  @media (max-width: 768px) {
    padding-top: 60px;

    .btns {
      justify-content: center;
    }

    .volume {
      display: none;
    }

    .playlist-header {
      flex-direction: column;
      align-items: flex-start;
      gap: 12px;
    }

    .feather-decoration {
      display: none;
    }

    /* 移动端列表文字适当放大 */
    .playlist li .left-col .title {
      font-size: 0.9rem;
    }
  }
}
</style>
