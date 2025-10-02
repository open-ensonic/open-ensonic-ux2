<template>
  <div class="compiler-panel">
    <div class="compiler-header">
      <h4>OpenEnsonic 编译器</h4>
      <el-button-group size="small">
        <el-button @click="generateCode" type="primary" :loading="isGenerating">
          <el-icon><Cpu /></el-icon>
          生成代码
        </el-button>
        <el-button @click="compileCode" :loading="isCompiling">
          <el-icon><Tools /></el-icon>
          编译
        </el-button>
      </el-button-group>
    </div>
    
    <el-tabs v-model="activeTab" type="card">
      <!-- 配置 -->
      <el-tab-pane label="编译配置" name="config">
        <div class="config-section">
          <el-form :model="compilerConfig" label-width="120px" size="small">
            <el-form-item label="目标语言">
              <el-select v-model="compilerConfig.targetLanguage">
                <el-option label="Python" value="python" />
                <el-option label="C++" value="cpp" />
                <el-option label="GNU Radio" value="gnuradio" />
              </el-select>
            </el-form-item>
            
            <el-form-item label="输出格式">
              <el-select v-model="compilerConfig.outputFormat">
                <el-option label="可执行文件" value="executable" />
                <el-option label="库文件" value="library" />
                <el-option label="源代码" value="source" />
              </el-select>
            </el-form-item>
            
            <el-form-item label="优化级别">
              <el-select v-model="compilerConfig.optimizationLevel">
                <el-option label="无优化 (-O0)" value="O0" />
                <el-option label="基本优化 (-O1)" value="O1" />
                <el-option label="标准优化 (-O2)" value="O2" />
                <el-option label="最大优化 (-O3)" value="O3" />
              </el-select>
            </el-form-item>
            
            <el-form-item label="包含调试信息">
              <el-switch v-model="compilerConfig.includeDebugInfo" />
            </el-form-item>
            
            <el-form-item label="启用并行处理">
              <el-switch v-model="compilerConfig.enableParallel" />
            </el-form-item>
            
            <el-form-item label="输出目录">
              <el-input v-model="compilerConfig.outputDirectory" placeholder="./build" />
            </el-form-item>
            
            <el-form-item label="编译器参数">
              <el-input
                v-model="compilerConfig.compilerFlags"
                type="textarea"
                :rows="3"
                placeholder="额外的编译器参数..."
              />
            </el-form-item>
          </el-form>
        </div>
      </el-tab-pane>
      
      <!-- 生成的代码 -->
      <el-tab-pane label="生成代码" name="generated">
        <div class="code-section">
          <div class="code-header">
            <span>{{ getLanguageDisplayName() }} 代码</span>
            <el-button-group size="small">
              <el-button @click="copyCode">
                <el-icon><CopyDocument /></el-icon>
                复制
              </el-button>
              <el-button @click="saveCode">
                <el-icon><Download /></el-icon>
                保存
              </el-button>
            </el-button-group>
          </div>
          
          <div class="code-content">
            <pre v-if="generatedCode" class="code-block">{{ generatedCode }}</pre>
            <div v-else class="empty-code">
              <el-empty description="暂无生成的代码，请先点击生成代码按钮" />
            </div>
          </div>
        </div>
      </el-tab-pane>
      
      <!-- 编译输出 -->
      <el-tab-pane label="编译输出" name="output">
        <div class="output-section">
          <div class="output-header">
            <span>编译日志</span>
            <el-button size="small" @click="clearOutput">
              <el-icon><Delete /></el-icon>
              清空
            </el-button>
          </div>
          
          <div ref="outputContent" class="output-content">
            <div
              v-for="(log, index) in compileLogs"
              :key="index"
              class="output-line"
              :class="log.type"
            >
              <span class="output-timestamp">{{ formatTime(log.timestamp) }}</span>
              <span class="output-text">{{ log.message }}</span>
            </div>
            
            <div v-if="compileLogs.length === 0" class="empty-output">
              <el-empty description="暂无编译输出" />
            </div>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, nextTick } from 'vue'
import { Cpu, Tools, CopyDocument, Download, Delete } from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus'

// 响应式数据
const activeTab = ref('config')
const isGenerating = ref(false)
const isCompiling = ref(false)
const generatedCode = ref('')
const outputContent = ref<HTMLElement>()

