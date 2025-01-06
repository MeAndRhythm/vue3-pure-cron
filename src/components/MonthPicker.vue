<script setup>
import { ref, watch } from 'vue';

const emit = defineEmits(['update:value']);

const monthType = ref('every');
const cycleStart = ref(1);
const cycleEnd = ref(2);
const stepStart = ref(1);
const stepValue = ref(1);
const specifyMonths = ref([1]);

// 生成1-12的数组，并添加月份名称
const monthOptions = Array.from({ length: 12 }, (_, i) => ({
  label: (i + 1).toString().padStart(2, '0'),
  value: i + 1,
  name: ['一月', '二月', '三月', '四月', '五月', '六月', '七月', '八月', '九月', '十月', '十一月', '十二月'][i]
}));

// 监听变化并发出事件
const updateValue = () => {
  let value = '*';
  
  switch (monthType.value) {
    case 'every':
      value = '*';
      break;
    case 'cycle':
      value = `${cycleStart.value}-${cycleEnd.value}`;
      break;
    case 'specify':
      value = specifyMonths.value.sort((a, b) => a - b).join(',');
      break;
    case 'step':
      value = `${stepStart.value}/${stepValue.value}`;
      break;
  }
  
  emit('update:value', value);
};

// 监听所有可能的变化
watch([monthType, cycleStart, cycleEnd, specifyMonths, stepStart, stepValue], updateValue, { immediate: true });

watch(specifyMonths, (newVal) => {
  if (newVal.length === 0) {
    specifyMonths.value = [1];
  }
});

const reset = () => {
  monthType.value = 'every';
  cycleStart.value = 1;
  cycleEnd.value = 2;
  specifyMonths.value = [1];
  stepStart.value = 1;
  stepValue.value = 1;
};

const updateFromExpression = (expression) => {
  if (expression === '*') {
    monthType.value = 'every';
  } else if (expression.includes('/')) {
    monthType.value = 'step';
    const [start, step] = expression.split('/');
    stepStart.value = parseInt(start);
    stepValue.value = parseInt(step);
  } else if (expression.includes('-')) {
    monthType.value = 'cycle';
    const [start, end] = expression.split('-');
    cycleStart.value = parseInt(start);
    cycleEnd.value = parseInt(end);
  } else {
    monthType.value = 'specify';
    specifyMonths.value = expression.split(',').map(v => parseInt(v));
  }
};

defineExpose({
  reset,
  updateFromExpression
});
</script>

<template>
  <div class="month-config">
    <div class="radio-group">
      <!-- 每月 -->
      <label class="radio-item">
        <input type="radio" v-model="monthType" value="every">
        <span class="radio-label">每月</span>
      </label>
      
      <!-- 周期 -->
      <label class="radio-item">
        <input type="radio" v-model="monthType" value="cycle">
        <span class="radio-label">周期结束</span>
        <template v-if="monthType === 'cycle'">
          <div class="cycle-picker">
            从
            <select v-model="cycleStart">
              <option v-for="option in monthOptions" :key="option.value" :value="option.value">
                {{ option.name }}
              </option>
            </select>
            开始,到
            <select v-model="cycleEnd">
              <option v-for="option in monthOptions" :key="option.value" :value="option.value">
                {{ option.name }}
              </option>
            </select>
            结束
          </div>
        </template>
      </label>

      <!-- 步长 -->
      <label class="radio-item">
        <input type="radio" v-model="monthType" value="step">
        <span class="radio-label">执行</span>
        <template v-if="monthType === 'step'">
          <div class="cycle-picker">
            从
            <select v-model="stepStart">
              <option v-for="option in monthOptions" :key="option.value" :value="option.value">
                {{ option.name }}
              </option>
            </select>
            开始,每
            <select v-model="stepValue">
              <option v-for="i in 12" :key="i" :value="i">{{ i }}</option>
            </select>
            月执行一次
          </div>
        </template>
      </label>

      <!-- 指定 -->
      <label class="radio-item">
        <input type="radio" v-model="monthType" value="specify">
        <span class="radio-label">指定</span>
      </label>
    </div>

    <!-- 指定月份的选择器 -->
    <div v-show="monthType === 'specify'" class="specify-months">
      <label 
        v-for="option in monthOptions" 
        :key="option.value" 
        class="specify-item"
      >
        <input 
          type="checkbox" 
          :value="option.value" 
          v-model="specifyMonths"
        >
        <span>{{ option.name }}</span>
      </label>
    </div>
  </div>
</template>

<style scoped>
.month-config {
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

.specify-months {
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