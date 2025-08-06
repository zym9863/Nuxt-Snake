<template>
  <div class="snake-game">
    <div class="game-info">
      <div class="info-card score-card">
        <span class="info-label">得分</span>
        <span class="info-value">{{ score }}</span>
      </div>
      <div class="info-card high-score-card">
        <span class="info-label">最高分</span>
        <span class="info-value">{{ highScore }}</span>
      </div>
      <div class="info-card speed-card">
        <span class="info-label">速度</span>
        <span class="info-value">{{ Math.floor((INITIAL_SPEED - gameSpeed + 50) / INITIAL_SPEED * 10) }}</span>
      </div>
    </div>
    
    <div class="game-board" ref="gameBoard">
      <canvas 
        ref="canvas" 
        :width="boardWidth" 
        :height="boardHeight"
        @keydown="handleKeyPress"
        tabindex="0"
      ></canvas>
      
      <div v-if="gameState === 'waiting'" class="game-overlay">
        <div class="overlay-content">
          <div class="snake-icon">🐍</div>
          <h2 class="overlay-title">贪吃蛇</h2>
          <p class="overlay-text pulse">按空格键开始游戏</p>
          <div class="controls-hint">
            <p>使用方向键或 WASD 控制方向</p>
            <p>ESC 暂停游戏</p>
          </div>
        </div>
      </div>
      
      <div v-else-if="gameState === 'paused'" class="game-overlay">
        <div class="overlay-content">
          <div class="pause-icon">⏸️</div>
          <h2 class="overlay-title">游戏暂停</h2>
          <p class="overlay-text pulse">按空格键继续</p>
        </div>
      </div>
      
      <div v-else-if="gameState === 'gameover'" class="game-overlay">
        <div class="overlay-content gameover-content">
          <div class="gameover-icon">💀</div>
          <h2 class="overlay-title">游戏结束</h2>
          <div class="final-score">
            <p class="score-label">最终得分</p>
            <p class="score-value">{{ score }}</p>
          </div>
          <p class="overlay-text pulse">按空格键重新开始</p>
          <div v-if="score === highScore && score > 0" class="new-record">🎉 新纪录！</div>
        </div>
      </div>
    </div>
    
    <div class="game-controls">
      <button @click="toggleGame" class="control-btn" :class="{ 'btn-playing': gameState === 'playing' }">
        <span class="btn-icon">{{ gameState === 'playing' ? '⏸️' : gameState === 'paused' ? '▶️' : '🎮' }}</span>
        <span class="btn-text">{{ gameState === 'playing' ? '暂停' : gameState === 'paused' ? '继续' : '开始' }}</span>
      </button>
      <button @click="resetGame" class="control-btn btn-reset">
        <span class="btn-icon">🔄</span>
        <span class="btn-text">重置</span>
      </button>
    </div>
    
    <div class="mobile-controls">
      <button @click="changeDirection('up')" class="arrow-btn">↑</button>
      <div class="horizontal-controls">
        <button @click="changeDirection('left')" class="arrow-btn">←</button>
        <button @click="changeDirection('down')" class="arrow-btn">↓</button>
        <button @click="changeDirection('right')" class="arrow-btn">→</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue'

// 游戏配置
const GRID_SIZE = 20
const CELL_SIZE = 20
const INITIAL_SPEED = 150
const SPEED_INCREMENT = 5

// 类型定义
type Direction = 'up' | 'down' | 'left' | 'right'
type GameState = 'waiting' | 'playing' | 'paused' | 'gameover'

interface Position {
  x: number
  y: number
}

// 响应式状态
const canvas = ref<HTMLCanvasElement>()
const gameBoard = ref<HTMLDivElement>()
const snake = ref<Position[]>([{ x: 10, y: 10 }])
const food = ref<Position>({ x: 15, y: 15 })
const direction = ref<Direction>('right')
const nextDirection = ref<Direction>('right')
const score = ref(0)
const highScore = ref(0)
const gameState = ref<GameState>('waiting')
const gameLoop = ref<number | null>(null)

// 计算属性
const boardWidth = computed(() => GRID_SIZE * CELL_SIZE)
const boardHeight = computed(() => GRID_SIZE * CELL_SIZE)
const gameSpeed = computed(() => Math.max(50, INITIAL_SPEED - Math.floor(score.value / 5) * SPEED_INCREMENT))

