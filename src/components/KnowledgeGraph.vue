<template>
  <div class="w-full h-full flex flex-col">
     <div class="p-2 px-4 border-b border-slate-700/50 bg-slate-800/30 flex justify-between items-center shrink-0 z-10">
        <span class="text-[11px] font-bold tracking-widest uppercase text-slate-400">知识图谱实时可视化</span>
        <span class="text-[10px] text-slate-500 uppercase font-mono">实时更新中</span>
     </div>
     
     <div class="flex-1 w-full relative overflow-hidden bg-slate-900/40">
       <div ref="chartRef" class="absolute inset-0"></div>
     </div>
     
     <div class="p-3 bg-slate-800/50 border-t border-slate-700/50 flex gap-4 shrink-0 z-10">
        <div class="flex-1">
           <div class="flex justify-between items-center mb-1">
             <div class="text-[9px] text-slate-500 uppercase font-bold tracking-wider">置信度</div>
             <div class="text-[9px] text-cyan-400 font-mono">{{ confidence ?? 94 }}%</div>
           </div>
           <div class="h-1 bg-slate-700 rounded-full overflow-hidden">
              <div class="h-full bg-cyan-400 transition-all duration-500 ease-out" :style="{ width: `${confidence ?? 94}%` }"></div>
           </div>
        </div>
        <div class="flex-1">
           <div class="flex justify-between items-center mb-1">
             <div class="text-[9px] text-slate-500 uppercase font-bold tracking-wider">相关性</div>
             <div class="text-[9px] text-indigo-400 font-mono">{{ relevance ?? 82 }}%</div>
           </div>
           <div class="h-1 bg-slate-700 rounded-full overflow-hidden">
              <div class="h-full bg-indigo-400 transition-all duration-500 ease-out" :style="{ width: `${relevance ?? 82}%` }"></div>
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
    nodes: Array<{id: string, name: string, category?: string}>
    links: Array<{source: string, target: string, label?: string}>
  } | null
  confidence?: number
  relevance?: number
}>()

const chartRef = ref<HTMLElement | null>(null)
const myChart = shallowRef<echarts.ECharts | null>(null)

const getBaseOption = () => {
  return {
    backgroundColor: 'transparent',
    legend: {
      show: true,
      data: ['核心实体', '关联知识'],
      bottom: 10,
      left: 10,
      textStyle: {
        color: '#94a3b8',
        fontSize: 10,
        fontFamily: 'monospace',
        fontWeight: 'bold'
      },
      itemWidth: 10,
      itemHeight: 10,
      icon: 'circle'
    },
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
        categories: [
          {
            name: '核心实体',
            symbolSize: 60,
            itemStyle: { color: '#0891b2', borderColor: '#22d3ee', borderWidth: 2, shadowBlur: 20, shadowColor: '#22d3ee' }
          },
          {
            name: '关联知识',
            symbolSize: 40,
            itemStyle: { color: '#1e1b4b', borderColor: '#818cf8', borderWidth: 1 }
          }
        ],
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
    { name: '机器学习', category: 0 },
    { name: '支持向量机', category: 1 },
    { name: '经验风险', category: 1 },
    { name: '深度学习', category: 1 },
    { name: '梯度下降', category: 1 }
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
    // If backend provides category use it, else fallback to primary/neighbor based on index
    const isCore = node.category === 'core' || (!node.category && index === 0);
    return {
      id: node.id || node.name,
      name: node.name || node.id,
      category: isCore ? 0 : 1,
    }
  })

  // Filter links to ensure both source and target exist to prevent ECharts crash
  const validNodeIds = new Set(formattedNodes.map(n => n.id))
  const validNodeNames = new Set(formattedNodes.map(n => n.name))
  
  const formattedLinks = (newData.links || []).filter(link => {
    const sourceExists = validNodeIds.has(link.source) || validNodeNames.has(link.source)
    const targetExists = validNodeIds.has(link.target) || validNodeNames.has(link.target)
    if (!sourceExists || !targetExists) {
      console.warn(`Link omitted due to missing source/target:`, link)
    }
    return sourceExists && targetExists
  }).map(link => ({
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
