<script setup>
import { ref, computed, watch, nextTick } from 'vue';
import SecondPicker from './SecondPicker.vue';
import MinutePicker from './MinutePicker.vue';
import HourPicker from './HourPicker.vue';
import DayPicker from './DayPicker.vue';
import MonthPicker from './MonthPicker.vue';
import YearPicker from './YearPicker.vue';
import WeekPicker from './WeekPicker.vue';

const props = defineProps({
  visible: {
    type: Boolean,
    default: false,
  },
  title: {
    type: String,
    default: "选择时间",
  },
  modelValue: {
    type: String,
    default: '*',
  }
});

const emit = defineEmits(["update:visible", "confirm", "cancel"]);
const activeTab = ref('second');
const secondValue = ref('*');
const minuteValue = ref('*');
const hourValue = ref('*');
const dayValue = ref('*');
const monthValue = ref('*');
const yearValue = ref('*');
const weekValue = ref('*');

const tabs = [
  { key: 'second', label: '秒' },
  { key: 'minute', label: '分' },
  { key: 'hour', label: '时' },
  { key: 'day', label: '日' },
  { key: 'month', label: '月' },
  { key: 'week', label: '周' },
  { key: 'year', label: '年' },
];

const closeDialog = () => {
  emit("update:visible", false);
};

const handleConfirm = () => {
  emit("confirm", cronExpression.value);
  closeDialog();
};

const handleCancel = () => {
  emit("cancel");
  closeDialog();
};

// 计算完整的 cron 表达式
const cronExpression = computed(() => {
  return `${secondValue.value} ${minuteValue.value} ${hourValue.value} ${dayValue.value} ${monthValue.value} ${weekValue.value} ${yearValue.value}`;
});

// 添加复制功能
const copyToClipboard = async () => {
  try {
    await navigator.clipboard.writeText(cronExpression.value);
    // 这里可以添加一个复制成功的提示
  } catch (err) {
    // 处理复制失败的情况
  }
};

// 添加组件引用
const secondPickerRef = ref(null);
const minutePickerRef = ref(null);
const hourPickerRef = ref(null);
const dayPickerRef = ref(null);
const monthPickerRef = ref(null);
const weekPickerRef = ref(null);
const yearPickerRef = ref(null);

// 修改重置函数
const resetValues = () => {
  // 先重置各组件的选项
  secondPickerRef.value?.reset();
  minutePickerRef.value?.reset();
  hourPickerRef.value?.reset();
  dayPickerRef.value?.reset();
  monthPickerRef.value?.reset();
  weekPickerRef.value?.reset();
  yearPickerRef.value?.reset();
  
  // 再重置值
  secondValue.value = '*';
  minuteValue.value = '*';
  hourValue.value = '*';
  dayValue.value = '*';
  monthValue.value = '*';
  weekValue.value = '*';
  yearValue.value = '*';
};

// 修改解析 cron 表达式的方法
const parseCronExpression = (expression) => {
  if (!expression || expression === '点击选择') return;
  
  const parts = expression.split(' ');
  if (parts.length >= 7) {
    // 设置值
    secondValue.value = parts[0];
    minuteValue.value = parts[1];
    hourValue.value = parts[2];
    dayValue.value = parts[3];
    monthValue.value = parts[4];
    weekValue.value = parts[5];
    yearValue.value = parts[6];

    // 更新各个组件的内部状态
    nextTick(() => {
      secondPickerRef.value?.updateFromExpression(parts[0]);
      minutePickerRef.value?.updateFromExpression(parts[1]);
      hourPickerRef.value?.updateFromExpression(parts[2]);
      dayPickerRef.value?.updateFromExpression(parts[3]);
      monthPickerRef.value?.updateFromExpression(parts[4]);
      weekPickerRef.value?.updateFromExpression(parts[5]);
      yearPickerRef.value?.updateFromExpression(parts[6]);
    });
  }
};

// 监听弹窗显示状态变化
watch(() => props.visible, (newVal) => {
  if (newVal) {
    parseCronExpression(props.modelValue);
  }
});
</script>

