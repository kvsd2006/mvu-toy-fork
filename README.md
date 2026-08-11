# mvu-toy-fork

> 《轮回战场 重构版 V3.2》角色卡的**外部补丁仓库**。
>
> 目的：在不大幅修改原卡的前提下，为角色卡追加「动态肉体档案」等自定义功能，且原卡未来发布新版本时能最小成本复用。

## 仓库定位

| 路径 | 用途 |
|---|---|
| `dist/V20260809/悬浮球状态栏.js` | **fork 版**悬浮球 JS，对原作者 [Unspoken-MomoTea/Battlefield-of-Reincarnation](https://github.com/Unspoken-MomoTea/Battlefield-of-Reincarnation) 同版本 JS 做最小补丁；通过 jsdelivr CDN 部署给角色卡 import |
| `patches/` | 卡内补丁的纯文本快照（ZOD schema 追加段、MVU 规则追加段）。原卡升级时作为「补丁内容参考」粘进新版本卡 |
| `docs/` | 设计文档、新版本迁移手册 |

## 与上游的关系

```
原作者 Unspoken-MomoTea/Battlefield-of-Reincarnation
   │  原版 dist/V20260809/悬浮球状态栏.js
   │
   ▼ fork（基线快照，本仓库 dist/）
本仓库 kvsd2006/mvu-toy-fork
   │  在基线上叠加「动态肉体档案」等补丁
   │
   ▼ 通过 jsdelivr CDN
角色卡 tavern_helper:
   import 'https://cdn.jsdelivr.net/gh/kvsd2006/mvu-toy-fork@main/dist/V20260809/悬浮球状态栏.js?v=N'
```

**关键原则**：补丁**集中在悬浮球 JS**（卡外，本仓库维护）；卡内补丁最小化（仅 ZOD schema 1 行 + 世界书条目 1 段），便于跨版本复用。

## 日常维护工作流

| 场景 | 操作 |
|---|---|
| 改「动态肉体档案」UI 渲染 | 改 `dist/.../悬浮球状态栏.js` → commit → push → 角色卡内 import URL 的 `?v=N` 递增 |
| 原作者发布新版悬浮球 JS | 拉原作者新代码 → diff → 把本仓库的补丁 cherry-pick 到新代码 → commit → push → 升级 `?v=N` |
| 原作者发布新**卡**（V3.3、V4...） | 在新卡里重新粘 2 处卡内补丁（参见 `docs/复用迁移手册.md`）；CDN URL 不动，悬浮球自动用 fork 版 |

## 详见

- `docs/复用迁移手册.md` — 跨版本迁移操作步骤（暂未编写，将在首次正式迁移时补充）
- `patches/` — 卡内补丁原文

## 上游致谢

- 原作者：[Unspoken-MomoTea](https://github.com/Unspoken-MomoTea)
- 原仓库：[Battlefield-of-Reincarnation](https://github.com/Unspoken-MomoTea/Battlefield-of-Reincarnation)
- 原版 UI 版本：`V20260809`
