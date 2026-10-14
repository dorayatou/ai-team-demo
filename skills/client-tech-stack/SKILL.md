---
name: client-tech-stack
description: 提供一套可复刻的前端技术栈与工程脚手架（React 19 + TypeScript 5.8 strict + Vite 8 + ESLint 9 flat config + pnpm + 原生 CSS），含依赖版本、配置文件与目录约定。新建前端项目或初始化项目骨架时使用；不含业务逻辑。
---

# 前端技术栈复用（React 19 + TypeScript + Vite）

本 skill 只包含技术栈与工程配置，用于在新项目中复刻同一套栈；业务逻辑、领域规则由新项目自行设计。

## 依赖清单（package.json）

```json
{
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "preview": "vite preview",
    "typecheck": "tsc -b",
    "lint": "eslint ."
  },
  "dependencies": {
    "react": "^19.1.0",
    "react-dom": "^19.1.0"
  },
  "devDependencies": {
    "@eslint/js": "^9.30.0",
    "@types/react": "^19.1.8",
    "@types/react-dom": "^19.1.6",
    "@vitejs/plugin-react": "^6.1.2",
    "eslint": "^9.30.0",
    "eslint-plugin-react-hooks": "^5.2.0",
    "eslint-plugin-react-refresh": "^0.4.20",
    "globals": "^16.2.0",
    "typescript": "~5.8.3",
    "typescript-eslint": "^8.35.0",
    "vite": "^8.3.4"
  }
}
```

原则：运行时依赖只有 react 和 react-dom；不预装路由、状态管理库、UI 组件库、axios，确有需要时再单独评估引入。

## 配置文件

### vite.config.ts

```ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    proxy: {
      // 需要后端联调时，把 /api 代理到本地后端，避免跨域
      "/api": {
        target: "http://localhost:8787", // 按新项目后端端口修改
        changeOrigin: true,
      },
    },
  },
});
```

### tsconfig.json（项目引用入口）

```json
{
  "files": [],
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" }
  ]
}
```

### tsconfig.app.json（src 代码）

```json
{
  "compilerOptions": {
    "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.app.tsbuildinfo",
    "target": "ES2022",
    "useDefineForClassFields": true,
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "moduleDetection": "force",
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedSideEffectImports": true
  },
  "include": ["src"]
}
```

### tsconfig.node.json（vite.config.ts）

```json
{
  "compilerOptions": {
    "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.node.tsbuildinfo",
    "target": "ES2023",
    "lib": ["ES2023"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "moduleDetection": "force",
    "noEmit": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["vite.config.ts"]
}
```

### eslint.config.js（ESLint 9 flat config）

```js
import js from "@eslint/js";
import globals from "globals";
import reactHooks from "eslint-plugin-react-hooks";
import reactRefresh from "eslint-plugin-react-refresh";
import tseslint from "typescript-eslint";
import { globalIgnores } from "eslint/config";

export default tseslint.config([
  globalIgnores(["dist"]),
  {
    files: ["**/*.{ts,tsx}"],
    extends: [
      js.configs.recommended,
      tseslint.configs.recommended,
      reactHooks.configs["recommended-latest"],
      reactRefresh.configs.vite,
    ],
    languageOptions: {
      ecmaVersion: 2022,
      globals: globals.browser,
    },
  },
]);
```

## 目录结构（src/）

- components/ — UI 组件，每个组件配套同名 .css（Foo.tsx + Foo.css）
- hooks/ — 自定义 Hook，承接状态与副作用
- services/ — fetch 封装与纯函数工具（校验等）
- types/ — index.ts 集中定义共享类型
- styles/ — global.css 全局样式与设计变量
- constants.ts — 项目常量
- App.tsx 组装页面；index.html 放项目根目录，lang 与 title 按新项目修改

入口骨架 main.tsx：

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import "./styles/global.css";
import App from "./App.tsx";

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

## 编码约定（随技术栈，不含业务）

- 只写函数组件与 Hooks。
- 类型引入必须用 import type（verbatimModuleSyntax 已开启）。
- 共享类型集中在 types/index.ts；公共 Props 与数据结构显式定义类型，禁止 any 与掩盖错误的断言。
- 样式用原生 CSS 文件，不用 Tailwind、CSS Modules、内联样式；全局变量进 global.css。
- API 请求走 services/ 的 fetch 封装，不引入 axios，组件不直接 fetch。
- 常量收进 constants.ts，不在组件里写魔法数字。
- 包管理器统一用 pnpm。

## 新项目初始化步骤

1. 新建项目目录，写入上述 package.json、四个配置文件与 index.html。
2. 创建 src 骨架：main.tsx、App.tsx、styles/global.css、types/index.ts、constants.ts 及空的 components/hooks/services 目录。
3. 执行 pnpm install。
4. pnpm dev 启动（5173 端口），确认页面正常渲染。
5. 提交前执行 pnpm typecheck 与 pnpm lint，全部通过才算完成。
