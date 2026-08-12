# 卡内补丁 #1：ZOD schema 追加段

## 位置

角色卡 JSON `extensions.tavern_helper.scripts[]` 内 ZOD schema 脚本中，**主 Schema 的 `主角: z.object({ ... })` 块内**。

> **⚠️ 防错（2026-08-12 实测踩坑）**：ZOD 脚本里有两个结构相似的块——
> - `const npc_schema = strictItem(z.object({ ... 形态库 ... 当前形态 ... 性格 ... }))`（**NPC 面板**，定义在前）
> - `export const Schema = z.object({ ... 主角: z.object({ ... 形态库 ... 当前形态 ... }) ... })`（**主角**，定义在后）
>
> 两者都含 `形态库` 和 `当前形态`，**只搜这两个词会先命中 npc_schema**，导致补丁打进 NPC schema、主角 schema 缺失该键。后果：MVU 把 AI 写入 `/主角/动态肉体档案` 判为 SCHEMA 违规直接跳过，悬浮球 4 个档案块永远为空。

## 安全锚点（用于定位主角 schema）

用 `主角: z.object({` 定位，并确认该块内**含 `空间币`、不含 `性格/喜爱/外貌/着装`**：

```javascript
主角: z.object({
    种族: safeStr('人类'),
    ...
    空间币: safeNum(0).transform(v => Math.max(0, v)),   // ← 主角专属，npc_schema 没有
    形态库: z.record(z.string(), form_item).prefault({}),
    【插入点在这里，形态库 之后、当前形态 之前】
    当前形态: current_form
}).prefault({}).transform(char => {
```

## 补丁原文

在 `形态库: z.record(z.string(), form_item).prefault({}),` 之后、`当前形态: current_form` 之前插入（缩进与 `主角` 块内其他字段一致，8 空格）：

```javascript
动态肉体档案: z.object({
    肉体外观: z.record(z.string(), z.string()).prefault({}),
    衣物服饰: z.record(z.string(), z.string()).prefault({}),
    情趣配饰: z.record(z.string(), z.string()).prefault({}),
    性器状态: z.record(z.string(), z.string()).prefault({}),
}).prefault({}),
```

完整效果（主角 schema 尾部）：

```javascript
        形态库: z.record(z.string(), form_item).prefault({}),
        动态肉体档案: z.object({
            肉体外观: z.record(z.string(), z.string()).prefault({}),
            衣物服饰: z.record(z.string(), z.string()).prefault({}),
            情趣配饰: z.record(z.string(), z.string()).prefault({}),
            性器状态: z.record(z.string(), z.string()).prefault({}),
        }).prefault({}),
        当前形态: current_form
    }).prefault({}).transform(char => {
```

## 应用后验证

1. 确认 `npc_schema` 块内**没有** `动态肉体档案`；
2. 确认 `主角: z.object({` 与 `}).prefault({}).transform(char => {` 之间**有** `动态肉体档案`；
3. 在 SillyTavern 重新导入角色卡后，浏览器控制台执行：

```js
Mvu.getMvuData()?.stat_data?.主角?.动态肉体档案
```

手动编辑/让 AI 写入后，此处应能读到数据。
