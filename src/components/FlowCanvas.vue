<template>
  <div class="flow-canvas-container">
    <div class="canvas-toolbar">
      <el-button-group size="small">
        <el-button @click="toggleGrid">
          <el-icon><Grid /></el-icon>
          {{ showGrid ? '隐藏网格' : '显示网格' }}
        </el-button>
        <el-button @click="toggleSnap">
          <el-icon><Magnet /></el-icon>
          {{ snapToGrid ? '关闭对齐' : '开启对齐' }}
        </el-button>
        <el-button @click="fitToScreen">
          <el-icon><FullScreen /></el-icon>
          适应屏幕
        </el-button>
      </el-button-group>
      
      <div class="canvas-info">
        <span>缩放: {{ Math.round(zoomLevel) }}%</span>
        <span>模块: {{ blocks.length }}</span>
        <span>连接: {{ connections.length }}</span>
      </div>
    </div>

    <div
      ref="canvasContainer"
      class="canvas-container"
      :class="{ 'show-grid': showGrid }"
      @drop="handleDrop"
      @dragover="handleDragOver"
      @click="handleCanvasClick"
      @wheel="handleWheel"
      @mousedown="handleMouseDown"
      @mousemove="handleMouseMove"
      @mouseup="handleMouseUp"
    >
      <div
        class="canvas-content"
        :style="{
          transform: `translate(${panX}px, ${panY}px) scale(${zoomLevel / 100})`,
          transformOrigin: '0 0'
        }"
      >
        <!-- 连接线 -->
        <svg class="connections-layer" :style="{ width: '100%', height: '100%' }">
          <defs>
            <marker
              id="arrowhead"
              markerWidth="10"
              markerHeight="7"
              refX="9"
              refY="3.5"
              orient="auto"
            >
              <polygon points="0 0, 10 3.5, 0 7" fill="#409eff" />
            </marker>
          </defs>
          
          <path
            v-for="connection in connections"
            :key="connection.id"
            :d="getConnectionPath(connection)"
            stroke="#409eff"
            stroke-width="2"
            fill="none"
            marker-end="url(#arrowhead)"
            class="connection-line"
          />
          
          <!-- 临时连接线 -->
          <path
            v-if="tempConnection"
            :d="tempConnection.path"
            stroke="#67c23a"
            stroke-width="2"
            stroke-dasharray="5,5"
            fill="none"
            class="temp-connection"
          />
        </svg>

        <!-- 模块 -->
        <div
          v-for="block in blocks"
          :key="block.id"
          class="flow-block"
          :class="{
            selected: selectedBlocks.includes(block.id),
            disabled: block.disabled
          }"
          :style="{
            left: block.x + 'px',
            top: block.y + 'px'
          }"
          @click.stop="selectBlock(block)"
          @mousedown.stop="startDrag(block, $event)"
        >
          <div class="block-header">
            <el-icon><component :is="getBlockIcon(block.type)" /></el-icon>
            <span class="block-title">{{ block.name }}</span>
            <el-button
              size="small"
              text
              @click.stop="removeBlock(block.id)"
              class="block-remove"
            >
              <el-icon><Close /></el-icon>
            </el-button>
          </div>
          
          <div class="block-body">
            <!-- 输入端口 -->
            <div class="input-ports">
              <div
                v-for="port in block.inputPorts"
                :key="port.id"
                class="port input-port"
                :data-port-id="port.id"
                :data-block-id="block.id"
                @mousedown.stop="startConnection(block, port, $event)"
              >
                <div class="port-dot"></div>
                <span class="port-label">{{ port.name }}</span>
              </div>
            </div>
            
            <!-- 输出端口 -->
            <div class="output-ports">
              <div
                v-for="port in block.outputPorts"
                :key="port.id"
                class="port output-port"
                :data-port-id="port.id"
                :data-block-id="block.id"
                @mousedown.stop="startConnection(block, port, $event)"
              >
                <span class="port-label">{{ port.name }}</span>
                <div class="port-dot"></div>
              </div>
            </div>
          </div>
          
          <!-- 参数显示 -->
          <div v-if="block.showParams" class="block-params">
            <div
              v-for="param in block.params"
              :key="param.name"
              class="param-item"
            >
              <span class="param-name">{{ param.name }}:</span>
              <span class="param-value">{{ param.value }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, nextTick } from 'vue'
