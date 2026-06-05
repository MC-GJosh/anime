<template>
  <section class="hero-section">
    <div class="hero-content">
      <h1 class="animate-fade-in-down">Set Sail for the Grand Line!</h1>
      <p class="animate-fade-in-up">Watch Anime shows for free!</p>
      <a href="#crew" class="btn btn-primary animate-fade-in-up" style="animation-delay: 0.2s">Watch Now!</a>
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
  font-size: clamp(1.2rem, 3vw, 1.5rem);
  margin-bottom: 2.5rem;
  color: var(--color-text);
  opacity: 0.9;
}
</style>