// 编译器配置
const compilerConfig = reactive({
  targetLanguage: 'python',
  outputFormat: 'executable',
  optimizationLevel: 'O2',
  includeDebugInfo: true,
  enableParallel: false,
  outputDirectory: './build',
  compilerFlags: ''
})

// 编译日志
const compileLogs = ref<any[]>([])

// 获取语言显示名称
const getLanguageDisplayName = () => {
  const nameMap: Record<string, string> = {
    python: 'Python',
    cpp: 'C++',
    gnuradio: 'GNU Radio'
  }
  return nameMap[compilerConfig.targetLanguage] || 'Unknown'
}

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
const addLog = (message: string, type = 'info') => {
  compileLogs.value.push({
    message,
    type,
    timestamp: new Date()
  })
  
  nextTick(() => {
    if (outputContent.value) {
      outputContent.value.scrollTop = outputContent.value.scrollHeight
    }
  })
}

// 生成代码
const generateCode = async () => {
  if (isGenerating.value) return
  
  isGenerating.value = true
  addLog('开始生成代码...', 'info')
  
  try {
    // 模拟代码生成过程
    await simulateCodeGeneration()
    activeTab.value = 'generated'
    ElMessage.success('代码生成完成')
  } catch (error) {
    addLog(`代码生成失败: ${error}`, 'error')
    ElMessage.error('代码生成失败')
  } finally {
    isGenerating.value = false
  }
}

// 模拟代码生成
const simulateCodeGeneration = async () => {
  return new Promise((resolve) => {
    setTimeout(() => {
      addLog('分析流程图结构...', 'info')
      
      setTimeout(() => {
        addLog('生成模块连接代码...', 'info')
        
        setTimeout(() => {
          addLog('优化代码结构...', 'info')
          
          // 根据目标语言生成不同的代码
          if (compilerConfig.targetLanguage === 'python') {
            generatedCode.value = generatePythonCode()
          } else if (compilerConfig.targetLanguage === 'cpp') {
            generatedCode.value = generateCppCode()
          } else if (compilerConfig.targetLanguage === 'gnuradio') {
            generatedCode.value = generateGnuRadioCode()
          }
          
          addLog('代码生成完成', 'success')
          resolve(true)
        }, 500)
      }, 500)
    }, 500)
  })
}

// 生成Python代码
const generatePythonCode = () => {
  return `#!/usr/bin/env python3
# -*- coding: utf-8 -*-

"""
OpenEnsonic 生成的 Python 代码
生成时间: ${new Date().toLocaleString()}
"""

import numpy as np
from gnuradio import gr, blocks, analog, filter
from gnuradio.qtgui import Range, RangeWidget
import sys
import signal
from PyQt5 import Qt
from argparse import ArgumentParser
from gnuradio.eng_arg import eng_float, intx
from gnuradio import eng_notation

class OpenEnsonicFlowGraph(gr.top_block, Qt.QWidget):
    def __init__(self):
        gr.top_block.__init__(self, "OpenEnsonic Flow Graph")
        Qt.QWidget.__init__(self)
        self.setWindowTitle("OpenEnsonic Flow Graph")
        
        # 变量定义
        self.samp_rate = samp_rate = 32000
        self.center_freq = center_freq = 100e6
        self.gain = gain = 20
        
        # 模块实例化
        self.analog_sig_source = analog.sig_source_c(
            samp_rate, analog.GR_COS_WAVE, 1000, 1, 0)
        
        self.blocks_throttle = blocks.throttle(
            gr.sizeof_gr_complex*1, samp_rate, True)
        
        self.qtgui_time_sink = qtgui.time_sink_c(
            1024, samp_rate, "", 1)
        
        # 连接模块
        self.connect((self.analog_sig_source, 0), (self.blocks_throttle, 0))
        self.connect((self.blocks_throttle, 0), (self.qtgui_time_sink, 0))
        
    def get_samp_rate(self):
        return self.samp_rate
        
    def set_samp_rate(self, samp_rate):
        self.samp_rate = samp_rate
        self.analog_sig_source.set_sampling_freq(self.samp_rate)
        self.blocks_throttle.set_sample_rate(self.samp_rate)

def main(top_block_cls=OpenEnsonicFlowGraph, options=None):
    from distutils.version import StrictVersion
    
    if StrictVersion("4.5.0") <= StrictVersion(Qt.qVersion()) < StrictVersion("5.0.0"):
        style = gr.prefs().get_string('qtgui', 'style', 'raster')
        Qt.QApplication.setGraphicsSystem(style)
    
    qapp = Qt.QApplication(sys.argv)
    
    tb = top_block_cls()
    tb.start()
    tb.show()
    
    def sig_handler(sig=None, frame=None):
        Qt.QApplication.quit()
    
    signal.signal(signal.SIGINT, sig_handler)
    signal.signal(signal.SIGTERM, sig_handler)
    
    timer = Qt.QTimer()
    timer.start(500)
    timer.timeout.connect(lambda: None)
    
    try:
        qapp.exec_()
    finally:
        tb.stop()
        tb.wait()

if __name__ == '__main__':
    main()
`
}

