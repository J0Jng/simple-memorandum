<template>
  <view class="page">
    <view class="header">
      <text class="header-title">看看还有什么没有干</text>
    </view>

    <scroll-view class="grid" scroll-y="true">
      <view v-if="notes.length===0" class="empty">暂无备忘，点击右下角 + 新建</view>
      <view class="grid-wrap" :style="{ gridTemplateColumns: `repeat(${cols}, 1fr)` }">
        <view class="card" v-for="note in notes" :key="note.id" @click="openDetailById(note.id)">
          <view class="delete-btn" @click.stop="deleteNote(note.id)">×</view>
          <text class="note-title">{{ note.title }}</text>
          <text class="note-body">{{ note.content }}</text>
        </view>
      </view>
    </scroll-view>

    <!-- floating add button -->
    <view class="fab" @click="openCreate" role="button">+</view>

    <!-- create modal -->
    <view v-if="showCreate" class="overlay">
      <view class="modal">
        <text class="modal-title">新建备忘</text>
        <input class="modal-input" placeholder="标题（必填）" v-model.trim="form.title" />
        <textarea class="modal-textarea" placeholder="内容（选填）" v-model.trim="form.content"></textarea>
        <view class="modal-actions">
          <button class="btn-cancel" @click="closeCreate">取消</button>
          <button class="btn-ok" @click="createNote">保存</button>
        </view>
      </view>
    </view>

    <!-- detail modal -->
    <view v-if="showDetail" class="overlay">
      <view class="modal">
        <text class="modal-title">{{ activeNote.title }}</text>
        <text class="modal-body">{{ activeNote.content }}</text>
        <view class="modal-actions">
          <button class="btn-cancel" @click="closeDetail">关闭</button>
        </view>
      </view>
    </view>
  </view>
</template>

<script setup>
import { ref, reactive, computed, onMounted, onBeforeUnmount, watch, nextTick } from 'vue'

const notes = ref([])
const cols = ref(2)
// render notes directly

function estimateHeight(note) {
  // simple heuristic: base + title len * 1 + content len * 0.6
  const base = 48
  const t = (note.title || '').length
  const c = (note.content || '').length
  return base + t * 1 + c * 0.6
}

function buildColumns() {
  const k = cols.value || 1
  const colsArr = Array.from({ length: k }, () => [])
  for (let i = 0; i < notes.value.length; i++) {
    const n = notes.value[i]
    const idx = i % k
    colsArr[idx].push(n)
  }
  columnsData.value = colsArr
}
const showCreate = ref(false)
const showDetail = ref(false)
const activeIndex = ref(-1)

const form = reactive({ title: '', content: '' })

const activeNote = computed(() => {
  return notes.value[activeIndex.value] || { title: '', content: '' }
})

const STORAGE_KEY = 'uni_memo_notes_v1'

onMounted(() => {
  try {
    const raw = localStorage.getItem(STORAGE_KEY)
    if (raw) notes.value = JSON.parse(raw)
  } catch (e) {
    console.warn('load notes failed', e)
  }
  updateCols()
  window.addEventListener('resize', updateCols)
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', updateCols)
})

watch(notes, () => {})

function updateCols() {
  try {
    const w = window.innerWidth || document.documentElement.clientWidth
  if (w >= 400) cols.value = 3
  else if (w >= 200) cols.value = 2
  else cols.value = 1
  // no masonry rebuild needed
  } catch (e) {
    cols.value = 2
  }
}

function saveNotes() {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(notes.value))
  } catch (e) {
    console.warn('save notes failed', e)
  }
}

function openCreate() {
  form.title = ''
  form.content = ''
  showCreate.value = true
}

function closeCreate() { showCreate.value = false }

