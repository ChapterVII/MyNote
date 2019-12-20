---
title: Vue源码学习（二）
date: 2018-10-31 19:35:04
tags: Vue
---

# shared

## util

 声明了一些工具函数

### isUndef

- 判断参数是否为undefined或null

### isDef

- 判断参数是否不为undefind，并且不为null

### isTrue

- 判断参数是不是true

### isFalse

- 判断参数是不是false

### isPrimitive

- 判断参数的类型是否是原始类型(string/number/boolean之一)

### isObject

- 判断参数是否是object并且不是null
- typeof null === 'object'

### toRawType

- 获取参数的原始类型字符串
- Object.prototype.toString.call()

### isPlainObject

- 判断是否是纯js对象

### isRegExp

- 判断参数是否是正则表达式

### isValidArrayIndex

- 检查参数是不是合法的数组索引(非负、整数、有限大)

### toString

- 把参数转换成可渲染的字符串
- Undefined == null
- JSON.stringify()可接受三个参数

### toNumber

- 把参数转换成数字，失败的话，返回原参数

### makeMap

- 用传入的第一个字符串参数生成一个map，返回一个函数来检查键是不是在设个map中，可以区分大小写
- 柯里化+闭包

### isBuiltInTag

- 用makeMap生成的函数：用来检查是否是内建标签（slot,component）

### isReservedAttribute

- 用makeMap生成的函数：用来检查是否是保留属性（key,ref,slot,slot-scope,is）

### remove

- 从数组中删除某项，返回被删除项

### hasOwn

- 检查一个对象是否包含某属性，不包括原型链上的
- 使用Object.prototype.hasOwnProperty.call()的原因：对象上的hasOwnProperty是可以被改写的

### cached

- 创建一个纯函数的缓存版本，返回了一个函数cachedFn
- cachedFn调用时，如果cache[str]有值，则直接返回该值，否则执行计算并将结果存到cache[str]中
- 因为一个纯函数的返回值只跟它的参数有关，所以可以将入参作为键，返回值作为值，缓存成键值的键值对，这样如果之后还想获取之前同样参数的计算结果时，不需要再重新计算了，直接获取之前计算过的结果就可以
- 纯函数：一个函数的返回结果只依赖于它的参数，并且在执行过程中没有副作用

### camelize

- 用cached生成的函数：用来把连字符分隔的字符串转换为驼峰式

### capitalize

- 用cached生成的函数：用来将字符串首字符转为大写字母

### hyphenate

- 用cached生成的函数：用来把驼峰式字符串转为连字符分隔的字符串

### bind

- 比原生bind还快的bind，返回boundFn方法
- 因为有的环境不支持原生bind
- call、apply

### toArray

- 把类数组对象转换为真正的数组

### extend

- 把源对象的属性混入到目标对象中，返回目标对象

### toObject

- 把对象数组合并到一个单一对象中，并返回这个单一对象

### noop

- 空函数
- 插入的参数是为了避免Flow使用rest操作符…产生无用的转换代码

### no

- 永远返回false的函数

### identity

- 永远返回自身的函数

### genStaticKeys

- 用compiler的modules生成静态键字符串
- 接受一个数组，把所有数组项的staticKeys的值拼成一个数组，在转换成逗号分隔的字符串
- reduce

### looseEqual

- 判断两个值是否宽松相等（内部结构和数据）
- every

### looseIndexOf

- 用looseEqual，返回元素在数组中的索引，没找到则返回-1

### once

- 确保函数只执行一次，返回一个函数
- 闭包

## constants

定义了三个常量

```js
const SSR_ATTR = 'data-server-rendered';
const ASSET_TYPES = [ 'component', 'directive', 'filter'];
const LIFECYCLE_HOOKS = ['beforeCreate', 'created', 'beforeMount', 'mounted', 'beforeUpdate', 'updated', 'beforeDestroy', 'destroyed', 'activated', 'deactivated', 'errorCaptured']
```

# core/config

```js
const config = {
    // user配置项
    optionMergeStrategies， // 自定义合并策略的选项
    silent， // 取消 Vue 所有的日志与警告
    productionTip， // 阻止 vue 在启动时生成生产提示
    devtools， // 允许 vue-devtools 检查代码
    performance， // 在浏览器开发工具的性能/时间线面板中启用对组件初始化、编译、渲染和打补丁的性能追踪
    errorHandler， // 指定组件的渲染和观察期间未捕获错误的处理函数
    warnHandler， // 为 Vue 的运行时警告赋予一个自定义处理函数
    ignoredElements， // 忽略在 Vue 之外的自定义元素
    keyCodes， // 给 v-on 自定义键位别名
    
    // platform配置项
    isReservedTag， // 检查标签是否保留，以便不能将其注册为组件。取决于平台，可能会被覆盖。
    isReservedAttr， // 检查一个属性是否保留，以便不能将其注册为组件属性。取决于平台，可能会被覆盖。
    parsePlatformTagName， // 解析特定平台的真实标记名称
    isUnknownElement， // 检查标签是否未知，取决于平台
    getTagNamespace， // 获取元素命名空间
    mustUseProp， // 检查属性是否必须使用属性绑定，取决于平台
    
    // legacy
    _lifecycleHooks // 由于遗留原因而暴露
}
```

