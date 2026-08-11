# 卡内补丁 #1：ZOD schema 追加段

## 位置

角色卡 JSON `extensions.tavern_helper.scripts[]` 内 ZOD schema 脚本中 `主角` schema 内部。

## 锚点稳定性

schema 是协议层描述，原作者迭代频率远低于剧情/数值层。schema 末尾追加不修改原文任何一行。

## 补丁原文

在 `主角` schema 内部追加：

```javascript
动态肉体档案: z.object({
  肉体外观: z.record(z.string(), z.string()).prefault({}),
  衣物服饰: z.record(z.string(), z.string()).prefault({}),
  情趣配饰: z.record(z.string(), z.string()).prefault({}),
  性器状态: z.record(z.string(), z.string()).prefault({}),
}).prefault({})
```
