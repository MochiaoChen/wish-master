<script setup>
import { ref, computed } from 'vue';
import { useWishEnergy } from '../composables/useWishEnergy';

// 定义组件事件
const emit = defineEmits(['submit']);

const { count, stability, countdownStr, consumeEnergy, MAX_ENERGY, MAX_STABILITY } = useWishEnergy();

// 愿望输入
const wishInput = ref('');
const nameInput = ref('');
const isSubmitting = ref(false);
// 呜呜，把字数限制提升到80啦
const maxLength = 80;

// 更改为更有“欲望感”的示例
const exampleWishes = [
  '想要永远不用工作',
  '希望变身亿万富翁',
  '想让大家都听我的'
];

// 处理提交
function handleSubmit() {
  if (!wishInput.value.trim() || !nameInput.value.trim() || isSubmitting.value || count.value <= 0 || stability.value <= 0) return;
  if (wishInput.value.length > maxLength) return;
  
  if (consumeEnergy()) {
    isSubmitting.value = true;
    emit('submit', { wish: wishInput.value, name: nameInput.value });
    
    // 重置状态
    setTimeout(() => {
      isSubmitting.value = false;
    }, 500);
  }
}

// 填充示例愿望
function fillExampleWish(wish) {
  wishInput.value = wish;
}

// 计算剩余字数
const remainingChars = computed(() => {
  return maxLength - wishInput.value.length;
});
</script>

<template>
  <div class="wish-input-container">
    <h2 class="input-title">契约内容</h2>
    
    <div class="input-wrapper">
      <input
        v-model="nameInput"
        type="text"
        class="name-input"
        placeholder="请输入你的名字..."
        maxlength="20"
        @keyup.enter="handleSubmit"
      />
    </div>

    <div class="input-wrapper">
      <input
        v-model="wishInput"
        type="text"
        class="wish-input"
        placeholder="在此写下你渴望之物..."
        :maxlength="maxLength"
        @keyup.enter="handleSubmit"
      />
      <span class="char-counter" :class="{ 'warning': remainingChars < 10 }">
        {{ remainingChars }}
      </span>
    </div>

    <div class="energy-status-row">
      <!-- CD 时间 -->
      <div v-if="count < MAX_ENERGY" class="cd-text">
        ({{ countdownStr }})
      </div>

      <!-- 紫色能量点 -->
      <div class="slots">
        <div 
          v-for="i in MAX_ENERGY" 
          :key="'e'+i" 
          class="dot purple" 
          :class="{ active: i <= count }"
        ></div>
      </div>

      <!-- 分隔符 -->
      <div class="separator">|</div>

      <!-- 红色能量点 -->
      <div class="slots">
        <div 
          v-for="i in MAX_STABILITY" 
          :key="'s'+i" 
          class="dot red" 
          :class="{ active: i <= stability }"
        ></div>
      </div>
    </div>

    <!-- 灵魂耗尽提示 -->
    <transition name="fade">
      <div v-if="stability <= 0" class="stability-alert">
        <p>「灵魂已支离破碎...」</p>
        <small>因果律已拒绝你的连接。唯有完成一次纯粹的愿望，方可重塑灵魂。</small>
      </div>
    </transition>
    
    <button 
      @click="handleSubmit" 
      class="submit-button"
      :disabled="!wishInput.trim() || !nameInput.trim() || wishInput.length > maxLength || isSubmitting || count <= 0 || stability <= 0"
    >
      <template v-if="!isSubmitting">
        <span v-if="stability <= 0">灵魂破碎</span>
        <span v-else-if="count > 0">签订契约</span>
        <span v-else>能量不足</span>
      </template>
      <span v-else class="loading-dots">因果计算中<span>.</span><span>.</span><span>.</span></span>
    </button>
    
    <div class="examples-container">
      <p class="examples-title">常见欲望：</p>
      <div class="examples-list">
        <button 
          v-for="(wish, index) in exampleWishes" 
          :key="index"
          @click="fillExampleWish(wish)"
          class="example-item"
        >
          {{ wish }}
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.wish-input-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  max-width: 500px;
  margin: 0 auto;
}

.input-title {
  font-size: 1.5rem;
  margin-bottom: 30px;
  color: #2c3e50;
  text-align: center;
  font-weight: 700;
  letter-spacing: 3px;
  text-transform: uppercase;
  position: relative;
  padding-bottom: 15px;
}

.input-title::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 60px;
  height: 3px;
  background: linear-gradient(90deg, #8e44ad, #e74c3c);
  border-radius: 2px;
}

.input-wrapper {
  position: relative;
  width: 100%;
  margin-bottom: 20px;
}

.name-input,
.wish-input {
  width: 100%;
  padding: 16px 20px;
  font-size: 1.1rem;
  border: 2px solid #e0e0e0;
  border-radius: 8px; 
  background: linear-gradient(to bottom, #ffffff, #fafafa);
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
}

.wish-input {
  padding-right: 50px;
}

.name-input:focus,
.wish-input:focus {
  outline: none;
  border-color: #8e44ad;
  background: #ffffff;
  box-shadow: 0 4px 12px rgba(142, 68, 173, 0.2);
  transform: translateY(-1px);
}

.char-counter {
  position: absolute;
  right: 16px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 0.85rem;
  color: #999;
  font-weight: 600;
  background: white;
  padding: 2px 6px;
  border-radius: 4px;
}

.char-counter.warning {
  color: #e74c3c;
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.6; }
}

