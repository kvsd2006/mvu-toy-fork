# 卡外补丁：悬浮球 JS secBlock 追加

## 文件

`01-动态肉体档案/dist/V20260809/悬浮球状态栏.js`
（上游原版：`Unspoken-MomoTea/Battlefield-of-Reincarnation@main/dist/V20260809/悬浮球状态栏.js`）

## 改动位置

函数：`renderInfoTab(sd)`（上游行号 4252–4353）
插入点：`return html;`（上游行号 4352）之前，`// 注: "当前形态"栏已移除...` 注释之后

## 改动性质

纯 UI 展示追加。在 `renderInfoTab` 返回前追加 4 个 secBlock，渲染 `主角.动态肉体档案` 下的 record key-value 表格。

- 不修改原版任何逻辑
- 不引入新依赖（复用 `secBlock` / `editInput` / `sam-fc-del-btn` / `sam-grid-2` / `sam-row`）
- `try/catch` 包裹，异常时只日志不报错
- 空档案时连标题都不展示

## 补丁代码

在 `renderInfoTab(sd)` 函数内、`return html;` 之前插入：

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
                if (!editMode && keys.length === 0) return;
                var body = '<div class="sam-grid-2">';
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

## 配套事件绑定

在悬浮球 JS 现有事件初始化区（搜 `sam-fc-del-btn`），旁边追加：

```javascript
$(document).on('click', '.sam-add-kv-btn', function() {
    var basePath = $(this).data('add-path');
    var newKey = window.prompt('新槽位名（key）：');
    if (!newKey) return;
    var val = window.prompt('初始值（可留空）：') || '';
    writeBackMvu(function(cloned) {
        _.set(cloned, basePath + '.' + newKey, val);
    });
});
```

> **注意**：`window.prompt` 在 about:srcdoc iframe 中可能被禁用（CDN 源码中 0 处使用），需在 SillyTavern 实测。若被禁，改用面板内 `<input>` 弹窗。

## 升级到新版 JS 时的 cherry-pick 步骤

1. 拉上游新版 JS → 定位 `renderInfoTab(sd)` → 找到 `return html;`
2. 把补丁代码贴到 `return html;` 之前
3. 把事件绑定贴到 `$(document).on('click', '.sam-fc-del-btn', ...)` 旁边
4. `node --check` 验证语法
5. 对比 SHA256 确认不同于上游

## 验证命令

```bash
node --check dist/V20260809/悬浮球状态栏.js   # 语法校验
sha256sum dist/V20260809/悬浮球状态栏.js        # 应与上游不同
wc -l dist/V20260809/悬浮球状态栏.js            # 应比上游多约 34 行（8530 vs 8496）
```
