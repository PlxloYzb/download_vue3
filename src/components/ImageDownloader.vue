<template>
  <div class="container">
    <!-- 初始下载按钮 -->
    <button 
      v-if="!showInput" 
      @click="handleButtonClick" 
      class="download-btn"
    >
      获取图片
    </button>

    <!-- 姓名输入框 -->
    <div v-else class="input-container">
      <input 
        v-model="name" 
        type="text" 
        placeholder="请输入姓名"
        @keyup.enter="handleDownload"
      >
      <button @click="handleDownload">确认</button>
    </div>

    <!-- 错误提示 -->
    <p v-if="error" class="error-message">{{ error }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const showInput = ref(false)
const name = ref('')
const error = ref('')

// 检查是否已下载过
const checkIfDownloaded = (name) => {
  return localStorage.getItem(`downloaded_${name}`) === 'true'
}

// 记录下载状态
const markAsDownloaded = (name) => {
  localStorage.setItem(`downloaded_${name}`, 'true')
}

// 处理按钮点击
const handleButtonClick = () => {
  showInput.value = true
  error.value = ''
}

// 处理下载逻辑
const handleDownload = async () => {
  if (!name.value.trim()) {
    error.value = '请输入姓名'
    return
  }

  // 检查是否已下载过
  if (checkIfDownloaded(name.value)) {
    error.value = '该姓名已经下载过图片'
    return
  }

  try {
    // 构建图片URL
    const imageUrl = `/images/${name.value}.png`
    
    // 检查文件是否存在
    const response = await fetch(imageUrl)
    if (!response.ok) {
      throw new Error('未找到对应的图片')
    }

    // 创建下载链接
    const blob = await response.blob()
    const url = window.URL.createObjectURL(blob)
    const a = document.createElement('a')
    a.href = url
    a.download = `${name.value}.png`
    document.body.appendChild(a)
    a.click()
    document.body.removeChild(a)
    window.URL.revokeObjectURL(url)

    // 标记为已下载
    markAsDownloaded(name.value)
    
    // 重置状态
    showInput.value = false
    name.value = ''
    error.value = ''
  } catch (err) {
    error.value = err.message
  }
}
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
}

.download-btn {
  padding: 12px 24px;
  font-size: 18px;
  cursor: pointer;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
}

.input-container {
  display: flex;
  gap: 10px;
}

input {
  padding: 8px;
  font-size: 16px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

button {
  padding: 8px 16px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.error-message {
  color: red;
  margin-top: 10px;
}
</style> 