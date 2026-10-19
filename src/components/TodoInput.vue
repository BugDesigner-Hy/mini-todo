<script setup>
import { ref } from 'vue'

const props = defineProps({
  color: {
    type: Object,
    default: () => ({ primary: '#8B5A2B', gradient: ['#8B5A2B', '#A0522D'] })
  }
})

const emit = defineEmits(['add'])

const newTodo = ref('')
const isFocused = ref(false)

const handleSubmit = () => {
  if (newTodo.value.trim()) {
    emit('add', newTodo.value)
    newTodo.value = ''
  }
}

const handleKeydown = (e) => {
  if (e.key === 'Enter') {
    handleSubmit()
  }
}
</script>

<template>
  <div class="input-container" :class="{ focused: isFocused }">
    <div class="input-wrapper">
      <span class="input-icon" :style="{ color: color.primary }">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="12" cy="12" r="10" opacity="0.3"/>
          <line x1="12" y1="8" x2="12" y2="16"/>
          <line x1="8" y1="12" x2="16" y2="12"/>
        </svg>
      </span>
      <input
        v-model="newTodo"
        type="text"
        placeholder="添加新的待办事项..."
        @keydown="handleKeydown"
        @focus="isFocused = true"
        @blur="isFocused = false"
        class="todo-input"
      />
      <button
        v-if="newTodo.trim()"
        @click="handleSubmit"
        class="add-btn"
        :style="{ background: `linear-gradient(135deg, ${color.gradient[0]}, ${color.gradient[1]})` }"
        :disabled="!newTodo.trim()"
      >
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <polyline points="9 18 15 12 9 6"/>
        </svg>
      </button>
    </div>
  </div>
</template>

<style scoped>
.input-container {
  margin-bottom: 20px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.input-wrapper {
  display: flex;
  align-items: center;
  background: rgba(0, 0, 0, 0.03);
  border: 2px solid rgba(0, 0, 0, 0.08);
  border-radius: 16px;
  padding: 4px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.input-container.focused .input-wrapper {
  border-color: var(--primary);
  background: rgba(0, 0, 0, 0.05);
  box-shadow: 0 0 0 4px rgba(0, 0, 0, 0.05);
}

.input-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0 12px;
  opacity: 0.6;
  transition: opacity 0.3s ease;
}

.input-container.focused .input-icon {
  opacity: 1;
}

.todo-input {
  flex: 1;
  border: none;
  background: transparent;
  padding: 14px 12px;
  font-size: 16px;
  color: #333;
  outline: none;
}

.todo-input::placeholder {
  color: #999;
}

.add-btn {
  width: 44px;
  height: 44px;
  border: none;
  color: #fff;
  border-radius: 12px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: scale(0.8);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

.add-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.add-btn:active {
  transform: scale(0.95);
}

/* 移动端适配 */
@media (max-width: 640px) {
  .input-wrapper {
    border-radius: 12px;
  }

  .todo-input {
    padding: 12px 10px;
    font-size: 15px;
  }

  .add-btn {
    width: 40px;
    height: 40px;
    border-radius: 10px;
  }
}
</style>
