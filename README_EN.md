# 🐍 Snake Game - Nuxt 3

English | [中文](./README.md)

A classic snake game built with Nuxt 3 and Vue 3.

## 🎮 Game Features

- Classic snake game mechanics
- Responsive design supporting desktop and mobile devices
- Keyboard and touch controls
- Score tracking and high score saving
- Game speed increases with score growth
- Smooth animations
- Dark theme interface

## 🎯 Game Controls

### Keyboard Controls
- **Arrow Keys** or **WASD** - Control snake movement direction
- **Space Bar** - Start/pause game
- **ESC** - Pause game

### Mobile Controls
- Use direction buttons at the bottom of the screen
- Tap "Start" button to begin game

## 🚀 Quick Start

### Install Dependencies

```bash
# Using pnpm (recommended)
pnpm install

# Or using npm
npm install

# Or using yarn
yarn install
```

### Start Development Server

```bash
# Using pnpm
pnpm dev

# Or using npm
npm run dev

# Or using yarn
yarn dev
```

The game will start at `http://localhost:3000`

## 🏗️ Build for Production

```bash
# Using pnpm
pnpm build

# Preview production build
pnpm preview
```

## 📁 Project Structure

```
Nuxt Snake/
├── app/
│   ├── assets/
│   │   └── css/
│   │       └── main.css       # Global styles
│   ├── components/
│   │   └── SnakeGame.vue      # Snake game core component
│   ├── pages/
│   │   └── index.vue          # Main game page
│   └── app.vue                # App root component
├── nuxt.config.ts             # Nuxt configuration
├── package.json               # Project dependencies
└── README.md                  # Project documentation
```

## 🎨 Game Rules

1. Use arrow keys to control snake movement
2. Eat red food to get 10 points
3. Snake grows longer when eating food
4. Game ends when hitting walls or snake's own body
5. Game speed increases with score
6. High score is automatically saved in browser

## 🛠️ Tech Stack

- **Nuxt 3** - Vue.js framework
- **Vue 3** - Frontend framework
- **TypeScript** - Type safety
- **Canvas API** - Game rendering
- **pnpm** - Package manager

## 📝 Development Notes

The game uses HTML5 Canvas for rendering, with main logic in the `SnakeGame.vue` component:

- Game grid size: 20x20
- Cell size: 20px
- Initial speed: 150ms
- Speed increment: Increases every 5 points

## 🔧 Custom Configuration

You can modify the following constants in `SnakeGame.vue` to customize the game:

```javascript
const GRID_SIZE = 20        // Grid size
const CELL_SIZE = 20        // Cell size
const INITIAL_SPEED = 150   // Initial speed (milliseconds)
const SPEED_INCREMENT = 5   // Speed increment
```

## 📄 License

MIT

## 🤝 Contributing

Issues and Pull Requests are welcome!

---

Enjoy the game! 🎮🐍