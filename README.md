# 当前仓库改动说明

本文档用于说明当前工作区相对 `HEAD` 的实际改动，不再保留原仓库的产品介绍内容。

## 改动概览

- 已修改文件
  - `dist/chrome/content_script.js`
  - `dist/firefox/content_script.js`
  - `dist/userscript/immersive-translate.user.js`
- 未跟踪文件
  - `dist/chrome.zip`
  - `dist/chrome/_metadata/generated_indexed_rulesets/_ruleset1`

## 核心变更

这次变更集中在内容脚本的 DOM mutation 处理逻辑，Chrome、Firefox 与 userscript 三个构建产物做了相同的同步更新。

新增了两个用于识别和跳过“五彩高亮标记”节点的辅助函数：

- `imtHasWucaiMarker`
  - 判断元素是否为 `markerow*` 标签。
  - 判断元素或其子节点的 class 中是否包含 `wucaiclr`。
- `imtSkipWucaiMutation`
  - 检查当前节点、`mutation.target`、以及 `addedNodes` / `removedNodes` 中是否存在上述标记。
  - 只要命中标记节点，就在后续 mutation 处理前直接跳过。

随后在原有的 `yq(...)` 判断链中插入了：

```js
imtSkipWucaiMutation(t, e)
```

这意味着当页面变更来自五彩高亮相关 DOM 时，翻译脚本不会继续把这类节点当作普通页面内容处理。

## 变更目的

从当前代码差异推断，这次修改的目标是：

- 避免五彩高亮插件插入的标记节点触发沉浸式翻译的重复解析或误处理。
- 降低无关 mutation 对翻译观察器的干扰。
- 保持三套发布产物在行为上一致。

## 产物状态

- `dist/chrome.zip` 为本地存在但尚未纳入版本控制的打包产物，当前大小约为 12 MB。
- `dist/chrome/_metadata/generated_indexed_rulesets/_ruleset1` 为未跟踪的 Chrome 扩展元数据文件。

## 结论

当前工作区的有效代码变更，本质上是为 mutation 观察流程补充了一层针对五彩高亮标记节点的过滤保护，并已同步体现在 Chrome、Firefox 与 userscript 的构建输出中。
