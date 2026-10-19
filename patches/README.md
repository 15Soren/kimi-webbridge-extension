# patches/custom-provider.patch

本地补丁：在官方 **2.0.26** 构建产物上加入「自定义 OpenAI 兼容提供商」，并禁用扩展的自动更新检查。

## 基线

补丁基于官方 **Kimi WebBridge 2.0.26** 的解包产物（纯 LF，未做任何改动）。
本仓库当前状态 = 基线 + 本补丁，四个被改文件与仓库对应文件逐字节一致。

## 用法

扩展自动升级后 `.js` / `.css` 产物会被覆盖。升级后在该目录执行：

```bash
git -c core.autocrlf=false apply --whitespace=nowarn patches/custom-provider.patch
```

Windows 上务必带 `-c core.autocrlf=false`。本机 `core.autocrlf=true`，不加这个参数 git 会在写入时把 LF 转成 CRLF，产物与仓库不再一致（补丁本身仍能应用，但字节变了）。

升级后若上游改动同几行，`apply` 会报冲突，改用三路合并：

```bash
git -c core.autocrlf=false apply --3way patches/custom-provider.patch
```

## 补丁内容

| 文件 | 改动 |
| --- | --- |
| `background.js` | ① 预设表加 `openai`；② base URL 规整（`wbBaseUrlOf`）；③ `manual-models` discovery（不请求 `/models`）；④ 连接解析支持 `models[]`；⑤ `contextWindow` / `vision` 透传；⑥ 推理强度去掉 `route === 'kimi'` 限制；⑦ **禁用自动更新检查** |
| `chunks/sidepanel-CUbsl7eV.js` | 同一套预设与 discovery；设置页「模型提供商」面板（提供商 + 多模型行 + 强度 chips）；模型菜单支持自定义提供商 |
| `chunks/i18n-pVQL_Rp2.js` | 中英文文案 |
| `assets/sidepanel-DD7JAQkM.css` | 模型行与强度 chip 样式 |

## 禁用自动更新检查

上游原本每 6 小时访问一次 `https://cdn.kimi.com/webbridge/latest/version.json`，把结果写进
`chrome.storage.local.manual_update_check`，设置页据此显示「新版本」小红点。

补丁把 `sV()` 改成：

```js
function sV(e={}){
  try{chrome.alarms.clear(nV)}catch{}
  try{chrome.alarms.clear(`manual-update-check`)}catch{}
  try{chrome.storage.local.remove(QB)}catch{}
}
```

`nV` / `QB` 分别是闹钟名和缓存键。效果：

- 不创建 6 小时定时器
- 启动时不再立即检查
- 清掉旧版本遗留的闹钟和缓存（设置页的红点随之消失）

验证方式：加载扩展后检查 `chrome.alarms.getAll()` 只剩 `webbridge-reconcile` 和 `install-popup`，
`manual_update_check` 为空，Service Worker 全程没有对 `cdn.kimi.com` 的请求。

注意：这只关掉**扩展自己发起的版本探测**。浏览器对扩展的更新（商店渠道）由浏览器管理，不受影响；
以「加载解压缩的扩展」方式安装时本来就不会自动升级。

## 升级到更新版本时

1. 把新版解包产物覆盖到本目录
2. 用本补丁的 `a/` 侧重新比对，确认改动位置是否漂移
3. 冲突时按 `--3way` 处理，重点保留上表中的七处逻辑

## 验证

在一份未改动的 2.0.26 副本上应用本补丁后，四个文件 md5 与仓库全部一致。
