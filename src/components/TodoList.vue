<script setup>
import { computed, ref, onMounted, onUnmounted, nextTick } from 'vue'
import TodoItem from './TodoItem.vue'

const props = defineProps({
  todos: {
    type: Array,
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

const isEmpty = computed(() => props.todos.length === 0)
const listRef = ref(null)
const visibleItems = ref(new Set())

let observer = null

onMounted(() => {
  nextTick(() => {
    setupIntersectionObserver()
  })
})

onUnmounted(() => {
  if (observer) {
    observer.disconnect()
  }
})

const setupIntersectionObserver = () => {
  observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        const id = entry.target.dataset.id
        if (entry.isIntersecting) {
          visibleItems.value.add(id)
          entry.target.classList.add('visible')
        }
      })
    },
    {
      root: null,
      rootMargin: '0px',
      threshold: 0.1
    }
  )

  observeItems()
}

const observeItems = () => {
  nextTick(() => {
    if (listRef.value) {
      const items = listRef.value.querySelectorAll('.todo-item')
      items.forEach((item) => {
        observer?.observe(item)
      })
    }
  })
}
</script>

<template>
  <div class="todo-list-container">
    <div ref="listRef" class="list-wrapper">
      <TransitionGroup
        name="list"
        tag="ul"
        class="todo-list"
        @after-enter="observeItems"
      >
        <TodoItem
          v-for="(todo, index) in todos"
          :key="todo.id"
          :todo="todo"
          :color="color"
          :always-show-time="alwaysShowTime"
          :data-id="String(todo.id)"
          :style="{ '--delay': `${index * 0.05}s` }"
          @toggle="emit('toggle', $event)"
          @delete="emit('delete', $event)"
          @edit="(id, text) => emit('edit', id, text)"
        />
      </TransitionGroup>
    </div>

    <Transition name="fade">
      <div v-if="isEmpty" class="empty-state">
        <div class="empty-icon" :style="{ color: color.primary }">
          <svg width="64" height="64" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
            <path d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2" stroke-linecap="round" stroke-linejoin="round"/>
            <path d="M9 12h6M9 16h6" stroke-linecap="round" stroke-linejoin="round" opacity="0.5"/>
          </svg>
        </div>
        <p class="empty-text">暂无待办事项</p>
        <p class="empty-subtext">添加一个新的待办，开始高效的一天</p>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
.todo-list-container {
  min-height: 200px;
}

.todo-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

/* 列表动画 */
.list-enter-active {
  animation: slideIn 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.list-leave-active {
  animation: slideOut 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.list-move {
  transition: transform 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(-30px) scale(0.9);
  }
  to {
    opacity: 1;
    transform: translateX(0) scale(1);
  }
}

@keyframes slideOut {
  from {
    opacity: 1;
    transform: translateX(0) scale(1);
  }
  to {
    opacity: 0;
    transform: translateX(30px) scale(0.9);
  }
}

/* 空状态 */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 60px 20px;
  text-align: center;
}

.empty-icon {
  opacity: 0.4;
  margin-bottom: 16px;
  animation: float 3s ease-in-out infinite;
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}

.empty-text {
  font-size: 18px;
  color: #333;
  margin: 0 0 8px;
  font-weight: 500;
}

.empty-subtext {
  font-size: 14px;
  color: #999;
  margin: 0;
}

/* 淡入淡出动画 */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
