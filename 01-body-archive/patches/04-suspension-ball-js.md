# 卡外补丁 #4：悬浮球 JS secBlock 追加

## 文件

`01-body-archive/dist/V20260917/suspension-ball.js`

| 项 | 值 |
|---|---|
| 上游原版 | `Unspoken-MomoTea/Battlefield-of-Reincarnation@main/dist/V20260917/悬浮球状态栏.js`（674,311 B / 580,143 字符 / 9,906 行 / sha256 `5a7faf41…`） |
| 本 fork 基底 | **同上游原版**（V3.6.11 卡不再内联脚本，见下） |
| 产出 | 677,030 B / 582,540 字符 / 9,960 行 / sha256 `7936c4a9…` |

> **⚠️ V3.6.11 起基底策略变了**：V3.5.x 时代作者会把改过的悬浮球**内联**进卡，fork 必须拿"卡内联"当基底才不会丢作者的本地修正。
> V3.6.11 的卡已改为
> ```
> import 'https://cdn.jsdelivr.net/gh/Unspoken-MomoTea/Battlefield-of-Reincarnation@main/dist/V20260917/悬浮球状态栏.js?v=2'
> ```
> 直接引用上游 CDN —— 卡里**没有**内联脚本了，所以**基底直接用上游 V20260917 原版**。
> 迁移时必须先确认新卡是"内联型"还是"import 型"：前者用卡内联，后者用上游原版 + 核对作者修正是否已合并回上游。
> （V3.5.6 时代那些本地修正 `心里话`→`态度`、资产类型图标、商城提示词档位等，在 V20260917 上游里已经合入：实测 `心里话` 0 次、`态度` 12 次。）

## 改动位置（V20260917 实测行号）

| 段 | 位置 | 锚点 |
|---|---|---|
| ① 事件绑定段 | 事件初始化区内、`// ★ 选择世界按钮(顶栏, …)` 之前 | 基底 line 3501 |
| ② 信息页渲染段 | `renderInfoTab(sd)` 内、`return html;` 之前 | 基底 line 5185（`// 注: "当前形态"栏已移除…` 之后） |

两段都是**纯插入**，不改动上游任何一行（构建脚本会断言「与基底的差异恰好是两段 insert」）。

## 改动性质

纯 UI 展示追加。在 `renderInfoTab` 返回前追加 4 个 secBlock，渲染 `角色.动态肉体档案` 下的 record key-value 表格。

- 不修改原版任何逻辑
- 不引入新依赖（复用 `secBlock` / `editInput` / `esc` / `safeStr` / `sam-fc-del-btn` / `sam-grid-2` / `sam-row` / `sam-empty`；均已在上游 V20260917 中存在）
- `try/catch` 包裹，异常时只日志不报错

> **路径已随变量改名同步**：挂载路径为 `角色.动态肉体档案.<子档案>.<槽位>`（V3.5.x 时代是 `主角.…`）。
> `renderInfoTab` 里 `var p = sd.角色 || {}`，所以补丁直接用 `p.动态肉体档案`，无需再写根名 —— 但 `data-add-path` / `data-del-path` 里写的是**完整点路径字符串**，必须带 `角色.` 前缀。

## 补丁代码（两段）

### ① 事件绑定段（插在「选择世界按钮」绑定之前）

```javascript
        // ★ 动态肉体档案: 添加槽位按钮(编辑模式)
        $panel.off("click.samAddKv").on("click.samAddKv", ".sam-add-kv-btn[data-add-path]", function(e) {
            e.stopPropagation();
            var basePath = $(this).attr("data-add-path") || "";
            if (!basePath) return;
            var newKey = window.prompt("新槽位名（key）：");
            if (!newKey) return;
            var val = window.prompt("初始值（可留空）：") || "";
            var ok = writeBackMvu(function(statData) {
                _.set(statData, basePath + "." + newKey, val);
            });
            if (ok) {
                samToast("success", "已添加: "+newKey);
                renderAll();
            } else {
                samToast("error", "添加失败: MVU写回不可用");
            }
        });
```

