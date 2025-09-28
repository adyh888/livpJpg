<template>
  <view class="container">
    <!-- 标题 -->
    <view class="header">
      <text class="title">IPhone-LIVP 图片提取工具</text>
      <text class="subtitle">(支持 HEIC 自动转换为 JPG工具-支持批量下载)</text>
    </view>

    <!-- 上传区域 -->
    <view class="upload-box" @tap="chooseFiles">
      <text class="upload-text">点击选择 .livp 文件</text>
    </view>

    <!-- 提示 -->
    <text class="tips">批量选择多个 LIVP 文件会自动处理，请耐心等待</text>

    <!-- 图片预览 -->
    <view style="display: flex;flex-wrap: wrap;justify-content: center">
      <view v-for="(item,index) in images" :key="index" class="preview-item">
        <image :src="item.url" mode="aspectFill" class="preview-img"></image>
        <view class="actions">
          <button size="mini" type="primary" @tap="downloadImage(item)">下载</button>
        </view>
      </view>
    </view>

    <!-- 下载全部 -->
    <button v-if="images.length" class="download-all" type="warn" @tap="downloadAll">
      下载全部图片
    </button>
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
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 30rpx;
  background-color: #f8f9fa;
  //min-height: 100vh;
  box-sizing: border-box;
}
.header {
  margin: 20rpx 0;
  text-align: center;
}
.title {
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
}
.subtitle {
  margin-left: 20rpx;
  font-size: 26rpx;
  color: #666;
}
.upload-box {
  width: 90%;
  height: 200rpx;
  border: 2rpx dashed #007aff;
  border-radius: 20rpx;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 30rpx 0;
  background-color: #f0f8ff;
}
.upload-text {
  color: #007aff;
  font-size: 28rpx;
}
.tips {
  font-size: 24rpx;
  color: #999;
  margin-bottom: 20rpx;
}
.preview {
  width: 100%;
  flex: 1;
}
.preview-item {
  margin-bottom: 20rpx;
  margin-right: 20rpx;
  background: white;
  border-radius: 10rpx;
  overflow: hidden;
  box-shadow: 0 4rpx 12rpx rgba(0,0,0,0.1);
}
.preview-img {
  width: 300rpx;
  height: 300rpx;
}
.actions {
  display: flex;
  justify-content: flex-end;
  padding: 10rpx;
}
.download-all {
  margin-top: 20rpx;
  width: 80%;
  border-radius: 50rpx;
}
</style>