import {
  Grid, Magnet, FullScreen, Close, Cpu, Filter, TrendCharts,
  Monitor, Tools, Connection
} from '@element-plus/icons-vue'

// 响应式状态
const canvasContainer = ref<HTMLElement>()
const showGrid = ref(true)
const snapToGrid = ref(true)
const zoomLevel = ref(100)
const panX = ref(0)
const panY = ref(0)

// 画布状态
const blocks = ref<any[]>([])
const connections = ref<any[]>([])
const selectedBlocks = ref<string[]>([])
const tempConnection = ref<any>(null)

// 交互状态
const isDragging = ref(false)
const isPanning = ref(false)
const isConnecting = ref(false)
const dragStart = reactive({ x: 0, y: 0 })
const lastMousePos = reactive({ x: 0, y: 0 })

// 定义事件
const emit = defineEmits<{
  'block-select': [block: any]
  'canvas-click': []
}>()

// 获取模块图标
const getBlockIcon = (type: string) => {
  const iconMap: Record<string, string> = {
    source: 'Connection',
    filter: 'Filter',
    analysis: 'TrendCharts',
    gui: 'Monitor',
    custom: 'Tools'
  }
  return iconMap[type] || 'Cpu'
}

// 切换网格显示
const toggleGrid = () => {
  showGrid.value = !showGrid.value
}

// 切换网格对齐
const toggleSnap = () => {
  snapToGrid.value = !snapToGrid.value
}

// 适应屏幕
const fitToScreen = () => {
  if (blocks.value.length === 0) return
  
  // 计算所有模块的边界
  let minX = Infinity, minY = Infinity, maxX = -Infinity, maxY = -Infinity
  
  blocks.value.forEach(block => {
    minX = Math.min(minX, block.x)
    minY = Math.min(minY, block.y)
    maxX = Math.max(maxX, block.x + 200) // 假设模块宽度为200
    maxY = Math.max(maxY, block.y + 100) // 假设模块高度为100
  })
  
  const containerRect = canvasContainer.value?.getBoundingClientRect()
  if (!containerRect) return
  
  const contentWidth = maxX - minX
  const contentHeight = maxY - minY
  const containerWidth = containerRect.width
  const containerHeight = containerRect.height
  
  const scaleX = containerWidth / contentWidth
  const scaleY = containerHeight / contentHeight
  const scale = Math.min(scaleX, scaleY, 1) * 0.8 // 留一些边距
  
  zoomLevel.value = scale * 100
  panX.value = (containerWidth - contentWidth * scale) / 2 - minX * scale
  panY.value = (containerHeight - contentHeight * scale) / 2 - minY * scale
}

// 处理拖放
const handleDrop = (event: DragEvent) => {
  event.preventDefault()
  
  const data = event.dataTransfer?.getData('application/json')
  if (!data) return
  
  const blockData = JSON.parse(data)
  const rect = canvasContainer.value?.getBoundingClientRect()
  if (!rect) return
  
  const x = (event.clientX - rect.left - panX.value) / (zoomLevel.value / 100)
  const y = (event.clientY - rect.top - panY.value) / (zoomLevel.value / 100)
  
  addBlock(blockData, x, y)
}

const handleDragOver = (event: DragEvent) => {
  event.preventDefault()
}

// 添加模块
const addBlock = (blockData: any, x: number, y: number) => {
  const newBlock = {
    id: `block_${Date.now()}`,
    name: blockData.name,
    type: blockData.type,
    x: snapToGrid.value ? Math.round(x / 20) * 20 : x,
    y: snapToGrid.value ? Math.round(y / 20) * 20 : y,
    disabled: false,
    showParams: false,
    inputPorts: generatePorts('input', blockData.type),
    outputPorts: generatePorts('output', blockData.type),
    params: generateParams(blockData.type)
  }
  
  blocks.value.push(newBlock)
}

