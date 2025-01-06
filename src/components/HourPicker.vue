<script setup>
import { ref, watch, computed } from 'vue';

const emit = defineEmits(['update:value']);

const hourType = ref('every');
const cycleStart = ref(0);
const cycleEnd = ref(1);
const specifyHours = ref([0]);
const stepStart = ref(0);
const stepValue = ref(1);

// 生成0-23的数组
const hourOptions = Array.from({ length: 24 }, (_, i) => ({
  label: i.toString().padStart(2, '0'),
  value: i
}));

// 监听变化并发出事件
const updateValue = () => {
  let value = '*';
  
  switch (hourType.value) {
    case 'every':
      value = '*';
      break;
    case 'cycle':
      value = `${cycleStart.value}-${cycleEnd.value}`;
      break;
    case 'specify':
      value = specifyHours.value.sort((a, b) => a - b).join(',');
      break;
    case 'step':
      value = `${stepStart.value}/${stepValue.value}`;
      break;
  }
  
  emit('update:value', value);
};

// 监听所有可能的变化
watch([hourType, cycleStart, cycleEnd, specifyHours, stepStart, stepValue], updateValue, { immediate: true });

watch(specifyHours, (newVal) => {
  if (newVal.length === 0) {
    specifyHours.value = [0];
  }
});

const reset = () => {
  hourType.value = 'every';
  cycleStart.value = 0;
  cycleEnd.value = 1;
  specifyHours.value = [0];
  stepStart.value = 0;
  stepValue.value = 1;
};

const updateFromExpression = (expression) => {
  if (expression === '*') {
    hourType.value = 'every';
  } else if (expression.includes('/')) {
    hourType.value = 'step';
    const [start, step] = expression.split('/');
    stepStart.value = parseInt(start);
    stepValue.value = parseInt(step);
  } else if (expression.includes('-')) {
    hourType.value = 'cycle';
    const [start, end] = expression.split('-');
    cycleStart.value = parseInt(start);
    cycleEnd.value = parseInt(end);
  } else {
    hourType.value = 'specify';
    specifyHours.value = expression.split(',').map(v => parseInt(v));
  }
};

watch(cycleStart, (newVal) => {
  if (cycleEnd.value <= newVal) {
    cycleEnd.value = newVal + 1;
  }
});

const cycleEndOptions = computed(() => {
  return Array.from({ length: 24 - cycleStart.value }, (_, i) => ({
    label: (cycleStart.value + i + 1).toString().padStart(2, '0'),
    value: cycleStart.value + i + 1
  }));
});

defineExpose({
  reset,
  updateFromExpression
});
</script>

<template>
  <div class="hour-config">
    <div class="radio-group">
      <!-- 每小时 -->
      <label class="radio-item">
        <input type="radio" v-model="hourType" value="every">
        <span class="radio-label">每小时</span>
      </label>
      
      <!-- 周期 -->
      <label class="radio-item">
        <input type="radio" v-model="hourType" value="cycle">
        <span class="radio-label">周期束</span>
        <template v-if="hourType === 'cycle'">
          <div class="cycle-picker">
            从第 
            <select v-model="cycleStart">
              <option v-for="i in 23" :key="i" :value="i">
                {{ i.toString().padStart(2, '0') }}
              </option>
            </select>
            时开始，到第
            <select v-model="cycleEnd">
              <option 
                v-for="option in cycleEndOptions" 
                :key="option.value" 
                :value="option.value"
              >
                {{ option.label }}
              </option>
            </select>
            时结束
          </div>
        </template>
      </label>

      <!-- 步长 -->
      <label class="radio-item">
        <input type="radio" v-model="hourType" value="step">
        <span class="radio-label">周期执行</span>
        <template v-if="hourType === 'step'">
          <div class="cycle-picker">
            从第
            <select v-model="stepStart">
              <option v-for="i in 24" :key="i-1" :value="i-1">{{ i-1 }}</option>
            </select>
            时开始，每
            <select v-model="stepValue">
              <option v-for="i in 24" :key="i-1" :value="i-1">{{ i-1 }}</option>
            </select>
            小时执行一次
          </div>
        </template>
      </label>

      <!-- 指定时间 -->
      <label class="radio-item">
        <input type="radio" v-model="hourType" value="specify">
        <span class="radio-label">指定</span>
      </label>
    </div>

    <!-- 指定时间的选择器 -->
    <div v-show="hourType === 'specify'" class="specify-hours">
      <label 
        v-for="option in hourOptions" 
        :key="option.value" 
        class="specify-item"
      >
        <input 
          type="checkbox" 
          :value="option.value" 
          v-model="specifyHours"
        >
        <span>{{ option.label }}</span>
      </label>
    </div>
  </div>
</template>

<style scoped>
.hour-config {
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

.specify-hours {
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