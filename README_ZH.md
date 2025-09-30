# 项目中文说明

本项目是一个基于 Vite + Vue 3 + Manifest V3 的浏览器扩展（支持 Chrome/Chromium、Firefox）模板，内置多页 UI（Action 弹窗、Options、Devtools Panel、Side Panel、内容脚本 IFrame、安装/更新页面），提供目录式路由、Pinia、i18n、Tailwind、DaisyUI、通知、错误边界、Loading 等常用基础设施。

为了帮助你快速理解与二次开发，本文件将详细介绍项目目录结构，以及关键目录与文件的职责说明。

---

## 目录结构总览

```bash
.
├── CHANGELOG.md                # 变更日志（供更新页面展示）
├── define.config.mjs           # 统一注入到 Vite define 的常量/配置
├── eslint.config.mjs           # ESLint 配置
├── manifest.chrome.config.ts   # Chrome/Chromium 专用 manifest 覆盖
├── manifest.config.ts          # 通用 Manifest V3 配置（核心）
├── manifest.firefox.config.ts  # Firefox 专用 manifest 覆盖
├── package.json                # 依赖与脚本命令
├── pnpm-lock.yaml              # pnpm 锁定文件
├── postcss.config.cjs          # PostCSS 配置
├── public/                     # 静态资源（打包时原样拷贝）
├── README.md                   # 英文说明
├── screenshots/                # 截图（用于 README 展示）
├── scripts/                    # 开发/调试脚本
├── src/                        # 源码根目录
│   ├── assets/                 # 全局样式与图片
│   ├── background/             # 背景 Service Worker 入口
│   ├── components/             # 共享组件（自动按需注册）
│   ├── composables/            # 可复用的组合式函数（自动导入）
│   ├── content-script/         # 内容脚本（注入网页）
│   ├── devtools/               # Devtools 页面（注册入口页）
│   ├── locales/                # i18n 语言包
│   ├── offscreen/              # Offscreen 页面（录音/录屏等）
│   ├── stores/                 # Pinia 状态管理（自动导入）
│   ├── types/                  # TS 类型定义与自动生成的 d.ts
│   ├── ui/                     # 各独立 UI 子应用（多页应用）
│   │   ├── action-popup/       # 浏览器工具栏弹窗
│   │   ├── common/             # 公共页面（关于/功能/隐私/条款/404/日志）
│   │   ├── content-script-iframe/ # 内容脚本挂载的 IFrame 应用
│   │   ├── devtools-panel/     # Devtools 面板 UI
│   │   ├── options-page/       # 扩展设置页（Options）
│   │   ├── setup/              # 安装/更新 向导页面
│   │   └── side-panel/         # 侧边面板（Chrome Side Panel）
│   └── utils/                  # 工具与框架初始化
├── tsconfig.json               # TS 配置（应用）
├── tsconfig.node.json          # TS 配置（Node/Vite 构建环境）
├── vite.chrome.config.ts       # Vite 构建（Chrome 变体）
├── vite.config.ts              # Vite 基础配置（核心）
├── vite.firefox.config.ts      # Vite 构建（Firefox 变体）
└── dist/                       # 构建产物（按浏览器区分）
```

---

## 顶层配置与脚本

- package.json
  - dependencies/devDependencies：项目依赖
  - scripts：
    - `npm run dev`：并行启动 Chrome 和 Firefox 开发模式
    - `npm run dev:chrome`：Chrome 开发模式（HMR）
    - `npm run dev:firefox`：Firefox 开发模式（watch 构建）
    - `npm run build`：打包 Chrome 与 Firefox 两个产物
    - `npm run build:chrome`：仅打包 Chrome
    - `npm run build:firefox`：仅打包 Firefox
    - `npm run lint`：ESLint 自修复并缓存
    - `npm run typecheck`：类型检查
    - `npm run launch` / `npm run launch:all`：尝试自动在已安装浏览器中加载扩展（见 `scripts/`）

- vite.config.ts（核心）
  - 多入口构建：注册 `src/ui/setup`、`src/ui/content-script-iframe`、`src/ui/devtools-panel` 的 HTML
  - 自动导入：Vue、Vue Router、Pinia、VueUse、vue-i18n、`browser`（webextension-polyfill）等
  - 组件自动注册：`src/components` 目录组件按需全局注册
  - 路由自动生成：扫描 `src/ui/*/pages` 生成各子应用的基于文件的路由
  - Tailwind、Icons、Devtools、TurboConsole、i18n 插件集成
  - `assets-rewrite`：构建后将根路径资源改写为相对路径

- vite.chrome.config.ts / vite.firefox.config.ts
  - 在基础配置上叠加浏览器差异化配置与 @crxjs/vite-plugin 设置

- manifest.config.ts（核心）
  - 通用 Manifest V3 定义：
    - `background.service_worker`: `src/background/index.ts`
    - `content_scripts`: 注入 `src/content-script/index.ts` 到所有页面
    - `side_panel.default_path`: `src/ui/side-panel/index.html`
    - `devtools_page`: `src/devtools/index.html`
    - `options_page`: `src/ui/options-page/index.html`
    - `web_accessible_resources`: 允许 IFrame/Setup/Devtools Panel HTML 被页面访问
    - `permissions`: `storage`, `tabs`, `background`, `sidePanel`
    - `icons`: 使用 `src/assets/logo_cat_mini.png`
  - 从 `package.json` 同步 `name/description/version`，并处理 `version_name`

- manifest.chrome.config.ts / manifest.firefox.config.ts
  - 在通用 manifest 基础上进行浏览器特定的权限或字段覆盖

- define.config.mjs
  - 向 Vite 注入编译期常量（供业务按需使用）

