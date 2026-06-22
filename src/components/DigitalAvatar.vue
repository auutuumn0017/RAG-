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

      <!-- Avatar Core -->
      <div class="relative w-32 h-32 rounded-full flex items-center justify-center z-10 transition-all duration-500 avatar-container"
           :class="{
             'avatar-idle': status === 'idle',
             'avatar-speaking': status === 'speaking',
             'avatar-thinking-wrapper': status === 'thinking'
           }">
         <img :src="'/avatar.png'" alt="Digital Avatar" class="w-full h-full object-cover rounded-full bg-slate-800 border-4 border-transparent z-10 relative" />
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
  if (props.status === 'idle') return '🟢 空闲'
  if (props.status === 'thinking') return '🟡 思考检索中'
  if (props.status === 'speaking') return '📢 正在讲话'
  return '未知'
})

const audioPlayer = ref<HTMLAudioElement | null>(null)
</script>

<style scoped>
@keyframes pulse-glow {
  0%, 100% {
    box-shadow: 0 0 20px rgba(34, 211, 238, 0.4), 0 0 40px rgba(34, 211, 238, 0.2);
    transform: scale(1.05);
  }
  50% {
    box-shadow: 0 0 50px rgba(34, 211, 238, 0.8), 0 0 80px rgba(34, 211, 238, 0.6);
    transform: scale(1.15);
  }
}

.avatar-speaking {
  animation: pulse-glow 1.5s ease-in-out infinite;
}

.avatar-speaking img {
  border-color: #22d3ee;
}

@keyframes spin-border {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.avatar-thinking-wrapper::before {
  content: '';
  position: absolute;
  top: -4px; right: -4px; bottom: -4px; left: -4px;
  border-radius: 50%;
  border: 4px solid transparent;
  border-top-color: #eab308;
  border-right-color: #eab308;
  animation: spin-border 1s linear infinite;
  z-index: 0;
}

.avatar-thinking-wrapper img {
  border-color: rgba(234, 179, 8, 0.3);
}

.avatar-idle {
  box-shadow: 0 0 20px rgba(34, 197, 94, 0.2);
  transform: scale(1);
}

.avatar-idle img {
  border-color: rgba(34, 197, 94, 0.5);
}

.avatar-container {
  /* Prevent margin collapse and ensure absolute positioning of pseudo elements works */
  position: relative;
}
</style>
