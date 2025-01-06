<script setup>
import { ref, watch } from 'vue';

const emit = defineEmits(['update:value']);

const yearType = ref('every');
const cycleStart = ref(2024);
const cycleEnd = ref(2025);
const stepStart = ref(2024);
const stepValue = ref(1);
const specifyYears = ref([new Date().getFullYear()]);

// 生成年份选项（从当前年份开始，往后20年）
const currentYear = new Date().getFullYear();
const yearOptions = Array.from({ length: 20 }, (_, i) => ({
  label: (currentYear + i).toString(),
  value: currentYear + i,
  name: (currentYear + i).toString() + '年'
}));

// 监听变化并发出事件
const updateValue = () => {
  let value = '*';
  
  switch (yearType.value) {
    case 'every':
      value = '*';
      break;
    case 'cycle':
      value = `${cycleStart.value}-${cycleEnd.value}`;
      break;
    case 'specify':
      value = specifyYears.value.sort((a, b) => a - b).join(',');
      break;
    case 'step':
      value = `${stepStart.value}/${stepValue.value}`;
      break;
  }
  
  emit('update:value', value);
};

// 监听所有可能的变化
watch([yearType, cycleStart, cycleEnd, specifyYears, stepStart, stepValue], updateValue, { immediate: true });

watch(specifyYears, (newVal) => {
  if (newVal.length === 0) {
    specifyYears.value = [currentYear];
  }
});

const reset = () => {
  yearType.value = 'every';
  cycleStart.value = currentYear;
  cycleEnd.value = currentYear + 1;
  specifyYears.value = [currentYear];
  stepStart.value = currentYear;
  stepValue.value = 1;
};

const updateFromExpression = (expression) => {
  if (expression === '*') {
    yearType.value = 'every';
  } else if (expression.includes('/')) {
    yearType.value = 'step';
    const [start, step] = expression.split('/');
    stepStart.value = parseInt(start);
    stepValue.value = parseInt(step);
  } else if (expression.includes('-')) {
    yearType.value = 'cycle';
    const [start, end] = expression.split('-');
    cycleStart.value = parseInt(start);
    cycleEnd.value = parseInt(end);
  } else {
    yearType.value = 'specify';
    specifyYears.value = expression.split(',').map(v => parseInt(v));
  }
};

defineExpose({
  reset,
  updateFromExpression
});
</script>

<template>
  <div class="year-config">
    <div class="radio-group">
      <!-- 每年 -->
      <label class="radio-item">
        <input type="radio" v-model="yearType" value="every">
        <span class="radio-label">每年</span>
      </label>
      
      <!-- 周期 -->
      <label class="radio-item">
        <input type="radio" v-model="yearType" value="cycle">
        <span class="radio-label">周期结束</span>
        <template v-if="yearType === 'cycle'">
          <div class="cycle-picker">
            从
            <select v-model="cycleStart">
              <option v-for="option in yearOptions" :key="option.value" :value="option.value">
                {{ option.name }}
              </option>
            </select>
            开始,到
            <select v-model="cycleEnd">
              <option v-for="option in yearOptions" :key="option.value" :value="option.value">
                {{ option.name }}
              </option>
            </select>
            结束
          </div>
        </template>
      </label>

      <!-- 步长 -->
      <label class="radio-item">
        <input type="radio" v-model="yearType" value="step">
        <span class="radio-label">周期执行</span>
        <template v-if="yearType === 'step'">
          <div class="cycle-picker">
            从
            <select v-model="stepStart">
              <option v-for="option in yearOptions" :key="option.value" :value="option.value">
                {{ option.name }}
              </option>
            </select>
            开始,每
            <select v-model="stepValue">
              <option v-for="option in yearOptions" :key="option.value" :value="option.value">
                {{ option.name }}
              </option>
            </select>
            年执行一次
          </div>
        </template>
      </label>

      <!-- 指定 -->
      <label class="radio-item">
        <input type="radio" v-model="yearType" value="specify">
        <span class="radio-label">指定</span>
      </label>
    </div>

    <!-- 指定年份的选择器 -->
    <div v-show="yearType === 'specify'" class="specify-years">
      <label 
        v-for="option in yearOptions" 
        :key="option.value" 
        class="specify-item"
      >
        <input 
          type="checkbox" 
          :value="option.value" 
          v-model="specifyYears"
        >
        <span>{{ option.name }}</span>
      </label>
    </div>
  </div>
</template>

<style scoped>
.year-config {
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
  min-width: 100px;
}

.cycle-picker select:hover {
  border-color: #1a73e8;
}

.cycle-picker select:focus {
  border-color: #1a73e8;
  box-shadow: 0 0 0 1px rgba(26, 115, 232, 0.2);
}

.specify-years {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
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