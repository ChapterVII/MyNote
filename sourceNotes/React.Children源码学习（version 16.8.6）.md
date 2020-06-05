# React.Children学习（version: 16.8.6）

## map

入参：第一个为要遍历的children，第二个为遍历执行的函数，第三个为执行遍历函数时的this。

返回：

- 一维数组。多维数组会被摊开成一维数组。
- 返回的每个节点如果isValidElement，则会给它加上个key，如果本来就有key，则会重新生成个新的key。
- 如果children=null，就直接返回。

以map方法入手梳理整体逻辑如下：

![Node跨平台](https://github.com/GrowLegend/MyNote/blob/master/static/images/react源码学习/ReactChildren.png)

### mapChildren

```javascript
function mapChildren(children, func, context) {
  if (children == null) {
    return children;
  }
  const result = [];
  // 遍历出来的元素会放到result中
  mapIntoWithKeyPrefixInternal(children, result, null, func, context);
  return result;
}
```

### mapIntoWithKeyPrefixInternal

```javascript
function mapIntoWithKeyPrefixInternal(children, array, prefix, func, context) {
  let escapedPrefix = '';
  if (prefix != null) {
    escapedPrefix = escapeUserProvidedKey(prefix) + '/';
  }
  const traverseContext = getPooledTraverseContext(
    array,
    escapedPrefix,
    func,
    context,
  );
  traverseAllChildren(children, mapSingleChildIntoContext, traverseContext);
  releaseTraverseContext(traverseContext);
}
```

### getPooledTraverseContext/releaseTraverseContext

```javascript
// 对象池的应用。
// 维护了一个大小为10的对象池，每次从池子里取一个对象去赋值，用完了就将对象上的属性置空后放回池子。因为频繁的创建对象会导致性能问题。
const POOL_SIZE = 10;
const traverseContextPool = [];
function getPooledTraverseContext(
  mapResult,
  keyPrefix,
  mapFunction,
  mapContext,
) {
  if (traverseContextPool.length) {
    const traverseContext = traverseContextPool.pop();
    traverseContext.result = mapResult;
    traverseContext.keyPrefix = keyPrefix;
    traverseContext.func = mapFunction;
    traverseContext.context = mapContext;
    traverseContext.count = 0;
    return traverseContext;
  } else {
    return {
      result: mapResult,
      keyPrefix: keyPrefix,
      func: mapFunction,
      context: mapContext,
      count: 0,
    };
  }
}

function releaseTraverseContext(traverseContext) {
  traverseContext.result = null;
  traverseContext.keyPrefix = null;
  traverseContext.func = null;
  traverseContext.context = null;
  traverseContext.count = 0;
  if (traverseContextPool.length < POOL_SIZE) {
    traverseContextPool.push(traverseContext);
  }
}
```

### traverseAllChildren

```javascript
function traverseAllChildren(children, callback, traverseContext) {
  if (children == null) {
    return 0;
  }

  return traverseAllChildrenImpl(children, '', callback, traverseContext);
}

```

### traverseAllChildrenImpl

核心递归函数，目的为摊平数组，对于可循环的children，都会重复调用traverseAllChildrenImpl，直到为可渲染节点，调用callback

逻辑：children类型

1. 可渲染节点：调用mapSingleChildIntoContext把children放到result数组中。
2. 数组：对数组中的每个元素调用traverseAllChildrenImpl，传入的key是最新拼接好的。
3. 对象：通过children[Symbol.iterator]获取到对象的迭代器iterator，将迭代结果放到traverseAllChildrenImpl处理。

```javascript
function traverseAllChildrenImpl(
  children,
  nameSoFar, // 父级key，会一层层拼接传递
  callback, 
  traverseContext, // 对象池中拿出来的一个对象
) {
  const type = typeof children;

  if (type === 'undefined' || type === 'boolean') {
    // All of the above are perceived as null.
    children = null;
  }

  let invokeCallback = false;

  if (children === null) {
    invokeCallback = true;
  } else {
    switch (type) {
      case 'string':
      case 'number':
        invokeCallback = true;
        break;
      case 'object':
        switch (children.$$typeof) {
          case REACT_ELEMENT_TYPE:
          case REACT_PORTAL_TYPE:
            invokeCallback = true;
        }
    }
  }

  if (invokeCallback) {
    callback(
      traverseContext,
      children,
      nameSoFar === '' ? SEPARATOR + getComponentKey(children, 0) : nameSoFar,
    );
    return 1;
  }

  let child;
  let nextName;
  let subtreeCount = 0;
  const nextNamePrefix =
    nameSoFar === '' ? SEPARATOR : nameSoFar + SUBSEPARATOR;

  if (Array.isArray(children)) {
    for (let i = 0; i < children.length; i++) {
      child = children[i];
      // 不手动设置key时 第一层为.0 第二个为.1
      nextName = nextNamePrefix + getComponentKey(child, i);
      subtreeCount += traverseAllChildrenImpl(
        child,
        nextName,
        callback,
        traverseContext,
      );
    }
  } else {
    const iteratorFn = getIteratorFn(children);
    if (typeof iteratorFn === 'function') {
      if (__DEV__) {
        // Warn about using Maps as children
        if (iteratorFn === children.entries) {
          warning(
            didWarnAboutMaps,
            'Using Maps as children is unsupported and will likely yield ' +
              'unexpected results. Convert it to a sequence/iterable of keyed ' +
              'ReactElements instead.',
          );
          didWarnAboutMaps = true;
        }
      }

      const iterator = iteratorFn.call(children);
      let step;
      let ii = 0;
      while (!(step = iterator.next()).done) {
        child = step.value;
        nextName = nextNamePrefix + getComponentKey(child, ii++);
        subtreeCount += traverseAllChildrenImpl(
          child,
          nextName,
          callback,
          traverseContext,
        );
      }
    } else if (type === 'object') {
      let addendum = '';
      if (__DEV__) {
        addendum =
          ' If you meant to render a collection of children, use an array ' +
          'instead.' +
          ReactDebugCurrentFrame.getStackAddendum();
      }
      const childrenString = '' + children;
      invariant(
        false,
        'Objects are not valid as a React child (found: %s).%s',
        childrenString === '[object Object]'
          ? 'object with keys {' + Object.keys(children).join(', ') + '}'
          : childrenString,
        addendum,
      );
    }
  }

  return subtreeCount;
}
```

### mapSingleChildIntoContext

```javascript
function mapSingleChildIntoContext(
 bookKeeping, // 从对象池里取出的对象
 child, 
 childKey
) {
  const {result, keyPrefix, func, context} = bookKeeping;

  // func为在getPooledTraverseContext获取traverseContext时被传入
  let mappedChild = func.call(context, child, bookKeeping.count++);
  if (Array.isArray(mappedChild)) {
    mapIntoWithKeyPrefixInternal(mappedChild, result, childKey, c => c);
  } else if (mappedChild != null) {
    if (isValidElement(mappedChild)) {
      // clone mappedChild并替换掉key
      mappedChild = cloneAndReplaceKey(
        mappedChild,
        // Keep both the (mapped) and old keys if they differ, just as
        // traverseAllChildren used to do for objects as children
        keyPrefix +
          (mappedChild.key && (!child || child.key !== mappedChild.key)
            ? escapeUserProvidedKey(mappedChild.key) + '/'
            : '') +
          childKey,
      );
    }
    result.push(mappedChild);
  }
}
```

## forEach

### forEachChildren

```javascript
function forEachChildren(children, forEachFunc, forEachContext) {
  if (children == null) {
    return children;
  }
  const traverseContext = getPooledTraverseContext(
    null,
    null,
    forEachFunc,
    forEachContext,
  );
  // 同map相比，对于可渲染节点调用的callback为forEachSingleChild函数
  traverseAllChildren(children, forEachSingleChild, traverseContext);
  releaseTraverseContext(traverseContext);
}

function forEachSingleChild(bookKeeping, child, name) {
  const {func, context} = bookKeeping;
  func.call(context, child, bookKeeping.count++);
}
```

## count

计算children的个数，为摊平后数组元素的个数

```javascript
function countChildren(children) {
  // subtreeCount, 表示子节点的个数
  return traverseAllChildren(children, () => null, null);
}
```

## toArray

```javascript
function toArray(children) {
  const result = [];
  mapIntoWithKeyPrefixInternal(children, result, null, child => child);
  return result;
}
```

## only

如果参数是一个 ReactElement，则直接返回它，否则报错，用在测试中，正式代码没什么用。

```javascript
function onlyChild(children) {
  invariant(
    isValidElement(children),
    'React.Children.only expected to receive a single React element child.',
  );
  return children;
}
```