// 生成端口
const generatePorts = (type: 'input' | 'output', blockType: string) => {
  const portConfigs: Record<string, any> = {
    source: { input: [], output: [{ id: 'out', name: 'out' }] },
    filter: { 
      input: [{ id: 'in', name: 'in' }], 
      output: [{ id: 'out', name: 'out' }] 
    },
    analysis: { input: [{ id: 'in', name: 'in' }], output: [] },
    gui: { input: [{ id: 'in', name: 'in' }], output: [] },
    custom: { 
      input: [{ id: 'in', name: 'in' }], 
      output: [{ id: 'out', name: 'out' }] 
    }
  }
  
  return portConfigs[blockType]?.[type] || []
}

// 生成参数
const generateParams = (blockType: string) => {
  const paramConfigs: Record<string, any[]> = {
    source: [
      { name: 'frequency', value: '1000', type: 'float' },
      { name: 'amplitude', value: '1.0', type: 'float' }
    ],
    filter: [
      { name: 'cutoff', value: '1000', type: 'float' },
      { name: 'transition', value: '100', type: 'float' }
    ],
    analysis: [
      { name: 'fft_size', value: '1024', type: 'int' }
    ],
    gui: [
      { name: 'update_rate', value: '10', type: 'float' }
    ],
    custom: []
  }
  
  return paramConfigs[blockType] || []
}

// 选择模块
const selectBlock = (block: any) => {
  selectedBlocks.value = [block.id]
  emit('block-select', block)
}

// 画布点击
const handleCanvasClick = () => {
  selectedBlocks.value = []
  emit('canvas-click')
}

// 删除模块
const removeBlock = (blockId: string) => {
  blocks.value = blocks.value.filter(b => b.id !== blockId)
  connections.value = connections.value.filter(c => 
    c.sourceBlockId !== blockId && c.targetBlockId !== blockId
  )
  selectedBlocks.value = selectedBlocks.value.filter(id => id !== blockId)
}

// 鼠标事件处理
const handleMouseDown = (event: MouseEvent) => {
  if (event.button === 1 || (event.button === 0 && event.ctrlKey)) {
    // 中键或Ctrl+左键开始平移
    isPanning.value = true
    lastMousePos.x = event.clientX
    lastMousePos.y = event.clientY
    event.preventDefault()
  }
}

const handleMouseMove = (event: MouseEvent) => {
  if (isPanning.value) {
    const deltaX = event.clientX - lastMousePos.x
    const deltaY = event.clientY - lastMousePos.y
    panX.value += deltaX
    panY.value += deltaY
    lastMousePos.x = event.clientX
    lastMousePos.y = event.clientY
  }
}

const handleMouseUp = () => {
  isPanning.value = false
  isDragging.value = false
  isConnecting.value = false
  tempConnection.value = null
}

// 缩放处理
const handleWheel = (event: WheelEvent) => {
  event.preventDefault()
  const delta = event.deltaY > 0 ? -10 : 10
  const newZoom = Math.max(25, Math.min(200, zoomLevel.value + delta))
  zoomLevel.value = newZoom
}

// 开始拖拽模块
const startDrag = (block: any, event: MouseEvent) => {
  isDragging.value = true
  dragStart.x = event.clientX - block.x * (zoomLevel.value / 100)
  dragStart.y = event.clientY - block.y * (zoomLevel.value / 100)
  
  const handleDragMove = (e: MouseEvent) => {
    if (!isDragging.value) return
    
    const newX = (e.clientX - dragStart.x) / (zoomLevel.value / 100)
    const newY = (e.clientY - dragStart.y) / (zoomLevel.value / 100)
    
    block.x = snapToGrid.value ? Math.round(newX / 20) * 20 : newX
    block.y = snapToGrid.value ? Math.round(newY / 20) * 20 : newY
  }
  
  const handleDragEnd = () => {
    isDragging.value = false
    document.removeEventListener('mousemove', handleDragMove)
    document.removeEventListener('mouseup', handleDragEnd)
  }
  
  document.addEventListener('mousemove', handleDragMove)
  document.addEventListener('mouseup', handleDragEnd)
}

