<template>
  <div class="app">
    <aside class="sidebar" :class="{ collapsed: isCollapsed }">
      <div class="sidebar-logo">
        <h1>{{ t('nav.companyName') }}</h1>
        <span class="subtitle">{{ t('nav.subtitle') }}</span>
      </div>
      <nav class="sidebar-nav">
        <router-link to="/">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/></svg>
          <span class="nav-label">{{ t('nav.overview') }}</span>
        </router-link>
        <router-link to="/inventory">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><path d="M3.27 6.96L12 12.01l8.73-5.05"/><path d="M12 22.08V12"/></svg>
          <span class="nav-label">{{ t('nav.inventory') }}</span>
        </router-link>
        <router-link to="/orders">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M6 2L3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4z"/><line x1="3" y1="6" x2="21" y2="6"/><path d="M16 10a4 4 0 0 1-8 0"/></svg>
          <span class="nav-label">{{ t('nav.orders') }}</span>
        </router-link>
        <router-link to="/spending">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="9"/><path d="M12 7v10M9 10h4.5a1.5 1.5 0 0 1 0 3H10a1.5 1.5 0 0 0 0 3h5"/></svg>
          <span class="nav-label">{{ t('nav.finance') }}</span>
        </router-link>
        <router-link to="/demand">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="22 7 13.5 15.5 8.5 10.5 2 17"/><polyline points="16 7 22 7 22 13"/></svg>
          <span class="nav-label">{{ t('nav.demandForecast') }}</span>
        </router-link>
        <router-link to="/reports">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="8" y1="13" x2="16" y2="13"/><line x1="8" y1="17" x2="16" y2="17"/></svg>
          <span class="nav-label">Reports</span>
        </router-link>
      </nav>
      <button class="sidebar-toggle" @click="toggleSidebar" :aria-label="isCollapsed ? 'Expand sidebar' : 'Collapse sidebar'">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline :points="isCollapsed ? '9 18 15 12 9 6' : '15 18 9 12 15 6'"/></svg>
      </button>
      <div class="sidebar-footer">
        <LanguageSwitcher />
        <ProfileMenu @show-profile-details="showProfileDetails = true" @show-tasks="showTasks = true" />
      </div>
    </aside>
    <div class="main-shell">
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, computed, watch } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    // Collapsed state persists across sessions; viewport <1024px auto-collapses on first mount
    // but the user can override — we don't re-check on resize to avoid fighting manual toggles
    const isCollapsed = ref(localStorage.getItem('sidebarCollapsed') === 'true')
    watch(isCollapsed, (val) => localStorage.setItem('sidebarCollapsed', String(val)))
    const toggleSidebar = () => { isCollapsed.value = !isCollapsed.value }

    onMounted(() => {
      loadTasks()
      // Only auto-collapse if nothing stored yet — respects prior user choice
      if (localStorage.getItem('sidebarCollapsed') === null && window.matchMedia('(max-width: 1024px)').matches) {
        isCollapsed.value = true
      }
    })

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      isCollapsed,
      toggleSidebar
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Sans:wght@400;500;600&family=IBM+Plex+Mono:wght@400;500&display=swap');

:root {
  --sidebar-w: 240px;
  --font-sans: 'IBM Plex Sans', system-ui, sans-serif;
  --font-mono: 'IBM Plex Mono', ui-monospace, 'SF Mono', Menlo, monospace;
  --bg-main: #fafafa;
  --bg-card: #ffffff;
  --bg-sidebar: #0f172a;
  --border: #e5e5e5;
  --text: #0a0a0a;
  --text-muted: #737373;
  --text-sidebar: #a3a3a3;
  --text-sidebar-active: #ffffff;
  --signal-green: #10b981;
  --signal-amber: #f59e0b;
  --signal-red: #ef4444;
  --signal-blue: #3b82f6;
}

* { margin: 0; padding: 0; box-sizing: border-box; }

body {
  font-family: var(--font-sans);
  font-size: 14px;
  background: var(--bg-main);
  color: var(--text);
  -webkit-font-smoothing: antialiased;
  font-feature-settings: 'tnum';
}

.app { display: flex; min-height: 100vh; }

/* ─── SIDEBAR ─────────────────────────────────────────────────── */
.sidebar {
  position: fixed;
  left: 0; top: 0; bottom: 0;
  width: var(--sidebar-w);
  background: var(--bg-sidebar);
  display: flex;
  flex-direction: column;
  border-right: 1px solid rgba(255,255,255,0.06);
  z-index: 100;
}

.sidebar-logo {
  padding: 20px;
  border-bottom: 1px solid rgba(255,255,255,0.06);
}
.sidebar-logo h1 {
  font-size: 15px;
  font-weight: 600;
  color: #fff;
  letter-spacing: -0.01em;
}
.sidebar-logo .subtitle {
  font-size: 11px;
  color: var(--text-sidebar);
  font-family: var(--font-mono);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  display: block;
  margin-top: 2px;
}

.sidebar-nav {
  display: flex;
  flex-direction: column;
  padding: 8px;
  gap: 1px;
}
.sidebar-nav a {
  height: 36px;
  padding: 0 12px;
  display: flex;
  align-items: center;
  gap: 10px;
  color: var(--text-sidebar);
  text-decoration: none;
  font-size: 13px;
  font-weight: 500;
  border-radius: 4px;
  transition: background-color 120ms ease, color 120ms ease;
  border-left: 2px solid transparent;
}
.sidebar-nav a:hover {
  background: rgba(255,255,255,0.04);
  color: #d4d4d4;
}
.sidebar-nav a.router-link-active {
  background: rgba(255,255,255,0.06);
  color: var(--text-sidebar-active);
  border-left-color: var(--text-sidebar-active);
}

