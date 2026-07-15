<template>
  <div class="editor-container">
    <div class="panel left-panel">
      <div class="panel-header">
        <h3>Markdown 编辑区</h3>
        <div class="header-actions">
          <!-- 1. 隐藏的文件选择框，绑定新的 ref -->
          <input 
            type="file" 
            ref="fileInputRef" 
            accept=".md,.markdown,.txt" 
            style="display: none;" 
            @change="handleFileImport" 
          />
          <!-- 2. 导入按钮，点击触发方法 -->
          <button class="action-btn import-btn" @click="triggerFileInput">📂 导入 MD</button>
          <button class="action-btn copy-btn" @click="copyToWechat">📋 复制富文本</button>
          <button class="action-btn export-rtf-btn" @click="exportAsRtf">📝 导出 RTF</button>
          <button class="action-btn export-html-btn" @click="exportAsHtml">🌐 导出 HTML</button>
        </div>
      </div>
      <textarea 
        v-model="markdownText" 
        placeholder="在这里输入 Markdown 内容..."
        class="editor-textarea"
      ></textarea>
    </div>

    <div class="panel right-panel">
      <h3>预览区</h3>
      <div class="preview-content" ref="previewRef">
        <section 
          style="width: auto; font-family: -apple-system-font, BlinkMacSystemFont, 'Helvetica Neue', 'PingFang SC', 'Microsoft YaHei', sans-serif; color: #333; line-height: 1.8; font-size: 16px;"
          v-html="renderedHtml"
        ></section>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import MarkdownIt from 'markdown-it'
import hljs from 'highlight.js'
import 'highlight.js/styles/github.css' 

// 明确声明类型，解决循环推断
const md: MarkdownIt = new MarkdownIt({
  highlight: function (str: string, lang: string): string {
    if (lang && hljs.getLanguage(lang)) {
      try {
        return `<pre class="hljs"><code>${hljs.highlight(str, { language: lang }).value}</code></pre>`
      } catch (__) {}
    }
    return `<pre class="hljs"><code>${md.utils.escapeHtml(str)}</code></pre>`
  }
})

const markdownText = ref(`## 🚀 破冰行动：核心链路\n\n这是你的第一个 **Markdown 转富文本** 工具！\n\n### 🎯 进阶目标\n\n> 周末搞到这个程度，成就感绝对爆棚！\n\n\`\`\`javascript\nconsole.log("Hello MD2WX!");\n\`\`\``)

// 明确声明正则参数类型，未使用的参数加下划线
const renderedHtml = computed(() => {
  let html = md.render(markdownText.value)
  
  html = html.replace(/<h2>(.*?)<\/h2>/g, (_match: string, p1: string) => {
    return `<h2 style="margin: 20px 0 10px; padding: 10px 15px; font-size: 20px; font-weight: bold; color: #333; border-left: 5px solid #07c160; background: linear-gradient(to right, #e8f8ef, #ffffff); border-radius: 4px;">${p1}</h2>`
  })

  html = html.replace(/<h3>(.*?)<\/h3>/g, (_match: string, p1: string) => {
    return `<h3 style="margin: 15px 0 10px; font-size: 18px; font-weight: bold; color: #1890ff; padding-bottom: 5px; border-bottom: 2px solid #1890ff; display: inline-block;">${p1}</h3>`
  })

  html = html.replace(/<blockquote>([\s\S]*?)<\/blockquote>/g, (_match: string, p1: string) => {
    return `<blockquote style="margin: 15px 0; padding: 15px 20px; background-color: #f8f9fa; border-left: 5px solid #1890ff; border-radius: 4px; color: #555; font-size: 15px; line-height: 1.8;">${p1}</blockquote>`
  })

  return html
})

// 明确声明 ref 类型
const previewRef = ref<HTMLElement | null>(null)
const fileInputRef = ref<HTMLInputElement | null>(null)

// 触发文件选择框
const triggerFileInput = () => {
  fileInputRef.value?.click()
}

const handleFileImport = (event: Event) => {
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]
  if (!file) return
  const reader = new FileReader()
  reader.onload = (e) => { markdownText.value = e.target?.result as string }
  reader.readAsText(file)
  target.value = '' 
}

const copyToWechat = async () => {
  if (!previewRef.value) return
  try {
    const htmlContent = previewRef.value.innerHTML
    await navigator.clipboard.write([
      new ClipboardItem({
        'text/html': new Blob([htmlContent], { type: 'text/html' }),
        'text/plain': new Blob([markdownText.value], { type: 'text/plain' })
      })
    ])
    alert('✅ 复制成功！快去微信公众号编辑器里 Ctrl+V 吧！')
  } catch (err) {
    console.error('复制失败:', err)
    alert('❌ 复制失败，请检查浏览器权限或手动复制。')
  }
}

const exportAsRtf = () => {
  if (!previewRef.value) return
  const htmlContent = previewRef.value.innerHTML
  const rtfContent = `{\\rtf1\\ansi\\deff0\n{\\fonttbl{\\f0\\fswiss\\fcharset0 Arial;}}\n{\\colortbl;\\red0\\green0\\blue0;\\red0\\green0\\blue255;}\n\\viewkind4\\uc1\\pard\\lang1033\\f0\\fs24\n${htmlContent}\n}`
  const blob = new Blob([rtfContent], { type: 'application/rtf' })
  downloadFile(blob, 'article.rtf')
}

const exportAsHtml = () => {
  if (!previewRef.value) return
  const htmlContent = previewRef.value.innerHTML
  const fullHtml = `<!DOCTYPE html>\n<html lang="zh-CN">\n<head>\n  <meta charset="UTF-8">\n  <meta name="viewport" content="width=device-width, initial-scale=1.0">\n  <title>Markdown 导出文章</title>\n</head>\n<body style="max-width: 800px; margin: 40px auto; padding: 0 20px;">\n  ${htmlContent}\n</body>\n</html>`
  const blob = new Blob([fullHtml], { type: 'text/html;charset=utf-8' })
  downloadFile(blob, 'article.html')
}

const downloadFile = (blob: Blob, fileName: string) => {
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = fileName
  document.body.appendChild(a)
  a.click()
  document.body.removeChild(a)
  URL.revokeObjectURL(url)
}
</script>

<style scoped>
.editor-container { display: flex; height: 90vh; gap: 10px; padding: 10px; }
.panel { flex: 1; display: flex; flex-direction: column; border: 1px solid #ddd; border-radius: 8px; padding: 10px; }
.panel-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
.header-actions { display: flex; gap: 8px; }
.action-btn { padding: 6px 12px; color: white; border: none; border-radius: 4px; cursor: pointer; font-size: 14px; white-space: nowrap; }
.import-btn { background-color: #1890ff; }
.import-btn:hover { background-color: #096dd9; }
.copy-btn { background-color: #07c160; }
.copy-btn:hover { background-color: #06ad56; }
.export-rtf-btn { background-color: #faad14; }
.export-rtf-btn:hover { background-color: #d48806; }
.export-html-btn { background-color: #722ed1; }
.export-html-btn:hover { background-color: #531dab; }
.editor-textarea { flex: 1; border: none; outline: none; resize: none; font-size: 16px; font-family: monospace; }
.preview-content { flex: 1; overflow-y: auto; padding: 10px; border: 1px solid #eee; border-radius: 4px; background: #fff; }
</style>