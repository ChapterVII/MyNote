---
title: 不常用的CSS
date: 2018-11-28 10:58:24
tags:
---

text-rendering： 定义浏览器渲染引擎如何渲染字体。浏览器会在速度、清晰度、几何精度之间进行权衡。一个视觉上很明显的效果是，optimizeLegibility
属性值会在某些字体（比如，微软的 Calibri，Candara，Constantia
和 Corbel 或者 DejaVu 系列字体）小于20px 时把有些相邻字符连接起来（ligatures，比如，ff、fi、fl 等) 

**font-smooth** ：用来控制字体渲染时的平滑效果。

**Webkit** 实现了名为-webkit-font-smoothing的相似属性。这个属性仅在 Mac OS X/macOS 下生效。

- none -      关闭字体平滑；展示有锯齿边缘的文字。
- antialiased -      平滑像素级别的字体，而不是子像素。从亚像素渲染切换到黑暗背景上的浅色文本的抗锯齿使其看起来更轻。
- subpixel-antialiased -      在大多数非视网膜显示器上，这将会提供最清晰的文字。

**Firefox** 实现了名为 -moz-osx-font-smoothing 的相似属性。这个属性仅在 Mac OS X / macOS 下生效。

- auto -      允许浏览器选择字体平滑的优化方式，通常为grayscale。
- grayscale -      用灰度抗锯齿渲染文本，而不是子像素。从亚像素渲染切换到黑暗背景上的浅色文本的抗锯齿使其看起来更轻。
- [inherit](https://developer.mozilla.org/zh-CN/docs/Web/CSS/inherit)
- [unset](https://developer.mozilla.org/zh-CN/docs/Web/CSS/unset)

 