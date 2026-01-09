<script setup>
import { onMounted, ref } from 'vue'
import { isMobileOrTabletDevice } from '@/composables/useDevice'

const appInfo = ref({
  version: '',
  versionCode: 0,
  downloadUrl: '',
  changelog: '',
  releaseDate: '',
})
const loading = ref(true)
const downloading = ref(false)
const error = ref(null)

// 获取版本信息
async function fetchAppInfo() {
  try {
    const response = await fetch('/app-version.json')
    if (!response.ok)
      throw new Error('获取版本信息失败')
    appInfo.value = await response.json()
  }
  catch (e) {
    error.value = e.message
  }
  finally {
    loading.value = false
  }
}

// 下载 APK
async function handleDownload() {
  if (!appInfo.value.downloadUrl || downloading.value)
    return

  downloading.value = true
  try {
    if (isMobileOrTabletDevice()) {
      window.location.href = appInfo.value.downloadUrl
      setTimeout(() => {
        downloading.value = false
      }, 1000)
      return
    }

    const response = await fetch(appInfo.value.downloadUrl)
    if (!response.ok)
      throw new Error('下载失败')

    const blob = await response.blob()
    const url = URL.createObjectURL(blob)
    const link = document.createElement('a')
    link.href = url
    link.download = `Wallpaper-Gallery-v${appInfo.value.version}.apk`
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
    URL.revokeObjectURL(url)
    downloading.value = false
  }
  catch (error) {
    console.error('下载失败:', error)
    window.location.href = appInfo.value.downloadUrl
    downloading.value = false
  }
}

// 格式化日期
function formatDate(dateStr) {
  if (!dateStr)
    return ''
  const date = new Date(dateStr)
  return date.toLocaleDateString('zh-CN', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  })
}

onMounted(() => {
  fetchAppInfo()
})
</script>

