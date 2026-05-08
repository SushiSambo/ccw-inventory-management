<template>
  <div class="app">
    <header class="top-nav">
      <div class="nav-container">
        <div class="logo">
          <span class="logo-mark"></span>
          <h1>{{ t('nav.companyName') }}</h1>
          <span class="subtitle">{{ t('nav.subtitle') }}</span>
        </div>
        <nav class="nav-tabs">
          <router-link to="/" :class="{ active: $route.path === '/' }">
            {{ t('nav.overview') }}
          </router-link>
          <router-link to="/inventory" :class="{ active: $route.path === '/inventory' }">
            {{ t('nav.inventory') }}
          </router-link>
          <router-link to="/orders" :class="{ active: $route.path === '/orders' }">
            {{ t('nav.orders') }}
          </router-link>
          <router-link to="/spending" :class="{ active: $route.path === '/spending' }">
            {{ t('nav.finance') }}
          </router-link>
          <router-link to="/demand" :class="{ active: $route.path === '/demand' }">
            {{ t('nav.demandForecast') }}
          </router-link>
          <router-link to="/reports" :class="{ active: $route.path === '/reports' }">
            Reports
          </router-link>
        </nav>
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </header>
    <FilterBar />
    <main class="main-content">
      <router-view />
    </main>

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
import { ref, onMounted, computed } from 'vue'
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

    onMounted(loadTasks)

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: #faf7f2;
  color: #1a1309;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.app {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.main-content {
  flex: 1;
  max-width: 1600px;
  width: 100%;
  margin: 0 auto;
  padding: 1.5rem 2rem;
  background: #faf7f2;
}

.page-header {
  margin-bottom: 1.5rem;
}

.page-header h2 {
  font-size: 1.875rem;
  font-weight: 700;
  color: #1a1309;
  margin-bottom: 0.375rem;
  letter-spacing: -0.025em;
}

.page-header p {
  color: #6b5c4e;
  font-size: 0.938rem;
}

/* ── Stats ─────────────────────────────────────────────────────────────── */

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.25rem;
  margin-bottom: 1.5rem;
}

.stat-card {
  background: #fffdf9;
  padding: 1.5rem 1.5rem 1.5rem 1.25rem;
  border-radius: 14px;
  border: 1px solid #e8e0d5;
  border-left-width: 3px;
  border-left-color: #e8e0d5;
  box-shadow: 0 1px 4px rgba(26, 19, 9, 0.07);
  transition: all 0.2s ease;
}

.stat-card:hover {
  box-shadow: 0 6px 20px rgba(26, 19, 9, 0.1);
  border-color: #d4c9bb;
  border-left-color: inherit;
}

.stat-card.warning {
  border-left-color: #f59e0b;
}

.stat-card.success {
  border-left-color: #10b981;
}

.stat-card.danger {
  border-left-color: #ef4444;
}

.stat-card.info {
  border-left-color: #d97757;
}

.stat-label {
  color: #6b5c4e;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.625rem;
}

.stat-value {
  font-size: 2.25rem;
  font-weight: 700;
  color: #1a1309;
  letter-spacing: -0.025em;
}

.stat-card.warning .stat-value {
  color: #ea580c;
}

.stat-card.success .stat-value {
  color: #059669;
}

.stat-card.danger .stat-value {
  color: #dc2626;
}

.stat-card.info .stat-value {
  color: #d97757;
}

/* ── Cards ──────────────────────────────────────────────────────────────── */

.card {
  background: #fffdf9;
  border-radius: 14px;
  padding: 1.25rem;
  border: 1px solid #e8e0d5;
  box-shadow: 0 1px 4px rgba(26, 19, 9, 0.07);
  margin-bottom: 1.25rem;
  transition: box-shadow 0.2s ease, border-color 0.2s ease;
}

.card:hover {
  border-color: #d4c9bb;
  box-shadow: 0 6px 20px rgba(26, 19, 9, 0.1);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.875rem;
  border-bottom: 1px solid #e8e0d5;
}

