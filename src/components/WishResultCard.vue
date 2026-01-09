<script setup>
import { ref, onMounted, nextTick } from 'vue';
import { animate } from 'animejs';
import { snapdom } from '@zumer/snapdom'; // 确保你安装了 npm install @zumer/snapdom

// 动态添加 DaisyUI CSS (保持你的原始逻辑)
onMounted(() => {
  if (!document.querySelector('link[href*="daisyui"]')) {
    const link = document.createElement('link');
    link.rel = 'stylesheet';
    link.href = 'https://cdn.bootcdn.net/ajax/libs/daisyui/4.12.23/full.min.css';
    document.head.appendChild(link);
  }
});

const props = defineProps({
  signData: {
    type: Object,
    required: true
  }
});

const emit = defineEmits(['restart']);
const cardRef = ref(null);
const isDownloading = ref(false); // 下载状态，防止重复点击

// 重新开始
function handleRestart() {
  emit('restart');
}

// 下载图片逻辑
async function handleDownload() {
  if (!cardRef.value || isDownloading.value) return;

  isDownloading.value = true;

  try {
    // 1. 临时强制设置宽度，为了让移动端截图也能像桌面端一样宽
    // 保存原始样式以便恢复
    const originalWidth = cardRef.value.style.width;
    const originalMaxWidth = cardRef.value.style.maxWidth;

    // 强制设定为桌面端的典型宽度 (例如 600px)，防止手机上截成细长条
    // 注意：用户可能会瞬间看到闪烁，但这是为了截图正确
    cardRef.value.style.width = '600px';
    cardRef.value.style.maxWidth = 'none';

    // 2. 使用 snapdom 截图
    // 这里的 scale: 2 是为了高清屏（Retina）效果，保证文字清晰
    const result = await snapdom(cardRef.value, {
      scale: 2,
      style: {
        // 确保截图背景是干净的，防止透明
        backgroundColor: '#fff9f0',
      }
    });

    // 3. 触发下载
    // 文件名带上时间戳，增加仪式感
    const filename = `PerfectWish_Contract_${Date.now()}`;
    await result.download({ format: 'png', filename: filename });

    // 4. 恢复样式
    cardRef.value.style.width = originalWidth;
    cardRef.value.style.maxWidth = originalMaxWidth;

  } catch (error) {
    console.error('保存契约失败:', error);
    alert('保存失败，请截图保存 w');
  } finally {
    isDownloading.value = false;
  }
}

// 入场动画
onMounted(async () => {
  await nextTick();
  if (cardRef.value) {
    animate(cardRef.value, {
      opacity: [0, 1],
      scale: [0.9, 1],
      duration: 1000,
      easing: 'easeOutElastic(1, .8)'
    });
  }
});
</script>

<template>
  <div class="result-card-container">
    <div ref="cardRef" class="card bg-base-100 shadow-xl result-card">
      <div class="contract-header"></div>

      <div class="card-body">
        <div class="wish-section">
          <div class="label-tag">许愿者</div>
          <p class="wisher-name">{{ props.signData.name }}</p>
          <div class="label-tag">原定愿望</div>
          <h2 class="wish-title">「 {{ props.signData.confirmed_wish }} 」</h2>
        </div>

        <div class="divider">契约达成情况</div>

        <div class="realization-section">
          <div class="label-tag danger">实现场景 (逻辑检查通过)</div>
          <p class="scenario-text">
            {{ props.signData.realization_scenario }}
          </p>

          <div class="contract-seal">已达成</div>
        </div>

        <div class="disclaimer">
          警告：本系统遵循严格的逻辑演绎，任何利益受损均由许愿者逻辑不严密引起。<br>
          Powered by DeepSeek & 完美许愿器 v2.0
        </div>

        <div class="promo-link">
          <div class="link-text">完美许愿器 · wish.closeai.moe</div>
          <div class="author-credit">@阿尼亚是安妮亞</div>
        </div>
      </div>
    </div>

    <div class="action-buttons">
      <button @click="handleDownload" class="btn btn-outline btn-secondary shadow-md" :disabled="isDownloading">
        <span v-if="!isDownloading">保存契约图片</span>
        <span v-else class="loading loading-spinner loading-sm"></span>
      </button>

      <button @click="handleRestart" class="btn btn-primary shadow-lg">
        重新修正愿望 w
      </button>
    </div>

    <div class="stats-hint">
      <p>
        完美许愿器已经处理超过 <span class="num">120万</span> 个愿望。
      </p>
      <p>
        如果你因此感到“快乐”，可以点击页面下方的按钮支持我w
      </p>
    </div>

  </div>