// 生成C++代码
const generateCppCode = () => {
  return `/*
 * OpenEnsonic 生成的 C++ 代码
 * 生成时间: ${new Date().toLocaleString()}
 */

#include <gnuradio/top_block.h>
#include <gnuradio/blocks/throttle.h>
#include <gnuradio/analog/sig_source_c.h>
#include <gnuradio/qtgui/time_sink_c.h>
#include <iostream>
#include <csignal>

class OpenEnsonicFlowGraph : public gr::top_block
{
private:
    // 变量定义
    double samp_rate;
    double center_freq;
    double gain;
    
    // 模块定义
    gr::blocks::throttle::sptr throttle_block;
    gr::analog::sig_source_c::sptr sig_source;
    gr::qtgui::time_sink_c::sptr time_sink;

public:
    OpenEnsonicFlowGraph() : gr::top_block("OpenEnsonic Flow Graph")
    {
        // 初始化变量
        samp_rate = 32000;
        center_freq = 100e6;
        gain = 20;
        
        // 创建模块实例
        sig_source = gr::analog::sig_source_c::make(
            samp_rate, gr::analog::GR_COS_WAVE, 1000, 1, 0);
            
        throttle_block = gr::blocks::throttle::make(
            sizeof(gr_complex), samp_rate, true);
            
        time_sink = gr::qtgui::time_sink_c::make(
            1024, samp_rate, "", 1);
        
        // 连接模块
        connect(sig_source, 0, throttle_block, 0);
        connect(throttle_block, 0, time_sink, 0);
    }
    
    ~OpenEnsonicFlowGraph() {}
    
    double get_samp_rate() const { return samp_rate; }
    void set_samp_rate(double rate) {
        samp_rate = rate;
        sig_source->set_sampling_freq(samp_rate);
        throttle_block->set_sample_rate(samp_rate);
    }
};

// 信号处理函数
static bool stop_signal_called = false;
void sig_handler(int sig) {
    stop_signal_called = true;
}

int main(int argc, char **argv)
{
    signal(SIGINT, sig_handler);
    signal(SIGTERM, sig_handler);
    
    // 创建流程图
    auto tb = gr::make_top_block("OpenEnsonic Flow Graph");
    auto flow_graph = std::make_shared<OpenEnsonicFlowGraph>();
    
    // 启动流程图
    tb->start();
    
    std::cout << "OpenEnsonic Flow Graph 正在运行..." << std::endl;
    std::cout << "按 Ctrl+C 停止" << std::endl;
    
    // 等待停止信号
    while (!stop_signal_called) {
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
    
    // 停止流程图
    tb->stop();
    tb->wait();
    
    std::cout << "流程图已停止" << std::endl;
    return 0;
}
`
}

