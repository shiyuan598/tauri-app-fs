<template>
  <div class="container">
    <h1>🧾 JSON 编辑器（Tauri 桌面版）</h1>
     
    <div class="buttons">
      <button @click="openFile">📂 打开 JSON 文件</button>
      <button @click="saveFile" :disabled="!jsonContent.trim()">💾 保存</button>
      <button @click="saveAsFile" :disabled="!jsonContent.trim()">💾 另存为</button>
    </div>
     
    <p v-if="filePath"><strong>当前文件：</strong>{{ filePath }}</p>
     
    <textarea
      v-model="jsonContent"
      placeholder="请选择 JSON 文件或直接输入 JSON 内容..."
      rows="20"
      cols="80"
    ></textarea>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { open, save } from '@tauri-apps/plugin-dialog'
import { readTextFile, writeTextFile } from '@tauri-apps/plugin-fs'

const filePath = ref('')
const jsonContent = ref('')

async function openFile() {
  const path = await open({
    filters: [{ name: 'JSON', extensions: ['json'] }]
  })
   
  if (path) {
    filePath.value = path
    try {
      const text = await readTextFile(path)
      const parsed = JSON.parse(text)
      jsonContent.value = JSON.stringify(parsed, null, 2)
    } catch (err) {
      alert('⚠️ 读取失败：' + err)
    }
  }
}

async function saveFile() {
  if (!filePath.value) {
    // 如果没有当前文件路径，则调用另存为
    await saveAsFile()
    return
  }
  
  try {
    const parsed = JSON.parse(jsonContent.value)
    await writeTextFile(filePath.value, JSON.stringify(parsed, null, 2))
    alert('✅ 已保存！')
  } catch (err) {
    alert('❌ 保存失败：' + err.message)
  }
}

async function saveAsFile() {
  if (!jsonContent.value.trim()) return
  
  try {
    // 验证 JSON 格式
    const parsed = JSON.parse(jsonContent.value)
    
    // 打开保存对话框
    const savePath = await save({
      filters: [{ name: 'JSON', extensions: ['json'] }],
      defaultPath: filePath.value || 'untitled.json'
    })
    
    if (savePath) {
      await writeTextFile(savePath, JSON.stringify(parsed, null, 2))
      filePath.value = savePath // 更新当前文件路径
      alert('✅ 文件已保存到：' + savePath)
    }
  } catch (err) {
    if (err instanceof SyntaxError) {
      alert('❌ JSON 格式错误，请检查语法')
    } else {
      alert('❌ 保存失败：' + err.message)
    }
  }
}
</script>

<style scoped>
.container {
  padding: 2rem;
  font-family: sans-serif;
}

textarea {
  width: 100%;
  font-family: monospace;
  margin-top: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 0.5rem;
}

.buttons {
  margin-bottom: 1rem;
}

button {
  margin-right: 1rem;
  padding: 0.5em 1em;
  border: none;
  border-radius: 4px;
  background-color: #007acc;
  color: white;
  cursor: pointer;
}

button:hover:not(:disabled) {
  background-color: #005a9e;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
</style>