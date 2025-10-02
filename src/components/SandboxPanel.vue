<template>
  <div class="sandbox-panel">
    <div class="sandbox-header">
      <div class="language-tabs">
        <el-radio-group v-model="activeLanguage" size="small">
          <el-radio-button label="python">Python</el-radio-button>
          <el-radio-button label="cpp">C++</el-radio-button>
        </el-radio-group>
      </div>
      
      <div class="sandbox-controls">
        <el-button-group size="small">
          <el-button @click="runCode" :loading="isRunning" type="success">
            <el-icon><VideoPlay /></el-icon>
            运行
          </el-button>
          <el-button @click="stopCode" :disabled="!isRunning" type="danger">
            <el-icon><VideoPause /></el-icon>
            停止
          </el-button>
          <el-button @click="clearOutput">
            <el-icon><Delete /></el-icon>
            清空输出
          </el-button>
        </el-button-group>
      </div>
    </div>
    
    <div class="sandbox-content">
      <div class="code-editor">
        <div class="editor-header">
          <span>代码编辑器</span>
          <el-button-group size="small">
            <el-button @click="loadTemplate">
              <el-icon><Document /></el-icon>
              模板
            </el-button>
            <el-button @click="saveCode">
              <el-icon><Download /></el-icon>
              保存
            </el-button>
          </el-button-group>
        </div>
        <textarea
          ref="codeEditor"
          v-model="codeContent[activeLanguage]"
          class="code-textarea"
          :placeholder="getPlaceholder()"
        ></textarea>
      </div>
      
      <div class="output-panel">
        <div class="output-header">
          <span>输出结果</span>
          <el-tag v-if="isRunning" type="success" size="small">
            <el-icon><Loading /></el-icon>
            运行中
          </el-tag>
        </div>
        <div ref="outputContent" class="output-content">
          <div
            v-for="(output, index) in outputs"
            :key="index"
            class="output-line"
            :class="output.type"
          >
            <span class="output-timestamp">{{ formatTime(output.timestamp) }}</span>
            <span class="output-text">{{ output.text }}</span>
          </div>
          
          <div v-if="outputs.length === 0" class="empty-output">
            <el-empty description="暂无输出" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, nextTick } from 'vue'
import { VideoPlay, VideoPause, Delete, Document, Download, Loading } from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus'

// 响应式数据
const activeLanguage = ref('python')
const isRunning = ref(false)
const codeEditor = ref<HTMLTextAreaElement>()
const outputContent = ref<HTMLElement>()

// 代码内容
const codeContent = reactive({
  python: `# Python 代码示例
import numpy as np
import matplotlib.pyplot as plt

# 生成信号
fs = 1000  # 采样频率
t = np.linspace(0, 1, fs)
freq = 50  # 信号频率
signal = np.sin(2 * np.pi * freq * t)

# 添加噪声
noise = 0.1 * np.random.randn(len(t))
noisy_signal = signal + noise

print(f"信号长度: {len(signal)}")
print(f"采样频率: {fs} Hz")
print(f"信号频率: {freq} Hz")
`,
  cpp: `// C++ 代码示例
#include <iostream>
#include <vector>
#include <cmath>
#include <random>

int main() {
    const int fs = 1000;  // 采样频率
    const double freq = 50.0;  // 信号频率
    const double duration = 1.0;  // 持续时间
    
    std::vector<double> signal;
    std::vector<double> time;
    
    // 生成时间序列
    for (int i = 0; i < fs; ++i) {
        double t = i / static_cast<double>(fs);
        time.push_back(t);
        signal.push_back(std::sin(2 * M_PI * freq * t));
    }
    
    std::cout << "信号长度: " << signal.size() << std::endl;
    std::cout << "采样频率: " << fs << " Hz" << std::endl;
    std::cout << "信号频率: " << freq << " Hz" << std::endl;
    
    return 0;
}
`
})

// 输出内容
const outputs = ref<any[]>([])

// 获取占位符文本
const getPlaceholder = () => {
  return activeLanguage.value === 'python' 
    ? '在此输入 Python 代码...'
    : '在此输入 C++ 代码...'
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

// 添加输出
const addOutput = (text: string, type = 'info') => {
  outputs.value.push({
    text,
    type,
    timestamp: new Date()
  })
  
  nextTick(() => {
    if (outputContent.value) {
      outputContent.value.scrollTop = outputContent.value.scrollHeight
    }
  })
}

// 运行代码
const runCode = async () => {
  if (isRunning.value) return
  
  const code = codeContent[activeLanguage.value].trim()
  if (!code) {
    ElMessage.warning('请输入代码')
    return
  }
  
  isRunning.value = true
  addOutput(`开始执行 ${activeLanguage.value.toUpperCase()} 代码...`, 'info')
  
  try {
    // 模拟代码执行
    await simulateCodeExecution(code)
  } catch (error) {
    addOutput(`执行错误: ${error}`, 'error')
  } finally {
    isRunning.value = false
    addOutput('代码执行完成', 'success')
  }
}

// 模拟代码执行
const simulateCodeExecution = async (code: string) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      if (activeLanguage.value === 'python') {
        addOutput('Python 解释器启动', 'info')
        addOutput('导入模块: numpy, matplotlib', 'info')
        addOutput('信号长度: 1000', 'output')
        addOutput('采样频率: 1000 Hz', 'output')
        addOutput('信号频率: 50 Hz', 'output')
        addOutput('信号生成完成', 'success')
      } else {
        addOutput('C++ 编译器启动', 'info')
        addOutput('编译中...', 'info')
        addOutput('编译成功', 'success')
        addOutput('程序执行中...', 'info')
        addOutput('信号长度: 1000', 'output')
        addOutput('采样频率: 1000 Hz', 'output')
        addOutput('信号频率: 50 Hz', 'output')
        addOutput('程序执行完成', 'success')
      }
      resolve(true)
    }, 2000)
  })
}

