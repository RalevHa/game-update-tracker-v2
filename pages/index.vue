<template>
  <div class="min-h-screen bg-gradient-to-b from-white to-slate-50">
    <div class="mx-auto max-w-3xl p-6">
      <header class="mb-6">
        <h1
          class="text-3xl font-extrabold tracking-tight bg-clip-text text-transparent bg-gradient-to-r from-indigo-600 to-emerald-600">
          Game Update Tracker</h1>
        <p class="mt-2 text-slate-600">Browse, edit, and export game update notes with a clean, quick workflow.</p>
      </header>

      <template v-if="showImport">
        <div class="fixed inset-0 bg-black bg-opacity-30 flex items-center justify-center z-50">
          <form @submit.prevent="importJson"
            class="rounded-lg border border-slate-300 bg-white p-6 shadow space-y-3 w-full max-w-md">
            <label class="block text-sm text-slate-700">Import JSON file
              <input type="file" accept="application/json" @change="handleImportFile" class="mt-1 block w-full" />
            </label>
            <div class="flex gap-2">
              <button type="submit"
                class="inline-flex items-center gap-1 rounded-lg bg-blue-600 px-4 py-2 text-white shadow hover:bg-blue-700">⬆
                Import</button>
              <button type="button" @click="showImport = false"
                class="inline-flex items-center gap-1 rounded-lg bg-slate-300 px-4 py-2 text-slate-900 shadow hover:bg-slate-400">✖
                Cancel</button>
            </div>
            <div v-if="importError" class="text-red-600 text-sm mt-2">{{ importError }}</div>
          </form>
        </div>
      </template>

      <section class="mb-6 rounded-xl border border-slate-200 bg-white/70 backdrop-blur p-4 shadow-sm">
        <div class="flex flex-col sm:flex-row sm:items-end sm:justify-between gap-2 mb-2">
          <div class="flex-3">
            <label class="block text-sm font-medium text-slate-700 mb-1">Search or select game</label>
            <input v-model="gameSearch" placeholder="Search games..."
              class="mb-2 w-full rounded-lg border border-slate-300 px-3 py-2 text-slate-800 shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500" />
            <select v-model="selectedGame"
              class="w-full appearance-none rounded-lg border border-slate-300 bg-white px-3 py-2 text-slate-800 shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500">
              <option v-for="game in filteredGames" :key="game.id" :value="game.id">{{ game.name }}</option>
            </select>
          </div>
          <div class="flex-1">
            <div class="flex flex-col sm:flex-row sm:items-end sm:justify-between gap-2 mb-2">
              <button @click="showImport = true"
                class="inline-flex items-center gap-1 rounded-lg bg-blue-600 px-4 py-2 text-white shadow hover:bg-blue-700">⬆
                Import JSON</button>
            </div>
            <div class="flex flex-col sm:flex-row sm:items-end sm:justify-between gap-2">

            <button @click="showNewGame = true"
              class="mt-2 sm:mt-0 inline-flex items-center gap-1 rounded-lg bg-indigo-600 px-4 py-2 text-white shadow hover:bg-indigo-700">＋
              Add New Game</button>
          </div>
          </div>
        </div>

        <Transition name="fade">
          <form v-if="showNewGame" @submit.prevent="addNewGame"
            class="mt-4 rounded-lg border border-slate-300 bg-white p-4 shadow space-y-3">
            <div class="flex flex-col sm:flex-row gap-3">
              <label class="flex-1 text-sm text-slate-700">Game Name
                <input v-model="newGame.name"
                  class="mt-1 w-full rounded-md border border-slate-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500"
                  placeholder="e.g. Minecraft" required />
              </label>
              <label class="flex-1 text-sm text-slate-700">Game ID
                <input v-model="newGame.id"
                  class="mt-1 w-full rounded-md border border-slate-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500"
                  placeholder="e.g. minecraft" required />
              </label>
            </div>
            <div class="flex gap-2">
              <button type="submit"
                class="inline-flex items-center gap-1 rounded-lg bg-emerald-600 px-4 py-2 text-white shadow hover:bg-emerald-700">✔
                Add Game</button>
              <button type="button" @click="cancelNewGame"
                class="inline-flex items-center gap-1 rounded-lg bg-slate-300 px-4 py-2 text-slate-900 shadow hover:bg-slate-400">✖
                Cancel</button>
            </div>
            <div v-if="newGameError" class="text-red-600 text-sm mt-2">{{ newGameError }}</div>
          </form>
        </Transition>
      </section>

      <section v-if="currentGame" class="rounded-xl border border-slate-200 bg-white/80 backdrop-blur p-4 shadow">
        <div class="flex justify-end mb-2">
          <button @click="confirmDeleteGame"
            class="inline-flex items-center gap-1 rounded-lg bg-red-600 px-3 py-1.5 text-white shadow hover:bg-red-700">🗑
            Delete Game</button>
        </div>
        <div class="flex flex-col sm:flex-row sm:items-start sm:justify-between gap-3 mb-4">
          <div class="flex-1">
            <h2 class="text-2xl font-semibold text-slate-800">
              <template v-if="!editMode">{{ currentGame.name }}</template>
              <template v-else>
                <input v-model="editableGame.name"
                  class="w-full rounded-lg border border-slate-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 mb-2"
                  placeholder="Game name" />
                <input v-model="editableGame.id"
                  class="w-full rounded-lg border border-slate-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500"
                  placeholder="Game ID" />
              </template>
            </h2>
            <p class="text-slate-500 text-sm mt-1">ID: <code class="text-slate-600">{{ editMode ? editableGame.id :
              currentGame.id
            }}</code></p>
          </div>
          <div class="flex gap-2 justify-end">
            <button v-if="!editMode" @click="startEdit"
              class="inline-flex items-center gap-1 rounded-lg bg-slate-800 px-4 py-2 text-white shadow hover:bg-slate-900">✏️
              Edit</button>
            <template v-else>
              <button @click="saveEdit"
                class="inline-flex items-center gap-1 rounded-lg bg-emerald-600 px-4 py-2 text-white shadow hover:bg-emerald-700">✔
                Save</button>
              <button @click="cancelEdit"
                class="inline-flex items-center gap-1 rounded-lg bg-slate-300 px-4 py-2 text-slate-900 shadow hover:bg-slate-400">✖
                Cancel</button>
            </template>
          </div>
        </div>

        <div class="flex flex-col sm:flex-row sm:items-center gap-2 mb-2">
          <div class="flex-1"></div>
          <div class="flex gap-2">
            <label class="text-sm text-slate-700">Sort by
              <select v-model="sortBy" class="ml-2 rounded border border-slate-300 px-2 py-1">
                <option value="date">Date</option>
                <option value="version">Version</option>
              </select>
            </label>
            <button @click="toggleSortOrder" class="rounded bg-slate-200 px-2 py-1 text-slate-700">{{ sortOrder ===
              'asc' ? '↑'
              : '↓' }}</button>
          </div>
        </div>
        <Transition name="fade" mode="out-in">
          <div v-if="!editMode" key="view" class="space-y-3">
            <div v-if="sortedUpdates.length" class="grid grid-cols-1 gap-3">
              <article v-for="update in sortedUpdates" :key="update.version + update.date"
                class="rounded-lg border border-slate-200 bg-white p-4 shadow-sm hover:shadow-md transition-shadow">
                <div class="flex flex-wrap items-center gap-2 mb-2">
                  <span
                    class="inline-flex items-center rounded-full bg-indigo-50 text-indigo-700 px-2 py-0.5 text-xs font-medium border border-indigo-100">v{{
                      update.version }}</span>
                  <span
                    class="inline-flex items-center rounded-full bg-emerald-50 text-emerald-700 px-2 py-0.5 text-xs font-medium border border-emerald-100">{{
                      formatDate(update.date) }}</span>
                </div>
                <div class="prose prose-sm max-w-none text-slate-800" v-html="renderMarkdown(update.note)"></div>
                <div class="mt-3">
                  <button @click="copyDiscordTimestamp(update.date)"
                    class="inline-flex items-center gap-1 rounded-md bg-blue-600 px-3 py-1.5 text-white shadow hover:bg-blue-700"
                    title="Copy Discord timestamp">🕒 Copy Discord Timestamp</button>
                </div>
              </article>
            </div>
            <div v-else class="rounded-lg border-2 border-dashed border-slate-200 p-8 text-center text-slate-500">No
              updates
              yet.</div>
          </div>

          <div v-else key="edit" class="space-y-4">
            <div v-if="editableGame.updates?.length" class="space-y-3">
              <article v-for="(update, idx) in editableGame.updates" :key="idx"
                class="rounded-lg border border-slate-200 bg-white p-4 shadow-sm">
                <div class="grid grid-cols-1 sm:grid-cols-3 gap-3">
                  <label class="text-sm text-slate-600">Version
                    <input v-model="update.version"
                      class="mt-1 w-full rounded-md border border-slate-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500"
                      placeholder="e.g. 1.2.3" />
                  </label>
                  <label class="text-sm text-slate-600">Date
                    <input v-model="update.date" type="datetime-local"
                      class="mt-1 w-full rounded-md border border-slate-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500" />
                  </label>
                  <label class="text-sm text-slate-600 sm:col-span-3">Note
                    <textarea v-model="update.note"
                      class="mt-1 w-full rounded-md border border-slate-300 px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500"
                      rows="2" placeholder="What changed?" />
                  </label>
                </div>
                <div class="mt-3 flex flex-wrap gap-2">
                  <button @click="copyDiscordTimestamp(update.date)"
                    class="inline-flex items-center gap-1 rounded-md bg-blue-600 px-3 py-1.5 text-white shadow hover:bg-blue-700">🕒
                    Copy Discord Timestamp</button>
                  <button @click="removeUpdate(idx)"
                    class="inline-flex items-center gap-1 rounded-md bg-red-600 px-3 py-1.5 text-white shadow hover:bg-red-700">🗑
                    Remove</button>
                </div>
              </article>
            </div>
            <div v-else class="rounded-lg border-2 border-dashed border-slate-200 p-8 text-center text-slate-500">No
              updates
              yet. Add one below.</div>

            <div class="flex flex-wrap gap-2">
              <button @click="addUpdate"
                class="inline-flex items-center gap-1 rounded-lg bg-indigo-600 px-4 py-2 text-white shadow hover:bg-indigo-700">＋
                Add Update</button>
              <span class="text-slate-400 text-sm">Updates are saved to your browser until you export JSON.</span>
            </div>

            <div class="pt-2 flex flex-wrap gap-2 border-t border-slate-200">
              <button @click="exportJson"
                class="inline-flex items-center gap-1 rounded-lg bg-emerald-600 px-4 py-2 text-white shadow hover:bg-emerald-700">⬇
                Export JSON</button>
              <button @click="copyJson"
                class="inline-flex items-center gap-1 rounded-lg bg-teal-600 px-4 py-2 text-white shadow hover:bg-teal-700">📋
                Copy JSON</button>
            </div>
          </div>
        </Transition>
        <div class="mt-4 text-slate-400 text-sm">* All changes are saved to your browser's local storage.
          Clearing browser data will remove them. Be sure to export your data before clearing.
        </div>
      </section>
      <div class="mt-6 text-center text-slate-500 text-sm">
        Made with ❤️ by <a class="text-indigo-600 hover:underline">Ralev</a>.
      </div>
    </div>
  </div>
