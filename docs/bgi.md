---
outline: BetterGI
---

# BetterGI · 更好的原神

<script setup>
import { ref, onMounted } from 'vue'

// hold the StreamSaver module once loaded
const streamSaver = ref(null)
const progress = ref(0)         // 进度百分比 0–100
const isDownloading = ref(true)

onMounted(async () => {
  if (typeof window !== 'undefined') {
    const module = await import('streamsaver')
    streamSaver.value = module.default
  }
})

function incrementStringOfLetters(str) {
  const chars = str.split('')
  let i = chars.length - 1
  while (i >= 0) {
    const code = chars[i].charCodeAt(0)
    if (code === 122 /* 'z' */) {
      chars[i] = 'a'
      i--
    } else {
      chars[i] = String.fromCharCode(code + 1)
      break
    }
  }
  if (i < 0) chars.unshift('a')
  return chars.join('')
}

async function downloadChunks(basePath, totalChunks) {
  if (!streamSaver.value) {
    console.warn('StreamSaver 未初始化')
    return
  }

  isDownloading.value = true
  progress.value = 0

  const fileStream = streamSaver.value.createWriteStream(basePath)
  const writer = fileStream.getWriter()
  let suffix = 'aa'

  for (let i = 0; i < totalChunks; i++) {
    const url = `${basePath}.part.${suffix}`
    console.log(`下载第 ${i + 1} / ${totalChunks} 片：`, url)

    const res = await fetch(url)
    if (!res.ok) {
      throw new Error(`下载失败：${url} → ${res.status}`)
    }

    const reader = res.body.getReader()
    while (true) {
      const { value, done } = await reader.read()
      if (done) break
      await writer.write(value)
    }

    suffix = incrementStringOfLetters(suffix)
    // 按“片”更新百分比
    progress.value = Math.floor(((i + 1) / totalChunks) * 100)
  }

  await writer.close()
  console.log('全部写入完毕！')
  isDownloading.value = false
}
</script>

<!-- 任何元素都能绑定点击 -->

## <a href="https://bettergi.com/"> 官方网站 </a>

## <a href="#" @click.prevent="downloadChunks('/static/BetterGI.Install.0.47.0.exe', 11)"> 立即下载（推荐） </a>

<!-- 显示进度条 -->
<div style="margin-top: 1em;">
    <!-- 加了高度，去掉了百分比文字 -->
    <progress
      :value="progress"
      max="100"
      style="width: 100%; height: 30px; display: block;"
    ></progress>
  </div>
