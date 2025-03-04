<template>
  <div class="flex flex-col mt-3 flex-1">
    <!-- 左右分栏布局 -->
    <div class="p-4 rounded-2xl bg-white h-[600px] flex gap-4">
      <!-- 左侧：Cookie 输入框 -->
      <div class="flex-1 border z-10 mt-3 flex flex-col min-w-[50%]">
        <label class="mb-2 font-bold">输入 Cookie</label>
        <textarea
            v-model="rawCookies"
            class="flex-1 p-2 border rounded resize-none overflow-auto"
            placeholder="在这里输入 Cookie（如：key1=value1; key2=value2）"
        ></textarea>
      </div>

      <!-- 右侧：格式化后的 JSON 展示 -->
      <div class="flex-1 p-4 border overflow-y-auto max-w-[50%]">
        <h3 class="font-bold mb-2">格式化后的 JSON</h3>
        <pre v-if="formattedJson" class="text-sm text-gray-800">{{ formattedJson }}</pre>
        <div v-else class="text-gray-500">暂无格式化内容</div>
      </div>
      <!-- 复制按钮 -->
      <div class="p-4 mt-3 rounded-2xl bg-white">
        <el-button type="primary" @click="copyJson">复制 JSON</el-button>
      </div>
    </div>


  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { copy } from '@/utils/string'

// 原始 Cookie 输入
const rawCookies = ref(`key1=value1; key2=value2; key3=value3`)

// 格式化后的 Cookie 转换为 JSON
const formattedJson = computed(() => {
  const cleanedCookies = rawCookies.value.replace(/\s*;\s*/g, ';') // 移除分号前后的多余空格
  const cookieArray = cleanedCookies
      .split(';') // 按分号分割
      .map((cookie) => cookie.trim()) // 去除多余空格
      .filter((cookie) => cookie.includes('=')) // 筛选包含等号的行
      .map((cookie) => {
        const [key, ...valueParts] = cookie.split('=') // 分割键值对
        return {
          key: key.trim(),
          value: valueParts.join('=').trim(), // 处理值中可能包含的等号
        }
      })

  // 转换为 JSON 格式
  const cookieObject = Object.fromEntries(
      cookieArray.map((cookie) => [cookie.key, cookie.value])
  )

  // 返回格式化的 JSON 字符串
  return cookieArray.length > 0 ? JSON.stringify(cookieObject, null, 2) : ''
})

// 复制功能
const copyJson = () => {
  if (formattedJson.value) {
    copy(formattedJson.value)
  }
}
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

/* 复制按钮样式 */
.el-button {
  margin-top: 10px;
}
</style>