// 生命周期
onMounted(() => {
  // 加载最高分
  const saved = localStorage.getItem('snakeHighScore')
  if (saved) {
    highScore.value = parseInt(saved)
  }
  
  // 添加键盘事件监听
  window.addEventListener('keydown', handleKeyPress)
  
  // 聚焦画布
  if (canvas.value) {
    canvas.value.focus()
  }
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyPress)
  if (gameLoop.value) {
    clearInterval(gameLoop.value)
  }
})

// 游戏逻辑
function startGame() {
  if (gameState.value === 'playing') return
  
  gameState.value = 'playing'
  if (gameLoop.value) {
    clearInterval(gameLoop.value)
  }
  
  gameLoop.value = setInterval(() => {
    updateGame()
  }, gameSpeed.value) as unknown as number
}

function pauseGame() {
  if (gameState.value !== 'playing') return
  
  gameState.value = 'paused'
  if (gameLoop.value) {
    clearInterval(gameLoop.value)
    gameLoop.value = null
  }
}

function resetGame() {
  if (gameLoop.value) {
    clearInterval(gameLoop.value)
    gameLoop.value = null
  }
  
  snake.value = [{ x: 10, y: 10 }]
  direction.value = 'right'
  nextDirection.value = 'right'
  score.value = 0
  gameState.value = 'waiting'
  generateFood()
  draw()
}

function toggleGame() {
  switch (gameState.value) {
    case 'waiting':
    case 'gameover':
      resetGame()
      startGame()
      break
    case 'playing':
      pauseGame()
      break
    case 'paused':
      startGame()
      break
  }
}

function updateGame() {
  // 更新方向
  direction.value = nextDirection.value
  
  // 计算新的头部位置
  const head = { ...snake.value[0] }
  
  switch (direction.value) {
    case 'up':
      head.y -= 1
      break
    case 'down':
      head.y += 1
      break
    case 'left':
      head.x -= 1
      break
    case 'right':
      head.x += 1
      break
  }
  
  // 检查碰撞
  if (checkCollision(head)) {
    endGame()
    return
  }
  
  // 将新头部添加到蛇身
  snake.value.unshift(head)
  
  // 检查是否吃到食物
  if (head.x === food.value.x && head.y === food.value.y) {
    score.value += 10
    
    // 更新最高分
    if (score.value > highScore.value) {
      highScore.value = score.value
      localStorage.setItem('snakeHighScore', highScore.value.toString())
    }
    
    // 生成新食物
    generateFood()
    
    // 调整游戏速度
    if (gameLoop.value) {
      clearInterval(gameLoop.value)
      gameLoop.value = setInterval(() => {
        updateGame()
      }, gameSpeed.value) as unknown as number
    }
  } else {
    // 如果没有吃到食物，移除尾部
    snake.value.pop()
  }
  
  // 绘制游戏
  draw()
}

function checkCollision(head: Position): boolean {
  // 检查墙壁碰撞
  if (head.x < 0 || head.x >= GRID_SIZE || head.y < 0 || head.y >= GRID_SIZE) {
    return true
  }
  
  // 检查自身碰撞
  for (let i = 1; i < snake.value.length; i++) {
    if (head.x === snake.value[i].x && head.y === snake.value[i].y) {
      return true
    }
  }
  
  return false
}

function generateFood() {
  let newFood: Position
  
  do {
    newFood = {
      x: Math.floor(Math.random() * GRID_SIZE),
      y: Math.floor(Math.random() * GRID_SIZE)
    }
  } while (snake.value.some(segment => segment.x === newFood.x && segment.y === newFood.y))
  
  food.value = newFood
}

function endGame() {
  gameState.value = 'gameover'
  if (gameLoop.value) {
    clearInterval(gameLoop.value)
    gameLoop.value = null
  }
}

function changeDirection(newDirection: Direction) {
  if (gameState.value !== 'playing') return
  
  // 防止反向移动
  const opposites: Record<Direction, Direction> = {
    up: 'down',
    down: 'up',
    left: 'right',
    right: 'left'
  }
  
  if (opposites[newDirection] !== direction.value) {
    nextDirection.value = newDirection
  }
}

function handleKeyPress(event: KeyboardEvent) {
  switch (event.key) {
    case ' ':
    case 'Space':
      event.preventDefault()
      toggleGame()
      break
    case 'ArrowUp':
    case 'w':
    case 'W':
      event.preventDefault()
      changeDirection('up')
      break
    case 'ArrowDown':
    case 's':
    case 'S':
      event.preventDefault()
      changeDirection('down')
      break
    case 'ArrowLeft':
    case 'a':
    case 'A':
      event.preventDefault()
      changeDirection('left')
      break
    case 'ArrowRight':
    case 'd':
    case 'D':
      event.preventDefault()
      changeDirection('right')
      break
    case 'Escape':
      event.preventDefault()
      if (gameState.value === 'playing') {
        pauseGame()
      }
      break
  }
}

