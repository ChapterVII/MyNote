# Javascript设计模式与开发实践

[TOC]



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

- 定义：定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。
- 一个基于策略模式的程序至少由两部分组成。第一部分是一组策略类，封装了具体的算法，并负责具体的计算过程。第二个部分是环境类Context，接受客户的请求，随后把请求委托给某一个策略类。
- 多态性的体现
- 在实际开发中，通常会把算法的含义扩散开来，可以用来封装一系列的业务规则。只要这些业务规则指向的目标一致，并可被替换使用：如表单校验。

```javascript
const strategies = {
  isNonEmpty: function(value, errorMsg) {
    if (value === '') {
      return errorMsg;
    }
  },
  minLength: function(value, length, errorMsg) {
    if (value.length < length) {
      return errorMsg;
    }
  },
  isMobile: function(value, errorMsg) {
    if (!/(^1[3|5|8][0-9]{9}$)/.test(value)) {
      return errorMsg;
    }
  }
}

const Validator = function() {
  this.cache = [];
}

Validator.prototype.add = function(dom, rule, errorMsg) {
  const ary = rule.split(':');
  this.cache.push(function() {
    const strategy = ary.shift();
    ary.unshift(dom.value);
    ary.push(errorMsg);
    return strategies[strategy].apply(dom, ary);
  })
}

Validator.prototype.start = function() {
  for (var i = 0, validataFunc; validataFunc = this.cache[i++];) {
    const msg = validataFunc();
    if (msg) return msg;
  }
}

const registerForm = document.getElementById('registerForm');

const validataFunc = function() {
  const validator = new Validator();
  validator.add(registerForm.userName, 'isNonEmpty', '用户名不能为空');
  validator.add(registerForm.password, 'minLength:6', '密码长度不能少于6位');
  validator.add(registerForm.phoneNumber, 'isMobile', '手机号码格式不正确');

  const errorMsg = validator.start();
  return errorMsg;
}

registerForm.onsubmit = function() {
  const errorMsg = validataFunc();
  if (errorMsg) {
    alert(errorMsg);
    return false;
  }
}
```

### 代理模式

- 当客户不方便直接访问一个对象或者不满足需要的时候，提供一个替身对象来控制对这个对象的访问，客户实际上访问的是替身对象。替身对象对请求做出一些处理之后，再把请求转交给本体对象。
- 保护代理：用于控制不同权限的对象对目标对象的访问，js不容易实现
- 虚拟代理：把一些开销很大的对象，延迟到真正需要它的时候才去创建。最常用，如图片预加载。

```javascript
const myImage = (function() {
  const imgNode = document.createElement('img');
  document.body.appendChild(imgNode);
  return {
    setSrc: function(src) {
      imgNode.src = src;
    }
  }
})();

const proxyImage = (function() {
  const img = new Image;
  img.onload = function() {
    myImage.setSrc(this.src);
  }
  return {
    setSrc: function(src) {
      myImage.setSrc('./loading.gif');
      img.src = src;
    }
  }
})();

proxyImage.setSrc('./photo.jpg');
```

- 代理和本体接口的一致性的好处：
  1. 用户可以放心地请求代理，只关心是否能得到想要的结果；
  2. 在任何使用本体的地方都可以替换成使用代理。

- 虚拟代理合并HTTP请求：

  web开发中最大的开销就是网络请求。假设一个文件同步的功能，选中一个checkbox时对应的文件就会同步到另一台备用服务器上。可以通过一个代理函数来收集一段时间之内的请求，最后一次性发送给服务器

```javascript
const synchronousFile = function(id) {
  console.log('文件id为：' + id);
}

const proxySynchronousFile = (function() {
  const cache = [], timer;
  return function(id) {
    cache.push(id);
    if (timer) {
      return;
    }
    timer = setTimeout(function() {
      synchronousFile(cache.join(','));
      clearTimeout(timer);
      timer = null;
      cache.length = 0;
    })
  }
})();

const checkbox = document.getElementsByTagName('input');
for (var i = 0, c; c = checkbox[i++];) {
  c.onclick = function() {
    if (this.checked === true) {
      proxySynchronousFile(this.id);
    }
  }
}
```

- 虚拟代理惰性加载

  等用户按下F2唤出控制台的时候才开始加载真正的miniConsole.js代码

```javascript
const miniConsole = (function() {
  const cache = [];
  const handler = function(ev) {
    if (ev.keyCode === 113) {
      const script = document.createElement('script');
      script.onload = function() {
        for (var i = 0, fn; fn = cache[i++];) {
          fn();
        }
      }
      script.src = 'miniConsole.js';
      document.getElementsByTagName('head')[0].appendChild(script);
      document.body.removeEventListener('keydown', handler); // 只加载一次
    }
  }

  document.body.addEventListener('keydown', handler, false);
  return {
    log: function() {
      const arg = arguments;
      cache.push(function() {
        return miniConsole.log.apply(miniConsole, args);
      });
    }
  }
})();

miniConsole.log(11);

// miniConsole.js代码
miniConsole = {
  log: function() {
    console.log(Array.prototype.join.call(arguments));
  }
}
```

