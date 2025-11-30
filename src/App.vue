<script setup>
</script>

<template>
  <div class="app-wrapper">
    <main>
      <router-view />
    </main>
  </div>
</template>

<style scoped>
.app-wrapper {
  min-height: 100vh;
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
}

header {
  height: 3rem;
  display: flex;
  background: linear-gradient(135deg, #0f3460 0%, #16213e 100%);
  color: #fff;
  font-size: 1.2rem;
  justify-content: center;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  position: sticky;
  top: 0;
  z-index: 100;
}

nav {
  column-gap: 2rem;
  margin-bottom: 0;
  display: flex;
  align-items: center;
}

nav a {
  text-decoration: none;
  color: #fff;
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
  transition: all 0.3s ease;
}

nav a:hover {
  background-color: #1a1a2e;
  height: 30px;
  padding-top: 0.5px;
}

nav a.router-link-active {
  background: linear-gradient(135deg, #e94560 0%, #ff6b88 100%);
  height: 30px;
  padding-top: 2px;
  font-weight: 600;
}

main {
  background: transparent;
  min-height: calc(100vh - 3rem);
}
</style>
