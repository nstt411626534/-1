# KaTeX 代码 Wiki

## 目录

1. [项目概述](#项目概述)
2. [架构](#架构)
3. [核心模块](#核心模块)
4. [关键类与函数](#关键类与函数)
5. [构建系统与依赖](#构建系统与依赖)
6. [使用示例](#使用示例)
7. [贡献指南](#贡献指南)

---

## 项目概述

**KaTeX** 是一个快速、易用的 JavaScript 库，用于在网页上渲染 TeX 数学公式。它基于 Donald Knuth 的 TeX 设计，具有轻量、快速和印刷质量的特点。

### 主要特性：
- **快速**：同步渲染，无需页面重排
- **印刷质量**：基于 TeX 黄金标准的布局
- **自包含**：无依赖项
- **服务端渲染**：跨环境输出一致
- **浏览器兼容**：支持所有主流浏览器

### 版本：0.16.47
### 许可证：MIT

---

## 架构

### 高级处理流程
KaTeX 通过以下流水线处理 LaTeX 数学公式：
```
TeX 字符串 → 词法分析器 → 标记化 → 宏展开 → 语法分析器 → 解析树
→ 构建树（HTML + MathML）→ 渲染
```

### 目录结构
```
/workspace
├── src/                    # 核心源代码
│   ├── Lexer.ts           # 词法分析器
│   ├── Parser.ts          # 语法分析器
│   ├── MacroExpander.ts   # 宏展开引擎
│   ├── buildTree.ts       # 构建 DOM/MathML 树
│   ├── buildHTML.ts       # HTML 树构建器
│   ├── buildMathML.ts     # MathML 树构建器
│   ├── functions/         # 函数定义与处理程序
│   ├── environments/      # 环境定义
│   ├── styles/            # SCSS 样式表
│   └── fonts/             # 字体处理
├── contrib/               # 贡献模块
│   ├── auto-render/       # 自动渲染扩展
│   ├── copy-tex/          # 复制到剪贴板扩展
│   └── mhchem/            # 化学符号支持
├── test/                  # 测试
├── types/                 # TypeScript 类型定义
└── website/               # 网站源码
```

---

## 核心模块

### 1. 词法分析器 ([`src/Lexer.ts`](file:///workspace/src/Lexer.ts))
**用途**：将 TeX 输入标记化为标记流

**关键组件**：
- `tokenRegexString`：用于标记匹配的正则表达式模式
- 标记类型：
  - 空白字符
  - 控制词（`\macro`）
  - 控制符号
  - 带重音的单个字符
  - Unicode 代理对

**方法**：
- `lex()`：读取并返回下一个标记
- 支持类别代码（catcodes）
- 处理逐字模式和重音

### 2. 宏展开器 ([`src/MacroExpander.ts`](file:///workspace/src/MacroExpander.ts))
**用途**：在解析前展开宏

**主要特性**：
- 管理宏定义和展开
- 处理宏参数（定界和未定界）
- 支持宏组（作用域）
- 实现类 TeX 的展开规则

**关键方法**：
- `expandNextToken()`：返回下一个完全展开的标记
- `expandMacro()`：展开特定宏
- `isDefined()`：检查命令是否已定义
- `scanArgument()`：读取宏参数

### 3. 语法分析器 ([`src/Parser.ts`](file:///workspace/src/Parser.ts))
**用途**：从标记流构建解析树

**特性**：
- 将 TeX 表达式解析为解析节点
- 处理数学模式和文本模式
- 管理定界符的左右深度
- 处理中缀运算符（如 `\over`）

**关键方法**：
- `parse()`：解析整个输入
- `parseExpression()`：解析表达式
- `parseAtom()`：解析带上下标的原子
- `parseGroup()`：解析组（花括号或隐式）
- `handleInfixNodes()`：处理类似 `\over` 的中缀运算符

### 4. 解析树 ([`src/parseTree.ts`](file:///workspace/src/parseTree.ts))
**用途**：提供解析表达式的入口点

**主要函数**：
- `parseTree(toParse, settings)`：解析并返回解析树
- 处理 `\tag` 特殊情况

### 5. 构建树 ([`src/buildTree.ts`](file:///workspace/src/buildTree.ts))
**用途**：将解析树转换为最终的 DOM/MathML 输出

**关键函数**：
- `buildTree(tree, expression, settings)`：构建组合的 HTML+MathML 树
- `buildHTMLTree(tree, expression, settings)`：仅构建 HTML 树
- 选项：HTML、MathML 或两者皆可

### 6. HTML 构建器 ([`src/buildHTML.ts`](file:///workspace/src/buildHTML.ts))
**用途**：构建 HTML 表示

**关键函数**：
- `buildHTML(tree, options)`：主要 HTML 构建函数
- `buildExpression(expression, options, ...)`：构建表达式的 HTML
- `buildGroup(group, options, ...)`：构建单个解析节点
- 处理间距、二元运算符消去、换行

### 7. MathML 构建器 ([`src/buildMathML.ts`](file:///workspace/src/buildMathML.ts))
**用途**：构建 MathML 表示（用于可访问性和回退）

### 8. 设置 ([`src/Settings.ts`](file:///workspace/src/Settings.ts))
**用途**：管理用户指定的设置

**可用设置**：
- `displayMode`：在显示模式下渲染（大号）
- `output`：输出格式（html、mathml、htmlAndMathml）
- `throwOnError`：抛出错误或将无效 TeX 渲染为错误文本
- `errorColor`：错误文本的颜色
- `macros`：自定义宏定义
- `strict`：严格模式行为
- `trust`：HTML 功能的信任级别
- `maxSize`、`maxExpand`：大小和展开的限制

---

## 关键类与函数

### 入口点 ([`katex.ts`](file:///workspace/katex.ts))

#### `katex.render(expression, baseNode, options)`
将 TeX 表达式直接渲染到 DOM 节点中。

#### `katex.renderToString(expression, options)`
生成渲染数学公式的 HTML 字符串（用于服务端渲染）。

#### `katex.version`
当前 KaTeX 版本。

### 核心类

#### `Lexer`
```typescript
class Lexer {
  constructor(input: string, settings: Settings)
  lex(): Token
  setCatcode(char: string, code: number)
}
```

#### `MacroExpander`
```typescript
class MacroExpander {
  constructor(input: string, settings: Settings, mode: Mode)
  expandNextToken(): Token
  expandMacro(name: string): Token[] | undefined
  isDefined(name: string): boolean
  beginGroup(): void
  endGroup(): void
}
```

#### `Parser`
```typescript
class Parser {
  constructor(input: string, settings: Settings)
  parse(): AnyParseNode[]
  parseExpression(breakOnInfix: boolean, breakOnTokenText?: BreakToken): AnyParseNode[]
  parseAtom(breakOnTokenText?: BreakToken): AnyParseNode | null | undefined
}
```

#### `Settings`
```typescript
class Settings {
  constructor(options: SettingsOptions = {})
  reportNonstrict(errorCode: string, errorMsg: string, token?: Token | AnyParseNode): void
  isTrusted(context: AnyTrustContext): boolean
}
```

### 关键解析节点类型
（来自 [`src/parseNode.ts`](file:///workspace/src/parseNode.ts)）
- `mord`、`mop`、`mbin`、`mrel`：数学原子类型
- `ordgroup`、`supsub`：组类型
- `sqrt`、`frac`、`genfrac`：数学构造
- `text`、`verb`：文本模式
- `color`、`styling`：样式节点
- `array`、`table`：环境

---

## 构建系统与依赖

### 构建工具
- **Rollup**：模块打包器
- **Webpack**：开发与分发打包器
- **TypeScript**：类型检查
- **Sass**：CSS 预处理器
- **Jest**：测试框架

### 构建命令
```bash
yarn build          # 构建生产文件
yarn watch          # 监听模式构建
yarn test           # 运行测试
yarn test:jest      # 运行 Jest 测试
yarn start          # 启动开发服务器
yarn analyze        # 分析包大小
```

### 主要依赖
**生产依赖**：
- `commander`：命令行界面（仅用于 CLI）

**开发依赖**：
- `@babel/*`：Babel 编译器套件
- `rollup`、`@rollup/*`：模块打包
- `webpack`、`webpack-dev-server`：开发服务器
- `jest`：测试
- `typescript`：TypeScript
- `sass`：Sass 编译

### 包导出
```json
{
  "exports": {
    ".": { ... },
    "./contrib/auto-render": { ... },
    "./contrib/mhchem": { ... },
    "./contrib/copy-tex": { ... },
    "./contrib/mathtex-script-type": { ... },
    "./contrib/render-a11y-string": { ... }
  }
}
```

---

## 使用示例

### 浏览器基本使用
```html
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.47/dist/katex.min.css">
  <script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.47/dist/katex.min.js"></script>
</head>
<body>
  <div id="math"></div>
  <script>
    katex.render("c = \\pm\\sqrt{a^2 + b^2}", document.getElementById("math"), {
      throwOnError: false
    });
  </script>
</body>
</html>
```

### Node.js 使用
```javascript
const katex = require('katex');
const html = katex.renderToString("c = \\pm\\sqrt{a^2 + b^2}", {
  throwOnError: false
});
console.log(html);
```

### 自动渲染使用
```html
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.47/dist/contrib/auto-render.min.js"
  onload="renderMathInElement(document.body);"></script>
```

---

## 贡献指南

### 项目设置
```bash
yarn install
yarn start  # 启动开发服务器
```

### 添加新函数
新函数在 `src/functions/` 中定义，并在 `src/defineFunction.ts` 中注册。

### 添加新环境
新环境在 `src/environments/` 中定义，并在 `src/defineEnvironment.ts` 中注册。

### 测试
```bash
yarn test              # 运行所有测试
yarn test:jest         # 运行 Jest 测试
yarn test:jest:watch   # 监听模式
yarn test:screenshots  # 运行截图测试（需要 Docker）
```

---

## 重要提示

1. **严格模式**：KaTeX 有严格模式，强制执行 LaTeX 兼容性
2. **字体度量**：使用类 TeX 的字体度量以获得正确间距
3. **可访问性**：生成 HTML（用于视觉渲染）和 MathML（用于可访问性）
4. **输出格式**：支持纯 HTML、纯 MathML 或两者结合
5. **性能**：设计为快速同步渲染

---

## 更多文档

- 官方文档：[katex.org/docs](https://katex.org/docs)
- API 参考：[katex.org/docs/api.html](https://katex.org/docs/api.html)
- 支持的函数：[katex.org/docs/supported.html](https://katex.org/docs/supported.html)
- GitHub 仓库：[github.com/KaTeX/KaTeX](https://github.com/KaTeX/KaTeX)
