<template>
  <div class="game-container">
    <h1 class="game-title">🐍 贪吃蛇游戏</h1>
    <SnakeGame />
  </div>
</template>

<script setup lang="ts">
import { useHead } from '#app'

useHead({
  title: '贪吃蛇游戏',
  meta: [
    { name: 'description', content: '使用 Nuxt 3 开发的贪吃蛇游戏' }
  ]
})
</script>

<style scoped>
.game-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
}

.game-title {
  font-size: 3rem;
  color: white;
  margin-bottom: 2rem;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

@media (max-width: 768px) {
  .game-title {
    font-size: 2rem;
  }
}
</style>
