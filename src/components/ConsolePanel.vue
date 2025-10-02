<template>
  <div class="console-panel">
    <div class="console-header">
      <div class="console-controls">
        <el-button-group size="small">
          <el-button @click="clearConsole">
            <el-icon><Delete /></el-icon>
            清空
          </el-button>
          <el-button @click="saveConsole">
            <el-icon><Download /></el-icon>
            保存
          </el-button>
          <el-button @click="toggleAutoScroll">
            <el-icon><Lock /></el-icon>
            {{ autoScroll ? '锁定滚动' : '自动滚动' }}
          </el-button>
        </el-button-group>
        
        <el-select v-model="logLevel" size="small" style="width: 120px">
          <el-option label="全部" value="all" />
          <el-option label="调试" value="debug" />
          <el-option label="信息" value="info" />
          <el-option label="警告" value="warn" />
          <el-option label="错误" value="error" />
        </el-select>
      </div>
    </div>
    
    <div ref="consoleContent" class="console-content">
      <div
        v-for="(log, index) in filteredLogs"
        :key="index"
        class="log-entry"
        :class="[`log-${log.level}`, { 'log-highlight': log.highlight }]"
      >
        <span class="log-timestamp">{{ formatTime(log.timestamp) }}</span>
        <span class="log-level">{{ log.level.toUpperCase() }}</span>
        <span class="log-message">{{ log.message }}</span>
      </div>
      
      <div v-if="filteredLogs.length === 0" class="empty-console">
        <el-empty description="暂无日志信息" />
      </div>
    </div>
    
    <div class="console-input">
      <el-input
        v-model="commandInput"
        placeholder="输入命令..."
        @keyup.enter="executeCommand"
      >
        <template #prepend>$</template>
        <template #append>
          <el-button @click="executeCommand">执行</el-button>
        </template>
      </el-input>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, nextTick, onMounted } from 'vue'
import { Delete, Download, Lock } from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus'

// 响应式数据
const consoleContent = ref<HTMLElement>()
const logLevel = ref('all')
const autoScroll = ref(true)
const commandInput = ref('')

// 日志数据
const logs = ref([
  {
    timestamp: new Date(),
    level: 'info',
    message: 'OpenEnsonic 编辑器已启动',
    highlight: false
  },
  {
    timestamp: new Date(Date.now() - 1000),
    level: 'debug',
    message: '加载模块库完成',
    highlight: false
  },
  {
    timestamp: new Date(Date.now() - 2000),
    level: 'info',
    message: '画布初始化完成',
    highlight: false
  },
  {
    timestamp: new Date(Date.now() - 3000),
    level: 'warn',
    message: '检测到未连接的输出端口',
    highlight: false
  }
])

// 过滤后的日志
const filteredLogs = computed(() => {
  if (logLevel.value === 'all') {
    return logs.value
  }
  return logs.value.filter(log => log.level === logLevel.value)
})

// 格式化时间
const formatTime = (timestamp: Date) => {
  return timestamp.toLocaleTimeString('zh-CN', {
    hour12: false,
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit'
  })
}

// 添加日志
const addLog = (level: string, message: string, highlight = false) => {
  logs.value.push({
    timestamp: new Date(),
    level,
    message,
    highlight
  })
  
  if (autoScroll.value) {
    nextTick(() => {
      scrollToBottom()
    })
  }
}

// 滚动到底部
const scrollToBottom = () => {
  if (consoleContent.value) {
    consoleContent.value.scrollTop = consoleContent.value.scrollHeight
  }
}

// 清空控制台
const clearConsole = () => {
  logs.value = []
  ElMessage.success('控制台已清空')
}

