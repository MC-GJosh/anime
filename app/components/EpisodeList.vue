<template>
  <div class="episode-list glass">
    <div class="header">
      <div class="header-left">
        <h3>Episodes</h3>
        <span class="total-badge" v-if="totalPages > 0">{{ (currentPage - 1) * 100 + episodes.length }} total</span>
      </div>
      <div class="header-right">
        <!-- Search -->
        <div class="search-box" :class="{ active: searchOpen }">
          <button class="search-toggle" @click="toggleSearch" :aria-label="searchOpen ? 'Close search' : 'Search episodes'">
            <svg v-if="!searchOpen" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="16" height="16">
              <circle cx="11" cy="11" r="8"/><path d="m21 21-4.35-4.35"/>
            </svg>
            <svg v-else viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="16" height="16">
              <path d="M18 6 6 18M6 6l12 12"/>
            </svg>
          </button>
          <input
            v-if="searchOpen"
            ref="searchInput"
            v-model="searchQuery"
            class="search-input"
            type="text"
            placeholder="Search episodes…"
            @keydown.escape="closeSearch"
          />
        </div>
        <!-- Pagination -->
        <div class="pagination" v-if="totalPages > 1">
          <button class="page-btn" :disabled="currentPage <= 1" @click="$emit('page-change', currentPage - 1)">‹</button>
          <span class="page-info">Page {{ currentPage }} / {{ totalPages }}</span>
          <button class="page-btn" :disabled="currentPage >= totalPages" @click="$emit('page-change', currentPage + 1)">›</button>
        </div>
      </div>
    </div>

    <!-- Loading overlay -->
    <div v-if="loadingEpisodes" class="loading-overlay">
      <div class="mini-spinner"></div>
      <span>Loading episodes...</span>
    </div>

    <div v-else class="grid">
      <!-- No results -->
      <div v-if="filteredEpisodes.length === 0" class="no-results">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" width="32" height="32"><circle cx="11" cy="11" r="8"/><path d="m21 21-4.35-4.35"/></svg>
        <p>No episodes matching <strong>"{{ searchQuery }}"</strong></p>
      </div>
      <button
        v-for="ep in filteredEpisodes"
        :key="ep.mal_id"
        class="episode-card"
        :class="{
          active: activeEpisode === ep.mal_id,
          filler: ep.filler,
          recap: ep.recap
        }"
        @click="$emit('select', ep.mal_id)"
      >
        <div class="ep-header">
          <span class="ep-num">EP {{ ep.mal_id }}</span>
          <div class="ep-tags">
            <span v-if="ep.filler" class="tag tag-filler">FILLER</span>
            <span v-if="ep.recap" class="tag tag-recap">RECAP</span>
          </div>
        </div>
        <span class="ep-title">{{ ep.title }}</span>
        <div class="ep-footer" v-if="ep.score">
          <span class="ep-score">★ {{ ep.score }}</span>
        </div>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, nextTick } from 'vue'

const props = defineProps({
  episodes:        { type: Array,   required: true },
  activeEpisode:   { type: Number,  required: true },
  currentPage:     { type: Number,  default: 1 },
  totalPages:      { type: Number,  default: 1 },
  loadingEpisodes: { type: Boolean, default: false },
})
defineEmits(['select', 'page-change'])

// Search
const searchOpen  = ref(false)
const searchQuery = ref('')
const searchInput = ref(null)

function toggleSearch() {
  searchOpen.value = !searchOpen.value
  if (searchOpen.value) {
    searchQuery.value = ''
    nextTick(() => searchInput.value?.focus())
  }
}

function closeSearch() {
  searchOpen.value = false
  searchQuery.value = ''
}

const filteredEpisodes = computed(() => {
  const q = searchQuery.value.trim().toLowerCase()
  if (!q) return props.episodes
  return props.episodes.filter(ep =>
    String(ep.mal_id).includes(q) ||
    ep.title?.toLowerCase().includes(q)
  )
})
</script>

<style scoped>
.episode-list {
  border-radius: 12px;
  overflow: hidden;
}

