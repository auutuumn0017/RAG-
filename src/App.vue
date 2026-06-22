<template>
  <div class="h-screen w-screen flex p-4 gap-4 bg-gradient-to-br from-[#1e293b] to-[#0f172a] text-slate-50 font-sans overflow-hidden">
    <!-- Left: Chat Area (60%) -->
    <div class="w-3/5 h-full flex flex-col bg-[#1e293b]/70 border border-[#94a3b8]/10 rounded-xl backdrop-blur-md overflow-hidden shrink-0">
      <div class="p-4 border-b border-slate-700/50 flex flex-col bg-slate-800/30">
        <h1 class="text-sm font-bold tracking-wider uppercase text-cyan-100 flex items-center gap-3">
           <div class="w-3 h-3 bg-cyan-400 rounded-full"></div>
           专属数字教师大脑
        </h1>
      </div>
      <ChatArea @status-change="handleStatusChange" @update-graph="handleGraphUpdate" />
    </div>

    <!-- Right: Avatar & Graph (40%) -->
    <div class="w-2/5 h-full flex flex-col gap-4">
      <!-- Top Right: Avatar (40%) -->
      <div class="h-[40%] bg-[#1e293b]/70 border border-[#94a3b8]/10 rounded-xl backdrop-blur-md overflow-hidden flex flex-col">
        <DigitalAvatar :status="teacherStatus" />
      </div>
      <!-- Bottom Right: Graph (60%) -->
      <div class="h-[60%] bg-[#1e293b]/70 border border-[#94a3b8]/10 rounded-xl backdrop-blur-md overflow-hidden flex flex-col">
        <KnowledgeGraph :graph-data="currentGraphData" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import ChatArea from './components/ChatArea.vue'
import DigitalAvatar from './components/DigitalAvatar.vue'
import KnowledgeGraph from './components/KnowledgeGraph.vue'

const teacherStatus = ref('idle')
const currentGraphData = ref<any>(null)

const handleStatusChange = (status: string) => {
  teacherStatus.value = status
}

const handleGraphUpdate = (graphData: any) => {
  currentGraphData.value = graphData
}
</script>
