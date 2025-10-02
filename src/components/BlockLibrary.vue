<template>
  <div class="block-library">
    <div class="library-header">
      <h3>模块库</h3>
      <el-input
        v-model="searchText"
        placeholder="搜索模块..."
        size="small"
        clearable
      >
        <template #prefix>
          <el-icon><Search /></el-icon>
        </template>
      </el-input>
    </div>

    <div class="library-content">
      <el-collapse v-model="activeCategories" accordion>
        <!-- 信号源模块 -->
        <el-collapse-item title="信号源模块" name="sources">
          <template #title>
            <el-icon><Cpu /></el-icon>
            <span>信号源模块</span>
          </template>
          <div class="block-category">
            <div
              v-for="block in filteredBlocks.sources"
              :key="block.id"
              class="block-item"
              draggable="true"
              @dragstart="handleDragStart($event, block)"
            >
              <el-icon><component :is="block.icon" /></el-icon>
              <span>{{ block.name }}</span>
            </div>
          </div>
        </el-collapse-item>

        <!-- 滤波模块 -->
        <el-collapse-item title="滤波模块" name="filters">
          <template #title>
            <el-icon><Filter /></el-icon>
            <span>滤波模块</span>
          </template>
          <div class="block-category">
            <div
              v-for="block in filteredBlocks.filters"
              :key="block.id"
              class="block-item"
              draggable="true"
              @dragstart="handleDragStart($event, block)"
            >
              <el-icon><component :is="block.icon" /></el-icon>
              <span>{{ block.name }}</span>
            </div>
          </div>
        </el-collapse-item>

        <!-- 分析模块 -->
        <el-collapse-item title="分析模块" name="analysis">
          <template #title>
            <el-icon><TrendCharts /></el-icon>
            <span>分析模块</span>
          </template>
          <div class="block-category">
            <div
              v-for="block in filteredBlocks.analysis"
              :key="block.id"
              class="block-item"
              draggable="true"
              @dragstart="handleDragStart($event, block)"
            >
              <el-icon><component :is="block.icon" /></el-icon>
              <span>{{ block.name }}</span>
            </div>
          </div>
        </el-collapse-item>

        <!-- QT GUI 模块 -->
        <el-collapse-item title="QT GUI 模块" name="gui">
          <template #title>
            <el-icon><Monitor /></el-icon>
            <span>QT GUI 模块</span>
          </template>
          <div class="block-category">
            <div
              v-for="block in filteredBlocks.gui"
              :key="block.id"
              class="block-item"
              draggable="true"
              @dragstart="handleDragStart($event, block)"
            >
              <el-icon><component :is="block.icon" /></el-icon>
              <span>{{ block.name }}</span>
            </div>
          </div>
        </el-collapse-item>

        <!-- 自定义模块 -->
        <el-collapse-item title="自定义模块" name="custom">
          <template #title>
            <el-icon><Tools /></el-icon>
            <span>自定义模块</span>
          </template>
          <div class="block-category">
            <div
              v-for="block in filteredBlocks.custom"
              :key="block.id"
              class="block-item"
              draggable="true"
              @dragstart="handleDragStart($event, block)"
            >
              <el-icon><component :is="block.icon" /></el-icon>
              <span>{{ block.name }}</span>
            </div>
          </div>
        </el-collapse-item>
      </el-collapse>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import {
  Search, Cpu, Filter, TrendCharts, Monitor, Tools,
  Connection, Microphone, VideoCamera, DataAnalysis,
  PieChart, BarChart, LineChart, Histogram
} from '@element-plus/icons-vue'

// 搜索文本
const searchText = ref('')
const activeCategories = ref(['sources'])

