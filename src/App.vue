<script setup>
// JavaScript逻辑部分与之前完全相同，未做任何改动
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
    <div class="background-texture"></div>
    <main class="main-container">
      <header class="card-header">
        <h1 class="main-title">AI 爆款文案神器</h1>
        <p class="subtitle">你的随身文案军师，一秒点燃社交圈</p>
      </header>
      
      <section class="control-panel">
        <label class="panel-label">1. 选择灵感风格</label>
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
      </section>

      <section class="control-panel">
        <label class="panel-label">2. 输入灵感火花</label>
        <div class="input-wrapper">
          <textarea 
            v-model="userInput" 
            :rows="textareaRows" 
            placeholder="例如：夏天, 海边, 冰西瓜, 开心"
          ></textarea>
        </div>
      </section>
      
      <button @click="generateCopy" :disabled="loading" class="action-button">
        <span v-if="!loading" class="button-content">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M15.042 21.672L13.684 16.6s.402-1.248 1.524-1.41c1.122-.162 1.628.402 1.628.402l4.974 1.358-1.892 3.722zM19.336 7.425L17.978 2.35s-.402-1.248-1.524-1.41c-1.122-.162-1.628.402-1.628.402l-4.974 1.358 1.892 3.722zM8.664 21.672L10.022 16.6s-.402-1.248-1.524-1.41c-1.122-.162-1.628.402-1.628.402l-4.974 1.358 1.892 3.722zM4.664 7.425L6.022 2.35s.402-1.248 1.524-1.41c1.122-.162 1.628.402 1.628.402l4.974 1.358-1.892 3.722z"></path></svg>
          点燃灵感
        </span>
        <span v-else class="loading-state">
          <svg class="spinner" viewBox="0 0 50 50"><circle class="path" cx="25" cy="25" r="20" fill="none" stroke-width="5"></circle></svg>
          灵感碰撞中...
        </span>
      </button>
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
/* --- “流体网格·最终版”样式 --- */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');

:root {
  /* 视觉风格变量保持不变 */
  --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', sans-serif;
  --bg-color: #111827;
  --card-bg-color: #1f2937;
  --text-color: #d1d5db;
  --text-dim-color: #9ca3af;
  --border-color: rgba(255, 255, 255, 0.1);
  --primary-color: #34d399;
  --primary-hover-color: #a7f3d0;
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
  /* 【核心修改】使用Grid布局 */
  display: grid;
  /* 定义一个三列网格：两侧自动分配空间，中间列最大680px */
  grid-template-columns: 1fr minmax(auto, 680px) 1fr;
  padding: 2rem 1rem;
}

.background-texture {
  position: fixed;
  top: 0; left: 0;
  width: 100%; height: 100%;
  background-image: url('data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 80 80" width="80" height="80"%3E%3Crect fill="%231a202c" width="80" height="80"%3E%3C/rect%3E%3Cg fill-opacity="0.1"%3E%3Ccircle fill="%232d3748" cx="40" cy="40" r="18"%3E%3C/circle%3E%3Ccircle fill="%234a5568" cx="40" cy="40" r="14"%3E%3C/circle%3E%3Ccircle fill="%23718096" cx="40" cy="40" r="10"%3E%3C/circle%3E%3Ccircle fill="%23a0aec0" cx="40" cy="40" r="6"%3E%3C/circle%3E%3C/g%3E%3C/svg%3E');
  opacity: 0.1;
  z-index: -1;
}