</template>

<style scoped>
.result-card-container {
  max-width: 600px;
  /* 这里的限制只影响网页显示 */
  margin: 0 auto;
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 2rem;
  /* 防止截图时强制撑大导致页面横向滚动条出现 */
  overflow-x: visible;
}

.card {
  background: linear-gradient(135deg, #fff9f0 0%, #fef8ed 100%) !important;
  /* 像旧纸张一样的颜色 */
  border: 3px solid #2c3e50;
  color: #2c3e50 !important;
  opacity: 0;
  position: relative;
  /* overflow: hidden; 去掉这个，防止印章被切掉 */
  border-radius: 12px;
  /* 稍微方正一点 */
  box-shadow: 0 8px 24px rgba(0,0,0,0.15), 0 4px 8px rgba(0,0,0,0.1);
}

/* 契约顶部的条纹 */
.contract-header {
  height: 16px;
  background: repeating-linear-gradient(45deg,
      #2c3e50,
      #2c3e50 12px,
      #e74c3c 12px,
      #e74c3c 24px);
  border-bottom: 3px solid #2c3e50;
}

.label-tag {
  font-size: 0.75rem;
  font-weight: bold;
  text-transform: uppercase;
  color: #7f8c8d;
  margin-bottom: 0.5rem;
  letter-spacing: 1.5px;
  position: relative;
  display: inline-block;
  padding: 4px 12px;
  background: rgba(127, 140, 141, 0.1);
  border-radius: 4px;
}

.label-tag.danger {
  color: #e74c3c;
  background: rgba(231, 76, 60, 0.1);
}

.wisher-name {
  font-size: 1.3rem;
  font-weight: 700;
  text-align: center;
  color: #2c3e50;
  margin-bottom: 1.2rem;
  padding: 8px 16px;
  background: linear-gradient(135deg, rgba(142, 68, 173, 0.1), rgba(142, 68, 173, 0.05));
  border-radius: 8px;
  border-left: 4px solid #8e44ad;
}

.wish-title {
  font-size: 1.5rem;
  font-weight: 800;
  text-align: center;
  padding: 1.2rem 0;
  font-style: italic;
  font-family: "Songti SC", "SimSun", serif;
  /* 增加一点衬线体感觉 */
  background: linear-gradient(135deg, #8e44ad, #e74c3c);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  line-height: 1.6;
}

.divider {
  font-size: 0.85rem;
  color: #bdc3c7;
  margin: 1rem 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  letter-spacing: 2px;
}

.divider::before,
.divider::after {
  content: '';
  flex: 1;
  height: 2px;
  background: linear-gradient(90deg, transparent, #bdc3c7, transparent);
  margin: 0 15px;
}

.realization-section {
  position: relative;
  background: linear-gradient(135deg, rgba(44, 62, 80, 0.04), rgba(44, 62, 80, 0.02));
  padding: 1.8rem;
  border-radius: 8px;
  border: 2px dashed #bdc3c7;
  min-height: 150px;
  box-shadow: inset 0 2px 8px rgba(0,0,0,0.05);
}

.scenario-text {
  font-size: 1.15rem;
  line-height: 1.9;
  color: #2c3e50;
  z-index: 1;
  position: relative;
  white-space: pre-wrap;
  text-align: justify;
  text-indent: 2em;
}

/* 装饰性印章样式 */
.contract-seal {
  position: absolute;
  right: 15px;
  bottom: 15px;
  width: 100px;
  height: 100px;
  border: 5px double rgba(231, 76, 60, 0.6);
  border-radius: 50%;
  color: rgba(231, 76, 60, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 900;
  transform: rotate(-20deg);
  font-size: 1.3rem;
  pointer-events: none;
  user-select: none;
  background: radial-gradient(circle, rgba(231, 76, 60, 0.05), transparent);
  box-shadow: 0 4px 12px rgba(231, 76, 60, 0.3);
}

.disclaimer {
  font-size: 0.75rem;
  color: #95a5a6;
  text-align: center;
  margin-top: 1.8rem;
  padding-top: 1.2rem;
  border-top: 2px solid rgba(0, 0, 0, 0.08);
  line-height: 1.6;
}

/* 修改原有的 .promo-link，并新增下级样式 */
.promo-link {
  text-align: center;
  font-family: monospace;
  margin-top: 10px;
  /* 使用 Flex 让两行文字垂直居中排列 */
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px; /* 两行文字之间的间距 */
}

.promo-link .link-text {
  font-size: 0.8rem;
  color: #bdc3c7;
  letter-spacing: 1.5px;
  opacity: 0.9;
}

.promo-link .author-credit {
  font-size: 0.75rem; /* 稍微小一点，作为署名 */
  color: #95a5a6;    /* 稍微深一点或者淡一点的灰色，看你喜好 */
  letter-spacing: 0.8px;
  opacity: 0.7;      /* 降低一点存在感，显得更精致 */
}

.action-buttons {
  display: flex;
  justify-content: center;
  gap: 20px;
  /* 按钮之间分开一点 */
  flex-wrap: wrap;
}

/* 按钮样式微调 */
.btn {
  min-width: 160px;
  padding: 12px 24px;
  font-size: 1rem;
  border-radius: 8px;
  font-weight: 600;
  letter-spacing: 1px;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(0,0,0,0.2);
}

.btn-primary {
  background: linear-gradient(135deg, #2c3e50, #34495e) !important;
  border-color: #2c3e50 !important;
  color: white !important;
}

.btn-primary:hover {
  background: linear-gradient(135deg, #1a252f, #2c3e50) !important;
}

.btn-secondary {
  background: white !important;
  border: 2px solid #2c3e50 !important;
  color: #2c3e50 !important;
}

.btn-secondary:hover {
  background: linear-gradient(135deg, #2c3e50, #34495e) !important;
  color: white !important;
}

@media (max-width: 640px) {
  .wish-title {
    font-size: 1.2rem;
  }

  .scenario-text {
    font-size: 1rem;
  }

  /* 移动端按钮竖排 */
  .action-buttons {
    flex-direction: column;
    width: 100%;
  }

  .btn {
    width: 100%;
  }
}

/* 新增样式：底部统计提示 */
.stats-hint {
  text-align: center;
  margin-top: 1rem;
  /* 与按钮拉开一点距离 */
  margin-bottom: 1rem;
  /* 底部留白 */
  font-size: 0.8rem;
  /* 字号偏小，显得精致 */
  color: #7f8c8d;
  /* 使用低调的灰色，不刺眼 */
  line-height: 1.6;
  /* 行高拉开，防止两行挤在一起 */
}

/* 数字强调 */
.stats-hint .num {
  color: #2c3e50;
  /* 深色突出数字 */
  font-weight: bold;
  font-family: 'Courier New', monospace;
  /* 等宽字体增加一点“系统数据”的感觉 */
  margin: 0 2px;
}

/* 移动端适配微调 */
@media (max-width: 640px) {
  .stats-hint {
    font-size: 0.8rem;
    /* 手机上字体再稍微小一点点 */
    padding: 12px 15px;
    /* 防止文字贴边 */
  }
}
  /* ...原有移动端样式... */

  .stats-hint {
    font-size: 0.75rem;
    /* 手机上字体再稍微小一点点 */
    padding: 0 10px;
    /* 防止文字贴边 */
  }
}
</style>