- 缓存代理

  为一些开销大的运算结果提供暂时的存储，下次运算时，如果参数和之前一直，则可以返回存储的结果

  例子：计算乘积、Ajax异步请求数据

- 其他代理模式

  防火墙代理、远程代理、智能引用代理、写时复制代理

### 迭代器模式

- 提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。

- 可以把迭代的过程从业务逻辑中分离出来，即使不关心对象的内部构造，也可以按顺序访问其中的每个元素。

- 内部迭代器

  内部已经定义好了迭代规则，完全接手整个迭代过程，外部只需要一次初始调用。

- 外部迭代器

  - 必须显式请求迭代下一个元素。可以手工控制迭代的过程或顺序

  ```javascript
  const Iterator = function(obj) {
    let current = 0;
    const next = function() {
      current += 1;
    }
    const isDone = function() {
      return current >= obj.length;
    }
    const getCurrentItem = function() {
      return obj[current];
    }
    return {
      next,
      isDone,
      getCurrentItem,
      length,
    }
  }
  
  const compare = function(iterator1, iterator2) {
    if (iterator1.length !== iterator2.length) {
      alert('iterator1和iterator2不相等');
    }
    while(!iterator1.isDone() && !iterator2.isDone()) {
      if (iterator1.getCurrentItem() !== iterator2.getCurrentItem()) {
        throw new Error('iterator1和iterator2不相等');
      }
      iterator1.next();
      iterator2.next();
    }
    alert('iterator1和iterator2不相等');
  }
  
  const iterator1 = Iterator([1, 2, 3]);
  const iterator2 = Iterator([1, 2, 3]);
  compare(iterator1, iterator2);
  ```

- 不仅可以迭代数组，还可以迭代类数组的对象。只要被迭代的聚合对象拥有length属性且可用下标访问，就可被迭代。

- 没有规定以顺序、倒序、还是中序来遍历对象

- 可以提供一种跳出循环的方法

  ```javascript
  const each = function(obj, callback) {
    const length = obj.length;
    const isArray = isArrayLike(obj);
    let value, i = 0;
    if (isArray) {
      // 倒序访问
      for (i = length - 1; i >= 0; i--) {
        value = callback.call(obj[i], i. obj[i]);
        // 中止迭代器
        if (value === false) {
          break;
        }
      }
    } else {
      for ( i in obj) {
        value = callback.call(obj[i], i, obj[i]);
        if (value === false) {
          break;
        }
      }
    }
    return obj;
  }
  ```

- 应用举例：根据不同浏览器获取相应的上传组件对象

  ```javascript
  const getActiveUploadObj = function() {
    try {
      return new ActiveXObject('TXFTNActiveX.FTNUpload'); // IE上传控件
    } catch(e) {
      return false;
    }
  }
  
  const getFlashUploadObj = function() {
    if (supportFlash()) {
      const str = '<object type="application/x-shockwave-flash"></object>';
      return $(str).appendTo($('body'));
    }
    return false;
  }
  
  const getFormUploadObj = function() {
    const str = '<input name="name" type="file"  />';
    return $(str).appendTo($('body'));
  }
  
  const iteratorUploadObj = function() {
    for (let i = 0, fn; fn = arguments[i++];) {
      const uploadObj = fn();
      if (uploadObj !== false) {
        return uploadObj;
      }
    }
  }
  
  const uploadObj = iteratorUploadObj(getActiveUploadObj, getFlashUploadObj, getFormUploadObj);
  ```

### 发布-订阅模式（观察者模式）

1. 定义：对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知。一般用事件模型来替代传统的发布订阅模式。

2. 作用：可以广泛应用于异步编程中，替代传递回调函数的方案；可以取代对象之间硬编码的通知机制，一个对象不用再显式地调用另一个对象的某个接口。当有新的订阅者出现时，发布者的代码不需修改，同样发布者需要改变时，也不会影响到之前的订阅者。

3. 实现：DOM事件、自定义事件。

   1. 首先指定谁充当发布者。
   2. 给发布者添加一个缓存列表，用于存放回调函数以便通知订阅者。
   3. 发布消息时，发布者遍历这个缓存列表，依次触发里面存放的订阅者回调函数（可以往回调函数里填入一些参数，订阅者可以接收这些参数）。

4. 取消订阅的事件：反向遍历订阅的回调函数列表，删除订阅者的回调函数。

5. 全局的发布订阅模式：可以用一个全局的Event对象来实现，订阅者不需要了解消息来自哪个发布者，发布者也不知道消息会推送给哪些订阅者，Event作为一个类似”中介者“的角色，把订阅者和发布者联系起来。

   注意：模块之间如果用了太多的全局发布-订阅模式来通信，那么模块与模块之间的联系就被隐藏到了背后，最终会搞不清楚消息来自哪个模块 ，或者消息会流向哪些模块，给维护带来一些麻烦，也许某个模块的作用就是暴露一些接口给其他模块调用。

