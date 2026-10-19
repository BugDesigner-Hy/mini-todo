<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import TodoList from './components/TodoList.vue'
import TodoInput from './components/TodoInput.vue'

const colorSchemes = [
  { name: '赭石', primary: '#8B5A2B', secondary: '#A0522D', accent: '#CD853F', bg: '#F5EBE0', gradient: ['#8B5A2B', '#A0522D'] },
  { name: '朱砂', primary: '#C0403F', secondary: '#E03F3F', accent: '#FF6B6B', bg: '#FFF0F0', gradient: ['#C0403F', '#E03F3F'] },
  { name: '青碧', primary: '#2E8B57', secondary: '#4A9E8E', accent: '#66CDAA', bg: '#F0FFF0', gradient: ['#2E8B57', '#4A9E8E'] },
  { name: '黛蓝', primary: '#2A5CAA', secondary: '#4169E1', accent: '#6495ED', bg: '#F0F8FF', gradient: ['#2A5CAA', '#4169E1'] },
  { name: '藤黄', primary: '#DAA520', secondary: '#F2A93B', accent: '#FFD700', bg: '#FFFAF0', gradient: ['#DAA520', '#F2A93B'] },
  { name: '胭脂', primary: '#9B2335', secondary: '#C41E3A', accent: '#DC143C', bg: '#FFF5F5', gradient: ['#9B2335', '#C41E3A'] },
  { name: '石青', primary: '#1E4D6B', secondary: '#2F6F8F', accent: '#4682B4', bg: '#F0F8FF', gradient: ['#1E4D6B', '#2F6F8F'] },
  { name: '藕荷', primary: '#8B668B', secondary: '#BA55D3', accent: '#DDA0DD', bg: '#FFF0FF', gradient: ['#8B668B', '#BA55D3'] },
  { name: '松花', primary: '#556B2F', secondary: '#6B8E23', accent: '#9ACD32', bg: '#F5FFF0', gradient: ['#556B2F', '#6B8E23'] },
  { name: '秋香', primary: '#B8860B', secondary: '#DAA520', accent: '#F0C050', bg: '#FFFEF0', gradient: ['#B8860B', '#DAA520'] }
]

const currentColorIndex = ref(0)
const currentColor = computed(() => colorSchemes[currentColorIndex.value])

const todos = ref([])
const filter = ref('active')
const showSettings = ref(false)
const randomColorOnAdd = ref(false)
const alwaysShowTime = ref(false)

const slogan = ref('掌控时间，掌控人生')

onMounted(() => {
  const saved = localStorage.getItem('mi-todo')
  if (saved) {
    try { todos.value = JSON.parse(saved) } catch (e) {}
  }
  const savedColor = localStorage.getItem('mi-todo-color')
  if (savedColor) currentColorIndex.value = parseInt(savedColor) || 0
  const savedRandomColor = localStorage.getItem('mi-todo-random-color')
  if (savedRandomColor) randomColorOnAdd.value = savedRandomColor === 'true'
  const savedAlwaysShowTime = localStorage.getItem('mi-todo-always-show-time')
  if (savedAlwaysShowTime) alwaysShowTime.value = savedAlwaysShowTime === 'true'
  const savedSlogan = localStorage.getItem('mi-todo-slogan')
  if (savedSlogan) slogan.value = savedSlogan
})

watch(todos, (v) => localStorage.setItem('mi-todo', JSON.stringify(v)), { deep: true })
watch(currentColorIndex, (v) => localStorage.setItem('mi-todo-color', v.toString()))
watch(randomColorOnAdd, (v) => localStorage.setItem('mi-todo-random-color', v.toString()))
watch(alwaysShowTime, (v) => localStorage.setItem('mi-todo-always-show-time', v.toString()))
watch(slogan, (v) => localStorage.setItem('mi-todo-slogan', v))

const filteredTodos = computed(() => {
  if (filter.value === 'active') return todos.value.filter(t => !t.completed)
  if (filter.value === 'completed') return todos.value.filter(t => t.completed)
  return todos.value
})

const stats = computed(() => ({
  total: todos.value.length,
  active: todos.value.filter(t => !t.completed).length,
  completed: todos.value.filter(t => t.completed).length
}))

