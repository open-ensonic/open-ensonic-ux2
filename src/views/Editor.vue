<template>
  <div class="vscode-editor" :class="{ 'light-theme': !isDarkTheme }">
    <!-- 顶部标题栏 -->
    <div class="title-bar">
      <div class="title-bar-left">
        <div class="window-controls">
          <div class="control minimize"></div>
          <div class="control maximize"></div>
          <div class="control close"></div>
        </div>
        <div class="app-title">OpenEnsonic</div>
      </div>
      <div class="title-bar-center">
        <div class="file-path">
          {{ currentFileName }}
          <span v-if="hasUnsavedChanges" class="dirty-indicator">●</span>
        </div>
      </div>
      <div class="title-bar-right">
        <div class="theme-toggle" @click="toggleTheme" :title="isDarkTheme ? '切换到浅色主题' : '切换到深色主题'">
          <el-icon>
            <Sunny v-if="isDarkTheme" />
            <Moon v-else />
          </el-icon>
        </div>
      </div>
    </div>

    <!-- 菜单栏 -->
    <div class="menu-bar">
      <div class="menu-item" v-for="menu in menus" :key="menu.name" @click="toggleMenu(menu.name)">
        {{ menu.label }}
      </div>
    </div>

    <!-- 工具栏 -->
    <div class="main-toolbar">
      <!-- 文件操作组 -->
      <div class="toolbar-buttons">
        <el-button size="small" @click="handleFileAction('open')" title="打开">
          <el-icon><FolderOpened /></el-icon>
        </el-button>
        <el-button size="small" @click="handleFileAction('save')" title="保存">
          <el-icon><Document /></el-icon>
        </el-button>
        <el-button size="small" @click="takeScreenshot" title="截图">
          <el-icon><Camera /></el-icon>
        </el-button>
      </div>

      <el-divider direction="vertical" class="toolbar-divider" />

      <!-- 画布编辑组 -->
      <div class="toolbar-buttons">
        <el-button size="small" @click="deleteSelected" :disabled="selectedNodes.length === 0" title="删除">
          <el-icon><Delete /></el-icon>
        </el-button>
        <el-button size="small" @click="copySelected" :disabled="selectedNodes.length === 0" title="复制">
          <el-icon><CopyDocument /></el-icon>
        </el-button>
        <el-button size="small" @click="pasteNodes" title="粘贴">
          <el-icon><DocumentCopy /></el-icon>
        </el-button>
        <el-button size="small" @click="undo" :disabled="!canUndo" title="撤销">
          <el-icon><RefreshLeft /></el-icon>
        </el-button>
        <el-button size="small" @click="redo" :disabled="!canRedo" title="重做">
          <el-icon><RefreshRight /></el-icon>
        </el-button>
      </div>

      <el-divider direction="vertical" class="toolbar-divider" />

      <!-- 运行控制组 -->
      <div class="toolbar-buttons">
        <el-button size="small" type="success" @click="runFlow" :disabled="isRunning" title="执行">
          <el-icon><VideoPlay /></el-icon>
        </el-button>
        <el-button size="small" type="danger" @click="stopFlow" :disabled="!isRunning" title="终止">
          <el-icon><VideoPause /></el-icon>
        </el-button>
      </div>

      <el-divider direction="vertical" class="toolbar-divider" />

      <!-- 全局设置组 -->
      <div class="toolbar-buttons">
        <el-button size="small" @click="openSettings" title="设置">
          <el-icon><Setting /></el-icon>
        </el-button>
      </div>
    </div>

    <!-- 主要内容区域 -->
    <div class="main-container">
      <!-- 侧边栏 -->
      <div class="sidebar" :class="{ collapsed: sidebarCollapsed }">
        <!-- VSCode风格侧边栏标签 -->
        <div class="vscode-sidebar-tabs">
          <div 
            class="vscode-tab" 
            :class="{ active: activeTab === 'blocks' }"
            @click="activeTab = 'blocks'"
            title="模块库"
          >
            <el-icon><Connection /></el-icon>
          </div>
          <div 
            class="vscode-tab" 
            :class="{ active: activeTab === 'files' }"
            @click="activeTab = 'files'"
            title="文件浏览器"
          >
            <el-icon><Folder /></el-icon>
          </div>
          <div 
            class="vscode-tab" 
            :class="{ active: activeTab === 'search' }"
            @click="activeTab = 'search'"
            title="搜索"
          >
            <el-icon><Search /></el-icon>
          </div>
        </div>

        <!-- 侧边栏内容 -->
        <div class="vscode-sidebar-content" v-show="!sidebarCollapsed">
          <!-- 模块库面板 -->
          <div v-show="activeTab === 'blocks'" class="vscode-panel">
            <div class="vscode-panel-header">
              <span class="panel-title">模块库</span>
              <div class="panel-actions">
                <el-icon class="action-btn" @click="sidebarCollapsed = true" title="折叠">
                  <Close />
                </el-icon>
              </div>
            </div>
            
            <div class="search-container">
              <el-input 
                v-model="searchText" 
                placeholder="搜索模块..." 
                size="small"
                clearable
                class="module-search"
              >
                <template #prefix>
                  <el-icon><Search /></el-icon>
                </template>
              </el-input>
            </div>
            
            <div class="module-tree-container">
              <div 
                v-for="category in blockCategories" 
                :key="category.name"
                class="tree-category"
              >
                <div 
                  class="tree-category-header" 
                  @click="toggleCategory(category.name)"
                >
                  <el-icon 
                    class="tree-expand-icon" 
                    :class="{ expanded: expandedCategories.includes(category.name) }"
                  >
                    <ArrowRight />
                  </el-icon>
                  <el-icon class="tree-category-icon">
                    <component :is="category.icon" />
                  </el-icon>
                  <span class="tree-category-label">{{ category.label }}</span>
                  <span class="tree-item-count">{{ category.blocks.length }}</span>
                </div>
                
                <div 
                  v-show="expandedCategories.includes(category.name)" 
                  class="tree-category-children"
                >
                  <div 
                    v-for="block in category.blocks" 
                    :key="block.name"
                    class="tree-module-item"
                    draggable="true"
                    @dragstart="handleBlockDragStart(block, $event)"
                    @click="selectBlock(block)"
                  >
                    <el-icon class="tree-module-icon">
                      <component :is="block.icon" />
                    </el-icon>
                    <span class="tree-module-label">{{ block.label }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- 文件浏览器面板 -->
          <div v-show="activeTab === 'files'" class="vscode-panel">
            <div class="vscode-panel-header">
              <span class="panel-title">文件浏览器</span>
              <div class="panel-actions">
                <el-icon class="action-btn" title="新建文件">
                  <Plus />
                </el-icon>
                <el-icon class="action-btn" @click="sidebarCollapsed = true" title="折叠">
                  <Close />
                </el-icon>
              </div>
            </div>
            
            <div class="file-tree-container">
              <div class="tree-category">
                <div class="tree-category-header">
                  <el-icon class="tree-expand-icon expanded">
                    <ArrowRight />
                  </el-icon>
                  <el-icon class="tree-category-icon">
                    <Folder />
                  </el-icon>
                  <span class="tree-category-label">OpenEnsonic项目</span>
                </div>
                <div class="tree-category-children">
                  <div class="tree-file-item">
                    <el-icon class="tree-file-icon">
                      <Document />
                    </el-icon>
                    <span class="tree-file-label">main.enc</span>
                  </div>
                  <div class="tree-file-item">
                    <el-icon class="tree-file-icon">
                      <Document />
                    </el-icon>
                    <span class="tree-file-label">filter_demo.enc</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- 搜索面板 -->
          <div v-show="activeTab === 'search'" class="vscode-panel">
            <div class="vscode-panel-header">
              <span class="panel-title">搜索</span>
              <div class="panel-actions">
                <el-icon class="action-btn" @click="sidebarCollapsed = true" title="折叠">
                  <Close />
                </el-icon>
              </div>
            </div>
            
            <div class="search-container">
              <el-input 
                v-model="globalSearchText" 
                placeholder="搜索项目中的内容..." 
                size="small"
                class="global-search"
              >
                <template #prefix>
                  <el-icon><Search /></el-icon>
                </template>
              </el-input>
              <div class="search-results">
                <div class="no-results">
                  <span>暂无搜索结果</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 编辑器区域 -->
      <div class="editor-container">
        <!-- 画布区域 -->
        <div class="canvas-area">
          <div class="canvas-toolbar">
            <div class="toolbar-left">
              <el-button-group size="small">
                <el-button @click="zoomOut" title="缩小">
                  <el-icon><ZoomOut /></el-icon>
                </el-button>
                <el-button disabled>{{ zoomLevel }}%</el-button>
                <el-button @click="zoomIn" title="放大">
                  <el-icon><ZoomIn /></el-icon>
                </el-button>
              </el-button-group>
            </div>
            
            <div class="toolbar-right">
              <span class="canvas-info">节点: {{ flowNodes.length }}</span>
            </div>
          </div>

          <div 
            class="canvas-viewport" 
            ref="canvasViewport"
            @drop="handleCanvasDrop"
            @dragover.prevent
            @click="clearSelection"
          >
            <div class="canvas-grid" :style="{ transform: `scale(${zoomLevel / 100})` }">
              <div class="canvas-content">
                <!-- 流程图节点 -->
                <div 
                  v-for="node in flowNodes" 
                  :key="node.id"
                  class="flow-node"
                  :class="{ selected: selectedNodes.includes(node.id) }"
                  :style="{ left: node.x + 'px', top: node.y + 'px' }"
                  @click.stop="selectNode(node.id)"
                  @mousedown="startDrag(node.id, $event)"
                >
                  <div class="node-header">
                    <el-icon class="node-icon">
                      <component :is="node.icon" />
                    </el-icon>
                    <span class="node-title">{{ node.title }}</span>
                  </div>
                  <div class="node-ports">
                    <div class="input-ports">
                      <div 
                        v-for="port in node.inputPorts" 
                        :key="port.name"
                        class="port input-port"
                        :title="port.name"
                      ></div>
                    </div>
                    <div class="output-ports">
                      <div 
                        v-for="port in node.outputPorts" 
                        :key="port.name"
                        class="port output-port"
                        :title="port.name"
                      ></div>
                    </div>
                  </div>
                </div>

                <!-- 欢迎消息 -->
                <div v-if="flowNodes.length === 0" class="welcome-overlay">
                  <div class="welcome-content">
                    <h2>欢迎使用 OpenEnsonic</h2>
                    <p>从左侧模块库拖拽模块到画布开始构建信号处理流程</p>
                    <div class="quick-actions">
                      <el-button type="primary" @click="createNewFlow">
                        <el-icon><Plus /></el-icon>
                        新建流程图
                      </el-button>
                      <el-button @click="openExistingFlow">
                        <el-icon><FolderOpened /></el-icon>
                        打开现有项目
                      </el-button>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 底部面板 -->
    <div class="bottom-panel" :class="{ collapsed: bottomPanelCollapsed }">
      <div class="panel-header">
        <div class="panel-tabs">
          <div 
            v-for="tab in bottomTabs" 
            :key="tab.name"
            class="panel-tab"
            :class="{ active: activeBottomTab === tab.name }"
            @click="activeBottomTab = tab.name"
          >
            <el-icon><component :is="tab.icon" /></el-icon>
            <span>{{ tab.label }}</span>
            <span v-if="tab.badge" class="tab-badge">{{ tab.badge }}</span>
          </div>
        </div>
        <div class="panel-actions">
          <el-icon class="action-icon" @click="clearConsole" v-if="activeBottomTab === 'console'">
            <Delete />
          </el-icon>
          <el-icon class="action-icon" @click="bottomPanelCollapsed = !bottomPanelCollapsed">
            <ArrowUp v-if="!bottomPanelCollapsed" />
            <ArrowDown v-else />
          </el-icon>
        </div>
      </div>

      <div class="panel-content" v-show="!bottomPanelCollapsed">
        <!-- 控制台 -->
        <div v-show="activeBottomTab === 'console'" class="console-panel">
          <div class="console-output" ref="consoleOutput">
            <div 
              v-for="(log, index) in consoleLogs" 
              :key="index"
              class="log-entry"
              :class="log.level"
            >
              <span class="log-time">{{ log.time }}</span>
              <span class="log-level">[{{ log.level.toUpperCase() }}]</span>
              <span class="log-message">{{ log.message }}</span>
            </div>
          </div>
        </div>

        <!-- 属性面板 -->
        <div v-show="activeBottomTab === 'properties'" class="properties-panel">
          <div v-if="selectedNodes.length === 0" class="no-selection">
            <p>选择一个模块查看其属性</p>
          </div>
          <div v-else class="property-editor">
            <div class="property-group">
              <h4>基本属性</h4>
              <div class="property-item">
                <label>ID:</label>
                <el-input size="small" value="signal_source_0" />
              </div>
              <div class="property-item">
                <label>标题:</label>
                <el-input size="small" value="信号源" />
              </div>
            </div>
          </div>
        </div>

        <!-- 变量面板 -->
        <div v-show="activeBottomTab === 'variables'" class="variables-panel">
          <div class="variables-toolbar">
            <el-button size="small" @click="addVariable">
              <el-icon><Plus /></el-icon>
              添加变量
            </el-button>
          </div>
          <div class="variables-list">
            <div v-for="variable in projectVariables" :key="variable.name" class="variable-item">
              <span class="variable-name">{{ variable.name }}</span>
              <span class="variable-value">{{ variable.value }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 状态栏 -->
    <div class="status-bar">
      <div class="status-left">
        <span class="status-item">
          <el-icon><CircleCheck /></el-icon>
          就绪
        </span>
        <span class="status-item">节点: {{ flowNodes.length }}</span>
      </div>
      <div class="status-right">
        <span class="status-item">{{ zoomLevel }}%</span>
        <span class="status-item">UTF-8</span>
        <span class="status-item">ENC</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, nextTick } from 'vue'
import {
  User, Search, ArrowLeft, ArrowRight, ArrowDown, ArrowUp,
  Folder, Document, Close, Plus, ZoomOut, ZoomIn,
  RefreshLeft, RefreshRight, VideoPlay, VideoPause,
  FolderOpened, Delete, CircleCheck, MoreFilled,
  Camera, CopyDocument, DocumentCopy, Sunny, Moon,
  // 模块图标
  Connection, Cpu, Monitor, Setting, Tools
} from '@element-plus/icons-vue'

// 响应式数据
const sidebarCollapsed = ref(false)
const bottomPanelCollapsed = ref(false)
const activeTab = ref('blocks')
const activeBottomTab = ref('console')
const searchText = ref('')
const globalSearchText = ref('')
const zoomLevel = ref(100)
const canUndo = ref(false)
const canRedo = ref(false)
const isRunning = ref(false)
const selectedNodes = ref<string[]>([])
const expandedCategories = ref(['waveform_generators', 'filters', 'gui_widgets'])
const currentFileName = ref('未命名流程图.enc')
const hasUnsavedChanges = ref(false)
const isDarkTheme = ref(true)

// 菜单数据
const menus = [
  { name: 'file', label: '文件(F)' },
  { name: 'edit', label: '编辑(E)' },
  { name: 'view', label: '视图(V)' },
  { name: 'run', label: '运行(R)' },
  { name: 'help', label: '帮助(H)' }
]

// 侧边栏标签
const sidebarTabs = [
  { name: 'blocks', label: '模块库', icon: 'Connection' },
  { name: 'files', label: '文件', icon: 'Folder' },
  { name: 'search', label: '搜索', icon: 'Search' }
]

// 底部面板标签
const bottomTabs = [
  { name: 'console', label: '控制台', icon: 'Monitor', badge: '2' },
  { name: 'properties', label: '属性', icon: 'Setting' },
  { name: 'variables', label: '变量', icon: 'Tools' }
]

// 模块分类
const blockCategories = [
  {
    name: 'waveform_generators',
    label: '波形生成器',
    icon: 'Connection',
    blocks: [
      { name: 'signal_source', label: '信号源', icon: 'Connection' },
      { name: 'noise_source', label: '噪声源', icon: 'Connection' },
      { name: 'constant_source', label: '常数源', icon: 'Connection' },
      { name: 'vector_source', label: '向量源', icon: 'Connection' }
    ]
  },
  {
    name: 'filters',
    label: '滤波器',
    icon: 'Cpu',
    blocks: [
      { name: 'low_pass_filter', label: '低通滤波器', icon: 'Cpu' },
      { name: 'high_pass_filter', label: '高通滤波器', icon: 'Cpu' },
      { name: 'band_pass_filter', label: '带通滤波器', icon: 'Cpu' },
      { name: 'band_reject_filter', label: '带阻滤波器', icon: 'Cpu' },
      { name: 'hilbert', label: '希尔伯特变换', icon: 'Cpu' },
      { name: 'root_raised_cosine_filter', label: '根升余弦滤波器', icon: 'Cpu' }
    ]
  },
  {
    name: 'gui_widgets',
    label: '图形控件',
    icon: 'Monitor',
    blocks: [
      { name: 'qt_gui_sink', label: 'QT图形显示', icon: 'Monitor' },
      { name: 'qt_gui_time_sink', label: 'QT时域显示', icon: 'Monitor' },
      { name: 'qt_gui_freq_sink', label: 'QT频域显示', icon: 'Monitor' },
      { name: 'qt_gui_waterfall_sink', label: 'QT瀑布图显示', icon: 'Monitor' },
      { name: 'qt_gui_number_sink', label: 'QT数值显示', icon: 'Monitor' },
      { name: 'qt_gui_range', label: 'QT范围控件', icon: 'Setting' }
    ]
  },
  {
    name: 'math_operators',
    label: '数学运算',
    icon: 'Tools',
    blocks: [
      { name: 'add', label: '加法器', icon: 'Tools' },
      { name: 'multiply', label: '乘法器', icon: 'Tools' },
      { name: 'divide', label: '除法器', icon: 'Tools' },
      { name: 'complex_to_mag', label: '复数转幅度', icon: 'Tools' },
      { name: 'complex_to_arg', label: '复数转相位', icon: 'Tools' },
      { name: 'conjugate', label: '共轭', icon: 'Tools' }
    ]
  },
  {
    name: 'fourier_analysis',
    label: '傅里叶分析',
    icon: 'Cpu',
    blocks: [
      { name: 'fft', label: '快速傅里叶变换', icon: 'Cpu' },
      { name: 'fft_vxx', label: 'FFT向量变换', icon: 'Cpu' },
      { name: 'goertzel_fc', label: 'Goertzel算法', icon: 'Cpu' },
      { name: 'stream_to_vector', label: '流转向量', icon: 'Cpu' },
      { name: 'vector_to_stream', label: '向量转流', icon: 'Cpu' }
    ]
  },
  {
    name: 'resamplers',
    label: '重采样器',
    icon: 'Connection',
    blocks: [
      { name: 'rational_resampler', label: '有理重采样器', icon: 'Connection' },
      { name: 'fractional_resampler', label: '分数重采样器', icon: 'Connection' },
      { name: 'keep_one_in_n', label: '保留N中的1', icon: 'Connection' },
      { name: 'repeat', label: '重复器', icon: 'Connection' }
    ]
  },
  {
    name: 'audio',
    label: '音频模块',
    icon: 'Monitor',
    blocks: [
      { name: 'audio_source', label: '音频源', icon: 'Monitor' },
      { name: 'audio_sink', label: '音频输出', icon: 'Monitor' },
      { name: 'wav_file_source', label: 'WAV文件源', icon: 'Document' },
      { name: 'wav_file_sink', label: 'WAV文件输出', icon: 'Document' }
    ]
  },
  {
    name: 'advanced_file',
    label: '高级文件',
    icon: 'Document',
    blocks: [
      { name: 'file_source', label: '文件源', icon: 'Document' },
      { name: 'file_sink', label: '文件输出', icon: 'Document' },
      { name: 'file_meta_source', label: '文件元数据源', icon: 'Document' },
      { name: 'file_meta_sink', label: '文件元数据输出', icon: 'Document' }
    ]
  },
  {
    name: 'channelizers',
    label: '通道化器',
    icon: 'Cpu',
    blocks: [
      { name: 'pfb_channelizer', label: 'PFB通道化器', icon: 'Cpu' },
      { name: 'pfb_synthesizer', label: 'PFB合成器', icon: 'Cpu' },
      { name: 'freq_xlating_fir_filter', label: '频率转换FIR滤波器', icon: 'Cpu' }
    ]
  },
  {
    name: 'debug_tools',
    label: '调试工具',
    icon: 'Setting',
    blocks: [
      { name: 'message_debug', label: '消息调试', icon: 'Setting' },
      { name: 'tag_debug', label: '标签调试', icon: 'Setting' },
      { name: 'probe_signal', label: '信号探测', icon: 'Setting' }
    ]
  },
  {
    name: 'instrumentation',
    label: '仪器显示',
    icon: 'Monitor',
    blocks: [
      { name: 'probe_avg_mag_sqrd', label: '平均幅度平方探测', icon: 'Monitor' },
      { name: 'probe_signal_vf', label: '信号探测VF', icon: 'Monitor' },
      { name: 'statistics', label: '统计分析', icon: 'Monitor' }
    ]
  },
  {
    name: 'message_tools',
    label: '消息工具',
    icon: 'Connection',
    blocks: [
      { name: 'message_strobe', label: '消息频闪', icon: 'Connection' },
      { name: 'message_debug', label: '消息调试', icon: 'Connection' },
      { name: 'pdu_to_tagged_stream', label: 'PDU转标签流', icon: 'Connection' }
    ]
  },
  {
    name: 'peak_detectors',
    label: '峰值检测器',
    icon: 'Tools',
    blocks: [
      { name: 'peak_detector_fb', label: '峰值检测器FB', icon: 'Tools' },
      { name: 'peak_detector2_fb', label: '峰值检测器2FB', icon: 'Tools' }
    ]
  },
  {
    name: 'variables',
    label: '变量模块',
    icon: 'Setting',
    blocks: [
      { name: 'variable', label: '变量', icon: 'Setting' },
      { name: 'variable_slider', label: '变量滑块', icon: 'Setting' },
      { name: 'variable_chooser', label: '变量选择器', icon: 'Setting' }
    ]
  },
  {
    name: 'zeromq_interfaces',
    label: 'ZeroMQ接口',
    icon: 'Connection',
    blocks: [
      { name: 'zeromq_pub_sink', label: 'ZeroMQ发布输出', icon: 'Connection' },
      { name: 'zeromq_sub_source', label: 'ZeroMQ订阅源', icon: 'Connection' },
      { name: 'zeromq_push_sink', label: 'ZeroMQ推送输出', icon: 'Connection' },
      { name: 'zeromq_pull_source', label: 'ZeroMQ拉取源', icon: 'Connection' }
    ]
  },
  {
    name: 'misc',
    label: '杂项模块',
    icon: 'Tools',
    blocks: [
      { name: 'throttle', label: '节流器', icon: 'Tools' },
      { name: 'head', label: '头部截取', icon: 'Tools' },
      { name: 'null_sink', label: '空输出', icon: 'Tools' },
      { name: 'null_source', label: '空源', icon: 'Tools' }
    ]
  }
]

// 流程图节点
const flowNodes = ref<any[]>([])

// 控制台日志
const consoleLogs = ref([
  { time: '23:15:32', level: 'info', message: 'OpenEnsonic 编辑器已启动' },
  { time: '23:15:32', level: 'info', message: '模块库加载完成' }
])

// 项目变量
const projectVariables = ref([
  { name: 'samp_rate', value: '32000' },
  { name: 'center_freq', value: '100e6' }
])

// 方法
const toggleMenu = (menuName: string) => {
  console.log('Toggle menu:', menuName)
}

const toggleCategory = (categoryName: string) => {
  const index = expandedCategories.value.indexOf(categoryName)
  if (index > -1) {
    expandedCategories.value.splice(index, 1)
  } else {
    expandedCategories.value.push(categoryName)
  }
}

const handleBlockDragStart = (block: any, event: DragEvent) => {
  if (event.dataTransfer) {
    event.dataTransfer.setData('application/json', JSON.stringify(block))
  }
}

const handleCanvasDrop = (event: DragEvent) => {
  event.preventDefault()
  const blockData = event.dataTransfer?.getData('application/json')
  if (blockData) {
    const block = JSON.parse(blockData)
    const rect = (event.target as HTMLElement).getBoundingClientRect()
    const x = event.clientX - rect.left
    const y = event.clientY - rect.top
    
    addNodeToCanvas(block, x, y)
  }
}

const addNodeToCanvas = (block: any, x: number, y: number) => {
  const newNode = {
    id: `${block.name}_${Date.now()}`,
    title: block.label,
    icon: block.icon,
    x: x - 75, // 居中
    y: y - 40,
    inputPorts: [{ name: 'in' }],
    outputPorts: [{ name: 'out' }]
  }
  
  flowNodes.value.push(newNode)
  hasUnsavedChanges.value = true
  addLog('info', `添加模块: ${block.label}`)
}

const handleFileAction = (command: string) => {
  switch (command) {
    case 'new':
      createNewFlow()
      break
    case 'open':
      openExistingFlow()
      break
    case 'save':
      saveCurrentFlow()
      break
    case 'saveAs':
      saveAsFlow()
      break
  }
}

const saveCurrentFlow = () => {
  hasUnsavedChanges.value = false
  addLog('info', `保存文件: ${currentFileName.value}`)
}

const saveAsFlow = () => {
  const fileName = prompt('请输入文件名:', currentFileName.value)
  if (fileName) {
    currentFileName.value = fileName
    hasUnsavedChanges.value = false
    addLog('info', `另存为: ${fileName}`)
  }
}

const selectNode = (nodeId: string) => {
  selectedNodes.value = [nodeId]
}

const clearSelection = () => {
  selectedNodes.value = []
}

const startDrag = (nodeId: string, event: MouseEvent) => {
  // 实现节点拖拽逻辑
  console.log('Start drag:', nodeId)
}

const zoomIn = () => {
  zoomLevel.value = Math.min(zoomLevel.value + 10, 200)
}

const zoomOut = () => {
  zoomLevel.value = Math.max(zoomLevel.value - 10, 25)
}

const undo = () => {
  console.log('Undo')
}

const redo = () => {
  console.log('Redo')
}

const runFlow = () => {
  isRunning.value = true
  addLog('info', '开始执行流程图')
  activeBottomTab.value = 'console'
  bottomPanelCollapsed.value = false
}

const stopFlow = () => {
  isRunning.value = false
  addLog('info', '停止执行')
}

const createNewFlow = () => {
  if (hasUnsavedChanges.value) {
    if (!confirm('当前文件有未保存的更改，是否继续？')) {
      return
    }
  }
  flowNodes.value = []
  selectedNodes.value = []
  currentFileName.value = '未命名流程图.enc'
  hasUnsavedChanges.value = false
  addLog('info', '创建新的流程图')
}

const openExistingFlow = () => {
  if (hasUnsavedChanges.value) {
    if (!confirm('当前文件有未保存的更改，是否继续？')) {
      return
    }
  }
  // 这里应该打开文件选择对话框
  const fileName = prompt('请输入要打开的文件名:', 'example.enc')
  if (fileName) {
    currentFileName.value = fileName
    hasUnsavedChanges.value = false
    addLog('info', `打开文件: ${fileName}`)
  }
}

const clearConsole = () => {
  consoleLogs.value = []
}

const addVariable = () => {
  const name = prompt('变量名:')
  const value = prompt('变量值:')
  if (name && value) {
    projectVariables.value.push({ name, value })
  }
}

const addLog = (level: string, message: string) => {
  const now = new Date()
  const time = now.toLocaleTimeString()
  consoleLogs.value.push({ time, level, message })
  
  nextTick(() => {
    const consoleOutput = document.querySelector('.console-output')
    if (consoleOutput) {
      consoleOutput.scrollTop = consoleOutput.scrollHeight
    }
  })
}

const selectBlock = (block: any) => {
  console.log('Select block:', block)
}

// 工具栏功能
const takeScreenshot = () => {
  addLog('info', '正在截图...')
  // 实现截图功能
}

const deleteSelected = () => {
  if (selectedNodes.value.length > 0) {
    flowNodes.value = flowNodes.value.filter(node => !selectedNodes.value.includes(node.id))
    selectedNodes.value = []
    hasUnsavedChanges.value = true
    addLog('info', '删除选中的模块')
  }
}

const copySelected = () => {
  if (selectedNodes.value.length > 0) {
    addLog('info', '复制选中的模块')
    // 实现复制功能
  }
}

const pasteNodes = () => {
  addLog('info', '粘贴模块')
  // 实现粘贴功能
}

const openSettings = () => {
  addLog('info', '打开设置面板')
  // 实现设置功能
}

const toggleTheme = () => {
  isDarkTheme.value = !isDarkTheme.value
  addLog('info', `切换到${isDarkTheme.value ? '深色' : '浅色'}主题`)
}

onMounted(() => {
  addLog('info', '编辑器初始化完成')
})
</script>

<style scoped>
.vscode-editor {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background: #1e1e1e;
  color: #cccccc;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  font-size: 11px;
}

/* 标题栏 */
.title-bar {
  display: flex;
  align-items: center;
  height: 28px;
  background: #323233;
  border-bottom: 1px solid #2d2d30;
  padding: 0 10px;
  -webkit-app-region: drag;
}

.title-bar-left {
  display: flex;
  align-items: center;
  gap: 10px;
}

.window-controls {
  display: flex;
  gap: 8px;
}

.control {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  cursor: pointer;
}

.control.close { background: #ff5f57; }
.control.minimize { background: #ffbd2e; }
.control.maximize { background: #28ca42; }

.app-title {
  font-weight: 600;
  color: #cccccc;
  font-size: 12px;
}

.title-bar-center {
  flex: 1;
  text-align: center;
}

.file-path {
  color: #cccccc;
  font-size: 11px;
  font-weight: 500;
}

.title-bar-right {
  display: flex;
  align-items: center;
}

.theme-toggle {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  cursor: pointer;
  border-radius: 4px;
  transition: background-color 0.2s;
  color: #cccccc;
}

.theme-toggle:hover {
  background: #3e3e42;
}

.theme-toggle .el-icon {
  font-size: 16px;
}

/* 菜单栏 */
.menu-bar {
  display: flex;
  height: 26px;
  background: #2d2d30;
  border-bottom: 1px solid #3e3e42;
}

.menu-item {
  padding: 0 10px;
  display: flex;
  align-items: center;
  cursor: pointer;
  transition: background-color 0.2s;
  font-size: 11px;
}

.menu-item:hover {
  background: #3e3e42;
}

/* 主容器 */
.main-container {
  display: flex;
  flex: 1;
  overflow: hidden;
}

/* VSCode风格侧边栏 */
.sidebar {
  width: 300px;
  background: #252526;
  border-right: 1px solid #3e3e42;
  display: flex;
  transition: width 0.3s ease;
}

.sidebar.collapsed {
  width: 48px;
}

/* VSCode侧边栏标签 */
.vscode-sidebar-tabs {
  width: 48px;
  background: #2c2c2c;
  border-right: 1px solid #3e3e42;
  display: flex;
  flex-direction: column;
  padding: 8px 0;
}

.vscode-tab {
  width: 48px;
  height: 48px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  position: relative;
  transition: all 0.2s;
  color: #858585;
}

.vscode-tab:hover {
  color: #cccccc;
}

.vscode-tab.active {
  color: #ffffff;
}

.vscode-tab.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  width: 2px;
  background: #007acc;
}

.vscode-tab .el-icon {
  font-size: 20px;
}

/* VSCode侧边栏内容 */
.vscode-sidebar-content {
  flex: 1;
  background: #252526;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.vscode-panel {
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.vscode-panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px 12px;
  background: #2d2d30;
  border-bottom: 1px solid #3e3e42;
  min-height: 35px;
}

.panel-title {
  font-weight: 600;
  color: #cccccc;
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.panel-actions {
  display: flex;
  gap: 4px;
}

.action-btn {
  cursor: pointer;
  padding: 2px;
  border-radius: 3px;
  transition: background-color 0.2s;
  color: #858585;
  font-size: 14px;
}

.action-btn:hover {
  background: #3e3e42;
  color: #cccccc;
}

/* 搜索容器 */
.search-container {
  padding: 8px;
  border-bottom: 1px solid #3e3e42;
}

.module-search,
.global-search {
  width: 100%;
}

/* 模块树容器 */
.module-tree-container,
.file-tree-container {
  flex: 1;
  overflow-y: auto;
  padding: 4px 0;
}

/* 树形结构 */
.tree-category {
  margin-bottom: 1px;
}

.tree-category-header {
  display: flex;
  align-items: center;
  padding: 4px 8px;
  cursor: pointer;
  transition: background-color 0.2s;
  gap: 4px;
  font-size: 12px;
}

.tree-category-header:hover {
  background: #2a2d2e;
}

.tree-expand-icon {
  font-size: 12px;
  color: #858585;
  transition: transform 0.2s;
  min-width: 16px;
}

.tree-expand-icon.expanded {
  transform: rotate(90deg);
}

.tree-category-icon {
  font-size: 16px;
  color: #dcb67a;
  min-width: 16px;
}

.tree-category-label {
  flex: 1;
  font-weight: 500;
  color: #cccccc;
}

.tree-item-count {
  font-size: 10px;
  color: #858585;
  background: #3e3e42;
  padding: 1px 4px;
  border-radius: 8px;
  min-width: 16px;
  text-align: center;
}

/* 树形子项 */
.tree-category-children {
  margin-left: 8px;
  border-left: 1px solid #3e3e42;
}

.tree-module-item,
.tree-file-item {
  display: flex;
  align-items: center;
  padding: 2px 8px 2px 16px;
  cursor: pointer;
  transition: background-color 0.2s;
  gap: 6px;
  font-size: 12px;
  position: relative;
}

.tree-module-item:hover,
.tree-file-item:hover {
  background: #2a2d2e;
}

.tree-module-item::before,
.tree-file-item::before {
  content: '';
  position: absolute;
  left: -1px;
  top: 50%;
  width: 8px;
  height: 1px;
  background: #3e3e42;
}

.tree-module-icon,
.tree-file-icon {
  font-size: 14px;
  color: #007acc;
  min-width: 14px;
}

.tree-file-icon {
  color: #519aba;
}

.tree-module-label,
.tree-file-label {
  color: #cccccc;
  font-size: 12px;
}

/* 搜索结果 */
.search-results {
  margin-top: 8px;
  padding: 8px;
}

.no-results {
  text-align: center;
  color: #858585;
  font-size: 12px;
  padding: 20px;
}

/* 编辑器容器 */
.editor-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.dirty-indicator {
  color: #ffb74d;
  font-weight: bold;
  margin-left: 6px;
}

/* 主工具栏 */
.main-toolbar {
  display: flex;
  align-items: center;
  padding: 6px 10px;
  background: #2d2d30;
  border-bottom: 1px solid #3e3e42;
  gap: 12px;
  min-height: 42px;
}

.toolbar-buttons {
  display: flex;
  gap: 2px;
}

.toolbar-buttons .el-button {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 6px;
  width: 32px;
  height: 32px;
  background: transparent;
  border: 1px solid transparent;
  color: #cccccc;
  transition: all 0.2s;
}

.toolbar-buttons .el-button:hover {
  background: #3e3e42;
  border-color: #007acc;
  color: #ffffff;
}

.toolbar-buttons .el-button:disabled {
  color: #666666;
  background: transparent;
  border-color: transparent;
}

.toolbar-buttons .el-button .el-icon {
  font-size: 16px;
}

.toolbar-buttons .el-button--success {
  color: #67c23a;
}

.toolbar-buttons .el-button--success:hover {
  background: #67c23a;
  color: white;
}

.toolbar-buttons .el-button--danger {
  color: #f56c6c;
}

.toolbar-buttons .el-button--danger:hover {
  background: #f56c6c;
  color: white;
}

.toolbar-divider {
  height: 24px;
  margin: 0 6px;
  border-color: #3e3e42;
}

.canvas-info {
  font-size: 11px;
  color: #969696;
}

/* 画布区域 */
.canvas-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.canvas-toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  background: #2d2d30;
  border-bottom: 1px solid #3e3e42;
}

.toolbar-left,
.toolbar-right {
  display: flex;
  align-items: center;
  gap: 8px;
}

.canvas-viewport {
  flex: 1;
  overflow: hidden;
  position: relative;
  background: #1e1e1e;
}

.canvas-grid {
  width: 100%;
  height: 100%;
  background-image: 
    linear-gradient(rgba(255,255,255,0.1) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255,255,255,0.1) 1px, transparent 1px);
  background-size: 20px 20px;
  transform-origin: 0 0;
}

.canvas-content {
  width: 100%;
  height: 100%;
  position: relative;
}

.welcome-overlay {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
  color: #969696;
}

.welcome-content h2 {
  color: #cccccc;
  margin-bottom: 16px;
}

.quick-actions {
  margin-top: 24px;
  display: flex;
  gap: 12px;
}

/* 流程图节点 */
.flow-node {
  position: absolute;
  width: 150px;
  background: #37373d;
  border: 1px solid #3e3e42;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
}

.flow-node:hover {
  border-color: #007acc;
  box-shadow: 0 0 8px rgba(0, 122, 204, 0.3);
}

.flow-node.selected {
  border-color: #007acc;
  box-shadow: 0 0 8px rgba(0, 122, 204, 0.5);
}

.node-header {
  display: flex;
  align-items: center;
  padding: 8px 12px;
  background: #3e3e42;
  border-bottom: 1px solid #464647;
  gap: 6px;
}

.node-icon {
  font-size: 14px;
  color: #007acc;
}

.node-title {
  font-size: 12px;
  font-weight: 500;
}

.node-ports {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
}

.port {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  border: 2px solid #007acc;
  background: #1e1e1e;
  cursor: pointer;
}

.input-port {
  margin-left: -4px;
}

.output-port {
  margin-right: -4px;
}

/* 底部面板 */
.bottom-panel {
  height: 250px;
  background: #252526;
  border-top: 1px solid #3e3e42;
  display: flex;
  flex-direction: column;
  transition: height 0.3s ease;
}

.bottom-panel.collapsed {
  height: 35px;
}

.panel-header {
  display: flex;
  height: 35px;
  background: #2d2d30;
  border-bottom: 1px solid #3e3e42;
}

.panel-tabs {
  display: flex;
  flex: 1;
}

.panel-tab {
  display: flex;
  align-items: center;
  padding: 0 12px;
  gap: 6px;
  cursor: pointer;
  border-right: 1px solid #3e3e42;
  transition: background-color 0.2s;
}

.panel-tab:hover {
  background: #37373d;
}

.panel-tab.active {
  background: #37373d;
  border-bottom: 2px solid #007acc;
}

.tab-badge {
  background: #007acc;
  color: white;
  font-size: 10px;
  padding: 1px 4px;
  border-radius: 8px;
  min-width: 14px;
  text-align: center;
}

.panel-actions {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 0 8px;
}

.panel-content {
  flex: 1;
  overflow: hidden;
}

/* 控制台面板 */
.console-panel {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.console-output {
  flex: 1;
  overflow-y: auto;
  padding: 8px;
  font-family: 'Consolas', 'Monaco', monospace;
  font-size: 12px;
  line-height: 1.4;
}

.log-entry {
  display: flex;
  gap: 8px;
  margin-bottom: 2px;
}

.log-time {
  color: #969696;
  min-width: 60px;
}

.log-level {
  min-width: 50px;
  font-weight: 600;
}

.log-entry.info .log-level {
  color: #4fc3f7;
}

.log-entry.warn .log-level {
  color: #ffb74d;
}

.log-entry.error .log-level {
  color: #f44336;
}

.log-message {
  flex: 1;
}

/* 属性面板 */
.properties-panel {
  padding: 12px;
  height: 100%;
  overflow-y: auto;
}

.no-selection {
  text-align: center;
  color: #969696;
  margin-top: 40px;
}

.property-group {
  margin-bottom: 16px;
}

.property-group h4 {
  margin: 0 0 8px 0;
  color: #cccccc;
  font-size: 13px;
}

.property-item {
  display: flex;
  align-items: center;
  margin-bottom: 8px;
  gap: 8px;
}

.property-item label {
  min-width: 60px;
  font-size: 12px;
  color: #969696;
}

/* 变量面板 */
.variables-panel {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.variables-toolbar {
  padding: 8px;
  border-bottom: 1px solid #3e3e42;
}

.variables-list {
  flex: 1;
  overflow-y: auto;
  padding: 8px;
}

.variable-item {
  display: flex;
  justify-content: space-between;
  padding: 4px 8px;
  border-radius: 3px;
  margin-bottom: 2px;
  transition: background-color 0.2s;
}

.variable-item:hover {
  background: #2a2d2e;
}

.variable-name {
  font-weight: 500;
  color: #9cdcfe;
}

.variable-value {
  color: #ce9178;
}

/* 状态栏 */
.status-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 22px;
  background: #007acc;
  color: white;
  padding: 0 12px;
  font-size: 12px;
}

.status-left,
.status-right {
  display: flex;
  gap: 16px;
}

.status-item {
  display: flex;
  align-items: center;
  gap: 4px;
}

/* 文件树 */
.file-tree {
  padding: 8px;
}

.file-item {
  display: flex;
  align-items: center;
  padding: 4px 8px;
  gap: 6px;
  cursor: pointer;
  border-radius: 3px;
  transition: background-color 0.2s;
}

.file-item:hover {
  background: #2a2d2e;
}

.file-item.folder {
  font-weight: 500;
}

/* 搜索面板 */
.search-panel {
  padding: 8px;
}

/* 响应式设计 */
@media (max-width: 1200px) {
  .sidebar {
    width: 250px;
  }
  
  .sidebar.collapsed {
    width: 48px;
  }
}

/* 滚动条样式 */
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: #1e1e1e;
}

::-webkit-scrollbar-thumb {
  background: #424242;
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: #4f4f4f;
}

/* 浅色主题 */
.vscode-editor.light-theme {
  background: #ffffff;
  color: #333333;
}

/* 浅色主题 - 标题栏 */
.light-theme .title-bar {
  background: #f3f3f3;
  border-bottom: 1px solid #e5e5e5;
  color: #333333;
}

.light-theme .app-title,
.light-theme .file-path {
  color: #333333;
}

.light-theme .theme-toggle {
  color: #333333;
}

.light-theme .theme-toggle:hover {
  background: #e5e5e5;
}

/* 浅色主题 - 菜单栏 */
.light-theme .menu-bar {
  background: #f8f8f8;
  border-bottom: 1px solid #e5e5e5;
}

.light-theme .menu-item {
  color: #333333;
}

.light-theme .menu-item:hover {
  background: #e5e5e5;
}

/* 浅色主题 - 工具栏 */
.light-theme .main-toolbar {
  background: #f8f8f8;
  border-bottom: 1px solid #e5e5e5;
}

.light-theme .toolbar-buttons .el-button {
  color: #333333;
}

.light-theme .toolbar-buttons .el-button:hover {
  background: #e5e5e5;
  border-color: #0078d4;
}

.light-theme .toolbar-divider {
  border-color: #e5e5e5;
}

/* 浅色主题 - 侧边栏 */
.light-theme .sidebar {
  background: #f8f8f8;
  border-right: 1px solid #e5e5e5;
}

.light-theme .vscode-sidebar-tabs {
  background: #f0f0f0;
  border-right: 1px solid #e5e5e5;
}

.light-theme .vscode-tab {
  color: #666666;
}

.light-theme .vscode-tab:hover {
  color: #333333;
}

.light-theme .vscode-tab.active {
  color: #0078d4;
}

.light-theme .vscode-sidebar-content {
  background: #f8f8f8;
}

.light-theme .vscode-panel-header {
  background: #f0f0f0;
  border-bottom: 1px solid #e5e5e5;
}

.light-theme .panel-title {
  color: #333333;
}

.light-theme .action-btn {
  color: #666666;
}

.light-theme .action-btn:hover {
  background: #e5e5e5;
  color: #333333;
}

.light-theme .search-container {
  border-bottom: 1px solid #e5e5e5;
}

.light-theme .tree-category-header:hover {
  background: #e8e8e8;
}

.light-theme .tree-expand-icon {
  color: #666666;
}

.light-theme .tree-category-icon {
  color: #f4a261;
}

.light-theme .tree-category-label {
  color: #333333;
}

.light-theme .tree-item-count {
  color: #666666;
  background: #e5e5e5;
}

.light-theme .tree-category-children {
  border-left: 1px solid #e5e5e5;
}

.light-theme .tree-module-item:hover,
.light-theme .tree-file-item:hover {
  background: #e8e8e8;
}

.light-theme .tree-module-item::before,
.light-theme .tree-file-item::before {
  background: #e5e5e5;
}

.light-theme .tree-module-icon {
  color: #0078d4;
}

.light-theme .tree-file-icon {
  color: #2b88d8;
}

.light-theme .tree-module-label,
.light-theme .tree-file-label {
  color: #333333;
}

/* 浅色主题 - 编辑器区域 */
.light-theme .editor-container {
  background: #ffffff;
}

.light-theme .canvas-area {
  background: #ffffff;
}

.light-theme .canvas-toolbar {
  background: #f8f8f8;
  border-bottom: 1px solid #e5e5e5;
}

.light-theme .canvas-info {
  color: #666666;
}

.light-theme .canvas-viewport {
  background: #ffffff;
}

.light-theme .canvas-grid {
  background-image: 
    linear-gradient(rgba(0,0,0,0.1) 1px, transparent 1px),
    linear-gradient(90deg, rgba(0,0,0,0.1) 1px, transparent 1px);
}

.light-theme .welcome-content h2 {
  color: #333333;
}

.light-theme .welcome-overlay {
  color: #666666;
}

/* 浅色主题 - 流程图节点 */
.light-theme .flow-node {
  background: #ffffff;
  border: 1px solid #e5e5e5;
  color: #333333;
}

.light-theme .flow-node:hover {
  border-color: #0078d4;
  box-shadow: 0 0 8px rgba(0, 120, 212, 0.3);
}

.light-theme .flow-node.selected {
  border-color: #0078d4;
  box-shadow: 0 0 8px rgba(0, 120, 212, 0.5);
}

.light-theme .node-header {
  background: #f0f0f0;
  border-bottom: 1px solid #e5e5e5;
}

.light-theme .node-icon {
  color: #0078d4;
}

.light-theme .port {
  border: 2px solid #0078d4;
  background: #ffffff;
}

/* 浅色主题 - 底部面板 */
.light-theme .bottom-panel {
  background: #f8f8f8;
  border-top: 1px solid #e5e5e5;
}

.light-theme .panel-header {
  background: #f0f0f0;
  border-bottom: 1px solid #e5e5e5;
}

.light-theme .panel-tab {
  border-right: 1px solid #e5e5e5;
  color: #333333;
}

.light-theme .panel-tab:hover {
  background: #e8e8e8;
}

.light-theme .panel-tab.active {
  background: #e8e8e8;
  border-bottom: 2px solid #0078d4;
}

.light-theme .tab-badge {
  background: #0078d4;
}

/* 浅色主题 - 控制台 */
.light-theme .console-output {
  background: #ffffff;
  color: #333333;
}

.light-theme .log-time {
  color: #666666;
}

.light-theme .log-entry.info .log-level {
  color: #0078d4;
}

.light-theme .log-entry.warn .log-level {
  color: #ff8c00;
}

.light-theme .log-entry.error .log-level {
  color: #d13438;
}

/* 浅色主题 - 属性面板 */
.light-theme .properties-panel {
  background: #ffffff;
  color: #333333;
}

.light-theme .no-selection {
  color: #666666;
}

.light-theme .property-group h4 {
  color: #333333;
}

.light-theme .property-item label {
  color: #666666;
}

/* 浅色主题 - 变量面板 */
.light-theme .variables-panel {
  background: #ffffff;
}

.light-theme .variables-toolbar {
  border-bottom: 1px solid #e5e5e5;
}

.light-theme .variable-item:hover {
  background: #f0f0f0;
}

.light-theme .variable-name {
  color: #0078d4;
}

.light-theme .variable-value {
  color: #d73a49;
}

/* 浅色主题 - 状态栏 */
.light-theme .status-bar {
  background: #0078d4;
  color: white;
}

/* 浅色主题 - 搜索结果 */
.light-theme .no-results {
  color: #666666;
}

/* 浅色主题 - 滚动条 */
.light-theme ::-webkit-scrollbar-track {
  background: #f0f0f0;
}

.light-theme ::-webkit-scrollbar-thumb {
  background: #c1c1c1;
}

.light-theme ::-webkit-scrollbar-thumb:hover {
  background: #a8a8a8;
}
</style>