.card-title {
  font-size: 1.125rem;
  font-weight: 700;
  color: #1a1309;
  letter-spacing: -0.025em;
}

/* ── Tables ─────────────────────────────────────────────────────────────── */

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: #faf7f2;
  border-top: 1px solid #e8e0d5;
  border-bottom: 1px solid #e8e0d5;
}

th {
  text-align: left;
  padding: 0.625rem 0.75rem;
  font-weight: 600;
  color: #6b5c4e;
  font-size: 0.688rem;
  text-transform: uppercase;
  letter-spacing: 0.06em;
}

td {
  padding: 0.5rem 0.75rem;
  border-top: 1px solid #e8e0d5;
  color: #1a1309;
  font-size: 0.875rem;
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: #faf7f2;
}

/* ── Badges ─────────────────────────────────────────────────────────────── */

.badge {
  display: inline-block;
  padding: 0.25rem 0.75rem;
  border-radius: 9999px;
  font-size: 0.688rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

.badge.success {
  background: #ecfdf5;
  color: #166534;
}

.badge.warning {
  background: #fef9ec;
  color: #92400e;
}

.badge.danger {
  background: #fef2f2;
  color: #991b1b;
}

.badge.info {
  background: #fdf3ee;
  color: #9a3412;
}

.badge.increasing {
  background: #fdf3ee;
  color: #9a3412;
}

.badge.decreasing {
  background: #fef2f2;
  color: #991b1b;
}

.badge.stable {
  background: #f0fdf4;
  color: #166534;
}

.badge.high {
  background: #fef2f2;
  color: #991b1b;
}

.badge.medium {
  background: #fef9ec;
  color: #92400e;
}

.badge.low {
  background: #f0fdf4;
  color: #166534;
}

/* ── Loading / Error ────────────────────────────────────────────────────── */

.loading {
  text-align: center;
  padding: 3rem;
  color: #6b5c4e;
  font-size: 0.938rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #9a3412;
  padding: 1rem;
  border-radius: 8px;
  margin: 1rem 0;
  font-size: 0.938rem;
}
</style>

<style scoped>
/* ── Top Nav ────────────────────────────────────────────────────────────── */

.top-nav {
  background: #fffdf9;
  border-bottom: 1px solid #e8e0d5;
  box-shadow: 0 1px 4px rgba(26, 19, 9, 0.06);
  position: sticky;
  top: 0;
  z-index: 100;
  height: 64px;
  display: flex;
  align-items: center;
}

.nav-container {
  max-width: 1600px;
  width: 100%;
  margin: 0 auto;
  display: flex;
  align-items: center;
  padding: 0 2rem;
  height: 100%;
}

.nav-container > .nav-tabs {
  margin-left: auto;
  margin-right: 1rem;
}

.nav-container > .language-switcher {
  margin-right: 1rem;
}

/* ── Logo ───────────────────────────────────────────────────────────────── */

.logo {
  display: flex;
  align-items: center;
  gap: 0.625rem;
}

.logo-mark {
  display: block;
  width: 8px;
  height: 28px;
  background: #d97757;
  border-radius: 4px;
  flex-shrink: 0;
}

.logo h1 {
  font-size: 1.25rem;
  font-weight: 700;
  color: #1a1309;
  letter-spacing: -0.03em;
  margin-right: 0.125rem;
}

.subtitle {
  font-size: 0.813rem;
  color: #6b5c4e;
  font-weight: 400;
  padding-left: 0.75rem;
  border-left: 1px solid #e8e0d5;
}

/* ── Nav Tabs ───────────────────────────────────────────────────────────── */

.nav-tabs {
  display: flex;
  gap: 0.125rem;
}

.nav-tabs a {
  padding: 0.5rem 1rem;
  color: #6b5c4e;
  text-decoration: none;
  font-weight: 500;
  font-size: 0.875rem;
  border-radius: 9999px;
  transition: all 0.15s ease;
  white-space: nowrap;
}

.nav-tabs a:hover {
  color: #1a1309;
  background: #f5f0e8;
}

.nav-tabs a.active {
  color: #d97757;
  background: #fdf3ee;
}
</style>
