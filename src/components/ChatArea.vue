<template>
  <div class="flex-1 flex flex-col h-full overflow-hidden">
    <!-- Messages Scroll Area -->
    <div class="flex-1 overflow-y-auto p-4 space-y-6" ref="chatContainer">
      <div v-for="msg in messages" :key="msg.id" class="flex flex-col gap-1" :class="msg.role === 'user' ? 'items-end' : 'items-start'">
        <span v-if="msg.role === 'user'" class="text-[10px] font-bold text-indigo-300 mr-1 uppercase">学生</span>
        <span v-else class="text-[10px] font-bold text-cyan-400 ml-1 uppercase">AI 导师</span>
        
        <div class="p-4 max-w-[90%] shadow-sm"
             :class="msg.role === 'user' ? 'bg-[#312e81] rounded-[12px_12px_0_12px] text-slate-100' : 'bg-[#1e293b] rounded-[0_12px_12px_12px] border-l-4 border-[#38bdf8] text-slate-200'">
          
          <!-- AI Think Process Container -->
          <details v-if="msg.think_process" class="mb-3 text-sm text-slate-400 [&_summary]:cursor-pointer group">
            <summary class="focus:outline-none opacity-80 hover:opacity-100 transition-opacity font-mono text-xs uppercase tracking-wider flex items-center gap-2">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"></path></svg>
              思考过程
            </summary>
            <div class="mt-2 p-2 bg-black/20 border border-slate-600/50 border-dashed rounded font-mono text-xs leading-relaxed text-slate-400 whitespace-pre-wrap">
              {{ msg.think_process }}
            </div>
          </details>

          <!-- Core Message Answer -->
          <div class="text-sm leading-relaxed whitespace-pre-wrap">
            {{ msg.answer || msg.content }}
          </div>
        </div>
      </div>
    </div>

    <!-- Input Area -->
    <div class="p-4 border-t border-slate-700/50 bg-slate-900/50">
      <div class="flex gap-3">
        <el-input
          v-model="inputQuery"
          type="textarea"
          :rows="3"
          placeholder="请输入您的问题... (按 Enter 发送，Shift+Enter 换行)"
          resize="none"
          @keydown.enter="handleEnter"
          class="custom-textarea flex-1 w-auto"
        />
        <button class="bg-cyan-600 hover:bg-cyan-500 text-white px-6 rounded-lg font-bold text-sm transition-colors uppercase tracking-widest h-20 shrink-0" @click="sendMessage" :disabled="isWaiting">
          发送
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, nextTick, watch } from 'vue'
import axios from 'axios'

const emit = defineEmits(['status-change', 'update-graph'])

interface Message {
  id: number
  role: 'user' | 'ai'
  content?: string
  answer?: string
  think_process?: string
}

const messages = ref<Message[]>([
  {
    id: 1,
    role: 'ai',
    answer: '你好！我是你的专属数字教师。右侧会实时更新我的知识图谱与状态。今天我们想学习什么内容？'
  }
])
const inputQuery = ref('')
const chatContainer = ref<HTMLElement | null>(null)
const isWaiting = ref(false)

const scrollToBottom = async () => {
  await nextTick()
  if (chatContainer.value) {
    chatContainer.value.scrollTop = chatContainer.value.scrollHeight
  }
}

watch(messages, scrollToBottom, { deep: true })

const handleEnter = (e: KeyboardEvent) => {
  if (!e.shiftKey) {
    e.preventDefault()
    sendMessage()
  }
}

const sendMessage = async () => {
  const query = inputQuery.value.trim()
  if (!query || isWaiting.value) return

  // 1. Push user message
  messages.value.push({
    id: Date.now(),
    role: 'user',
    content: query
  })
  
  inputQuery.value = ''
  isWaiting.value = true
  emit('status-change', 'thinking')

  // 2. Perform API request to standard localhost:8000
  try {
    const res = await axios.post('http://127.0.0.1:8000/api/chat', { query })
    
    emit('status-change', 'speaking')
    messages.value.push({
      id: Date.now(),
      role: 'ai',
      answer: res.data.answer || res.data.content || '未获取到回答。',
      think_process: res.data.think_process
    })

    if (res.data.graph_data) {
      emit('update-graph', res.data.graph_data)
    }
  } catch (error: any) {
    console.error('API Error:', error)
    
    // Simulate robust fallback if the developer's local backend isn't up
    emit('status-change', 'speaking')
    messages.value.push({
      id: Date.now(),
      role: 'ai',
      answer: `(网络请求失败：${error.message})\n由于本地测试后端 http://127.0.0.1:8000 未启动，这是应用生成的演示回答。\n\n机器学习是一门多领域交叉学科，涉及概率论、统计学、逼近论、凸分析、算法复杂度理论等多门学科。专门研究计算机怎样模拟或实现人类的学习行为，以获取新的知识或技能...`,
      think_process: '检测到目标本地接口无法连接（Connection Refused / CORS）。启动前端 Mock 数据作为备用，确保教学体验连续性，并在回答中标注连接失败的状态以便开发人员调试。'
    })

    // Mock graph data on failure
    emit('update-graph', {
      nodes: [
        { id: '机器学习', name: '机器学习' },
        { id: '深度学习', name: '深度学习' }
      ],
      links: [
        { source: '机器学习', target: '深度学习', label: '包含' }
      ]
    })
  } finally {
    isWaiting.value = false
    // Restore IDLE status after "speaking" simulation period
    setTimeout(() => {
      emit('status-change', 'idle')
    }, 4000)
  }
}
</script>

<style scoped>
:deep(.custom-textarea .el-textarea__inner) {
  background-color: #1e293b;
  border-color: #334155;
  color: #e2e8f0;
  font-family: inherit;
  font-size: 14px;
  line-height: 1.5;
  height: 5rem; /* h-20 */
}
:deep(.custom-textarea .el-textarea__inner:focus) {
  border-color: #06b6d4; /* cyan-500 */
  box-shadow: none;
}
:deep(.custom-textarea .el-textarea__inner::placeholder) {
  color: #64748b; /* slate-500 */
}
</style>
