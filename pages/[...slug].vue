<template>
  <main>
    <UContainer>
      <div class="flex mb-12" v-for="(page, index) in data">
        <div class="flex-none w-48 h-48 relative">
          <img :src="page.image" class="absolute inset-0 w-full h-full object-cover" />
        </div>
        <div class="flex-auto p6">
          <div class="flex flex-wrap">
            <h1 class="flex-auto font-medium text-slate-800">
              Relaxing Bear
            </h1>
            <div class="w-full flex-none mt-2 order-1 text-3xl text-slate-950">
              {{ page.title }}
            </div>
            <div class="w-full flex-none mt-2 order-2 relative" :id="`waveform-${index}`" v-if="page.audioFile">
              <div :id="`waveform-${index}-current-time`" class="absolute z-10 left-0 top-1/2 bg-slate-700 font-thin text-slate-200 px-1 py-0">0:00</div>
              <div :id="`waveform-${index}-duration`" class="absolute z-10 right-0 top-1/2 bg-slate-700 font-thin text-slate-200 px-1 py-0">0:00</div>
            </div>
          </div>
        </div>
      </div>
    </UContainer>
  </main>
</template>

<script setup>
import WaveSurfer from 'wavesurfer.js'
import Hover from 'wavesurfer.js/dist/plugins/hover'
const { data } = await useAsyncData('home', () => queryContent('/shows').where({_partial: false}).find())

function formatTime(seconds) {
  const minutes = Math.floor(seconds / 60)
  const secondsRemainder = Math.round(seconds) % 60
  const paddedSeconds = `0${secondsRemainder}`.slice(-2)
  return `${minutes}:${paddedSeconds}`
}

function createWaveform(audioFile, container, waveformData) {
  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')
  // Define the waveform gradient
  const gradient = ctx.createLinearGradient(0, 0, 0, canvas.height * 1.35)
  gradient.addColorStop(0, '#656666') // Top color
  gradient.addColorStop((canvas.height * 0.7) / canvas.height, '#656666') // Top color
  gradient.addColorStop((canvas.height * 0.7 + 1) / canvas.height, '#ffffff') // White line
  gradient.addColorStop((canvas.height * 0.7 + 2) / canvas.height, '#ffffff') // White line
  gradient.addColorStop((canvas.height * 0.7 + 3) / canvas.height, '#B1B1B1') // Bottom color
  gradient.addColorStop(1, '#B1B1B1') // Bottom color

  // Define the progress gradient
  const progressGradient = ctx.createLinearGradient(0, 0, 0, canvas.height * 1.35)
  progressGradient.addColorStop(0, '#EE772F') // Top color
  progressGradient.addColorStop((canvas.height * 0.7) / canvas.height, '#EB4926') // Top color
  progressGradient.addColorStop((canvas.height * 0.7 + 1) / canvas.height, '#ffffff') // White line
  progressGradient.addColorStop((canvas.height * 0.7 + 2) / canvas.height, '#ffffff') // White line
  progressGradient.addColorStop((canvas.height * 0.7 + 3) / canvas.height, '#F6B094') // Bottom color
  progressGradient.addColorStop(1, '#F6B094') // Bottom color

  const wavesurfer = WaveSurfer.create({
    container: `#${container}`,
    waveColor: gradient,
    progressColor: progressGradient,
    barWidth: 2,
    url: audioFile,
    peaks: waveformData,
    plugins: [
      Hover.create({
        lineColor: '#ff0000',
        lineWidth: 2,
        labelBackground: '#555',
        labelColor: '#fff',
        labelSize: '11px',
      })
    ]
  })

  // Play/pause on click
  wavesurfer.on('interaction', () => {
    wavesurfer.playPause()
  })

  const durationEl = document.querySelector(`#${container}-duration`)
  const currentTimeEl = document.querySelector(`#${container}-current-time`)

  wavesurfer.on('decode', (duration) => {
    // Set the duration of the waveform
    durationEl.textContent = formatTime(duration)
  });

  wavesurfer.on('timeupdate', (currentTime) => {
    // Set the current time of the waveform
    currentTimeEl.textContent = formatTime(currentTime)
  });
}

onMounted(async () => {
  data.value.forEach((page, index) => {
    if (page.audioFile) {
      createWaveform(page.audioFile, `waveform-${index}`, page.waveformData)
    }
  })
})

</script>