6. 某些情况下需要拥有先发布后订阅的能力。建立一个存放离线事件的堆栈，发布时，如果此时还没有订阅者来订阅，暂时把发布事件的动作包裹在一个函数里，这些包装函数被存入堆栈中，等到有对象来订阅此事件时，将遍历堆栈并依次执行包装函数，重新发布里面的事件。离线事件的生命周期只有一次。

   ```javascript
   const Event = (function() {
     const clientList = [];
     let listen, 
         trigger, 
         remove;
     listen = function(key, fn) {
       if (!clientList[key]) {
         clientList[key] = [];
       }
       clientList[key].push(fn);
     };
   
     trigger = function() {
       const key = Array.prototype.shift.call(arguments);
       const fns = clientList[key];
       if (!fns || fns.length === 0) {
         return false;
       }
       for(let i = 0, fn; fn = fns[i++];) {
         fn.apply(this, arguments);
       }
     };
   
     remove = function(key, fn) {
       const fns = clientList[key];
       if (!fns) {
         return false;
       }
       if (!fn) {
         fns && (fns.length = 0);
       } else {
         for (let l = fns.length - 1; l >= 0; l--) {
           const _fn = fns[l];
           if (_fn === fn) {
             fns.splice(l, 1);
           }
         }
       }
     };
   
     return {
       listen,
       trigger,
       remove,
     }
   })();
   ```

### 命令模式

- 应用场景：需要向某些对象发送请求，但并不知道请求的接收者是谁，也不知道请求的操作是什么。此时希望用一种松耦合的方式来设计程序，使得请求发送者和接收者能消除彼此之间的耦合关系。

- 相对于过程化的请求调用，command对象拥有更长的生命周期。对象的生命周期是和初始请求无关的，因为这个请求已经被封装在了command对象的方法中，成为了这个对象的行为，可以在程序运行的任意时刻去调用这个方法。

- 命令模式的由来，其实是回调函数的一个面向对象的替代品。

- 可以使用闭包来完成

- 撤销：给命令对象增加名为unexecude或undo的方法，在该方法里执行execute的反向操作。如果需要撤销一系列命令。可以把所有执行过的命令都储存在一个历史列表中，然后倒叙循环来依次执行这些命令的undo操作。

- 重做：有些情况下无法顺利地用undo操作让对象回到execute之前的状态。如Canvas画图，使用命令模式在一些点之间画了N条曲线连接，但很难定义一个擦除某条曲线的undo操作。最好的办法是先清除画布，然后把执行过的命令全部重新执行以便，同样可以利用一个历史堆栈办到。记录命令日志，然后重复执行，这是逆转不可逆命令的一个好办法。

  ```javascript
  const MoveCommand = function(receiver, pos) {
    this.receiver = receiver;
    this.pos = pos;
    this.oldPos = null;
  }
  
  MoveCommand.prototype.execute = function() {
    this.receiver.start('left', this.pos, 1000, 'strongEaseOut');
    this.oldPos = this.receiver.dom.getBoundingClientRect()[this.receiver.propertyName];
  }
  
  MoveCommand.prototype.undo = function() {
    this.receiver.start('left', this.oldPos, 1000, 'strongEaseOut');
  }
  
  let moveCommand;
  const commandStack = [];
  
  moveBtn.onclick = function() {
    let animate = new Animate(ball);
    moveCommand = new MoveCommand(animate, pos.value);
    commandStack.push(moveCommand);
    moveCommand.execute();
  }
  
  cancelBtn.onclick = function() {
    moveCommand.undo();
  }
  
  replyBtn.onclick = function() {
    let command;
    while(command = commandStack.shift()) {
      command();
    }
  }
  ```

- 命令队列：把命令对象压进一个队列堆栈，当当前command对象的职责完成之后，回主动通知队列，此时取出正在队列中等待的第一个命令对象执行。

- 宏命令：是一组命令的集合，通过执行宏命令的方式，可以一次执行一批命令。

- 智能命令&傻瓜命令：一般来说，命令模式都会在command对象中保存一个接收者来负责真正执行客户的请求，这种情况下命令对象是“傻瓜式”的，只负责把客户的请求转交给接收者来执行，好处是请求发起者和请求接收者之间尽可能地得到了解耦。但是，“聪明地”命对象可以直接实现请求。这样就不再需要接收者的存在，也叫做智能命令，退化到和策略模式非常相近。

  ```javascript
  const closeDoorCommand = {
    execute: function() {
      console.log('关门');
    }
  };
  
  const openPcCommand = {
    execute: function() {
      console.log('开电脑');
    }
  };
  
  const openQQCommand = {
    execute: function() {
      console.log('登录QQ');
    }
  }
  
  const MacroCommand = function() {
    return{
      commandList: [],
      add: function(command) {
        this.commandList.push(command);
      },
      execute: function() {
        for (let i = 0, command; command = this.commandList[i++];) {
          command.execute();
        }
      }
    }
  }
  
  const macroCommand = MacroCommand();
  macroCommand.add(closeDoorCommand);
  macroCommand.add(openPcCommand);
  macroCommand.add(openQQCommand);
  
  macroCommand.execute();
  ```

### 组合模式

- 命令模式中的宏命令中的macroCommand就是组合对象。closeDoorCommand、openPcCommand、openQQCommand都是叶对象。macroCommand只负责传递请求给叶对象，目的不在于控制对叶对象的访问。