<template>
  <transition name="dialog-fade" appear>
    <div v-if="visible" class="dialog-overlay" @click="closeDialog">
      <div class="dialog" @click.stop>
        <div class="dialog-header">
          <h3>{{ title }}</h3>
          <button class="close-button" @click="closeDialog">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="20"
              height="20"
              viewBox="0 0 20 20"
            >
              <path
                d="M15.898,4.045c-0.271-0.272-0.713-0.272-0.986,0l-4.71,4.711L5.493,4.045c-0.272-0.272-0.714-0.272-0.986,0s-0.272,0.714,0,0.986l4.709,4.711l-4.71,4.711c-0.272,0.271-0.272,0.713,0,0.986c0.136,0.136,0.314,0.203,0.492,0.203c0.179,0,0.357-0.067,0.493-0.203l4.711-4.711l4.71,4.711c0.137,0.136,0.314,0.203,0.494,0.203c0.178,0,0.355-0.067,0.492-0.203c0.273-0.273,0.273-0.715,0-0.986l-4.711-4.711l4.711-4.711C16.172,4.759,16.172,4.317,15.898,4.045z"
                fill="currentColor"
              />
            </svg>
          </button>
        </div>
        <div class="dialog-content">
          <!-- Tab 切换器 -->
          <div class="tabs">
            <div 
              v-for="tab in tabs" 
              :key="tab.key"
              class="tab-item"
              :class="{ active: activeTab === tab.key }"
              @click="activeTab = tab.key"
            >
              {{ tab.label }}
            </div>
          </div>
          <!-- Tab 内容区域 -->
          <div class="tab-content">
            <!-- 秒的配置内容 -->
            <div v-show="activeTab === 'second'">
              <SecondPicker ref="secondPickerRef" v-model:value="secondValue" />
            </div>

            <!-- 分的配置内容 -->
            <div v-show="activeTab === 'minute'">
              <MinutePicker ref="minutePickerRef" v-model:value="minuteValue" />
            </div>

            <!-- 时的配置内容 -->
            <div v-show="activeTab === 'hour'">
              <HourPicker ref="hourPickerRef" v-model:value="hourValue" />
            </div>

            <!-- 日的配置内容 -->
            <div v-show="activeTab === 'day'">
              <DayPicker ref="dayPickerRef" v-model:value="dayValue" />
            </div>

            <!-- 月的配置内容 -->
            <div v-show="activeTab === 'month'">
              <MonthPicker ref="monthPickerRef" v-model:value="monthValue" />
            </div>

            <!-- 周的配置内容 -->
            <div v-show="activeTab === 'week'">
              <WeekPicker ref="weekPickerRef" v-model:value="weekValue" />
            </div>

            <div v-show="activeTab === 'year'">
              <YearPicker ref="yearPickerRef" v-model:value="yearValue" />
            </div>
          </div>
        </div>
        <div class="cron-preview">
          <div class="preview-header">
            <span class="preview-label">表达式预览</span>
            <button class="copy-button" @click="copyToClipboard" title="复制到剪贴板">
              <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect>
                <path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path>
              </svg>
            </button>
          </div>
          <div class="expression-container">
            <div class="expression-item">
              <span class="expression-label">秒</span>
              <span class="expression-value">{{ secondValue }}</span>
            </div>
            <div class="expression-item">
              <span class="expression-label">分</span>
              <span class="expression-value">{{ minuteValue }}</span>
            </div>
            <div class="expression-item">
              <span class="expression-label">时</span>
              <span class="expression-value">{{ hourValue }}</span>
            </div>
            <div class="expression-item">
              <span class="expression-label">日</span>
              <span class="expression-value">{{ dayValue }}</span>
            </div>
            <div class="expression-item">
              <span class="expression-label">月</span>
              <span class="expression-value">{{ monthValue }}</span>
            </div>
            <div class="expression-item">
              <span class="expression-label">周</span>
              <span class="expression-value">{{ weekValue }}</span>
            </div>
            <div class="expression-item">
              <span class="expression-label">年</span>
              <span class="expression-value">{{ yearValue }}</span>
            </div>
          </div>
          <div class="complete-expression">
            {{ cronExpression }}
          </div>
        </div>
        <div class="dialog-footer">
          <div class="footer-left">
            <button class="dialog-button reset" @click="resetValues">
              <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M3 12a9 9 0 0 1 9-9 9.75 9.75 0 0 1 6.74 2.74L21 8"></path>
                <path d="M21 3v5h-5"></path>
                <path d="M21 12a9 9 0 0 1-9 9-9.75 9.75 0 0 1-6.74-2.74L3 16"></path>
                <path d="M3 21v-5h5"></path>
              </svg>
              <span>重置</span>
            </button>
          </div>
          <div class="footer-right">
            <button class="dialog-button cancel" @click="handleCancel">取消</button>
            <button class="dialog-button confirm" @click="handleConfirm">确定</button>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<style scoped>
