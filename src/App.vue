// “AI文案神器” 最终版 App.vue
<script setup>
import { ref, computed } from 'vue';

const userInput = ref('');
const loading = ref(false);
const resultText = ref('');
const isResultVisible = ref(false);
// 【新】用于存储用户选择的风格，默认为'小红书爆款'
const selectedStyle = ref('小红书爆款'); 

const apiKey = import.meta.env.VITE_ZHIPU_API_KEY;

// 【新】风格选择列表
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

  // 【新】为“文案神器”量身定制的全新Prompt
  const prompt = `你是一位深谙社交媒体传播之道的文案大师，尤其精通小红书和抖音的爆款文体。请根据以下信息，为我创作5条风格各异、能引发点赞和评论的文案。

  【核心要求】
  1. 文案风格：紧密围绕 “${selectedStyle.value}” 风格进行创作。
  2. 核心关键词：『${userInput.value}』
  3. 必备元素：每条文案都必须包含生动有趣、符合语境的Emoji。
  4. 输出格式：请直接输出5条文案，每条之间用两个换行符隔开，不要添加任何额外的解释或标题。

  请开始你的创作。`;

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
    <div class="aurora-background">
      <div class="aurora-shape shape-1"></div>
      <div class="aurora-shape shape-2"></div>
      <div class="aurora-shape shape-3"></div>
    </div>

    <main class="content-area">
      <div class="glass-card">
          <header class="card-header">
            <div class="logo">AI</div>
            <h1>AI 爆款文案神器</h1>
            <p>你的随身文案军师，一键引爆社交圈</p>
          </header>
          
          <section class="style-selector">
            <button 
              v-for="style in styles" 
              :key="style"
              :class="{ active: selectedStyle === style }"
              @click="selectedStyle = style"
            >
              {{ style }}
            </button>
          </section>
          
          <section class="input-container">
            <textarea 
              v-model="userInput" 
              :rows="textareaRows" 
              placeholder="输入灵感关键词，用逗号或空格隔开...
