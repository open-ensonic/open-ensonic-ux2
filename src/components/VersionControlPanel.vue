<template>
  <div class="version-control-panel">
    <div class="vc-header">
      <h4>版本控制</h4>
      <el-button-group size="small">
        <el-button @click="refreshStatus">
          <el-icon><Refresh /></el-icon>
          刷新
        </el-button>
        <el-button @click="showCommitDialog = true" type="primary">
          <el-icon><Upload /></el-icon>
          提交
        </el-button>
      </el-button-group>
    </div>
    
    <el-tabs v-model="activeTab" type="card">
      <!-- 更改 -->
      <el-tab-pane label="更改" name="changes">
        <div class="changes-section">
          <div class="section-header">
            <span>未暂存的更改 ({{ unstagedFiles.length }})</span>
            <el-button size="small" @click="stageAll">全部暂存</el-button>
          </div>
          
          <div class="file-list">
            <div
              v-for="file in unstagedFiles"
              :key="file.path"
              class="file-item"
              :class="file.status"
            >
              <div class="file-info">
                <el-icon><component :is="getFileIcon(file.status)" /></el-icon>
                <span class="file-path">{{ file.path }}</span>
                <el-tag :type="getStatusColor(file.status)" size="small">
                  {{ getStatusText(file.status) }}
                </el-tag>
              </div>
              <div class="file-actions">
                <el-button size="small" @click="stageFile(file)">暂存</el-button>
                <el-button size="small" @click="discardChanges(file)">丢弃</el-button>
              </div>
            </div>
          </div>
          
          <div class="section-header">
            <span>已暂存的更改 ({{ stagedFiles.length }})</span>
            <el-button size="small" @click="unstageAll">全部取消暂存</el-button>
          </div>
          
          <div class="file-list">
            <div
              v-for="file in stagedFiles"
              :key="file.path"
              class="file-item staged"
            >
              <div class="file-info">
                <el-icon><component :is="getFileIcon(file.status)" /></el-icon>
                <span class="file-path">{{ file.path }}</span>
                <el-tag type="success" size="small">已暂存</el-tag>
              </div>
              <div class="file-actions">
                <el-button size="small" @click="unstageFile(file)">取消暂存</el-button>
              </div>
            </div>
          </div>
        </div>
      </el-tab-pane>
      
      <!-- 历史 -->
      <el-tab-pane label="历史" name="history">
        <div class="history-section">
          <div class="commit-list">
            <div
              v-for="commit in commits"
              :key="commit.hash"
              class="commit-item"
              @click="selectCommit(commit)"
            >
              <div class="commit-header">
                <span class="commit-hash">{{ commit.hash.substring(0, 7) }}</span>
                <span class="commit-time">{{ formatTime(commit.timestamp) }}</span>
              </div>
              <div class="commit-message">{{ commit.message }}</div>
              <div class="commit-author">{{ commit.author }}</div>
            </div>
          </div>
        </div>
      </el-tab-pane>
      
      <!-- 分支 -->
      <el-tab-pane label="分支" name="branches">
        <div class="branches-section">
          <div class="current-branch">
            <h5>当前分支</h5>
            <div class="branch-item current">
              <el-icon><Branch /></el-icon>
              <span>{{ currentBranch }}</span>
              <el-tag type="success" size="small">当前</el-tag>
            </div>
          </div>
          
          <div class="branch-list">
            <div class="section-header">
              <span>本地分支</span>
              <el-button size="small" @click="showCreateBranchDialog = true">
                <el-icon><Plus /></el-icon>
                新建分支
              </el-button>
            </div>
            
            <div
              v-for="branch in localBranches"
              :key="branch.name"
              class="branch-item"
              @click="switchBranch(branch)"
            >
              <el-icon><Branch /></el-icon>
              <span>{{ branch.name }}</span>
              <span class="branch-info">{{ branch.lastCommit }}</span>
            </div>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
    
    <!-- 提交对话框 -->
    <el-dialog v-model="showCommitDialog" title="提交更改" width="500px">
      <el-form :model="commitForm" label-width="80px">
        <el-form-item label="提交信息" required>
          <el-input
            v-model="commitForm.message"
            type="textarea"
            :rows="3"
            placeholder="请输入提交信息..."
          />
        </el-form-item>
        <el-form-item label="作者">
          <el-input v-model="commitForm.author" placeholder="作者姓名" />
        </el-form-item>
      </el-form>
      
      <template #footer>
        <el-button @click="showCommitDialog = false">取消</el-button>
        <el-button type="primary" @click="commitChanges">提交</el-button>
      </template>
    </el-dialog>
    
    <!-- 创建分支对话框 -->
    <el-dialog v-model="showCreateBranchDialog" title="创建新分支" width="400px">
      <el-form :model="branchForm" label-width="80px">
        <el-form-item label="分支名称" required>
          <el-input v-model="branchForm.name" placeholder="请输入分支名称" />
        </el-form-item>
        <el-form-item label="基于分支">
          <el-select v-model="branchForm.baseBranch" placeholder="选择基础分支">
            <el-option
              v-for="branch in localBranches"
              :key="branch.name"
              :label="branch.name"
              :value="branch.name"
            />
          </el-select>
        </el-form-item>
      </el-form>
      
      <template #footer>
        <el-button @click="showCreateBranchDialog = false">取消</el-button>
        <el-button type="primary" @click="createBranch">创建</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import {
  Refresh, Upload, Branch, Plus, Document, Edit, Delete, Warning
} from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus'

