<template>
  <div id="app-container" :class="theme === 'light' ? 'light-theme' : ''">
    <header class="spotify-header">
      <div class="header-content">
        <h1>音乐智能分析 ✨</h1>
        <p>上传你的音乐，即刻获得AI生成的评分与评论</p>
      </div>
      <button @click="toggleTheme" class="theme-switcher" aria-label="切换主题">
        <svg v-if="theme === 'dark'" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path></svg>
        <svg v-if="theme === 'light'" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line></svg>
      </button>
    </header>

    <main class="content-area">
      <div class="upload-section">
        <input type="file" @change="handleFileSelect" ref="fileInput" style="display: none;" accept="audio/*">
        <button @click="triggerFileInput" class="upload-button" :disabled="isLoading">
          <span v-if="!isLoading">选择音乐文件</span>
          <span v-else>分析中...</span>
        </button>
        <p v-if="selectedFileName" class="file-name">已选择: {{ selectedFileName }}</p>
      </div>

      <div v-if="isLoading" class="loading-spinner"></div>

      <div v-if="error" class="error-message">
        <p>出错了: {{ error }}</p>
      </div>

      <div v-if="result" class="result-card">
        <div class="card-content">
          <div class="info-section">
            <h2 class="track-title">🎧 {{ result.track_name }}</h2>
            <div class="rating">
              <span class="stars">{{ '★'.repeat(Math.round(result.rating)) }}{{ '☆'.repeat(5 - Math.round(result.rating)) }}</span>
              <span class="score">{{ result.rating.toFixed(1) }} / 5.0</span>
            </div>

            <audio
              v-if="musicObjectURL"
              :src="musicObjectURL"
              @play="isAudioPlaying = true"
              @pause="isAudioPlaying = false"
              @ended="isAudioPlaying = false"
              controls
              class="audio-player">
            </audio>

            <p class="comment">"{{ result.comment }}"</p>
          </div>

          <div class="cover-section">
            <div class="vinyl-wrapper">
              <img
                :src="albumArtUrl"
                alt="Album Cover"
                class="vinyl-cover"
                :class="{ 'playing': isAudioPlaying }"
              >
            </div>
          </div>
        </div>
      </div>
    </main>

    <footer class="app-footer">
      <p>Crafted with ❤️ by the Gemini & Vue.js Team &copy; 2025</p>
      <p><strong>制作团队：王怡潇、冯昱嘉、蔡意娴、刘奕志、胡育诚</strong></p>
      <p>音乐分析技术由基于 腾讯MuQ-Mulan 的自训练模型 提供支持</p>
    </footer>

  </div>
</template>

<script>
import axios from 'axios';

// 【修复】直接导入库的 UMD 版本来解决 Vite 的解析问题
import 'jsmediatags/dist/jsmediatags.min.js';
const jsmediatags = window.jsmediatags;