.nav-icon { width: 18px; height: 18px; flex-shrink: 0; }

.sidebar-toggle {
  margin: 8px; margin-top: auto;
  width: calc(100% - 16px); height: 32px;
  background: transparent; border: 1px solid rgba(255,255,255,0.08); border-radius: 4px;
  color: var(--text-sidebar); cursor: pointer;
  display: flex; align-items: center; justify-content: center;
  transition: background-color 120ms ease, color 120ms ease;
}
.sidebar-toggle:hover { background: rgba(255,255,255,0.04); color: #d4d4d4; }
.sidebar-toggle svg { width: 16px; height: 16px; }

.sidebar-footer {
  margin-top: 0;
  padding: 12px;
  border-top: 1px solid rgba(255,255,255,0.06);
  display: flex;
  flex-direction: column;
  gap: 8px;
}

/* ─── COLLAPSED STATE ─────────────────────────────────────────── */
.sidebar.collapsed { width: 60px; }
.sidebar.collapsed ~ .main-shell { margin-left: 60px; }
.sidebar.collapsed .nav-label,
.sidebar.collapsed .sidebar-logo h1,
.sidebar.collapsed .sidebar-logo .subtitle { display: none; }
.sidebar.collapsed .sidebar-nav a { justify-content: center; padding: 0; }
.sidebar.collapsed .sidebar-logo { padding: 16px 0; text-align: center; }
.sidebar.collapsed .sidebar-footer { padding: 8px 4px; align-items: center; }
.sidebar.collapsed .sidebar-footer > * { max-width: 100%; overflow: hidden; }

/* ─── TRANSITIONS ─────────────────────────────────────────────── */
.sidebar, .main-shell { transition: width 160ms ease, margin-left 160ms ease; }

/* ─── MAIN SHELL ───────────────────────────────────────────────────── */
.main-shell {
  margin-left: var(--sidebar-w);
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}
.main-content {
  flex: 1;
  padding: 24px 28px;
}

/* ─── PAGE HEADER ──────────────────────────────────────────────────── */
.page-header { margin-bottom: 20px; }
.page-header h2 {
  font-size: 20px;
  font-weight: 600;
  letter-spacing: -0.01em;
}
.page-header p {
  color: var(--text-muted);
  font-size: 13px;
  margin-top: 2px;
}

/* ─── STAT CARDS ──────────────────────────────────────────────────── */
.stats-grid, .kpi-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 12px;
  margin-bottom: 20px;
}
.stat-card, .kpi-card {
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 4px;
  padding: 16px;
}
.stat-label, .kpi-label {
  font-family: var(--font-mono);
  font-size: 10px;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--text-muted);
  margin-bottom: 6px;
  display: block;
}
.stat-value, .kpi-value {
  font-family: var(--font-mono);
  font-size: 28px;
  font-weight: 500;
  line-height: 1;
  letter-spacing: -0.02em;
  font-variant-numeric: tabular-nums;
}
.stat-card.warning .stat-value { color: var(--signal-amber); }
.stat-card.success .stat-value { color: var(--signal-green); }
.stat-card.danger .stat-value { color: var(--signal-red); }
.stat-card.info .stat-value { color: var(--signal-blue); }

/* ─── GENERAL CARD ─────────────────────────────────────────────────── */
.card {
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 4px;
  padding: 0;
  margin-bottom: 16px;
}
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 14px 16px;
  border-bottom: 1px solid var(--border);
}
.card-title {
  font-size: 13px;
  font-weight: 600;
  letter-spacing: -0.01em;
}

/* ─── TABLE ───────────────────────────────────────────────────────── */
.table-container { overflow-x: auto; }
table { width: 100%; border-collapse: collapse; }
thead { border-bottom: 1px solid var(--border); }
th {
  text-align: left;
  padding: 10px 16px;
  font-family: var(--font-mono);
  font-size: 10px;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--text-muted);
}
td {
  padding: 10px 16px;
  border-top: 1px solid var(--border);
  font-size: 13px;
  font-variant-numeric: tabular-nums;
}
.num, td strong {
  font-family: var(--font-mono);
  font-variant-numeric: tabular-nums;
}
tbody tr { transition: background-color 100ms ease; }
tbody tr:hover { background: #fafafa; }

/* ─── BADGES ───────────────────────────────────────────────────────── */
.badge {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-family: var(--font-mono);
  font-size: 11px;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  background: transparent;
  padding: 0;
}
.badge::before {
  content: '';
  width: 6px;
  height: 6px;
  border-radius: 50%;
  flex-shrink: 0;
}
.badge.success::before, .badge.increasing::before { background: var(--signal-green); }
.badge.warning::before, .badge.medium::before { background: var(--signal-amber); }
.badge.danger::before, .badge.high::before, .badge.decreasing::before { background: var(--signal-red); }
.badge.info::before, .badge.low::before, .badge.stable::before { background: var(--signal-blue); }
.badge.success, .badge.increasing { color: #047857; }
.badge.warning, .badge.medium { color: #b45309; }
.badge.danger, .badge.high, .badge.decreasing { color: #b91c1c; }
.badge.info, .badge.low, .badge.stable { color: #1d4ed8; }

/* ─── LOADING / ERROR ────────────────────────────────────────────────── */
.loading {
  text-align: center;
  padding: 40px;
  color: var(--text-muted);
  font-family: var(--font-mono);
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 0.08em;
}
.error {
  border: 1px solid var(--signal-red);
  border-left-width: 3px;
  background: #fef2f2;
  color: #991b1b;
  padding: 12px 14px;
  border-radius: 4px;
  margin: 12px 0;
  font-size: 13px;
}
</style>
