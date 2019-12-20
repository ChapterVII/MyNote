---
title: Vue源码学习（一）
date: 2018-10-30 20:10:22
tags: Vue
---

# 前期工作

将vue源码下载到source目录下

```bash
git clone git@github.com:vuejs/vue.git source
```

版本=2.5.0

# 入口提取

1. package.json关键字

   "module": "dist/vue.runtime.esm.js" => 以ES方式引用的文件

   script的"build": "node build/build.js" => 执行build文件夹下的build.js构建

2. build.js => 主要功能就是用rollup工具利用config.js的配置进行构建

3. config.js => 可看到entry为web/entry-runtime.js

   ```
   const builds = {
   	...
   	'web-runtime-esm': {
       	entry: resolve('web/entry-runtime.js'),
       	dest: resolve('dist/vue.runtime.esm.js'),
       	format: 'es',
       	banner
     	},
     	...
   }
   ```

   考虑到使用了alias，别名为web的路径 => src/platforms/web

   确定文件构建入口 => src/platforms/web/entry-runtime.js

4. src/platforms/web/entry-runtime.js => 只是 import Vue from './runtime/index'

5. src/platforms/web/runtime/index 做了以下几件事

   - import Vue from 'src/core/index'
   - 修改Vue.config的某些属性
   - 调用src/shared的extend方法
   - 在Vue.prototype上添加\__patch__和$mount方法
   - 执行Vue.nextTick()，传入了回调

6. src/core/index

   - import Vue from './instance/index'
   - 执行initGlobalAPI方法，传入Vue
   - 在Vue.prototype上定义了两个属性\$isServer和\$ssrContext的get方法
   - 给Vue增加了version属性

7. src/core/instance/index

   - 导入了一些mixin方法
   - 声明了function Vue
   - 执行导入的mixin方法，传入Vue

至此，确定了文件入口就是**src/core/instance/index**

# 文件加载顺序

通过从src/core/instance/index开始看一层一层的导入顺序，可确定顺序为(src/)

1. shared/util
2. shared/constants
3. core/config
4. core/util/lang
5. core/util/debug
6. core/util/error
7. core/util/env
8. core/observer/dep
9. core/vdom/vnode
10. core/observer/array
11. core/observer/index
12. core/util/options
13. core/util/props
14. core/instance/proxy
15. core/util/perf
16. core/vdom/helpers/update-listeners
17. core/vdom/helpers/merge-hook
18. core/vdom/helpers/extract-props
19. core/vdom/helpers/normalize-children
20. core/vdom/helpers/resolve-async-component
21. core/vdom/helpers/is-async-placeholder
22. core/vdom/helpers/get-first-component-child
23. core/instance/events
24. core/instance/render-helpers/resolve-slots
25. core/instance/lifecycle
26. core/observer/scheduler
27. core/observer/watcher
28. core/instance/state
29. code/instance/inject
30. core/instance/render-helpers/render-list
31. core/instance/render-helpers/render-slot
32. core/instance/render-helpers/resolve-filter
33. core/instance/render-helpers/check-keycodes
34. core/instance/render-helpers/bind-object-props
35. core/instance/render-helpers/render-static
36. core/instance/render-helpers/bind-object-listeners
37. core/instance/render-helpers/index
38. core/vdom/create-functional-component
39. core/vdom/create-component
40. core/vdom/create-element
41. core/instance/render
42. core/instance/init
43. core/instance/index
44. core/global-api/use
45. core/global-api/mixin
46. core/global-api/extend
47. core/global-api/assets
48. core/components/keep-alive
49. core/global-api/index
50. core/index
51. platforms/web/util/attrs
52. platforms/web/util/class
53. platforms/web/util/element
54. platforms/web/util/index
55. platforms/web/runtime/node-ops
56. core/vdom/modules/ref
57. core/vdom/patch
58. core/vdom/modules/directives
59. platforms/web/runtime/modules/attrs
60. platforms/web/runtime/modules/class
61. platforms/web/compiler/directives/model
62. platforms/web/runtime/modules/events
63. platforms/web/runtime/modules/dom-props
64. platforms/web/util/style
65. platforms/web/runtime/modules/style
66. platforms/web/runtime/class-util
67. platforms/web/runtime/transition-util
68. platforms/web/runtime/modules/transition
69. platforms/web/runtime/patch
70. platforms/web/runtime/directives/model
71. platforms/web/runtime/directives/show
72. platforms/web/runtime/components/transition
73. platforms/web/runtime/components/transition-group
74. platforms/web/runtime/index