- 用途：

  - 将对象组合成树形结构，以表示“部分-整体“的层次结构。
  - 通过对象的多态性表现，使得用户对单个对象和组合对象的使用具有一致性。

- 请求在书中传递的过程：如果子节点是叶对象，叶对象自身会处理这个请求。如果子节点是组合对象，请求会继续往下传递。叶对象下面不会再有其他子节点，一个叶对象就是树的这条职业的尽头，组合对象下面可能还会有子节点。请求从上到下沿着树进行传递，知道树的尽头。

- javascript中实现组合模式的难点在于要保证组合对象和叶对象拥有同样的方法，通常需要用鸭子类型的思想进行接口检查。

- 组合模式的透明性使得发起请求的客户不用去顾忌树中组合对象和叶对象的区别，也许会发生一些误操作，如往叶对象中添加子节点。解决方案通常是给叶对象添加add方法，调用时抛出异常。

- 注意：

  - 不是父子关系。是一种HAS-A（聚合）的关系，而不是IS-A。组合对象把请求委托给它所包含的所有叶对象，能够合作的关键是拥有相同的接口。
  - 除了要求组合对象和叶对象拥有相同的接口外，还有一个必要条件，对一组叶对象的操作必须具有一致性。
  - 复合情况下必须给父节点和子节点建立双向映射关系。一个简单的方法是增加集合来保存对方的引用，但相当复杂，而且对象之间产生了过多的耦合性，修改或删除都更困难。此时可以引入中介者模式来管理这些对象。
  - 用职责链模式提高组合模式性能。当书的结构复杂，节点多，性能不好时，可以避免遍历整棵树，借助职责链模式。让请求顺着链条从父对象往子对象传递，或者反向传递，知道遇到可以处理该请求的对象为止。

- 引用父对象：有时需要在子节点上保持对父节点的引用，比如使用职责链时可能需要让请求从子节点往父节点上冒泡传递。

  ```javascript
  const Folder = function(name) {
    this.name = name;
    this.parent = null;
    this.files = [];
  }
  
  Folder.prototype.add = function(file) {
    file.parent = this;
    this.files.push(file);
  }
  
  Folder.prototype.scan = function() {
    console.log('开始扫描文件夹： ' + this.name);
    for (let i = 0, file, files = this.files; file = files[i++]; ) {
      file.scan();
    }
  }
  
  Folder.prototype.remove = function() {
    if (!this.parent) {
      return;
    }
    for (let files = this.parent.files, l = files.length - 1; l >= 0; l--) {
      const file = files[l];
      if (file === this) {
        files.splice(l, 1);
      }
    }
  }
  
  const File = function(name) {
    this.name = name;
    this.parent = null;
  }
  
  File.prototype.add = function() {
    throw new Error('不能添加在文件下面');
  }
  
  File.prototype.scan = function() {
    console.log('开始扫描文件： ' + this.name);
  }
  
  File.prototype.remove = function() {
    if (!this.parent) {
      return;
    }
    for (let files = this.parent.files, l = files.length - 1; l >= 0; l--) {
      const file = files[l];
      if (file === this) {
        files.splice(l, 1);
      }
    }
  }
  
  const folder = new Folder('学习资料');
  const folder1 = new Folder('Javascript');
  const file1 = new Folder('深入浅出Node.js');
  folder1.add(new File('Javascript设计模式与开发实践'));
  folder.add(folder1);
  folder.add(file1);
  
  folder1.remove();
  folder.scan();
  ```

  

### 模板方法模式

- 定义和组成：只需使用继承就可实现。由抽象父类和具体的实现子类组成。抽象父类中封装了子类的算法框架，包括实现一些公共方法以及封装子类中所有方法的执行顺序。子类通过继承这个抽象类，也继承了整个算法结构，并可以选择重写父类的方法。在模板方法模式中，子类视线中的相同部分被上移到父类中，而将不同的部分留待子类来实现。也体现了泛化的思想。

- JavaScript没有抽象类

  - 缺点：使用原型继承来模拟传统的类式继承时，没有编译器任何形式的检查，没有办法保证子类会重写父类中的抽象方法。
  - 解决方案：
    - 用鸭子类型来模拟接口检查。带来不必要的复杂性，要求添加一些跟业务无关的代码。
    - 让父类的抽象方法直接抛出异常。实现简单，额外代价很少，但得到错误信息的时间点太靠后。

- 使用场景：常被架构师用于搭建项目的框架、构建UI组件。

- 钩子方法hook：放置钩子是隔离变化的一种常见手段。在父类中容易变化的地方放置钩子，钩子可以有一个默认的实现，究竟要不要”挂钩“，这由子类自行决定。钩子方法的返回结果巨顶了模板方法后面的执行步骤，也就是程序接下来的走向。

