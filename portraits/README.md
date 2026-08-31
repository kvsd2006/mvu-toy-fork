# portraits — 角色头像素材

存放**小尺寸角色头像**，供 SillyTavern 插件通过 URL 引用（例如把图片作为角色头像插入到对话里）。

## URL 规则

push 到 GitHub 后，本目录文件通过 jsDelivr 变成固定 URL：

```
https://cdn.jsdelivr.net/gh/kvsd2006/mvu-toy-fork@main/portraits/<文件名>?v=1
```

- 国内访问可把 `cdn` 换成 `testingcf`（Cloudflare 线路），需要时可再试 `fastly` / `gcore`；
- 修改图片后把 `?v=1` 递增（`?v=2`…）刷新 CDN 缓存；
- 也可访问 `https://purge.jsdelivr.net/gh/kvsd2006/mvu-toy-fork@main/portraits/<文件名>` 手动强刷。

## 引用示例

在插件里填图片 URL 即可（Markdown 写法等价于插件的 URL 输入）：

```
![主角](https://cdn.jsdelivr.net/gh/kvsd2006/mvu-toy-fork@main/portraits/protagonist.png?v=1)
```

## 文件规范

- 尺寸：正方形，256×256 起步，最大 512×512；
- 格式：PNG 或 WebP；
- 体积：单张 ≤ 100 KB（越小加载越快，建议 30–60 KB）；
- 命名：小写英文 + 短横线，如 `protagonist.png`、`npc-blacksmith.png`；
- 本目录只放小图头像，大尺寸立绘请放其它目录。

## 素材清单

| 文件名 | 用途 | 状态 |
|---|---|---|
| `protagonist.png` | 主角头像（示例槽位，可改名） | 待添加 |
| `npc-<名字>.png` | 常用 NPC 头像（按需命名） | 待添加 |
