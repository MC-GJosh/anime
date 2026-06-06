<template>
  <div class="video-player-wrapper glass">
    <!-- Video Element (hidden until play is clicked) -->
    <div class="video-container">
      <video
        v-if="isPlaying"
        ref="videoRef"
        class="actual-video"
        controls
        autoplay
        :poster="`https://placehold.co/1920x1080/0f172a/ef4444?text=${encodeURIComponent(episodeTitle)}`"
      >
        <!-- Sample open-source video for demo purposes -->
        <source src="https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4" type="video/mp4" />
        Your browser does not support the video tag.
      </video>

      <!-- Placeholder shown before play -->
      <div v-else class="video-placeholder" @click="startPlaying">
        <div class="ambient-glow"></div>
        <div class="thumbnail-overlay">
          <img
            v-if="posterImage"
            :src="posterImage"
            :alt="episodeTitle"
            class="poster-bg"
          />
          <div class="dark-overlay"></div>
        </div>
        <div class="placeholder-content">
          <button class="play-btn" aria-label="Play Episode">
            <svg viewBox="0 0 24 24" fill="currentColor" width="40" height="40">
              <path d="M8 5v14l11-7z"/>
            </svg>
          </button>
          <p class="now-playing-label">Click to Play</p>
          <h2 class="ep-display">{{ episodeTitle }}</h2>
        </div>
      </div>
    </div>

    <!-- Controls Bar -->
    <div class="video-controls">
      <h2 class="episode-title">{{ episodeTitle }}</h2>
      <div class="actions">
        <button class="ctrl-btn" :disabled="!hasPrev" @click="onPrev">
          <svg viewBox="0 0 24 24" fill="currentColor" width="18" height="18"><path d="M6 6h2v12H6zm3.5 6l8.5 6V6z"/></svg>
          Prev
        </button>
        <button class="ctrl-btn primary" :disabled="!hasNext" @click="onNext">
          Next
          <svg viewBox="0 0 24 24" fill="currentColor" width="18" height="18"><path d="M6 18l8.5-6L6 6v12zM16 6v12h2V6h-2z"/></svg>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  episodeTitle: { type: String, default: 'Episode 1' },
  episodeNumber: { type: Number, default: 1 },
  hasPrev: { type: Boolean, default: false },
  hasNext: { type: Boolean, default: true },
  posterImage: { type: String, default: null },
})

const emit = defineEmits(['prev', 'next'])

const isPlaying = ref(false)
const videoRef = ref(null)

function startPlaying() {
  isPlaying.value = true
}

// Reset player when episode changes
watch(() => props.episodeNumber, () => {
  isPlaying.value = false
})

function onPrev() {
  isPlaying.value = false
  emit('prev')
}

function onNext() {
  isPlaying.value = false
  emit('next')
}
</script>

<style scoped>
.video-player-wrapper {
  border-radius: 12px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
}

.video-container {
  width: 100%;
  aspect-ratio: 16 / 9;
  background: #000;
  position: relative;
  cursor: pointer;
  overflow: hidden;
}

/* Actual video */
.actual-video {
  width: 100%;
  height: 100%;
  object-fit: contain;
  background: #000;
  display: block;
}

/* Placeholder */
.video-placeholder {
  width: 100%;
  height: 100%;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.thumbnail-overlay {
  position: absolute;
  inset: 0;
}

.poster-bg {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: top;
  filter: blur(2px) brightness(0.4);
  transform: scale(1.05);
}

.dark-overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(
    to bottom,
    rgba(15, 23, 42, 0.3) 0%,
    rgba(15, 23, 42, 0.7) 100%
  );
}

.ambient-glow {
  position: absolute;
  inset: 0;
  background: radial-gradient(ellipse at center, rgba(239, 68, 68, 0.1) 0%, transparent 70%);
  pointer-events: none;
  z-index: 1;
}

.placeholder-content {
  position: relative;
  z-index: 2;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  text-align: center;
  padding: 2rem;
}

.play-btn {
  width: 88px;
  height: 88px;
  border-radius: 50%;
  background: rgba(239, 68, 68, 0.2);
  border: 3px solid var(--color-primary);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  backdrop-filter: blur(4px);
}

.play-btn:hover {
  background: rgba(239, 68, 68, 0.5);
  transform: scale(1.15);
  box-shadow: 0 0 50px rgba(239, 68, 68, 0.4), 0 0 0 8px rgba(239, 68, 68, 0.1);
}

.video-placeholder:hover .play-btn {
  box-shadow: 0 0 60px rgba(239, 68, 68, 0.5), 0 0 0 12px rgba(239, 68, 68, 0.1);
}

.now-playing-label {
  color: var(--color-text-muted);
  font-size: 0.85rem;
  text-transform: uppercase;
  letter-spacing: 3px;
  margin: 0;
}

.ep-display {
  color: var(--color-text);
  font-family: var(--font-body);
  font-size: clamp(1rem, 2vw, 1.5rem);
  font-weight: 600;
  max-width: 600px;
  margin: 0;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.8);
}

/* Controls bar */
.video-controls {
  padding: 1.25rem 1.75rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1rem;
  background: rgba(0, 0, 0, 0.3);
  border-top: 1px solid rgba(255, 255, 255, 0.06);
}

.episode-title {
  font-family: var(--font-body);
  font-size: 1rem;
  font-weight: 600;
  margin: 0;
  color: var(--color-text-muted);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 60%;
}

.actions {
  display: flex;
  gap: 0.75rem;
  flex-shrink: 0;
}

.ctrl-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1.25rem;
  border-radius: 6px;
  border: 1px solid rgba(255, 255, 255, 0.15);
  background: rgba(255, 255, 255, 0.05);
  color: var(--color-text);
  font-family: var(--font-body);
  font-size: 0.9rem;
  cursor: pointer;
  transition: all 0.2s ease;
}

.ctrl-btn:hover:not(:disabled) {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(255, 255, 255, 0.3);
}

.ctrl-btn.primary {
  background: var(--color-primary);
  border-color: var(--color-primary);
  color: white;
}

.ctrl-btn.primary:hover:not(:disabled) {
  background: #f87171;
}

.ctrl-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}
</style>
