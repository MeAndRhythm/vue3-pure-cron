<script setup>
import { ref, watch, computed } from 'vue';

const emit = defineEmits(['update:value']);

const secondType = ref('every');
const cycleStart = ref(1);
const cycleEnd = ref(2);
const specifySeconds = ref([0]);
const stepStart = ref(0);
const stepValue = ref(1);

// 生成0-59的数组
const secondOptions = Array.from({ length: 60 }, (_, i) => ({
  label: i.toString().padStart(2, '0'),
  value: i
}));

// 监听变化并发出事件
const updateValue = () => {
  let value = '*';
  
  switch (secondType.value) {
    case 'every':
      value = '*';
      break;
    case 'cycle':
      value = `${cycleStart.value}-${cycleEnd.value}`;
      break;
    case 'specify':
      value = specifySeconds.value.sort((a, b) => a - b).join(',');
      break;
    case 'step':
      value = `${stepStart.value}/${stepValue.value}`;
      break;
  }
  
  emit('update:value', value);
};

// 监听所有可能的变化
watch([secondType, cycleStart, cycleEnd, specifySeconds, stepStart, stepValue], updateValue, { immediate: true });

// 监听 specifySeconds 的变化，确保至少选中一个值
watch(specifySeconds, (newVal) => {
  if (newVal.length === 0) {
    specifySeconds.value = [0];  // 如果没有选中值，默认选中0
  }
});

// 监听 cycleStart 的变化，确保 cycleEnd 不小于 cycleStart
watch(cycleStart, (newVal) => {
  if (cycleEnd.value <= newVal) {
    cycleEnd.value = newVal + 1;
  }
});

// 计算结束值的可选范围
const cycleEndOptions = computed(() => {
  return Array.from({ length: 60 - cycleStart.value }, (_, i) => ({
    label: (cycleStart.value + i + 1).toString().padStart(2, '0'),
    value: cycleStart.value + i + 1
  }));
});

// 添加重置方法
const reset = () => {
  secondType.value = 'every';
  cycleStart.value = 1;
  cycleEnd.value = 2;
  specifySeconds.value = [0];
  stepStart.value = 0;
  stepValue.value = 1;
};

// 添加从表达式更新状态的方法
const updateFromExpression = (expression) => {
  if (expression === '*') {
    secondType.value = 'every';
  } else if (expression.includes('/')) {
    secondType.value = 'step';
    const [start, step] = expression.split('/');
    stepStart.value = parseInt(start);
    stepValue.value = parseInt(step);
  } else if (expression.includes('-')) {
    secondType.value = 'cycle';
    const [start, end] = expression.split('-');
    cycleStart.value = parseInt(start);
    cycleEnd.value = parseInt(end);
  } else {
    secondType.value = 'specify';
    specifySeconds.value = expression.split(',').map(v => parseInt(v));
  }
};

// 暴露方法给父组件
defineExpose({
  reset,
  updateFromExpression
});
</script>

<template>
  <div class="second-config">
    <div class="radio-group">
      <!-- 每秒 -->
      <label class="radio-item">
        <input type="radio" v-model="secondType" value="every">
        <span class="radio-label">每秒</span>
      </label>
      
      <!-- 周期 -->
      <label class="radio-item">
        <input type="radio" v-model="secondType" value="cycle">
        <span class="radio-label">周期结束</span>
        <template v-if="secondType === 'cycle'">
          <div class="cycle-picker">
            从第 
            <select v-model="cycleStart">
              <option v-for="i in 59" :key="i" :value="i">
                {{ i.toString().padStart(2, '0') }}
              </option>
            </select>
            秒开始，到第
            <select v-model="cycleEnd">
              <option 
                v-for="option in cycleEndOptions" 
                :key="option.value" 
                :value="option.value"
              >
                {{ option.label }}
              </option>
            </select>
            秒结束
          </div>
        </template>
      </label>

      <!-- 步长 -->
      <label class="radio-item">
        <input type="radio" v-model="secondType" value="step">
        <span class="radio-label">周期执行</span>
        <template v-if="secondType === 'step'">
          <div class="cycle-picker">
            从第
            <select v-model="stepStart">
              <option v-for="i in 60" :key="i-1" :value="i-1">{{ i-1 }}</option>
            </select>
            秒开始，每
            <select v-model="stepValue">
              <option v-for="i in 60" :key="i-1" :value="i-1">{{ i-1 }}</option>
            </select>
            秒执行一次
          </div>
        </template>
      </label>

      <!-- 指定时间 -->
      <label class="radio-item">
        <input type="radio" v-model="secondType" value="specify">
        <span class="radio-label">指定</span>
      </label>
    </div>

    <!-- 指定时间的选择器 -->
    <div v-show="secondType === 'specify'" class="specify-seconds">
      <label 
        v-for="option in secondOptions" 
        :key="option.value" 
        class="specify-item"
      >
        <input 
          type="checkbox" 
          :value="option.value" 
          v-model="specifySeconds"
        >
        <span>{{ option.label }}</span>
      </label>
    </div>
  </div>
</template>

<style scoped>
.second-config {
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
  flex-wrap: nowrap;
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
  flex-wrap: nowrap;
  gap: 8px;
  margin-left: 8px;
  color: #5f6368;
  font-size: 14px;
  white-space: nowrap;
}

.cycle-picker select {
  padding: 4px 8px;
  border: 1px solid #dadce0;
  border-radius: 4px;
  color: #202124;
  font-size: 14px;
  outline: none;
  transition: all 0.2s;
  width: auto;
  min-width: 60px;
}

.cycle-picker select:hover {
  border-color: #1a73e8;
}

.cycle-picker select:focus {
  border-color: #1a73e8;
  box-shadow: 0 0 0 1px rgba(26, 115, 232, 0.2);
}

.specify-seconds {
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