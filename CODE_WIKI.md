# KaTeX Code Wiki

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Core Modules](#core-modules)
4. [Key Classes & Functions](#key-classes--functions)
5. [Build System & Dependencies](#build-system--dependencies)
6. [Usage Examples](#usage-examples)
7. [Contribution Guidelines](#contribution-guidelines)

---

## Project Overview

**KaTeX** is a fast, easy-to-use JavaScript library for TeX math rendering on the web. It is designed to be lightweight, fast, and print-quality, based on Donald Knuth's TeX.

### Key Features:
- **Fast**: Renders synchronously without page reflows
- **Print-quality**: Layout based on TeX's gold standard
- **Self-contained**: No dependencies
- **Server-side rendering**: Same output across environments
- **Browser compatible**: Works with all major browsers

### Version: 0.16.47
### License: MIT

---

## Architecture

### High-Level Pipeline
KaTeX processes LaTeX math in a pipeline:
```
TeX String → Lexer → Tokenizer → Macro Expansion → Parser → Parse Tree
→ Build Tree (HTML + MathML) → Render
```

### Directory Structure
```
/workspace
├── src/                    # Core source code
│   ├── Lexer.ts           # Lexical analyzer
│   ├── Parser.ts          # Parser
│   ├── MacroExpander.ts   # Macro expansion engine
│   ├── buildTree.ts       # Builds DOM/MathML tree
│   ├── buildHTML.ts       # HTML tree builder
│   ├── buildMathML.ts     # MathML tree builder
│   ├── functions/         # Function definitions & handlers
│   ├── environments/      # Environment definitions
│   ├── styles/            # SCSS stylesheets
│   └── fonts/             # Font handling
├── contrib/               # Contributed modules
│   ├── auto-render/       # Auto-render extension
│   ├── copy-tex/          # Copy-to-clipboard extension
│   └── mhchem/            # Chemistry notation support
├── test/                  # Tests
├── types/                 # TypeScript definitions
└── website/               # Website source
```

---

## Core Modules

### 1. Lexer (`src/Lexer.ts`)
**Purpose**: Tokenizes TeX input into a stream of tokens

**Key Components**:
- `tokenRegexString`: Regex pattern for token matching
- Token types:
  - Whitespace
  - Control words (`\macro`)
  - Control symbols
  - Single characters with accents
  - Unicode surrogate pairs

**Methods**:
- `lex()`: Reads and returns the next token
- Supports category codes (catcodes)
- Handles verbatim mode and accents

### 2. Macro Expander (`src/MacroExpander.ts`)
**Purpose**: Expands macros before parsing

**Key Features**:
- Manages macro definitions and expansions
- Handles macro arguments (both delimited and undelimited)
- Supports macro groups (scoping)
- Implements TeX-like expansion rules

**Key Methods**:
- `expandNextToken()`: Returns next fully expanded token
- `expandMacro()`: Expands a specific macro
- `isDefined()`: Checks if a command is defined
- `scanArgument()`: Reads macro arguments

### 3. Parser (`src/Parser.ts`)
**Purpose**: Builds a parse tree from token stream

**Features**:
- Parses TeX expressions into parse nodes
- Handles math mode and text mode
- Manages left-right depth for delimiters
- Processes infix operators (e.g., `\over`)

**Key Methods**:
- `parse()`: Parses entire input
- `parseExpression()`: Parses an expression
- `parseAtom()`: Parses an atom with super/subscripts
- `parseGroup()`: Parses a group (braces or implicit)
- `handleInfixNodes()`: Handles infix operators like `\over`

### 4. Parse Tree (`src/parseTree.ts`)
**Purpose**: Provides entry point to parse expressions

**Main Function**:
- `parseTree(toParse, settings)`: Parses and returns the parse tree
- Handles `\tag` special case

### 5. Build Tree (`src/buildTree.ts`)
**Purpose**: Converts parse tree to final DOM/MathML output

**Key Functions**:
- `buildTree(tree, expression, settings)`: Builds combined HTML+MathML tree
- `buildHTMLTree(tree, expression, settings)`: Builds only HTML tree
- Options: HTML, MathML, or both

### 6. HTML Builder (`src/buildHTML.ts`)
**Purpose**: Builds HTML representation

**Key Functions**:
- `buildHTML(tree, options)`: Main HTML building function
- `buildExpression(expression, options, ...)`: Builds an expression's HTML
- `buildGroup(group, options, ...)`: Builds a single parse node
- Handles spacing, binary operator cancellation, line breaking

### 7. MathML Builder (`src/buildMathML.ts`)
**Purpose**: Builds MathML representation (for accessibility and fallback)

### 8. Settings (`src/Settings.ts`)
**Purpose**: Manages user-specified settings

**Available Settings**:
- `displayMode`: Render in display mode (large)
- `output`: Output format (html, mathml, htmlAndMathml)
- `throwOnError`: Throw errors or render invalid TeX as error text
- `errorColor`: Color for error text
- `macros`: Custom macro definitions
- `strict`: Strict mode behavior
- `trust`: Trust level for HTML features
- `maxSize`, `maxExpand`: Limits for size and expansions

---

## Key Classes & Functions

### Entry Points (`katex.ts`)

#### `katex.render(expression, baseNode, options)`
Renders a TeX expression directly into a DOM node.

#### `katex.renderToString(expression, options)`
Generates an HTML string of rendered math (for server-side rendering).

#### `katex.version`
Current KaTeX version.

### Core Classes

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

### Key Parse Node Types
(From `src/parseNode.ts`)
- `mord`, `mop`, `mbin`, `mrel`: Math atom types
- `ordgroup`, `supsub`: Group types
- `sqrt`, `frac`, `genfrac`: Math constructs
- `text`, `verb`: Text modes
- `color`, `styling`: Styling nodes
- `array`, `table`: Environments

---

## Build System & Dependencies

### Build Tools
- **Rollup**: Module bundler
- **Webpack**: Development & distribution bundler
- **TypeScript**: Type checking
- **Sass**: CSS preprocessor
- **Jest**: Testing framework

### Build Commands
```bash
yarn build          # Build production files
yarn watch          # Build with watch mode
yarn test           # Run tests
yarn test:jest      # Run Jest tests
yarn start          # Start dev server
yarn analyze        # Analyze bundle size
```

### Key Dependencies
**Production**:
- `commander`: CLI interface (only used for CLI)

**Development**:
- `@babel/*`: Babel compiler suite
- `rollup`, `@rollup/*`: Module bundling
- `webpack`, `webpack-dev-server`: Development server
- `jest`: Testing
- `typescript`: TypeScript
- `sass`: Sass compilation

### Package Exports
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

## Usage Examples

### Basic Browser Usage
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

### Node.js Usage
```javascript
const katex = require('katex');
const html = katex.renderToString("c = \\pm\\sqrt{a^2 + b^2}", {
  throwOnError: false
});
console.log(html);
```

### Auto-Render Usage
```html
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.47/dist/contrib/auto-render.min.js"
  onload="renderMathInElement(document.body);"></script>
```

---

## Contribution Guidelines

### Project Setup
```bash
yarn install
yarn start  # Start dev server
```

### Adding New Functions
New functions are defined in `src/functions/` and registered in `src/defineFunction.ts`.

### Adding New Environments
New environments are defined in `src/environments/` and registered in `src/defineEnvironment.ts`.

### Testing
```bash
yarn test              # Run all tests
yarn test:jest         # Run Jest tests
yarn test:jest:watch   # Watch mode
yarn test:screenshots  # Run screenshot tests (requires Docker)
```

---

## Important Notes

1. **Strict Mode**: KaTeX has a strict mode that enforces LaTeX compatibility
2. **Font Metrics**: Uses TeX-like font metrics for proper spacing
3. **Accessibility**: Generates both HTML (for visual rendering) and MathML (for accessibility)
4. **Output Formats**: Supports pure HTML, pure MathML, or both combined
5. **Performance**: Designed for speed with synchronous rendering

---

## Further Documentation

- Official docs: [katex.org/docs](https://katex.org/docs)
- API reference: [katex.org/docs/api.html](https://katex.org/docs/api.html)
- Supported functions: [katex.org/docs/supported.html](https://katex.org/docs/supported.html)
- GitHub repository: [github.com/KaTeX/KaTeX](https://github.com/KaTeX/KaTeX)
