<template>
  <div class="crew-page">
    <div class="container">

      <!-- Back -->
      <NuxtLink to="/" class="back-link">← Back to Home</NuxtLink>

      <!-- Header -->
      <div class="page-header animate-fade-in-down">
        <h1>The Straw Hat Pirates</h1>
        <p class="subtitle">The crew that will conquer the Grand Line</p>
        <div class="crew-bounty glass" v-if="totalBounty">
          <span class="label">Combined Bounty</span>
          <span class="value">฿ {{ totalBounty }}</span>
        </div>
      </div>

      <!-- Loading -->
      <div v-if="loading" class="loading-state">
        <div class="spinner"></div>
        <p>Gathering the crew…</p>
      </div>

      <!-- Error -->
      <div v-else-if="error" class="error-state glass">
        <h2>⚠️ Couldn't find the crew!</h2>
        <p>{{ error }}</p>
        <button class="btn" @click="fetchCrew">Try Again</button>
      </div>

      <!-- Crew Grid -->
      <div v-else class="crew-grid">
        <div
          v-for="member in crewMembers"
          :key="member.id"
          class="crew-card glass"
        >
          <!-- Avatar placeholder -->
          <div class="avatar-area">
            <div class="avatar-circle" :style="{ background: getColor(member.id) }">
              <span class="avatar-initial">{{ member.name.charAt(0) }}</span>
            </div>
            <span class="role-badge">{{ member.job }}</span>
          </div>

          <!-- Info -->
          <div class="card-body">
            <h2 class="name">{{ member.name }}</h2>

            <div class="stats-row">
              <div class="stat" v-if="member.bounty">
                <span class="stat-label">Bounty</span>
                <span class="stat-value bounty-value">฿ {{ member.bounty }}</span>
              </div>
              <div class="stat" v-if="member.age">
                <span class="stat-label">Age</span>
                <span class="stat-value">{{ member.age }}</span>
              </div>
              <div class="stat" v-if="member.size">
                <span class="stat-label">Height</span>
                <span class="stat-value">{{ member.size }}</span>
              </div>
            </div>

            <!-- Devil Fruit -->
            <div class="fruit-info" v-if="member.fruit">
              <div class="fruit-header">
                <span class="fruit-icon">🍈</span>
                <span class="fruit-name">{{ member.fruit.name }}</span>
              </div>
              <span class="fruit-type" :class="fruitTypeClass(member.fruit.type)">
                {{ member.fruit.type }}
              </span>
              <p class="fruit-desc" v-if="member.fruit.description">
                {{ truncate(member.fruit.description, 120) }}
              </p>
            </div>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const crewMembers = ref([])
const loading = ref(true)
const error = ref(null)

// Straw Hat crew IDs (1-10)
const STRAW_HAT_IDS = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

const totalBounty = computed(() => {
  if (crewMembers.value.length === 0) return null
  // The crew object has total_prime
  const crew = crewMembers.value[0]?.crew
  return crew?.total_prime || null
})

async function fetchCrew() {
  loading.value = true
  error.value = null
  try {
    const res = await fetch('https://api.api-onepiece.com/v2/characters/en')
    if (!res.ok) throw new Error('Failed to fetch characters')
    const data = await res.json()
    // Filter to Straw Hat crew members only (IDs 1-10)
    crewMembers.value = data.filter(c => STRAW_HAT_IDS.includes(c.id))
  } catch (e) {
    error.value = e.message
  } finally {
    loading.value = false
  }
}

function truncate(str, len) {
  if (!str) return ''
  return str.length > len ? str.slice(0, len) + '…' : str
}

function fruitTypeClass(type) {
  if (!type) return ''
  const t = type.toLowerCase()
  if (t.includes('logia')) return 'logia'
  if (t.includes('zoan')) return 'zoan'
  return 'paramecia'
}

// Assign unique accent colors per member
const colors = [
  'linear-gradient(135deg, #ef4444, #dc2626)',
  'linear-gradient(135deg, #22c55e, #16a34a)',
  'linear-gradient(135deg, #f97316, #ea580c)',
  'linear-gradient(135deg, #facc15, #eab308)',
  'linear-gradient(135deg, #3b82f6, #2563eb)',
  'linear-gradient(135deg, #ec4899, #db2777)',
  'linear-gradient(135deg, #8b5cf6, #7c3aed)',
  'linear-gradient(135deg, #06b6d4, #0891b2)',
  'linear-gradient(135deg, #a855f7, #9333ea)',
  'linear-gradient(135deg, #14b8a6, #0d9488)',
]
function getColor(id) {
  return colors[(id - 1) % colors.length]
}