</template>

<script setup>

import { ref, computed, onMounted } from 'vue'
import { DateTime } from 'luxon'
import { marked } from 'marked'

const gameSearch = ref('')
const filteredGames = computed(() => {
  const q = gameSearch.value.trim().toLowerCase()
  if (!q) return games.value
  return games.value.filter(g => g.name.toLowerCase().includes(q) || g.id.toLowerCase().includes(q))
})


function renderMarkdown(text) {
  return marked.parse(text || '')
}

const sortBy = ref('date')
const sortOrder = ref('desc')
const sortedUpdates = computed(() => {
  if (!currentGame.value || !currentGame.value.updates) return []
  const arr = [...currentGame.value.updates]
  if (sortBy.value === 'date') {
    arr.sort((a, b) => sortOrder.value === 'asc' ? new Date(a.date) - new Date(b.date) : new Date(b.date) - new Date(a.date))
  } else {
    arr.sort((a, b) => sortOrder.value === 'asc' ? a.version.localeCompare(b.version) : b.version.localeCompare(a.version))
  }
  return arr
})

function toggleSortOrder() {
  sortOrder.value = sortOrder.value === 'asc' ? 'desc' : 'asc'
}

function confirmDeleteGame() {
  if (window.confirm('Are you sure you want to delete this game? This cannot be undone.')) {
    deleteGame()
  }
}
function deleteGame() {
  const idx = games.value.findIndex(g => g.id === currentGame.value.id)
  if (idx !== -1) {
    games.value.splice(idx, 1)
    persist()
    selectedGame.value = games.value[0]?.id || ''
  }
}

