<template>
  <div class="snake-game">
    <div class="game-info">
      <div class="score">得分: {{ score }}</div>
      <div class="high-score">最高分: {{ highScore }}</div>
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
        <h2>🐍 贪吃蛇</h2>
        <p>按空格键开始游戏</p>
        <p class="controls">使用方向键或 WASD 控制方向</p>
      </div>
      
      <div v-else-if="gameState === 'paused'" class="game-overlay">
        <h2>⏸️ 游戏暂停</h2>
        <p>按空格键继续</p>
      </div>
      
      <div v-else-if="gameState === 'gameover'" class="game-overlay">
        <h2>💀 游戏结束</h2>
        <p>最终得分: {{ score }}</p>
        <p>按空格键重新开始</p>
      </div>
    </div>
    
    <div class="game-controls">
      <button @click="toggleGame" class="control-btn">
        {{ gameState === 'playing' ? '暂停' : gameState === 'paused' ? '继续' : '开始' }}
      </button>
      <button @click="resetGame" class="control-btn">重置</button>
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
  
  // 清空画布
  ctx.fillStyle = '#2a2a2a'
  ctx.fillRect(0, 0, boardWidth.value, boardHeight.value)
  
  // 绘制网格
  ctx.strokeStyle = '#3a3a3a'
  ctx.lineWidth = 1
  for (let i = 0; i <= GRID_SIZE; i++) {
    ctx.beginPath()
    ctx.moveTo(i * CELL_SIZE, 0)
    ctx.lineTo(i * CELL_SIZE, boardHeight.value)
    ctx.stroke()
    
    ctx.beginPath()
    ctx.moveTo(0, i * CELL_SIZE)
    ctx.lineTo(boardWidth.value, i * CELL_SIZE)
    ctx.stroke()
  }
  
  // 绘制蛇
  snake.value.forEach((segment, index) => {
    if (index === 0) {
      // 蛇头
      ctx.fillStyle = '#4ade80'
      ctx.fillRect(
        segment.x * CELL_SIZE + 2,
        segment.y * CELL_SIZE + 2,
        CELL_SIZE - 4,
        CELL_SIZE - 4
      )
      
      // 绘制眼睛
      ctx.fillStyle = '#ffffff'
      const eyeSize = 3
      const eyeOffset = 5
      
      switch (direction.value) {
        case 'up':
          ctx.fillRect(segment.x * CELL_SIZE + eyeOffset, segment.y * CELL_SIZE + eyeOffset, eyeSize, eyeSize)
          ctx.fillRect(segment.x * CELL_SIZE + CELL_SIZE - eyeOffset - eyeSize, segment.y * CELL_SIZE + eyeOffset, eyeSize, eyeSize)
          break
        case 'down':
          ctx.fillRect(segment.x * CELL_SIZE + eyeOffset, segment.y * CELL_SIZE + CELL_SIZE - eyeOffset - eyeSize, eyeSize, eyeSize)
          ctx.fillRect(segment.x * CELL_SIZE + CELL_SIZE - eyeOffset - eyeSize, segment.y * CELL_SIZE + CELL_SIZE - eyeOffset - eyeSize, eyeSize, eyeSize)
          break
        case 'left':
          ctx.fillRect(segment.x * CELL_SIZE + eyeOffset, segment.y * CELL_SIZE + eyeOffset, eyeSize, eyeSize)
          ctx.fillRect(segment.x * CELL_SIZE + eyeOffset, segment.y * CELL_SIZE + CELL_SIZE - eyeOffset - eyeSize, eyeSize, eyeSize)
          break
        case 'right':
          ctx.fillRect(segment.x * CELL_SIZE + CELL_SIZE - eyeOffset - eyeSize, segment.y * CELL_SIZE + eyeOffset, eyeSize, eyeSize)
          ctx.fillRect(segment.x * CELL_SIZE + CELL_SIZE - eyeOffset - eyeSize, segment.y * CELL_SIZE + CELL_SIZE - eyeOffset - eyeSize, eyeSize, eyeSize)
          break
      }
    } else {
      // 蛇身
      const gradient = ctx.createLinearGradient(
        segment.x * CELL_SIZE,
        segment.y * CELL_SIZE,
        (segment.x + 1) * CELL_SIZE,
        (segment.y + 1) * CELL_SIZE
      )
      gradient.addColorStop(0, '#22c55e')
      gradient.addColorStop(1, '#16a34a')
      ctx.fillStyle = gradient
      
      ctx.fillRect(
        segment.x * CELL_SIZE + 2,
        segment.y * CELL_SIZE + 2,
        CELL_SIZE - 4,
        CELL_SIZE - 4
      )
    }
  })
  
  // 绘制食物
  ctx.fillStyle = '#ef4444'
  ctx.beginPath()
  ctx.arc(
    food.value.x * CELL_SIZE + CELL_SIZE / 2,
    food.value.y * CELL_SIZE + CELL_SIZE / 2,
    CELL_SIZE / 2 - 2,
    0,
    Math.PI * 2
  )
  ctx.fill()
  
  // 食物高光效果
  ctx.fillStyle = 'rgba(255, 255, 255, 0.3)'
  ctx.beginPath()
  ctx.arc(
    food.value.x * CELL_SIZE + CELL_SIZE / 2 - 3,
    food.value.y * CELL_SIZE + CELL_SIZE / 2 - 3,
    CELL_SIZE / 4,
    0,
    Math.PI * 2
  )
  ctx.fill()
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
  gap: 1rem;
}

.game-info {
  display: flex;
  gap: 2rem;
  font-size: 1.2rem;
  color: white;
  font-weight: bold;
}

.score,
.high-score {
  background: rgba(255, 255, 255, 0.2);
  padding: 0.5rem 1rem;
  border-radius: 0.5rem;
  backdrop-filter: blur(10px);
}

.game-board {
  position: relative;
  border: 3px solid #ffffff;
  border-radius: 0.5rem;
  overflow: hidden;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

canvas {
  display: block;
  background: #2a2a2a;
  outline: none;
}

.game-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: white;
  backdrop-filter: blur(5px);
}

.game-overlay h2 {
  font-size: 2rem;
  margin-bottom: 1rem;
}

.game-overlay p {
  font-size: 1.2rem;
  margin: 0.5rem;
}

.controls {
  font-size: 0.9rem;
  opacity: 0.8;
  margin-top: 1rem;
}

.game-controls {
  display: flex;
  gap: 1rem;
}

.control-btn {
  padding: 0.75rem 2rem;
  font-size: 1rem;
  font-weight: bold;
  color: white;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border: none;
  border-radius: 0.5rem;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.control-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.control-btn:active {
  transform: translateY(0);
}

.mobile-controls {
  display: none;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  margin-top: 1rem;
}

.horizontal-controls {
  display: flex;
  gap: 0.5rem;
}

.arrow-btn {
  width: 60px;
  height: 60px;
  font-size: 1.5rem;
  color: white;
  background: rgba(255, 255, 255, 0.2);
  border: 2px solid white;
  border-radius: 0.5rem;
  cursor: pointer;
  transition: background 0.2s;
  backdrop-filter: blur(10px);
}

.arrow-btn:active {
  background: rgba(255, 255, 255, 0.4);
}

@media (max-width: 768px) {
  .mobile-controls {
    display: flex;
  }
  
  .game-info {
    font-size: 1rem;
  }
  
  .control-btn {
    padding: 0.5rem 1.5rem;
  }
}
</style>
