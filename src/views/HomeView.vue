<script setup lang="ts">
import { ref } from 'vue'
import type { UploadFile } from 'element-plus'
// import { Config, removeBackgroundFromImage } from '@imgly/background-removal'
import { removeBackground } from '@imgly/background-removal'
type image_src = ImageData | ArrayBuffer | Uint8Array | Blob | URL | string

const testFetch = async () => {
  const url =
    'https://staticimgly.com/@imgly/background-removal-data/1.5.5/dist/f27ae13d9f59f61a6a1b5d9a91c203f2c613346083f02072c27c91672ccda8cf'
  const response = await fetch(url)
  console.log('reponse', response)
}

testFetch()

// 选择图片
const orginImg = ref<Blob | string>('')
const uploadImg = (file: UploadFile) => {
  // 拿到本地图片地址
  orginImg.value = file.url
  // removeBackgorund(file.url)
  removeBackgorundFn(file.url)
}

const loading = ref<boolean>(false)
let config = {
  publicPath: 'https://staticimgly.com/@imgly/background-removal-data/1.5.5/dist/', //将模型文件存入本地，直接从项目本地跑模型，省去了下载模型的时间
  // publicPath: '/public/models',
  debug: true,
  model: 'isnet_quint8',
  progress: (key: string, current: number, total) => {
    console.log(`Downloading ${key}: ${current} of ${total}`)
    //进度回调，目前只会返回加载模型的进度，处理图片的进度不会返回
    loading.value = !(key === 'compute:inference' && current === 1)
    loading.value = current !== total
  }
}
// 抠图方法

const feedBackImg = ref<Blob | string>('')
const removeBackgorundFn = async (temUrl: image_src) => {
  const blob: Blob = await removeBackground(temUrl, config)
  console.log('blob', blob)
  const url = URL.createObjectURL(blob)
  feedBackImg.value = url
}
</script>

<template>
  <main>
    <h1>本地抠图工具</h1>
    <h2>选择图片：</h2>
    <el-upload
      action="#"
      :auto-upload="false"
      :on-change="uploadImg"
      list-type="picture-card"
      accept=".png,.jpg"
    >
      <el-icon></el-icon>
    </el-upload>
    <el-row :gutter="20">
      <el-col :span="12" v-if="orginImg">
        <h2>原图：</h2>
        <el-image :src="orginImg" fit="contain" />
      </el-col>
      <el-col :span="12" v-loading="loading">
        <h2>抠图结果：</h2>
        <el-image v-if="feedBackImg" :src="feedBackImg" fit="contain" />
      </el-col>
    </el-row>
  </main>
</template>
