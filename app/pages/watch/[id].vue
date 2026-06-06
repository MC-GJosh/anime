<template>
  <div class="watch-page">
    <div class="container">

      <!-- Back button -->
      <NuxtLink to="/" class="back-link">← Back to Home</NuxtLink>

      <!-- Loading State -->
      <div v-if="loading" class="loading-state">
        <div class="spinner"></div>
        <p>Sailing to the Grand Line…</p>
      </div>

      <!-- Error State -->
      <div v-else-if="error" class="error-state glass">
        <h2>⚠️ Yikes!</h2>
        <p>{{ error }}</p>
        <button class="btn" @click="fetchAll">Try Again</button>
      </div>

      <!-- Main Content -->
      <template v-else>

        <!-- Page grid: left (player + episodes) | right (details sidebar) -->
        <div class="page-grid">

          <!-- LEFT: Player + Episodes stacked -->
          <div class="main-col">
            <section class="player-section">
              <VideoPlayer
                :episodeTitle="currentEpisodeTitle"
                :episodeNumber="currentEpisodeId"
                :hasPrev="currentEpisodeId > 1"
                :hasNext="currentEpisodeId < totalEpisodes"
                :posterImage="animeData?.image"
                @prev="handlePrev"
                @next="handleNext"
              />
            </section>

            <section class="episodes-section">
              <EpisodeList
                :episodes="episodes"
                :activeEpisode="currentEpisodeId"
                :currentPage="currentPage"
                :totalPages="totalPages"
                :loadingEpisodes="loadingEpisodes"
                @select="handleEpisodeSelect"
                @page-change="handlePageChange"
              />
            </section>
          </div>

          <!-- RIGHT: Anime Details sidebar spanning full height -->
          <aside class="details-sidebar" v-if="animeData">
            <AnimeDetails :anime="animeData" />
          </aside>

        </div>

      </template>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const ANIME_ID = 21 // One Piece MAL ID

const animeData        = ref(null)
const episodes         = ref([])
const currentEpisodeId = ref(1)
const currentPage      = ref(1)
const totalPages       = ref(1)
const totalEpisodes    = ref(1200)
const loading          = ref(true)
const loadingEpisodes  = ref(false)
const error            = ref(null)

const currentEpisodeTitle = computed(() => {
  const ep = episodes.value.find(ep => ep.mal_id === currentEpisodeId.value)
  return ep ? `EP ${ep.mal_id} — ${ep.title}` : `Episode ${currentEpisodeId.value}`
})

async function fetchAnimeDetails() {
  const res = await fetch(`https://api.jikan.moe/v4/anime/${ANIME_ID}`)
  if (!res.ok) throw new Error('Failed to fetch anime details')
  const { data: d } = await res.json()

  animeData.value = {
    id:         d.mal_id,
    title:      d.title,
    rating:     d.score ?? 'N/A',
    year:       d.year ?? d.aired?.prop?.from?.year ?? '—',
    status:     d.status,
    genres:     [...d.genres.map(g => g.name), ...d.demographics.map(g => g.name)],
    synopsis:   d.synopsis,
    image:      d.images?.webp?.large_image_url || d.images?.jpg?.large_image_url,
    studio:     d.studios?.[0]?.name ?? '—',
    type:       d.type,
    episodes:   d.episodes,
    duration:   d.duration,
    rank:       d.rank,
    popularity: d.popularity,
    members:    d.members,
  }

  totalEpisodes.value = d.episodes || 1200
}

async function fetchEpisodes(page = 1) {
  loadingEpisodes.value = true
  try {
    const res = await fetch(`https://api.jikan.moe/v4/anime/${ANIME_ID}/episodes?page=${page}`)
    if (!res.ok) throw new Error('Failed to fetch episodes')
    const json = await res.json()
    episodes.value  = json.data
    totalPages.value = json.pagination.last_visible_page
    currentPage.value = page
  } finally {
    loadingEpisodes.value = false
  }
}

async function fetchAll() {
  loading.value = true
  error.value   = null
  try {
    await fetchAnimeDetails()
    await new Promise(r => setTimeout(r, 400)) // respect Jikan rate limit
    await fetchEpisodes(1)
  } catch (e) {
    error.value = e.message
  } finally {
    loading.value = false
  }
}

function handleEpisodeSelect(id) {
  currentEpisodeId.value = id
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

async function handlePageChange(page) {
  await fetchEpisodes(page)
}

function handlePrev() {
  if (currentEpisodeId.value > 1) {
    currentEpisodeId.value--
    navigateToEpisodePage(currentEpisodeId.value)
  }
}

function handleNext() {
  if (currentEpisodeId.value < totalEpisodes.value) {
    currentEpisodeId.value++
    navigateToEpisodePage(currentEpisodeId.value)
  }
}

async function navigateToEpisodePage(epId) {
  const targetPage = Math.ceil(epId / 100)
  if (targetPage !== currentPage.value) {
    await fetchEpisodes(targetPage)
  }
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

onMounted(fetchAll)
</script>

<style scoped>
.watch-page {
  min-height: 100vh;
  padding-bottom: 6rem;
  background-color: var(--color-bg-dark);
}

.container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 0 2rem;
}

.back-link {
  display: inline-block;
  color: var(--color-text-muted);
  text-decoration: none;
  padding: 1.5rem 0;
  transition: color 0.2s;
  font-size: 1rem;
}
.back-link:hover { color: var(--color-accent); }

/* ── Page Grid: left col (player+eps) | right col (details) ── */
.page-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1.75rem;
  align-items: start;
}

@media (min-width: 1024px) {
  .page-grid {
    /* left takes remaining space, right is fixed 360px */
    grid-template-columns: 1fr 360px;
  }
}

.main-col {
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.player-section {
  width: 100%;
}

.episodes-section {
  width: 100%;
}

/* RIGHT sidebar — sticky to top, scrolls independently */
.details-sidebar {
  position: sticky;
  top: 1.5rem;
  max-height: calc(100vh - 3rem);
  overflow-y: auto;
  scrollbar-width: thin;
  scrollbar-color: var(--color-primary) transparent;
}
.details-sidebar::-webkit-scrollbar { width: 4px; }
.details-sidebar::-webkit-scrollbar-thumb {
  background: var(--color-primary);
  border-radius: 2px;
}

/* ── Loading ── */
.loading-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 70vh;
  gap: 1.5rem;
}
.loading-state p {
  color: var(--color-text-muted);
  font-size: 1.1rem;
  font-family: var(--font-heading);
  font-size: 1.5rem;
  letter-spacing: 2px;
}
.spinner {
  width: 52px;
  height: 52px;
  border: 4px solid rgba(255,255,255,0.08);
  border-top-color: var(--color-primary);
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }

/* ── Error ── */
.error-state {
  text-align: center;
  padding: 4rem 2rem;
  border-radius: 12px;
  max-width: 500px;
  margin: 10vh auto;
}
.error-state h2 {
  color: var(--color-primary);
  margin-bottom: 1rem;
}
.error-state p {
  color: var(--color-text-muted);
  margin-bottom: 2rem;
}
</style>
