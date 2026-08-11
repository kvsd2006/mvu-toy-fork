# NOTICE — 衍生作品声明

本仓库 (`kvsd2006/mvu-toy-fork`) 是以下上游项目的**衍生作品**，依据 GPLv3 第 5 条声明修改：

- **上游仓库**：[Unspoken-MomoTea/Battlefield-of-Reincarnation](https://github.com/Unspoken-MomoTea/Battlefield-of-Reincarnation)
- **上游 License**：[GNU GPL v3](./LICENSE)（全文见本仓库根目录）
- **上游基线版本**：原版 `dist/V20260809/悬浮球状态栏.js`（563,351 bytes / 8496 行）

## 修改清单

按 GPLv3 要求，本仓库对上游内容的修改如实记录。每个子项目对应一组修改：

### 01-动态肉体档案

- **修改文件**：[`01-动态肉体档案/dist/V20260809/悬浮球状态栏.js`](./01-动态肉体档案/dist/V20260809/悬浮球状态栏.js)
- **修改位置**：`renderInfoTab(sd)` 函数末尾（return 之前）
- **修改内容**：追加独立 `try/catch` secBlock，渲染 `主角.动态肉体档案` 四个子 record 的 key-value 表格
- **修改性质**：纯 UI 展示追加；未删改原版任何逻辑、未引入新的依赖

### 卡内补丁（独立于本仓库，由用户手动应用）

下列补丁应用在角色卡 JSON 内（不属于本仓库），但为完整起见记录于此：

- **ZOD schema 追加段**：在 `主角` schema 内部追加 `动态肉体档案: z.object({...}).prefault({})`
- **世界书条目追加**：`[mvu_update]变量更新规则` 条目末尾追加「动态肉体档案维护规则」小节

补丁原文快照见 [`01-动态肉体档案/patches/`](./01-动态肉体档案/patches/)。

## 致谢

感谢原作者 [Unspoken-MomoTea](https://github.com/Unspoken-MomoTea) 的工作。