- 好莱坞原则：允许底层组件将自己挂钩到高层组件中，而高层组件会决定什么时候、以何种方式去使用这些底层组件。

  - 当使用模板方法模式时，意味着子类放弃了对自己的控制权，而是改为父类通知子类，哪些方法应该在什么时候被调用。作为子类，只负责提供一些设计上的细节。
  - 其他场景：发布-订阅模式、回调函数。

  

  ```javascript
  const Beverage = function() {};
  
  Beverage.prototype.boilWater = function() {
    console.log('把水煮沸');
  }
  
  Beverage.prototype.brew = function() {
    throw new Error('子类必须重写brew方法');
  };
  
  Beverage.prototype.pourInCup = function() {
    throw new Error('子类必须重写pourInCup方法');
  };
  
  Beverage.prototype.addCondiments = function() {
    throw new Error('子类必须重写addCondiments方法');
  };
  
  Beverage.prototype.customerWantsCondiments = function() {
    return true;  // 默认需要调料
  }
  
  // 模板方法：该方法中封装了子类的算法框架，作为一个算法的模板，知道子类以何种顺序去执行哪些方法
  Beverage.prototype.init = function() {
    this.boilWater();
    this.brew();
    this.pourInCup();
    if (this.customerWantsCondiments()) {
      this.addCondiments();
    }
  }
  
  const Coffee = function() {};
  Coffee.prototype = new Beverage();
  
  Coffee.prototype.brew = function() {
    console.log('用沸水冲泡咖啡');
  }
  
  Coffee.prototype.pourInCup = function() {
    console.log('把咖啡倒进杯子');
  }
  
  Coffee.prototype.addCondiments = function() {
    console.log('加糖和牛奶');
  }
  
  Coffee.prototype.customerWantsCondiments = function() {
    return window.confirm('请问需要调料嘛？');
  }
  
  const coffee = new Coffee();
  coffee.init();
  
  const Tea = function() {};
  Tea.prototype = new Beverage();
  
  Tea.prototype.brew = function() {
    console.log('用沸水浸泡茶叶');
  }
  
  Tea.prototype.pourInCup = function() {
    console.log('把茶倒进杯子');
  }
  
  Tea.prototype.addCondiments = function() {
    console.log('加柠檬');
  }
  
  const tea = new Tea();
  tea.init();
  ```

  

### 享元模式

- 用于性能优化，如系统中因为创建了大量类似的对象而导致内存占用过高。

- 核心：运用共享技术来有效支持大量细粒度的对象。

- 关键：将对象划分为内部状态和外部状态。

  - 内部状态存储于对象内部；
  - 内部状态可以被一些对象共享；
  - 内部状态独立于具体的场景，通常不会改变；
  - 外部状态取决于具体场景，并根据场景变化，不能被共享。

- 过程：剥离外部状态，并把外部状态保存在其他地方，在核实的时刻再把外部状态组装进共享对象。

- 适用性：

  - 使用了大量的相似对象；
  - 由于使用了大量对象，造成很大的内存开销；
  - 大多数状态都可以变成外部状态；
  - 剥离出对象的外部状态之后，可以用相对较少的共享对象取代大量的对象。

  ```javascript
  const Upload = function(uploadType) {
    this.uploadType = uploadType;
  }
  
  Upload.prototype.delFile = function(id) {
    uploadManager.setExternalState(id, this);
    if (this.fileSize < 3000) {
      return this.dom.parentNode.removeChild(this);
    }
    if (window.confirm('确定删除该文件嘛？ ' + this.fileName)) {
      return this.dom.parentNode.removeChild(this.dom);
    }
  }
  
  const UploadFactory = (function() {
    const createdFlyWeightObjs = {};
    return {
      create: function(uploadType) {
        if (createdFlyWeightObjs[uploadType]) {
          return createdFlyWeightObjs[uploadType];
        }
        return createdFlyWeightObjs[uploadType] = new Upload(uploadType);
      }
    }
  })();
  
  const uploadManager = (function() {
    const uploadDatabase = {};
    return {
      add: function(id, uploadType, fileName, fileSize) {
        const flyWeightObj = UploadFactory.create(uploadType);
        const dom = document.createElement('div');
        dom.innerHTML = '<span>文件名称：' + fileName + ', 文件大小：' + fileSize + '</span>' + 
                        '<button class="delFile">删除</button>';
        dom.querySelector('.delFile').onclick = function() {
          flyWeightObj.delFile(id);
        }
        document.body.appendChild(dom);
  
        uploadDatabase[id] = {
          fileName,
          fileSize,
          dom,
        }
        return flyWeightObj;
      },
      setExternalState: function(id, flyWeightObj) {
        const uploadData = uploadDatabase[id];
        for (let i in uploadData) {
          flyWeightObj[i] = uploadData[i];
        }
      }
    }
  })();
  ```

- 可以没有内部状态，也可以没有外部状态。

