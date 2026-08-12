# 卡内补丁 #3：`[variables]当前变量` EJS 追加段

> **⚠️ P0 关键 — 不加此补丁 AI 看不到动态肉体档案**

## 位置

角色卡世界书条目 `[variables]当前变量`（卡内 1780–1821 行），在构建 `current.主角` 对象的代码块末尾。

## 原因

该 EJS 的 `current.主角` 是**逐字段手写**的（`{ 种族, 身份, 职业, 空间币, HP, THP, EP }`），不是对 `data.主角` 整体展开。不加此补丁，`动态肉体档案` 不会出现在 `<status_current_variables>` 里，AI 能写但**读不到**当前值。

## 锚点

找到 `current.主角` 的构建块（约在 `EP: data.主角.EP` 行），在 `EP` 行之后追加。

## 补丁原文

在 `EP: data.主角.EP` 之后追加：

```javascript
,
  动态肉体档案: data.主角.动态肉体档案
```

完整效果（`current.主角` 块尾部）：

```javascript
current.主角 = {
  种族: data.主角.种族,
  身份: data.主角.身份,
  职业: data.主角.职业,
  空间币: data.主角.空间币,
  HP: data.主角.HP,
  THP: data.主角.THP || 0,
  EP: data.主角.EP,
  动态肉体档案: data.主角.动态肉体档案
};
```

## 注意

空 record 序列化为 `{}` 几乎不占 token，直接带上即可。
