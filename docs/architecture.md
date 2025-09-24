# bee-taro 架构与技术说明

## 1. 项目定位与目标
- **模板定位**：bee-taro 是一个基于 Taro 4 的多端应用模板，强调与官方模板高度一致，方便在其基础上进行二次开发。【F:README.md†L1-L13】
- **兼容性要求**：项目强调不要随意升级 Vite 版本，以维持与当前 Taro 版本的兼容性，这也是后续演进需要优先遵守的约束。【F:README.md†L4-L9】

## 2. 核心技术栈
### 2.1 前端框架与运行时
- 使用 React 18 作为前端框架，结合 `@tarojs/react` 与 `@tarojs/taro` 形成跨端运行时能力。【F:package.json†L44-L62】
- 引入多端平台适配插件（微信、支付宝、字节跳动、百度、京东、QQ、H5、Harmony Hybrid 等），确保同一代码可在多平台构建。【F:package.json†L44-L56】

### 2.2 构建工具链
- 采用 Taro 官方 CLI 与 `@tarojs/vite-runner` 构建，底层 bundler 为 Vite 4，配套 Babel 与 Terser 完成语法转换和压缩。【F:package.json†L65-L74】
- 通过 `@vitejs/plugin-react`、`react-refresh` 和 `sass` 支持 React 快速刷新与 Sass 预处理。【F:package.json†L75-L83】

### 2.3 语言与类型支持
- 模板默认启用 TypeScript，并通过 `@types/react` 提供类型补全。【F:package.json†L6-L10】【F:package.json†L75-L83】
- `types/global.d.ts` 声明了常用静态资源模块类型，并扩展了构建时环境变量类型，降低在 TS 项目中引用非代码资源的门槛。【F:types/global.d.ts†L1-L27】

## 3. 目录与代码结构
- `src/app.ts` 是应用入口，使用 `useLaunch` 生命周期钩子执行初始化逻辑，并直接渲染子页面组件。【F:src/app.ts†L1-L13】
- `src/app.config.ts` 描述小程序/多端应用的路由栈和窗口样式，目前仅注册首页 `pages/index/index`，默认导航栏主题为浅色。【F:src/app.config.ts†L1-L11】
- `src/pages/index/index.tsx` 是默认页面，展示 "Hello world!"，并在 `useLoad` 钩子中输出页面加载日志，可作为新页面的创建参考。【F:src/pages/index/index.tsx†L1-L15】
- 额外的样式与全局资源（如 `src/app.scss`、`src/pages/index/index.scss`）预留为空，方便团队根据设计体系快速填充样式。

## 4. 配置体系
### 4.1 通用配置（`config/index.ts`）
- 通过 `defineConfig` 动态合并基础配置与环境配置，统一设置项目名称、创建日期以及设计稿宽度等全局参数。【F:config/index.ts†L1-L92】
- 预设 `deviceRatio`、`sourceRoot`、`outputRoot` 等关键参数，并内嵌 H5 端 CSS 抽离、Autoprefixer 等常见选项，保证多端样式一致性。【F:config/index.ts†L10-L73】
- 在运行期同步 `BROWSERSLIST_ENV` 与当前 `NODE_ENV`，确保前端构建与浏览器兼容策略一致。【F:config/index.ts†L84-L91】

### 4.2 环境差异化配置
- 开发环境 (`config/dev.ts`) 保持最简配置，方便团队按需引入代理、Mock 或调试插件。【F:config/dev.ts†L1-L7】
- 生产环境 (`config/prod.ts`) 默认启用 H5 Legacy 模式，保障旧设备兼容性；同时预留 WebpackChain 钩子以便集成包体分析、预渲染等高级优化能力。【F:config/prod.ts†L1-L35】

## 5. 构建与运行流程
- `package.json` 的 `build:*` 与 `dev:*` 脚本覆盖微信、支付宝、百度、字节、QQ、京东、H5、React Native、Harmony Hybrid 等平台，可选择一次性构建或启用监听式开发模式。【F:package.json†L12-L31】
- Browserslist 分别为开发与生产指定兼容目标，保障开发调试体验与生产环境的终端覆盖范围。【F:package.json†L32-L41】
- `pnpm-workspace.yaml` 的 `onlyBuiltDependencies` 确认在安装时需要预编译的原生依赖，避免首次构建缓慢或失败。【F:pnpm-workspace.yaml†L1-L8】

## 6. 工具链与工程能力
- Babel 通过 `babel-preset-taro` 统一处理 React + TypeScript 语法，并在 H5 环境按需注入 Polyfill，确保不同端的兼容性。【F:babel.config.js†L1-L12】
- TypeScript 配置开启严格模式、路径别名 `@/*`、以及 JSON/JSX 支持，为多端项目提供一致的类型检查体验。【F:tsconfig.json†L1-L32】
- `types/global.d.ts` 的环境变量声明让团队在读取 `process.env` 时具备类型提示，有助于管理跨端差异化配置。【F:types/global.d.ts†L14-L26】

## 7. 多端与平台适配策略
- 项目依赖 `@tarojs/plugin-platform-*` 系列插件实现多平台输出，每个平台的特定能力由 Taro 官方维护，减少自定义适配成本。【F:package.json†L44-L56】
- 对于 React Native 与 Harmony Hybrid 等端，虽然默认配置较轻量，但 CLI 脚本与依赖已经准备就绪，可在需求出现时快速启用。【F:package.json†L18-L31】【F:package.json†L44-L56】

## 8. 小程序工具链与发布
- `project.config.json` 提供微信小程序开发者工具所需的项目配置，指定产物目录、appid 以及构建选项，便于构建后直接导入预览或提交代码审核。【F:project.config.json†L1-L14】
- 生产构建输出位于 `dist/`，与 `config/index.ts` 中的 `outputRoot` 保持一致，确保小程序工具配置与 Taro 构建产物协同。【F:config/index.ts†L20-L91】【F:project.config.json†L1-L14】

## 9. 二次开发与演进建议
- 二次开发时需优先遵守 Vite 版本约束，避免与当前 Taro 4 生态产生不兼容问题。【F:README.md†L4-L9】
- 建议按照 `config/` 的分层方式新增环境文件（如 `staging.ts`）、或通过 `types/global.d.ts` 扩展自定义环境变量，保持配置一致性。【F:config/index.ts†L1-L92】【F:types/global.d.ts†L14-L26】
- 结合 `config/prod.ts` 中预留的 WebpackChain 插槽，可在增长阶段加入包体分析、预渲染等能力，提升性能与可观测性。【F:config/prod.ts†L5-L35】