function createNote() {
  if (!form.title || !form.title.trim()) {
    if (typeof uni !== 'undefined' && uni.showToast) {
      uni.showToast({ title: '请填写标题', icon: 'none' })
    } else {
      alert('请填写标题')
    }
    return
  }
  const newNote = { id: Date.now(), title: form.title.trim(), content: form.content.trim() }
  notes.value.unshift(newNote)
  saveNotes()
  showCreate.value = false
  if (typeof uni !== 'undefined' && uni.showToast) uni.showToast({ title: '已保存', icon: 'success' })
}

function openDetailById(noteId) {
  const idx = notes.value.findIndex(n => n.id === noteId)
  if (idx === -1) return
  activeIndex.value = idx
  showDetail.value = true
}

function closeDetail() { showDetail.value = false }

function deleteNote(noteId) {
  const idx = notes.value.findIndex(n => n.id === noteId)
  if (idx === -1) return
  notes.value.splice(idx, 1)
  saveNotes()
  buildColumns()
  // if detail was open for this note, close it
  if (activeIndex.value === idx) {
    showDetail.value = false
  }
  // adjust activeIndex if needed (shifted indices)
  if (activeIndex.value > idx) activeIndex.value -= 1
}
</script>

<style scoped>
  .page { min-height:100vh; background: linear-gradient(135deg,#f8fafc 0%,#e9eeff 100%); padding: 12px; box-sizing: border-box }
  .header { width:100%; display:flex; justify-content:center; padding-top:12px }
  .header-title { font-size:18px; color:#3b4cca; font-weight:700 }
  .grid { margin-top:14px; padding-bottom:120px }
  .grid-wrap { display:grid; gap:12px; width:100%; }
  .slot { width:100%; }
  .card { position:relative; aspect-ratio: 1 / 1; background:#fff; border-radius:10px; padding:10px; box-shadow:0 6px 18px rgba(60,80,180,0.06); box-sizing:border-box; display:flex; flex-direction:column; justify-content:flex-start }
  .empty-card { background: linear-gradient(180deg,#fbfdff,#f6f8ff); opacity:0.6 }
  .delete-btn { position: absolute; right:8px; top:8px; width:24px; height:24px; border-radius:12px; background:rgba(0,0,0,0.06); color:#374151; display:flex; align-items:center; justify-content:center; font-weight:700; cursor:pointer }
  .note-title { display:block; font-weight:700; color:#111827; margin-bottom:6px; font-size:14px }
  .note-body { color:#6b7280; font-size:13px; line-height:1.4; max-height:68px; overflow:hidden }
  .empty { width:100%; text-align:center; color:#9ca3af; padding:24px 0 }

  .fab { position:fixed; right:18px; bottom:44px; width:56px; height:56px; border-radius:28px; background:linear-gradient(90deg,#3b4cca 0%,#6366f1 100%); color:#fff; font-size:34px; line-height:56px; text-align:center; box-shadow:0 10px 26px rgba(99,102,241,0.18); z-index:40; }

  .overlay { position:fixed; left:0; right:0; top:0; bottom:0; background:rgba(0,0,0,0.35); display:flex; align-items:center; justify-content:center; z-index:50 }
  .modal { width:88vw; max-width:420px; background:#fff; border-radius:12px; padding:14px; box-sizing:border-box }
  .modal-title { font-weight:700; color:#111827; font-size:16px; margin-bottom:10px }
  .modal-input { width:100%; height:36px; border:1px solid #e6e9f2; border-radius:6px; padding:6px 10px; box-sizing:border-box; margin-bottom:8px }
  .modal-textarea { width:100%; min-height:120px; border:1px solid #e6e9f2; border-radius:6px; padding:8px 10px; box-sizing:border-box; margin-bottom:10px; resize:none }
  .modal-body { color:#6b7280; margin-bottom:12px }
  .modal-actions { display:flex; gap:10px; justify-content:flex-end }
  .btn-cancel { background:#f3f4f6; color:#374151; padding:8px 12px; border-radius:8px }
  .btn-ok { background:linear-gradient(90deg,#3b4cca 0%,#6366f1 100%); color:#fff; padding:8px 12px; border-radius:8px }
</style>