function draw() {
  if (!canvas.value) return
  
  const ctx = canvas.value.getContext('2d')
  if (!ctx) return
  
  // 创建渐变背景
  const gradient = ctx.createLinearGradient(0, 0, boardWidth.value, boardHeight.value)
  gradient.addColorStop(0, '#1a1a2e')
  gradient.addColorStop(1, '#16213e')
  ctx.fillStyle = gradient
  ctx.fillRect(0, 0, boardWidth.value, boardHeight.value)
  
  // 绘制点阵网格
  ctx.fillStyle = 'rgba(255, 255, 255, 0.03)'
  for (let i = 0; i < GRID_SIZE; i++) {
    for (let j = 0; j < GRID_SIZE; j++) {
      ctx.beginPath()
      ctx.arc(
        i * CELL_SIZE + CELL_SIZE / 2,
        j * CELL_SIZE + CELL_SIZE / 2,
        1,
        0,
        Math.PI * 2
      )
      ctx.fill()
    }
  }
  
  // 绘制蛇
  snake.value.forEach((segment, index) => {
    if (index === 0) {
      // 蛇头 - 圆角矩形with发光效果
      ctx.shadowBlur = 10
      ctx.shadowColor = '#4ade80'
      
      const headGradient = ctx.createRadialGradient(
        segment.x * CELL_SIZE + CELL_SIZE / 2,
        segment.y * CELL_SIZE + CELL_SIZE / 2,
        0,
        segment.x * CELL_SIZE + CELL_SIZE / 2,
        segment.y * CELL_SIZE + CELL_SIZE / 2,
        CELL_SIZE / 2
      )
      headGradient.addColorStop(0, '#86efac')
      headGradient.addColorStop(1, '#4ade80')
      ctx.fillStyle = headGradient
      
      // 绘制圆角矩形
      ctx.beginPath()
      ctx.roundRect(
        segment.x * CELL_SIZE + 2,
        segment.y * CELL_SIZE + 2,
        CELL_SIZE - 4,
        CELL_SIZE - 4,
        4
      )
      ctx.fill()
      
      ctx.shadowBlur = 0
      
      // 绘制眼睛 - 更生动的眼睛
      ctx.fillStyle = '#1a1a2e'
      const eyeSize = 4
      const eyeOffset = 6
      const pupilSize = 2
      
      let eye1x, eye1y, eye2x, eye2y
      
      switch (direction.value) {
        case 'up':
          eye1x = segment.x * CELL_SIZE + eyeOffset
          eye1y = segment.y * CELL_SIZE + eyeOffset
          eye2x = segment.x * CELL_SIZE + CELL_SIZE - eyeOffset
          eye2y = segment.y * CELL_SIZE + eyeOffset
          break
        case 'down':
          eye1x = segment.x * CELL_SIZE + eyeOffset
          eye1y = segment.y * CELL_SIZE + CELL_SIZE - eyeOffset
          eye2x = segment.x * CELL_SIZE + CELL_SIZE - eyeOffset
          eye2y = segment.y * CELL_SIZE + CELL_SIZE - eyeOffset
          break
        case 'left':
          eye1x = segment.x * CELL_SIZE + eyeOffset
          eye1y = segment.y * CELL_SIZE + eyeOffset
          eye2x = segment.x * CELL_SIZE + eyeOffset
          eye2y = segment.y * CELL_SIZE + CELL_SIZE - eyeOffset
          break
        case 'right':
          eye1x = segment.x * CELL_SIZE + CELL_SIZE - eyeOffset
          eye1y = segment.y * CELL_SIZE + eyeOffset
          eye2x = segment.x * CELL_SIZE + CELL_SIZE - eyeOffset
          eye2y = segment.y * CELL_SIZE + CELL_SIZE - eyeOffset
          break
      }
      
      // 绘制眼白
      ctx.fillStyle = '#ffffff'
      ctx.beginPath()
      ctx.arc(eye1x, eye1y, eyeSize, 0, Math.PI * 2)
      ctx.fill()
      ctx.beginPath()
      ctx.arc(eye2x, eye2y, eyeSize, 0, Math.PI * 2)
      ctx.fill()
      
      // 绘制瞳孔
      ctx.fillStyle = '#1a1a2e'
      ctx.beginPath()
      ctx.arc(eye1x, eye1y, pupilSize, 0, Math.PI * 2)
      ctx.fill()
      ctx.beginPath()
      ctx.arc(eye2x, eye2y, pupilSize, 0, Math.PI * 2)
      ctx.fill()
    } else {
      // 蛇身 - 渐变效果with圆角
      const bodyGradient = ctx.createRadialGradient(
        segment.x * CELL_SIZE + CELL_SIZE / 2,
        segment.y * CELL_SIZE + CELL_SIZE / 2,
        0,
        segment.x * CELL_SIZE + CELL_SIZE / 2,
        segment.y * CELL_SIZE + CELL_SIZE / 2,
        CELL_SIZE / 2
      )
      
      // 根据位置调整颜色深浅
      const intensity = 1 - (index / snake.value.length) * 0.4
      bodyGradient.addColorStop(0, `rgba(74, 222, 128, ${intensity})`)
      bodyGradient.addColorStop(1, `rgba(34, 197, 94, ${intensity})`)
      ctx.fillStyle = bodyGradient
      
      ctx.beginPath()
      ctx.roundRect(
        segment.x * CELL_SIZE + 3,
        segment.y * CELL_SIZE + 3,
        CELL_SIZE - 6,
        CELL_SIZE - 6,
        3
      )
      ctx.fill()
    }
  })
  
  // 绘制食物 - 带动画效果的苹果
  const time = Date.now() / 1000
  const pulseScale = 1 + Math.sin(time * 5) * 0.1
  
  ctx.save()
  ctx.translate(
    food.value.x * CELL_SIZE + CELL_SIZE / 2,
    food.value.y * CELL_SIZE + CELL_SIZE / 2
  )
  ctx.scale(pulseScale, pulseScale)
  
  // 发光效果
  ctx.shadowBlur = 15
  ctx.shadowColor = '#ef4444'
  
  // 主体
  const foodGradient = ctx.createRadialGradient(0, 0, 0, 0, 0, CELL_SIZE / 2 - 2)
  foodGradient.addColorStop(0, '#fca5a5')
  foodGradient.addColorStop(0.5, '#ef4444')
  foodGradient.addColorStop(1, '#dc2626')
  ctx.fillStyle = foodGradient
  
  ctx.beginPath()
  ctx.arc(0, 0, CELL_SIZE / 2 - 2, 0, Math.PI * 2)
  ctx.fill()
  
  ctx.shadowBlur = 0
  
  // 高光效果
  ctx.fillStyle = 'rgba(255, 255, 255, 0.4)'
  ctx.beginPath()
  ctx.arc(-3, -3, CELL_SIZE / 4, 0, Math.PI * 2)
  ctx.fill()
  
  ctx.restore()
  
  // 持续重绘以实现动画
  if (gameState.value === 'playing') {
    requestAnimationFrame(() => draw())
  }
}

