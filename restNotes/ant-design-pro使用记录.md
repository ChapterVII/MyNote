---
title: ant-design-pro 使用记录
date: 2018-11-27 11:26:41
tags:
---

dva new project

Roadhog:配置.roadhogrc

1. extraBabelPlugins:      

 dva-hmr: 基于 [babel-plugin-dva-hmr](https://github.com/dvajs/babel-plugin-dva-hmr) 实现 components、routes 和 models 的 HMR(Hot Module Replacement热模块重载)

<https://dvajs.com/guide/#%E7%89%B9%E6%80%A7>

transform-runtime：所有的 helper 都会引用模块 babel-runtime，以避免编译输出的重复问题。这个运行时会被编译到你的构建版本当中。另外一个目的就是为你的代码创建一个沙盒环境。

<https://www.babeljs.cn/docs/plugins/transform-runtime/>

transform-decorators-legacy：Babel 6的插件：复制了Babel 5中的旧装饰器行为

<https://github.com/loganfsmyth/babel-plugin-transform-decorators-legacy>

 babel-plugin-import:  进行按需加载