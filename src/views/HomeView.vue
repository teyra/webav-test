<template>
  <div>
    <button @click="start">启动！</button>
    <video :src="video" controls autoPlay></video>
    <video :src="newVideoUrl" controls autoPlay></video>
  </div>
</template>

<script setup lang="ts">
import {
  Combinator,
  ImgClip,
  MP4Clip,
  OffscreenSprite,
  renderTxt2ImgBitmap
} from '@webav/av-cliper'
import video from '@/assets/trailer.mp4'
import { ref } from 'vue'
let newVideoUrl = ref()
async function start() {
  const spr1 = new OffscreenSprite('spr1', new MP4Clip((await fetch(video)).body!))

  const spr2 = new OffscreenSprite(
    'spr2',
    new ImgClip(
      await renderTxt2ImgBitmap(
        '水印',
        `font-size:40px; color: white; text-shadow: 2px 2px 6px red;`
      )
    )
  )
  spr2.setAnimation(
    {
      '0%': { x: 0, y: 0 },
      '25%': { x: 1200, y: 680 },
      '50%': { x: 1200, y: 0 },
      '75%': { x: 0, y: 680 },
      '100%': { x: 0, y: 0 }
    },
    { duration: 4, iterCount: 1 }
  )
  spr2.zIndex = 10
  spr2.opacity = 0.5

  const com = new Combinator({
    width: 1280,
    height: 720,
    bgColor: 'white'
  })

  await com.add(spr1, { main: true })
  await com.add(spr2, { offset: 0, duration: 5 })
  const srcBlob = await new Response(com?.output()).blob()
  console.log('🚀 ~ start ~ srcBlob:', srcBlob)
  newVideoUrl.value = URL.createObjectURL(srcBlob)
}
</script>
