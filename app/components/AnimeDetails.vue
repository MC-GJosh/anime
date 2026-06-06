<template>
  <div class="anime-details">

    <!-- Top: Poster + Key Info -->
    <div class="hero-row">
      <div class="poster-wrapper" v-if="anime.image">
        <img :src="anime.image" :alt="anime.title" class="poster" />
      </div>

      <div class="info">
        <div class="title-row">
          <h1 class="title">{{ anime.title }}</h1>
        </div>

        <div class="meta">
          <span class="rating">★ {{ anime.rating }}</span>
          <span class="sep">•</span>
          <span>{{ anime.year }}</span>
          <span class="sep">•</span>
          <span class="status" :class="{ airing: anime.status === 'Currently Airing' }">
            {{ anime.status }}
          </span>
          <span class="sep">•</span>
          <span class="type-badge">{{ anime.type }}</span>
        </div>

        <!-- Stats row -->
        <div class="stats">
          <div class="stat" v-if="anime.rank">
            <span class="stat-label">Ranked</span>
            <span class="stat-value">#{{ anime.rank }}</span>
          </div>
          <div class="stat" v-if="anime.popularity">
            <span class="stat-label">Popularity</span>
            <span class="stat-value">#{{ anime.popularity }}</span>
          </div>
          <div class="stat" v-if="anime.members">
            <span class="stat-label">Members</span>
            <span class="stat-value">{{ formatNumber(anime.members) }}</span>
          </div>
          <div class="stat" v-if="anime.studio">
            <span class="stat-label">Studio</span>
            <span class="stat-value">{{ anime.studio }}</span>
          </div>
          <div class="stat" v-if="anime.duration">
            <span class="stat-label">Duration</span>
            <span class="stat-value">{{ anime.duration }}</span>
          </div>
        </div>

        <!-- Genres -->
        <div class="genres">
          <span v-for="genre in anime.genres" :key="genre" class="genre-tag">{{ genre }}</span>
        </div>

        <!-- Synopsis -->
        <div class="synopsis">
          <h3>Synopsis</h3>
          <p>{{ anime.synopsis }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  anime: { type: Object, required: true }
})

function formatNumber(n) {
  if (n >= 1_000_000) return (n / 1_000_000).toFixed(1) + 'M'
  if (n >= 1_000)     return (n / 1_000).toFixed(0) + 'K'
  return n
}
</script>

<style scoped>
.anime-details {
  background: var(--color-bg-card);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 12px;
  padding: 2.5rem;
  animation: fadeInUp 0.6s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}

@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: translateY(0); }
}

.hero-row {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  align-items: flex-start;
}

/* Poster */
.poster-wrapper { width: 100%; }
.poster {
  width: 100%;
  max-width: 260px;
  border-radius: 8px;
  box-shadow: 0 10px 40px rgba(0,0,0,0.6);
  border: 1px solid rgba(255,255,255,0.1);
  display: block;
}

/* Info */
.info { flex: 1; min-width: 0; width: 100%; }

.title-row { margin-bottom: 0.75rem; }
.title {
  font-size: clamp(2.2rem, 4vw, 3.5rem);
  color: var(--color-text);
  text-shadow: 2px 2px 10px rgba(0,0,0,0.4);
  line-height: 1.1;
}

/* Meta */
.meta {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-bottom: 1.5rem;
  color: var(--color-text-muted);
  font-size: 0.95rem;
}
.sep { opacity: 0.3; }
.rating { color: var(--color-accent); font-weight: 700; }
.status.airing { color: #34d399; }
.type-badge {
  background: rgba(255,255,255,0.08);
  padding: 0.15rem 0.6rem;
  border-radius: 4px;
  font-size: 0.8rem;
}

/* Stats */
.stats {
  display: flex;
  gap: 2rem;
  flex-wrap: wrap;
  margin-bottom: 1.5rem;
  padding: 1.25rem 0;
  border-top: 1px solid rgba(255,255,255,0.06);
  border-bottom: 1px solid rgba(255,255,255,0.06);
}
.stat { display: flex; flex-direction: column; gap: 0.2rem; }
.stat-label {
  font-size: 0.7rem;
  text-transform: uppercase;
  letter-spacing: 1px;
  color: var(--color-text-muted);
}
.stat-value {
  font-size: 1rem;
  font-weight: 700;
  color: var(--color-text);
}

/* Genres */
.genres {
  display: flex;
  flex-wrap: wrap;
  gap: 0.6rem;
  margin-bottom: 1.75rem;
}
.genre-tag {
  background: rgba(255,255,255,0.05);
  padding: 0.35rem 0.9rem;
  border-radius: 20px;
  font-size: 0.85rem;
  color: var(--color-text);
  border: 1px solid rgba(255,255,255,0.1);
  transition: all 0.2s;
}
.genre-tag:hover {
  border-color: var(--color-accent);
  background: rgba(250,204,21,0.06);
}

/* Synopsis */
.synopsis h3 {
  font-size: 1.6rem;
  color: var(--color-primary);
  margin-bottom: 0.75rem;
}
.synopsis p {
  line-height: 1.85;
  color: var(--color-text-muted);
  font-size: 1rem;
}

/* Mobile */
@media (max-width: 700px) {
  .hero-row {
    flex-direction: column;
    align-items: center;
    text-align: center;
  }
  .poster { width: 160px; }
  .meta, .genres, .stats { justify-content: center; }
  .synopsis p { text-align: left; }
}
</style>