// 初始化绘制
onMounted(() => {
  generateFood()
  draw()
})
</script>

<style scoped>
.snake-game {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
  position: relative;
  z-index: 1;
}

/* 游戏信息卡片 */
.game-info {
  display: flex;
  gap: 1.5rem;
  animation: slideDown 0.5s ease-out;
}

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.info-card {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.1) 0%, rgba(255, 255, 255, 0.05) 100%);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
  padding: 0.75rem 1.5rem;
  backdrop-filter: blur(20px);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.25rem;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.info-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.5), transparent);
  animation: shimmer 3s infinite;
}

@keyframes shimmer {
  0% {
    transform: translateX(-100%);
  }
  100% {
    transform: translateX(100%);
  }
}

.info-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
}

.info-label {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.7);
  text-transform: uppercase;
  letter-spacing: 0.1em;
}

.info-value {
  font-size: 1.5rem;
  font-weight: bold;
  color: #ffffff;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

.score-card .info-value {
  color: #60efff;
}

.high-score-card .info-value {
  color: #ffd700;
}

.speed-card .info-value {
  color: #ff6ec7;
}

/* 游戏板 */
.game-board {
  position: relative;
  border: 2px solid transparent;
  background: linear-gradient(#0f0f23, #0f0f23) padding-box,
              linear-gradient(135deg, #60efff, #ff006e) border-box;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5),
              0 0 60px rgba(96, 239, 255, 0.2);
  animation: fadeIn 0.8s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: scale(0.9);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

canvas {
  display: block;
  outline: none;
}

/* 游戏覆盖层 */
.game-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: radial-gradient(circle at center, rgba(0, 0, 0, 0.9) 0%, rgba(0, 0, 0, 0.95) 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  backdrop-filter: blur(10px);
  animation: overlayFade 0.3s ease-out;
}

@keyframes overlayFade {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.overlay-content {
  text-align: center;
  animation: zoomIn 0.5s ease-out;
}

@keyframes zoomIn {
  from {
    opacity: 0;
    transform: scale(0.8);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

.snake-icon, .pause-icon, .gameover-icon {
  font-size: 4rem;
  margin-bottom: 1rem;
  animation: bounce 2s ease-in-out infinite;
}

@keyframes bounce {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-10px);
  }
}

.overlay-title {
  font-size: 2.5rem;
  font-weight: 800;
  background: linear-gradient(135deg, #60efff, #ff006e);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 1.5rem;
  letter-spacing: 0.05em;
}

.overlay-text {
  font-size: 1.2rem;
  color: rgba(255, 255, 255, 0.9);
  margin: 1rem 0;
}

.pulse {
  animation: pulse 1.5s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}

.controls-hint {
  margin-top: 2rem;
  padding: 1rem;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.controls-hint p {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.7);
  margin: 0.5rem 0;
}

.final-score {
  margin: 1.5rem 0;
}

.score-label {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.7);
  margin-bottom: 0.5rem;
}

.score-value {
  font-size: 3rem;
  font-weight: bold;
  color: #ffd700;
  text-shadow: 0 4px 8px rgba(255, 215, 0, 0.4);
}

.new-record {
  margin-top: 1rem;
  padding: 0.5rem 1rem;
  background: linear-gradient(135deg, #ffd700, #ffed4e);
  color: #1a1a2e;
  border-radius: 20px;
  font-weight: bold;
  animation: recordPulse 1s ease-in-out infinite;
}

@keyframes recordPulse {
  0%, 100% {
    transform: scale(1);
    box-shadow: 0 0 20px rgba(255, 215, 0, 0.5);
  }
  50% {
    transform: scale(1.05);
    box-shadow: 0 0 30px rgba(255, 215, 0, 0.8);
  }
}

/* 控制按钮 */
.game-controls {
  display: flex;
  gap: 1rem;
  animation: slideUp 0.5s ease-out;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.control-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.75rem 1.5rem;
  font-size: 1rem;
  font-weight: 600;
  color: white;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border: none;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.control-btn::before {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 0;
  height: 0;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.2);
  transform: translate(-50%, -50%);
  transition: width 0.6s, height 0.6s;
}

.control-btn:hover::before {
  width: 300px;
  height: 300px;
}

.control-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 30px rgba(102, 126, 234, 0.4);
}

.control-btn:active {
  transform: translateY(0);
}

.btn-playing {
  background: linear-gradient(135deg, #f97316 0%, #ea580c 100%);
}

.btn-reset {
  background: linear-gradient(135deg, #6b7280 0%, #4b5563 100%);
}

.btn-icon {
  font-size: 1.2rem;
  position: relative;
  z-index: 1;
}

.btn-text {
  position: relative;
  z-index: 1;
}

/* 移动端控制 */
.mobile-controls {
  display: none;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  margin-top: 1.5rem;
}

.horizontal-controls {
  display: flex;
  gap: 0.5rem;
}

.arrow-btn {
  width: 60px;
  height: 60px;
  font-size: 1.8rem;
  font-weight: bold;
  color: white;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.15) 0%, rgba(255, 255, 255, 0.05) 100%);
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
  backdrop-filter: blur(15px);
  display: flex;
  align-items: center;
  justify-content: center;
}

.arrow-btn:hover {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.25) 0%, rgba(255, 255, 255, 0.1) 100%);
  transform: scale(1.05);
}

.arrow-btn:active {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.35) 0%, rgba(255, 255, 255, 0.15) 100%);
  transform: scale(0.95);
  border-color: rgba(255, 255, 255, 0.5);
}

@media (max-width: 768px) {
  .mobile-controls {
    display: flex;
  }
  
  .game-info {
    gap: 1rem;
  }
  
  .info-card {
    padding: 0.5rem 1rem;
  }
  
  .info-label {
    font-size: 0.65rem;
  }
  
  .info-value {
    font-size: 1.2rem;
  }
  
  .control-btn {
    padding: 0.6rem 1.2rem;
    font-size: 0.9rem;
  }
  
  .overlay-title {
    font-size: 2rem;
  }
  
  .snake-icon, .pause-icon, .gameover-icon {
    font-size: 3rem;
  }
}
</style>
