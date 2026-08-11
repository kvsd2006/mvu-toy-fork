# 01 - 动态肉体档案

> 在悬浮球「信息」页面追加「动态肉体档案」，集中追踪主角的**肉体外观 / 衣物服饰 / 情趣配饰 / 性器状态**。
>
> 纯外观展示，**不参与战斗/属性/EP/HP 计算**；同时作为 NSFW 场景的快速语义参考。

## 设计要点

| 维度 | 选择 |
|---|---|
| 数据流方向 | AI 双向（既写也读） |
| 字段结构 | `主角.动态肉体档案.{肉体外观,衣物服饰,情趣配饰,性器状态}` 各 `record<string,string>` |
| 填表风格 | 短语 / 关键词，不写整段描述 |
| 维护时机 | 剧情变化 / OOC 元指令 / NSFW 场景进入 |
| 改动原则 | 最小补丁 + 外部 fork |

## 改动清单

### 卡内补丁（手动应用到角色卡 JSON）

| # | 位置 | 内容 |
|---|---|---|
| 1 | `extensions.tavern_helper.scripts[]` 内 ZOD schema 脚本中 `主角` schema 内部 | 追加 `动态肉体档案: z.object({...}).prefault({})` |
| 2 | 世界书条目 `[mvu_update]变量更新规则` 末尾 | 追加「动态肉体档案维护规则」小节 |
| 3 | 世界书条目 `[variables]当前变量` EJS 中 `current.主角` 块末尾 | 追加 `动态肉体档案: data.主角.动态肉体档案`（**不加此条 AI 看不到档案**） |

补丁原文见 [`patches/`](./patches/)。

### 卡外补丁（fork 部署）

| 文件 | 用途 |
|---|---|
| [`dist/V20260809/悬浮球状态栏.js`](./dist/V20260809/悬浮球状态栏.js) | 完整 fork JS，通过 jsdelivr CDN 给角色卡 import |

补丁具体位置：`renderInfoTab(sd)` 函数末尾（return html 之前）追加独立 `try/catch` secBlock，渲染 4 个 record 的 key-value 表格，支持编辑模式增删槽位。

## 部署步骤

1. **应用卡内补丁**：按 `patches/01-zod-schema.md`、`patches/02-mvu-update-rules.md`、`patches/03-current-variables-ejs.md` 的说明，把 3 处补丁粘到角色卡对应位置
2. **改 import URL**：找到角色卡 `extensions.tavern_helper.scripts[]` 里 `name: "悬浮球状态栏"` 的脚本，把 `content` 改为：
   ```
   import 'https://cdn.jsdelivr.net/gh/kvsd2006/mvu-toy-fork@main/01-动态肉体档案/dist/V20260809/悬浮球状态栏.js?v=1'
   ```
3. **验证**：在 SillyTavern 加载角色卡 → 悬浮球 → 信息 Tab → 看到「动态肉体档案」区块（首次为空，编辑模式下手动加槽位，或剧情推进后 AI 自动填入）

## 升级工作流

| 场景 | 操作 |
|---|---|
| 改 UI 渲染 | 改 `dist/.../悬浮球状态栏.js` → commit → push → 卡内 `?v=N` 递增 |
| 原作者发新悬浮球 JS | 拉上游新代码 → 把补丁 cherry-pick 到新版本 → push → `?v=N` 递增 |
| 原作者发新卡（V3.3 等） | 重新粘 3 处卡内补丁（位置锚点见 `patches/`）；CDN URL 不动，悬浮球自动用 fork 版 |

详细见 [`docs/复用迁移手册.md`](./docs/复用迁移手册.md)（首次正式迁移时编写）。

## 状态

- **当前版本**：v0.1.0（补丁已应用到 dist/JS；3 patches 文件已落盘；待部署到角色卡实测）
- **设计文档**：[`docs/设计文档.md`](./docs/设计文档.md)（完整 spec 见外部 `docs/superpowers/specs/2026-08-11-动态肉体档案-design.md`）
- **首次实现 commit**：待定

## 上游关系

本子项目修改自上游 GPLv3 代码。衍生声明见 [根 NOTICE.md](../NOTICE.md)。
