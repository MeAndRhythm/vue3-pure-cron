<script setup>
import { ref } from "vue";
import TimeDialog from "./components/TimeDialog.vue";

const isFocused = ref(false);
const isDialogVisible = ref(false);
const displayValue = ref("点击选择");

const handleClick = () => {
  isDialogVisible.value = true;
};

const handleConfirm = (value) => {
  displayValue.value = value;  // 更新显示值为 cron 表达式
};

const handleCancel = () => {
  // 可以保持原来的值不变，或者根据需要重置
};

const clearValue = (e) => {
  e.stopPropagation(); // 阻止事件冒泡，防止触发 input 的点击事件
  displayValue.value = "点击选择";
};
</script>

<template>
  <div class="input-container">
    <div 
      class="material-input" 
      :class="{ 'is-focused': isFocused }" 
      @click="handleClick"
    >
      <div class="fake-input">
        <span class="placeholder" :title="displayValue">{{ displayValue }}</span>
        <button 
          v-if="displayValue !== '点击选择'" 
          class="clear-button"
          @click="clearValue"
          title="清除"
        >
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <line x1="18" y1="6" x2="6" y2="18"></line>
            <line x1="6" y1="6" x2="18" y2="18"></line>
          </svg>
        </button>
      </div>
    </div>
    
    <TimeDialog 
      v-model:visible="isDialogVisible"
      :model-value="displayValue"
      title="选择时间"
      @confirm="handleConfirm"
      @cancel="handleCancel"
    />
  </div>
</template>

<style scoped>
.input-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100px;
  padding: 20px;
}

.material-input {
  position: relative;
  width: 300px;
  min-height: 40px;
  cursor: pointer;
  border: 1px solid #dadce0;
  border-radius: 8px;
  transition: all 0.15s ease;
  background: #fff;
  display: flex;
  align-items: center;
  padding: 0 8px;
}

.fake-input {
  flex: 1;
  height: 100%;
  display: flex;
  align-items: center;
  position: relative;
  padding: 0 4px;
  justify-content: space-between;
  min-width: 0;
}

.placeholder {
  color: #5f6368;
  font-size: 14px;
  font-weight: 500;
  user-select: none;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  flex: 1;
  margin-right: 8px;
}

.material-input:hover {
  border-color: #202124;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.material-input:active {
  border-color: #1a73e8;
  border-width: 2px;
}

.material-input.is-focused {
  border-color: #1a73e8;
  border-width: 2px;
  box-shadow: 0 1px 2px rgba(26, 115, 232, 0.1);
}

.clear-button {
  flex-shrink: 0;
  width: 24px;
  height: 24px;
  padding: 4px;
  border: none;
  background: transparent;
  color: #5f6368;
  cursor: pointer;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
  outline: none;
  -webkit-tap-highlight-color: transparent;
  margin-left: 4px;
}

.clear-button:hover {
  background: #f1f3f4;
  color: #202124;
}

.clear-button:active {
  background: #e8eaed;
}
</style>
