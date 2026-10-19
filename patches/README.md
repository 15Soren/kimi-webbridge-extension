# patches/custom-provider.patch

本地补丁：在官方 2.0.24 构建产物上加入「自定义 OpenAI 兼容提供商」。

## 基线

补丁基于官方 **Kimi WebBridge 2.0.24** 的解包产物（纯 LF，未做任何改动）。
本仓库当前状态 = 基线 + 本补丁，所以四个文件与本仓库对应文件逐字节一致。

## 用法

扩展自动升级后会覆盖 `background.js` / `chunks/*` / `assets/*`。升级后在该目录执行：

```bash
git -c core.autocrlf=false apply --whitespace=nowarn patches/custom-provider.patch
```

Windows 上务必带 `-c core.autocrlf=false`。本机 `core.autocrlf=true`，若不加这个参数，git 会在写入时把 LF 转成 CRLF，导致产物与仓库不一致（补丁本身能应用，但字节变了）。

升级后若上游改动同几行，`apply` 会报冲突。此时用：

```bash
git -c core.autocrlf=false apply --3way patches/custom-provider.patch
```

冲突文件按标记手工合并，重点是保留三处逻辑：
- `bc` / `fE` 预设表里的 `openai` 项与 `manual-models` discovery
- `Dc` / `wbBaseUrlOf` 的 base URL 规整
- 推理强度：`ou` / `Aj` 去掉 `route === 'kimi'` 限制，`RT` 只发 `reasoning_effort`

## 补丁内容

| 文件 | 改动 |
| --- | --- |
| `background.js` | 预设表加 `openai`；base URL 规整；`manual-models` discovery；连接解析支持 `models[]`；`contextWindow`/`vision` 透传；推理强度去 kimi 限制 |
| `chunks/sidepanel-CUbsl7eV.js` | 同一套预设与 discovery；设置页「模型提供商」面板（提供商 + 多模型行 + 强度 chips）；模型菜单支持自定义提供商 |
| `chunks/i18n-pVQL_Rp2.js` | 中英文文案 |
| `assets/sidepanel-DD7JAQkM.css` | 模型行与强度 chip 样式 |

## 验证

在一份未改动的 2.0.24 副本上应用本补丁后，四个文件与本仓库 md5 全部一致。