const addTodo = (text) => {
  if (text.trim()) {
    todos.value.unshift({ id: Date.now(), text: text.trim(), completed: false, createdAt: new Date().toISOString() })
    if (randomColorOnAdd.value) {
      let i = Math.floor(Math.random() * colorSchemes.length)
      while (i === currentColorIndex.value && colorSchemes.length > 1) i = Math.floor(Math.random() * colorSchemes.length)
      currentColorIndex.value = i
    }
  }
}

const toggleTodo = (id) => {
  const todo = todos.value.find(t => t.id === id)
  if (todo) {
    todo.completed = !todo.completed
    todo.completedAt = todo.completed ? new Date().toISOString() : null
  }
}

const deleteTodo = (id) => {
  const i = todos.value.findIndex(t => t.id === id)
  if (i > -1) todos.value.splice(i, 1)
}

const editTodo = (id, newText) => {
  const todo = todos.value.find(t => t.id === id)
  if (todo && newText.trim()) todo.text = newText.trim()
}

const clearCompleted = () => { todos.value = todos.value.filter(t => !t.completed) }

const clearAll = () => {
  if (confirm('确定要清除所有待办事项吗？此操作不可恢复！')) {
    todos.value = []
    showSettings.value = false
  }
}
</script>

<template>
  <div class="app-container" :style="{ '--primary': currentColor.primary, '--secondary': currentColor.secondary, '--accent': currentColor.accent, '--bg': currentColor.bg }">
    <div class="todo-app">
      <header class="app-header" :style="{ background: `linear-gradient(135deg, ${currentColor.gradient[0]} 0%, ${currentColor.gradient[1]} 100%)` }">
        <div class="header-content">
          <div class="logo">
            <span class="logo-icon">威</span>
            <h1>威力待办</h1>
          </div>
          <p class="subtitle">{{ slogan }}</p>
        </div>
        <button class="settings-btn" @click="showSettings = !showSettings" title="设置">
          <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="12" cy="12" r="3"/>
            <path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"/>
          </svg>
        </button>
      </header>

      <div class="fixed-section">
        <Transition name="slide">
          <div v-if="showSettings" class="settings-panel">
            <div class="settings-content">
              <div class="settings-section">
                <h3>个性签名</h3>
                <input
                  v-model="slogan"
                  type="text"
                  class="slogan-input"
                  placeholder="输入你的个性签名..."
                  maxlength="20"
                />
              </div>
              <div class="settings-section">
                <h3>主题颜色</h3>
                <div class="color-options">
                  <button
                    v-for="(scheme, index) in colorSchemes"
                    :key="index"
                    class="color-option"
                    :class="{ active: currentColorIndex === index }"
                    :style="{ background: `linear-gradient(135deg, ${scheme.gradient[0]}, ${scheme.gradient[1]})` }"
                    @click="currentColorIndex = index"
                    :title="scheme.name"
                  >
                    <span v-if="currentColorIndex === index" class="check-mark">✓</span>
                  </button>
                </div>
                <p class="current-color-name">{{ currentColor.name }}</p>
              </div>
              <div class="settings-section">
                <h3>颜色模式</h3>
                <label class="toggle-option">
                  <span class="toggle-label">新增待办颜色随机变化</span>
                  <div class="toggle-switch" :class="{ active: randomColorOnAdd }" @click="randomColorOnAdd = !randomColorOnAdd">
                    <div class="toggle-thumb"></div>
                  </div>
                </label>
              </div>
              <div class="settings-section">
                <h3>时间显示</h3>
                <label class="toggle-option">
                  <span class="toggle-label">始终显示时间</span>
                  <div class="toggle-switch" :class="{ active: alwaysShowTime }" @click="alwaysShowTime = !alwaysShowTime">
                    <div class="toggle-thumb"></div>
                  </div>
                </label>
              </div>
              <div class="settings-section">
                <h3>数据管理</h3>
                <button class="danger-btn" @click="clearAll">
                  <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                    <polyline points="3 6 5 6 21 6"/>
                    <path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"/>
                  </svg>
                  一键清除所有记录
                </button>
              </div>
            </div>
          </div>
        </Transition>

        <div class="input-section">
          <TodoInput @add="addTodo" :color="currentColor" />
        </div>

        <div class="filter-tabs">
          <div class="filter-tabs-inner">
            <div
              class="filter-indicator"
              :style="{
                transform: `translateX(${['active', 'completed', 'all'].indexOf(filter) * 100}%)`,
                background: `linear-gradient(135deg, ${currentColor.gradient[0]}, ${currentColor.gradient[1]})`
              }"
            ></div>
            <button
              v-for="f in ['active', 'completed', 'all']"
              :key="f"
              :class="['filter-btn', { active: filter === f }]"
              @click="filter = f"
            >
              {{ f === 'active' ? '待办' : f === 'completed' ? '已办' : '全部' }}
              <span class="count" v-if="f === 'all'">{{ stats.total }}</span>
              <span class="count" v-else-if="f === 'active'">{{ stats.active }}</span>
              <span class="count" v-else>{{ stats.completed }}</span>
            </button>
          </div>
        </div>
      </div>

      <main class="scrollable-section">
        <TodoList
          :todos="filteredTodos"
          :color="currentColor"
          :always-show-time="alwaysShowTime"
          @toggle="toggleTodo"
          @delete="deleteTodo"
          @edit="editTodo"
        />
      </main>

      <footer class="app-footer" v-if="stats.total > 0">
        <div class="stats">
          <span>{{ stats.active }} 项待办</span>
        </div>
        <button
          v-if="stats.completed > 0"
          class="clear-btn"
          @click="clearCompleted"
        >
          清除已完成
        </button>
      </footer>
    </div>
  </div>