// 模块定义
const blockLibrary = {
  sources: [
    { id: 'signal_source', name: '信号源', icon: 'Connection', type: 'source' },
    { id: 'noise_source', name: '噪声源', icon: 'Microphone', type: 'source' },
    { id: 'file_source', name: '文件源', icon: 'VideoCamera', type: 'source' },
    { id: 'vector_source', name: '向量源', icon: 'DataAnalysis', type: 'source' }
  ],
  filters: [
    { id: 'low_pass_filter', name: '低通滤波器', icon: 'Filter', type: 'filter' },
    { id: 'high_pass_filter', name: '高通滤波器', icon: 'Filter', type: 'filter' },
    { id: 'band_pass_filter', name: '带通滤波器', icon: 'Filter', type: 'filter' },
    { id: 'band_stop_filter', name: '带阻滤波器', icon: 'Filter', type: 'filter' }
  ],
  analysis: [
    { id: 'fft', name: 'FFT', icon: 'TrendCharts', type: 'analysis' },
    { id: 'spectrum_analyzer', name: '频谱分析仪', icon: 'BarChart', type: 'analysis' },
    { id: 'power_meter', name: '功率计', icon: 'PieChart', type: 'analysis' },
    { id: 'constellation', name: '星座图', icon: 'Histogram', type: 'analysis' }
  ],
  gui: [
    { id: 'time_sink', name: '时域显示', icon: 'LineChart', type: 'gui' },
    { id: 'freq_sink', name: '频域显示', icon: 'BarChart', type: 'gui' },
    { id: 'waterfall_sink', name: '瀑布图', icon: 'TrendCharts', type: 'gui' },
    { id: 'number_sink', name: '数值显示', icon: 'Monitor', type: 'gui' }
  ],
  custom: [
    { id: 'custom_block', name: '自定义Block', icon: 'Tools', type: 'custom' },
    { id: 'python_block', name: 'Python Block', icon: 'Cpu', type: 'custom' },
    { id: 'cpp_block', name: 'C++ Block', icon: 'Connection', type: 'custom' }
  ]
}

// 过滤后的模块
const filteredBlocks = computed(() => {
  if (!searchText.value) {
    return blockLibrary
  }

  const filtered: any = {}
  const search = searchText.value.toLowerCase()

  Object.keys(blockLibrary).forEach(category => {
    filtered[category] = (blockLibrary as any)[category].filter((block: any) =>
      block.name.toLowerCase().includes(search)
    )
  })

  return filtered
})

// 定义事件
const emit = defineEmits<{
  'block-drag': [blockType: string, event: DragEvent]
}>()

// 拖拽开始处理
const handleDragStart = (event: DragEvent, block: any) => {
  if (event.dataTransfer) {
    event.dataTransfer.setData('application/json', JSON.stringify(block))
    event.dataTransfer.effectAllowed = 'copy'
  }
  emit('block-drag', block.id, event)
}
</script>

<style scoped>
.block-library {
  height: 100%;
  display: flex;
  flex-direction: column;
  background: white;
}

.library-header {
  padding: 16px;
  border-bottom: 1px solid #e4e7ed;
}

.library-header h3 {
  margin: 0 0 12px 0;
  color: #303133;
  font-size: 16px;
  font-weight: 600;
}

.library-content {
  flex: 1;
  overflow-y: auto;
  padding: 8px;
}

.block-category {
  padding: 8px 0;
}

.block-item {
  display: flex;
  align-items: center;
  padding: 8px 12px;
  margin: 2px 0;
  border-radius: 6px;
  cursor: grab;
  transition: all 0.2s ease;
  user-select: none;
}

.block-item:hover {
  background-color: #f5f7fa;
  transform: translateX(4px);
}

.block-item:active {
  cursor: grabbing;
  transform: scale(0.98);
}

.block-item .el-icon {
  margin-right: 8px;
  color: #409eff;
}

.block-item span {
  font-size: 13px;
  color: #606266;
}

.el-collapse {
  border: none;
}

.el-collapse :deep(.el-collapse-item__header) {
  background-color: #fafafa;
  border: none;
  padding-left: 12px;
  font-weight: 500;
}

.el-collapse :deep(.el-collapse-item__content) {
  padding-bottom: 0;
}

.el-collapse :deep(.el-collapse-item__wrap) {
  border-bottom: none;
}

.el-collapse :deep(.el-collapse-item__header .el-icon) {
  margin-right: 8px;
  color: #409eff;
}
</style>