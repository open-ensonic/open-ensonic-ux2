<template>
  <div class="toolbar">
    <!-- 文件操作组 -->
    <div class="toolbar-group">
      <el-button-group>
        <el-button size="small" @click="emit('tool-action', 'file.open')" title="打开">
          <el-icon><FolderOpened /></el-icon>
        </el-button>
        <el-button size="small" @click="emit('tool-action', 'file.save')" title="保存">
          <el-icon><Document /></el-icon>
        </el-button>
        <el-button size="small" @click="emit('tool-action', 'file.screenshot')" title="截图">
          <el-icon><Camera /></el-icon>
        </el-button>
      </el-button-group>
    </div>

    <el-divider direction="vertical" />

    <!-- 画布编辑组 -->
    <div class="toolbar-group">
      <el-button-group>
        <el-button size="small" @click="emit('tool-action', 'edit.delete')" title="删除">
          <el-icon><Delete /></el-icon>
        </el-button>
        <el-button size="small" @click="emit('tool-action', 'edit.copy')" title="复制">
          <el-icon><CopyDocument /></el-icon>
        </el-button>
        <el-button size="small" @click="emit('tool-action', 'edit.paste')" title="粘贴">
          <el-icon><DocumentCopy /></el-icon>
        </el-button>
        <el-button size="small" @click="emit('tool-action', 'edit.undo')" title="撤销">
          <el-icon><RefreshLeft /></el-icon>
        </el-button>
        <el-button size="small" @click="emit('tool-action', 'edit.redo')" title="重做">
          <el-icon><RefreshRight /></el-icon>
        </el-button>
      </el-button-group>
    </div>

    <el-divider direction="vertical" />

    <!-- 运行控制组 -->
    <div class="toolbar-group">
      <el-button-group>
        <el-button 
          size="small" 
          type="success" 
          @click="emit('tool-action', 'run.execute')" 
          title="执行"
          :disabled="isRunning"
        >
          <el-icon><VideoPlay /></el-icon>
        </el-button>
        <el-button 
          size="small" 
          type="danger" 
          @click="emit('tool-action', 'run.terminate')" 
          title="终止"
          :disabled="!isRunning"
        >
          <el-icon><VideoPause /></el-icon>
        </el-button>
      </el-button-group>
    </div>

    <el-divider direction="vertical" />

    <!-- 视图控制组 -->
    <div class="toolbar-group">
      <el-button-group>
        <el-button size="small" @click="emit('tool-action', 'view.zoomOut')" title="缩小">
          <el-icon><ZoomOut /></el-icon>
        </el-button>
        <el-button size="small" @click="emit('tool-action', 'view.resetZoom')" title="重置缩放">
          {{ zoomLevel }}%
        </el-button>
        <el-button size="small" @click="emit('tool-action', 'view.zoomIn')" title="放大">
          <el-icon><ZoomIn /></el-icon>
        </el-button>
      </el-button-group>
    </div>

    <el-divider direction="vertical" />

    <!-- 全局设置组 -->
    <div class="toolbar-group">
      <el-button size="small" @click="emit('tool-action', 'edit.preferences')" title="设置">
        <el-icon><Setting /></el-icon>
      </el-button>
    </div>

    <!-- 右侧状态信息 -->
    <div class="toolbar-status">
      <el-tag v-if="isRunning" type="success" size="small">
        <el-icon><Loading /></el-icon>
        运行中
      </el-tag>
      <el-tag v-else type="info" size="small">就绪</el-tag>
      
      <span class="status-text">网格: {{ showGrid ? '开' : '关' }}</span>
      <span class="status-text">对齐: {{ snapToGrid ? '开' : '关' }}</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  FolderOpened, Document, Camera, Delete, CopyDocument, DocumentCopy,
  RefreshLeft, RefreshRight, VideoPlay, VideoPause, ZoomIn, ZoomOut,
  Setting, Loading
} from '@element-plus/icons-vue'

// 定义属性
defineProps<{
  isRunning?: boolean
  zoomLevel?: number
  showGrid?: boolean
  snapToGrid?: boolean
}>()

// 定义事件
const emit = defineEmits<{
  'tool-action': [action: string, data?: any]
}>()
</script>

<style scoped>
.toolbar {
  display: flex;
  align-items: center;
  padding: 8px 16px;
  background: white;
  border-bottom: 1px solid #e4e7ed;
  gap: 8px;
}

.toolbar-group {
  display: flex;
  align-items: center;
}

.toolbar-status {
  margin-left: auto;
  display: flex;
  align-items: center;
  gap: 12px;
}

.status-text {
  font-size: 12px;
  color: #606266;
}

.el-button-group .el-button {
  padding: 6px 8px;
}

.el-divider--vertical {
  height: 20px;
  margin: 0 8px;
}
</style>