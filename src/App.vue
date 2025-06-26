<script setup>
import { ref, computed } from 'vue';

const userInput = ref('');
const loading = ref(false);
const resultText = ref('');
const isResultVisible = ref(false);
const selectedStyle = ref('小红书爆款'); 
const copiedIndex = ref(-1);

const apiKey = import.meta.env.VITE_ZHIPU_API_KEY;

// 风格列表，包含精准的角色定义
const styles = [
  { name: '小红书爆款', icon: '🔥', description: "一位深谙小红书流量密码的博主，文案充满种草感和精致生活气息，善用Emoji和Tag。" },
  { name: '朋友圈文艺', icon: '📜', description: "一位内心细腻的文艺青年，语言充满诗意和淡淡的忧伤，句子简短，留白很多。" },
  { name: '抖音热门', icon: '🎵', description: "一位追求潮流和热点的抖音达人，文案直接、有梗、富有节奏感，能引发模仿和挑战。" },
  { name: '知乎金句', icon: '💡', description: "一位专业的知乎答主，喜欢用“先问是不是，再问为什么”的句式，语言理性、有洞察力，能一语中的。" },
  { name: '搞笑男女', icon: '🤪', description: "一个脑回路清奇、精神状态遥遥领先的搞笑男女。说话风格必须极度沙雕、无厘头、自嘲且接地气，常用网络热梗。拒绝心灵鸡汤和任何正常的情感抒发。例如，提到“努力”，你会说“努力不一定成功，但不努力一定很轻松喔~”。你的目标是让看到的人在三秒内笑出声，并觉得你“有那个大病”。" },
  { name: '搞笑病娇', icon: '🔪', description: "一位在极度占有欲和撒娇痴缠之间反复横跳的“病娇”，文案表面可爱，实则充满了“不可以离开我哦”的警告，又好笑又让人背脊发凉。" }
];

// 输入框的行高计算
const textareaRows = computed(() => {
  const newlines = (userInput.value.match(/\n/g) || []).length;
  return Math.max(3, newlines + 1);
});

// 【核心修复】重新定义我们之前不小心删除的 placeholderText 变量！
const placeholderText = computed(() => {
    return `✨ 例如: 夏天、海边、许愿瓜、开心\n 输入你想要的关键词，用逗号分隔\n 让AI为你创造无限可能的爆款文案\n 每个词都是灵感的种子，等待绽放...`;
});

// 将结果字符串拆分为数组
const resultList = computed(() => {
    if (!resultText.value) return [];
    return resultText.value.split('\n\n').filter(item => item.trim() !== '');
});


