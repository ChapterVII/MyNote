# 你不知道的JavaScript（上）

## 作用域和闭包

1. ReferenceError同作用域判别失败相关，而TypeError则代表作用域判别成功了，但是对结果的操作是非法或不合理的。

### 作用域是什么

### 词法作用域

是一套引擎如何寻找变量及会在何处找到变量的规则。定义过程发生在代码的书写阶段。

### 函数作用域和块作用域

### 提升

### 作用域闭包

1. ```javascript
   for (var i = 1; i <= 5; i++) {
      setTimeout(function timer() {
       console.log(i);
      }, i * 1000);
    }
   ```

   运行结果：以每秒一次的频类输出五次6

   原因：尽管循环中的五个函数是在各个迭代中分别定义的，但是它们都被封闭在一个共享的全局作用域中，因此实际上只有一个i

   解决思路：没次迭代都需要一个块作用域

   ```javascript
   for (let i = 1; i <= 5; i++) {
      setTimeout(function timer() {
       console.log(i);
      }, i * 1000);
    }
   ```

   运行结果：分别输出数字1~5，每秒一次，每次一个

   原因：

   	1. let生命可以用来劫持块作用域，并且在这个块作用域中声明一个变量。本质上是将一个块转换成一个可以被关闭的作用域。
   
    	2. for循环头部的let生命还会有一个特殊的行为。这个行为指出变量在循环过程中不知被生命一次。每次迭代都会生命。随后的每个迭代都会使用上一个迭代结束时的值来初始化这个变量。

## this 和 对象原型

### 关于this

this 既不指向函数自身也不指向函数的词法作用域。**是在运行时进行绑定的，并不是在编写时绑定，它的上下文取决于函数调用时的各种条件**。this的绑定和函数生命的位置没有任何关系，只取决于函数的调用方式。

### this全面解析

调用位置：函数在代码中被调用的位置，不是声明的位置。分析调用栈

绑定规则：（优先级由高到低）

1. new 绑定

   构造函数只是一些使用new操作符时被调用的函数。它们并不会属于某个类，也不会实例化一个类，甚至不能说是一种特殊的函数类型，只是被new操作符调用的普通函数而已。

   使用new来调用函数，或者说发生构造函数调用时，会自动执行：

   1. 创建一个全新的对象；
   2. 这个新对象会被执行[[Prototype]]连接
   3. 这个新对象会绑定到函数调用的this
   4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象

2. 显示绑定

   1. call(...)和apply(...)方法

      如果传入了一个原始值（string/boolean/number）来当作this的绑定对象，这个原始值会被转换成它的对象形式

   2. 硬绑定

      ```js
      // example
      function foo(sth) {
          console.log(this.a, sth);
          return this.a + sth;
      }
      
      var obj = {a: 2};
      var bar = function() {
          return foo.apply(obj, arguments);
      }
      
      var b = bar(3);
      console.log(b); // 5
      ```

      1. 可以解决显示绑定仍然存在的丢失绑定问题
      2. 典型场景：创建一个包裹函数，负责接收参数并返回值
      3. ES5内置bind方法

   3. API调用的上下文

      如forEach函数：arr.forEach(callback[, thisArg]);

3. 隐式绑定

   ```javascript
   function foo() {
       console.log(this.a);
   }
   var obj = {a: 2, foo: foo};
   obj.foo(); // 2
   
   var obj1 = {a: 2, obj2: obj2};
   var obj2 = {a: 42, foo: foo};
   obj1.obj2.foo(); // 42
   // 隐式丢失
   var bar = obj.foo;
   var a = 'global';
   bar(); // 'global'
   ```

   1. 调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含
   2. 对象属性引用链中只有上一层或最后一层在调用位置中起作用
   3. 隐式丢失：丢失绑定对象，应用默认绑定

4. 默认绑定

   1. 最常用的函数调用类型：独立函数调用
   2. 无法应用其他规则时的默认规则，指向全局对象
   3. 如果运行在严格模式，则不能将全局对象用于默认绑定，this会绑定到undefined

绑定例外

1. 被忽略的this：call、apply、bind并不关心this的话，可以传入占位值null，但可能产生副作用。如果某个函数确实使用了this（如第三方库），则默认绑定规则会把this绑定到全局对象

2. 更安全的this：一个空的非委托对象，Object.create(null)

3. 间接引用

4. 软绑定

   1. 硬绑定会降低函数的灵活性，无法使用隐式绑定或显示绑定来修改this

   2. 给默认绑定指定一个全局对象和undefined以外的值，可以实现和硬绑定相同的效果，也保留隐式绑定或显示绑定修改this的能力

      ```javascript
      Function.prototype.softBind = function(obj) {
          var fn = this;
          var curried = [].slice.call(arguments, 1);
          var bound = function() {
              return fn.apply((!this || this === (window || global)) ? obj : this, curried.concat.apply(curried, arguments));
          };
          bound.prototype = Object.create(fn.prototype);
          return bound;
      }
      ```

箭头函数：根据外层作用域决定

## 对象

### 语法

### 类型

1. js六种主要类型：string、number、boolean、null、undefined、object

2. 简单基本类型（string、boolean、number、null、undefined）

   typeof null 返回object是js语言本身的一个bug：不同的对象在底层都表示为二进制，在js中二进制前三位都为0的话会被判断为object类型，null的二进制表示全是0

