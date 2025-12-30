<script setup>
import { gsap } from 'gsap'
import { onMounted, ref } from 'vue'
import { getPopularWallpapers } from '@/utils/supabase'

const props = defineProps({
  series: {
    type: String,
    default: 'desktop',
  },
  // 所有壁纸数据，用于匹配完整信息
  wallpapers: {
    type: Array,
    default: () => [],
  },
})

const emit = defineEmits(['select'])

const containerRef = ref(null)
const loading = ref(true)
const popularList = ref([])

// 获取热门壁纸数据
async function fetchPopular() {
  loading.value = true
  try {
    const stats = await getPopularWallpapers(props.series, 10)

    // 将统计数据与壁纸数据匹配，获取完整信息
    popularList.value = stats
      .map((stat) => {
        const wallpaper = props.wallpapers.find(w => w.filename === stat.filename)
        if (wallpaper) {
          return {
            ...wallpaper,
            downloadCount: stat.download_count || 0,
            viewCount: stat.view_count || 0,
            popularityScore: stat.popularity_score || 0,
          }
        }
        return null
      })
      .filter(Boolean)
      .slice(0, 10)
  }
  catch (error) {
    console.error('获取热门壁纸失败:', error)
    popularList.value = []
  }
  finally {
    loading.value = false
  }
}

function handleClick(wallpaper) {
  emit('select', wallpaper)
}

onMounted(async () => {
  await fetchPopular()

  if (containerRef.value && popularList.value.length > 0) {
    gsap.fromTo(
      containerRef.value,
      { opacity: 0, y: 30 },
      { opacity: 1, y: 0, duration: 0.8, ease: 'power3.out', delay: 0.3 },
    )
  }
})
</script>

<template>
  <div v-if="!loading && popularList.length > 0" ref="containerRef" class="popular-wallpapers">
    <div class="popular-wallpapers__header">
      <div class="popular-wallpapers__badge">
        <svg viewBox="0 0 24 24" fill="none">
          <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z" fill="currentColor" />
        </svg>
        <span>🔥 热门壁纸</span>
      </div>
      <span class="popular-wallpapers__subtitle">根据下载和浏览量排名</span>
    </div>

    <div class="popular-wallpapers__grid">
      <div
        v-for="(wallpaper, index) in popularList"
        :key="wallpaper.id"
        class="popular-item"
        @click="handleClick(wallpaper)"
      >
        <div class="popular-item__rank">
          {{ index + 1 }}
        </div>
        <div class="popular-item__image-wrapper">
          <img
            :src="wallpaper.thumbnailUrl || wallpaper.previewUrl"
            :alt="wallpaper.filename"
            class="popular-item__image"
            loading="lazy"
          >
          <div class="popular-item__overlay">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <circle cx="11" cy="11" r="8" />
              <path d="M21 21l-4.35-4.35" />
            </svg>
          </div>
        </div>
        <div class="popular-item__stats">
          <span class="stat-item">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" />
              <polyline points="7 10 12 15 17 10" />
              <line x1="12" y1="15" x2="12" y2="3" />
            </svg>
            {{ wallpaper.downloadCount }}
          </span>
        </div>
      </div>
    </div>
  </div>

  <!-- Loading skeleton -->
  <div v-else-if="loading" class="popular-wallpapers popular-wallpapers--loading">
    <div class="popular-wallpapers__header">
      <div class="skeleton-badge" />
    </div>
    <div class="popular-wallpapers__grid">
      <div v-for="i in 5" :key="i" class="popular-item popular-item--skeleton">
        <div class="skeleton-image" />
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.popular-wallpapers {
  margin-bottom: $spacing-xl;
  opacity: 0;

  &--loading {
    opacity: 1;
  }
}

.popular-wallpapers__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: $spacing-md;
}

.popular-wallpapers__badge {
  display: flex;
  align-items: center;
  gap: $spacing-xs;
  padding: $spacing-xs $spacing-md;
  background: linear-gradient(135deg, #f97316, #ef4444);
  color: white;
  border-radius: $radius-full;
  font-size: $font-size-sm;
  font-weight: $font-weight-semibold;

  svg {
    width: 16px;
    height: 16px;
  }
}

.popular-wallpapers__subtitle {
  font-size: $font-size-sm;
  color: var(--color-text-muted);
}

.popular-wallpapers__grid {
  display: flex;
  gap: $spacing-md;
  overflow-x: auto;
  padding-bottom: $spacing-sm;
  scroll-snap-type: x mandatory;

  // 隐藏滚动条但保持功能
  scrollbar-width: none;
  -ms-overflow-style: none;
  &::-webkit-scrollbar {
    display: none;
  }
}

.popular-item {
  position: relative;
  flex: 0 0 180px;
  scroll-snap-align: start;
  cursor: pointer;
  transition: transform 0.3s ease;

  @include mobile-only {
    flex: 0 0 140px;
  }

  &:hover {
    transform: translateY(-4px);

    .popular-item__overlay {
      opacity: 1;
    }
  }
}

.popular-item__rank {
  position: absolute;
  top: $spacing-xs;
  left: $spacing-xs;
  z-index: 2;
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #f97316, #ef4444);
  color: white;
  font-size: $font-size-xs;
  font-weight: $font-weight-bold;
  border-radius: $radius-full;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);

  // Top 3 特殊样式
  .popular-item:nth-child(1) & {
    background: linear-gradient(135deg, #fbbf24, #f59e0b);
  }
  .popular-item:nth-child(2) & {
    background: linear-gradient(135deg, #9ca3af, #6b7280);
  }
  .popular-item:nth-child(3) & {
    background: linear-gradient(135deg, #d97706, #b45309);
  }
}

.popular-item__image-wrapper {
  position: relative;
  aspect-ratio: 16 / 10;
  border-radius: $radius-md;
  overflow: hidden;
  background: var(--color-bg-secondary);
}

.popular-item__image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;

  .popular-item:hover & {
    transform: scale(1.05);
  }
}

.popular-item__overlay {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(0, 0, 0, 0.4);
  opacity: 0;
  transition: opacity 0.3s ease;

  svg {
    width: 32px;
    height: 32px;
    color: white;
  }
}

.popular-item__stats {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: $spacing-sm;
  margin-top: $spacing-xs;
  font-size: $font-size-xs;
  color: var(--color-text-muted);
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 2px;

  svg {
    width: 12px;
    height: 12px;
  }
}

// Skeleton styles
.skeleton-badge {
  width: 120px;
  height: 28px;
  background: var(--color-bg-secondary);
  border-radius: $radius-full;
  animation: pulse 2s ease-in-out infinite;
}

.popular-item--skeleton {
  .skeleton-image {
    aspect-ratio: 16 / 10;
    background: linear-gradient(135deg, var(--color-bg-secondary) 0%, var(--color-bg-hover) 100%);
    border-radius: $radius-md;
    animation: pulse 2s ease-in-out infinite;
  }
}

@keyframes pulse {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}
</style>
