<template>
  <view class="container">
    <!-- 背景装饰 -->
    <view class="bg-decoration"></view>
    
    <!-- 标题区域 -->
    <view class="header">
      <view class="header-icon">
        <text class="icon">📸</text>
      </view>
      <text class="title">LIVP 图片提取工具</text>
      <text class="subtitle">支持 HEIC 自动转换为 JPG · 支持批量下载</text>
    </view>

    <!-- 上传区域 -->
    <view class="upload-section">
      <view class="upload-box" @tap="chooseFiles">
        <view class="upload-icon">
          <text class="upload-icon-text">📁</text>
        </view>
        <text class="upload-text">点击选择 .livp 文件</text>
        <text class="upload-hint">支持批量选择，最多 200 个文件</text>
      </view>
    </view>

    <!-- 提示信息 -->
    <view class="tips-box">
      <text class="tips-icon">💡</text>
      <text class="tips">批量选择多个 LIVP 文件会自动处理，请耐心等待</text>
    </view>

    <!-- 图片预览区域 -->
    <view v-if="images.length > 0" class="preview-section">
      <view class="preview-header">
        <text class="preview-count">已提取 {{ images.length }} 张图片</text>
      </view>
      <view class="preview-grid">
        <view v-for="(item,index) in images" :key="index" class="preview-item">
          <view class="preview-img-wrapper" @tap="previewImage(index)">
            <image :src="item.url" mode="aspectFill" class="preview-img"></image>
            <view class="preview-mask">
              <text class="preview-hint">点击放大</text>
            </view>
          </view>
          <view class="preview-footer">
            <text class="preview-name">{{ item.name }}</text>
            <view class="action-btn" @tap.stop="downloadImage(item)">
              <text class="action-icon">⬇️</text>
              <text class="action-text">下载</text>
            </view>
          </view>
        </view>
      </view>
    </view>

    <!-- 下载全部按钮 -->
    <view v-if="images.length" class="download-section">
      <view class="download-all-btn" @tap="downloadAll">
        <text class="download-icon">📦</text>
        <text class="download-text">下载全部图片</text>
      </view>
    </view>
  </view>
</template>

<script>
import JSZip from "jszip"
import { heicTo } from "heic-to"
export default {
  data() {
    return {
      images: [], // {url, name}
      //传入文件的长度
      fileLength:0
    }
  },
  methods: {
    async chooseFiles() {
      const res = await uni.chooseFile({
        count: 200,
        type: "all",
        extension: [".livp"]
      })
      this.handleLivpFiles(res.tempFiles)
    },

    async handleLivpFiles(files) {
      this.fileLength = files.length
      uni.showLoading({
        title: '转换解析中,请稍等....'
      });
      for (const f of files) {
        const fileData = await this.readAsArrayBuffer(f)
        const zip = new JSZip()
        const zipContent = await zip.loadAsync(fileData)

        for (const [filename, fileObj] of Object.entries(zipContent.files)) {
          if (/\.(jpg|jpeg)$/i.test(filename)) {
            const blob = await fileObj.async("blob")
            this.pushImage(blob, filename)
          } else if (/\.heic$/i.test(filename)) {
            const heicBlob = await fileObj.async("blob")
            try {
              const jpgBlob = await heicTo({
                blob: heicBlob,
                type: "image/jpeg",
                quality: 0.9
              });
              const newName = filename.replace(/\.heic$/i, ".jpg")
              this.pushImage(jpgBlob, newName)
            } catch (err) {
              console.error("HEIC 转换失败:", err)
              uni.showToast({ title: "HEIC 转换失败", icon: "none" })
            }
          }
        }
      }
    },

    readAsArrayBuffer(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader()
        reader.onload = e => resolve(e.target.result)
        reader.onerror = reject
        reader.readAsArrayBuffer(file)
      })
    },

    pushImage(blob, name) {
      const url = URL.createObjectURL(blob)
      this.images.push({ url, name })
      if(this.images.length === this.fileLength) uni.hideLoading()
    },

    previewImage(index) {
      // 使用 uni.previewImage 进行图片预览
      const urls = this.images.map(img => img.url)
      uni.previewImage({
        urls: urls,
        current: index
      })
    },

    downloadImage(item) {
      // H5 支持 a 标签下载，小程序需改成保存到相册
      const a = document.createElement("a")
      a.href = item.url
      a.download = item.name
      a.click()
    },

    async downloadAll() {
      const zip = new JSZip()
      for (const img of this.images) {
        const blob = await fetch(img.url).then(r => r.blob())
        zip.file(img.name, blob)
      }
      const content = await zip.generateAsync({ type: "blob" })
      const url = URL.createObjectURL(content)
      const a = document.createElement("a")
      a.href = url
      a.download = "images.zip"
      a.click()
    }
  }
}
</script>

<style scoped>
.container {
  position: relative;
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  box-sizing: border-box;
  padding-bottom: 40rpx;
}

/* 背景装饰 */
.bg-decoration {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
  opacity: 0.95;
  z-index: 0;
}

.bg-decoration::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(255,255,255,0.1) 1px, transparent 1px);
  background-size: 60rpx 60rpx;
  animation: float 20s infinite linear;
}

@keyframes float {
  0% { transform: translate(0, 0) rotate(0deg); }
  100% { transform: translate(-60rpx, -60rpx) rotate(360deg); }
}

/* 标题区域 */
.header {
  position: relative;
  z-index: 1;
  margin: 60rpx 0 40rpx;
  text-align: center;
  padding: 0 40rpx;
}

.header-icon {
  margin-bottom: 20rpx;
}

