# 科学计算器 - KaTeX 公式渲染计算器

基于 HBuilderX 和 uni-app 开发的手机端科学计算器，集成 KaTeX 数学公式渲染功能。

## 功能特性

### 基础计算功能
- ✅ 加、减、乘、除四则运算
- ✅ 小数运算
- ✅ 百分比计算
- ✅ 正负号切换

### 科学计算功能
- ✅ 三角函数：sin、cos、tan
- ✅ 反三角函数：asin、acos、atan
- ✅ 平方根：√
- ✅ 对数：log（常用对数）、ln（自然对数）
- ✅ 幂运算：x²
- ✅ 数学常量：π、e
- ✅ 模式切换：标准/科学

### 特色功能
- 🎨 美观的渐变UI设计
- 📱 完全适配
- ✨ KaTeX 数学公式渲染
- 📝 历史记录显示

## 技术架构

### 项目结构
```
calculator-app/
├── pages/
│   └── index/
│       └── index.vue       # 计算器主页面
├── static/
│   └── css/
│       └── common.css   # 全局样式
├── App.vue              # 应用入口组件
├── main.js              # 应用入口文件
├── manifest.json        # 应用配置文件
├── pages.json         # 页面路由配置
├── package.json       # 依赖管理
├── vite.config.js     # Vite 配置文件
└── index.html         # HTML 入口
```

### 核心技术栈
- **框架**：uni-app (Vue 3)
- **数学渲染**：KaTeX 0.16.10
- **构建工具**：Vite 5.x
- **样式**：Sass/SCSS
- **开发工具**：HBuilderX

### 参考项目
本项目架构参考了 [KaTeX](https://github.com/KaTeX/KaTeX) 项目的设计理念和代码结构。

## 快速开始

### 使用 HBuilderX 开发
1. 下载并安装 [HBuilderX](https://www.dcloud.io/hbuilderx.html)
2. 将 `calculator-app` 文件夹导入 HBuilderX
3. 运行到浏览器或模拟器
4. 选择运行 -> 运行到手机模拟器 -> Chrome 浏览器

### 使用命令行开发
```bash
# 进入项目目录
cd calculator-app

# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建 H5 版本
npm run build:h5

# 构建 App 版本
npm run build:app-plus
```

## 使用说明

### 基本操作
1. **数字输入：点击数字按钮
2. **运算操作
3. 点击运算符按钮
4. 点击 `=` 获取结果

### 科学计算
1. 切换到 "科学" 模式
2. 使用三角函数、对数、平方根等功能

### 快捷操作
- `AC`：清除所有
- `⌫`：退格删除
- `±`：正负号切换

## 核心代码解析

### KaTeX 集成

```javascript
// 在 index.vue 中
import katex from 'katex'
import 'katex/dist/katex.min.css'

renderExpression(expr) {
    let tex = expr
        .replace(/\*/g, '\\times')
        .replace(/\//g, '\\div')
        .replace(/sqrt/g, '\\sqrt')
        .replace(/sin/g, '\\sin')
    return katex.renderToString(tex || '0', {
        throwOnError: false,
        displayMode: false
    })
}
```

### 计算引擎

```javascript
calculate() {
    let evalExpr = this.expression
        .replace(/pi/g, Math.PI.toString())
        .replace(/e(?![xp])/g, Math.E.toString())
        .replace(/sqrt\(/g, 'Math.sqrt(')
        .replace(/sin\(/g, 'Math.sin(')
        .replace(/cos\(/g, 'Math.cos(')
        .replace(/tan\(/g, 'Math.tan(')
        .replace(/log\(/g, 'Math.log10(')
        .replace(/ln\(/g, 'Math.log(')
        .replace(/\^/g, '**')
    
    const result = new Function(`return ${evalExpr}`)()
    // 更新显示
}
```

## 开发建议

### 后续优化方向
1. 更安全的计算
2. 更多科学计算功能
3. 计算历史记录
4. 主题切换
5. 国际化支持

## 许可证

MIT License