- 对象池：维护一个装载空闲对象的池子，如果需要对象的时候，不是直接new，而是转从对象池里获取。如果对象池里没有空闲对象，则创建新的对象，当获取出的对象完成职责后，再进入池子等待下次获取。是另外一种性能优化方案，但没有分离内部状态和外部状态的过程。

  ```javascript
  const objectPoolFactory = function(createObjFn) {
    const objectPool = [];
    return {
      create: function() {
        const obj = objectPool.length === 0 ? createObjFn.apply(this, arguments) : objectPool.shift();
        return obj;
      },
      recover: function(obj) {
        objectPool.push(obj);
      }
    }
  }
  
  const iframeFactory = objectPoolFactory(function() {
    const iframe = document.createElement('iframe');
    document.body.appendChild(iframe);
    iframe.onload = function() {
      iframe.onload = null;
      iframeFactory.recover(iframe);
    }
    return iframe;
  });
  
  const iframe1 = iframeFactory.create();
  iframe1.src = 'http://baidu.com';
  
  const iframe2 = iframeFactory.create();
  iframe2.src = 'http://QQ.com';
  
  setTimeout(function() {
    const iframe3 = iframeFactory.create();
    iframe3.src = 'http://163.com';
  }, 3000);
  ```

  

### 职责链模式

- 定义：使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系，将这些对象（节点）连成一条链，并沿着这条链传递该请求，知道有一个对象处理它为止。

- 优点：

  - 解耦了请求发送者和N个接收者之间的复杂关系，由于不知道链中的哪个节点可以处理请求，所以只需把请求传递给第一个节点即可。
  - 链中的节点对象可以灵活地拆分重组，增加或删除一个节点，或者改变节点在链中的位置都是轻而易举的事情。
  - 可以手动指定起始节点，请求并不是非得从第一个节点开始传递。

- 缺点：

  - 不能保证某个请求一定会被链中的节点处理。可以在链尾增加一个保底的接收者节点来处理这种即将离开链尾的请求。
  - 使得程序中多了一些节点对象，可能大部分节点并没有起到实质性作用，仅是让请求传递下去，从性能方面考虑，要避免过长的职责链带来的性能损耗。

  ```javascript
  const Chain = function(fn) {
    this.fn = fn;
    this.successor = null;
  }
  
  Chain.prototype.setNextSuccessor = function(successor) {
    return this.successor = successor;
  }
  
  Chain.prototype.passRequest = function() {
    const ret = this.fn.apply(this, arguments);
    if (ret === 'nextSuccessor') {
      return this.successor && this.successor.passRequest.apply(this.successor, arguments);
    }
    return ret;
  }
  
  // 异步
  Chain.prototype.next = function() {
    return this.successor && this.successor.passRequest.apply(this.successor, arguments);
  }
  
  const fn1 = new Chain(function() {
    console.log(1);
    return 'nextSuccessor';
  });
  
  const fn2 = new Chain(function() {
    console.log(2);
    const self = this;
    setTimeout(function() {
      self.next();
    }, 1000);
  });
  
  const fn3 = new Chain(function() {
    console.log(3);
  });
  
  fn1.setNextSuccessor(fn2).setNextSuccessor(fn3);
  fn1.passRequest();
  ```

- 用AOP实现

  ```javascript
  Function.prototype.after = function(fn) {
    const self = this;
    return function() {
      const ret = self.apply(this, arguments);
      if (ret === 'nextSuccessor') {
        return fn.apply(this, arguments);
      }
      return ret;
    }
  }
  ```

  

### 中介者模式

- 作用：解除对象与对象之间的紧耦合关系。增加一个中介者对象后，所有的相关对象都通过中介者对象来通信，而不是互相引用，所以当一个对象发生改变时，只需要通知中介者对象即可。中介者使对象之间耦合松散，而且可以独立地改变它们之间地交互。

- 是迎合迪米特法则的一种实现。迪米特法则也叫最少知识原则，是指一个对象应尽可能少的了解另外的对象。

- 优点：使对象之间得以解耦，以中介者和对象之间的一对多关系取代了对象之间的网状多对多关系。各个对象只需关注自身功能的实现，对象之间的交互关系交给了中介者对象来实现和维护。

- 缺点：系统中会新增一个中介者对象，因为对象之间交互的复杂性，转移成了中介者对象的复杂性，使得中介者对象经常是巨大的，难以维护。

  ```javascript
  function Player(name, teamColor) {
    this.name = name;
    this.teamColor = teamColor;
    this.state = 'alive';
  }
  
  Player.prototype.win = function() {
    console.log(this.name + ' won');
  }
  
  Player.prototype.lose = function() {
    console.log(this.name + ' lost');
  }
  
  Player.prototype.die = function() {
    this.state = 'dead';
    playerDirector.ReceiveMessage('playerDead', this);
  }
  
  Player.prototype.remove = function() {
    playerDirector.ReceiveMessage('removePlayer', this);
  }
  
  Player.prototype.changeTeam = function(color) {
    playerDirector.ReceiveMessage('changeTeam', this, color);
  }
  
  const playerFactory = function(name, teamColor) {
    const newPlayer = new Player(name, teamColor);
    playerDirector.ReceiveMessage('addPlayer', newPlayer);
  }
  
  const playerDirector = (function() {
    const players = {}, operations = {};
  
    operations.addPlayer = function(player) {
      const teamColor = player.teamColor;
      players[teamColor] = players[teamColor] || [];
      players[teamColor].push(player);
    }
  
    operations.removePlayer = function(player) {
      const teamColor = player.teamColor;
      const teamPlayers = players[teamColor] || [];
      for (let i = teamPlayers.length - 1; i >= 0; i--) {
        if (teamPlayers[i] === player) {
          teamPlayers.splice(i, 1);
        }
      }
    }
  
    operations.changeTeam = function(player, newTeamColor) {
      operations.removePlayer(player);
      player.teamColor = newTeamColor;
      operations.addPlayer(player);
    }
  
    operations.playerDead = function(player) {
      const teamColor = player.teamColor;
      const teamPlayers = players[teamColor];
      let all_dead = true;
      for (let i = 0, player; player = teamPlayers[i++];) {
        if (player.state !== 'dead') {
          all_dead = false;
          break;
        }
      }
  
      if (all_dead === true) {
        for (let i = 0, player; player = teamPlayers[i++];) {
          player.lose();
        }
        for (let color in players) {
          if (color !== teamColor) {
            const teamPlayers = players[color];
            for (let i = 0, player; player = teamPlayers[i++];) {
              player.win();
            }
          }
        }
      }
    }
  
    const ReceiveMessage = function() {
      const message = Array.prototype.shift.call(arguments);
      operations[message].apply(this, arguments);
    }
  
    return {
      ReceiveMessage,
    }
  })();
  ```

  