.icon {
  font-size: 80rpx;
  display: inline-block;
  animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.1); }
}

.title {
  display: block;
  font-size: 48rpx;
  font-weight: 700;
  color: #ffffff;
  margin-bottom: 16rpx;
  letter-spacing: 2rpx;
  text-shadow: 0 4rpx 12rpx rgba(0,0,0,0.2);
}

.subtitle {
  display: block;
  font-size: 26rpx;
  color: rgba(255,255,255,0.9);
  font-weight: 300;
  line-height: 1.6;
}

/* 上传区域 */
.upload-section {
  position: relative;
  z-index: 1;
  padding: 0 40rpx;
  margin-bottom: 30rpx;
}

.upload-box {
  width: 100%;
  min-height: 280rpx;
  border: 3rpx dashed rgba(255,255,255,0.6);
  border-radius: 24rpx;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: rgba(255,255,255,0.15);
  backdrop-filter: blur(10rpx);
  transition: all 0.3s ease;
  box-shadow: 0 8rpx 32rpx rgba(0,0,0,0.1);
}

.upload-box:active {
  transform: scale(0.98);
  background: rgba(255,255,255,0.25);
}

.upload-icon {
  margin-bottom: 20rpx;
}

.upload-icon-text {
  font-size: 64rpx;
  display: inline-block;
}

.upload-text {
  color: #ffffff;
  font-size: 32rpx;
  font-weight: 600;
  margin-bottom: 12rpx;
}

.upload-hint {
  color: rgba(255,255,255,0.8);
  font-size: 24rpx;
}

/* 提示信息 */
.tips-box {
  position: relative;
  z-index: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20rpx 40rpx;
  margin: 0 40rpx 30rpx;
  background: rgba(255,255,255,0.2);
  backdrop-filter: blur(10rpx);
  border-radius: 16rpx;
  box-shadow: 0 4rpx 16rpx rgba(0,0,0,0.1);
}

.tips-icon {
  font-size: 32rpx;
  margin-right: 12rpx;
}

.tips {
  font-size: 24rpx;
  color: rgba(255,255,255,0.95);
  line-height: 1.6;
  flex: 1;
}

/* 图片预览区域 */
.preview-section {
  position: relative;
  z-index: 1;
  width: 100%;
  padding: 0 30rpx;
  margin-top: 20rpx;
}

.preview-header {
  padding: 20rpx 30rpx;
  margin-bottom: 20rpx;
  text-align: center;
}

.preview-count {
  font-size: 28rpx;
  color: rgba(255,255,255,0.95);
  font-weight: 600;
  background: rgba(255,255,255,0.2);
  backdrop-filter: blur(10rpx);
  padding: 12rpx 32rpx;
  border-radius: 50rpx;
  display: inline-block;
}

.preview-grid {
  display: flex;
  flex-wrap: wrap;
  justify-content: flex-start;
  padding: 0 10rpx;
}

.preview-item {
  width: calc((100% - 40rpx) / 5);
  margin-right: 10rpx;
  margin-bottom: 20rpx;
  background: rgba(255,255,255,0.95);
  border-radius: 12rpx;
  overflow: hidden;
  box-shadow: 0 4rpx 16rpx rgba(0,0,0,0.12);
  transition: all 0.3s ease;
}

.preview-item:nth-child(5n) {
  margin-right: 0;
}

.preview-item:active {
  transform: translateY(-2rpx);
  box-shadow: 0 6rpx 20rpx rgba(0,0,0,0.18);
}

.preview-img-wrapper {
  position: relative;
  width: 100%;
  padding-top: 100%;
  overflow: hidden;
  background: #f5f5f5;
}

.preview-img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.preview-mask {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.3);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.preview-img-wrapper:active .preview-mask {
  opacity: 1;
}

.preview-hint {
  color: #ffffff;
  font-size: 18rpx;
  font-weight: 500;
  background: rgba(0,0,0,0.5);
  padding: 6rpx 12rpx;
  border-radius: 6rpx;
}

.preview-footer {
  padding: 8rpx 10rpx;
  display: flex;
  flex-direction: column;
  gap: 8rpx;
}

.preview-name {
  font-size: 18rpx;
  color: #666;
  text-align: center;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  line-height: 1.3;
}

.action-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 8rpx 12rpx;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 8rpx;
  transition: all 0.2s ease;
  box-shadow: 0 2rpx 8rpx rgba(102,126,234,0.3);
}

.action-btn:active {
  transform: scale(0.96);
  box-shadow: 0 2rpx 6rpx rgba(102,126,234,0.2);
}

.action-icon {
  font-size: 20rpx;
  margin-right: 4rpx;
}

.action-text {
  font-size: 20rpx;
  color: #ffffff;
  font-weight: 600;
}

/* 下载全部按钮 */
.download-section {
  position: relative;
  z-index: 1;
  padding: 40rpx 40rpx 20rpx;
  margin-top: 20rpx;
}

.download-all-btn {
  width: 100%;
  height: 96rpx;
  background: linear-gradient(135deg, #ff6b6b 0%, #ee5a6f 100%);
  border-radius: 48rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 8rpx 24rpx rgba(238,90,111,0.4);
  transition: all 0.3s ease;
}

.download-all-btn:active {
  transform: translateY(2rpx);
  box-shadow: 0 4rpx 12rpx rgba(238,90,111,0.3);
}

.download-icon {
  font-size: 36rpx;
  margin-right: 12rpx;
}

.download-text {
  font-size: 32rpx;
  color: #ffffff;
  font-weight: 600;
  letter-spacing: 2rpx;
}
</style>