export default {
  data() {
    return {
      selectedFile: null,
      selectedFileName: '',
      result: null,
      isLoading: false,
      error: null,
      musicObjectURL: null,
      albumArtUrl: '/vinyl-placeholder.jpg',
      isAudioPlaying: false,
      // 【新功能】添加 theme 状态，默认为 'dark'
      theme: 'dark',
      isAudioPlaying: false,
    };
  },
  methods: {
    triggerFileInput() {
      this.$refs.fileInput.value = '';
      this.$refs.fileInput.click();
    },

    async handleFileSelect(event) {
      this.selectedFile = event.target.files[0];
      if (!this.selectedFile) return;

      this.result = null;
      this.error = null;
      this.isAudioPlaying = false;
      this.isLoading = true;
      this.selectedFileName = this.selectedFile.name;

      if (this.musicObjectURL) URL.revokeObjectURL(this.musicObjectURL);
      if (this.albumArtUrl.startsWith('blob:')) URL.revokeObjectURL(this.albumArtUrl);

      this.musicObjectURL = URL.createObjectURL(this.selectedFile);

      // 【重要】确保在使用 jsmediatags 之前，它已经被成功加载
      if (jsmediatags) {
        new jsmediatags.Reader(this.selectedFile)
          .setTagsToRead(['picture'])
          .read({
            onSuccess: (tag) => {
              const {picture} = tag.tags;
              if (picture) {
                const base64String = btoa(String.fromCharCode.apply(null, picture.data));
                this.albumArtUrl = `data:${picture.format};base64,${base64String}`;
              } else {
                this.albumArtUrl = '/vinyl-placeholder.jpg';
              }
            },
            onError: (error) => {
              console.error('无法读取音频元数据:', error);
              this.albumArtUrl = '/vinyl-placeholder.jpg';
            }
          });
      } else {
        console.error("jsmediatags 未能加载");
        this.albumArtUrl = '/vinyl-placeholder.jpg';
      }

      const formData = new FormData();
      formData.append('musicFile', this.selectedFile);

      try {
        const response = await axios.post('http://localhost:5000/analyze', formData);
        this.result = response.data;
      } catch (err) {
        this.error = err.response ? (err.response.data.error || '分析失败') : '无法连接到服务器';
        console.error(err);
      } finally {
        this.isLoading = false;
      }
    },

  toggleTheme() {
    // 在 'dark' 和 'light' 之间切换
    this.theme = this.theme === 'dark' ? 'light' : 'dark';
    // 将用户的选择保存到 localStorage
    localStorage.setItem('theme', this.theme);
  },
},

  mounted() {
    // 组件加载时，尝试从 localStorage 读取已保存的主题
    const savedTheme = localStorage.getItem('theme');
    if (savedTheme) {
      this.theme = savedTheme;
    }
  },

  beforeUnmount() {
    if (this.musicObjectURL) URL.revokeObjectURL(this.musicObjectURL);
    if (this.albumArtUrl.startsWith('blob:')) URL.revokeObjectURL(this.albumArtUrl);
  },
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');

/* =================================================================
   CSS 变量定义
 ================================================================= */
:root {
  /* 深色模式 */
  --color-background: #121212;
  --color-surface: #181818;
  --color-card: #282828;
  --color-text-primary: #FFFFFF;
  --color-text-secondary: #b3b3b3;
  --color-accent: #1DB954;
  --color-accent-hover: #1ED760;
  --color-star: #f5b32a;
  --color-error-bg: rgba(255, 77, 77, 0.1);
  --color-error-text: #ff4d4d;
  --color-shadow: rgba(0, 0, 0, 0.5);
  --color-button-disabled: #535353;
  --color-player-bg: #333;
}

#app-container.light-theme {
  /* 浅色模式 */
  --color-background: #f4f4f4;
  --color-surface: #ffffff;
  --color-card: #ffffff;
  --color-text-primary: #000000;
  --color-text-secondary: #5f5f5f;
  --color-accent: #1DB954;
  --color-accent-hover: #1aa34a;
  --color-star: #f5b32a;
  --color-error-bg: rgba(220, 53, 69, 0.1);
  --color-error-text: #dc3545;
  --color-shadow: rgba(0, 0, 0, 0.1);
  --color-button-disabled: #bDBDBD;
  --color-player-bg: #E0E0E0;
}

/* =================================================================
   全局和布局样式
 ================================================================= */

body {
  color: var(--color-text-primary);
  font-family: 'Roboto', sans-serif;
  margin: 0;
  padding: 0;
  transition: background-color 0.3s ease, color 0.3s ease;

  /* 【新效果】动态渐变背景 */
  background: linear-gradient(-45deg, #ee7752, #e73c7e, #23a6d5, #23d5ab);
  background-size: 400% 400%;
  animation: animate-gradient 15s ease infinite;
}

/* 【新增】定义渐变背景的移动动画 */
@keyframes animate-gradient {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}


#app-container {
  width: 100%;
  min-height: 100vh;
  padding: 5vh 0;
  box-sizing: border-box;
  /* 【修改】背景色现在有一定透明度，能透出底下的动态渐变 */
  background-color: rgba(18, 18, 18, 0.6);
  /* 【新增】毛玻璃效果，让背景渐变显得更柔和、有质感 */
  backdrop-filter: blur(10px);
  transition: background-color 0.3s ease;
}

/* 【新增】浅色模式下的容器背景 */
#app-container.light-theme {
  background-color: rgba(255, 255, 255, 0.6);
}

.spotify-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 90%;
  margin-left: auto;
  margin-right: auto;
  padding: 0 2rem;
  box-sizing: border-box;
}

.content-area {
  width: 90%;
  margin-left: auto;
  margin-right: auto;
  margin-top: 4rem;
  padding: 0 2rem;
  box-sizing: border-box;
}

.spotify-header h1 {
  font-size: 2.5em;
  font-weight: 700;
  margin-bottom: 8px;
  letter-spacing: 0.03em;
  color: var(--color-text-primary);
}

.spotify-header p {
  line-height: 1.7;
  letter-spacing: 0.01em;
  color: var(--color-text-secondary);
}

/* =================================================================
   组件样式 (所有其他样式保持不变)
 ================================================================= */