### 装饰者模式

- 可以动态地某个对象添加一些额外地职责，而不会影响从这个类中派生的其他对象；
- 比“继承”更加灵活，是一种即用即付的方式。
- 用AOP装饰函数

```javascript
// example1
Function.prototype.after = function(afterfn) {
  const _self = this;
  return function() {
    const ret = _self.apply(this, arguments);
    afterfn.apply(this, arguments);
    return ret;
  }
}

const showLogin = function() {
  console.log('打开登录浮层');
}

const log = function() {
  console.log('上报标签为： ' + this.getAttribute('tag'));
}

showLogin = showLogin.after(log);

document.getElementById('button').onclick = showLogin;

// example2
Function.prototype.before = function(beforefn) {
  const _self = this;
  return function() {
    if (beforefn.apply(this, arguments) === false) {
      return;
    }
    return _self.apply(this, arguments);
  }
}

const ajax = function(type, url, param) {
  console.log(param);
}

const getToken = function() {
  return 'Token';
}

ajax = ajax.before(function(type, url, param) {
  param.Token = getToken();
});

const validata = function() {
  if (username.value === '') {
    alert('用户名不能为空');
    return false;
  }
  if (password.value === '') {
    alert('密码不能为空');
    return false;
  }
}

const formSubmit = function() {
  const param = {
    username: userName.value,
    password: password.value,
  };
  ajax('post', 'http://xxx.com/login', param);
}

formSubmit = formSubmit.before(validata);

submitBtn.onclick = function() {
  formSubmit();
}

```



### 状态模式

- 定义：允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类 。
  - 将对象封装成独立的类，并将请求委托给当前的状态对象，当内部状态改变时，会带来不同的行为变化。
  - 从客户的角度看，对象在不同的状态下具有不同的行为，看起来是从不同的类中实例化而来，实际上是使用了委托的效果。

- 关键：区分事物内部的状态，事物内部状态的改变往往会带来事物的行为改变。把事物的每种状态都封装成单独的类，跟此种状态有关的行为都被封装在这个类的内部。
- 通用结构：
  1. 首先定义Context上下文；
  2. 创建每一个状态类的实例对象，Context将持有这些状态对象的引用，以便请求委托给状态对象。用户的请求也是实现在Context中的。
  3. 编写各种状态类

