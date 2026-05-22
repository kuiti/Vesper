<template>
  <div class="reminder-list">
    <div class="add-item">
      <input v-model="newContent" placeholder="提醒内容" />
      <input type="datetime-local" v-model="newTargetTime" />
      <select v-model="newLevel">
        <option v-for="l in levels" :key="l.level" :value="l.level">{{ l.name }} ({{ l.advance_minutes }}分钟前)</option>
      </select>
      <button @click="addReminder">+</button>
    </div>
    <div class="items">
      <div v-for="r in reminders" :key="r.id" :class="['item', { done: r.done }]" :data-level="r.level">
        <div>
          <strong>{{ r.content }}</strong>
          <div class="meta">目标: {{ formatTime(r.target_time) }} | 等级: {{ getLevelName(r.level) }}</div>
        </div>
        <div class="actions">
          <button v-if="!r.done" class="btn-done" @click="markDone(r.id)" title="标记完成">✓</button>
          <button @click="deleteReminder(r.id)">🗑️</button>
        </div>
      </div>
      <div v-if="!reminders.length" class="empty">暂无提醒</div>
    </div>
  </div>
</template>

<script>
import api from '../api.js'

const LEVELS = [
  { level: 7, name: '强制', advance_minutes: 0 },
  { level: 6, name: '重要', advance_minutes: 21*24*60 },
  { level: 5, name: '考试', advance_minutes: 14*24*60 },
  { level: 4, name: '待办', advance_minutes: 5*24*60 },
  { level: 3, name: '计划', advance_minutes: 2*24*60 },
  { level: 2, name: '日常', advance_minutes: 5*60 },
  { level: 1, name: '喝水', advance_minutes: 30 }
]

export default {
  inject: ['showConfirm'],
  data() {
    return {
      reminders: [],
      newContent: '',
      newTargetTime: '',
      newLevel: 4,
      levels: LEVELS
    }
  },
  mounted() {
    this.load()
  },
  methods: {
    async load() {
      try {
        const res = await api.get('/reminders/')
        this.reminders = res.data
      } catch (err) {
        console.error('加载提醒失败', err)
      }
    },
    formatTime(iso) {
      if (!iso) return ''
      return new Date(iso).toLocaleString()
    },
    getLevelName(level) {
      const l = LEVELS.find(l => l.level === level)
      return l ? l.name : '未知'
    },
    async addReminder() {
      if (!this.newContent || !this.newTargetTime) return
      try {
        await api.post('/reminders/', {
          content: this.newContent,
          target_time: this.newTargetTime,
          level: this.newLevel
        })
        this.newContent = ''
        this.newTargetTime = ''
        this.newLevel = 4
        await this.load()
      } catch (err) {
        console.error('添加失败', err)
      }
    },
    async markDone(id) {
      try {
        await api.patch(`/reminders/${id}/done`)
        await this.load()
      } catch (err) {
        console.error('标记失败', err)
      }
    },
    async deleteReminder(id) {
      if (!await this.showConfirm('删除此提醒？')) return
      try {
        await api.delete(`/reminders/${id}`)
        await this.load()
      } catch (err) {
        console.error('删除失败', err)
      }
    }
  }
}
</script>

<style scoped>
.reminder-list { padding: 8px; }
.add-item { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px; }
.add-item input, .add-item select { flex: 1; min-width: 120px; padding: 6px; border-radius: 6px; background: #1a1a2e; color: white; border: 1px solid #2c3e50; }
.add-item button { background: #4e89ae; border: none; color: white; border-radius: 6px; cursor: pointer; padding: 0 16px; }
.item { display: flex; justify-content: space-between; align-items: center; padding: 8px 10px; background: #16213e; border-radius: 6px; margin-bottom: 8px; border-left: 3px solid transparent; transition: transform .15s, background .15s, border-color .2s; }
.item:hover { background: rgba(255,255,255,.04); transform: translateX(2px); }
.item.done { opacity: 0.5; }
.item.done strong { text-decoration: line-through; }
.item[data-level="7"], .item[data-level="6"] { border-left-color: #e74c3c; }
.item[data-level="5"], .item[data-level="4"] { border-left-color: #f59e0b; }
.item[data-level="3"], .item[data-level="2"] { border-left-color: #5390d4; }
.item[data-level="1"] { border-left-color: #4caf50; }
.meta { font-size: 12px; color: #aaa; margin-top: 4px; }
.actions { display: flex; gap: 6px; align-items: center; }
.actions button { background: none; border: none; cursor: pointer; opacity: .4; transition: opacity .15s; }
.actions button:hover { opacity: 1; }
.btn-done { color: #4caf50 !important; font-size: 16px; font-weight: bold; opacity: 1 !important; }
.empty { text-align: center; color: #7f8c8d; font-size: 13px; padding: 20px; }
</style>