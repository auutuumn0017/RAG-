<template>
  <div class="w-full h-full flex flex-col">
     <div class="p-2 px-4 border-b border-slate-700/50 bg-slate-800/30 flex justify-between items-center shrink-0 z-10">
        <span class="text-[11px] font-bold tracking-widest uppercase text-slate-400">知识图谱实时可视化</span>
        <span class="text-[10px] text-slate-500 uppercase font-mono">实时更新中</span>
     </div>
     
     <div class="flex-1 w-full relative overflow-hidden bg-slate-900/40">
       <div ref="chartRef" class="absolute inset-0"></div>
       
       <div class="absolute bottom-4 left-4 space-y-1 pointer-events-none z-10">
          <div class="flex items-center gap-2 text-[9px]">
             <div class="w-2 h-2 bg-cyan-500 rounded-full"></div>
             <span class="text-slate-300 uppercase font-mono font-bold tracking-wider">核心实体</span>
          </div>
          <div class="flex items-center gap-2 text-[9px]">
             <div class="w-2 h-2 bg-indigo-500 rounded-full"></div>
             <span class="text-slate-300 uppercase font-mono font-bold tracking-wider">当前主题</span>
          </div>
       </div>
     </div>
     
     <div class="p-3 bg-slate-800/50 border-t border-slate-700/50 flex gap-4 shrink-0 z-10">
        <div class="flex-1">
           <div class="text-[9px] text-slate-500 uppercase font-bold tracking-wider">置信度</div>
           <div class="h-1 bg-slate-700 mt-1 rounded-full overflow-hidden">
              <div class="h-full bg-cyan-400 w-[94%]"></div>
           </div>
        </div>
        <div class="flex-1">
           <div class="text-[9px] text-slate-500 uppercase font-bold tracking-wider">相关性</div>
           <div class="h-1 bg-slate-700 mt-1 rounded-full overflow-hidden">
              <div class="h-full bg-indigo-400 w-[82%]"></div>
           </div>
        </div>
     </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, shallowRef, watch } from 'vue'
import * as echarts from 'echarts'

const props = defineProps<{
  graphData?: {
    nodes: Array<{id: string, name: string}>
    links: Array<{source: string, target: string, label?: string}>
  } | null
}>()

const chartRef = ref<HTMLElement | null>(null)
const myChart = shallowRef<echarts.ECharts | null>(null)

const getBaseOption = () => {
  return {
    backgroundColor: 'transparent',
    tooltip: {
      show: true,
      theme: 'dark',
      backgroundColor: 'rgba(30, 41, 59, 0.9)',
      borderColor: '#334155',
      textStyle: { color: '#f8fafc', fontSize: 12 },
      formatter: (params: any) => {
        if (params.dataType === 'node') {
          return `<div class="font-bold font-mono">${params.data.name}</div>`
        }
        return `<div class="font-mono text-xs">${params.data.source} &gt; ${params.data.value || ''} &gt; ${params.data.target}</div>`
      }
    },
    series: [
      {
        type: 'graph',
        layout: 'force',
        data: [],
        links: [],
        roam: true,
        label: {
          show: true,
          position: 'right',
          color: '#94a3b8',
          fontSize: 10,
          fontWeight: 'bold',
          fontFamily: 'monospace'
        },
        edgeSymbol: ['none', 'arrow'],
        edgeSymbolSize: [4, 8],
        edgeLabel: {
          show: true,
          fontSize: 9,
          formatter: '{c}',
          color: '#64748b',
          fontFamily: 'monospace'
        },
        force: {
          repulsion: 300,
          edgeLength: [60, 120],
          friction: 0.2
        },
        lineStyle: {
          color: '#475569',
          width: 1,
          curveness: 0.1,
          opacity: 0.8
        }
      }
    ]
  }
}

onMounted(() => {
  if (!chartRef.value) return

  myChart.value = echarts.init(chartRef.value)

  // Initial Graph Data
  const data = [
    { name: '机器学习', symbolSize: 60, category: 0, itemStyle: { color: '#0891b2', borderColor: '#22d3ee', borderWidth: 2, shadowBlur: 20, shadowColor: '#22d3ee' } },
    { name: '支持向量机', symbolSize: 45, category: 1, itemStyle: { color: '#1e1b4b', borderColor: '#818cf8', borderWidth: 2 } },
    { name: '经验风险', symbolSize: 35, category: 1, itemStyle: { color: '#1e293b', borderColor: '#38bdf8', borderWidth: 1 } },
    { name: '深度学习', symbolSize: 50, category: 2, itemStyle: { color: '#1e293b', borderColor: '#38bdf8', borderWidth: 1 } },
    { name: '梯度下降', symbolSize: 40, category: 2, itemStyle: { color: '#1e293b', borderColor: '#475569', borderWidth: 1 } }
  ]

  const links = [
    { source: '机器学习', target: '支持向量机', value: '包含' },
    { source: '机器学习', target: '深度学习', value: '包含' },
    { source: '支持向量机', target: '经验风险', value: '最小化' },
    { source: '深度学习', target: '梯度下降', value: '依赖' },
    { source: '机器学习', target: '梯度下降', value: '优化' }
  ]

  const option = getBaseOption()
  option.series[0].data = data as any
  option.series[0].links = links as any

  myChart.value.setOption(option)

  window.addEventListener('resize', handleResize)
})

watch(() => props.graphData, (newData) => {
  if (!myChart.value) return

  if (!newData || !newData.nodes || newData.nodes.length === 0) {
    myChart.value.clear()
    myChart.value.setOption({
      title: {
        text: '未命中图谱节点',
        left: 'center',
        top: 'center',
        textStyle: {
          color: '#64748b',
          fontSize: 12,
          fontFamily: 'monospace'
        }
      }
    })
    return
  }

  const formattedNodes = newData.nodes.map((node, index) => {
    const isPrimary = index === 0
    return {
      name: node.name || node.id,
      symbolSize: isPrimary ? 60 : 40,
      itemStyle: {
        color: isPrimary ? '#0891b2' : '#1e1b4b',
        borderColor: isPrimary ? '#22d3ee' : '#818cf8',
        borderWidth: isPrimary ? 2 : 1,
        shadowBlur: isPrimary ? 20 : 0,
        shadowColor: isPrimary ? '#22d3ee' : 'transparent'
      }
    }
  })

  const formattedLinks = (newData.links || []).map(link => ({
    source: link.source,
    target: link.target,
    value: link.label || ''
  }))

  myChart.value.clear()
  const option = getBaseOption()
  option.series[0].data = formattedNodes as any
  option.series[0].links = formattedLinks as any
  myChart.value.setOption(option)
}, { deep: true })

const handleResize = () => {
  myChart.value?.resize()
}

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})
</script>
