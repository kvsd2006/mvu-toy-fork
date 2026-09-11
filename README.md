# mvu-toy-fork

> 《轮回战场 重构版》角色卡的**多项目外部补丁仓库**。
>
> 每个子目录是一个独立的补丁项目，可单独部署、单独迁移、单独删除。

## 上游关系

- **上游**：[Unspoken-MomoTea/Battlefield-of-Reincarnation](https://github.com/Unspoken-MomoTea/Battlefield-of-Reincarnation)
- **License**：[GPLv3](./LICENSE)（继承自上游）
- **衍生声明**：[NOTICE.md](./NOTICE.md)

## 子项目索引

| 序号 | 项目 | 部署目标 | 状态 |
|---|---|---|---|
| 01 | [动态肉体档案](./01-body-archive/) | 悬浮球 JS + ZOD + MVU 规则 | 可用（v0.4.0，基线 V3.5.6） |
| -- | [portraits](./portraits/) | 角色头像素材（CDN URL 引用） | 素材库，随加随用 |

> 新项目按 `NN-中文名/` 规则新建目录，并在本表追加索引行。

## 通用部署工作流

每个子项目独立部署，通用流程：

1. 按子项目 `README.md` 应用 2-3 处**卡内补丁**（ZOD schema / 世界书条目）
2. 修改角色卡的 `extensions.tavern_helper.scripts[]` 里对应的 `import 'CDN'` URL，指向本仓库对应子项目的 `dist/`：
   ```
   https://cdn.jsdelivr.net/gh/kvsd2006/mvu-toy-fork@main/<子项目>/dist/<文件路径>?v=N
   ```
3. 每次子项目 `dist/` 改动后，把 `?v=N` 递增刷新 jsdelivr 缓存

## 仓库约定

| 路径模式 | 用途 |
|---|---|
| `<NN-项目名>/README.md` | 子项目说明（设计意图、改动清单、部署步骤） |
| `<NN-项目名>/dist/` | CDN 部署目标（完整 fork 文件） |
| `<NN-项目名>/patches/` | 卡内补丁原文（按编号顺序应用） |
| `<NN-项目名>/docs/` | 子项目设计文档、迁移手册 |

## 修改记录

| 日期 | 内容 |
|---|---|
| 2026-08-11 | 基线（init fork from upstream V20260809） |
| 2026-08-11 | 目录重组为多项目结构 + 加 LICENSE/NOTICE |
| 2026-08-31 | 01 子项目基线升级至轮回战场 V3.5.1 内联脚本（上游 V20260829），产出 dist/V20260831 |
| 2026-09-11 | 01 子项目基线升级至轮回战场 V3.5.6 内联脚本（上游 V20260901），产出 dist/V20260901 + 迁移手册 |