const games = ref([])
const selectedGame = ref('')
const currentGame = computed(() => games.value.find(g => g.id === selectedGame.value))
const editMode = ref(false)
const editableGame = ref({ id: '', name: '', updates: [] })

const showNewGame = ref(false)
const newGame = ref({ name: '', id: '' })
const newGameError = ref('')

const showImport = ref(false)
const importError = ref('')
let importFile = null

function handleImportFile(e) {
  importFile = e.target.files[0]
}

async function importJson() {
  if (!importFile) {
    importError.value = 'Please select a file.'
    return
  }
  try {
    const text = await importFile.text()
    const data = JSON.parse(text)
    if (!Array.isArray(data.games) && !Array.isArray(data)) {
      importError.value = 'Invalid format.'
      return
    }
    games.value = Array.isArray(data.games) ? data.games : data
    persist()
    selectedGame.value = games.value[0]?.id || ''
    showImport.value = false
    importError.value = ''
  } catch (err) {
    importError.value = 'Failed to import: ' + err.message
  }
}

onMounted(async () => {
  const stored = localStorage.getItem('gamesData')
  if (stored) {
    try {
      const parsed = JSON.parse(stored)
      if (parsed && Array.isArray(parsed.games)) {
        games.value = parsed.games
      }
    } catch { }
  }
  if (games.value.length === 0) {
    const res = await fetch('/updates.json')
    games.value = (await res.json()).games
  }
  selectedGame.value = games.value[0]?.id || ''
})