<template>
  <div class="download-page">
    <div class="download-container">
      <!-- App 图标和名称 -->
      <div class="app-header">
        <div class="app-icon">
          <img src="/icon-192.png" alt="Wallpaper Gallery">
        </div>
        <h1 class="app-name">
          Wallpaper Gallery
        </h1>
        <p class="app-desc">
          精选高清4K壁纸
        </p>
      </div>

      <!-- 加载状态 -->
      <div v-if="loading" class="loading-state">
        <div class="spinner" />
        <p>加载中...</p>
      </div>

      <!-- 错误状态 -->
      <div v-else-if="error" class="error-state">
        <p>{{ error }}</p>
        <button class="retry-btn" @click="fetchAppInfo">
          重试
        </button>
      </div>

      <!-- 版本信息和下载 -->
      <div v-else class="version-info">
        <div class="version-row">
          <div class="version-badge">
            <span class="version-label">最新版本</span>
            <span class="version-number">v{{ appInfo.version }}</span>
          </div>
          <span v-if="appInfo.releaseDate" class="release-date">
            {{ formatDate(appInfo.releaseDate) }}
          </span>
        </div>

        <!-- 下载按钮 -->
        <button
          class="download-btn"
          :class="{ 'is-downloading': downloading }"
          :disabled="!appInfo.downloadUrl || downloading"
          @click="handleDownload"
        >
          <svg v-if="!downloading" class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" />
            <polyline points="7 10 12 15 17 10" />
            <line x1="12" y1="15" x2="12" y2="3" />
          </svg>
          <div v-else class="btn-spinner" />
          <span>{{ downloading ? '下载中...' : '下载 APK' }}</span>
        </button>

        <!-- 更新日志 -->
        <p v-if="appInfo.changelog" class="changelog">
          {{ appInfo.changelog }}
        </p>
      </div>

      <!-- 功能特性 -->
      <div class="features">
        <ul>
          <li>
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <polyline points="20 6 9 17 4 12" />
            </svg>
            <span>海量高清壁纸</span>
          </li>
          <li>
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <polyline points="20 6 9 17 4 12" />
            </svg>
            <span>电脑/手机/头像</span>
          </li>
          <li>
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <polyline points="20 6 9 17 4 12" />
            </svg>
            <span>一键下载保存</span>
          </li>
          <li>
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <polyline points="20 6 9 17 4 12" />
            </svg>
            <span>真机预览效果</span>
          </li>
        </ul>
      </div>

      <!-- 安装说明 -->
      <div class="install-tips">
        <ol>
          <li>点击下载按钮获取 APK</li>
          <li>打开文件进行安装</li>
          <li>如提示风险，选择「仍要安装」</li>
        </ol>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.download-page {
  position: fixed;
  inset: 0;
  overflow: hidden;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.download-container {
  max-width: 340px;
  width: 100%;
  background: #ffffff;
  border-radius: 20px;
  padding: 20px 16px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.app-header {
  text-align: center;
  margin-bottom: 12px;
}

.app-icon {
  width: 64px;
  height: 64px;
  margin: 0 auto 8px;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 6px 18px rgba(99, 102, 241, 0.3);

  img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
}

.app-name {
  font-size: 18px;
  font-weight: 700;
  color: #1a1a2e;
  margin-bottom: 2px;
}

.app-desc {
  font-size: 12px;
  color: #6c757d;
}

.loading-state,
.error-state {
  text-align: center;
  padding: 16px 0;
  color: #6c757d;
}

.spinner,
.btn-spinner {
  width: 24px;
  height: 24px;
  border: 3px solid #e9ecef;
  border-top-color: #6366f1;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
  margin: 0 auto 8px;
}

.btn-spinner {
  width: 16px;
  height: 16px;
  border-width: 2px;
  border-color: rgba(255, 255, 255, 0.3);
  border-top-color: #ffffff;
  margin: 0;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.retry-btn {
  margin-top: 8px;
  padding: 6px 16px;
  background: #6366f1;
  color: #ffffff;
  border: none;
  border-radius: 8px;
  font-size: 12px;
  cursor: pointer;

  &:hover {
    background: #4f46e5;
  }
}

.version-info {
  margin-bottom: 12px;
}

.version-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
}

.version-badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  background: #f0f1ff;
  padding: 4px 10px;
  border-radius: 12px;
}

.version-label {
  font-size: 10px;
  color: #6c757d;
}

.version-number {
  font-size: 12px;
  font-weight: 600;
  color: #6366f1;
}

.release-date {
  font-size: 11px;
  color: #adb5bd;
}

.download-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  width: 100%;
  padding: 12px 16px;
  background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
  color: #ffffff;
  border: none;
  border-radius: 10px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 6px 20px rgba(99, 102, 241, 0.4);

  .icon {
    width: 18px;
    height: 18px;
  }

  &:hover:not(:disabled) {
    transform: translateY(-2px);
    box-shadow: 0 10px 28px rgba(99, 102, 241, 0.5);
  }

  &:active:not(:disabled) {
    transform: translateY(0);
  }

  &:disabled {
    opacity: 0.7;
    cursor: not-allowed;
  }

  &.is-downloading {
    background: #8b5cf6;
  }
}

.changelog {
  margin-top: 10px;
  padding: 8px 10px;
  background: #f8f9fa;
  border-radius: 8px;
  font-size: 11px;
  color: #6c757d;
  line-height: 1.4;
}

.features {
  padding-top: 12px;
  border-top: 1px solid #e9ecef;

  ul {
    list-style: none;
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 6px;

    li {
      display: flex;
      align-items: center;
      gap: 4px;
      font-size: 11px;
      color: #495057;

      svg {
        width: 12px;
        height: 12px;
        color: #10b981;
        flex-shrink: 0;
      }
    }
  }
}

.install-tips {
  margin-top: 12px;
  padding-top: 12px;
  border-top: 1px solid #e9ecef;

  ol {
    list-style: none;
    counter-reset: step;
    display: flex;
    flex-direction: column;
    gap: 6px;

    li {
      position: relative;
      padding-left: 24px;
      font-size: 11px;
      color: #495057;
      line-height: 1.3;

      &::before {
        content: counter(step);
        counter-increment: step;
        position: absolute;
        left: 0;
        top: 0;
        width: 16px;
        height: 16px;
        background: #f0f1ff;
        color: #6366f1;
        border-radius: 50%;
        font-size: 10px;
        font-weight: 600;
        display: flex;
        align-items: center;
        justify-content: center;
      }
    }
  }
}
</style>
