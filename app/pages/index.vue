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
  background: #0f0f23;
  padding: 20px;
  position: relative;
  overflow: hidden;
}

/* 动态背景效果 */
.game-container::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle at 20% 50%, #2d1b69 0%, transparent 50%),
              radial-gradient(circle at 80% 50%, #0f3460 0%, transparent 50%),
              radial-gradient(circle at 50% 20%, #16213e 0%, transparent 50%);
  animation: backgroundMove 20s ease-in-out infinite;
  opacity: 0.7;
}

@keyframes backgroundMove {
  0%, 100% {
    transform: translate(0, 0) scale(1);
  }
  33% {
    transform: translate(-5%, 5%) scale(1.1);
  }
  66% {
    transform: translate(5%, -5%) scale(0.9);
  }
}

.game-title {
  font-size: 4rem;
  background: linear-gradient(135deg, #60efff 0%, #0061ff 25%, #ff006e 75%, #ffb800 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 3rem;
  font-weight: 800;
  text-align: center;
  position: relative;
  z-index: 1;
  animation: titleGlow 3s ease-in-out infinite;
  letter-spacing: 0.1em;
}

@keyframes titleGlow {
  0%, 100% {
    filter: drop-shadow(0 0 20px rgba(96, 239, 255, 0.5));
    transform: translateY(0);
  }
  50% {
    filter: drop-shadow(0 0 40px rgba(96, 239, 255, 0.8));
    transform: translateY(-5px);
  }
}

@media (max-width: 768px) {
  .game-title {
    font-size: 2.5rem;
  }
}
</style>