/* 弹窗样式 */
.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(32, 33, 36, 0.4);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(4px);
  transition: backdrop-filter 0.2s ease;
}

.dialog {
  background: #fff;
  border-radius: 16px;
  width: 90%;
  max-width: 800px;
  min-height: 650px;
  height: 80vh;
  max-height: 650px;
  box-shadow: 0 8px 24px rgba(32, 33, 36, 0.16);
  transform-origin: center;
  transition: transform 0.2s ease;
  display: flex;
  flex-direction: column;
}

.dialog-header {
  padding: 20px 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-shrink: 0;
}

.dialog-header h3 {
  margin: 0;
  font-size: 20px;
  color: #202124;
  font-weight: 500;
  line-height: 1.4;
}

.close-button {
  width: 36px;
  height: 36px;
  border: none;
  background: transparent;
  color: #5f6368;
  cursor: pointer;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
  padding: 0;
  margin: -8px;
  outline: none;
  -webkit-tap-highlight-color: transparent;
}

.close-button svg {
  width: 20px;
  height: 20px;
}

.close-button:hover {
  background-color: #f1f3f4;
  color: #202124;
}

.dialog-content {
  padding: 0 24px;
  flex: 1;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  /* 添加硬件加速 */
  transform: translateZ(0);
  -webkit-transform: translateZ(0);
  will-change: transform;
}

.dialog-footer {
  padding: 16px 24px;
  display: flex;
  justify-content: space-between; /* 改为两端对齐 */
  border-top: 1px solid #f1f3f4;
  flex-shrink: 0;
}

.footer-left {
  display: flex;
  align-items: center;
}

.footer-right {
  display: flex;
  gap: 12px;
}

.dialog-button {
  min-width: 80px;
  padding: 0 16px;
  height: 36px;
  border: none;
  border-radius: 18px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
  outline: none;
  -webkit-tap-highlight-color: transparent;
  user-select: none;
}

.dialog-button.cancel {
  color: #1a73e8;
  background: transparent;
}

.dialog-button.cancel:hover {
  background: #f6fafe;
}

.dialog-button.confirm {
  color: #fff;
  background: #1a73e8;
}

.dialog-button.confirm:hover {
  background: #1557b0;
  box-shadow: 0 1px 3px rgba(26, 115, 232, 0.24);
}

/* 添加重置按钮样式 */
.dialog-button.reset {
  color: #5f6368;
  background: transparent;
  gap: 8px;
}

.dialog-button.reset svg {
  width: 16px;
  height: 16px;
}

.dialog-button.reset:hover {
  background: #f1f3f4;
  color: #202124;
}

/* 动画 */
.dialog-fade-enter-active,
.dialog-fade-leave-active {
  transition: opacity 0.2s ease, transform 0.2s ease;
}

.dialog-fade-enter-from,
.dialog-fade-leave-to {
  opacity: 0;
}

.dialog-fade-enter-from .dialog,
.dialog-fade-leave-to .dialog {
  transform: scale(0.5);
}

.dialog-fade-enter-to .dialog,
.dialog-fade-leave-from .dialog {
  transform: scale(1);
}

/* 添加 Tab 相关样式 */
.tabs {
  display: flex;
  border-bottom: 1px solid #f1f3f4;
  margin: 0 -24px;
  padding: 0 24px;
  overflow-x: auto;
  scrollbar-width: none;
  -ms-overflow-style: none;
  flex-shrink: 0;
}

.tabs::-webkit-scrollbar {
  display: none; /* Chrome, Safari, Opera */
}

.tab-item {
  padding: 12px 16px;
  font-size: 14px;
  color: #5f6368;
  cursor: pointer;
  border-bottom: 2px solid transparent;
  margin-bottom: -1px;
  white-space: nowrap;
  transition: all 0.2s;
}

.tab-item:hover {
  color: #1a73e8;
}

.tab-item.active {
  color: #1a73e8;
  border-bottom-color: #1a73e8;
}

.tab-content {
  flex: 1;
  overflow-y: auto;
  padding: 20px 0;
  max-height: calc(100% - 50px);
  /* 优化滚动行为 */
  -webkit-overflow-scrolling: touch;
  scroll-behavior: smooth;
  overscroll-behavior: contain;
  /* 添加硬件加速 */
  transform: translateZ(0);
  -webkit-transform: translateZ(0);
  will-change: transform;
}

