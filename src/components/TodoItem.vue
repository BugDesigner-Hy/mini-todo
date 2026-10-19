<script setup>
import { ref, nextTick, onMounted, computed } from 'vue'

const props = defineProps({
  todo: {
    type: Object,
    required: true
  },
  color: {
    type: Object,
    default: () => ({ primary: '#8B5A2B', gradient: ['#8B5A2B', '#A0522D'] })
  },
  alwaysShowTime: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['toggle', 'delete', 'edit'])

const isEditing = ref(false)
const editText = ref('')
const inputRef = ref(null)
const itemRef = ref(null)
const isVisible = ref(false)
const showTime = ref(false)

onMounted(() => {
  setTimeout(() => {
    isVisible.value = true
  }, 50)
})

const formatTime = (isoString) => {
  if (!isoString) return ''
  const date = new Date(isoString)
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  const hours = String(date.getHours()).padStart(2, '0')
  const minutes = String(date.getMinutes()).padStart(2, '0')
  const seconds = String(date.getSeconds()).padStart(2, '0')
  return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`
}

const toggleTime = () => {
  showTime.value = !showTime.value
}

const startEdit = () => {
  isEditing.value = true
  editText.value = props.todo.text
  nextTick(() => {
    inputRef.value?.focus()
  })
}

const finishEdit = () => {
  if (editText.value.trim()) {
    emit('edit', props.todo.id, editText.value)
  }
  isEditing.value = false
}

const cancelEdit = () => {
  isEditing.value = false
  editText.value = ''
}

const handleKeydown = (e) => {
  if (e.key === 'Enter') {
    finishEdit()
  } else if (e.key === 'Escape') {
    cancelEdit()
  }
}
</script>

<template>
  <li
    ref="itemRef"
    class="todo-item"
    :class="{ completed: todo.completed, editing: isEditing, visible: isVisible, expanded: showTime }"
  >
    <div class="todo-content" @click="toggleTime">
      <button
        class="checkbox"
        :class="{ checked: todo.completed }"
        :style="todo.completed ? { background: `linear-gradient(135deg, ${color.gradient[0]}, ${color.gradient[1]})`, borderColor: 'transparent' } : { borderColor: color.primary }"
        @click.stop="emit('toggle', todo.id)"
      >
        <Transition name="check">
          <svg v-if="todo.completed" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3">
            <polyline points="20 6 9 17 4 12"/>
          </svg>
        </Transition>
      </button>

      <div class="todo-text-wrapper" @dblclick.stop="startEdit">
        <template v-if="!isEditing">
          <span class="todo-text">{{ todo.text }}</span>
        </template>
        <template v-else>
          <input
            ref="inputRef"
            v-model="editText"
            type="text"
            class="edit-input"
            @blur="finishEdit"
            @keydown="handleKeydown"
            @click.stop
          />
        </template>
      </div>

      <div class="todo-actions">
        <button
          v-if="!isEditing"
          class="action-btn edit-btn"
          @click.stop="startEdit"
          title="编辑"
        >
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M11 4H4a2 2 0 00-2 2v14a2 2 0 002 2h14a2 2 0 002-2v-7"/>
            <path d="M18.5 2.5a2.121 2.121 0 013 3L12 15l-4 1 1-4 9.5-9.5z"/>
          </svg>
        </button>
        <button
          class="action-btn delete-btn"
          @click.stop="emit('delete', todo.id)"
          title="删除"
        >
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <polyline points="3 6 5 6 21 6"/>
            <path d="M19 6v14a2 2 0 01-2 2H7a2 2 0 01-2-2V6m3 0V4a2 2 0 012-2h4a2 2 0 012 2v2"/>
            <line x1="10" y1="11" x2="10" y2="17"/>
            <line x1="14" y1="11" x2="14" y2="17"/>
          </svg>
        </button>
      </div>
    </div>
    <Transition name="slide-time">
      <div v-if="alwaysShowTime || showTime" class="time-info">
        <span v-if="todo.completed && todo.completedAt" class="time-text">
          <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="12" cy="12" r="10"/>
            <polyline points="12 6 12 12 16 14"/>
          </svg>
          完成于 {{ formatTime(todo.completedAt) }}
        </span>
        <span v-else class="time-text">
          <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="12" cy="12" r="10"/>
            <polyline points="12 6 12 12 16 14"/>
          </svg>
          创建于 {{ formatTime(todo.createdAt) }}
        </span>
      </div>
    </Transition>
  </li>
</template>

<style scoped>
.todo-item {
  background: rgba(0, 0, 0, 0.02);
  border-radius: 12px;
  margin-bottom: 8px;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  overflow: hidden;
  opacity: 0;
  transform: translateY(20px) scale(0.95);
}

.todo-item.visible {
  opacity: 1;
  transform: translateY(0) scale(1);
}

.todo-item:hover {
  background: rgba(0, 0, 0, 0.04);
}

.todo-item.expanded {
  background: rgba(0, 0, 0, 0.04);
}

.todo-item.completed {
  background: rgba(0, 0, 0, 0.01);
}

.todo-content {
  display: flex;
  align-items: center;
  padding: 16px;
  gap: 12px;
  cursor: pointer;
}

.checkbox {
  width: 24px;
  height: 24px;
  border: 2px solid;
  border-radius: 50%;
  background: transparent;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  flex-shrink: 0;
}

.checkbox:hover {
  opacity: 0.8;
  transform: scale(1.1);
}

.checkbox svg {
  color: #fff;
}

/* 勾选动画 */
.check-enter-active {
  animation: checkIn 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.check-leave-active {
  animation: checkOut 0.2s cubic-bezier(0.4, 0, 0.2, 1);
}

@keyframes checkIn {
  from {
    opacity: 0;
    transform: scale(0) rotate(-180deg);
  }
  to {
    opacity: 1;
    transform: scale(1) rotate(0deg);
  }
}

@keyframes checkOut {
  from {
    opacity: 1;
    transform: scale(1) rotate(0deg);
  }
  to {
    opacity: 0;
    transform: scale(0) rotate(180deg);
  }
}

.todo-text-wrapper {
  flex: 1;
  min-width: 0;
}

.todo-text {
  color: #333;
  font-size: 15px;
  line-height: 1.5;
  word-break: break-word;
  transition: all 0.3s ease;
}

.todo-item.completed .todo-text {
  color: #999;
  text-decoration: line-through;
}

.edit-input {
  width: 100%;
  border: none;
  background: rgba(0, 0, 0, 0.05);
  padding: 8px 12px;
  border-radius: 8px;
  font-size: 15px;
  color: #333;
  outline: none;
  transition: all 0.3s ease;
}

.edit-input:focus {
  background: rgba(0, 0, 0, 0.08);
  box-shadow: 0 0 0 2px var(--primary);
}

.todo-actions {
  display: flex;
  gap: 8px;
  opacity: 0;
  transform: translateX(10px);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.todo-item:hover .todo-actions {
  opacity: 1;
  transform: translateX(0);
}

.action-btn {
  width: 32px;
  height: 32px;
  border: none;
  background: transparent;
  border-radius: 8px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #999;
  transition: all 0.3s ease;
}

.action-btn:hover {
  background: rgba(0, 0, 0, 0.08);
}

.edit-btn:hover {
  color: var(--primary);
}

.delete-btn:hover {
  color: #C41E3A;
  background: rgba(196, 30, 58, 0.1);
}

/* 时间信息 */
.time-info {
  padding: 0 16px 12px 52px;
}

.time-text {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 12px;
  color: #999;
}

.time-text svg {
  opacity: 0.6;
}

/* 时间展开动画 */
.slide-time-enter-active {
  animation: slideTimeIn 0.3s ease;
}

.slide-time-leave-active {
  animation: slideTimeOut 0.2s ease;
}

@keyframes slideTimeIn {
  from {
    opacity: 0;
    max-height: 0;
    padding-bottom: 0;
  }
  to {
    opacity: 1;
    max-height: 40px;
    padding-bottom: 12px;
  }
}

@keyframes slideTimeOut {
  from {
    opacity: 1;
    max-height: 40px;
    padding-bottom: 12px;
  }
  to {
    opacity: 0;
    max-height: 0;
    padding-bottom: 0;
  }
}

/* 移动端适配 */
@media (max-width: 640px) {
  .todo-content {
    padding: 14px 12px;
  }

  .todo-actions {
    opacity: 1;
    transform: translateX(0);
  }

  .action-btn {
    width: 28px;
    height: 28px;
  }

  .time-info {
    padding: 0 12px 10px 48px;
  }
}
</style>
