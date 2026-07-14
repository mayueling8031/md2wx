# 🚀 MD2WX - Markdown 转微信公众号排版神器

一个基于 **Vue 3 + Vite** 的纯前端 Markdown 排版工具。专为微信公众号创作者设计，支持实时预览、一键复制富文本、精美样式注入，让排版不再痛苦！

## ✨ 核心特性

- **⚡️ 实时预览**：左侧 Markdown，右侧富文本，所见即所得。
- **📋 一键复制**：完美保留标题、代码高亮、引用卡片等样式，直接粘贴到微信编辑器。
- **🎨 微信排版**：内置 H2/H3 标题样式、精美引用块，开箱即用。
- **📂 文件导入**：支持一键导入本地 `.md` / `.txt` 文件。
- **💾 多格式导出**：支持导出为 `HTML` 网页或 `RTF` 富文本文件。
- **📖 语法速查**：内置折叠式 Markdown 语法说明，新手友好。
- **🔒 隐私安全**：纯前端实现，无后端，数据不上传，本地处理。

## 🛠️ 技术栈

- **框架**：Vue 3 (Composition API + `<script setup>`)
- **构建**：Vite
- **语言**：TypeScript
- **核心库**：
  - `markdown-it`：Markdown 解析
  - `highlight.js`：代码语法高亮

## 🏃‍♂️ 快速开始

```bash
# 1. 克隆项目
git clone <your-repo-url>
cd md2wx

# 2. 安装依赖
npm install

# 3. 启动开发服务器
npm run dev