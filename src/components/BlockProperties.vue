<template>
  <div class="block-properties">
    <div v-if="!selectedBlock" class="no-selection">
      <el-empty description="请选择一个模块查看属性" />
    </div>
    
    <div v-else class="properties-content">
      <div class="property-header">
        <h4>{{ selectedBlock.name }}</h4>
        <el-tag :type="getBlockTypeColor(selectedBlock.type)" size="small">
          {{ getBlockTypeName(selectedBlock.type) }}
        </el-tag>
      </div>
      
      <el-form :model="selectedBlock" label-width="80px" size="small">
        <el-form-item label="ID">
          <el-input v-model="selectedBlock.id" readonly />
        </el-form-item>
        
        <el-form-item label="名称">
          <el-input v-model="selectedBlock.name" @change="updateBlock" />
        </el-form-item>
        
        <el-form-item label="状态">
          <el-switch
            v-model="selectedBlock.enabled"
            active-text="启用"
            inactive-text="禁用"
            @change="updateBlock"
          />
        </el-form-item>
        
        <el-divider content-position="left">参数设置</el-divider>
        
        <el-form-item
          v-for="param in selectedBlock.params"
          :key="param.name"
          :label="param.name"
        >
          <el-input
            v-if="param.type === 'string'"
            v-model="param.value"
            @change="updateBlock"
          />
          <el-input-number
            v-else-if="param.type === 'int'"
            v-model="param.value"
            :step="1"
            @change="updateBlock"
          />
          <el-input-number
            v-else-if="param.type === 'float'"
            v-model="param.value"
            :step="0.1"
            :precision="3"
            @change="updateBlock"
          />
          <el-switch
            v-else-if="param.type === 'bool'"
            v-model="param.value"
            @change="updateBlock"
          />
          <el-input
            v-else
            v-model="param.value"
            @change="updateBlock"
          />
        </el-form-item>
        
        <el-divider content-position="left">端口信息</el-divider>
        
        <div class="ports-info">
          <div class="port-group">
            <h5>输入端口</h5>
            <div v-if="selectedBlock.inputPorts.length === 0" class="no-ports">
              无输入端口
            </div>
            <div
              v-for="port in selectedBlock.inputPorts"
              :key="port.id"
              class="port-item"
            >
              <el-tag size="small" type="info">{{ port.name }}</el-tag>
              <span class="port-type">{{ port.type || 'complex' }}</span>
            </div>
          </div>
          
          <div class="port-group">
            <h5>输出端口</h5>
            <div v-if="selectedBlock.outputPorts.length === 0" class="no-ports">
              无输出端口
            </div>
            <div
              v-for="port in selectedBlock.outputPorts"
              :key="port.id"
              class="port-item"
            >
              <el-tag size="small" type="success">{{ port.name }}</el-tag>
              <span class="port-type">{{ port.type || 'complex' }}</span>
            </div>
          </div>
        </div>
        
        <el-divider content-position="left">操作</el-divider>
        
        <div class="block-actions">
          <el-button size="small" @click="duplicateBlock">
            <el-icon><CopyDocument /></el-icon>
            复制模块
          </el-button>
          <el-button size="small" type="danger" @click="deleteBlock">
            <el-icon><Delete /></el-icon>
            删除模块
          </el-button>
        </div>
      </el-form>
    </div>
  </div>
</template>

<script setup lang="ts">
import { CopyDocument, Delete } from '@element-plus/icons-vue'

// 定义属性
const props = defineProps<{
  selectedBlock?: any
}>()

// 定义事件
const emit = defineEmits<{
  'update-block': [block: any]
  'duplicate-block': [block: any]
  'delete-block': [blockId: string]
}>()

// 获取模块类型颜色
const getBlockTypeColor = (type: string) => {
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
const getBlockTypeName = (type: string) => {
  const nameMap: Record<string, string> = {
    source: '信号源',
    filter: '滤波器',
    analysis: '分析',
    gui: 'GUI',
    custom: '自定义'
  }
  return nameMap[type] || '未知'
}

// 更新模块
const updateBlock = () => {
  if (props.selectedBlock) {
    emit('update-block', props.selectedBlock)
  }
}

// 复制模块
const duplicateBlock = () => {
  if (props.selectedBlock) {
    emit('duplicate-block', props.selectedBlock)
  }
}

// 删除模块
const deleteBlock = () => {
  if (props.selectedBlock) {
    emit('delete-block', props.selectedBlock.id)
  }
}
</script>

<style scoped>
.block-properties {
  height: 100%;
  padding: 16px;
  overflow-y: auto;
}

.no-selection {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
}

.properties-content {
  height: 100%;
}

.property-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.property-header h4 {
  margin: 0;
  color: #303133;
}

.ports-info {
  margin: 16px 0;
}

.port-group {
  margin-bottom: 16px;
}

.port-group h5 {
  margin: 0 0 8px 0;
  color: #606266;
  font-size: 13px;
}

.port-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 4px 0;
}

.port-type {
  font-size: 12px;
  color: #909399;
}

.no-ports {
  color: #c0c4cc;
  font-size: 12px;
  font-style: italic;
}

.block-actions {
  display: flex;
  gap: 8px;
  margin-top: 16px;
}

.el-form-item {
  margin-bottom: 12px;
}

.el-divider {
  margin: 16px 0 12px 0;
}
</style>