.header {
  padding: 1.25rem 1.5rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 0.75rem;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.header-left h3 {
  margin: 0;
  color: var(--color-primary);
  font-size: 1.8rem;
}

.total-badge {
  background: rgba(255, 255, 255, 0.08);
  padding: 0.2rem 0.7rem;
  border-radius: 20px;
  font-size: 0.85rem;
  font-family: var(--font-body);
  color: var(--color-text-muted);
}

.header-right {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  flex-wrap: wrap;
}

/* Search */
.search-box {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  background: rgba(0, 0, 0, 0.25);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  padding: 0.35rem 0.6rem;
  transition: border-color 0.2s, box-shadow 0.2s;
}
.search-box.active {
  border-color: var(--color-primary);
  box-shadow: 0 0 0 3px rgba(239, 68, 68, 0.12);
}
.search-toggle {
  background: none;
  border: none;
  cursor: pointer;
  color: var(--color-text-muted);
  display: flex;
  align-items: center;
  padding: 0;
  transition: color 0.2s;
  flex-shrink: 0;
}
.search-toggle:hover { color: var(--color-primary); }
.search-input {
  background: none;
  border: none;
  outline: none;
  color: var(--color-text);
  font-family: var(--font-body);
  font-size: 0.9rem;
  width: 180px;
  transition: width 0.3s ease;
}
.search-input::placeholder { color: var(--color-text-muted); }

/* No results */
.no-results {
  grid-column: 1 / -1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
  padding: 3rem 1rem;
  color: var(--color-text-muted);
  text-align: center;
}
.no-results strong { color: var(--color-accent); }

/* Loading */
.loading-overlay {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  padding: 3rem;
  color: var(--color-text-muted);
}

.mini-spinner {
  width: 24px;
  height: 24px;
  border: 3px solid rgba(255,255,255,0.1);
  border-top-color: var(--color-primary);
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
  flex-shrink: 0;
}

@keyframes spin { to { transform: rotate(360deg); } }

/* Grid */
.grid {
  padding: 1.25rem;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 0.75rem;
}

.episode-card {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  padding: 1rem;
  background: rgba(0, 0, 0, 0.25);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 8px;
  color: var(--color-text);
  cursor: pointer;
  text-align: left;
  transition: all 0.2s ease;
  font-family: var(--font-body);
  position: relative;
  overflow: hidden;
}

.episode-card::before {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(135deg, rgba(239,68,68,0.05), transparent);
  opacity: 0;
  transition: opacity 0.2s;
}

.episode-card:hover {
  border-color: rgba(250, 204, 21, 0.3);
  transform: translateY(-2px);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.3);
}

.episode-card:hover::before {
  opacity: 1;
}

.episode-card.active {
  background: rgba(239, 68, 68, 0.12);
  border-color: var(--color-primary);
  box-shadow: 0 0 20px rgba(239, 68, 68, 0.15);
}

.episode-card.active .ep-num {
  color: var(--color-primary);
}

.episode-card.filler {
  border-left: 3px solid #f59e0b;
  opacity: 0.7;
}

.episode-card.recap {
  border-left: 3px solid #6366f1;
  opacity: 0.7;
}

.ep-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.ep-num {
  font-size: 0.85rem;
  font-weight: 700;
  color: var(--color-text-muted);
  letter-spacing: 0.5px;
}

.ep-title {
  font-size: 0.9rem;
  line-height: 1.4;
  color: var(--color-text);
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  flex: 1;
}

.ep-footer {
  margin-top: auto;
}

.ep-score {
  font-size: 0.8rem;
  color: var(--color-accent);
}

.ep-tags {
  display: flex;
  gap: 0.3rem;
}

.tag {
  font-size: 0.65rem;
  padding: 0.1rem 0.35rem;
  border-radius: 3px;
  font-weight: 700;
  letter-spacing: 0.5px;
}

.tag-filler {
  background: rgba(245, 158, 11, 0.2);
  color: #fbbf24;
}

.tag-recap {
  background: rgba(99, 102, 241, 0.2);
  color: #a5b4fc;
}

/* Pagination */
.pagination {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.page-btn {
  width: 32px;
  height: 32px;
  border-radius: 6px;
  border: 1px solid rgba(255,255,255,0.12);
  background: rgba(0,0,0,0.3);
  color: var(--color-text);
  font-size: 1.2rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  line-height: 1;
}

.page-btn:hover:not(:disabled) {
  background: rgba(239, 68, 68, 0.2);
  border-color: var(--color-primary);
}

.page-btn:disabled {
  opacity: 0.25;
  cursor: not-allowed;
}

.page-info {
  font-family: var(--font-body);
  font-size: 0.85rem;
  color: var(--color-text-muted);
  white-space: nowrap;
}
</style>
