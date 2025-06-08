.output-path {
  margin-bottom: 1rem;
}

.output-path label {
  display: inline-block;
  width: 80px;
  font-weight: bold;
}

.path-display {
  font-family: monospace;
  background-color: #f5f5f5;
  padding: 0.5rem;
  border-radius: 4px;
  border: 1px solid #ddd;
}<template>
  <div class="container">
    <h1>🧾 JSON 编辑器（Tauri 桌面版）</h1>
     
    <div class="buttons">
      <button @click="openFile">📂 打开 JSON 文件</button>
      <button @click="selectOutputDir">📁 选择输出目录</button>
      <button @click="saveFile" :disabled="!jsonContent.trim()">💾 保存</button>
      <button @click="saveToSelectedDir" :disabled="!jsonContent.trim() || !outputDir">💾 保存到选择的目录</button>
    </div>
     
    <div class="output-path" v-if="outputDir">
      <label>选择的目录：</label>
      <span class="path-display">{{ outputDir }}</span>
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
import { open } from '@tauri-apps/plugin-dialog'
import { readTextFile, writeTextFile } from '@tauri-apps/plugin-fs'

const filePath = ref('')
const jsonContent = ref('')
const outputDir = ref('') // 选择的输出目录

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

async function selectOutputDir() {
  const selectedDir = await open({
    directory: true,
    multiple: false
  })
  
  if (selectedDir) {
    outputDir.value = selectedDir
    alert(`✅ 已选择输出目录：${selectedDir}`)
  }
}

async function saveFile() {
  if (!filePath.value) {
    // 如果没有当前文件路径，则保存到选择的目录
    await saveToSelectedDir()
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

async function saveToSelectedDir() {
  if (!jsonContent.value.trim()) {
    alert('❌ 请先输入 JSON 内容')
    return
  }
  
  if (!outputDir.value) {
    alert('❌ 请先选择输出目录')
    return
  }
  
  try {
    // 验证 JSON 格式
    const parsed = JSON.parse(jsonContent.value)
    
    // 生成文件名（使用时间戳避免重名）
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-').slice(0, -5)
    const fileName = `json-${timestamp}.json`
    const fullPath = `${outputDir.value}/${fileName}`
    
    await writeTextFile(fullPath, JSON.stringify(parsed, null, 2))
    alert(`✅ 文件已保存到：${fullPath}`)
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