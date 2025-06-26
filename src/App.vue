<script setup>
import { ref, computed } from 'vue';

const userInput = ref('');
const loading = ref(false);
const resultText = ref('');
const isResultVisible = ref(false);
const selectedStyle = ref('小红书爆款'); 

const apiKey = import.meta.env.VITE_ZHIPU_API_KEY;

const styles = ['小红书爆款', '朋友圈文艺', '抖音热门', '知乎答主', '搞笑男女'];

const textareaRows = computed(() => {
  const newlines = (userInput.value.match(/\n/g) || []).length;
  return Math.max(5, newlines + 1);
});

async function generateCopy() {
  if (!userInput.value.trim()) {
    alert('请输入你的灵感关键词！');
    return;
  }
  loading.value = true;
  isResultVisible.value = false;
  const prompt = `你是一位深谙社交媒体传播之道的文案大师，尤其精通小红书和抖音的爆款文体。请根据以下信息，为我创作5条风格各异、能引发点赞和评论的文案。\n\n【核心要求】\n1. 文案风格：紧密围绕 “${selectedStyle.value}” 风格进行创作。\n2. 核心关键词：『${userInput.value}』\n3. 必备元素：每条文案都必须包含生动有趣、符合语境的Emoji。\n4. 输出格式：请直接输出5条文案，每条之间用两个换行符隔开，不要添加任何额外的解释或标题。\n\n请开始你的创作。`;
  try {
    const response = await fetch("https://open.bigmodel.cn/api/paas/v4/chat/completions", {
      method: "POST",
      headers: { "Content-Type": "application/json", "Authorization": `Bearer ${apiKey}` },
      body: JSON.stringify({
        model: "glm-3-turbo",
        messages: [{ role: "user", content: prompt }]
      })
    });
    if (!response.ok) {
        const errorBody = await response.json();
        throw new Error(errorBody.error?.message || `HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    resultText.value = data.choices[0].message.content;
    isResultVisible.value = true;
  } catch (error) {
    console.error("请求AI API失败:", error);
    alert(`生成文案时遇到问题: ${error.message}`);
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <div class="page-wrapper">
    <main class="neumorphism-card">
      <header class="card-header">
        <h1 class="main-title">AI 爆款文案神器</h1>
        <p class="subtitle">一秒点燃灵感，轻松引爆社交圈</p>
      </header>
      
      <div class="control-panel">
        <div class="panel-group">
          <label class="group-label">选择一个灵感风格</label>
          <div class="style-selector">
            <button 
              v-for="style in styles" 
              :key="style"
              :class="{ active: selectedStyle === style }"
              @click="selectedStyle = style"
            >
              {{ style }}
            </button>
          </div>
        </div>

        <div class="panel-group">
          <label class="group-label">输入你的灵感火花</label>
          <div class="input-container">
            <svg class="input-icon" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21.44 11.05l-9.19 9.19a6 6 0 0 1-8.49-8.49l9.19-9.19a4 4 0 0 1 5.66 5.66l-9.2 9.19a2 2 0 0 1-2.83-2.83l8.49-8.48"></path></svg>
            <textarea 
              v-model="userInput" 
              :rows="textareaRows" 
              placeholder="例如：夏天, 海边, 冰西瓜, 开心"
            ></textarea>
          </div>
        </div>
      
      <button @click="generateCopy" :disabled="loading" class="action-button">
        <span v-if="!loading">🚀 点燃灵感！</span>
        <span v-else class="loading-state">
          <svg class="spinner" viewBox="0 0 50 50"><circle class="path" cx="25" cy="25" r="20" fill="none" stroke-width="5"></circle></svg>
          灵感碰撞中...
        </span>
      </button>
      </div>
    </main>
    
    <footer class="page-footer">
      <p>由 <a href="https://github.com/WangTong07" target="_blank">WangTong07</a> 匠心打造</p>
    </footer>
    
    <transition name="slide-fade">
      <div v-if="isResultVisible" class="result-overlay">
        <div class="result-card">
          <div class="result-header">
            <h3>✍️ AI为你生成的文案</h3>
            <button @click="isResultVisible=false" class="close-btn">×</button>
          </div>
          <div class="result-content">
            <pre>{{ resultText }}</pre>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<style>
:root {
  --font-sans: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Helvetica Neue', 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', sans-serif;
  --bg-color: #f0f2f5; 
  --text-color: #374151;
  --text-dim-color: #6b7280;
  --primary-gradient: linear-gradient(125deg, #f97316, #ec4899, #8b5cf6, #14b8a6);
  --shadow-light: #ffffff;
  --shadow-dark: #d1d5db;
  --card-radius: 24px;
}
*, *::before, *::after { box-sizing: border-box; }
body {
  font-family: var(--font-sans);
  background-color: var(--bg-color);
  color: var(--text-color);
  margin: 0;
}

.page-wrapper {
  width: 100%;
  min-height: 100vh;
  display: grid;
  place-items: center;
  padding: 2rem 1rem;
}
.neumorphism-card {
  width: 100%;
  max-width: 560px;
  background: var(--bg-color);
  border-radius: var(--card-radius);
  padding: 2.5rem;
  box-shadow: 9px 9px 18px var(--shadow-dark), -9px -9px 18px var(--shadow-light);
  display: flex;
  flex-direction: column;
  gap: 1.75rem;
}

.card-header { text-align: center; }
.main-title {
  font-size: 2.25rem;
  font-weight: 700;
  margin: 0 0 0.5rem;
  background: var(--primary-gradient);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-size: 300% 300%;
  animation: gradient-flow 5s ease infinite;
}
.subtitle { font-size: 1rem; color: var(--text-dim-color); margin: 0; }

@keyframes gradient-flow {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

.control-panel { display: flex; flex-direction: column; gap: 1.5rem; }
.group-label { font-size: 0.875rem; font-weight: 600; color: var(--text-dim-color); margin-bottom: 0.75rem; display: block; }

.style-selector { display: grid; grid-template-columns: repeat(auto-fit, minmax(90px, 1fr)); gap: 0.75rem; }
.style-selector button {
  padding: 0.6rem 0.5rem;
  border: none;
  border-radius: 12px;
  background: var(--bg-color);
  color: var(--text-color);
  font-size: 0.875rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease-in-out;
  box-shadow: 5px 5px 10px var(--shadow-dark), -5px -5px 10px var(--shadow-light);
}
.style-selector button:hover { transform: translateY(-2px); box-shadow: 7px 7px 14px var(--shadow-dark), -7px -7px 14px var(--shadow-light); }
.style-selector button.active {
  box-shadow: inset 5px 5px 10px var(--shadow-dark), inset -5px -5px 10px var(--shadow-light);
  color: #ec4899;
  font-weight: 700;
}

.input-container {
  position: relative;
  border-radius: 16px;
  padding: 1rem 1rem 1rem 3.25rem;
  box-shadow: inset 5px 5px 10px var(--shadow-dark), inset -5px -5px 10px var(--shadow-light);
}
.input-icon { position: absolute; top: 1.1rem; left: 1.1rem; color: var(--text-dim-color); }
textarea { width: 100%; background: transparent; border: none; resize: none; color: var(--text-color); font-size: 1rem; line-height: 1.6; }
textarea:focus { outline: none; }

.action-button {
  width: 100%;
  padding: 1rem;
  border: none;
  border-radius: 16px;
  color: white;
  font-size: 1.125rem;
  font-weight: 600;
  cursor: pointer;
  background: var(--primary-gradient);
  background-size: 300% 300%;
  box-shadow: 5px 5px 10px var(--shadow-dark), -5px -5px 10px var(--shadow-light);
  transition: all 0.2s ease-in-out;
}
.action-button:hover:not(:disabled) {
  animation: gradient-flow 4s ease infinite;
  transform: translateY(-3px);
  box-shadow: 8px 8px 16px var(--shadow-dark), -8px -8px 16px var(--shadow-light);
}
.action-button:active:not(:disabled) { box-shadow: inset 5px 5px 10px #c62e65, inset -5px -5px 10px #f8d387; }
.action-button:disabled { background: #d1d9e6; box-shadow: none; color: var(--text-dim-color); cursor: not-allowed; animation: none; }

.loading-state { display: flex; align-items: center; justify-content: center; gap: 0.75rem; }
.spinner { animation: rotate 2s linear infinite; width: 24px; height: 24px; }
.path { stroke: white; stroke-linecap: round; animation: dash 1.5s ease-in-out infinite; }
@keyframes rotate { 100% { transform: rotate(360deg); } }
@keyframes dash { 0% { stroke-dasharray: 1, 150; stroke-dashoffset: 0; } 50% { stroke-dasharray: 90, 150; stroke-dashoffset: -35; } 100% { stroke-dasharray: 90, 150; stroke-dashoffset: -124; } }
.page-footer { text-align: center; color: var(--text-dim-color); margin-top: 2rem; }
.page-footer a { color: var(--text-color); font-weight: 500; text-decoration: none; }
.result-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background-color: rgba(238, 240, 244, 0.7); backdrop-filter: blur(8px); -webkit-backdrop-filter: blur(8px); display: flex; justify-content: center; align-items: center; z-index: 100; }
.result-card { width: 90%; max-width: 800px; height: 80vh; background: var(--bg-color); border-radius: var(--card-radius); padding: 2rem; box-shadow: 8px 8px 16px var(--shadow-dark), -8px -8px 16px var(--shadow-light); display: flex; flex-direction: column; }
.result-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 1rem; flex-shrink: 0; }
.result-header h3 { margin: 0; font-size: 1.5rem; }
.close-btn { background: none; border: none; font-size: 2.5rem; color: var(--text-dim-color); cursor: pointer; transition: color 0.3s ease; line-height: 1; padding: 0; }
.result-content { flex-grow: 1; overflow-y: auto; background: var(--bg-color); border-radius: 20px; padding: 1.5rem; box-shadow: inset 5px 5px 10px var(--shadow-dark), inset -5px -5px 10px var(--shadow-light); }
.result-content pre { white-space: pre-wrap; word-wrap: break-word; font-family: var(--font-sans); font-size: 1rem; line-height: 1.8; margin: 0; color: var(--text-color); }
.slide-fade-enter-active, .slide-fade-leave-active { transition: all 0.5s cubic-bezier(0.16, 1, 0.3, 1); }
.slide-fade-enter-from, .slide-fade-leave-to { opacity: 0; transform: scale(0.95) translateY(20px); }
@media (max-width: 768px) { .neumorphism-card { padding: 1.5rem; } .card-header h1 { font-size: 1.75rem; } }
</style>