<template>
  <div>
    <div style="display: flex; justify-content: flex-start; padding: 10px">
      <button @click="combinator" style="margin: 10px">添加字幕logo</button>
      <button @click="videoSplice" style="margin: 10px">视频拼接</button>
      <button @click="openKeyFrame" style="margin: 10px">拆分关键帧</button>
    </div>
    <video :src="video3" autoPlay style="width: 800px; height: 500px" controls></video>
    <video :src="newVideoUrl" autoPlay style="width: 800px; height: 500px" controls></video>
    <div style="display: flex; justify-content: flex-start">
      <img
        :src="item.img"
        alt=""
        v-for="(item, index) in list"
        :key="index"
        style="padding: 10px"
      />
    </div>
    <a :href="newVideoUrl" :download="`WebAV-${Date.now()}.mp4`" target="_self"> 导出视频 </a>
  </div>
</template>

<script setup lang="ts">
import {
  Combinator,
  ImgClip,
  MP4Clip,
  OffscreenSprite,
  fastConcatMP4,
  renderTxt2ImgBitmap
} from '@webav/av-cliper'
import video from '@/assets/webav1.mp4'
import video1 from '@/assets/video1.mp4'
import video2 from '@/assets/video2.mp4'
import video3 from '@/assets/trailer.mp4'
import logo from '@/assets/logo.png'
import { onMounted, ref } from 'vue'
const list = ref<any[]>([])
let newVideoUrl = ref()
const textList = ref([
  '素胚勾勒出青花笔锋浓转淡',
  '瓶身描绘的牡丹一如你初妆',
  '冉冉檀香透过窗心事我了然',
  '宣纸上走笔至此搁一半',
  '釉色渲染仕女图韵味被私藏',
  '而你嫣然的一笑如含苞待放',
  '你的美一缕飘散',
  '去到我去不了的地方'
])
const colorList = ref([
  '#5DACEA',
  '#66C947',
  '#FF9800',
  '#F44336',
  '#F0B3FA',
  '#AED581',
  '#48DED0',
  '#FFFFFF'
])
onMounted(async () => {
  console.log(await Combinator.isSupported())
})
const openKeyFrame = async () => {
  const clip = new MP4Clip((await fetch(video3)).body!)
  await clip.ready
  list.value = (await clip.thumbnails()).map((it) => ({
    ts: it.ts,
    img: URL.createObjectURL(it.img)
  }))
}
const videoSplice = async () => {
  const stream = await fastConcatMP4(
    await Promise.all([(await fetch(video1)).body!, (await fetch(video2)).body!])
  )
  const srcBlob = await new Response(stream).blob()
  newVideoUrl.value = URL.createObjectURL(srcBlob)
}
const combinator = async () => {
  const com = new Combinator({
    width: 1280,
    height: 720,
    bgColor: 'white'
  })
  const videoSprite = new OffscreenSprite('videoSprite', new MP4Clip((await fetch(video)).body!))
  const duration = 2
  let offset = 0
  textList.value.map(async (v, i) => {
    const spr = new OffscreenSprite(
      'spr' + i,
      new ImgClip(await renderTxt2ImgBitmap(v, `font-size:40px; color: ${colorList.value[i]};`))
    )
    spr.rect.x = 100
    spr.rect.y = 100
    spr.zIndex = 10
    spr.opacity = 0.9
    offset += i ? 2 : 0
    console.log('🚀 ~ textList.value.map ~ duration:', duration)
    console.log('🚀 ~ textList.value.map ~ offset:', offset)
    await com.add(spr, { duration, offset })
  })
  const logoSprite = new OffscreenSprite(
    'logoSprite',
    new ImgClip(await createImageBitmap(await (await fetch(logo)).blob()))
  )
  logoSprite.rect.x = 20
  logoSprite.rect.y = 20

  await com.add(videoSprite, { main: true })
  await com.add(logoSprite)
  const srcBlob = await new Response(com?.output()).blob()
  newVideoUrl.value = URL.createObjectURL(srcBlob)
}
</script>
