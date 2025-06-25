<template>
  <div class="flex flex-col mt-3 flex-1">
    <div class="p-4 rounded-2xl bg-white flex flex-col gap-4">
      <div class="flex flex-col">
        <label class="mb-2 font-bold">输入curl命令：</label>
        <textarea
          v-model="curl"
          class="w-full p-2 border rounded resize-none"
          rows="6"
          placeholder="请输入curl命令，如：curl 'https://api.example.com' -H 'Authorization: Bearer xxx' -d 'key=value'"
        ></textarea>
      </div>
      <div class="flex flex-row items-center gap-2">
        <el-button type="primary" @click="convert">转换</el-button>
      </div>
      <div class="flex flex-col">
        <label class="mb-2 font-bold">转换结果（Python requests）：</label>
        <textarea
          class="w-full p-2 border rounded resize-none"
          rows="8"
          readonly
          :value="result"
        ></textarea>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const curl = ref('')
const result = ref('')

const convert = () => {
  if (!curl.value.trim()) {
    result.value = '请输入curl命令！'
    return
  }
  try {
    result.value = curlToRequests(curl.value)
  } catch (e) {
    result.value = '转换失败：' + e.message
  }
}

// 简单实现，支持常见curl参数，复杂情况建议用后端库
function curlToRequests(curl: string): string {
  // 只做简单的正则和字符串处理，适合常见用法
  let url = ''
  let method = 'get'
  let headers: Record<string, string> = {}
  let data = ''

  // 匹配url
  const urlMatch = curl.match(/curl ['"]([^'"]+)['"]/)
  if (urlMatch) url = urlMatch[1]

  // 匹配-H头部
  const headerRegex = /-H ['"]([^'"]+)['"]/g
  let m
  while ((m = headerRegex.exec(curl)) !== null) {
    const [k, v] = m[1].split(/: (.+)/)
    headers[k] = v
  }

  // 匹配-d数据
  const dataMatch = curl.match(/-d ['"]([^'"]+)['"]/)
  if (dataMatch) {
    data = dataMatch[1]
    method = 'post'
  }

  // 匹配-X方法
  const methodMatch = curl.match(/-X (\w+)/)
  if (methodMatch) method = methodMatch[1].toLowerCase()

  // 生成requests代码
  let code = 'import requests\n\n'
  code += `url = '${url}'\n`
  if (Object.keys(headers).length > 0) {
    code += 'headers = {\n'
    for (const k in headers) {
      code += `    '${k}': '${headers[k]}',\n`
    }
    code += '}\n'
  }
  if (data) code += `data = '${data}'\n`
  code += '\n'
  code += 'response = requests.' + method + '('
  code += 'url'
  if (Object.keys(headers).length > 0) code += ', headers=headers'
  if (data) code += ', data=data'
  code += ')\n'
  code += 'print(response.text)'
  return code
}
</script>

<style scoped>
textarea {
  font-family: monospace;
  font-size: 14px;
}
</style> 