<template>
  <div>
    <button @click="start">启动！</button>
    <video :src="video" controls autoPlay style="width: 800px; height: 500px"></video>
    <div v-for="(item, index) in list" :key="index">
      <img :src="item.img" alt="" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { Combinator, MP4Clip } from '@webav/av-cliper'
import { onMounted, ref } from 'vue'
import video from '@/assets/video2.mp4'
const list = ref<any[]>([])
onMounted(async () => {
  console.log(await Combinator.isSupported())
})
const start = async () => {
  const clip = new MP4Clip((await fetch(video)).body!)
  await clip.ready
  list.value = (await clip.thumbnails()).map((it) => ({
    ts: it.ts,
    img: URL.createObjectURL(it.img)
  }))
}
</script>