/* 修改指定选择器的样式 */
.specify-seconds,
.specify-minutes,
.specify-hours,
.specify-days,
.specify-months,
.specify-weeks,
.specify-years {
  max-height: 300px;
  overflow-y: auto;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(80px, 1fr));
  gap: 8px;
  margin-top: 16px;
  padding: 16px;
  background: #f8f9fa;
  border-radius: 8px;
  /* 优化滚动行为 */
  -webkit-overflow-scrolling: touch;
  scroll-behavior: smooth;
  overscroll-behavior: contain;
  /* 添加硬件加速 */
  transform: translateZ(0);
  -webkit-transform: translateZ(0);
  will-change: transform;
}

/* 优化滚动条样式 */
.tab-content::-webkit-scrollbar,
.specify-seconds::-webkit-scrollbar,
.specify-minutes::-webkit-scrollbar,
.specify-hours::-webkit-scrollbar,
.specify-days::-webkit-scrollbar,
.specify-months::-webkit-scrollbar,
.specify-weeks::-webkit-scrollbar,
.specify-years::-webkit-scrollbar {
  width: 6px;  /* 减小滚动条宽度 */
}

.tab-content::-webkit-scrollbar-track,
.specify-seconds::-webkit-scrollbar-track,
.specify-minutes::-webkit-scrollbar-track,
.specify-hours::-webkit-scrollbar-track,
.specify-days::-webkit-scrollbar-track,
.specify-months::-webkit-scrollbar-track,
.specify-weeks::-webkit-scrollbar-track,
.specify-years::-webkit-scrollbar-track {
  background: transparent;
  border-radius: 3px;
}

.tab-content::-webkit-scrollbar-thumb,
.specify-seconds::-webkit-scrollbar-thumb,
.specify-minutes::-webkit-scrollbar-thumb,
.specify-hours::-webkit-scrollbar-thumb,
.specify-days::-webkit-scrollbar-thumb,
.specify-months::-webkit-scrollbar-thumb,
.specify-weeks::-webkit-scrollbar-thumb,
.specify-years::-webkit-scrollbar-thumb {
  background-color: rgba(218, 220, 224, 0.8);  /* 使用半透明背景 */
  border-radius: 3px;
  /* 添加过渡效果 */
  transition: background-color 0.2s ease;
}

.tab-content::-webkit-scrollbar-thumb:hover,
.specify-seconds::-webkit-scrollbar-thumb:hover,
.specify-minutes::-webkit-scrollbar-thumb:hover,
.specify-hours::-webkit-scrollbar-thumb:hover,
.specify-days::-webkit-scrollbar-thumb:hover,
.specify-months::-webkit-scrollbar-thumb:hover,
.specify-weeks::-webkit-scrollbar-thumb:hover,
.specify-years::-webkit-scrollbar-thumb:hover {
  background-color: rgba(189, 193, 198, 0.9);
}

.cron-preview {
  padding: 16px 24px;
  border-top: 1px solid #f1f3f4;
  background: #f8f9fa;
}

.preview-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.preview-label {
  font-size: 14px;
  font-weight: 500;
  color: #202124;
}

.copy-button {
  padding: 8px;
  background: transparent;
  border: none;
  border-radius: 4px;
  color: #5f6368;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
  outline: none;
  -webkit-tap-highlight-color: transparent;
}

.copy-button:hover {
  background: #e8f0fe;
  color: #1a73e8;
}

.expression-container {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-bottom: 16px;
  padding: 12px;
  background: #fff;
  border-radius: 8px;
  border: 1px solid #dadce0;
  word-break: break-word;
}

.expression-item {
  display: flex;
  align-items: center;
  gap: 8px;
}

.expression-label {
  font-size: 13px;
  color: #5f6368;
  min-width: 24px;
}

.expression-value {
  font-size: 14px;
  color: #202124;
  font-family: monospace;
  padding: 2px 6px;
  background: #f1f3f4;
  border-radius: 4px;
}

.complete-expression {
  font-family: monospace;
  font-size: 14px;
  padding: 12px;
  background: #fff;
  border: 1px solid #dadce0;
  border-radius: 8px;
  color: #202124;
  word-break: break-all;
}

/* 可以添加一个全局的按钮焦点样式 */
button:focus {
  outline: none;
}

/* 如果需要保持键盘访问性，可以只在鼠标操作时移除轮廓 */
button:focus:not(:focus-visible) {
  outline: none;
}

button:focus-visible {
  outline: 2px solid #1a73e8;
  outline-offset: 2px;
}
</style>
