# 🐍 贪吃蛇游戏 - Snake Game

一个使用 Nuxt 3 和 Vue 3 开发的经典贪吃蛇游戏。

## 🎮 游戏特性

- 经典贪吃蛇游戏玩法
- 响应式设计，支持桌面和移动设备
- 键盘和触屏控制
- 分数记录和最高分保存
- 游戏速度随分数增长而加快
- 流畅的动画效果
- 暗色主题界面

## 🎯 游戏控制

### 键盘控制
- **方向键** 或 **WASD** - 控制蛇的移动方向
- **空格键** - 开始/暂停游戏
- **ESC** - 暂停游戏

### 移动端控制
- 使用屏幕下方的方向按钮控制
- 点击"开始"按钮开始游戏

## 🚀 快速开始

### 安装依赖

```bash
# 使用 pnpm（推荐）
pnpm install

# 或使用 npm
npm install

# 或使用 yarn
yarn install
```

### 启动开发服务器

```bash
# 使用 pnpm
pnpm dev

# 或使用 npm
npm run dev

# 或使用 yarn
yarn dev
```

游戏将在 `http://localhost:3000` 启动

## 🏗️ 构建生产版本

```bash
# 使用 pnpm
pnpm build

# 预览生产版本
pnpm preview
```

## 📁 项目结构

```
Nuxt Snake/
├── app/
│   ├── assets/
│   │   └── css/
│   │       └── main.css       # 全局样式
│   ├── components/
│   │   └── SnakeGame.vue      # 贪吃蛇游戏核心组件
│   ├── pages/
│   │   └── index.vue          # 游戏主页面
│   └── app.vue                # 应用根组件
├── nuxt.config.ts             # Nuxt 配置文件
├── package.json               # 项目依赖
└── README.md                  # 项目说明
```

## 🎨 游戏规则

1. 使用方向键控制蛇的移动
2. 吃到红色的食物得 10 分
3. 蛇会随着吃食物而变长
4. 撞到墙壁或自己的身体游戏结束
5. 游戏速度会随着分数增加而加快
6. 最高分会自动保存在浏览器中

## 🛠️ 技术栈

- **Nuxt 3** - Vue.js 框架
- **Vue 3** - 前端框架
- **TypeScript** - 类型安全
- **Canvas API** - 游戏渲染
- **pnpm** - 包管理器

## 📝 开发说明

游戏使用 HTML5 Canvas 进行渲染，主要逻辑在 `SnakeGame.vue` 组件中：

- 游戏网格大小：20x20
- 单元格大小：20px
- 初始速度：150ms
- 速度增量：每 5 分加快一次

## 🔧 自定义配置

你可以在 `SnakeGame.vue` 中修改以下常量来自定义游戏：

```javascript
const GRID_SIZE = 20        // 网格大小
const CELL_SIZE = 20        // 单元格大小
const INITIAL_SPEED = 150   // 初始速度（毫秒）
const SPEED_INCREMENT = 5   // 速度增量
```

## 📄 许可证

MIT

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

---

祝你游戏愉快！ 🎮🐍