// 生成GNU Radio代码
const generateGnuRadioCode = () => {
  return `options:
  parameters:
    author: OpenEnsonic
    category: '[GRC Hier Blocks]'
    cmake_opt: ''
    comment: ''
    copyright: ''
    description: 'OpenEnsonic 生成的 GNU Radio 流程图'
    gen_cmake: 'On'
    gen_linking: dynamic
    generate_options: qt_gui
    hier_block_src_path: '.:'
    id: openensonic_flowgraph
    max_nouts: '0'
    output_language: python
    placement: (0,0)
    qt_qss_theme: ''
    realtime_scheduling: ''
    run: 'True'
    run_command: '{python} -u {filename}'
    run_options: prompt
    sizing_mode: fixed
    thread_safe_setters: ''
    title: OpenEnsonic Flow Graph
    window_size: ''
  states:
    bus_sink: false
    bus_source: false
    bus_structure: null
    coordinate: [8, 8]
    rotation: 0
    state: enabled

blocks:
- name: samp_rate
  id: variable
  parameters:
    comment: ''
    value: '32000'
  states:
    bus_sink: false
    bus_source: false
    bus_structure: null
    coordinate: [184, 12]
    rotation: 0
    state: enabled

- name: analog_sig_source_x_0
  id: analog_sig_source_x
  parameters:
    affinity: ''
    alias: ''
    amp: '1'
    comment: ''
    freq: '1000'
    maxoutbuf: '0'
    minoutbuf: '0'
    offset: '0'
    phase: '0'
    samp_rate: samp_rate
    type: complex
    waveform: analog.GR_COS_WAVE
  states:
    bus_sink: false
    bus_source: false
    bus_structure: null
    coordinate: [200, 156]
    rotation: 0
    state: true

- name: blocks_throttle_0
  id: blocks_throttle
  parameters:
    affinity: ''
    alias: ''
    comment: ''
    ignoretag: 'True'
    maxoutbuf: '0'
    minoutbuf: '0'
    samples_per_second: samp_rate
    type: complex
    vlen: '1'
  states:
    bus_sink: false
    bus_source: false
    bus_structure: null
    coordinate: [416, 196]
    rotation: 0
    state: enabled

- name: qtgui_time_sink_x_0
  id: qtgui_time_sink_x
  parameters:
    affinity: ''
    alias: ''
    alpha1: '1.0'
    alpha10: '1.0'
    alpha2: '1.0'
    alpha3: '1.0'
    alpha4: '1.0'
    alpha5: '1.0'
    alpha6: '1.0'
    alpha7: '1.0'
    alpha8: '1.0'
    alpha9: '1.0'
    autoscale: 'False'
    axislabels: 'True'
    color1: blue
    color10: dark blue
    color2: red
    color3: green
    color4: black
    color5: cyan
    color6: magenta
    color7: yellow
    color8: dark red
    color9: dark green
    comment: ''
    ctrlpanel: 'False'
    entags: 'True'
    grid: 'False'
    gui_hint: ''
    label1: Signal 1
    label10: Signal 10
    label2: Signal 2
    label3: Signal 3
    label4: Signal 4
    label5: Signal 5
    label6: Signal 6
    label7: Signal 7
    label8: Signal 8
    label9: Signal 9
    legend: 'True'
    marker1: '-1'
    marker10: '-1'
    marker2: '-1'
    marker3: '-1'
    marker4: '-1'
    marker5: '-1'
    marker6: '-1'
    marker7: '-1'
    marker8: '-1'
    marker9: '-1'
    name: '""'
    nconnections: '1'
    size: '1024'
    srate: samp_rate
    stemplot: 'False'
    style1: '1'
    style10: '1'
    style2: '1'
    style3: '1'
    style4: '1'
    style5: '1'
    style6: '1'
    style7: '1'
    style8: '1'
    style9: '1'
    tr_chan: '0'
    tr_delay: '0'
    tr_level: '0.0'
    tr_mode: qtgui.TRIG_MODE_FREE
    tr_slope: qtgui.TRIG_SLOPE_POS
    tr_tag: '""'
    type: complex
    update_time: '0.10'
    width1: '1'
    width10: '1'
    width2: '1'
    width3: '1'
    width4: '1'
    width5: '1'
    width6: '1'
    width7: '1'
    width8: '1'
    width9: '1'
    ylabel: Amplitude
    ymax: '1'
    ymin: '-1'
    yunit: '""'
  states:
    bus_sink: false
    bus_source: false
    bus_structure: null
    coordinate: [632, 156]
    rotation: 0
    state: true

connections:
- [analog_sig_source_x_0, '0', blocks_throttle_0, '0']
- [blocks_throttle_0, '0', qtgui_time_sink_x_0, '0']

metadata:
  file_format: 1
`
}

// 编译代码
const compileCode = async () => {
  if (isCompiling.value) return
  
  if (!generatedCode.value) {
    ElMessage.warning('请先生成代码')
    return
  }
  
  isCompiling.value = true
  addLog('开始编译...', 'info')
  
  try {
    await simulateCompilation()
    activeTab.value = 'output'
    ElMessage.success('编译完成')
  } catch (error) {
    addLog(`编译失败: ${error}`, 'error')
    ElMessage.error('编译失败')
  } finally {
    isCompiling.value = false
  }
}

