<template>
  <section class="hero-section">
    <div class="hero-content">
      <h1 class="animate-fade-in-down">Set Sail for the Grand Line!</h1>
      <p class="animate-fade-in-up">Join Monkey D. Luffy and the Straw Hat Pirates on their epic quest to find the ultimate treasure and become the Pirate King!</p>
      <div class="hero-buttons animate-fade-in-up" style="animation-delay: 0.2s">
        <NuxtLink to="/watch/one-piece" class="btn btn-primary">Watch OP!</NuxtLink>
        <NuxtLink to="/crew" class="btn btn-secondary">Meet the Crew</NuxtLink>
      </div>
    </div>
  </section>
</template>

<style scoped>
.hero-section {
  height: 100vh;
  min-height: 600px;
  background-image: linear-gradient(rgba(15, 23, 42, 0.6), rgba(15, 23, 42, 0.9)), url('/images/hero_bg.png');
  background-size: cover;
  background-position: center;
  background-attachment: fixed;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 2rem;
}

.hero-content {
  max-width: 800px;
}

h1 {
  font-size: clamp(3rem, 8vw, 6rem);
  color: var(--color-accent);
  margin-bottom: 1rem;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
}

p {
  font-size: clamp(1.1rem, 2.5vw, 1.35rem);
  margin-bottom: 2.5rem;
  color: var(--color-text);
  opacity: 0.9;
  line-height: 1.7;
}

.hero-buttons {
  display: flex;
  gap: 1.25rem;
  justify-content: center;
  flex-wrap: wrap;
}

.btn-secondary {
  background: transparent;
  color: var(--color-text);
  border: 2px solid var(--color-accent);
  box-shadow: 0 4px 15px rgba(250, 204, 21, 0.15);
}

.btn-secondary:hover {
  background: rgba(250, 204, 21, 0.1);
  box-shadow: 0 6px 20px rgba(250, 204, 21, 0.3);
}
</style>