// 开始连接
const startConnection = (block: any, port: any, event: MouseEvent) => {
  isConnecting.value = true
  // 连接逻辑实现
}

// 获取连接路径
const getConnectionPath = (connection: any) => {
  // 简化的连接路径计算
  return `M ${connection.startX} ${connection.startY} L ${connection.endX} ${connection.endY}`
}

// 暴露方法给父组件
defineExpose({
  takeScreenshot: () => {
    console.log('Taking screenshot...')
  },
  undo: () => {
    console.log('Undo...')
  },
  redo: () => {
    console.log('Redo...')
  },
  setZoom: (zoom: number) => {
    zoomLevel.value = zoom
  }
})
</script>

<style scoped>
.flow-canvas-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  background: #fafafa;
}

.canvas-toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 16px;
  background: white;
  border-bottom: 1px solid #e4e7ed;
}

.canvas-info {
  display: flex;
  gap: 16px;
  font-size: 12px;
  color: #606266;
}

.canvas-container {
  flex: 1;
  position: relative;
  overflow: hidden;
  cursor: grab;
}

.canvas-container:active {
  cursor: grabbing;
}

.canvas-container.show-grid {
  background-image: 
    linear-gradient(to right, #e4e7ed 1px, transparent 1px),
    linear-gradient(to bottom, #e4e7ed 1px, transparent 1px);
  background-size: 20px 20px;
}

.canvas-content {
  position: absolute;
  width: 100%;
  height: 100%;
}

.connections-layer {
  position: absolute;
  top: 0;
  left: 0;
  pointer-events: none;
  z-index: 1;
}

.connection-line {
  transition: stroke-width 0.2s ease;
}

.connection-line:hover {
  stroke-width: 3;
}

.temp-connection {
  animation: dash 1s linear infinite;
}

@keyframes dash {
  to {
    stroke-dashoffset: -10;
  }
}

.flow-block {
  position: absolute;
  background: white;
  border: 2px solid #e4e7ed;
  border-radius: 8px;
  min-width: 180px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  cursor: move;
  z-index: 2;
  transition: all 0.2s ease;
}

.flow-block:hover {
  border-color: #409eff;
  box-shadow: 0 4px 12px rgba(64, 158, 255, 0.2);
}

.flow-block.selected {
  border-color: #409eff;
  box-shadow: 0 0 0 2px rgba(64, 158, 255, 0.2);
}

.flow-block.disabled {
  opacity: 0.6;
  border-color: #dcdfe6;
}

.block-header {
  display: flex;
  align-items: center;
  padding: 8px 12px;
  background: #f5f7fa;
  border-bottom: 1px solid #e4e7ed;
  border-radius: 6px 6px 0 0;
}

.block-header .el-icon {
  margin-right: 8px;
  color: #409eff;
}

.block-title {
  flex: 1;
  font-size: 13px;
  font-weight: 500;
  color: #303133;
}

.block-remove {
  opacity: 0;
  transition: opacity 0.2s ease;
}

.flow-block:hover .block-remove {
  opacity: 1;
}

.block-body {
  padding: 12px;
  display: flex;
  justify-content: space-between;
}

.input-ports,
.output-ports {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.port {
  display: flex;
  align-items: center;
  font-size: 11px;
  color: #606266;
  cursor: crosshair;
}

.input-port {
  justify-content: flex-start;
}

.output-port {
  justify-content: flex-end;
}

.port-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #409eff;
  border: 2px solid white;
  box-shadow: 0 0 0 1px #409eff;
  transition: all 0.2s ease;
}

.port:hover .port-dot {
  transform: scale(1.3);
  box-shadow: 0 0 0 2px #409eff;
}

.port-label {
  margin: 0 6px;
  white-space: nowrap;
}

.block-params {
  padding: 8px 12px;
  border-top: 1px solid #e4e7ed;
  background: #fafafa;
}

.param-item {
  display: flex;
  justify-content: space-between;
  font-size: 11px;
  margin: 2px 0;
}

.param-name {
  color: #909399;
}

.param-value {
  color: #303133;
  font-weight: 500;
}
</style>