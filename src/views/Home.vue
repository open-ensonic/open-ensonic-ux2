<template>
  <div class="home-container">
    <div class="hero-section">
      <div class="hero-content">
        <div class="logo-section">
          <div class="logo">
            <el-icon size="64" color="#409eff">
              <Cpu />
            </el-icon>
          </div>
          <h1 class="title">OpenEnsonic</h1>
          <p class="subtitle">可视化信号处理编辑器</p>
        </div>
        
        <div class="features">
          <div class="feature-card">
            <el-icon size="32" color="#67c23a">
              <Connection />
            </el-icon>
            <h3>可视化编程</h3>
            <p>拖拽式模块连接，直观的信号流程设计</p>
          </div>
          
          <div class="feature-card">
            <el-icon size="32" color="#e6a23c">
              <TrendCharts />
            </el-icon>
            <h3>实时分析</h3>
            <p>实时频谱分析、波形显示和性能监控</p>
          </div>
          
          <div class="feature-card">
            <el-icon size="32" color="#f56c6c">
              <Tools />
            </el-icon>
            <h3>代码生成</h3>
            <p>自动生成Python/C++代码，支持GNU Radio</p>
          </div>
        </div>
        
        <div class="action-buttons">
          <el-button type="primary" size="large" @click="goToEditor">
            <el-icon><Edit /></el-icon>
            开始使用编辑器
          </el-button>
          
          <el-button size="large" @click="showAbout">
            <el-icon><InfoFilled /></el-icon>
            了解更多
          </el-button>
        </div>
      </div>
    </div>
    
    <!-- 关于对话框 -->
    <el-dialog v-model="aboutVisible" title="关于 OpenEnsonic" width="600px">
      <div class="about-content">
        <h3>OpenEnsonic 信号处理编辑器</h3>
        <p>OpenEnsonic 是一个类似 GNU Radio 的可视化信号处理编辑器，专为软件定义无线电(SDR)和数字信号处理(DSP)应用而设计。</p>
        
        <h4>主要特性：</h4>
        <ul>
          <li>🎯 直观的拖拽式界面设计</li>
          <li>📊 丰富的信号处理模块库</li>
          <li>⚡ 实时信号分析和可视化</li>
          <li>🔧 强大的编译器和运行时系统</li>
          <li>📝 自动代码生成(Python/C++)</li>
          <li>🔄 版本控制和项目管理</li>
          <li>🧪 内置沙盒测试环境</li>
        </ul>
        
        <h4>技术栈：</h4>
        <p>Vue 3 + TypeScript + Element Plus + Vite</p>
        
        <div class="version-info">
          <el-tag>版本 1.0.0</el-tag>
          <el-tag type="success">开源项目</el-tag>
        </div>
      </div>
    </el-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { 
  Cpu, Connection, TrendCharts, Tools, Edit, InfoFilled 
} from '@element-plus/icons-vue'

const router = useRouter()
const aboutVisible = ref(false)

const goToEditor = () => {
  router.push('/editor')
}

const showAbout = () => {
  aboutVisible.value = true
}
</script>

<style scoped>
.home-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

.hero-section {
  max-width: 1200px;
  width: 100%;
}

.hero-content {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 60px 40px;
  text-align: center;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
}

.logo-section {
  margin-bottom: 60px;
}

.logo {
  margin-bottom: 20px;
}

.title {
  font-size: 3.5rem;
  font-weight: 700;
  color: #2c3e50;
  margin-bottom: 10px;
  background: linear-gradient(45deg, #409eff, #67c23a);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.subtitle {
  font-size: 1.2rem;
  color: #606266;
  margin-bottom: 0;
}

.features {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 30px;
  margin-bottom: 60px;
}

.feature-card {
  padding: 30px 20px;
  border-radius: 15px;
  background: white;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.feature-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.15);
}

.feature-card h3 {
  font-size: 1.3rem;
  color: #2c3e50;
  margin: 15px 0 10px 0;
}

.feature-card p {
  color: #606266;
  line-height: 1.6;
}

.action-buttons {
  display: flex;
  gap: 20px;
  justify-content: center;
  flex-wrap: wrap;
}

.action-buttons .el-button {
  padding: 15px 30px;
  font-size: 16px;
  border-radius: 10px;
}

.about-content h3 {
  color: #409eff;
  margin-bottom: 15px;
}

.about-content h4 {
  color: #2c3e50;
  margin: 20px 0 10px 0;
}

.about-content ul {
  text-align: left;
  margin: 10px 0;
}

.about-content li {
  margin: 8px 0;
  color: #606266;
}

.version-info {
  margin-top: 20px;
  display: flex;
  gap: 10px;
  justify-content: center;
}

@media (max-width: 768px) {
  .hero-content {
    padding: 40px 20px;
  }
  
  .title {
    font-size: 2.5rem;
  }
  
  .features {
    grid-template-columns: 1fr;
  }
  
  .action-buttons {
    flex-direction: column;
    align-items: center;
  }
  
  .action-buttons .el-button {
    width: 100%;
    max-width: 300px;
  }
}
</style>