# Javascript设计模式与开发实践

## 基础知识

### 面向对象的JavaScript

鸭子类型：只关心对象的行为，而不关注对象本身

面向接口编程，而不是面向实现编程

多态：”做什么“和”谁去做以及怎样去做“分离开来，也就是将”不变的事物“与”可能改变的事物“分离开来

封装：封装数据、封装实现、封装类型、封装变化

### this、call、apply

this的指向大致分为：

1. 作为对象的方法调用：指向该对象；
2. 作为普通函数调用：指向全局对象window，严格模式指向undefined；
3. 构造器调用：指向返回的对象；
4. call、apply调用

this丢失

```javascript
var getId = document.getElementById;
getId('div1');
```

这段代码抛出异常，是因为许多引擎的document.getElementById方法的内部实现中需要用到this，这个this本来被期望指向document，但当用getId来引用之后，在调用，就成了普通函数调用，函数内部的this指向了window

call和apply的区别：当调用一个函数时，js的解释器不会计较形参和实参在数量、类型、顺序上的区别，js的参数在内部就是用一个数组来表示的。从这个意义上说，apply比call的使用率更高；call是包装在apply上面的一颗语法糖

### 闭包和高阶函数

闭包的作用：

1. 封装变量

```javascript
var mult = function() {
  var a = 1;
  for (var i = 0, l = arguments.length; i < l; i++) {
    a = a * arguments[i];
  }
  return a;
}

// 对于相同的参数来说，每次都进行计算是一种浪费，加入缓存机制提高性能，如下
var mult = (function() {
  var cache = [];
  var calculate = function() {
    var a = 1;
    for (var i = 0, l = arguments.length; i < l; i++) {
      a = a * arguments[i];
    }
    return a;
  }
  return function() {
    var args = Array.prototype.join.call(arguments, ',');
    if (args in cache) {
      return cache[args];
    }
    return cache[args] = calculate.apply(null, arguments);
  }
})();
```

2. 延续局部变量的寿命

```javascript
// img对象经常用于进行数据上报
var report = function(src) {
  var img = new Image();
  img.src = src;
}

report('http://xxx.com/getUserInfo');

// 低版本浏览器的实现存在bug，在这些浏览器下使用report函数进行数据上报会丢失30%左右的数据，也就是说，report函数并不是每一次都成功发起了HTTP请求。丢失数据的原因是img是report函数中的局部变量，当report函数的调用结束后，img局部变量随即被销毁，而此时获取还没来得及发出HTTP请求。使用闭包解决
var report = (function() {
  var imgs = [];
  return function(src) {
    var img = new Image();
    imgs.push(img);
    img.src = src;
  }
})();
```

高阶函数：

1. 函数柯里化currying，又称部分求值。

   一个currying的函数接收了一些参数之后，并不会立即求值，而是继续返回另外一个函数，刚传入的参数在函数形成的闭包中被保存，待到函数被真正需要求值的时候，之前传入的所有参数都会被一次性用于求值。

   ```javascript
   var currying = function(fn) {
     var args = [];
     return function() {
       if (arguments.length === 0) {
         return fn.apply(this, args);
       } else {
         [].push.apply(args, arguments);
         return arguments.callee;
       }
     }
   };
   
   var cost = (function() {
     var money = 0;
     return function() {
       for (var i = 0, l = arguments.length; i < l; i++) {
         money += arguments[i];
       }
       return money;
     }
   })();
   
   var cost = currying(cost);  // 转化成currying函数
   
   cost(100);  // 未真正求值
   cost(200);  // 未真正求值
   cost(300);  // 未真正求值
   
   alert(cost());  // 求值并输出：600
   ```

