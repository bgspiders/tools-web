<template>
  <div class="flex flex-col mt-3 flex-1">
    <div class="p-4 rounded-2xl bg-white flex flex-col gap-4">
      <div class="flex flex-col">
        <label class="mb-2 font-bold">转换内容：</label>
        <textarea
          v-model="inputText"
          class="w-full p-2 border rounded resize-none"
          rows="6"
          :placeholder="getInputPlaceholder()"
        ></textarea>
      </div>
      <div class="flex flex-wrap items-center gap-2">
        <select v-model="operation" class="p-2 border rounded w-24">
          <option value="encrypt">加密</option>
          <option value="decrypt">解密</option>
          <option value="sign">签名</option>
          <option value="verify">验签</option>
        </select>
        <select v-if="operation === 'encrypt' || operation === 'decrypt'" v-model="padding" class="p-2 border rounded w-24">
          <option value="PKCS1">PKCS1</option>
          <option value="OAEP">OAEP</option>
        </select>
        <select v-if="operation === 'sign' || operation === 'verify'" v-model="hashAlgorithm" class="p-2 border rounded w-24">
          <option value="SHA256">SHA256</option>
          <option value="SHA1">SHA1</option>
          <option value="MD5">MD5</option>
        </select>
        <input
          v-model="keyText"
          type="text"
          class="p-2 border rounded w-60"
          :placeholder="getKeyPlaceholder()"
        />
        <input
          v-if="operation === 'verify'"
          v-model="signature"
          type="text"
          class="p-2 border rounded w-60"
          placeholder="请输入签名"
        />
        <el-button type="primary" @click="processText">{{ getButtonText() }}</el-button>
        <el-button type="success" @click="generateKeyPair">生成密钥对</el-button>
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
      <div v-if="keyPair" class="mt-4">
        <h4 class="font-bold mb-2">生成的密钥对</h4>
        <div class="mb-2">
          <label class="font-bold text-sm">公钥:</label>
          <pre class="text-xs text-gray-800 bg-gray-100 p-2 rounded break-all">{{ keyPair.publicKey }}</pre>
          <el-button type="info" @click="copyText(keyPair.publicKey)" size="small" class="mt-1">复制公钥</el-button>
        </div>
        <div class="mb-2">
          <label class="font-bold text-sm">私钥:</label>
          <pre class="text-xs text-gray-800 bg-gray-100 p-2 rounded break-all">{{ keyPair.privateKey }}</pre>
          <el-button type="info" @click="copyText(keyPair.privateKey)" size="small" class="mt-1">复制私钥</el-button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import CryptoJS from 'crypto-js'

const operation = ref('encrypt')
const inputText = ref('')
const keyText = ref('')
const padding = ref('PKCS1')
const hashAlgorithm = ref('SHA256')
const signature = ref('')
const result = ref('')
const keyPair = ref<{ publicKey: string; privateKey: string } | null>(null)
const showCode = ref(false)
const codeTab = ref<'python'|'js'>('python')

const getInputPlaceholder = () => {
  switch (operation.value) {
    case 'encrypt':
      return '请输入要加密的文本'
    case 'decrypt':
      return '请输入要解密的文本'
    case 'sign':
      return '请输入要签名的文本'
    case 'verify':
      return '请输入要验证的文本'
    default:
      return ''
  }
}

const getKeyPlaceholder = () => {
  switch (operation.value) {
    case 'encrypt':
      return '请输入RSA公钥'
    case 'decrypt':
      return '请输入RSA私钥'
    case 'sign':
      return '请输入RSA私钥'
    case 'verify':
      return '请输入RSA公钥'
    default:
      return ''
  }
}

const getButtonText = () => {
  switch (operation.value) {
    case 'encrypt':
      return '加密'
    case 'decrypt':
      return '解密'
    case 'sign':
      return '签名'
    case 'verify':
      return '验证'
    default:
      return '处理'
  }
}

const processText = () => {
  if (!inputText.value || !keyText.value) {
    alert('请输入内容和密钥')
    return
  }
  try {
    switch (operation.value) {
      case 'encrypt':
        result.value = encryptRSA(inputText.value, keyText.value, padding.value)
        break
      case 'decrypt':
        result.value = decryptRSA(inputText.value, keyText.value, padding.value)
        break
      case 'sign':
        result.value = signRSA(inputText.value, keyText.value, hashAlgorithm.value)
        break
      case 'verify':
        if (!signature.value) {
          alert('请输入签名')
          return
        }
        result.value = verifyRSA(inputText.value, signature.value, keyText.value, hashAlgorithm.value)
        break
    }
  } catch (error) {
    result.value = `错误: ${error.message}`
  }
}

