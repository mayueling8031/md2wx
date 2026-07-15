<template>
  <div class="editor-container">
    <!-- 左侧：Markdown 输入区 -->
    <div class="panel left-panel">
    <h3>Markdown 编辑区</h3>
      <div class="panel-header">
        <div class="header-actions">
          <!-- 新建文件按钮 -->
          <button class="action-btn new-btn" @click="createNewFile">📄 新建</button>
          
          <!-- 新建成功提示 -->
          <span v-if="showNewTip" class="action-tip">✅ 已新建</span>

          <!-- 隐藏的文件选择框 -->
          <input 
            type="file" 
            ref="fileInputRef" 
            accept=".md,.markdown,.txt" 
            style="display: none;" 
            @change="handleFileImport" 
          />
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

      <!-- 使用说明折叠面板 -->
      <div class="help-panel">
        <button class="help-toggle" @click="showHelp = !showHelp">
          {{ showHelp ? '📖 收起使用说明' : '📖 展开使用说明' }}
        </button>
        <div v-show="showHelp" class="help-content">
          <div class="help-grid">
            <div class="help-item">
              <span class="help-label">标题</span>
              <code>## 二级标题</code>
              <code>### 三级标题</code>
            </div>
            <div class="help-item">
              <span class="help-label">加粗 / 斜体</span>
              <code>**加粗**</code>
              <code>*斜体*</code>
            </div>
            <div class="help-item">
              <span class="help-label">引用</span>
              <code>> 引用内容</code>
            </div>
            <div class="help-item">
              <span class="help-label">代码块</span>
              <code>```语言名</code>
              <code>代码内容</code>
              <code>```
</code>
            </div>
            <div class="help-item">
              <span class="help-label">无序列表</span>
              <code>- 列表项</code>
            </div>
            <div class="help-item">
              <span class="help-label">有序列表</span>
              <code>1. 列表项</code>
            </div>
            <div class="help-item">
              <span class="help-label">链接</span>
              <code>[文字](URL)</code>
            </div>
            <div class="help-item">
              <span class="help-label">图片</span>
              <code>![描述](图片URL)</code>
            </div>
          </div>
          <p class="help-tip">💡 提示：点击「📋 复制富文本」后，直接到微信公众号编辑器中 Ctrl+V 粘贴即可保留排版样式。</p>
        </div>
      </div>
    </div>

    <!-- 右侧：富文本预览区 -->
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

// 明确声明类型，解决 TS 循环推断报错
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

// 初始默认文本（仅在首次打开时显示）
const markdownText = ref(`## 🚀 欢迎使用 MD2WX

在这里输入你的 **Markdown** 内容，右侧会实时预览排版效果。

### 🎯 核心功能

- 实时预览，所见即所得
- 一键复制富文本，完美兼容微信公众号
- 支持导入本地 .md 文件

> 祝你创作愉快！`)

// 明确声明正则参数类型，解决 TS6133 和 TS7006 报错
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
const showHelp = ref(false)

// 新增：控制新建提示的显示
const showNewTip = ref(false)

// 新建文件功能：直接清空编辑区内容
const createNewFile = () => {
  // 如果当前有内容，弹出确认框防误触
  if (markdownText.value.trim() !== '') {
    const isConfirm = confirm('当前编辑区有内容，确定要清空并新建吗？')
    if (!isConfirm) return
  }
  
  // 核心修改：直接将内容置为空字符串
  markdownText.value = ''
  
  // 显示成功提示
  showNewTip.value = true
  setTimeout(() => {
    showNewTip.value = false
  }, 2000) // 2秒后自动隐藏提示
}

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
.header-actions { display: flex; gap: 8px; flex-wrap: wrap; align-items: center; }
.action-btn { padding: 6px 12px; color: white; border: none; border-radius: 4px; cursor: pointer; font-size: 14px; white-space: nowrap; }
.new-btn { background-color: #555; }
.new-btn:hover { background-color: #333; }
.import-btn { background-color: #1890ff; }
.import-btn:hover { background-color: #096dd9; }
.copy-btn { background-color: #07c160; }
.copy-btn:hover { background-color: #06ad56; }
.export-rtf-btn { background-color: #faad14; }
.export-rtf-btn:hover { background-color: #d48806; }
.export-html-btn { background-color: #722ed1; }
.export-html-btn:hover { background-color: #531dab; }
.editor-textarea { flex: 1; border: none; outline: none; resize: none; font-size: 16px; font-family: monospace; }

/* 新建成功提示样式 */
.action-tip {
  font-size: 12px;
  color: #07c160;
  font-weight: bold;
  display: flex;
  align-items: center;
  animation: fadeInOut 2s ease-in-out;
}

@keyframes fadeInOut {
  0% { opacity: 0; }
  20% { opacity: 1; }
  80% { opacity: 1; }
  100% { opacity: 0; }
}

.help-panel { margin-top: 10px; border-top: 1px dashed #ddd; padding-top: 8px; }
.help-toggle { background: none; border: none; color: #888; font-size: 13px; cursor: pointer; padding: 4px 0; }
.help-toggle:hover { color: #333; }
.help-content { margin-top: 8px; padding: 10px; background: #fafafa; border-radius: 6px; border: 1px solid #eee; }
.help-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px 16px; }
.help-item { display: flex; flex-direction: column; gap: 2px; }
.help-label { font-size: 12px; color: #999; font-weight: bold; }
.help-item code { font-size: 12px; color: #555; background: #eee; padding: 1px 4px; border-radius: 3px; font-family: monospace; }
.help-tip { margin-top: 10px; font-size: 12px; color: #888; }

.preview-content { flex: 1; overflow-y: auto; padding: 10px; border: 1px solid #eee; border-radius: 4px; background: #fff; }
</style>