onMounted(fetchCrew)
</script>

<style scoped>
.crew-page {
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

/* Header */
.page-header {
  text-align: center;
  padding: 3rem 0 2.5rem;
}
.page-header h1 {
  font-size: clamp(2.5rem, 6vw, 4.5rem);
  color: var(--color-accent);
  text-shadow: 2px 2px 8px rgba(0,0,0,0.5);
  margin-bottom: 0.75rem;
}
.subtitle {
  color: var(--color-text-muted);
  font-size: 1.15rem;
  margin-bottom: 2rem;
}
.crew-bounty {
  display: inline-flex;
  flex-direction: column;
  padding: 1rem 2rem;
  border-radius: 12px;
  gap: 0.25rem;
}
.crew-bounty .label {
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 1.5px;
  color: var(--color-text-muted);
}
.crew-bounty .value {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--color-accent);
  font-family: var(--font-heading);
  letter-spacing: 1px;
}

/* Grid */
.crew-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: 1.5rem;
  padding-top: 1rem;
}

/* Card */
.crew-card {
  border-radius: 14px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  animation: fadeInUp 0.5s ease-out both;
}
.crew-card:nth-child(1) { animation-delay: 0s; }
.crew-card:nth-child(2) { animation-delay: 0.05s; }
.crew-card:nth-child(3) { animation-delay: 0.1s; }
.crew-card:nth-child(4) { animation-delay: 0.15s; }
.crew-card:nth-child(5) { animation-delay: 0.2s; }
.crew-card:nth-child(6) { animation-delay: 0.25s; }
.crew-card:nth-child(7) { animation-delay: 0.3s; }
.crew-card:nth-child(8) { animation-delay: 0.35s; }
.crew-card:nth-child(9) { animation-delay: 0.4s; }
.crew-card:nth-child(10) { animation-delay: 0.45s; }

@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(24px); }
  to   { opacity: 1; transform: translateY(0); }
}

.crew-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 12px 40px rgba(0,0,0,0.4);
}

/* Avatar */
.avatar-area {
  padding: 2rem 1.5rem 1.25rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
}
.avatar-circle {
  width: 90px;
  height: 90px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 6px 24px rgba(0,0,0,0.3);
  border: 3px solid rgba(255,255,255,0.15);
}
.avatar-initial {
  font-size: 2.5rem;
  font-family: var(--font-heading);
  color: white;
  text-shadow: 0 2px 4px rgba(0,0,0,0.4);
}
.role-badge {
  background: rgba(255,255,255,0.08);
  padding: 0.3rem 1rem;
  border-radius: 20px;
  font-size: 0.8rem;
  color: var(--color-text-muted);
  text-transform: capitalize;
  letter-spacing: 0.5px;
}

/* Body */
.card-body {
  padding: 0 1.5rem 1.75rem;
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
.name {
  font-size: 1.6rem;
  color: var(--color-text);
  text-align: center;
}

/* Stats */
.stats-row {
  display: flex;
  justify-content: center;
  gap: 1.5rem;
  flex-wrap: wrap;
}
.stat {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.15rem;
}
.stat-label {
  font-size: 0.7rem;
  text-transform: uppercase;
  letter-spacing: 1px;
  color: var(--color-text-muted);
}
.stat-value {
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--color-text);
}
.bounty-value {
  color: var(--color-accent);
}

/* Devil Fruit */
.fruit-info {
  background: rgba(0,0,0,0.25);
  padding: 1rem;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
  margin-top: auto;
}
.fruit-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}
.fruit-icon {
  font-size: 1.1rem;
}
.fruit-name {
  font-size: 0.9rem;
  font-weight: 600;
  color: var(--color-text);
}
.fruit-type {
  display: inline-block;
  width: fit-content;
  padding: 0.15rem 0.5rem;
  border-radius: 4px;
  font-size: 0.7rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}
.fruit-type.paramecia {
  background: rgba(139, 92, 246, 0.2);
  color: #a78bfa;
}
.fruit-type.zoan {
  background: rgba(34, 197, 94, 0.2);
  color: #4ade80;
}
.fruit-type.logia {
  background: rgba(59, 130, 246, 0.2);
  color: #60a5fa;
}
.fruit-desc {
  font-size: 0.8rem;
  line-height: 1.5;
  color: var(--color-text-muted);
}

/* Loading */
.loading-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 50vh;
  gap: 1.5rem;
}
.loading-state p {
  color: var(--color-text-muted);
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

/* Error */
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

@media (max-width: 700px) {
  .crew-grid {
    grid-template-columns: 1fr;
  }
}
</style>
