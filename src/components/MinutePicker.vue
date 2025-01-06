<script setup>
import { ref, watch, computed } from 'vue';

const emit = defineEmits(['update:value']);

const minuteType = ref('every');
const cycleStart = ref(1);
const cycleEnd = ref(2);
const specifyMinutes = ref([0]);
const stepStart = ref(0);
const stepValue = ref(1);

// 生成0-59的数组
const minuteOptions = Array.from({ length: 60 }, (_, i) => ({
  label: i.toString().padStart(2, '0'),
  value: i
}));

// 监听变化并发出事件
const updateValue = () => {
  let value = '*';
  
  switch (minuteType.value) {
    case 'every':
      value = '*';
      break;
    case 'cycle':
      value = `${cycleStart.value}-${cycleEnd.value}`;
      break;
    case 'specify':
      value = specifyMinutes.value.sort((a, b) => a - b).join(',');
      break;
    case 'step':
      value = `${stepStart.value}/${stepValue.value}`;
      break;
  }
  
  emit('update:value', value);
};

// 监听所有可能的变化
watch([minuteType, cycleStart, cycleEnd, specifyMinutes, stepStart, stepValue], updateValue, { immediate: true });

watch(specifyMinutes, (newVal) => {
  if (newVal.length === 0) {
    specifyMinutes.value = [0];
  }
});

const reset = () => {
  minuteType.value = 'every';
  cycleStart.value = 1;
  cycleEnd.value = 2;
  specifyMinutes.value = [0];
  stepStart.value = 0;
  stepValue.value = 1;
};

// 添加从表达式更新状态的方法
const updateFromExpression = (expression) => {
  if (expression === '*') {
    minuteType.value = 'every';
  } else if (expression.includes('/')) {
    minuteType.value = 'step';
    const [start, step] = expression.split('/');
    stepStart.value = parseInt(start);
    stepValue.value = parseInt(step);
  } else if (expression.includes('-')) {
    minuteType.value = 'cycle';
    const [start, end] = expression.split('-');
    cycleStart.value = parseInt(start);
    cycleEnd.value = parseInt(end);
  } else {
    minuteType.value = 'specify';
    specifyMinutes.value = expression.split(',').map(v => parseInt(v));
  }
};

// 监听 cycleStart 的变化，确保 cycleEnd 不小于 cycleStart
watch(cycleStart, (newVal) => {
  if (cycleEnd.value <= newVal) {
    cycleEnd.value = newVal + 1;
  }
});

const cycleEndOptions = computed(() => {
  return Array.from({ length: 60 - cycleStart.value }, (_, i) => ({
    label: (cycleStart.value + i + 1).toString().padStart(2, '0'),
    value: cycleStart.value + i + 1
  }));
});

// 暴露方法给父组件
defineExpose({
  reset,
  updateFromExpression
});
</script>

<template>
  <div class="minute-config">
    <div class="radio-group">
      <!-- 每分钟 -->
      <label class="radio-item">
        <input type="radio" v-model="minuteType" value="every">
        <span class="radio-label">每分钟</span>
      </label>
      
      <!-- 周期 -->
      <label class="radio-item">
        <input type="radio" v-model="minuteType" value="cycle">
        <span class="radio-label">周期结束</span>
        <template v-if="minuteType === 'cycle'">
          <div class="cycle-picker">
            从第 
            <select v-model="cycleStart">
              <option v-for="i in 59" :key="i" :value="i">
                {{ i.toString().padStart(2, '0') }}
              </option>
            </select>
            分钟开始，到第
            <select v-model="cycleEnd">
              <option 
                v-for="option in cycleEndOptions" 
                :key="option.value" 
                :value="option.value"
              >
                {{ option.label }}
              </option>
            </select>
            分钟结束
          </div>
        </template>
      </label>

      <!-- 步长 -->
      <label class="radio-item">
        <input type="radio" v-model="minuteType" value="step">
        <span class="radio-label">周期执行</span>
        <template v-if="minuteType === 'step'">
          <div class="cycle-picker">
            从第
            <select v-model="stepStart">
              <option v-for="i in 60" :key="i-1" :value="i-1">{{ i-1 }}</option>
            </select>
            分钟开始，每
            <select v-model="stepValue">
              <option v-for="i in 60" :key="i-1" :value="i-1">{{ i-1 }}</option>
            </select>
            分钟执行一次
          </div>
        </template>
      </label>

      <!-- 指定时间 -->
      <label class="radio-item">
        <input type="radio" v-model="minuteType" value="specify">
        <span class="radio-label">指定</span>
      </label>
    </div>

    <!-- 指定时间的选择器 -->
    <div v-show="minuteType === 'specify'" class="specify-minutes">
      <label 
        v-for="option in minuteOptions" 
        :key="option.value" 
        class="specify-item"
      >
        <input 
          type="checkbox" 
          :value="option.value" 
          v-model="specifyMinutes"
        >
        <span>{{ option.label }}</span>
      </label>
    </div>
  </div>
</template>

<style scoped>
.minute-config {
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

.specify-minutes {
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