---
title: Vue源码学习（四）
date: 2018-11-02 19:35:04
tags: Vue
---

## Observer

### dep

#### Dep类

- 是一个可观察的对象，可以被很多指令订阅
- 拥有的属性
  - id
  - subs: Array
- 拥有的方法
  - addSub：往subs中添加一个watcher
  - removeSub：从subs中删除传入的watcher
  - depend：给target添加此依赖
  - notify：执行subs中所有watcher的update方法

- 静态属性：
  - target：全局唯一，只能有一个随时都能被评估的watcher

#### pushTarget方法

- 把当前的target存入一个数组，并且用传入的target替换当前的target

#### popTarget方法

- 从保存target的数组中的删除最后一个，并用删除的这个target替换当前的target

### watcher

#### Watcher类

- 解析表达式，收集依赖，并且当表达式当值变化时触发回调

- 用于$watcher、directives

- 参数：vm, expOrFn, cb, options

- 拥有的属性及其初始化

  - vm: Component => 传入的vm

  - expression: string  => 非production环境下为将expOrFn转为字符串，production环境下为''

  - cb: Function => 传入的cb

  - id: number => 为了批量，从0自增的uid

  - deep: boolean

  - user: boolean

  - lazy: boolean

  - sync: boolean

    => 以上四项，如若options有传，则为其对应项转化为boolean后的值，如没穿，则四项均默认为false

  - dirty: boolean => 为lazy的值

  - active: boolean => true

  - deps: Array<Dep> => []

  - newDeps: Array<Dep> => []

  - depIds: ISet => new Set()

  - newDepIds: ISet => new Set()

  - getter: Function => 如expOrFn为function则为expOrFn，否则，解析expOrFn的路径，getter为其嵌套最深层的key的value，如果得到的getter还是== false， 则认为getter是一个空函数

  - value: any => lazy为true时为undefined, 否则，为执行get方法的返回值

- 拥有的方法

  - get
    - 执行getter的结果保存到value变量中并将value返回
    - 重新收集依赖项
    - 如果deep为true，将value传入traverse执行，追踪所有的属性将其作为依赖项
    - _traverse：对于数组或可扩展的对象，对其所有的属性值递归执行，将所有值的__ob__.dep.id添加到一个Set中
  - addDep
    - 添加依赖dep
    - newDepIds不存在dep.id的情况下
    - 添加dep.id到newDepIds，添加dep到newDeps
    - 然后，如果depIds也不存在该id，使用dep.addSub将本watcher实例添加到dep的subs中
  - cleanupDeps
    - 清除依赖项
    - 找到deps中所有id不在newDepIds中的dep，从其subs中移除本watcher实例，将depIds置为原newDepIds，将deps置为原newDeps，并清空newDepIds和newDeps
  - update
    - 当一个依赖变化时被订阅者调用
    - 如果lazy为true，则把dirty置为true
    - 如果lazy为false,但是sync为true,则run
    - 否则调用scheduler的queueWatcher，将watcher排队异步执行
  - run
    - active为true时，执行get方法返回的值记为value
    - value和属性value不完全相等 或 value不是object 或 deep为true 的这三种情况下
    - 把属性value保存为oldValue，把value属性修改为value
    - 执行属性cb，传入value，oldValue
  - evaluate
    - 只被lazy watcher调用
    - 修改属性value为调用get方法的返回值，并将dirty属性修改为false
  - depend
    - 对此watcher实例收集的deps所有依赖项执行depend方法
  - teardown
    - active为true的情况下
    - 如果vm属性的_isBeingDestroyed属性为false，则从其vm属性的\_watchers属性中移除此watcher实例
    - 对于此watcher实例的所有依赖项，遍历，从每一个依赖中移除此watcher实例
    - 将active属性修改为false

### scheduler

#### queueWatcher方法

- 把watcher实例放入到watchers队列中，id重复的watcher会被跳过，除非在队列执行的时候被放进来

- 队列没有执行的时候，则把新的watcher放到队列的尾部，如果正在执行，则放在id比它小的后面

- waiting为false时，将waiting改为true，执行nextTick(flushSchedulerQueue)

- flushSchedulerQueue

  - 执行队列并执行watchers
  - 将队列所有项按id大小排序，原因是
    - 组件是从parent到child更新的，因为parent是在child之前创建的
    - 组件用户的watchers要在render watcher之前执行，因为user watchers 在render watcher之前创建
    - 如果一个组件在一个父组件的wathcer执行期间被销毁，它的watchers会被跳过

  - 遍历所有的watchers，将has对象的该watcher.id项改为null，执行每一个watcher的run方法，注意遍历时不要缓存队列的length，因为在执行wathcer时可能会有其他wathcers被加入进来，如果执行后has对象的该watcher.id项不为null，则需要检查并停止circular的更新
  - 缓存activatedChildren和queue
  - 重置所有的变量
  - 对于缓存的activatedChildren中所有的watcher，改变其_inactive为true，并调用lifecycle的激活子组件的方法
  - 对于缓存的queue中所有的watcher，如果其vm属性的_isMounted为true，则调用lifecycle的callHooks方法，传入vm和'update'

#### queueActivatedComponent方法

- 传入参数为vm
- 修改vm的_isactive为false
- 把vm放入activedChildren中

### array

拦截数组编译方法并书法notify

arrayMethods

- 以Array.prototype作为`__proto_`
- hasOwnProperty：push、pop、shift、unshift、splice、sort、reverse，调用这些方法时，则会通知ob的dep
- 对于push、unshift、splice三个方法，观察数组中的每一项

### index

#### observerState

默认情况下，当设置一个响应式属性时，新的value也被转换成了响应式，然而，当向下传递属性时，不想强制转换，因为value可能是一个冻结当数据结构下当嵌套的值，强制转换回破坏优化

#### Observer类

被附加到每个可观察的对象上，观察者将目标对象的属性转换成getter/setter,用来收集依赖/触发更新

- 属性

  value：

  - 定义了一个__ob__属性
  - value为数组，把arrayMethods的所有方法传递给value，作为`__proto_`或其自身属性；并执行observeArray方法observe观察数组每一项
  - value为对象，遍历目标object的所有属性，并执行walk将其转换成getter/setter

  dep：Dep实例

  vmCount

- 方法

  - walk：调用defineReactive将其转换成getter/setter
  - observeArray：遍历数组每一项，执行observe

#### observe方法

返回一个现有的observer，如果没有，则创建一个新的observer返回

#### defineReactive方法

- 在obj上定义一个响应式属性，定义了get：收集依赖 和 set：通知
- dependArray：收集依赖，因为数组元素不能通过getter访问

#### set方法

在target上设置属性key的值为val，如果属性不存在，则增加这个属性，并且触发notify

#### del方法

删除一个属性，并且触发notify