例如：夏天, 傍晚, 冰西瓜, 开心"
            ></textarea>
          </section>
          
          <button @click="generateCopy" :disabled="loading" class="action-button">
            <span v-if="!loading">✨ 生成爆款文案</span>
            <span v-else class="loading-state">
              <svg class="spinner" viewBox="0 0 50 50"><circle class="path" cx="25" cy="25" r="20" fill="none" stroke-width="4"></circle></svg>
              灵感碰撞中...
            </span>
          </button>
      </div>
    </main>
    
    <footer class="page-footer">
      <p>由 <a href="https://gitee.com/wangtong07" target="_blank">WangTong07</a> 匠心打造</p>
    </footer>
    
    <transition name="slide-fade">
      <div v-if="isResultVisible" class="result-overlay">
        <div class="result-card">
              <div class="result-header">
                  <h3>AI为你生成的文案</h3>
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
/* 这份CSS复用了我们最终的完美样式，并为新模块增加了样式 */
:root {
  --font-sans: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Helvetica Neue', 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', sans-serif;
  --color-text: #e2e8f0;
  --color-text-dim: #94a3b8;
  --color-bg: #0f172a;
  --color-glass-bg: rgba(30, 41, 59, 0.75);
  --color-border: rgba(148, 163, 184, 0.2);
  --color-primary: #818cf8;
  --color-aurora-1: #f43f5e;
  --color-aurora-2: #3b82f6;
  --color-aurora-3: #14b8a6;
}
*, *::before, *::after { box-sizing: border-box; }
html, body { height: 100%; margin: 0; }
body { font-family: var(--font-sans); background-color: var(--color-bg); color: var(--color-text); }
.page-wrapper { position: relative; width: 100%; min-height: 100%; display: grid; place-items: center; padding: 2rem 1rem; overflow-x: hidden; }
.aurora-background { position: fixed; top: 0; left: 0; width: 100%; height: 100%; overflow: hidden; z-index: -1; }
.content-area { width: 100%; max-width: 680px; display: flex; flex-direction: column; align-items: center; position: relative; z-index: 10; }
.glass-card { width: 100%; background: var(--color-glass-bg); border: 1px solid var(--color-border); border-radius: 24px; padding: 2.5rem; backdrop-filter: blur(20px); -webkit-backdrop-filter: blur(20px); box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.37); display: flex; flex-direction: column; gap: 1.75rem; }
.page-footer { margin-top: 1.5rem; width: 100%; text-align: center; color: var(--color-text-dim); font-size: 0.875rem; }
.aurora-shape { position: absolute; filter: blur(120px); opacity: 0.3; border-radius: 50%; animation: move 20s infinite alternate; }
.shape-1 { width: 500px; height: 500px; background-color: var(--color-aurora-1); top: -20%; left: -20%; }
.shape-2 { width: 400px; height: 400px; background-color: var(--color-aurora-2); top: 30%; right: -10%; animation-delay: 5s; }
.shape-3 { width: 450px; height: 450px; background-color: var(--color-aurora-3); bottom: -25%; left: 10%; animation-delay: 10s; }
@keyframes move { from { transform: translate(0, 0) rotate(0deg); } to { transform: translate(100px, 50px) rotate(60deg); } }
.result-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background-color: rgba(15, 23, 42, 0.5); backdrop-filter: blur(8px); -webkit-backdrop-filter: blur(8px); display: flex; justify-content: center; align-items: center; z-index: 100; }
.result-card { width: 90%; max-width: 800px; height: 80vh; background: var(--color-glass-bg); border: 1px solid var(--color-border); border-radius: 24px; padding: 2rem; box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.37); display: flex; flex-direction: column; }
.result-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 1rem; flex-shrink: 0; gap: 1rem; }
.result-header h3 { flex-grow: 1; margin: 0; font-size: 1.5rem; }
.close-btn { background: none; border: none; font-size: 2.5rem; color: var(--color-text-dim); cursor: pointer; transition: color 0.3s ease; line-height: 1; padding: 0; }
.close-btn:hover { color: var(--color-text); }
.result-content { flex-grow: 1; overflow-y: auto; background: rgba(15, 23, 42, 0.7); border-radius: 8px; padding: 1.5rem; }
.result-content pre { white-space: pre-wrap; word-wrap: break-word; font-family: var(--font-sans); font-size: 1rem; line-height: 1.8; margin: 0; color: #cbd5e1; }
.slide-fade-enter-active, .slide-fade-leave-active { transition: all 0.5s cubic-bezier(0.16, 1, 0.3, 1); }
.slide-fade-enter-from, .slide-fade-leave-to { opacity: 0; transform: scale(0.95) translateY(20px); }
.card-header { text-align: center; }
.logo { display: inline-block; background: linear-gradient(135deg, var(--color-aurora-1), var(--color-aurora-2)); color: white; width: 48px; height: 48px; border-radius: 12px; font-weight: 700; font-size: 1.5rem; line-height: 48px; margin-bottom: 1rem; }
.card-header h1 { font-size: 2rem; margin: 0; }
.card-header p { color: var(--color-text-dim); margin: 0.5rem 0 0; }
.style-selector { display: grid; grid-template-columns: repeat(auto-fit, minmax(100px, 1fr)); gap: 0.75rem; }
.style-selector button { padding: 0.75rem 0.5rem; border: 1px solid var(--color-border); background-color: transparent; color: var(--color-text-dim); font-size: 0.875rem; font-weight: 500; border-radius: 8px; cursor: pointer; transition: all 0.3s ease; }
.style-selector button.active { background-color: var(--color-primary); color: white; border-color: var(--color-primary); box-shadow: 0 2px 10px rgba(129, 140, 248, 0.2); }
.input-container textarea { width: 100%; background: rgba(15, 23, 42, 0.7); border: 1px solid var(--color-border); border-radius: 12px; padding: 1rem; color: var(--color-text); font-size: 1rem; line-height: 1.6; resize: none; transition: all 0.3s ease; }
.input-container textarea:focus { outline: none; border-color: var(--color-primary); box-shadow: 0 0 0 3px rgba(129, 140, 248, 0.3); }
.action-button { width: 100%; padding: 1rem; border: none; border-radius: 12px; background: linear-gradient(90deg, var(--color-primary), var(--color-aurora-2)); color: white; font-size: 1.125rem; font-weight: 600; cursor: pointer; transition: all 0.3s ease; box-shadow: 0 4px 15px rgba(129, 140, 248, 0.2); }
.action-button:hover:not(:disabled) { transform: translateY(-3px); box-shadow: 0 7px 20px rgba(129, 140, 248, 0.3); }
.action-button:disabled { background: #334155; cursor: not-allowed; box-shadow: none; }
.loading-state { display: flex; align-items: center; justify-content: center; gap: 0.75rem; }
.page-footer a { color: var(--color-primary); text-decoration: none; font-weight: 500; transition: color 0.3s ease; }
.page-footer a:hover { color: white; }
.spinner { animation: rotate 2s linear infinite; width: 20px; height: 20px; }
.path { stroke: white; stroke-linecap: round; animation: dash 1.5s ease-in-out infinite; }
@keyframes rotate { 100% { transform: rotate(360deg); } }
@keyframes dash { 0% { stroke-dasharray: 1, 150; stroke-dashoffset: 0; } 50% { stroke-dasharray: 90, 150; stroke-dashoffset: -35; } 100% { stroke-dasharray: 90, 150; stroke-dashoffset: -124; } }
@media (max-width: 768px) { .content-area { padding: 1rem; } .glass-card { padding: 1.5rem; } .card-header h1 { font-size: 1.75rem; } .style-selector { grid-template-columns: repeat(2, 1fr); } .result-card { width: calc(100% - 2rem); height: 85vh; border-radius: 16px; padding: 1.5rem;} }
</style>