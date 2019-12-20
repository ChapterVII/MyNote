---
title: Vue源码学习（三）
date: 2018-11-1 18:22:54
tags: Vue
---

# core/util

## lang

### emptyObject

- 冻结对象，不可扩展，不可配置
- Object**.**freeze()

### isReserved

- 检查传入字符串是否是以$或_开头
- charCodeAt
- $: 36 = 0x24; _ : 95 = 0x5F

### def

- 给对象定义一个可写可配置的属性
- Object.defineProperty

### parsePath

- 解析简单路径

- 返回了一个函数

- Usage

  ```js
  const obj = {
      a: {
          b: {
              c: 'path'
          }
      }
  }
  // 1.
  const fn = parsePath('a.b.c');
  fn(obj) => "path"
  // 2.
  parsePath('a.b.c')(obj);
  ```

## debug

process.env.NODE_ENV === 'production' 时以下四项都为noop，

### formatComponentName

- 格式化组件名，接收第一个参数为vue实例，第二个参数为是否包含文件
- 获取options选项的name和\__file字段；若name不存在但\__file存在，则取\__file的.vue前的作为name
- 返回classify后的name，若第二个参数为true，还返回file
- classify：将-或_分隔的组件名驼峰化，如page-menu_log => PageMenuLog
- 输入如<HelloWorld> at src/components/hello-world.vue

### generateComponentTrace

- 生成组件轨迹

- 二分法实现repeat

  ```js
  const repeat = (str, n) => {
      let res = ''
      while (n) {
        if (n % 2 === 1) res += str
        if (n > 1) str += str
        n >>= 1
      }
      return res
  }
  ```

### warn

- 如果用户定义了config.warnHandler则调用，如果没有，则console.error输出信息和轨迹

### tip

- 使用console.warn输出信息和轨迹

## error

### handleError

- 输出错误

## env

### hasProto

- 是否可以使用\__proto__

### 浏览器环境

inBrowser、UA、isIE、isIE9、isEdge、isAndroid、isIOS、isChrome

### nativeWatch

- *Firefox has a "watch" function on Object.prototype*

### supportsPassive

- [addEventListener是否支持第三个参数为对象](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/addEventListener)

### isServerRendering

- 是否是服务端渲染

### devtools

- inBrowser **&&** window.\__VUE_DEVTOOLS_GLOBAL_HOOK__

### isNative

- 是否是内置方法，如Function、Object等，用来与用户自定义的函数做区分

### hasSymbol

- 是否支持Symbol和Reflect，使用isNative判断

### nextTick

- 异步执行一个任务
- event loop task microtask setImmediate MessageChannel Promise setTimeout

### _Set

- 集合类，内置Set或自定义
- Set

## options

#### mergeDataOrFn

- parentVal, childVal必传,  vm可传

- 返回的方法的作用$a-mergeDataOrFn​$：有childVal时，返回childVal本身或其（函数）执行后的结果，或者把parentVal的属性合并到childVal后再将其返回；没有childVal时，就返回parentVal或其（函数）执行后的结果

- vm传入时

  - childVal如果不存在则返回parentVal

  - childVal存在，parentVal不存在的时候返回childVal

  - parentVal和childVal都存在的时候返回方法mergeDataFn

    ```
    function mergedDataFn () {
    	// 返回执行mergeData的结果，childVal或parentVal如果为函数，则执行后的结果作为参数传入mergeData, 否则传入childVal和parentVal自身
        return mergeData(typeof childVal === 'function' ? childVal.call(this) : childVal, typeof parentVal === 'function' ? parentVal.call(this) : parentVal)
    }
    ```