3. 对象子类型（复杂基本类型）：函数&数组

4. 对象子类型（内置对象）

   String、Number、Boolean、Object、Function、Array、Date、RegExp、Error

### 内容

1. 存储在对象容器内部的是属性的名称，它们就像指针（引用）一样，指向这些值真正的存储位置

2. 属性名永远都是字符串，如果使用string意外的其他值作为属性名，那它首先会被转换为一个字符串。

3. 可以给数组添加属性，如果添加的属性名像个数字（如字符串‘3’），那它会变成一个数值下标，因此会修改数组的内容而不是添加一个属性。

4. 复制：

   ​	浅复制：如Object.assign()方法

   ​	深复制：如JSON.parse(JSON.stringify(obj))

5. 属性（数据）描述符

   ```javascript
   var myObject = {a: 2};
   Object.getOwnPropertyDescriptor(myObject, "a");
   // {
   	value: 2,
       writable: true,  // 可写，是否可以修改属性的值
       enumerable: true,  // 可枚举，
       configurable: true  // 可配置
   }
   
   ```

   可以使用Object.defineProperty来添加一个新属性或修改一个已有属性（configurable）并对特性进行设置

   1. 可以把writable: false 看作属性不可改变，相当于定义了一个空操作setter，严格来说，setter被调用时应当抛出一个TypeError错误
   2. 把configurable修改成false是单向操作，无法撤销。
   3. 例外：即便属性是configurable: false，还是可以把writable状态由ture改为false，但是无法由false改为true。
   4. configurable: false还会禁止删除这个属性

6. 不变性

   1. 对象常量：结和writable: false和configurable: false就可以创建（不可修改、重定义或删除）
   2. 禁止扩展：Object.preventExtensions()，禁止一个对象添加新属性并且保留已有属性。
   3. 密封：Object.seal()，实际上会在一个现有对象上调用Object.preventExtensions()，并把所有现有属性标记为configurable: false。
   4. 冻结：Object.freeze()，实际上会在一个现有对象上调用Object.seal()并把所有“数据访问”属性标记为writable: false

7. [[Get]]、[[Put]]

   myObject.a在myObject上实际上是实现了[[Get]]操作

8. Getter&Setter(访问描述符)

   对象默认的[[Get]]、[[Put]]可以控制属性值的获取和设置。可以使用getter和setter部分改写默认操作，但是只能应用在单个属性上，无法应用在整个对象上。

   ```javascript
   var myObject = {
       get a() {
           return 2;
       }
   }
   
   Object.defineProperty(myObject, 'b', {
       get: function() {
           return this.a * 2;
       }
       enumberable: true,
   });
   
   myObject.a;  // 2
   myObject.b;  // 4
   
   myObject.a = 3;
   myObject.a;  // 2
   ```

   1. 不管是对象文字语法中的get a() {} 还是defineProperty中的显示定义，都会在对象中创建一个不包含值的属性，对于这个属性的访问会自动调用一个隐藏函数。

   2. 由于之定义了a的getter，所以对a的值进行设置时set操作会忽略赋值操作，不会抛出错误。而且即便有合法的setter，由于自定义的getter只会返回2，所以set操作时没有意义的；

   3. 通常来说getter和setter是成对出现的

      ```javascript
      var myObject = {
          get a() {
              return this._a_;
          }
          set a(val) {
              this._a_ = val * 2;
          }
      };
      
      myObject.a = 2;
      Object.a;  // 4
      ```

9. 存在性：在不访问属性值的情况下判断对象中是否存在这个属性

   1. in操作符：会检查属性是否在对象及其[[Prototype]]原型链中。
   2. hasOwnProperty指挥检查属性是否在对象中，不会检查[[Prototype]]链。
   3. 所有的普通对象都可以通过对于Object.prototype得分委托来访问hasOwnProperty，但是有的对象可能没有连接到Object.prototype（通过Object.create(null)来创建），在这种情况下，形如myObject.hasOwnProperty()就会失败，可以使用一种更加强硬的方法来判断：Object.prototype.hasOwnProperty.call(myObject, 'a')
   4. 对于数组来说，4 in [2, 4, 6]的结果并不是true，因为 [2, 4, 6]包含的属性名是0、1、2，没有4
   5. enumerable可枚举就相当于可以出现在对象属性的遍历中。
   6. propertyIsEnumerable()会检查给定的属性名是否直接存在于对象中（而不是在原型链上），并且满足enumerable: true
   7. Object.keys()会返回一个数组，包含所有可枚举属性
   8. Object.getOwnPropertyNames()会返回一个数组，包含所有属性，无论是否可枚举。

### 遍历

遍历数组下边时采用的是数字顺序，但是遍历对象属性时的顺序是不确定的。在不同的javascript引擎中可能不一样。

for...of 遍历值，会寻找内置或自定义的@@iterator对象并调用它的next()方法来遍历数据值。



## 类

### 理论

### 机制

### 继承

### 混入

## 原型

### [[Prototype]]

### 类

### 原型继承

### 对象关联

## 行为委托

### 面向委托的涉及

### 类与对象

### 更简洁的设计

### 更好的语法

### 内省

## ES6的class