- postcss.config.cjs
  - Tailwind 4 与 PostCSS 处理链

- eslint.config.mjs
  - ESLint 规则（集成 TypeScript、Vue、Auto-Import 生成的规则）

---

## 源码（src/）详解

### assets/
- base.scss：全局样式入口，会被除 `content-script/index.scss` 外的 SFC 自动注入
- logo.png / logo_cat.png / logo_cat_mini.png：图标资源
- 其他图片资源

### background/
- index.ts：后台 Service Worker 入口，适合放置：
  - 生命周期事件监听（安装/更新）
  - 与各 UI/内容脚本的消息通信
  - 长期运行的任务与定时逻辑

### components/
- AppHeader.vue / AppFooter.vue：通用页眉/页脚
- LocaleSwitch.vue：语言切换器（结合 i18n）
- ThemeSwitch.vue：主题切换器（明暗模式）
- RouterLinkUp.vue：简化返回上一级路由的链接组件
- state/LoadingSpinner.vue：加载状态指示
- state/DisplayError.vue：错误边界显示
- state/tailwind-empty-state.vue：空状态展示
- TestComponent.vue：示例组件

这些组件通过 `unplugin-vue-components` 自动按需全局注册，可直接在各页面使用。

### composables/
- useBrowserStorage.ts：统一封装 `sync/local` 存储（读写响应式数据）
- useLocale.ts：处理语言切换与持久化
- useTheme.ts：处理主题切换、持久化与 DOM 应用

通过 `unplugin-auto-import` 自动导入，在 SFC/脚本中可直接使用。

### content-script/
- index.ts：注入到匹配网页的内容脚本，负责：
  - 与背景页通信（webext-bridge 或 `browser.runtime`）
  - 业务 DOM 注入、事件监听
- index.scss：内容脚本专用样式（不会注入 base.scss）

### devtools/
- index.html：注册 Devtools 页（由 manifest `devtools_page` 引用）
- index.ts：Devtools 页入口脚本，可挂载面板或通信逻辑

### locales/
- en.json / zh.json：多语言资源文件

### offscreen/
- index.html / index.ts：Offscreen 文档，用于录音/录屏等无需可见 UI 的场景

### stores/
- options.store.ts：Options 设置相关的 Pinia Store（支持持久化/响应式）
- test.store.ts：示例 Store

### types/
- auto-imports.d.ts：由 Auto Import 生成的类型声明
- components.d.ts：由组件自动注册插件生成的类型声明
- router-meta.d.ts / typed-router.d.ts：由 `unplugin-vue-router` 生成的类型与元信息
- vite-env.d.ts：Vite 环境类型增强
- .eslintrc-auto-import.json：Auto Import 的 ESLint 辅助配置

### ui/（多页子应用）
路由按「目录式」生成：扫描每个子目录下的 `pages/` 作为路由页面，入口为同级的 `index.html`（见 Vite 配置）。

- action-popup/
  - app.vue / index.ts / pages/index.vue / pages/playground.vue：工具栏弹窗 UI 与页面

- common/
  - pages/about.vue / features.vue / change-log.vue / help.vue / privacy-policy.vue / terms-of-service.vue / 404.vue：通用信息页面

- content-script-iframe/
  - app.vue / index.ts / pages/index.vue：注入到页面中的 IFrame 应用

- devtools-panel/
  - app.vue / index.ts / pages/index.vue：Devtools 自定义面板

- options-page/
  - app.vue / index.ts / pages/index.vue：扩展选项页（Options）

- setup/
  - app.vue / index.ts / pages/install.vue / pages/update.vue：安装/更新向导页面（通常在扩展生命周期事件时展示）

- side-panel/
  - app.vue / index.ts / pages/index.vue：侧边面板（Chrome Side Panel）

### utils/
- i18n.ts：初始化 vue-i18n，提供 `useI18n`/`t` 等能力
- notifications.ts：封装 Notivue 通知的便捷调用
- pinia.ts：创建 Pinia 实例与插件接入
- router/index.ts：创建 Vue Router 实例（与 `unplugin-vue-router` 生成的路由结合）

---

## scripts/
- getInstalledBrowsers.ts：检测本机已安装的浏览器（用于自动启动调试）
- launch.ts：基于 `web-ext`/系统安装的浏览器，尝试启动并加载 `dist` 目录中的扩展

---

## public/
用于放置无需打包处理的静态文件（如 `icons/`）。构建时会原样复制到产物目录。

---

## 构建与产物

- 开发：
  - `npm run dev` 同时启动 Chrome/Firefox 变体
  - Chrome 使用 Vite HMR；Firefox 使用 watch 构建

- 打包：
  - `npm run build` 生成 `dist/chrome` 与 `dist/firefox`
  - 浏览器中以「加载已解压扩展」方式选择相应目录进行安装调试

---

## 关键数据流与通信

- 背景页（`src/background`）与：
  - 内容脚本（`src/content-script`）
  - 各 UI 子应用（`src/ui/*`）
  之间可通过 `webextension-polyfill` 或 `webext-bridge` 进行消息通信。

- 状态管理通过 Pinia（`src/stores`），跨页面共享配置建议持久化到 `browser.storage`（可通过 `useBrowserStorage` 统一读写）。

---

## 贡献建议

- 新增页面：在对应 `src/ui/<app>/pages/` 下新增 `.vue` 文件即可自动注册路由
- 新增组件：放入 `src/components/` 即可自动按需引入
- 新增组合函数/工具：放入 `src/composables/` 或 `src/utils/`，会被自动导入/可全局使用

---

如果你需要更细颗粒的文件级说明或希望我补充代码注释，请告诉我你关注的模块或业务场景，我会继续完善。