- vm没传时

  - parentVal或者childVal至少有一个存在时，返回方法mergedInstanceDataFn

    ```js
    function mergedInstanceDataFn () {
        // 将childVal（为函数时）执行后的结果或其本身作为instanceData，将parentVal执行后的结果或其本身作为defaultData
        const instanceData = typeof childVal === 'function' ? childVal.call(vm) : childVal;
        const defaultData = typeof parentVal === 'function' ? parentVal.call(this): parentVal;
        // 如果instanceData存在，则执行mergeData，并将结果返回，否则返回defaultData
        if (instanceData) {
            return mergeData(instanceData, defaultData);
        } else {
            return defaultData
        }
    }
    ```

- mergeData: (to: Object, from: Object) => 返回to
  - 过程解析：对于from的每一个key，如果to没有此key，则使用set(to, key, fromVal)给to增加一个职位fromVal的响应式属性key，如果to有此key，并且to[key]和from[key]都是纯对象的话，递归执行mergeData(to[key], from[key])
  - 作用：把from中有的，to中没有的，都一层层解析出来给to，并且要成为响应式

#### strats

- 父选项和子选项的合并策略

- defaultStrat: (parentVal, childVal) => childVal存在则返回childVal，否则返回parentVal

- el 、propsData：(parent, child, vm, key) => vm必传， 返回defaultStrat(parent, child);

- data: (parentVal, childVal, vm) => 如果传入vm，则将执行mergeDataOrFn(parentVal,childVal,vm)的结果返回，如果没传入并且childVal存在且为function，则将mergeDataOrFn(parentVal,childVal)的结果返回

- LIFECYCLE_HOOKS：每一个hook的对应值都是mergeHook方法

  - hook: 'beforeCreate', 'created', 'beforeMount', 'mounted', 'beforeUpdate', 'updated', 'beforeDestroy', 'destroyed', 'activated', 'deactivated', 'errorCaptured'
  - mergeHook: (parentVal, childVal) => childVal不存在的话返回parentVal,存在的话，如果parentVal也存在，则把parentVal和childVal拼接后的结果返回，parentVal不存在的话，则返回数组形式的childVal

- ASSET_TYPES：每一个asset+'s'的对应值都是mergeAssets方法

  - asset+'s'：'components', 'directives', 'filters'
  - mergeAssets: 以parentVal或null为原型创建一个新对象，如果childVal不存在，则返回此新对象，如果存在并且childVal是纯对象，则使用extend方法将childVal的属性扩展到此新对象，再返回此新对象，总之，作用就是把childVal合并到parentVal，把此操作的新结果返回

- watch: (parentVal, childVal, vm, key)

  - 如果childVal不存在，则返回以parentVal或null为原型的新对象，如果存在，必须为纯对象，往下
  - 如果parentVal不存在，则返回childVal
  - 创建了一个新对象ret
  - 通过extend方法使ret拥有parentVal中的所有属性
  - 对于每一个childVal中的属性，如果ret中有该属性并且值不是数组，则将其数组化，ret中的该属性则为该数组值拼接childVal的该key的value的结果，如果ret中不存在该属性，则ret增加该属性，值为数组化的childVal的该key的值
  - 总结：输出的是childVal或parentVal或ret,其中ret的每个属性值必须是数组，要么为parentVal和childVal拼接合并的，要么就是childVal

- props、methods、inject、computed: (parentVal, childVal, vm, key) 

  - childVal如果存在则必须是纯对象
  - parentVal如果不存在，则返回childVal
  - 把parentVal中所有属性extend到一个空对象，如果childVal存在，也将childVal的所有属性extend到这个对象里，并将其返回
  - 作用：返回一个拥有parentVal和childVal的全部属性的对象

- provide: mergeDataOrFn

  ```
  strats: 
  {
      el: defaultStrat,
      propsData: defaultStrat,
      data: mergeDataOrFn,
      beforeCreate: mergeHook,
      created: mergeHook,
      beforeMount: mergeHook,
      mounted: mergeHook,
      beforeUpdate: mergeHook,
      updated: mergeHook,
      beforeDestroy: mergeHook,
      destroyed: mergeHook,
      activated: mergeHook,
      deactivated: mergeHook,
      errorCaptured: mergeHook,
      components: mergeAssets,
      directives: mergeAssets,
      filters: mergeAssets,
      watch,
      props,
      methods,
      inject,
      computed,
      provide: mergeDataOrFn
  }
  ```

