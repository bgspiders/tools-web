<template>
  <div class="flex flex-col mt-3 flex-1">
    <div class="p-4 rounded-2xl bg-white flex flex-col gap-4">
      <div class="flex flex-col">
        <label class="mb-2 font-bold">转换内容：</label>
        <textarea
          v-model="inputText"
          class="w-full p-2 border rounded resize-none"
          rows="6"
          placeholder="请输入要加密或解密的内容"
        ></textarea>
      </div>
      <div class="flex flex-wrap items-center gap-2">
        <select v-model="mode" class="p-2 border rounded w-24">
          <option value="CBC">CBC</option>
          <option value="ECB">ECB</option>
          <option value="CFB">CFB</option>
          <option value="OFB">OFB</option>
        </select>
        <select v-model="padding" class="p-2 border rounded w-28">
          <option value="PKCS7">PKCS7</option>
          <option value="ISO10126">ISO10126</option>
          <option value="ANSIX923">ANSIX923</option>
          <option value="ZeroPadding">ZeroPadding</option>
        </select>
        <input
          v-model="key"
          type="text"
          class="p-2 border rounded w-40"
          placeholder="请输入密钥(16位推荐)"
        />
        <input
          v-model="iv"
          type="text"
          class="p-2 border rounded w-40"
          placeholder="IV(16位，可选)"
          maxlength="16"
        />
        <el-button type="primary" @click="encrypt">加密</el-button>
        <el-button type="primary" @click="decrypt">解密</el-button>
        <el-button type="info" @click="showCode = !showCode">查看代码</el-button>
      </div>
      <div class="flex flex-col">
        <label class="mb-2 font-bold">转换结果：</label>
        <textarea
          class="w-full p-2 border rounded resize-none"
          rows="6"
          readonly
          :value="result"
        ></textarea>
      </div>
      <div v-if="showCode" class="mt-4">
        <div class="mb-2 flex gap-2">
          <el-button size="small" :type="codeTab==='python'?'primary':'default'" @click="codeTab='python'">Python</el-button>
          <el-button size="small" :type="codeTab==='js'?'primary':'default'" @click="codeTab='js'">JavaScript</el-button>
        </div>
        <pre class="bg-gray-100 p-3 rounded text-xs overflow-x-auto"><code>{{ codeTab==='python' ? pythonCode : jsCode }}</code></pre>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import CryptoJS from 'crypto-js'

const inputText = ref('')
const key = ref('')
const mode = ref('CBC')
const padding = ref('PKCS7')
const iv = ref('')
const result = ref('')
const showCode = ref(false)
const codeTab = ref<'python'|'js'>('python')

const encrypt = () => {
  if (!inputText.value || !key.value) {
    alert('请输入内容和密钥')
    return
  }
  try {
    const keyBytes = CryptoJS.enc.Utf8.parse(key.value.padEnd(16, '0').substring(0, 16))
    const ivBytes = iv.value ? CryptoJS.enc.Utf8.parse(iv.value.padEnd(16, '0').substring(0, 16)) : undefined
    let encrypted
    switch (mode.value) {
      case 'CBC':
        encrypted = CryptoJS.AES.encrypt(inputText.value, keyBytes, {
          iv: ivBytes,
          mode: CryptoJS.mode.CBC,
          padding: getPadding(padding.value)
        })
        break
      case 'ECB':
        encrypted = CryptoJS.AES.encrypt(inputText.value, keyBytes, {
          mode: CryptoJS.mode.ECB,
          padding: getPadding(padding.value)
        })
        break
      case 'CFB':
        encrypted = CryptoJS.AES.encrypt(inputText.value, keyBytes, {
          iv: ivBytes,
          mode: CryptoJS.mode.CFB,
          padding: getPadding(padding.value)
        })
        break
      case 'OFB':
        encrypted = CryptoJS.AES.encrypt(inputText.value, keyBytes, {
          iv: ivBytes,
          mode: CryptoJS.mode.OFB,
          padding: getPadding(padding.value)
        })
        break
      default:
        throw new Error('不支持的加密模式')
    }
    result.value = encrypted.toString()
  } catch (error) {
    result.value = `错误: ${(error as Error).message}`
  }
}

