# 音乐智能分析平台 - 前端 
(Music Intelligence Analysis Platform - Frontend)

这是一个现代化的、响应式的 Web 应用前端。它提供了一个优雅的用户界面，允许用户上传音频文件，并通过调用后端 AI 模型，获得即时的、智能化的音乐评分和评论。

**[➡️ 点击此处访问线上 Demo](https://qua-drant.github.io/music-analyzer-app/)**

---

### ✨ 主要功能 (Features)

- **音频上传**: 支持用户从本地选择并上传音频文件。
- **智能分析展示**: 以美观的卡片形式，动态展示后端返回的乐曲名、星级评分和 AI 生成的评论。
- **客户端处理**: 在文件上传前，利用 `jsmediatags` 在浏览器端直接提取专辑封面，提升用户体验。
- **主题切换**: 支持一键切换浅色/深色模式，并能记住用户的选择。
- **动态视觉效果**: 包含优雅的背景辉光、卡片悬浮光晕及流畅的进入动画，增强了应用的现代感。
- **响应式设计**: 界面能够完美适配桌面、平板和手机等不同尺寸的设备。

### 🛠️ 技术栈 (Technology Stack)

- **核心框架**: [Vue.js 3](https://vuejs.org/)
- **构建工具**: [Vite](https://vitejs.dev/)
- **HTTP 请求**: [Axios](https://axios-http.com/)
- **音频元数据**: [jsmediatags](https://github.com/aadsm/jsmediatags)
- **部署**: 通过 **GitHub Actions** 自动化部署到 **GitHub Pages**

### 🚀 本地开发 (Local Development)

如需在本地运行此项目，请按以下步骤操作：

1.  **前提条件**:
    确保您的电脑已安装 [Node.js](https://nodejs.org/) (v18+)。

2.  **克隆仓库**:
    ```bash
    git clone [https://github.com/Qua-Drant/music-analyzer-app.git](https://github.com/Qua-Drant/music-analyzer-app.git)
    ```

3.  **进入项目目录**:
    ```bash
    cd music-analyzer-app 
    ```
    *(如果您将前后端放在一个仓库，请确保进入 `frontend` 目录)*

4.  **安装依赖**:
    ```bash
    npm install
    ```

5.  **启动开发服务器**:
    ```bash
    npm run dev
    ```
    项目将在 `http://localhost:5173` (或另一个可用端口) 上运行。

### 🔗 连接后端

此前端项目需要一个正在运行的后端服务来提供分析功能。

-   请在 `src/App.vue` 文件中，找到 `axios.post` 这一行。
-   将其中的 URL 地址修改为您后端服务的实际地址。

```javascript
// src/App.vue
const response = await axios.post('http://您的后端服务地址/analyze', formData, ...);
```

---