```javascript
const plugin = (function() {
  const plugin = document.createElement('embed');
  plugin.style.display = 'none';

  plugin.type = 'application/txftn-webkit';

  plugin.sign = function() {
    console.log('开始文件扫描');
  }

  plugin.pause = function() {
    console.log('暂停文件上传');
  }

  plugin.uploading = function() {
    console.log('开始文件上传');
  }

  plugin.del = function() {
    console.log('删除文件上传');
  }

  plugin.done = function() {
    console.log('文件上传完成');
  }

  return plugin;
})();

const Upload = function(fileName) {
  this.plugin = plugin;
  this.fileName = fileName;
  this.button1 = null;
  this.button2 = null;
  this.signState = new SignState(this);
  this.uploadingState = new UploadingState(this);
  this.pauseState = new PauseState(this);
  this.doneState = new DoneState(this);
  this.errorState = new ErrorState(this);
  this.currState = this.signState;
}

Upload.prototype.init = function() {
  this.dom = document.createElement('div');
  this.dom.innerHTML = '<span>文件名称：' + this.fileName + '</span>' + 
                      '<button data-action="button1">扫描中</button>' + 
                      '<button data-action="button2">删除</button>';
  document.body.appendChild(this.dom);
  this.button1 = this.dom.querySelector('[data-action="button1"]');
  this.button2 = this.dom.querySelector('[data-action="button2"]');
  this.bindEvent();
}

Upload.prototype.bindEvent = function() {
  const self = this;
  this.button1.onclick = function() {
    self.currState.clickHandler1();
  }
  this.button2.onclick = function() {
    self.currState.clickHandler2();
  }
}

Upload.prototype.sign = function() {
  this.plugin.sign();
  this.currState = this.signState;
}

Upload.prototype.uploading = function() {
  this.button1.innerHTML = '正在上传，点击暂停';
  this.plugin.uploading();
  this.currState = this.uploadingState;
}

Upload.prototype.pause = function() {
  this.button1.innerHTML = '已暂停，点击继续上传';
  this.plugin.pause();
  this.currState = this.pauseState;
}

Upload.prototype.done = function() {
  this.button1.innerHTML = '上传完成';
  this.plugin.done();
  this.currState = this.doneState;
}

Upload.prototype.error = function() {
  this.button1.innerHTML = '上传失败';
  this.currState = this.errorState;
}

Upload.prototype.del = function() {
  this.plugin.del();
  this.dom.parentNode.removeChild(this.dom);
}

const StateFactory = (function() {
  const State = function() {};

  State.prototype.clickHandler1 = function() {
    throw new Error('子类必须重写父类的clickHandler1方法');
  }

  State.prototype.clickHandler2 = function() {
    throw new Error('子类必须重写父类的clickHandler2方法');
  }

  return function(param) {
    const F = function(uploadObj) {
      this.uploadObj = uploadObj;
    }
    F.prototype = new State();
    for(let i in param) {
      f.prototype[i] = param[i];
    }
    return F;
  }
})();

const SignState = StateFactory({
  clickHandler1: function() {
    console.log('扫描中，点击无效...');
  },
  clickHandler2: function() {
    console.log('文件正在上传中，不能删除');
  }
});

const UploadingState = StateFactory({
  clickHandler1: function() {
    this.uploadObj.pause();
  },
  clickHandler2: function() {
    console.log('文件正在上传中，不能删除');
  }
});

const PauseState = StateFactory({
  clickHandler1: function() {
    this.uploadObj.uploading();
  },
  clickHandler2: function() {
    this.uploadObj.del();
  }
});

const DoneState = StateFactory({
  clickHandler1: function() {
    console.log('文件已完成上传，点击无效');
  },
  clickHandler2: function() {
    this.uploadObj.del();
  }
});

const ErrorState = StateFactory({
  clickHandler1: function() {
    console.log('文件上传失败，点击无效');
  },
  clickHandler2: function() {
    this.uploadObj.del();
  }
});
```

- 优点：

  - 通过增加新的状态类，很容易增加新的状态和转换。
  - 避免Context无限膨胀，状态切换的逻辑分布在状态类中，也去掉了Context中原本过多的条件分支。
  - 用对象代替字符串来记录当前状态，使得状态的切换更一目了然。
  - Context中的请求动作和状态类中封装的行为非常容易地独立变化而互不影响。

- 缺点

  - 在系统中定义许多状态类，系统中因此增加不少对象。
  - 造成了逻辑分散的问题，无法在一个地方看出整个状态机的逻辑。

- 性能优化点：

  - 有两种选择来管理state对象的创建和销毁。
    - 仅当state对象被需要时才创建并随后销毁：如果state对象比较庞大，可以节省内存。
    - 一开始就创建好所有的状态对象：如果状态改变很频繁
  - state对象之间是可以共享的，也是享元模式的应用场景之一。

- Javascript版本的状态机：没有规定让状态对象一定要从类中创建而来，可以非常方便的使用委托技术。

  ```javascript
  const delegate = function(client, delegation) {
    return {
      buttonWasPressed: function() { // 将客户的操作委托给delegation对象
        return delegation.buttonWasPressed.apply(client, arguments);
      }
    }
  }
  
  const FSM = {
    off: {
      buttonWasPressed: function() {
        console.log('关灯');
        this.button.innerHTML = '下一次按我是开灯';
        this.currState = this.onState;
      }
    },
    on: {
      buttonWasPressed: function() {
        console.log('开灯');
        this.button.innerHTML = '下一次按我是关灯';
        this.currState = this.offState;
      }
    }
  }
  
  const Light = function() {
    this.offState = delegate(this, FSM.off);
    this.onState = delegate(this, FSM.on);
    this.currState = this.offState;
    this.button = null;
  }
  
  Light.prototype.init = function() {
    const button = document.createElement('button');
    const self = this;
    button.innerHTML = '已关灯';
    this.button = document.body.appendChild(button);
    this.button.onclick = function() {
      self.currState.buttonWasPressed();
    }
  }
  
  const light = new Light();
  light.init();
  ```

- 表驱动的有限状态机

  []: https://github.com/jakesgordon/javascript-state-machine

  ```javascript
  const fsm = StateMachine.create({
    initial: 'off',
    events: [
      {name: 'buttonWasPressed', from: 'off', to: 'on'},
      {name: 'buttonWasPressed', from: 'on', to: 'off'},
    ],
    callbacks: {
      onbuttonWasPressed: function(event, from, to) {
        console.log(arguments);
      }
    },
    error: function(eventName, from, to, args, errorCode, errorMsg) {
      console.log(arguments);
    }
  });
  ```

  

### 适配器模式

## 设计原则和编程技巧

### 单一职责原则

### 最少知识原则

### 开放-封闭原则

### 接口和面向接口编程

### 代码重构