> 删除按钮复用上游既有的 `$panel.off('click.samFcDel')…'.sam-fc-del-btn[data-del-path]'` 委托（V20260917 基底 line 3466），
> 无需新增绑定；本补丁只补「添加槽位」按钮。
> `window.prompt` 在 about:srcdoc iframe 中的可用性仍需在 SillyTavern 实测（上游 CDN 源码 0 处使用 prompt）。

### ② 信息页渲染段（插在 `return html;` 之前）

```javascript
        // === 动态肉体档案（独立补丁，可整段删除不影响其他功能）===
        try {
            var archive = p.动态肉体档案 || {};
            var archSections = [
                {key:'肉体外观', icon:'🧍'},
                {key:'衣物服饰', icon:'👕'},
                {key:'情趣配饰', icon:'💎'},
                {key:'性器状态', icon:'🔥'}
            ];
            archSections.forEach(function(sec) {
                var sub = archive[sec.key] || {};
                var keys = Object.keys(sub);
                var body = '<div class="sam-grid-2">';
                if (keys.length === 0) {
                    body += '<div class="sam-empty">[空]</div>';
                }
                keys.forEach(function(k) {
                    var v = safeStr(sub[k]);
                    var path = '角色.动态肉体档案.' + sec.key + '.' + k;
                    var valCell = editMode ? editInput(path, v, 'text') : esc(v);
                    var delBtn = editMode
                        ? '<button type="button" class="sam-fc-del-btn" data-del-path="'+esc(path)+'">✕</button>'
                        : '';
                    body += '<div class="sam-row"><span class="k">'+esc(k)+'</span>'
                          + '<span class="v">'+valCell+delBtn+'</span></div>';
                });
                body += '</div>';
                if (editMode) {
                    body += '<button type="button" class="sam-add-kv-btn" '
                          + 'data-add-path="角色.动态肉体档案.'+sec.key+'">+ 添加槽位</button>';
                }
                html += secBlock(sec.icon + ' ' + sec.key, body);
            });
        } catch (e) {
            console.warn('[主神终端] 动态肉体档案渲染失败:', e);
        }
```

## 重建步骤（原作者发新 JS / 新卡时）

1. 判断新卡是「import 型」还是「内联型」：
   - import 型（V3.6.11+）→ 基底 = 上游对应版本的 `悬浮球状态栏.js`
   - 内联型（V3.5.x-）→ 基底 = 卡内 `name: "悬浮球状态栏"` 的 `content`
2. 断言两个锚点各出现 1 次：
   - `        // ★ 选择世界按钮(顶栏, 仅在主神空间且非战斗时渲染): 点击发送【选择世界】到输入框`
   - `        // 注: "当前形态"栏已移除 — 顶部头像旁已显示形态名, 由能力面板激活按钮统一管理`
   - 且第二个锚点紧跟 `        return html;`
3. 按上表位置插入两段代码（路径前缀按当前卡的实际根名：V3.6.11+ 用 `角色.`）→ 写入 `dist/<版本>/suspension-ball.js`
4. 校验（见下）。

> 参考实现：`tmp/build_fork_v20260917.py`（读基底 → 断言两锚点 → 插入两段 hunk → 断言"回滚 hunk 后与基底逐字相同" → `?v=N` 递增）。

## 验证命令

```bash
node --check dist/V20260917/suspension-ball.js               # 语法校验（应为 0）
python tmp/build_fork_v20260917.py                           # 重建并断言差异只有 2 段 insert
```

行数/体量基线：

| 文件 | 字符 | 行数 | 字节 |
|---|---|---|---|
| 卡内联（V3.5.6 基底） | 545,953 | 9,396 | 610,657 |
| 上游 V20260901 | 545,662 | 9,391 | — |
| fork V20260831（旧，V3.5.1 基底） | 536,898 | 9,257 | — |
| fork V20260901（历史） | 548,350 | 9,450 | 639,065 |
| 上游 V20260917 | 580,143 | 9,906 | 674,311 |
| **fork V20260917（本版）** | **582,540** | **9,960** | **677,030** |
