<template>
  <div class="w-full h-full flex flex-col">
    <!-- Header -->
    <div class="p-2 px-4 border-b border-slate-700/50 bg-slate-800/30 flex justify-between items-center z-10 shrink-0">
      <span class="text-[11px] font-bold tracking-widest uppercase text-slate-400">数字分身</span>
      <div class="flex items-center gap-2">
        <span class="w-2 h-2 rounded-full transition-colors duration-500" :class="statusColorClass"></span>
        <span class="text-[10px] uppercase font-bold" :class="statusTextClass">{{ statusLabel }}</span>
      </div>
    </div>

    <!-- Canvas -->
    <div class="flex-1 relative bg-slate-950 flex items-center justify-center overflow-hidden">
      <!-- Grid Background -->
      <div class="absolute inset-0 opacity-20" style="background-image: radial-gradient(#38bdf8 1px, transparent 1px); background-size: 20px 20px;"></div>

      <!-- Animated rings (active during thinking/speaking) -->
      <div v-if="status !== 'idle'" class="absolute w-48 h-48 rounded-full border border-cyan-500/20 animate-ping" style="animation-duration: 2.5s;"></div>
      <div v-if="status !== 'idle'" class="absolute w-40 h-40 rounded-full border border-cyan-500/40 animate-ping" style="animation-duration: 2s; animation-delay: 0.5s"></div>

      <!-- Avatar Core -->
      <div class="relative w-32 h-32 rounded-full bg-gradient-to-t from-cyan-900 to-cyan-400 border-4 border-cyan-400/30 flex items-center justify-center shadow-[0_0_50px_rgba(34,211,238,0.3)] z-10 transition-transform duration-500" :class="{'scale-110': status === 'speaking' || status === 'thinking'}">
         <svg class="w-16 h-16 text-slate-900 transition-colors duration-300" fill="currentColor" viewBox="0 0 24 24">
           <path d="M12,2A10,10 0 0,0 2,12A10,10 0 0,0 12,22A10,10 0 0,0 22,12A10,10 0 0,0 12,2M12,4A8,8 0 0,1 20,12A8,8 0 0,1 12,20A8,8 0 0,1 4,12A8,8 0 0,1 12,4M12,6A6,6 0 0,0 6,12A6,6 0 0,0 12,18A6,6 0 0,0 18,12A6,6 0 0,0 12,6Z"></path>
         </svg>
      </div>

      <!-- Bottom Data overlay -->
      <div v-if="status !== 'idle'" class="absolute bottom-4 left-4 right-4 bg-black/60 backdrop-blur-md p-2 rounded border border-white/10 z-20">
        <div class="h-1 bg-slate-700 w-full rounded-full overflow-hidden">
          <div class="h-full bg-cyan-500 animate-pulse" :style="{ width: status === 'speaking' ? '65%' : '30%' }"></div>
        </div>
        <div class="flex justify-between mt-1 text-[9px] text-slate-400 font-mono tracking-widest uppercase">
          <span>语音引擎: {{ status === 'speaking' ? '工作中' : '待机中' }}</span>
          <span>音量: {{ status === 'speaking' ? '-14.2dB' : '-60.0dB' }}</span>
        </div>
      </div>
    </div>

    <!-- Hidden audio API element -->
    <audio ref="audioPlayer" class="hidden"></audio>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'

const props = defineProps<{
  status: string // 'idle' | 'thinking' | 'speaking'
}>()

const statusColorClass = computed(() => {
  if (props.status === 'idle') return 'bg-green-500' // green
  if (props.status === 'thinking') return 'bg-yellow-500' // yellow
  if (props.status === 'speaking') return 'bg-cyan-500' // cyan
  return 'bg-slate-500'
})

const statusTextClass = computed(() => {
  if (props.status === 'idle') return 'text-green-400'
  if (props.status === 'thinking') return 'text-yellow-400'
  if (props.status === 'speaking') return 'text-cyan-400'
  return 'text-slate-400'
})

const statusLabel = computed(() => {
  if (props.status === 'idle') return '空闲'
  if (props.status === 'thinking') return '检索中'
  if (props.status === 'speaking') return '正在讲话'
  return '未知'
})

const audioPlayer = ref<HTMLAudioElement | null>(null)
</script>