.energy-status-row {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  margin-bottom: 25px;
  padding: 12px 20px;
  background: rgba(44, 62, 80, 0.03);
  border-radius: 8px;
  font-family: monospace;
}

.cd-text {
  font-size: 0.85rem;
  color: #7f8c8d;
  font-weight: 600;
}

.slots {
  display: flex;
  gap: 8px;
}

.dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  border: 2px solid #ddd;
  background-color: transparent;
  transition: all 0.3s ease;
}

.dot.purple.active {
  background: linear-gradient(135deg, #8e44ad, #9b59b6);
  border-color: #8e44ad;
  box-shadow: 0 0 8px rgba(142, 68, 173, 0.6), 0 0 12px rgba(142, 68, 173, 0.3);
  animation: glow-purple 2s infinite;
}

.dot.red.active {
  background: linear-gradient(135deg, #e74c3c, #c0392b);
  border-color: #e74c3c;
  box-shadow: 0 0 8px rgba(231, 76, 60, 0.6), 0 0 12px rgba(231, 76, 60, 0.3);
  animation: glow-red 2s infinite;
}

@keyframes glow-purple {
  0%, 100% { box-shadow: 0 0 8px rgba(142, 68, 173, 0.6), 0 0 12px rgba(142, 68, 173, 0.3); }
  50% { box-shadow: 0 0 12px rgba(142, 68, 173, 0.8), 0 0 16px rgba(142, 68, 173, 0.5); }
}

@keyframes glow-red {
  0%, 100% { box-shadow: 0 0 8px rgba(231, 76, 60, 0.6), 0 0 12px rgba(231, 76, 60, 0.3); }
  50% { box-shadow: 0 0 12px rgba(231, 76, 60, 0.8), 0 0 16px rgba(231, 76, 60, 0.5); }
}

.separator {
  color: #bdc3c7;
  font-weight: bold;
  margin: 0 6px;
  font-size: 1.1rem;
}

.stability-alert {
  background: linear-gradient(135deg, rgba(231, 76, 60, 0.1), rgba(192, 57, 43, 0.05));
  border: 2px solid #e74c3c;
  padding: 15px 20px;
  border-radius: 8px;
  margin-bottom: 20px;
  text-align: center;
  color: #c0392b;
  box-shadow: 0 4px 12px rgba(231, 76, 60, 0.2);
}

.stability-alert p {
  margin: 0 0 8px 0;
  font-weight: bold;
  font-size: 1rem;
}

.stability-alert small {
  font-size: 0.85rem;
  display: block;
  line-height: 1.4;
}

.submit-button {
  background: linear-gradient(135deg, #2c3e50, #34495e);
  color: white;
  border: none;
  padding: 16px 40px;
  font-size: 1.15rem;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  min-width: 180px;
  letter-spacing: 3px;
  font-weight: bold;
  box-shadow: 0 4px 12px rgba(44, 62, 80, 0.3);
  position: relative;
  overflow: hidden;
}

.submit-button::before {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 0;
  height: 0;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.2);
  transform: translate(-50%, -50%);
  transition: width 0.6s, height 0.6s;
}

.submit-button:hover:not(:disabled)::before {
  width: 300px;
  height: 300px;
}

.submit-button:hover:not(:disabled) {
  background: linear-gradient(135deg, #1a252f, #2c3e50);
  transform: translateY(-3px);
  box-shadow: 0 6px 20px rgba(44, 62, 80, 0.4);
}

.submit-button:active:not(:disabled) {
  transform: translateY(-1px);
}

.submit-button:disabled {
  background: linear-gradient(135deg, #bdc3c7, #95a5a6);
  cursor: not-allowed;
  box-shadow: none;
}

.examples-container {
  margin-top: 40px;
  width: 100%;
}

.examples-title {
  font-size: 0.9rem;
  color: #7f8c8d;
  margin-bottom: 15px;
  text-align: center;
  font-weight: 600;
  letter-spacing: 1px;
}

.examples-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 12px;
}

.example-item {
  background: linear-gradient(135deg, #ffffff, #f8f9fa);
  border: 2px dashed #ddd;
  border-radius: 6px;
  padding: 8px 16px;
  font-size: 0.9rem;
  color: #7f8c8d;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
}

.example-item:hover {
  border-style: solid;
  border-color: #8e44ad;
  color: #8e44ad;
  background: linear-gradient(135deg, #faf5ff, #f3e5f5);
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(142, 68, 173, 0.2);
}

/* 加载动画 */
.loading-dots span {
  animation: loadingDots 1.4s infinite;
  animation-fill-mode: both;
}

.loading-dots span:nth-child(2) {
  animation-delay: 0.2s;
}

.loading-dots span:nth-child(3) {
  animation-delay: 0.4s;
}

@keyframes loadingDots {
  0% { opacity: 0.2; }
  20% { opacity: 1; }
  100% { opacity: 0.2; }
}

/* 简单的淡入淡出过渡 */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.4s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>