// 模拟编译过程
const simulateCompilation = async () => {
  return new Promise((resolve) => {
    setTimeout(() => {
      addLog('检查依赖项...', 'info')
      
      setTimeout(() => {
        addLog('编译器配置检查通过', 'success')
        addLog(`目标语言: ${getLanguageDisplayName()}`, 'info')
        addLog(`优化级别: ${compilerConfig.optimizationLevel}`, 'info')
        addLog(`输出目录: ${compilerConfig.outputDirectory}`, 'info')
        
        setTimeout(() => {
          addLog('开始编译源代码...', 'info')
          
          setTimeout(() => {
            addLog('链接库文件...', 'info')
            
            setTimeout(() => {
              addLog('生成可执行文件...', 'info')
              addLog('编译成功完成', 'success')
              addLog(`输出文件: ${compilerConfig.outputDirectory}/openensonic_flowgraph`, 'success')
              resolve(true)
            }, 800)
          }, 600)
        }, 400)
      }, 300)
    }, 200)
  })
}

// 复制代码
const copyCode = async () => {
  if (!generatedCode.value) {
    ElMessage.warning('没有可复制的代码')
    return
  }
  
  try {
    await navigator.clipboard.writeText(generatedCode.value)
    ElMessage.success('代码已复制到剪贴板')
  } catch (error) {
    ElMessage.error('复制失败')
  }
}

// 保存代码
const saveCode = () => {
  if (!generatedCode.value) {
    ElMessage.warning('没有可保存的代码')
    return
  }
  
  const extensions: Record<string, string> = {
    python: 'py',
    cpp: 'cpp',
    gnuradio: 'grc'
  }
  
  const extension = extensions[compilerConfig.targetLanguage] || 'txt'
  const filename = `openensonic_flowgraph.${extension}`
  
  const blob = new Blob([generatedCode.value], { type: 'text/plain' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = filename
  a.click()
  URL.revokeObjectURL(url)
  
  ElMessage.success('代码已保存')
}

// 清空输出
const clearOutput = () => {
  compileLogs.value = []
  ElMessage.success('输出已清空')
}
</script>

<style scoped>
.compiler-panel {
  height: 100%;
  display: flex;
  flex-direction: column;
  padding: 16px;
}

.compiler-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.compiler-header h4 {
  margin: 0;
  color: #303133;
}

.config-section {
  padding: 16px;
  overflow-y: auto;
}

.code-section {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.code-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  background: #fafafa;
  border-bottom: 1px solid #e4e7ed;
  font-weight: 500;
  color: #303133;
}

.code-content {
  flex: 1;
  overflow: auto;
  background: #1e1e1e;
}

.code-block {
  margin: 0;
  padding: 16px;
  color: #d4d4d4;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  font-size: 13px;
  line-height: 1.5;
  white-space: pre-wrap;
  word-wrap: break-word;
}

.empty-code {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  background: white;
}

.output-section {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.output-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  background: #2d2d30;
  color: #d4d4d4;
  font-weight: 500;
}

.output-content {
  flex: 1;
  overflow-y: auto;
  padding: 12px 16px;
  background: #1e1e1e;
  color: #d4d4d4;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  font-size: 12px;
  line-height: 1.4;
}

.output-line {
  display: flex;
  margin: 2px 0;
  padding: 2px 0;
}

.output-timestamp {
  color: #808080;
  margin-right: 8px;
  min-width: 80px;
  font-size: 11px;
}

.output-text {
  flex: 1;
  word-break: break-all;
}

.output-line.info .output-text {
  color: #4ec9b0;
}

.output-line.success .output-text {
  color: #4caf50;
}

.output-line.warning .output-text {
  color: #ff9800;
}

.output-line.error .output-text {
  color: #f44747;
}

.empty-output {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: #808080;
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

/* 滚动条样式 */
.code-content::-webkit-scrollbar,
.output-content::-webkit-scrollbar {
  width: 8px;
}

.code-content::-webkit-scrollbar-track,
.output-content::-webkit-scrollbar-track {
  background: #2d2d30;
}

.code-content::-webkit-scrollbar-thumb,
.output-content::-webkit-scrollbar-thumb {
  background: #464647;
  border-radius: 4px;
}

.code-content::-webkit-scrollbar-thumb:hover,
.output-content::-webkit-scrollbar-thumb:hover {
  background: #5a5a5a;
}
</style>