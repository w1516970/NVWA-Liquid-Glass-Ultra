# NVWA 液态玻璃 Ultra

基于 [NVWA-Liquid-Glass](https://github.com/jjing8235-afk/NVWA-Liquid-Glass) 的二次性能优化版（在 Lite 版基础上再砍一刀）。
**核心思路：消息卡片彻底退出 SVG 滤镜体系，只保留纯 CSS 透明玻璃观感；顶栏 / 输入框 / 弹窗维持原版完整效果。**

> 适合谁：消息卡片折射一直是关闭的、只想留"透明磨砂玻璃"质感、长聊天 / 低端设备想要明显更流畅的用户。
> 如果你想要边缘折射彩边，请用原版或 [Lite 版](https://github.com/w1516970/NVWA-Liquid-Glass-Lite)。

## 安装

SillyTavern → 扩展 → Install extension，粘贴本仓库地址：

```
https://github.com/w1516970/NVWA-Liquid-Glass-Ultra
```

## 相对 Lite 版（v1）的改动

### 1. 消息卡纯 CSS 玻璃（最大的一刀）

v1 里每张消息卡仍要：一份 SVG 滤镜（20+ 节点）、一张逐像素 JS 位移图、一个 ResizeObserver、两个 IntersectionObserver、一次 getComputedStyle。一屏十几张卡就是十几条滤镜链常驻显存参与每帧合成。

v2 里消息卡**完全不进入滤镜体系**：只挂一个 `.ilg-mes-css` 类，由浏览器合成器直接处理：

- 一层半透明底（沿用酒馆主题的 `--SmartTheme*MesBlurTintColor`，深浅色 / 不透明度设置自动跟随）
- `backdrop-filter: blur(14px) saturate(1.35)`
- 1px 内亮边 + 单层浮起阴影
- 离屏卡片启用 `content-visibility: auto`（v1 因滤镜会被裁掉而不敢开）

### 2. 连带删除的机制（消息卡不再需要）

| 机制 | 说明 |
|---|---|
| 流式输出降级（busyEls） | 卡片没有滤镜，高度变化不再触发滤镜重算，整套机制连同 MutationObserver 删除 |
| io / ioFar 两个 IntersectionObserver | 没有滤镜需要预渲染 / 回收 |
| buildPhysical 的 lite 分支 | 「消息卡省电模式」开关移除（卡片已无滤镜可省） |
| tagCard 整树 getComputedStyle 遍历 | 改为限制深度 4 层、跳过内联标签，开销大幅下降；只维护"整卡不透明"标记 |

### 3. 设置项变化

- **移除**：「消息卡省电模式」（无用武之地）
- **「消息卡片也折射」默认改为关闭**，语义变为"用 CSS 模拟一点色散感"，不再建滤镜

### 4. 保持不动

顶栏、输入框、快捷回复、弹窗、抽屉、提示条、悬浮岛、首页、思考小球：与 Lite / 原版完全一致，仍是物理折射。

## 实测数据（Chromium，60 条消息，软件渲染沙箱）

| 指标 | Lite v1 | Ultra v2 |
|---|---|---|
| SVG 滤镜数 | 10 | **4**（仅顶栏/输入框等） |
| 滤镜内节点数 | 392 | **116** |
| 卡片引用 url(#) 滤镜 | 6/40 | **0** |
| DOM 节点 | 987 | **705** |
| 滚动中位帧耗时 | 77.8ms | **58.8ms**（约 -25%） |
| 视觉 | — | 截图对比：卡片观感一致 |

> 软件渲染沙箱绝对值偏低，仅供参考相对差异；真机（尤其手机）上滤镜驻留与离屏合成的差距会更明显。

## 与 Lite 版的关系

两版可独立安装（二选一），不要同时启用。Ultra 的所有上游优化（滚动自适应采样、楼层二分查找、鼠标高光节流、LRU 位移图缓存、card-clarity 降频等）都保留。
