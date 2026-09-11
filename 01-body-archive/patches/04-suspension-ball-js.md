# 卡外补丁 #4：悬浮球 JS secBlock 追加

## 文件

`01-body-archive/dist/V20260901/suspension-ball.js`

| 项 | 值 |
|---|---|
| 上游原版 | `Unspoken-MomoTea/Battlefield-of-Reincarnation@main/dist/V20260901/悬浮球状态栏.js`（626,585 B / 9,391 行 / sha256 `a2892edf…dcf3`） |
| 本 fork 基底 | **轮回战场 重构版 V3.5.6 卡内联脚本**（= 上游 V20260901 + 作者本地修正，545,953 字符 / 9,396 行） |
| 产出 | 548,350 字符 / 9,450 行（sha256 `fb38af8c…7c57`） |

> 基底为什么用卡内联而不是上游原版：V3.5.6 卡内联脚本比上游 V20260901 多出约 290 字符，
> 是作者随卡发布的本地修正（`心里话`→`态度` 四处、资产类型图标兼容、商城/血统融合提示词模板档位调整等）。
> 用上游原版做基底会丢掉这些修正（例如 NPC 详情面板会重新去找已经改名的 `心里话`）。

## 改动位置（V3.5.6 基底实测行号）

| 段 | 位置 | 锚点 |
|---|---|---|
| ① 事件绑定段 | 事件初始化区内、`// ★ 选择世界按钮(顶栏, …)` 之前 | 基底 line 3406 |
| ② 信息页渲染段 | `renderInfoTab(sd)` 内、`return html;` 之前 | 基底 line 4971（`// 注: "当前形态"栏已移除…` 之后） |

两段都是**纯插入**，不改动上游任何一行（构建脚本会断言「与卡内联的差异恰好是两段 insert」）。

## 改动性质

纯 UI 展示追加。在 `renderInfoTab` 返回前追加 4 个 secBlock，渲染 `主角.动态肉体档案` 下的 record key-value 表格。

- 不修改原版任何逻辑
- 不引入新依赖（复用 `secBlock` / `editInput` / `esc` / `safeStr` / `sam-fc-del-btn` / `sam-grid-2` / `sam-row` / `sam-empty`；均已在上游 V20260901 中存在）
- `try/catch` 包裹，异常时只日志不报错

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

> 删除按钮复用上游既有的 `$panel.off('click.samFcDel')…'.sam-fc-del-btn[data-del-path]'` 委托，
> 无需新增绑定；本补丁只补「添加槽位」按钮。
> `window.prompt` 在 about:srcdoc iframe 中的可用性需在 SillyTavern 实测（上游 CDN 源码 0 处使用 prompt）。

### ② 信息页渲染段（插在 `return html;` 之前）

```javascript
        // === 动态肉体档案（独立补丁，可整段删除不影响其他功能）===
        try {
            var archive = p.动态肉体档案 || {};
            var archSections = [
                {key:'肉体外观', icon:'💪'},
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
                    var path = '主角.动态肉体档案.' + sec.key + '.' + k;
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
                          + 'data-add-path="主角.动态肉体档案.'+sec.key+'">+ 添加槽位</button>';
                }
                html += secBlock(sec.icon + ' ' + sec.key, body);
            });
        } catch (e) {
            console.warn('[主神终端] 动态肉体档案渲染失败:', e);
        }
```

## 重建步骤（原作者发新 JS / 新卡时）

1. 从新卡内联脚本取出 `name: "悬浮球状态栏"` 的 `content` 作为基底（不要用上游原版，除非卡内联与上游一致）。
2. 断言两个锚点各出现 1 次：
   - `        // ★ 选择世界按钮(顶栏, 仅在主神空间且非战斗时渲染): 点击发送【选择世界】到输入框`
   - `        // 注: "当前形态"栏已移除 — 顶部头像旁已显示形态名, 由能力面板激活按钮统一管理`
3. 按上表位置插入两段代码 → 写入 `dist/<版本>/suspension-ball.js`。
4. 校验（见下）。

> 参考实现：`tmp/build_fork_v20260901.py`（读取卡内联 + 从上一版 fork dist 提取两段 hunk + 插入 + 断言差异只有两段 insert）。

## 验证命令

```bash
node --check dist/V20260901/suspension-ball.js                 # 语法校验（应为 0）
python -c "..."                                                # 与卡内联 diff 应只有 2 段 insert
```

行数/体量基线：

| 文件 | 字符 | 行数 |
|---|---|---|
| 卡内联（V3.5.6 基底） | 545,953 | 9,396 |
| 上游 V20260901 | 545,662 | 9,391 |
| fork V20260831（旧，V3.5.1 基底） | 536,898 | 9,257 |
| **fork V20260901（本版）** | **548,350** | **9,450** |