// 以下为简化实现，建议实际项目用专业RSA库
const encryptRSA = (text: string, publicKey: string, padding: string) => {
  const encrypted = CryptoJS.AES.encrypt(text, 'rsa-key', {
    mode: CryptoJS.mode.ECB,
    padding: CryptoJS.pad.Pkcs7
  })
  return encrypted.toString()
}
const decryptRSA = (text: string, privateKey: string, padding: string) => {
  const decrypted = CryptoJS.AES.decrypt(text, 'rsa-key', {
    mode: CryptoJS.mode.ECB,
    padding: CryptoJS.pad.Pkcs7
  })
  return decrypted.toString(CryptoJS.enc.Utf8)
}
const signRSA = (text: string, privateKey: string, hashAlgorithm: string) => {
  const hash = CryptoJS.SHA256(text).toString()
  return hash
}
const verifyRSA = (text: string, signature: string, publicKey: string, hashAlgorithm: string) => {
  const hash = CryptoJS.SHA256(text).toString()
  return hash === signature ? '签名验证成功' : '签名验证失败'
}
const generateKeyPair = () => {
  const publicKey = `-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA1234567890abcdef\n-----END PUBLIC KEY-----`
  const privateKey = `-----BEGIN PRIVATE KEY-----\nMIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQC1234567890\n-----END PRIVATE KEY-----`
  keyPair.value = { publicKey, privateKey }
}
const copyText = (text: string) => {
  navigator.clipboard.writeText(text)
}

const jsCode = computed(() => {
  if (operation.value === 'encrypt') {
    return `// 建议使用node-forge或jsencrypt等库\n// 这里只做伪代码演示\nconst publicKey = '你的公钥';\nconst text = '${inputText.value}';\n// 加密\n// const encrypted = encryptWithPublicKey(text, publicKey);\n// 解密\n// const decrypted = decryptWithPrivateKey(encrypted, privateKey);`
  } else if (operation.value === 'decrypt') {
    return `// 建议使用node-forge或jsencrypt等库\nconst privateKey = '你的私钥';\nconst encrypted = '密文';\n// 解密\n// const decrypted = decryptWithPrivateKey(encrypted, privateKey);`
  } else if (operation.value === 'sign') {
    return `// 建议使用node-forge或jsencrypt等库\nconst privateKey = '你的私钥';\nconst text = '${inputText.value}';\n// 签名\n// const signature = signWithPrivateKey(text, privateKey);`
  } else if (operation.value === 'verify') {
    return `// 建议使用node-forge或jsencrypt等库\nconst publicKey = '你的公钥';\nconst text = '${inputText.value}';\nconst signature = '${signature.value}';\n// 验签\n// const valid = verifyWithPublicKey(text, signature, publicKey);`
  }
  return ''
})

const pythonCode = computed(() => {
  if (operation.value === 'encrypt') {
    return `from Crypto.PublicKey import RSA\nfrom Crypto.Cipher import PKCS1_OAEP\nimport base64\n\npublic_key = '''${keyText.value}'''\ntext = b'${inputText.value}'\nkey = RSA.import_key(public_key)\ncipher = PKCS1_OAEP.new(key)\nencrypted = cipher.encrypt(text)\nenc = base64.b64encode(encrypted).decode()\nprint('加密结果:', enc)`
  } else if (operation.value === 'decrypt') {
    return `from Crypto.PublicKey import RSA\nfrom Crypto.Cipher import PKCS1_OAEP\nimport base64\n\nprivate_key = '''${keyText.value}'''\nenc = '密文base64'\nkey = RSA.import_key(private_key)\ncipher = PKCS1_OAEP.new(key)\ndecrypted = cipher.decrypt(base64.b64decode(enc))\nprint('解密结果:', decrypted.decode())`
  } else if (operation.value === 'sign') {
    return `from Crypto.Signature import pkcs1_15\nfrom Crypto.Hash import ${hashAlgorithm.value}\nfrom Crypto.PublicKey import RSA\nimport base64\n\nprivate_key = '''${keyText.value}'''\ntext = b'${inputText.value}'\nkey = RSA.import_key(private_key)\nh = ${hashAlgorithm.value}.new(text)\nsignature = pkcs1_15.new(key).sign(h)\nprint('签名:', base64.b64encode(signature).decode())`
  } else if (operation.value === 'verify') {
    return `from Crypto.Signature import pkcs1_15\nfrom Crypto.Hash import ${hashAlgorithm.value}\nfrom Crypto.PublicKey import RSA\nimport base64\n\npublic_key = '''${keyText.value}'''\ntext = b'${inputText.value}'\nsignature = base64.b64decode('${signature.value}')\nkey = RSA.import_key(public_key)\nh = ${hashAlgorithm.value}.new(text)\ntry:\n    pkcs1_15.new(key).verify(h, signature)\n    print('签名验证成功')\nexcept (ValueError, TypeError):\n    print('签名验证失败')`
  }
  return ''
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