2. uncurrying：用来解决把泛化this的过程提取出来

   ```javascript
   Function.prototype.uncurrying = function() {
     var self = this;  // self此时是Array.prototype.push
     return function() {
       // arguments对象的第一个元素被截去，obj是{length: 1, 0: 1}，arguments是[2]
       var obj = Array.prototype.shift.call(arguments);
       // 相当于Array.prototype.push.apply(obj, 2)
       return self.apply(obj, arguments);
     }
   }
   
   var push = Array.prototype.push.currying();
   var obj = {
     length: 1,
     0: 1,
   }
   
   push(obj, 2);
   ```

3. 函数节流

   少数情况下函数的触发不是由用户直接控制的，被非常频繁的调用

   场景：

   - window.onresize事件
   - mousemove事件
   - 上传进度

   代码实现

   ```javascript
   var throttle = function(fn, interval) {
     var _self = fn,
         timer,
         firstTime = true;
     return function() {
       var args = arguments,
           _me = this;
       if (firstTime) {
         _self.apply(_me, args);
         return firstTime = false;
       }
       if (timer) {
         return false;
       }
       timer = setTimeout(function() {
         clearTimeout(timer);
         timer = null;
         _self.apply(_me, args);
       }, interval || 500);
     }
   }
   
   window.onresize = throttle(function() {
     console.log(1);
   }, 500);
   ```

4. 分时函数：如短时间内往页面中大量添加DOM节点会造成浏览器卡顿甚至假死的问题，可以让创建节点的工作分批进行，如把1s创建1000个节点改为每隔200ms创建8个节点

   ```javascript
   var timeChunk = function(ary, fn, count) {
     var obj, t;
     var len = ary.length;
     var start = function() {
       for (var i = 0; i < Math.min(count || 1, arr.length); i++) {
         var obj = ary.shift();
         fn(obj);
       }
     }
   
     return function() {
       t = setInterval(function() {
         if (arr.length === 0) {
           return clearInterval(t);
         }
         start();
       }, 200);
     }
   }
   
   var ary = [];
   for (var i = 0; i <  1000; i++) {
     ary.push(i);
   }
   
   var renderFriendList = timeChunk(ary, function(n) {
     var div = document.createElement('div');
     div.innerHTML = n;
     document.body.appendChild(div);
   }, 8);
   
   renderFriendList();
   ```

5. 惰性加载函数

   如浏览器嗅探

   ```javascript
   var addEvent = function(elem, type, handler) {
     if (window.addEventListener) {
       addEvent = function(elem, type, handler) {
         elem.addEventListener(type, handler);
       }
     } else if (window.attachEvent) {
       addEvent = function(elem, type, handler) {
         elem.attachEvent('on' + type, handler);
       }
     }
     addEvent(elem, type, handler);
   }
   ```

## 设计模式

### 单例模式

核心：只有一个实例，并提供全局访问

惰性单例：在合适的时候才创建对象，并之创建唯一的一个

```javascript
var getSingle = function(fn) {
  var result;
  return function() {
    return result || (result = fn.apply(this, arguments));
  }
}

var createLoginLayer = function() {
  div = document.createElement('div');
  div.innerHTML = '登录浮窗';
  div.style.display = 'none';
  document.body.appendChild(div);
  return div;
};

var createSingleLoginLayer = getSingle(createLoginLayer);

document.getElementById('loginBtn').onclick = function() {
  var loginLayer = createSingleLoginLayer();
  loginLayer.style.display = 'block';
}

var createSingleIframe = getSingle(function() {
  var iframe = document.createElement('iframe');
  document.body.appendChild(iframe);
  return iframe;
});

document.getElementById('loginBtn').onclick = function() {
  var loginLayer = createSingleIframe();
  loginLayer.src = 'http://baidu.com';
}
```

### 策略模式

### 代理模式

### 迭代器模式

### 发布-订阅模式

### 命令模式

### 组合模式

### 模板方法模式

### 享元模式

### 职责链模式

### 中介者模式

### 装饰者模式

### 状态模式

### 适配器模式

## 设计原则和编程技巧

### 单一职责原则

### 最少知识原则

### 开放-封闭原则

### 接口和面向接口编程

### 代码重构