.main-container {
  /* 【核心修改】让主内容区占据网格的第二列 */
  grid-column: 2 / 3;
  width: 100%;
  background-color: var(--card-bg-color);
  border: 1px solid var(--border-color);
  border-radius: 24px;
  padding: 2.5rem;
  box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.page-footer {
  /* 【核心修改】页脚也放在网格的第二列 */
  grid-column: 2 / 3;
  text-align: center;
  color: var(--text-dim-color);
  padding: 2rem 0;
}

/* --- 【核心修改】媒体查询，实现响应式 --- */
@media (max-width: 900px) {
  .page-wrapper {
    /* 在小屏幕上，变回单列布局 */
    grid-template-columns: 1fr;
  }
  .main-container, .page-footer {
    grid-column: 1 / -1; /* 占据整个单列 */
  }
}

/* ... 其他所有样式保持不变 ... */
.card-header { text-align: center; }
.main-title { font-size: 2.25rem; font-weight: 700; margin: 0 0 0.5rem; color: #f9fafb; }
.main-title span { color: var(--primary-color); }
.subtitle { font-size: 1rem; color: var(--text-dim-color); margin: 0; }
.control-panel { }
.panel-label { font-size: 0.875rem; font-weight: 500; color: var(--text-dim-color); margin-bottom: 0.75rem; display: block; }
.style-selector { display: grid; grid-template-columns: repeat(auto-fit, minmax(100px, 1fr)); gap: 0.75rem; }
.style-selector button { padding: 0.75rem 0.5rem; border: 1px solid var(--border-color); background-color: transparent; color: var(--text-dim-color); font-size: 0.875rem; font-weight: 500; border-radius: 8px; cursor: pointer; transition: all 0.2s ease; }
.style-selector button:hover { background-color: rgba(255, 255, 255, 0.05); color: var(--text-color); }
.style-selector button.active { background-color: var(--primary-color); color: var(--bg-color); border-color: var(--primary-color); font-weight: 700; }
.input-wrapper { position: relative; }
textarea { width: 100%; background: #111827; border: 1px solid var(--border-color); border-radius: 12px; padding: 1rem; color: var(--text-color); font-size: 1rem; line-height: 1.6; resize: none; transition: all 0.2s ease; }
textarea:focus { outline: none; border-color: var(--primary-color); box-shadow: 0 0 0 3px rgba(52, 211, 153, 0.3); }
.action-button { width: 100%; padding: 1.125rem; border: none; border-radius: 12px; color: #111827; font-size: 1.125rem; font-weight: 600; cursor: pointer; background: var(--primary-color); box-shadow: 0 0 20px rgba(52, 211, 153, 0.3); transition: all 0.2s ease-in-out; animation: pulse 2s infinite; }
.action-button:hover:not(:disabled) { transform: translateY(-3px); box-shadow: 0 0 30px rgba(52, 211, 153, 0.5); }
.action-button:disabled { background: #374151; box-shadow: none; color: var(--text-dim-color); cursor: not-allowed; animation: none; }
.button-content, .loading-state { display: flex; align-items: center; justify-content: center; gap: 0.75rem; }
@keyframes pulse { 0% { box-shadow: 0 0 20px rgba(52, 211, 153, 0.3); } 50% { box-shadow: 0 0 30px rgba(52, 211, 153, 0.5); } 100% { box-shadow: 0 0 20px rgba(52, 211, 153, 0.3); } }
.page-footer a { color: var(--text-dim-color); font-weight: 500; text-decoration: none; }
.page-footer a:hover { color: var(--primary-color); }
.result-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background-color: rgba(17, 24, 39, 0.8); backdrop-filter: blur(8px); -webkit-backdrop-filter: blur(8px); display: flex; justify-content: center; align-items: center; z-index: 100; }
.result-card { width: 90%; max-width: 800px; height: 80vh; background-color: var(--card-bg-color); border: 1px solid var(--border-color); border-radius: 24px; padding: 2rem; box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25); display: flex; flex-direction: column; }
.result-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 1rem; flex-shrink: 0; }
.result-header h3 { margin: 0; font-size: 1.5rem; color: #f9fafb;}
.close-btn { background: none; border: none; font-size: 2.5rem; color: var(--text-dim-color); cursor: pointer; transition: color 0.3s ease; line-height: 1; padding: 0; }
.result-content { flex-grow: 1; overflow-y: auto; background-color: #111827; border-radius: 12px; padding: 1.5rem; }
.result-content pre { white-space: pre-wrap; word-wrap: break-word; font-family: var(--font-sans); font-size: 1rem; line-height: 1.8; margin: 0; color: var(--text-color); }
.slide-fade-enter-active, .slide-fade-leave-active { transition: all 0.5s cubic-bezier(0.16, 1, 0.3, 1); }
.slide-fade-enter-from, .slide-fade-leave-to { opacity: 0; transform: scale(0.95) translateY(20px); }
.spinner { animation: rotate 2s linear infinite; width: 24px; height: 24px; }
.path { stroke: #111827; stroke-linecap: round; animation: dash 1.5s ease-in-out infinite; }
@keyframes rotate { 100% { transform: rotate(360deg); } }
@keyframes dash { 0% { stroke-dasharray: 1, 150; stroke-dashoffset: 0; } 50% { stroke-dasharray: 90, 150; stroke-dashoffset: -35; } 100% { stroke-dasharray: 90, 150; stroke-dashoffset: -124; } }
</style>