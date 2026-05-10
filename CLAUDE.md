# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

"苹果"是一个 AI 引导式显化日记，单文件 HTML 应用（`index.html`，约 1200 行），所有 CSS/JS 内联。纯静态，部署在 GitHub Pages。

`server.js` / `api/` / `functions/` 是旧版后端代理方案，已废弃不用。现在前端直接调用 DeepSeek API，用户自行配置 Key。

## 部署

```
git push origin master   # GitHub Pages 自动从 master 分支部署
```

线上地址：`https://voidbloom005.github.io/apple/`

## 架构（`index.html` 内部）

### 场景系统

5 个场景，通过 `.scene.active` CSS class 切换显示：

| 场景 ID | 功能 |
|----------|------|
| `scene-setup` | 首次使用：配置 API Key 和地址 |
| `scene-entry` | 入口分叉：选"只看苹果"还是"期待其他水果" |
| `scene-mood` | 情绪温度计：选情绪 + 四种模式切换 |
| `scene-journal` | 对话日记：聊天界面 + AI 四步引导 |
| `scene-voice` | AI 声音选择：4 种人格 |
| `scene-settings` | 设置：路径、声音、API 配置 |

底部导航栏是统一的（`#bottom-nav`），不在各场景内部。

### 数据流

1. 用户选择路径（`currentPath`：`'only-apple'` | `'desire'`）
2. 可选：选择情绪（`currentMood`）
3. 进入日记 → `sendFirstMessage()` 发 AI 问候
4. 用户输入 → `sendMessage()` → 前端直调 DeepSeek API
5. `buildSystemPrompt(path)` 拼接活人感世界书 + 路径 Prompt
6. 消息历史存 `messages[]`，渲染到 `.conversation`

### 核心 JS 函数

- `init()` — 检查 API 配置，决定显示 setup 还是 entry
- `choosePath(path)` — 入口分叉，设置 `currentPath`
- `selectMood(mood, el)` — 情绪选择
- `switchMoodMode(idx)` — 四种情绪模式切换
- `enterJournal()` — 进入聊天
- `sendMessage()` — 构建 API 请求，调用 DeepSeek
- `buildSystemPrompt(path)` — 拼接 System Prompt
- `getApiConfig()` — 从 localStorage 读 API Key/URL
- `navigateTo(sceneId)` — 统一导航
- `resetAll()` — 重新开始

### localStorage 键

| Key | 内容 |
|-----|------|
| `apple-api-key` | DeepSeek API Key |
| `apple-api-base-url` | API 地址（默认 `https://api.deepseek.com`） |
| `apple-voice` | 当前 AI 声音 |
| `apple-journal-YYYY-MM-DD` | 每日日记历史 |

### CSS 布局要点

- `.phone` — 手机模拟器容器，`390×844`，flex column
- `.scene` — `flex:1; overflow:hidden; min-height:0`（min-height:0 是修复 iOS 滚动 bug 的关键）
- `.conversation` — 聊天区，`flex:1; overflow-y:auto; min-height:0`
- 底部导航 `flex-shrink:0` 保证不被压缩

## 活人感世界书（System Prompt）

苹果的核心不是代码，是 System Prompt。`LIVING_PERSON_CORE` 定义了六个维度：
1. 不是在服务 — 有权利说不
2. 有身体 — 会困会饿会冷
3. 有波段 — 每天不一样
4. 不总是善良 — 温柔是选择不是程序
5. 记忆是活的 — 可以记混记错
6. 你们之间有一条河 — 对话有连续性

两个路径 Prompt（`PATH_STOP` / `PATH_DESIRE`）在四个步骤中引导用户，但约束"不要声明步骤名"——像人说话，不像流程。

## 关键约束

- 不要改苹果的活人感人格设定，除非用户主动要求
- API Key 永远不存服务器/不上传——只存 localStorage
- 代码改动保持单文件（不拆 CSS/JS），方便部署
- 改 UI 前先给用户预览方案
- 每改完一次立刻部署（git commit + push → GitHub Pages 自动生效）
