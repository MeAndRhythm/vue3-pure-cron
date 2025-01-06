<script setup>
import { ref, watch } from 'vue';

const emit = defineEmits(['update:value']);

const weekType = ref('every');
const cycleStart = ref(1);
const cycleEnd = ref(2);
const stepStart = ref(1);
const stepValue = ref(1);
const workDay = ref(1);
const specifyWeeks = ref([1]);

// 生成周选项
const weekOptions = [
  { label: '周日', value: 1 },
  { label: '周一', value: 2 },
  { label: '周二', value: 3 },
  { label: '周三', value: 4 },
  { label: '周四', value: 5 },
  { label: '周五', value: 6 },
  { label: '周六', value: 7 }
];

// 监听变化并发出事件
const updateValue = () => {
  let value = '*';
  
  switch (weekType.value) {
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
    case 'lastWeek':
      value = `${workDay.value}L`;
      break;
    case 'specify':
      value = specifyWeeks.value.sort((a, b) => a - b).join(',');
      break;
  }
  
  emit('update:value', value);
};

// 监听所有可能的变化
watch(
  [weekType, cycleStart, cycleEnd, stepStart, stepValue, workDay, specifyWeeks],
  updateValue,
  { immediate: true }
);

watch(specifyWeeks, (newVal) => {
  if (newVal.length === 0) {
    specifyWeeks.value = [1];
  }
});

const reset = () => {
  weekType.value = 'every';
  cycleStart.value = 1;
  cycleEnd.value = 2;
  stepStart.value = 1;
  stepValue.value = 1;
  workDay.value = 1;
  specifyWeeks.value = [1];
};

const updateFromExpression = (expression) => {
  if (expression === '*') {
    weekType.value = 'every';
  } else if (expression === '?') {
    weekType.value = 'none';
  } else if (expression.includes('L')) {
    weekType.value = 'lastWeek';
    workDay.value = parseInt(expression.replace('L', ''));
  } else if (expression.includes('/')) {
    weekType.value = 'step';
    const [start, step] = expression.split('/');
    stepStart.value = parseInt(start);
    stepValue.value = parseInt(step);
  } else if (expression.includes('-')) {
    weekType.value = 'cycle';
    const [start, end] = expression.split('-');
    cycleStart.value = parseInt(start);
    cycleEnd.value = parseInt(end);
  } else {
    weekType.value = 'specify';
    specifyWeeks.value = expression.split(',').map(v => parseInt(v));
  }
};

defineExpose({
  reset,
  updateFromExpression
});
</script>

<template>
  <div class="week-config">
    <div class="radio-group">
      <!-- 每周 -->
      <label class="radio-item">
        <input type="radio" v-model="weekType" value="every">
        <span class="radio-label">每周</span>
      </label>

      <!-- 不指定 -->
      <label class="radio-item">
        <input type="radio" v-model="weekType" value="none">
        <span class="radio-label">不指定</span>
      </label>
      
      <!-- 周期 -->
      <label class="radio-item">
        <input type="radio" v-model="weekType" value="cycle">
        <span class="radio-label">周期</span>
        <template v-if="weekType === 'cycle'">
          <div class="cycle-picker">
            从
            <select v-model="cycleStart">
              <option v-for="option in weekOptions" :key="option.value" :value="option.value">
                {{ option.label }}
              </option>
            </select>
            开始,到
            <select v-model="cycleEnd">
              <option v-for="option in weekOptions" :key="option.value" :value="option.value">
                {{ option.label }}
              </option>
            </select>
            结束
          </div>
        </template>
      </label>

      <!-- 步长 -->
      <label class="radio-item">
        <input type="radio" v-model="weekType" value="step">
        <span class="radio-label">按步执行</span>
        <template v-if="weekType === 'step'">
          <div class="cycle-picker">
            从
            <select v-model="stepStart">
              <option v-for="option in weekOptions" :key="option.value" :value="option.value">
                {{ option.label }}
              </option>
            </select>
            开始,每
            <select v-model="stepValue">
              <option v-for="option in weekOptions" :key="option.value" :value="option.value">
                {{ option.label }}
              </option>
            </select>
            执行一次
          </div>
        </template>
      </label>

      <!-- 最后一个 -->
      <label class="radio-item">
        <input type="radio" v-model="weekType" value="lastWeek">
        <span class="radio-label">本月最后一个</span>
        <template v-if="weekType === 'lastWeek'">
          <div class="cycle-picker">
            本月最后一个
            <select v-model="workDay">
              <option v-for="option in weekOptions" :key="option.value" :value="option.value">
                {{ option.label }}
              </option>
            </select>
          </div>
        </template>
      </label>

      <!-- 指定 -->
      <label class="radio-item">
        <input type="radio" v-model="weekType" value="specify">
        <span class="radio-label">指定</span>
      </label>
    </div>

    <!-- 指定周的选择器 -->
    <div v-show="weekType === 'specify'" class="specify-weeks">
      <label 
        v-for="option in weekOptions" 
        :key="option.value" 
        class="specify-item"
      >
        <input 
          type="checkbox" 
          :value="option.value" 
          v-model="specifyWeeks"
        >
        <span>{{ option.label }}</span>
      </label>
    </div>
  </div>
</template>

<style scoped>
.week-config {
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
  min-width: 80px;
}

.cycle-picker select:hover {
  border-color: #1a73e8;
}

.cycle-picker select:focus {
  border-color: #1a73e8;
  box-shadow: 0 0 0 1px rgba(26, 115, 232, 0.2);
}

.specify-weeks {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(80px, 1fr));
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