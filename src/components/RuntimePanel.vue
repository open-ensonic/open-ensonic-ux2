<template>
  <div class="runtime-panel">
    <div class="runtime-header">
      <h4>OpenEnsonic 运行时</h4>
      <div class="runtime-status">
        <el-tag v-if="isRunning" type="success" size="small">
          <el-icon><Loading /></el-icon>
          运行中
        </el-tag>
        <el-tag v-else type="info" size="small">已停止</el-tag>
      </div>
    </div>
    
    <el-tabs v-model="activeTab" type="card">
      <!-- 性能监控 -->
      <el-tab-pane label="性能监控" name="performance">
        <div class="performance-section">
          <div class="metrics-grid">
            <div class="metric-card">
              <div class="metric-header">
                <el-icon><Cpu /></el-icon>
                <span>CPU 使用率</span>
              </div>
              <div class="metric-value">{{ cpuUsage }}%</div>
              <div class="metric-chart">
                <div class="progress-bar">
                  <div 
                    class="progress-fill cpu" 
                    :style="{ width: cpuUsage + '%' }"
                  ></div>
                </div>
              </div>
            </div>
            
            <div class="metric-card">
              <div class="metric-header">
                <el-icon><Monitor /></el-icon>
                <span>内存使用</span>
              </div>
              <div class="metric-value">{{ memoryUsage }} MB</div>
              <div class="metric-chart">
                <div class="progress-bar">
                  <div 
                    class="progress-fill memory" 
                    :style="{ width: (memoryUsage / 1024) * 100 + '%' }"
                  ></div>
                </div>
              </div>
            </div>
            
            <div class="metric-card">
              <div class="metric-header">
                <el-icon><TrendCharts /></el-icon>
                <span>数据吞吐量</span>
              </div>
              <div class="metric-value">{{ throughput }} MB/s</div>
              <div class="metric-chart">
                <div class="progress-bar">
                  <div 
                    class="progress-fill throughput" 
                    :style="{ width: Math.min(throughput * 10, 100) + '%' }"
                  ></div>
                </div>
              </div>
            </div>
            
            <div class="metric-card">
              <div class="metric-header">
                <el-icon><Timer /></el-icon>
                <span>运行时间</span>
              </div>
              <div class="metric-value">{{ formatRuntime(runtime) }}</div>
              <div class="metric-info">
                启动时间: {{ startTime }}
              </div>
            </div>
          </div>
          
          <div class="performance-chart">
            <div class="chart-header">
              <span>实时性能图表</span>
              <el-button-group size="small">
                <el-button @click="resetChart">重置</el-button>
                <el-button @click="exportChart">导出</el-button>
              </el-button-group>
            </div>
            <div class="chart-content">
              <canvas ref="performanceChart" width="800" height="300"></canvas>
            </div>
          </div>
        </div>
      </el-tab-pane>
      
      <!-- 模块状态 -->
      <el-tab-pane label="模块状态" name="modules">
        <div class="modules-section">
          <div class="modules-header">
            <span>活动模块 ({{ activeModules.length }})</span>
            <el-button size="small" @click="refreshModules">
              <el-icon><Refresh /></el-icon>
              刷新
            </el-button>
          </div>
          
          <el-table :data="activeModules" size="small" height="400">
            <el-table-column prop="name" label="模块名称" width="150" />
            <el-table-column prop="type" label="类型" width="100">
              <template #default="{ row }">
                <el-tag :type="getModuleTypeColor(row.type)" size="small">
                  {{ getModuleTypeName(row.type) }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column prop="status" label="状态" width="80">
              <template #default="{ row }">
                <el-tag 
                  :type="row.status === 'running' ? 'success' : 'danger'" 
                  size="small"
                >
                  {{ row.status === 'running' ? '运行' : '停止' }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column prop="cpuUsage" label="CPU %" width="80" />
            <el-table-column prop="memoryUsage" label="内存 (MB)" width="100" />
            <el-table-column prop="throughput" label="吞吐量" width="100" />
            <el-table-column label="操作" width="120">
              <template #default="{ row }">
                <el-button-group size="small">
                  <el-button @click="inspectModule(row)">
                    <el-icon><View /></el-icon>
                  </el-button>
                  <el-button @click="restartModule(row)">
                    <el-icon><Refresh /></el-icon>
                  </el-button>
                </el-button-group>
              </template>
            </el-table-column>
          </el-table>
        </div>
      </el-tab-pane>
      
      <!-- 配置管理 -->
      <el-tab-pane label="配置管理" name="config">
        <div class="config-section">
          <el-form :model="runtimeConfig" label-width="120px" size="small">
            <el-form-item label="缓冲区大小">
              <el-input-number 
                v-model="runtimeConfig.bufferSize" 
                :min="1024" 
                :max="65536" 
                :step="1024"
              />
              <span class="form-help">样本数，影响延迟和性能</span>
            </el-form-item>
            
            <el-form-item label="线程数">
              <el-input-number 
                v-model="runtimeConfig.threadCount" 
                :min="1" 
                :max="16" 
              />
              <span class="form-help">并行处理线程数量</span>
            </el-form-item>
            
            <el-form-item label="采样率">
              <el-select v-model="runtimeConfig.sampleRate">
                <el-option label="8 kHz" :value="8000" />
                <el-option label="16 kHz" :value="16000" />
                <el-option label="32 kHz" :value="32000" />
                <el-option label="44.1 kHz" :value="44100" />
                <el-option label="48 kHz" :value="48000" />
                <el-option label="96 kHz" :value="96000" />
              </el-select>
            </el-form-item>
            
            <el-form-item label="实时优先级">
              <el-switch v-model="runtimeConfig.realtimePriority" />
              <span class="form-help">启用实时调度优先级</span>
            </el-form-item>
            
            <el-form-item label="性能监控">
              <el-switch v-model="runtimeConfig.performanceMonitoring" />
              <span class="form-help">启用详细性能监控</span>
            </el-form-item>
            
            <el-form-item label="日志级别">
              <el-select v-model="runtimeConfig.logLevel">
                <el-option label="调试" value="debug" />
                <el-option label="信息" value="info" />
                <el-option label="警告" value="warn" />
                <el-option label="错误" value="error" />
              </el-select>
            </el-form-item>
            
            <el-form-item label="自动保存">
              <el-switch v-model="runtimeConfig.autoSave" />
              <span class="form-help">自动保存运行状态</span>
            </el-form-item>
            
            <el-form-item>
              <el-button type="primary" @click="applyConfig">应用配置</el-button>
              <el-button @click="resetConfig">重置配置</el-button>
              <el-button @click="exportConfig">导出配置</el-button>
            </el-form-item>
          </el-form>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, onUnmounted } from 'vue'
import {
  Loading, Cpu, Monitor, TrendCharts, Timer, Refresh, View
} from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus'

// 响应式数据
const activeTab = ref('performance')
const isRunning = ref(false)
const performanceChart = ref<HTMLCanvasElement>()

// 性能指标
const cpuUsage = ref(45)
const memoryUsage = ref(256)
const throughput = ref(8.5)
const runtime = ref(0)
const startTime = ref('')

// 活动模块
const activeModules = ref([
  {
    id: 'sig_source_1',
    name: '信号源',
    type: 'source',
    status: 'running',
    cpuUsage: 12,
    memoryUsage: 45,
    throughput: '2.1 MB/s'
  },
  {
    id: 'filter_1',
    name: '低通滤波器',
    type: 'filter',
    status: 'running',
    cpuUsage: 28,
    memoryUsage: 67,
    throughput: '2.1 MB/s'
  },
  {
    id: 'time_sink_1',
    name: '时域显示',
    type: 'gui',
    status: 'running',
    cpuUsage: 15,
    memoryUsage: 89,
    throughput: '2.1 MB/s'
  }
])

// 运行时配置
const runtimeConfig = reactive({
  bufferSize: 8192,
  threadCount: 4,
  sampleRate: 32000,
  realtimePriority: false,
  performanceMonitoring: true,
  logLevel: 'info',
  autoSave: true
})

// 性能监控定时器
let performanceTimer: number | null = null
let chartContext: CanvasRenderingContext2D | null = null
const performanceData: number[][] = [[], [], []] // CPU, Memory, Throughput

// 获取模块类型颜色
const getModuleTypeColor = (type: string) => {
  const colorMap: Record<string, string> = {
    source: 'success',
    filter: 'primary',
    analysis: 'warning',
    gui: 'info',
    custom: 'danger'
  }
  return colorMap[type] || 'info'
}

// 获取模块类型名称
const getModuleTypeName = (type: string) => {
  const nameMap: Record<string, string> = {
    source: '信号源',
    filter: '滤波器',
    analysis: '分析',
    gui: 'GUI',
    custom: '自定义'
  }
  return nameMap[type] || '未知'
}

// 格式化运行时间
const formatRuntime = (seconds: number) => {
  const hours = Math.floor(seconds / 3600)
  const minutes = Math.floor((seconds % 3600) / 60)
  const secs = seconds % 60
  
  if (hours > 0) {
    return `${hours}:${minutes.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
  } else {
    return `${minutes}:${secs.toString().padStart(2, '0')}`
  }
}

// 更新性能数据
const updatePerformanceData = () => {
  // 模拟性能数据变化
  cpuUsage.value = Math.max(0, Math.min(100, cpuUsage.value + (Math.random() - 0.5) * 10))
  memoryUsage.value = Math.max(100, Math.min(1024, memoryUsage.value + (Math.random() - 0.5) * 20))
  throughput.value = Math.max(0, Math.min(20, throughput.value + (Math.random() - 0.5) * 2))
  
  if (isRunning.value) {
    runtime.value++
  }
  
  // 更新图表数据
  performanceData[0].push(cpuUsage.value)
  performanceData[1].push(memoryUsage.value / 10) // 缩放内存数据
  performanceData[2].push(throughput.value * 5) // 缩放吞吐量数据
  
  // 保持数据点数量在合理范围内
  if (performanceData[0].length > 100) {
    performanceData[0].shift()
    performanceData[1].shift()
    performanceData[2].shift()
  }
  
  drawChart()
}

// 绘制性能图表
const drawChart = () => {
  if (!chartContext || !performanceChart.value) return
  
  const canvas = performanceChart.value
  const ctx = chartContext
  const width = canvas.width
  const height = canvas.height
  
  // 清空画布
  ctx.clearRect(0, 0, width, height)
  
  // 绘制网格
  ctx.strokeStyle = '#e4e7ed'
  ctx.lineWidth = 1
  
  // 垂直网格线
  for (let i = 0; i <= 10; i++) {
    const x = (width / 10) * i
    ctx.beginPath()
    ctx.moveTo(x, 0)
    ctx.lineTo(x, height)
    ctx.stroke()
  }
  
  // 水平网格线
  for (let i = 0; i <= 5; i++) {
    const y = (height / 5) * i
    ctx.beginPath()
    ctx.moveTo(0, y)
    ctx.lineTo(width, y)
    ctx.stroke()
  }
  
  // 绘制数据线
  const colors = ['#409eff', '#67c23a', '#e6a23c']
  const labels = ['CPU', 'Memory', 'Throughput']
  
  performanceData.forEach((data, index) => {
    if (data.length < 2) return
    
    ctx.strokeStyle = colors[index]
    ctx.lineWidth = 2
    ctx.beginPath()
    
    data.forEach((value, i) => {
      const x = (width / Math.max(data.length - 1, 1)) * i
      const y = height - (value / 100) * height
      
      if (i === 0) {
        ctx.moveTo(x, y)
      } else {
        ctx.lineTo(x, y)
      }
    })
    
    ctx.stroke()
  })
  
  // 绘制图例
  ctx.font = '12px Arial'
  colors.forEach((color, index) => {
    ctx.fillStyle = color
    ctx.fillRect(10 + index * 80, 10, 12, 12)
    ctx.fillStyle = '#303133'
    ctx.fillText(labels[index], 25 + index * 80, 21)
  })
}

// 初始化图表
const initChart = () => {
  if (!performanceChart.value) return
  
  chartContext = performanceChart.value.getContext('2d')
  if (chartContext) {
    drawChart()
  }
}

// 重置图表
const resetChart = () => {
  performanceData[0] = []
  performanceData[1] = []
  performanceData[2] = []
  drawChart()
  ElMessage.success('图表已重置')
}

// 导出图表
const exportChart = () => {
  if (!performanceChart.value) return
  
  const link = document.createElement('a')
  link.download = `performance-chart-${new Date().toISOString().slice(0, 19).replace(/:/g, '-')}.png`
  link.href = performanceChart.value.toDataURL()
  link.click()
  
  ElMessage.success('图表已导出')
}

// 刷新模块状态
const refreshModules = () => {
  // 模拟刷新模块状态
  activeModules.value.forEach(module => {
    module.cpuUsage = Math.floor(Math.random() * 50)
    module.memoryUsage = Math.floor(Math.random() * 100 + 50)
  })
  
  ElMessage.success('模块状态已刷新')
}

// 检查模块
const inspectModule = (module: any) => {
  ElMessage.info(`检查模块: ${module.name}`)
}

// 重启模块
const restartModule = (module: any) => {
  ElMessage.success(`模块 ${module.name} 已重启`)
}

// 应用配置
const applyConfig = () => {
  ElMessage.success('配置已应用')
}

// 重置配置
const resetConfig = () => {
  Object.assign(runtimeConfig, {
    bufferSize: 8192,
    threadCount: 4,
    sampleRate: 32000,
    realtimePriority: false,
    performanceMonitoring: true,
    logLevel: 'info',
    autoSave: true
  })
  ElMessage.success('配置已重置')
}

// 导出配置
const exportConfig = () => {
  const config = JSON.stringify(runtimeConfig, null, 2)
  const blob = new Blob([config], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'runtime-config.json'
  a.click()
  URL.revokeObjectURL(url)
  
  ElMessage.success('配置已导出')
}

// 组件挂载
onMounted(() => {
  startTime.value = new Date().toLocaleTimeString()
  isRunning.value = true
  
  // 初始化图表
  setTimeout(() => {
    initChart()
  }, 100)
  
  // 启动性能监控
  performanceTimer = window.setInterval(updatePerformanceData, 1000)
})

// 组件卸载
onUnmounted(() => {
  if (performanceTimer) {
    clearInterval(performanceTimer)
  }
})
</script>

<style scoped>
.runtime-panel {
  height: 100%;
  display: flex;
  flex-direction: column;
  padding: 16px;
}

.runtime-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.runtime-header h4 {
  margin: 0;
  color: #303133;
}

.performance-section {
  height: 100%;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.metrics-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 16px;
}

.metric-card {
  background: white;
  border: 1px solid #e4e7ed;
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.metric-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 8px;
  color: #606266;
  font-size: 13px;
}

.metric-value {
  font-size: 24px;
  font-weight: 600;
  color: #303133;
  margin-bottom: 8px;
}

.metric-chart {
  margin-bottom: 8px;
}

.progress-bar {
  width: 100%;
  height: 6px;
  background: #f5f7fa;
  border-radius: 3px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  border-radius: 3px;
  transition: width 0.3s ease;
}

.progress-fill.cpu {
  background: linear-gradient(90deg, #67c23a, #409eff);
}

.progress-fill.memory {
  background: linear-gradient(90deg, #409eff, #e6a23c);
}

.progress-fill.throughput {
  background: linear-gradient(90deg, #e6a23c, #f56c6c);
}

.metric-info {
  font-size: 12px;
  color: #909399;
}

.performance-chart {
  flex: 1;
  background: white;
  border: 1px solid #e4e7ed;
  border-radius: 8px;
  padding: 16px;
  min-height: 350px;
}

.chart-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  font-weight: 500;
  color: #303133;
}

.chart-content {
  display: flex;
  justify-content: center;
  align-items: center;
}

.modules-section {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.modules-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  font-weight: 500;
  color: #303133;
}

.config-section {
  padding: 16px;
  overflow-y: auto;
}

.form-help {
  margin-left: 8px;
  font-size: 12px;
  color: #909399;
}

.el-tabs {
  height: 100%;
}

.el-tabs :deep(.el-tabs__content) {
  height: calc(100% - 40px);
  overflow: auto;
}

.el-tabs :deep(.el-tab-pane) {
  height: 100%;
}
</style>