// 停止代码执行
const stopCode = () => {
  if (!isRunning.value) return
  
  isRunning.value = false
  addOutput('代码执行已停止', 'warning')
}

// 清空输出
const clearOutput = () => {
  outputs.value = []
  ElMessage.success('输出已清空')
}

// 加载模板
const loadTemplate = () => {
  const templates = {
    python: `# GNU Radio Python 模块模板
#!/usr/bin/env python3

import numpy as np
from gnuradio import gr

class custom_block(gr.sync_block):
    """
    自定义信号处理模块
    """
    def __init__(self):
        gr.sync_block.__init__(
            self,
            name="custom_block",
            in_sig=[np.complex64],
            out_sig=[np.complex64]
        )
    
    def work(self, input_items, output_items):
        in0 = input_items[0]
        out = output_items[0]
        
        # 在这里实现你的信号处理逻辑
        out[:] = in0 * 2.0  # 示例: 信号放大2倍
        
        return len(output_items[0])
`,
    cpp: `// GNU Radio C++ 模块模板
#ifndef INCLUDED_CUSTOM_BLOCK_H
#define INCLUDED_CUSTOM_BLOCK_H

#include <gnuradio/sync_block.h>

class custom_block : virtual public gr::sync_block
{
public:
    typedef boost::shared_ptr<custom_block> sptr;
    
    static sptr make();
    
    virtual int work(int noutput_items,
                     gr_vector_const_void_star &input_items,
                     gr_vector_void_star &output_items) = 0;
};

#endif /* INCLUDED_CUSTOM_BLOCK_H */

// 实现文件
#include "custom_block.h"
#include <gnuradio/io_signature.h>

custom_block::sptr custom_block::make()
{
    return gnuradio::get_initial_sptr(new custom_block_impl());
}

int custom_block::work(int noutput_items,
                       gr_vector_const_void_star &input_items,
                       gr_vector_void_star &output_items)
{
    const gr_complex *in = (const gr_complex *) input_items[0];
    gr_complex *out = (gr_complex *) output_items[0];
    
    // 在这里实现你的信号处理逻辑
    for (int i = 0; i < noutput_items; i++) {
        out[i] = in[i] * gr_complex(2.0, 0.0);  // 示例: 信号放大2倍
    }
    
    return noutput_items;
}
`
  }
  
  codeContent[activeLanguage.value] = templates[activeLanguage.value]
  ElMessage.success('模板已加载')
}

// 保存代码
const saveCode = () => {
  const code = codeContent[activeLanguage.value]
  const extension = activeLanguage.value === 'python' ? 'py' : 'cpp'
  const filename = `sandbox_code.${extension}`
  
  const blob = new Blob([code], { type: 'text/plain' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = filename
  a.click()
  URL.revokeObjectURL(url)
  
  ElMessage.success('代码已保存')
}
</script>

<style scoped>
.sandbox-panel {
  height: 100%;
  display: flex;
  flex-direction: column;
  background: #f5f7fa;
}

.sandbox-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  background: white;
  border-bottom: 1px solid #e4e7ed;
}

.sandbox-content {
  flex: 1;
  display: flex;
  gap: 1px;
  overflow: hidden;
}

.code-editor,
.output-panel {
  flex: 1;
  display: flex;
  flex-direction: column;
  background: white;
}

.editor-header,
.output-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  background: #fafafa;
  border-bottom: 1px solid #e4e7ed;
  font-size: 13px;
  font-weight: 500;
  color: #303133;
}

.code-textarea {
  flex: 1;
  border: none;
  outline: none;
  padding: 16px;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  font-size: 13px;
  line-height: 1.5;
  resize: none;
  background: #1e1e1e;
  color: #d4d4d4;
}

.code-textarea::placeholder {
  color: #6a6a6a;
}

.output-content {
  flex: 1;
  overflow-y: auto;
  padding: 12px;
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

.output-line.output .output-text {
  color: #d4d4d4;
  font-weight: 500;
}

.empty-output {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: #808080;
}

/* 滚动条样式 */
.output-content::-webkit-scrollbar {
  width: 8px;
}

.output-content::-webkit-scrollbar-track {
  background: #2d2d30;
}

.output-content::-webkit-scrollbar-thumb {
  background: #464647;
  border-radius: 4px;
}

.output-content::-webkit-scrollbar-thumb:hover {
  background: #5a5a5a;
}
</style>