function cancelNewGame() {
  showNewGame.value = false
  newGame.value = { name: '', id: '' }
  newGameError.value = ''
}

function addNewGame() {
  const name = newGame.value.name.trim()
  const id = newGame.value.id.trim()
  if (!name || !id) {
    newGameError.value = 'Name and ID are required.'
    return
  }
  if (!/^[a-zA-Z0-9-_]+$/.test(id)) {
    newGameError.value = 'Game ID must be alphanumeric, dash or underscore.'
    return
  }
  if (games.value.some(g => g.id === id)) {
    newGameError.value = 'Game ID already exists.'
    return
  }
  games.value.push({ id, name, updates: [] })
  persist()
  selectedGame.value = id
  cancelNewGame()
}

function formatDate(dateStr) {
  const date = new Date(dateStr)
  if (isNaN(date)) return dateStr
  return DateTime.fromJSDate(date).setZone('Asia/Bangkok').toFormat('d MMMM y H:mm')
}

function copyDiscordTimestamp(dateStr) {
  const unix = Math.floor(new Date(dateStr).getTime() / 1000)
  const discordTs = `<t:${unix}:F>`
  navigator.clipboard.writeText(discordTs)
  alert('Copied: ' + discordTs)
}

function persist() {
  localStorage.setItem('gamesData', JSON.stringify({ games: games.value }))
}

function startEdit() {
  if (!currentGame.value) return
  editMode.value = true
  editableGame.value = JSON.parse(JSON.stringify(currentGame.value))
}

function cancelEdit() {
  editMode.value = false
}

function saveEdit() {
  const idx = games.value.findIndex(g => g.id === editableGame.value.id)
  if (idx !== -1) {
    if (!editableGame.value.name.trim()) {
      alert('Game name is required.')
      return
    }
    if (!editableGame.value.id.trim()) {
      alert('Game ID is required.')
      return
    }
    if (!/^[a-zA-Z0-9-_]+$/.test(editableGame.value.id)) {
      alert('Game ID must be alphanumeric, dash or underscore.')
      return
    }
    if (games.value.some((g, i) => g.id === editableGame.value.id && i !== idx)) {
      alert('Game ID already exists.')
      return
    }
    games.value[idx] = JSON.parse(JSON.stringify(editableGame.value))
    persist()
    selectedGame.value = editableGame.value.id
  }
  editMode.value = false
}

function addUpdate() {
  const now = new Date()
  const isoLocal = new Date(now.getTime() - now.getTimezoneOffset() * 60000).toISOString().slice(0, 16)
  editableGame.value.updates.unshift({ version: '', date: isoLocal, note: '' })
}

function removeUpdate(i) {
  editableGame.value.updates.splice(i, 1)
}

function exportJson() {
  const blob = new Blob([JSON.stringify({ games: games.value }, null, 2)], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'updates.json'
  document.body.appendChild(a)
  a.click()
  a.remove()
  URL.revokeObjectURL(url)
}

async function copyJson() {
  const json = JSON.stringify({ games: games.value }, null, 2)
  await navigator.clipboard.writeText(json)
  alert('JSON copied to clipboard')
}
</script>

<style>
body {
  background: #f9fafb;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 150ms ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>