async function generateCopy() {
  if (!userInput.value.trim()) { alert('请输入你的灵感关键词！'); return; }
  
  loading.value = true;
  isResultVisible.value = false;

  const currentStyleObject = styles.find(s => s.name === selectedStyle.value);
  const styleDescription = currentStyleObject ? currentStyleObject.description : "一位有创意的文案作者";

  const prompt = `你将扮演一个特定的角色，来为我创作社交媒体文案。\n\n【你的角色定义】\n${styleDescription}\n\n【你的创作任务】\n请根据以下核心关键词，严格按照你的角色设定，创作5条风格鲜明、能引发点赞和评论的文案。\n\n【核心要求】\n1. 核心关键词：『${userInput.value}』\n2. 必备元素：每条文案都必须包含生动有趣、符合语境的Emoji。\n3. 输出格式：请直接输出5条文案，每条之间用两个换行符隔开，不要添加任何额外的解释或标题。\n\n请开始你的创作。`;

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

async function copyResult(textToCopy, index) {
  if (!textToCopy) return;
  try {
    await navigator.clipboard.writeText(textToCopy);
    copiedIndex.value = index;
    setTimeout(() => {
      if (copiedIndex.value === index) {
        copiedIndex.value = -1;
      }
    }, 2000);
  } catch (err) {
    console.error('复制失败:', err);
    alert('复制失败，您的浏览器可能不支持此功能。');
  }
}
</script>

<template>
  <div class="page-wrapper">
    <div class="background-texture"></div>
    <div class="floating-elements">
      <div class="float-el el-1"></div>
      <div class="float-el el-2">⭐</div>
      <div class="float-el el-3"></div>
    </div>
    
    <main class="main-card">
      <header class="card-header">
        <h1 class="main-title">AI爆款文案提取器</h1>
        <p class="subtitle">🚀 你的随身文案军师，一秒点燃社交圈</p>
      </header>
      
      <section class="control-panel">
        <label class="panel-label">
          <span class="label-icon-wrapper">
            <span class="label-icon"></span>
          </span>
          选择灵感风格
        </label>
        <div class="style-selector">
          <button 
            v-for="style in styles" 
            :key="style.name"
            :class="{ active: selectedStyle === style.name }"
            @click="selectedStyle = style.name"
          >
            {{ style.icon }} {{ style.name }}
          </button>
        </div>
      </section>

      <section class="control-panel">
        <label class="panel-label">
          <span class="label-icon-wrapper chat-icon">
            <span class="label-icon"></span>
          </span>
          输入灵感火花
        </label>
        <div class="textarea-wrapper">
          <textarea v-model="userInput" :rows="textareaRows" :placeholder="placeholderText"></textarea>
        </div>
      </section>
      
      <button @click="generateCopy" :disabled="loading" class="action-button">
        <span v-if="!loading" class="button-content">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M13 2L3 14h9l-1 8 10-12h-9l1-8z"></path></svg>
          点燃创意火花
        </span>
        <span v-else class="loading-state">
          <svg class="spinner" viewBox="0 0 50 50"><circle class="path" cx="25" cy="25" r="20" fill="none" stroke-width="5"></circle></svg>
          灵感碰撞中...
        </span>
      </button>
    </main>
    
    <transition name="slide-fade">
      <div v-if="isResultVisible" class="result-overlay">
        <div class="result-card">
          <div class="result-header">
            <h3>✍️ AI为你生成的文案</h3>
            <button @click="isResultVisible=false" class="close-btn">×</button>
          </div>
          <div class="result-content">
            <div class="copy-item" v-for="(item, index) in resultList" :key="index">
              <p class="copy-text">{{ item }}</p>
              <button @click="copyResult(item, index)" class="item-copy-btn">
                <svg v-if="copiedIndex !== index" xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path></svg>
                <svg v-else xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12"></polyline></svg>
              </button>
            </div>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@700;800&display=swap');
:root {
  --font-display: 'Nunito', sans-serif;
  --font-body: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Helvetica Neue', sans-serif;
  --bg-color: #f0fdf4;
  --card-bg-color: #ffffff;
  --text-color: #1f2937;
  --text-dim-color: #6b7280;
  --primary-color: #10b981;
  --primary-gradient: linear-gradient(45deg, #34d399, #10b981);
  --secondary-color: #fde68a;
  --border-color: #f3f4f6;
  --shadow-color: rgba(100, 116, 139, 0.1);
  --success-color: #4ade80;
}
*, *::before, *::after { box-sizing: border-box; }
body { font-family: var(--font-body); background-color: var(--bg-color); color: var(--text-color); margin: 0; }
.page-wrapper { width: 100%; min-height: 100vh; display: grid; place-items: center; padding: 2rem 1rem; position: relative; overflow: hidden; }
.background-texture { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background-image: url('data:image/svg+xml,%3Csvg width="60" height="60" viewBox="0 0 60 60" xmlns="http://www.w3.org/2000/svg"%3E%3Cg fill="none" fill-rule="evenodd"%3E%3Cg fill="%23d4f5e9" fill-opacity="0.4"%3E%3Cpath d="M36 34v-4h-2v4h-4v2h4v4h2v-4h4v-2h-4zm0-30V0h-2v4h-4v2h4v4h2V6h4V4h-4zM6 34v-4H4v4H0v2h4v4h2v-4h4v-2H6zM6 4V0H4v4H0v2h4v4h2V6h4V4H6z"/%3E%3C/g%3E%3C/g%3E%3C/svg%3E'); }
.floating-elements { position: absolute; top: 0; left: 0; width: 100%; height: 100%; overflow: hidden; pointer-events: none; z-index: 0; }
.float-el { position: absolute; animation: float 6s ease-in-out infinite; }
.el-1 { width: 120px; height: 120px; top: 10%; right: 5%; background-color: #fef9c3; border-radius: 50%; opacity: 0.6; }
.el-2 { font-size: 2rem; top: 15%; right: 20%; animation-delay: -2s; color: #facc15; }
.el-3 { width: 80px; height: 80px; bottom: 10%; left: 5%; background-color: #cffafe; border-radius: 50%; opacity: 0.5; animation-delay: -4s;}
@keyframes float { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-20px); } }
.main-card { position: relative; z-index: 1; width: 100%; max-width: 560px; background-color: var(--card-bg-color); border: 1px solid white; border-radius: 40px; padding: 2.5rem; box-shadow: 0 25px 50px -12px var(--shadow-color); display: flex; flex-direction: column; gap: 1.75rem; }
.card-header { text-align: center; }
.main-title { font-family: var(--font-display); font-size: 2.25rem; font-weight: 800; margin: 0 0 0.5rem; color: var(--text-color); }
.subtitle { font-size: 1rem; color: var(--text-dim-color); margin: 0; }
.control-panel { }
.panel-label { display: flex; align-items: center; gap: 0.75rem; font-weight: 700; margin-bottom: 1rem; color: var(--text-color); }
.label-icon-wrapper { width: 28px; height: 28px; border-radius: 8px; background-color: var(--primary-color); display: grid; place-items: center; }
.label-icon-wrapper .label-icon { width: 8px; height: 8px; background-color: white; border-radius: 50%; }
.label-icon-wrapper.chat-icon { background-color: #fbbf24; }
.style-selector { display: grid; grid-template-columns: repeat(auto-fit, minmax(130px, 1fr)); gap: 0.75rem; }
.style-selector button { padding: 0.8rem; border: 1px solid var(--border-color); background-color: #f9fafb; color: var(--text-dim-color); font-size: 0.875rem; font-weight: 600; border-radius: 16px; cursor: pointer; transition: all 0.2s ease; display: flex; align-items: center; justify-content: center; gap: 0.35rem; }
.style-selector button:hover { transform: translateY(-2px); box-shadow: 0 4px 12px var(--shadow-color); border-color: white; }
.style-selector button.active { background: var(--primary-gradient); color: white; border-color: transparent; box-shadow: 0 7px 15px rgba(16, 185, 129, 0.25); }
.textarea-wrapper { border: 1px solid var(--border-color); border-radius: 20px; background-color: white; }
textarea { width: 100%; background: transparent; border: none; resize: none; color: var(--text-color); font-size: 1rem; line-height: 1.6; padding: 1rem; }
textarea::placeholder { color: var(--text-dim-color); white-space: pre-wrap; }
textarea:focus { outline: none; }
.action-button { width: 100%; padding: 1.25rem; border: none; border-radius: 20px; color: white; font-size: 1.25rem; font-weight: 700; cursor: pointer; background: var(--primary-gradient); box-shadow: 0 10px 20px rgba(16, 185, 129, 0.3); transition: all 0.2s ease-in-out; }
.action-button:hover:not(:disabled) { transform: translateY(-3px) scale(1.02); box-shadow: 0 12px 25px rgba(16, 185, 129, 0.4); }
.action-button:disabled { background: #e5e7eb; box-shadow: none; color: #9ca3af; cursor: not-allowed; }
.button-content, .loading-state { display: flex; align-items: center; justify-content: center; gap: 0.75rem; }
.spinner { animation: rotate 2s linear infinite; width: 24px; height: 24px; }
.path { stroke: white; stroke-linecap: round; animation: dash 1.5s ease-in-out infinite; }
@keyframes rotate { 100% { transform: rotate(360deg); } }
@keyframes dash { 0% { stroke-dasharray: 1, 150; stroke-dashoffset: 0; } 50% { stroke-dasharray: 90, 150; stroke-dashoffset: -35; } 100% { stroke-dasharray: 90, 150; stroke-dashoffset: -124; } }
.result-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background-color: rgba(240, 253, 244, 0.8); backdrop-filter: blur(8px); -webkit-backdrop-filter: blur(8px); display: flex; justify-content: center; align-items: center; z-index: 100; }
.result-card { width: 90%; max-width: 800px; height: 80vh; background-color: white; border-radius: 40px; padding: 2.5rem; box-shadow: 0 25px 50px -12px rgba(100, 116, 139, 0.2); display: flex; flex-direction: column; }
.result-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 1.5rem; flex-shrink: 0; }
.result-header h3 { margin: 0; font-size: 1.5rem; font-family: var(--font-display); }
.close-btn { background: none; border: none; font-size: 2.5rem; color: #9ca3af; cursor: pointer; transition: color 0.3s ease; line-height: 1; padding: 0; }
.result-content { flex-grow: 1; overflow-y: auto; background-color: #f9fafb; border-radius: 20px; padding: 1rem; }
.copy-item { display: flex; align-items: center; gap: 1rem; padding: 1rem; border-radius: 16px; transition: background-color 0.2s ease; }
.copy-item:hover { background-color: #f3f4f6; }
.copy-text { flex-grow: 1; margin: 0; font-family: var(--font-body); font-size: 1rem; line-height: 1.8; color: var(--text-color); }
.item-copy-btn { flex-shrink: 0; width: 36px; height: 36px; border: 1px solid var(--border-color); background-color: white; border-radius: 50%; color: var(--text-dim-color); cursor: pointer; transition: all 0.2s ease; display: grid; place-items: center; }
.item-copy-btn:hover { transform: scale(1.1); border-color: var(--primary-color); }
.item-copy-btn svg { transition: all 0.2s ease; }
.item-copy-btn:hover svg { color: var(--primary-color); }
.item-copy-btn.copied { background-color: var(--success-color); border-color: var(--success-color); }
.item-copy-btn.copied svg { color: white; }
.slide-fade-enter-active, .slide-fade-leave-active { transition: all 0.5s cubic-bezier(0.16, 1, 0.3, 1); }
.slide-fade-enter-from, .slide-fade-leave-to { opacity: 0; transform: scale(0.95) translateY(20px); }
@media (max-width: 768px) { .main-card { padding: 1.5rem; } .card-header h1 { font-size: 1.75rem; } .result-card { width: calc(100% - 2rem); height: 85vh; border-radius: 16px; padding: 1.5rem;} }
</style>