// 响应式数据
const activeTab = ref('changes')
const showCommitDialog = ref(false)
const showCreateBranchDialog = ref(false)
const currentBranch = ref('main')

// 文件状态
const unstagedFiles = ref([
  { path: 'src/views/Editor.vue', status: 'modified' },
  { path: 'src/components/FlowCanvas.vue', status: 'modified' },
  { path: 'src/components/NewBlock.vue', status: 'added' },
  { path: 'README.md', status: 'deleted' }
])

const stagedFiles = ref([
  { path: 'src/components/MenuBar.vue', status: 'modified' },
  { path: 'package.json', status: 'modified' }
])

// 提交历史
const commits = ref([
  {
    hash: 'a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0',
    message: '添加流程图编辑器基础功能',
    author: 'OpenEnsonic Team',
    timestamp: new Date(Date.now() - 3600000)
  },
  {
    hash: 'b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1',
    message: '实现模块拖拽功能',
    author: 'OpenEnsonic Team',
    timestamp: new Date(Date.now() - 7200000)
  },
  {
    hash: 'c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2',
    message: '添加菜单栏和工具栏',
    author: 'OpenEnsonic Team',
    timestamp: new Date(Date.now() - 10800000)
  }
])

// 分支列表
const localBranches = ref([
  { name: 'main', lastCommit: 'a1b2c3d' },
  { name: 'feature/canvas-editor', lastCommit: 'b2c3d4e' },
  { name: 'feature/block-library', lastCommit: 'c3d4e5f' }
])

// 表单数据
const commitForm = reactive({
  message: '',
  author: 'OpenEnsonic Team'
})

const branchForm = reactive({
  name: '',
  baseBranch: 'main'
})

// 获取文件图标
const getFileIcon = (status: string) => {
  const iconMap: Record<string, string> = {
    modified: 'Edit',
    added: 'Document',
    deleted: 'Delete',
    renamed: 'Warning'
  }
  return iconMap[status] || 'Document'
}

// 获取状态颜色
const getStatusColor = (status: string) => {
  const colorMap: Record<string, string> = {
    modified: 'warning',
    added: 'success',
    deleted: 'danger',
    renamed: 'info'
  }
  return colorMap[status] || 'info'
}

// 获取状态文本
const getStatusText = (status: string) => {
  const textMap: Record<string, string> = {
    modified: '已修改',
    added: '新增',
    deleted: '已删除',
    renamed: '重命名'
  }
  return textMap[status] || '未知'
}

// 格式化时间
const formatTime = (timestamp: Date) => {
  const now = new Date()
  const diff = now.getTime() - timestamp.getTime()
  const hours = Math.floor(diff / (1000 * 60 * 60))
  
  if (hours < 1) {
    const minutes = Math.floor(diff / (1000 * 60))
    return `${minutes} 分钟前`
  } else if (hours < 24) {
    return `${hours} 小时前`
  } else {
    const days = Math.floor(hours / 24)
    return `${days} 天前`
  }
}

// 刷新状态
const refreshStatus = () => {
  ElMessage.success('状态已刷新')
}

// 暂存文件
const stageFile = (file: any) => {
  const index = unstagedFiles.value.findIndex(f => f.path === file.path)
  if (index !== -1) {
    unstagedFiles.value.splice(index, 1)
    stagedFiles.value.push(file)
    ElMessage.success(`已暂存 ${file.path}`)
  }
}

// 暂存所有文件
const stageAll = () => {
  stagedFiles.value.push(...unstagedFiles.value)
  unstagedFiles.value = []
  ElMessage.success('已暂存所有更改')
}