// 保存控制台
const saveConsole = () => {
  const content = logs.value
    .map(log => `[${formatTime(log.timestamp)}] ${log.level.toUpperCase()}: ${log.message}`)
    .join('\n')
  
  const blob = new Blob([content], { type: 'text/plain' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `console-${new Date().toISOString().slice(0, 19).replace(/:/g, '-')}.log`
  a.click()
  URL.revokeObjectURL(url)
  
  ElMessage.success('控制台日志已保存')
}

// 切换自动滚动
const toggleAutoScroll = () => {
  autoScroll.value = !autoScroll.value
  if (autoScroll.value) {
    scrollToBottom()
  }
}

// 执行命令
const executeCommand = () => {
  if (!commandInput.value.trim()) return
  
  const command = commandInput.value.trim()
  addLog('info', `> ${command}`, true)
  
  // 模拟命令执行
  setTimeout(() => {
    switch (command.toLowerCase()) {
      case 'help':
        addLog('info', '可用命令: help, clear, status, version, exit')
        break
      case 'clear':
        clearConsole()
        break
      case 'status':
        addLog('info', '系统状态: 正常运行')
        addLog('info', '活动模块: 0')
        addLog('info', '内存使用: 45.2MB')
        break
      case 'version':
        addLog('info', 'OpenEnsonic v1.0.0')
        addLog('info', 'Build: 2024-01-01')
        break
      case 'exit':
        addLog('info', '退出命令行模式')
        break
      default:
        addLog('error', `未知命令: ${command}`)
        addLog('info', '输入 help 查看可用命令')
    }
  }, 100)
  
  commandInput.value = ''
}

// 组件挂载后的操作
onMounted(() => {
  // 模拟一些初始日志
  setTimeout(() => {
    addLog('info', '系统初始化完成')
  }, 1000)
  
  setTimeout(() => {
    addLog('debug', '开始监听用户操作')
  }, 2000)
})

// 暴露方法给父组件
defineExpose({
  addLog,
  clearConsole
})
</script>

<style scoped>
.console-panel {
  height: 100%;
  display: flex;
  flex-direction: column;
  background: #1e1e1e;
  color: #d4d4d4;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
}

.console-header {
  padding: 8px 16px;
  background: #2d2d30;
  border-bottom: 1px solid #3e3e42;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.console-controls {
  display: flex;
  align-items: center;
  gap: 12px;
}

.console-content {
  flex: 1;
  overflow-y: auto;
  padding: 8px 16px;
  font-size: 13px;
  line-height: 1.4;
}

.log-entry {
  display: flex;
  margin: 2px 0;
  padding: 2px 0;
  border-radius: 2px;
}

.log-entry.log-highlight {
  background: rgba(255, 255, 255, 0.05);
}

.log-timestamp {
  color: #808080;
  margin-right: 8px;
  min-width: 80px;
  font-size: 11px;
}

.log-level {
  margin-right: 8px;
  min-width: 50px;
  font-weight: bold;
  font-size: 11px;
}

.log-debug .log-level {
  color: #569cd6;
}

.log-info .log-level {
  color: #4ec9b0;
}

.log-warn .log-level {
  color: #dcdcaa;
}

.log-error .log-level {
  color: #f44747;
}

.log-message {
  flex: 1;
  word-break: break-all;
}

.log-debug .log-message {
  color: #9cdcfe;
}

.log-info .log-message {
  color: #d4d4d4;
}

.log-warn .log-message {
  color: #dcdcaa;
}

.log-error .log-message {
  color: #f44747;
}

.empty-console {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: #808080;
}

.console-input {
  padding: 8px 16px;
  background: #2d2d30;
  border-top: 1px solid #3e3e42;
}

.console-input :deep(.el-input-group__prepend) {
  background: #3c3c3c;
  color: #d4d4d4;
  border-color: #3e3e42;
}

.console-input :deep(.el-input__inner) {
  background: #3c3c3c;
  color: #d4d4d4;
  border-color: #3e3e42;
}

.console-input :deep(.el-input-group__append) {
  background: #007acc;
  border-color: #007acc;
}

.console-input :deep(.el-input-group__append .el-button) {
  background: transparent;
  color: white;
  border: none;
}

/* 滚动条样式 */
.console-content::-webkit-scrollbar {
  width: 8px;
}

.console-content::-webkit-scrollbar-track {
  background: #2d2d30;
}

.console-content::-webkit-scrollbar-thumb {
  background: #464647;
  border-radius: 4px;
}

.console-content::-webkit-scrollbar-thumb:hover {
  background: #5a5a5a;
}
</style>