.theme-switcher { background: none; border: none; color: var(--color-text-secondary); cursor: pointer; padding: 8px; border-radius: 50%; display: flex; align-items: center; justify-content: center; transition: color 0.3s ease, background-color 0.3s ease; }
.theme-switcher:hover { color: var(--color-text-primary); background-color: var(--color-card); }
.upload-button { background-color: var(--color-accent); color: #ffffff; border: none; padding: 15px 30px; border-radius: 500px; font-size: 1em; font-weight: 700; cursor: pointer; transition: background-color 0.2s, transform 0.2s; }
.upload-button:hover { background-color: var(--color-accent-hover); transform: scale(1.05); }
.upload-button:disabled { background-color: var(--color-button-disabled); cursor: not-allowed; }
.file-name { color: var(--color-text-secondary); margin-top: 15px; font-size: 0.9em; }
.loading-spinner { border: 4px solid rgba(128, 128, 128, 0.2); border-left-color: var(--color-accent); border-radius: 50%; width: 40px; height: 40px; animation: spin 1s linear infinite; margin: 40px auto; }
@keyframes spin { to { transform: rotate(360deg); } }
.error-message { margin-top: 20px; color: var(--color-error-text); background-color: var(--color-error-bg); padding: 15px; border-radius: 8px; }
.result-card { margin-top: 40px; background-color: var(--color-card); padding: 24px; border-radius: 8px; box-shadow: 0 4px 20px var(--color-shadow); transition: transform 0.3s ease, box-shadow 0.3s ease, background-color 0.3s ease; animation: slide-fade-in 0.6s ease-out; }
.result-card:hover { transform: translateY(-8px); box-shadow: 0 10px 30px var(--color-shadow), 0 0 25px var(--color-accent-hover); }
.card-content { display: flex; gap: 30px; align-items: center; }
.info-section { flex: 1; min-width: 0; }
.cover-section { flex-shrink: 0; }
@keyframes slide-fade-in { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }
.track-title { font-size: 1.8em; margin: 0 0 10px 0; color: var(--color-text-primary); }
.rating { display: flex; align-items: center; gap: 15px; margin-bottom: 20px; }
.stars { font-size: 1.5em; color: var(--color-star); }
.score { font-size: 1.2em; color: var(--color-text-secondary); }
.comment { font-size: 1.1em; line-height: 1.6; font-style: italic; color: var(--color-text-secondary); }
.audio-player { width: 100%; margin: 20px 0; }
audio::-webkit-media-controls-panel { background-color: var(--color-player-bg); }
audio::-webkit-media-controls-play-button { background-color: var(--color-accent); border-radius: 50%; }
audio::-webkit-media-controls-current-time-display, audio::-webkit-media-controls-time-remaining-display { color: var(--color-text-primary); }
.vinyl-wrapper { width: 200px; height: 200px; position: relative; }
.vinyl-cover { width: 100%; height: 100%; border-radius: 50%; object-fit: cover; box-shadow: 0 0 20px var(--color-shadow); animation-name: rotate-vinyl; animation-duration: 3s; animation-iteration-count: infinite; animation-timing-function: linear; animation-play-state: paused; }
.vinyl-cover::after { content: ''; position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 20px; height: 20px; background-color: var(--color-surface); border: 2px solid var(--color-player-bg); }
.vinyl-cover.playing { animation-play-state: running; }
@keyframes rotate-vinyl { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }

/* =================================================================
   【新增】页脚样式
 ================================================================= */

.app-footer {
  width: 100%;
  text-align: center;
  margin-top: 6rem; /* 与上方内容拉开足够的距离 */
  padding: 2rem 0;
  color: var(--color-text-secondary);
  font-size: 0.85em; /* 使用稍小的字体 */
  opacity: 0.6; /* 让页脚信息更柔和，不那么显眼 */
}

.app-footer p {
  margin: 0.5rem 0; /* 为多行文本提供一些间距 */
  line-height: 1.5;
}

/* =================================================================
   响应式样式
 ================================================================= */
@media (max-width: 768px) {
  #app-container { padding: 2rem 0; }
  .spotify-header, .content-area { width: 100%; padding: 0 1rem; }
  .spotify-header { flex-direction: column; align-items: flex-start; gap: 1rem; }
  .spotify-header h1 { font-size: 2em; }
  .card-content { flex-direction: column; }
  .cover-section { margin-bottom: 20px; }
  .vinyl-wrapper { width: 180px; height: 180px; }
  .track-title { font-size: 1.5em; text-align: center; }
  .rating { justify-content: center; }
  .comment { text-align: center; }
}
</style>
