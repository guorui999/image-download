<template>
  <el-card class="box-card">
    <template #header>
      <div class="card-header">批量下载页面图片</div>
    </template>
    <div class="card-content">
      <div class="content-title">页面html</div>
      <el-input class="content-input" clearable :rows="4" v-model="inputValue" type="textarea" placeholder="请输入页面html" />
      <div class="content-title">图片最小尺寸(b)</div>
      <el-input class="content-input" clearable v-model="imgSize" type="number" placeholder="图片最小尺寸" />
      <template v-if="videoUrls.length">
        <div class="content-title">需要手动下载的视频链接</div>
        <el-input class="content-input" v-for="item in videoUrls" clearable v-model="item.value"
          placeholder="请输入页面html" />
      </template>
      <div class="content-button">
        <el-button type="primary" :disabled="!inputValue || !imgSize" @click="handleClick">获取图片</el-button>
      </div>
    </div>
  </el-card>

  <div class="home">

  </div>
</template>
<script setup>

import { ref } from 'vue'
const path = require('path')
const inputValue = ref('')
const imgSize = ref('200000')
const videoUrls = ref([])
const imageUrls = ref([])


// 图片和视频保存的文件夹路径


/**
 * 获取图片
 * @param {type} 参数
 * @returns {type} 返回值
 */
const handleClick = async () => {
  const html = inputValue.value

  const imgRegex = /<img.*?src="(.*?)"/g;
  const videoRegex = /<video.*?src="(.*?)"/g;
  let match;
  imageUrls.value = []
  while ((match = imgRegex.exec(html))) {
    let imageUrl = match[1];
    if (imageUrl.startsWith('//img') || imageUrl.startsWith('//sc') || imageUrl.startsWith('//gd') || imageUrl.startsWith('//cbu') || imageUrl.startsWith('http')) {
      imageUrl = !imageUrl.startsWith('http') ? ('https:' + imageUrl) : imageUrl;
      // 下载图片并保存
      downloadPicture(imageUrl);
    }
  }

  videoUrls.value = []
  const videoUrls2 = []

  while ((match = videoRegex.exec(html))) {
    let videoUrl = match[1];
    videoUrl = !videoUrl.startsWith('http') ? ('https:' + videoUrl) : videoUrl;
    if (videoUrl.includes('http:')) {
      // http: 只能手动下载
      videoUrls.value.push({ value: videoUrl })
    } else {
      if (!videoUrls2.includes(videoUrl)) {
        videoUrls2.push(videoUrl)
        downloadVideo(videoUrl);
      }
    }
  }
}


/**
 * 下载图片
 * @param {type} 参数
 * @returns {type} 返回值
 */
function downloadPicture(imgSrc) {
  if (imgSrc.includes('csr-open') || imgSrc.includes('atmgatew') || imgSrc.includes('TB1I_')) return
  const fileName = path.basename(imgSrc).replace(/\?.*/g, '');
  const image = new Image()
  // 解决跨域Canvas 污染问题
  image.setAttribute('crossOrigin', 'anonymous')
  image.src = imgSrc
  image.onload = () => {
    const canvas = document.createElement('canvas')
    canvas.width = image.width
    canvas.height = image.height
    const context = canvas.getContext('2d')
    context.drawImage(image, 0, 0, image.width, image.height)
    canvas.toBlob((blob) => {
      if (blob.size < Number(imgSize.value)) return
      if (imageUrls.value.find(v => v.name === fileName && v.size === blob.size)) return
      imageUrls.value.push({ name: fileName, size: blob.size })
      const url = URL.createObjectURL(blob)
      const a = document.createElement('a')
      a.download = fileName
      a.href = url
      a.click()
      a.remove()
      // 用完释放URL对象
      URL.revokeObjectURL(url)
    })
  }
}

/**
 * 下载视频
 * @param {type} 参数
 * @returns {type} 返回值
 */
function downloadVideo(videoSrc) {
  const fileName = path.basename(videoSrc);

  fetch(videoSrc).then(res => res.blob()).then(blob => {
    const a = document.createElement('a');
    document.body.appendChild(a)
    a.style.display = 'none'
    const url = window.URL.createObjectURL(blob);
    a.href = url;
    a.download = fileName; //视频下载后的名称
    a.click();
    document.body.removeChild(a)
    window.URL.revokeObjectURL(url);
  })

}

</script>
<style scoped lang="scss">
.box-card {
  margin: 100px auto 0;
  width: 500px;

  .card-header {
    font-size: 18px;
    font-weight: 550;
  }

  .card-content {
    text-align: left;

    .content-title {
      margin-bottom: 15px;
    }

    .content-input {
      margin-bottom: 20px;
    }

    .content-button {
      text-align: center;
    }
  }
}
</style>
