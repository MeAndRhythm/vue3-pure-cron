<script setup>
import { ref, watch } from 'vue';

const emit = defineEmits(['update:value']);

const dayType = ref('every');
const cycleStart = ref(1);
const cycleEnd = ref(2);
const stepStart = ref(1);
const stepValue = ref(1);
const workDay = ref(1);
const specifyDays = ref([1]);

// 生成1-31的数组
const dayOptions = Array.from({ length: 31 }, (_, i) => ({
  label: (i + 1).toString().padStart(2, '0'),
  value: i + 1
}));

// 监听变化并发出事件
const updateValue = () => {
  let value = '*';
  
  switch (dayType.value) {
    case 'every':
      value = '*';
      break;
    case 'none':
      value = '?';
      break;
    case 'cycle':
      value = `${cycleStart.value}-${cycleEnd.value}`;
      break;
    case 'step':
      value = `${stepStart.value}/${stepValue.value}`;
      break;
    case 'workDay':
      value = `${workDay.value}W`;
      break;
    case 'lastDay':
      value = 'L';
      break;
    case 'specify':
      value = specifyDays.value.sort((a, b) => a - b).join(',');
      break;
  }
  
  emit('update:value', value);
};

// 监听所有可能的变化
watch(
  [dayType, cycleStart, cycleEnd, stepStart, stepValue, workDay, specifyDays],
  updateValue,
  { immediate: true }
);

watch(specifyDays, (newVal) => {
  if (newVal.length === 0) {
    specifyDays.value = [1];
  }
});

const reset = () => {
  dayType.value = 'every';
  cycleStart.value = 1;
  cycleEnd.value = 2;
  stepStart.value = 1;
  stepValue.value = 1;
  workDay.value = 1;
  specifyDays.value = [1];
};

const updateFromExpression = (expression) => {
  if (expression === '*') {
    dayType.value = 'every';
  } else if (expression === '?') {
    dayType.value = 'none';
  } else if (expression.includes('W')) {
    dayType.value = 'workDay';
    workDay.value = parseInt(expression.replace('W', ''));
  } else if (expression === 'L') {
    dayType.value = 'lastDay';
  } else if (expression.includes('/')) {
    dayType.value = 'step';
    const [start, step] = expression.split('/');
    stepStart.value = parseInt(start);
    stepValue.value = parseInt(step);
  } else if (expression.includes('-')) {
    dayType.value = 'cycle';
    const [start, end] = expression.split('-');
    cycleStart.value = parseInt(start);
    cycleEnd.value = parseInt(end);
  } else {
    dayType.value = 'specify';
    specifyDays.value = expression.split(',').map(v => parseInt(v));
  }
};

defineExpose({
  reset,
  updateFromExpression
});
</script>

<template>
  <div class="day-config">
    <div class="radio-group">
      <!-- 每日 -->
      <label class="radio-item">
        <input type="radio" v-model="dayType" value="every">
        <span class="radio-label">每日</span>
      </label>

      <!-- 不指定 -->
      <label class="radio-item">
        <input type="radio" v-model="dayType" value="none">
        <span class="radio-label">不指定</span>
      </label>
      
      <!-- 周期 -->
      <label class="radio-item">
        <input type="radio" v-model="dayType" value="cycle">
        <span class="radio-label">周期</span>
        <template v-if="dayType === 'cycle'">
          <div class="cycle-picker">
            从
            <select v-model="cycleStart">
              <option v-for="i in 31" :key="i" :value="i">{{ i }}</option>
            </select>
            号开始,到
            <select v-model="cycleEnd">
              <option v-for="i in 31" :key="i" :value="i">{{ i }}</option>
            </select>
            号结束
          </div>
        </template>
      </label>

      <!-- 步长 -->
      <label class="radio-item">
        <input type="radio" v-model="dayType" value="step">
        <span class="radio-label">按步执行</span>
        <template v-if="dayType === 'step'">
          <div class="cycle-picker">
            从
            <select v-model="stepStart">
              <option v-for="i in 31" :key="i" :value="i">{{ i }}</option>
            </select>
            号开始,每
            <select v-model="stepValue">
              <option v-for="i in 31" :key="i" :value="i">{{ i }}</option>
            </select>
            天执行一次
          </div>
        </template>
      </label>

      <!-- 工作日 -->
      <label class="radio-item">
        <input type="radio" v-model="dayType" value="workDay">
        <span class="radio-label">最近工作日</span>
        <template v-if="dayType === 'workDay'">
          <div class="cycle-picker">
            每月
            <select v-model="workDay">
              <option v-for="i in 31" :key="i" :value="i">{{ i }}</option>
            </select>
            号最近的工作日
          </div>
        </template>
      </label>

      <!-- 最后一天 -->
      <label class="radio-item">
        <input type="radio" v-model="dayType" value="lastDay">
        <span class="radio-label">本月最后一天</span>
      </label>

      <!-- 指定 -->
      <label class="radio-item">
        <input type="radio" v-model="dayType" value="specify">
        <span class="radio-label">指定</span>
      </label>
    </div>

    <!-- 指定日期的选择器 -->
    <div v-show="dayType === 'specify'" class="specify-days">
      <label 
        v-for="option in dayOptions" 
        :key="option.value" 
        class="specify-item"
      >
        <input 
          type="checkbox" 
          :value="option.value" 
          v-model="specifyDays"
        >
        <span>{{ option.label }}</span>
      </label>
    </div>
  </div>
</template>

<style scoped>
.day-config {
  padding: 16px 0;
}

.radio-group {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.radio-item {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}

.radio-label {
  color: #202124;
  font-size: 14px;
}

.cycle-picker {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  margin-left: 8px;
  color: #5f6368;
  font-size: 14px;
}

.cycle-picker select {
  padding: 4px 8px;
  border: 1px solid #dadce0;
  border-radius: 4px;
  color: #202124;
  font-size: 14px;
  outline: none;
  transition: all 0.2s;
}

.cycle-picker select:hover {
  border-color: #1a73e8;
}

.cycle-picker select:focus {
  border-color: #1a73e8;
  box-shadow: 0 0 0 1px rgba(26, 115, 232, 0.2);
}

.specify-days {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(60px, 1fr));
  gap: 8px;
  margin-top: 16px;
  padding: 16px;
  background: #f8f9fa;
  border-radius: 8px;
}

.specify-item {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 4px;
  cursor: pointer;
  font-size: 14px;
  color: #5f6368;
}

.specify-item:hover {
  color: #1a73e8;
}

input[type="radio"],
input[type="checkbox"] {
  accent-color: #1a73e8;
  cursor: pointer;
  margin: 0;
}
</style> 