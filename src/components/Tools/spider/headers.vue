<template>
  <div class="flex flex-col mt-3 flex-1">
    <!-- 左右分栏布局 -->
    <div class="p-4 rounded-2xl bg-white h-[600px] flex gap-4">
      <!-- 左侧：Headers 输入框 -->
      <div class="flex-1 border z-10 mt-3 flex flex-col min-w-[50%]">
        <label class="mb-2 font-bold">输入 Headers</label>
        <textarea
            v-model="rawHeaders"
            class="flex-1 p-2 border rounded resize-none overflow-auto"
            placeholder="在这里输入 Headers（每行一个键值对，如：Key: Value）"
        ></textarea>
      </div>

      <!-- 右侧：格式化后的 JSON 展示 -->
      <div class="flex-1 p-4 border overflow-y-auto max-w-[50%]">
        <h3 class="font-bold mb-2">格式化后的 JSON</h3>
        <pre v-if="formattedJson" class="text-sm text-gray-800">{{ formattedJson }}</pre>
        <div v-else class="text-gray-500">暂无格式化内容</div>
      </div>
      <el-button type="primary" @click="copyJson">复制HEADER</el-button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

// 原始 Headers 输入
const rawHeaders = ref(`Content-Type: application/json
Authorization: Bearer token
Accept: */*`)
import { copy } from '@/utils/string'

const copyJson = () => {
  if (formattedJson.value) {
    copy(formattedJson.value)
  }
}
// 格式化后的 Headers 转换为 JSON
const formattedJson = computed(() => {
  const cleanedHeaders = rawHeaders.value.replace(/:\s*\n/g, ': ')
  const headersArray = cleanedHeaders
      .split('\n') // 按行分割
      .map((line) => line.trim()) // 去除多余空格
      .filter((line) => line.includes(':')) // 筛选包含冒号的行
      .map((line) => {
        const [key, ...valueParts] = line.split(':') // 分割键值对
        return {
          key: key.trim(),
          value: valueParts.join(':').trim(), // 处理值中可能包含的冒号
        }
      })

  // 转换为 JSON 格式
  const headersObject = Object.fromEntries(
      headersArray.map((header) => [header.key, header.value])
  )

  // 返回格式化的 JSON 字符串
  return headersArray.length > 0 ? JSON.stringify(headersObject, null, 2) : ''
})
</script>

<style scoped>
/* 添加样式 */
textarea {
  height: 100%; /* 确保 textarea 填充父容器高度 */
  font-family: monospace; /* 使用等宽字体 */
  font-size: 14px; /* 设置字体大小 */
}

pre {
  white-space: pre-wrap; /* 自动换行 */
  word-break: break-word; /* 长单词自动断行 */
  background-color: #f5f5f5; /* 背景颜色 */
  padding: 10px; /* 内边距 */
  border-radius: 4px; /* 圆角 */
}
</style>