#### mergeOptions

- 合并两个options对象,并生成一个新的对象，是实例化和继承中使用的核心方法。

- 实现步骤解析

  ```js
  export function mergeOptions (parent: Object, child: Object, vm?: Component) {
      if (process.env.NODE_ENV !== 'production') {
          // 检查child.components下的所有key是否是内建或保留的tag
          checkComponents(child);
      }
      if (typeof child === 'function') {
          child = child.options;
      }
      // 确保将所有props选项语法规范化为Object-based格式
      normalizeProps(child, vm);
      // 将所有injections规范化为Object-based格式
      normalizeInject(child, vm); 
  	// 将原始函数指令规范化为对象格式。
      normalizeDirectives(child, vm);
      // 如果child存在extends项，则递归执行本函数，传入的child为该extends项
      const extendsFrom = child.extends;
      if (extendsFrom) {
          parent = mergeOptions(parent, extendsFrom, vm)
      }
      // 如果child存在mixins项，则对于mixins项中的每一项将其作为child参数，递归执行本函数
      if (child.mixins) {
          for (let i = 0, l = child.mixins.length; i < l; i++) {
              parent = mergeOptions(parent, child.mixins[i], vm)
          }
      }
      // 将parent中的每一个键和child中的但parent不存在的键放进options中，对应每个键的值是:strats如果存在该键，则执行strats中这个键对应的方法，否则执行defaultStrat，传入parent中该建的值，child中该键的值，vm组件，该键，执行后返回的方法就是options中该键的值，最后将options对象返回
      const options = {};
      let key;
      for (key in parent) {
          mergeField(key);
      }
      for (key in child) {
          if (!hasOwn(parent, key)) {
              mergeField(key);
          }
      }
      function mergeField (key) {
          const strat = strats[key] || defaultStrat
          options[key] = strat(parent[key], child[key], vm, key);
      }
      return options
  }
  ```

  - normalizeProps

    ```js
    例1
    const child1 = {
        props: ['title', 'message-type']
    }
    normalizeProps(child1)
    child1 = {
        props: {
            title: {
                type: null
            },
            messageType: {
                type: null
            }
        }
    }
    例2
    const child2 = {
        props: {
            title: {
                default: 'app'
            },
            'message-type': 'data'
        }
    }
    normalizeProps(child2)
    child2 = {
        props: {
            title: {
                default: 'app'
            },
            messageType: {
                type: 'app'
            }
        }
    }
    ```

    

  - normalizeInject

    ```
    例1
    const child1 = {
        inject: ['title', 'message-type']
    }
    normalizeInject(child1)
    child1 = {
        inject: {
            title: {
                from: 'title'
            },
            message-type: {
                from: 'message-type'
            }
        }
    }
    例2
    const child2 = {
        inject: {
            title: {
                default: 'app'
            },
            'message-type': 'data'
        }
    }
    normalizeInject(child2)
    child2 = {
        inject: {
            title: {
                from: 'title',
                default: 'app',
            },
            'message-type': {
                from: 'data'
            }
        }
    }
    ```

  - normalizeDirectives

    ```js
    const child = {
        directives: {
            focus: () => console.log('focus')
        }
    }
    normalizeDirectives(child);
    child = {
        directives: {
            focus: {
                bind: () => console.log('focus'),
                update: () => console.log('focus')
            }
        }
    }
    ```


#### resolveAsset

- (options: Object, type: String, id: String, warnMissing)
- 取options[type]为assets
- 返回assets[id]或aseets[驼峰化的id]或assets[首字母大些的id]

### props

#### validateProp

- getPropDefaultValue
- getPropDefaultValue
- assertProp
- isType