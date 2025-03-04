<template>
  <div class="flex flex-col mt-3 flex-1">
    <!-- 左右分栏布局 -->
    <div class="p-4 rounded-2xl bg-white  flex gap-4">
      <!-- 左侧：HTML 输入框 -->
      <div class="flex-1 border  mt-3 flex flex-col min-w-[50%] min-h-[100%]">
        <textarea
            v-model="htmlInput"
            class="flex-1 p-2 border rounded resize-none overflow-auto"
            placeholder="在这里输入 HTML 代码..."
        ></textarea>
      </div>
      <!-- 右侧：HTML 渲染预览 -->
      <div class="flex-1 p-4 border overflow-y-auto max-w-[50%] min-h-[100%]">
        <div v-html="htmlInput"></div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
// HTML 输入内容
const htmlInput = ref('<p>在这里输入 HTML 代码...</p>')
// 复制功能
</script>

<style scoped>
/* 添加样式 */
textarea {
  height: 100%; /* 确保 textarea 填充父容器高度 */
  font-family: monospace; /* 使用等宽字体 */
  font-size: 14px; /* 设置字体大小 */
}
</style>
