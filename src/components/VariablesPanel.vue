<template>
  <div class="variables-panel">
    <div class="panel-header">
      <h4>项目变量</h4>
      <el-button size="small" type="primary" @click="showAddDialog = true">
        <el-icon><Plus /></el-icon>
        添加变量
      </el-button>
    </div>
    
    <div class="variables-list">
      <el-table :data="variables" size="small" height="100%">
        <el-table-column prop="name" label="变量名" width="120">
          <template #default="{ row }">
            <el-input
              v-if="row.editing"
              v-model="row.name"
              size="small"
              @blur="saveVariable(row)"
              @keyup.enter="saveVariable(row)"
            />
            <span v-else>{{ row.name }}</span>
          </template>
        </el-table-column>
        
        <el-table-column prop="value" label="值" min-width="100">
          <template #default="{ row }">
            <el-input
              v-if="row.editing"
              v-model="row.value"
              size="small"
              @blur="saveVariable(row)"
              @keyup.enter="saveVariable(row)"
            />
            <span v-else>{{ row.value }}</span>
          </template>
        </el-table-column>
        
        <el-table-column prop="type" label="类型" width="80">
          <template #default="{ row }">
            <el-select
              v-if="row.editing"
              v-model="row.type"
              size="small"
              @change="saveVariable(row)"
            >
              <el-option label="字符串" value="string" />
              <el-option label="整数" value="int" />
              <el-option label="浮点" value="float" />
              <el-option label="布尔" value="bool" />
            </el-select>
            <el-tag v-else :type="getTypeColor(row.type)" size="small">
              {{ getTypeName(row.type) }}
            </el-tag>
          </template>
        </el-table-column>
        
        <el-table-column label="操作" width="100">
          <template #default="{ row, $index }">
            <el-button-group size="small">
              <el-button
                v-if="!row.editing"
                size="small"
                @click="editVariable(row)"
              >
                <el-icon><Edit /></el-icon>
              </el-button>
              <el-button
                v-else
                size="small"
                type="success"
                @click="saveVariable(row)"
              >
                <el-icon><Check /></el-icon>
              </el-button>
              <el-button
                size="small"
                type="danger"
                @click="deleteVariable($index)"
              >
                <el-icon><Delete /></el-icon>
              </el-button>
            </el-button-group>
          </template>
        </el-table-column>
      </el-table>
    </div>
    
    <!-- 添加变量对话框 -->
    <el-dialog
      v-model="showAddDialog"
      title="添加变量"
      width="400px"
    >
      <el-form :model="newVariable" label-width="80px">
        <el-form-item label="变量名" required>
          <el-input v-model="newVariable.name" placeholder="请输入变量名" />
        </el-form-item>
        <el-form-item label="值" required>
          <el-input v-model="newVariable.value" placeholder="请输入变量值" />
        </el-form-item>
        <el-form-item label="类型" required>
          <el-select v-model="newVariable.type" placeholder="请选择类型">
            <el-option label="字符串" value="string" />
            <el-option label="整数" value="int" />
            <el-option label="浮点" value="float" />
            <el-option label="布尔" value="bool" />
          </el-select>
        </el-form-item>
        <el-form-item label="描述">
          <el-input
            v-model="newVariable.description"
            type="textarea"
            placeholder="请输入变量描述（可选）"
          />
        </el-form-item>
      </el-form>
      
      <template #footer>
        <el-button @click="showAddDialog = false">取消</el-button>
        <el-button type="primary" @click="addVariable">确定</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import { Plus, Edit, Check, Delete } from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus'

// 响应式数据
const showAddDialog = ref(false)
const variables = ref([
  { name: 'sample_rate', value: '32000', type: 'int', description: '采样率', editing: false },
  { name: 'center_freq', value: '100e6', type: 'float', description: '中心频率', editing: false },
  { name: 'gain', value: '20', type: 'float', description: '增益', editing: false },
  { name: 'enable_gui', value: 'True', type: 'bool', description: '启用GUI', editing: false }
])

const newVariable = reactive({
  name: '',
  value: '',
  type: 'string',
  description: ''
})

// 获取类型颜色
const getTypeColor = (type: string) => {
  const colorMap: Record<string, string> = {
    string: 'primary',
    int: 'success',
    float: 'warning',
    bool: 'info'
  }
  return colorMap[type] || 'info'
}

// 获取类型名称
const getTypeName = (type: string) => {
  const nameMap: Record<string, string> = {
    string: '字符串',
    int: '整数',
    float: '浮点',
    bool: '布尔'
  }
  return nameMap[type] || '未知'
}

// 编辑变量
const editVariable = (variable: any) => {
  variable.editing = true
}

// 保存变量
const saveVariable = (variable: any) => {
  if (!variable.name.trim()) {
    ElMessage.error('变量名不能为空')
    return
  }
  
  if (!variable.value.trim()) {
    ElMessage.error('变量值不能为空')
    return
  }
  
  variable.editing = false
  ElMessage.success('变量已保存')
}

// 删除变量
const deleteVariable = (index: number) => {
  variables.value.splice(index, 1)
  ElMessage.success('变量已删除')
}

// 添加变量
const addVariable = () => {
  if (!newVariable.name.trim()) {
    ElMessage.error('请输入变量名')
    return
  }
  
  if (!newVariable.value.trim()) {
    ElMessage.error('请输入变量值')
    return
  }
  
  // 检查变量名是否已存在
  if (variables.value.some(v => v.name === newVariable.name)) {
    ElMessage.error('变量名已存在')
    return
  }
  
  variables.value.push({
    name: newVariable.name,
    value: newVariable.value,
    type: newVariable.type,
    description: newVariable.description,
    editing: false
  })
  
  // 重置表单
  Object.assign(newVariable, {
    name: '',
    value: '',
    type: 'string',
    description: ''
  })
  
  showAddDialog.value = false
  ElMessage.success('变量已添加')
}
</script>

<style scoped>
.variables-panel {
  height: 100%;
  display: flex;
  flex-direction: column;
  padding: 16px;
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.panel-header h4 {
  margin: 0;
  color: #303133;
}

.variables-list {
  flex: 1;
  overflow: hidden;
}

.el-table {
  border: 1px solid #ebeef5;
}

.el-input,
.el-select {
  width: 100%;
}

.el-button-group .el-button {
  padding: 4px 6px;
}
</style>