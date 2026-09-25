# CircleChat-MiniProgram

CircleChat 官方小程序的源码仓库，同时也是一个可直接被平台加载的**索引源**。

- `index.json` —— 索引清单，平台启动时会拉取并合并（默认官方源地址就是本文件的 raw 链接）
- `roll/` —— 掷骰子：`#roll`，演示 `message.send`
- `stats/` —— 会话统计：`#stats`，演示 `chat.read`
- `notes/` —— 会话便签：`#note`，演示 `kv.read` / `kv.write`

小程序本身是纯前端静态页面：没有构建步骤、没有依赖、没有后端，被平台放进沙箱 `iframe` 里运行。

## 默认源地址

```
https://raw.githubusercontent.com/CircleChat-Team/CircleChat-MiniProgram/main/index.json
```

部署者可以在「管理面板 → 小程序」里增删源（官方源 + 任意第三方 URL）。

## 加载 SDK

小程序托管在任意站点时，SDK 只能从**运行它的平台**加载，因此页面里用这段引导脚本定位平台地址：

```html
<script>
(function () {
  function safeOrigin(u) {
    try { var x = new URL(u); return (x.protocol === 'http:' || x.protocol === 'https:') ? x.origin : ''; } catch (e) { return ''; }
  }
  var origin = '';
  var m = /(?:^|[#&])ccsdk=([^&]+)/.exec(location.hash || '');   // 宿主显式传入（首选）
  if (m) origin = safeOrigin(decodeURIComponent(m[1]));
  if (!origin) origin = safeOrigin(document.referrer);            // 退回 referrer
  var s = document.createElement('script');
  s.src = origin + '/mini-sdk.js';
  s.onload = function () { window.dispatchEvent(new Event('minisdk-ready')); };
  document.head.appendChild(s);
})();
</script>
```

SDK 是异步加载的，业务代码等 `minisdk-ready` 事件再启动：

```js
function boot() { /* 用 CircleChat.* 写业务 */ }
if (window.CircleChat) boot();
else window.addEventListener('minisdk-ready', boot);
```

## 新增一个小程序

1. 新建目录，写好 `index.html`（可选 `icon.svg`）
2. 在 `index.json` 的 `apps` 里加一条 manifest（`id` 全局唯一，建议反向域名）
3. 提交到 `main` 分支，平台下次刷新索引即可看到

字段规格与 SDK API 见 <https://circlechat-team.github.io/CircleChat-Docs/>（小程序章节）。
