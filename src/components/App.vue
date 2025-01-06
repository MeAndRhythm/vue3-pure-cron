<script setup>
import { ref } from 'vue';
import TimeDialog from './components/TimeDialog.vue';

const dialogVisible = ref(false);
const cronValue = ref(''); // 存储 cron 表达式

const handleConfirm = (value) => {
  cronValue.value = value; // 更新 fake-input 的值
};
</script>

<template>
  <div class="container">
    <div class="fake-input" @click="dialogVisible = true">
      {{ cronValue || '请选择定时规则' }}
    </div>
    
    <TimeDialog 
      v-model:visible="dialogVisible"
      title="选择时间"
      @confirm="handleConfirm"
    />
  </div>
</template>

<style scoped>
.container {
  padding: 20px;
}

.fake-input {
  width: 100%;
  max-width: 500px;
  padding: 8px 12px;
  border: 1px solid #dadce0;
  border-radius: 4px;
  font-size: 14px;
  color: #202124;
  cursor: pointer;
  background: #fff;
  min-height: 40px;
  display: flex;
  align-items: center;
  transition: all 0.2s;
}

.fake-input:hover {
  border-color: #1a73e8;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

.fake-input:empty::before {
  content: '请选择定时规则';
  color: #5f6368;
}
</style> 