// 取消暂存文件
const unstageFile = (file: any) => {
  const index = stagedFiles.value.findIndex(f => f.path === file.path)
  if (index !== -1) {
    stagedFiles.value.splice(index, 1)
    unstagedFiles.value.push(file)
    ElMessage.success(`已取消暂存 ${file.path}`)
  }
}

// 取消暂存所有文件
const unstageAll = () => {
  unstagedFiles.value.push(...stagedFiles.value)
  stagedFiles.value = []
  ElMessage.success('已取消暂存所有更改')
}

// 丢弃更改
const discardChanges = (file: any) => {
  const index = unstagedFiles.value.findIndex(f => f.path === file.path)
  if (index !== -1) {
    unstagedFiles.value.splice(index, 1)
    ElMessage.success(`已丢弃 ${file.path} 的更改`)
  }
}

// 提交更改
const commitChanges = () => {
  if (!commitForm.message.trim()) {
    ElMessage.error('请输入提交信息')
    return
  }
  
  if (stagedFiles.value.length === 0) {
    ElMessage.error('没有已暂存的更改')
    return
  }
  
  const newCommit = {
    hash: Math.random().toString(36).substring(2, 42),
    message: commitForm.message,
    author: commitForm.author,
    timestamp: new Date()
  }
  
  commits.value.unshift(newCommit)
  stagedFiles.value = []
  
  // 重置表单
  commitForm.message = ''
  showCommitDialog.value = false
  
  ElMessage.success('提交成功')
}

// 选择提交
const selectCommit = (commit: any) => {
  ElMessage.info(`选择了提交: ${commit.hash.substring(0, 7)}`)
}

// 切换分支
const switchBranch = (branch: any) => {
  if (branch.name === currentBranch.value) return
  
  currentBranch.value = branch.name
  ElMessage.success(`已切换到分支: ${branch.name}`)
}

// 创建分支
const createBranch = () => {
  if (!branchForm.name.trim()) {
    ElMessage.error('请输入分支名称')
    return
  }
  
  if (localBranches.value.some(b => b.name === branchForm.name)) {
    ElMessage.error('分支名称已存在')
    return
  }
  
  localBranches.value.push({
    name: branchForm.name,
    lastCommit: commits.value[0]?.hash.substring(0, 7) || 'a1b2c3d'
  })
  
  // 重置表单
  branchForm.name = ''
  branchForm.baseBranch = 'main'
  showCreateBranchDialog.value = false
  
  ElMessage.success(`分支 ${branchForm.name} 创建成功`)
}
</script>

<style scoped>
.version-control-panel {
  height: 100%;
  display: flex;
  flex-direction: column;
  padding: 16px;
}

.vc-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.vc-header h4 {
  margin: 0;
  color: #303133;
}

.changes-section,
.history-section,
.branches-section {
  height: 100%;
  overflow-y: auto;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 0;
  margin-bottom: 8px;
  font-size: 13px;
  font-weight: 500;
  color: #606266;
  border-bottom: 1px solid #e4e7ed;
}

.file-list {
  margin-bottom: 20px;
}

.file-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  margin: 2px 0;
  border-radius: 6px;
  transition: background-color 0.2s ease;
}

.file-item:hover {
  background-color: #f5f7fa;
}

.file-item.staged {
  background-color: #f0f9ff;
}

.file-info {
  display: flex;
  align-items: center;
  gap: 8px;
  flex: 1;
}

.file-path {
  font-size: 13px;
  color: #303133;
}

.file-actions {
  display: flex;
  gap: 4px;
}

.commit-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.commit-item {
  padding: 12px;
  border: 1px solid #e4e7ed;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.commit-item:hover {
  border-color: #409eff;
  box-shadow: 0 2px 8px rgba(64, 158, 255, 0.1);
}

.commit-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 4px;
}

.commit-hash {
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  font-size: 12px;
  color: #409eff;
  font-weight: 500;
}

.commit-time {
  font-size: 12px;
  color: #909399;
}

.commit-message {
  font-size: 14px;
  color: #303133;
  margin-bottom: 4px;
}

.commit-author {
  font-size: 12px;
  color: #606266;
}

.current-branch {
  margin-bottom: 20px;
}

.current-branch h5 {
  margin: 0 0 8px 0;
  color: #606266;
  font-size: 13px;
}

.branch-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  margin: 2px 0;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.branch-item:hover {
  background-color: #f5f7fa;
}

.branch-item.current {
  background-color: #f0f9ff;
  cursor: default;
}

.branch-info {
  margin-left: auto;
  font-size: 12px;
  color: #909399;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
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