const decrypt = () => {
  if (!inputText.value || !key.value) {
    alert('请输入内容和密钥')
    return
  }
  try {
    const keyBytes = CryptoJS.enc.Utf8.parse(key.value.padEnd(16, '0').substring(0, 16))
    const ivBytes = iv.value ? CryptoJS.enc.Utf8.parse(iv.value.padEnd(16, '0').substring(0, 16)) : undefined
    let decrypted
    switch (mode.value) {
      case 'CBC':
        decrypted = CryptoJS.AES.decrypt(inputText.value, keyBytes, {
          iv: ivBytes,
          mode: CryptoJS.mode.CBC,
          padding: getPadding(padding.value)
        })
        break
      case 'ECB':
        decrypted = CryptoJS.AES.decrypt(inputText.value, keyBytes, {
          mode: CryptoJS.mode.ECB,
          padding: getPadding(padding.value)
        })
        break
      case 'CFB':
        decrypted = CryptoJS.AES.decrypt(inputText.value, keyBytes, {
          iv: ivBytes,
          mode: CryptoJS.mode.CFB,
          padding: getPadding(padding.value)
        })
        break
      case 'OFB':
        decrypted = CryptoJS.AES.decrypt(inputText.value, keyBytes, {
          iv: ivBytes,
          mode: CryptoJS.mode.OFB,
          padding: getPadding(padding.value)
        })
        break
      default:
        throw new Error('不支持的加密模式')
    }
    result.value = decrypted.toString(CryptoJS.enc.Utf8)
  } catch (error) {
    result.value = `错误: ${(error as Error).message}`
  }
}

const getPadding = (padding: string) => {
  switch (padding) {
    case 'PKCS7':
      return CryptoJS.pad.Pkcs7
    case 'ISO10126':
      return CryptoJS.pad.Iso10126
    case 'ANSIX923':
      return CryptoJS.pad.AnsiX923
    case 'ZeroPadding':
      return CryptoJS.pad.ZeroPadding
    default:
      return CryptoJS.pad.Pkcs7
  }
}

const jsCode = computed(() => {
  return `const CryptoJS = require('crypto-js');\n\nconst key = '${key.value}';\nconst iv = '${iv.value}';\nconst text = '${inputText.value}';\n\n// 加密\nconst encrypted = CryptoJS.AES.encrypt(text, CryptoJS.enc.Utf8.parse(key.padEnd(16, '0').substring(0, 16)), {\n  ${mode.value !== 'ECB' ? "iv: CryptoJS.enc.Utf8.parse(iv.padEnd(16, '0').substring(0, 16))," : ''}\n  mode: CryptoJS.mode.${mode.value},\n  padding: CryptoJS.pad.${padding.value}\n});\nconsole.log('加密结果:', encrypted.toString());\n\n// 解密\nconst decrypted = CryptoJS.AES.decrypt(encrypted.toString(), CryptoJS.enc.Utf8.parse(key.padEnd(16, '0').substring(0, 16)), {\n  ${mode.value !== 'ECB' ? "iv: CryptoJS.enc.Utf8.parse(iv.padEnd(16, '0').substring(0, 16))," : ''}\n  mode: CryptoJS.mode.${mode.value},\n  padding: CryptoJS.pad.${padding.value}\n});\nconsole.log('解密结果:', decrypted.toString(CryptoJS.enc.Utf8));`
})

const pythonCode = computed(() => {
  return `from Crypto.Cipher import AES\nfrom Crypto.Util.Padding import pad, unpad\nimport base64\n\nkey = b'${key.value.padEnd(16, '0').substring(0, 16)}'\n${mode.value !== 'ECB' ? `iv = b'${iv.value.padEnd(16, '0').substring(0, 16)}'` : ''}\ntext = b'${inputText.value}'\n\n# 加密\ncipher = AES.new(key, AES.MODE_${mode.value}${mode.value !== 'ECB' ? ', iv=iv' : ''})\nct_bytes = cipher.encrypt(pad(text, 16))\nenc = base64.b64encode(ct_bytes).decode()\nprint('加密结果:', enc)\n\n# 解密\ncipher2 = AES.new(key, AES.MODE_${mode.value}${mode.value !== 'ECB' ? ', iv=iv' : ''})\npt = unpad(cipher2.decrypt(base64.b64decode(enc)), 16)\nprint('解密结果:', pt.decode())`
})
</script>

<style scoped>
textarea {
  font-family: monospace;
  font-size: 14px;
}
pre {
  font-family: monospace;
  font-size: 13px;
  white-space: pre-wrap;
}
</style> 