</template>

<style scoped>
.app-container {
  min-height: 100vh;
  height: 100%;
  background: linear-gradient(135deg, var(--bg) 0%, #fff 50%, var(--bg) 100%);
  padding: 20px;
  box-sizing: border-box;
  transition: all 0.5s ease;
}

.todo-app {
  max-width: 500px;
  height: calc(100vh - 40px);
  margin: 0 auto;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  box-shadow:
    0 20px 60px rgba(0, 0, 0, 0.1),
    0 8px 24px rgba(0, 0, 0, 0.08);
  overflow: hidden;
  backdrop-filter: blur(10px);
  display: flex;
  flex-direction: column;
}

.app-header {
  padding: 24px 20px;
  text-align: center;
  position: relative;
  overflow: hidden;
  transition: background 0.5s ease;
  flex-shrink: 0;
}

.app-header::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(255,255,255,0.15) 0%, transparent 60%);
  animation: shimmer 8s infinite linear;
}

@keyframes shimmer {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.header-content { position: relative; z-index: 1; }

.logo {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  margin-bottom: 8px;
}

.logo-icon {
  width: 40px;
  height: 40px;
  background: rgba(255, 255, 255, 0.25);
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 22px;
  color: #fff;
  font-weight: bold;
  backdrop-filter: blur(10px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.app-header h1 {
  margin: 0;
  font-size: 24px;
  font-weight: 700;
  color: #fff;
  letter-spacing: 2px;
  text-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.subtitle {
  margin: 0;
  font-size: 11px;
  color: rgba(255, 255, 255, 0.85);
  letter-spacing: 2px;
  margin-top: 2px;
}

.settings-btn {
  position: absolute;
  top: 14px;
  right: 14px;
  width: 40px;
  height: 40px;
  background: rgba(255, 255, 255, 0.2);
  border: none;
  border-radius: 12px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
  z-index: 10;
}

.settings-btn:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: rotate(90deg);
}

.fixed-section {
  flex-shrink: 0;
  background: rgba(255, 255, 255, 0.95);
  z-index: 5;
}

.input-section { padding: 16px 20px 8px; }

.settings-panel {
  background: rgba(255, 255, 255, 0.98);
  border-bottom: 1px solid rgba(0, 0, 0, 0.06);
  backdrop-filter: blur(10px);
}

.settings-content { padding: 16px 20px; }

.settings-section { margin-bottom: 16px; }
.settings-section:last-child { margin-bottom: 0; }

.settings-section h3 {
  font-size: 13px;
  font-weight: 600;
  color: var(--primary);
  margin: 0 0 10px;
  letter-spacing: 1px;
}

.slogan-input {
  width: 100%;
  padding: 10px 14px;
  border: 2px solid rgba(0, 0, 0, 0.08);
  border-radius: 10px;
  font-size: 14px;
  color: #333;
  background: rgba(0, 0, 0, 0.02);
  outline: none;
  transition: border-color 0.3s ease;
  box-sizing: border-box;
}

.slogan-input:focus {
  border-color: var(--primary);
}

.color-options {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.color-option {
  width: 32px;
  height: 32px;
  border-radius: 8px;
  border: 2px solid transparent;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.color-option:hover {
  transform: scale(1.1);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.color-option.active {
  border-color: #fff;
  box-shadow: 0 0 0 2px var(--primary);
}

.check-mark { color: #fff; font-size: 12px; font-weight: bold; }

.current-color-name {
  font-size: 12px;
  color: var(--primary);
  margin: 8px 0 0;
  opacity: 0.7;
}

.danger-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 16px;
  background: linear-gradient(135deg, #C41E3A 0%, #DC143C 100%);
  color: #fff;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  font-size: 13px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.danger-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(196, 30, 58, 0.3);
}

.toggle-option {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px 0;
}

.toggle-label { font-size: 13px; color: #666; }

.toggle-switch {
  width: 44px;
  height: 24px;
  background: rgba(0, 0, 0, 0.15);
  border-radius: 12px;
  cursor: pointer;
  position: relative;
  transition: background 0.3s ease;
}

.toggle-switch.active { background: var(--primary); }

.toggle-thumb {
  width: 20px;
  height: 20px;
  background: #fff;
  border-radius: 50%;
  position: absolute;
  top: 2px;
  left: 2px;
  transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.toggle-switch.active .toggle-thumb { transform: translateX(20px); }

.slide-enter-active, .slide-leave-active { transition: all 0.3s ease; }
.slide-enter-from, .slide-leave-to { opacity: 0; transform: translateY(-10px); }

.filter-tabs {
  display: flex;
  padding: 8px 20px 16px;
  justify-content: center;
}

.filter-tabs-inner {
  display: flex;
  position: relative;
  background: rgba(0, 0, 0, 0.05);
  border-radius: 20px;
  padding: 4px;
}

.filter-indicator {
  position: absolute;
  top: 4px;
  left: 4px;
  width: calc(33.333% - 2.67px);
  height: calc(100% - 8px);
  border-radius: 16px;
  transition: transform 0.4s cubic-bezier(0.4, 0, 0.2, 1), background 0.3s ease;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}

.filter-btn {
  position: relative;
  z-index: 1;
  padding: 8px 16px;
  border: none;
  background: transparent;
  color: var(--primary);
  border-radius: 16px;
  cursor: pointer;
  font-size: 13px;
  font-weight: 500;
  transition: color 0.3s ease;
  display: flex;
  align-items: center;
  gap: 6px;
  min-width: 70px;
  justify-content: center;
}

.filter-btn.active { color: #fff; }

.count {
  background: rgba(255, 255, 255, 0.25);
  padding: 2px 7px;
  border-radius: 10px;
  font-size: 11px;
}

.filter-btn:not(.active) .count { background: rgba(0, 0, 0, 0.1); }

.scrollable-section {
  flex: 1;
  overflow-y: auto;
  overflow-x: hidden;
  padding: 0 20px;
  scroll-behavior: smooth;
}

.scrollable-section::-webkit-scrollbar { width: 6px; }
.scrollable-section::-webkit-scrollbar-track { background: transparent; }
.scrollable-section::-webkit-scrollbar-thumb { background: var(--primary); border-radius: 3px; opacity: 0.3; }
.scrollable-section::-webkit-scrollbar-thumb:hover { opacity: 0.5; }

.app-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 14px 20px;
  border-top: 1px solid rgba(0, 0, 0, 0.06);
  background: rgba(255, 255, 255, 0.95);
  flex-shrink: 0;
}

.stats { font-size: 13px; color: var(--primary); opacity: 0.7; }

.clear-btn {
  padding: 8px 14px;
  border: none;
  background: transparent;
  color: var(--accent);
  cursor: pointer;
  font-size: 13px;
  border-radius: 8px;
  transition: all 0.3s ease;
}

.clear-btn:hover {
  background: rgba(0, 0, 0, 0.05);
  color: var(--primary);
}

@media (max-width: 640px) {
  .app-container {
    padding: 0;
    min-height: 100%;
    height: 100%;
  }

  .todo-app {
    max-width: 100%;
    max-height: none;
    height: 100vh;
    border-radius: 0;
    box-shadow: none;
  }

  .app-header { padding: 20px 16px; }
  .input-section { padding: 12px 16px 6px; }
  .filter-tabs { padding: 6px 16px 12px; }
  .scrollable-section { padding: 0 16px; }
  .app-footer { padding: 12px 16px; }
}
</style>
