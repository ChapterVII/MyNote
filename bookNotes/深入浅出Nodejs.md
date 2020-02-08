# 深入浅出Nodejs

## Node简介

### Node的诞生历程

### Node的命名与起源

- 考虑到高性能、符合事件驱动、没有历史包袱这3个主要原因，JavaScript成为了Node的实现语言

### Node给JavaScript带来的意义

- Node与浏览器的对比

  ![Chrome VS Node](https://github.com/GrowLegend/MyNote/blob/master/static/images/深入浅出Nodejs/Chrome%20VS%20Node.png)

  - 浏览器中除了V8作为JavaScript引擎外还有一个WebKit布局引擎。HTML5在发展过程中定义了更多更丰富的API。在实现上，浏览器提供了越来越多的功能暴露给Javascript和HTML标签。但对于前端浏览器的发展现状而言，HTML5标准统一的过长是相对缓慢的。Javascript作为一门图灵完备的语言，长久以来却限制在浏览器的沙箱中运行，它的能力取决于浏览器中间层提供的支持有多少。
  - 除了HTML、webKit和显卡这些UI相关技术没有支持外，Node与Chrome十分相似。都是基于事件驱动的异步架构，浏览器通过事件驱动来服务界面上的交互，Node通过事件驱动来服务I/O。Node打破了过去JavaScript只能在浏览器中运行的局面。前后端编程环境统一，可以大大降低前后端转换所需要的上下文交换代价。

### Node的特点

将前端中广泛运用的思想迁移到了服务器端。

- 异步I/O：Node中绝大多数的操作都以异步的方式进行调用，底层构建了很多异步I/O，从文件读取到网络请求等。意义在于，可以从语言层面很自然地进行并行I/O操作。每个调用之间无须等待之前地I/O调用结束。在编程模型上可以极大提升效率。

- 事件与回调函数：事件地编程方式具有轻量级、松耦合、只关注事务点等优势，但在多个异步任务的场景下，事件与事件之间各自独立，如何协作是一个问题。回调函数是最好的接受异步调用返回数据的方式。

- 单线程：Node中JavaScript与其余线程是无法共享任何状态的。

  - 好处：不用处处在意状态的同步问题，没有死锁的存在，也没有线程上下文交换所带来的性能上的开销。

  - 弱点：

    - 无法利用多核CPU
    - 错误会引起整个应用退出，应用的健壮性值得考验。
    - 大量计算占用CPU导致无法继续调用异步I/O。

    Node采用了child_process来解决单线程中大计算量的问题。子进程的出现，意味着Node可以从容地应对单线程在健壮性和无法利用多核CPU方面地问题。通过将计算分发到各个子进程，可以将大量计算分解掉，然后再通过 进程之间地事件消息来传递结果，可以很好地保持应用模型地简单和低依赖。通过Master-Worker的管理方式，也可以很好地管理各个工作进程，以达到更高地健壮性。

- 跨平台：兼容Windows和*nix平台主要得益于Node在架构层面的改动，在操作系统与Node上层模块系统之间构建了一层平台层架构，即libuv。libuv已成为许多系统实现跨平台的基础组件。通过良好的架构，Node的第三方C++模块也可以借助libuv实现跨平台。

  ![Node跨平台](https://github.com/GrowLegend/MyNote/blob/master/static/images/深入浅出Nodejs/Node跨平台.png)

### Node的应用场景

- I/O密集型：Node面向网络且擅长并行I/O，能够有效地组织起更多的硬件资源，从而提供更好的服务。I/O密集的优势主要在于Node利用事件循环的处理能力，而不是启动每一个线程为每一个请求服务，资源占用极少。
- CPU密集型
  - V8的执行效率是十分高的。Node优秀的运算能力主要来自V8的深度性能优化。
  - CPU密集型应用给Node带来的挑战主要是：由于JavaScript单线程的原因，如果有长时间运行的计算（比如大循环），将会导致CPU时间片不能释放，使得后续I/O无法发起。但是适当调整和分解大型运算任务为多个小任务，使得运算能够适时释放，不阻塞I/O调用的发起，这样既可同时享受到并行异步I/O的好处，又能充分利用CPU。I/O阻塞造成的性能浪费远比CPU的影响小。
  - 对于长时间运行的计算，如果耗时超过普通阻塞I/O的耗时，那么应用场景就需要重新评估，因为这类计算比阻塞I/O还影响效率，甚至说就是一个纯计算的场景，根本没有I/O。
    - Node可以通过编写C/C++扩展的方式更高效地利用CPU，将一些V8不能做到性能极致地地方通过C/C++来实现。
    - 如果单线程地Node不能满足需求，甚至用了C/C++扩展后还觉得不够，那么通过子进程的方式，将一部分Node进程当做常驻服务进程用于计算，然后利用进程间的消息来传递结果，将计算与I/O分离，这样还能充分利用多CPU。
- 与遗留系统和平共处：旧有的系统具有非常稳定的数据输出，持续为传统网站服务，同时为移动版提供数据源，Node将该数据源当作数据接口，发挥异步并行的优势，而不用关心背后是用什么语言实现的。
- 分布式应用：对可伸缩性的要求非常高

### Node的使用者

- 前后端编程语言环境统一：雅虎Cocktail框架。
- Node带来的高性能I/O用于实时应用：Voxer实时语音，腾讯朋友网长连接，花瓣网、蘑菇街通过socket.io实时通知。
- 并行I/O使得使用者可以更高效地利用分布式环境：阿里巴巴的NodeFox、eBay的ql.io。
- 并行I/O，有效利用稳定接口提升Web渲染能力：雪球财经和LinkedIn的移动版网站。
- 云计算平台提供Node支持：微软Azure、阿里云、百度云服务器提供Node应用托管服务。
- 游戏开发领域：网易pomelo实时框架
- 工具类应用

## 模块机制

### CommonJS规范

- 愿景：希望JavaScript能在任何地方运行。

- 主要是为了弥补当前JavaScript没有标准的缺陷：

  - 没有模块系统。
  - 标准库较少：ECMAScript仅定义了部分核心库，对于文件系统、I/O流等常见需求却没有标准的API。
  - 没有标准接口：JavaScript几乎没有定义过如Web服务器或数据库之类的标准统一接口。
  - 缺乏包管理系统：导致JavaScript应用中基本没有自动加载和安装依赖的能力。

  以达到像Python、Ruby、Java具备开发大型应用的基础能力，而不是停留在小脚本程序的阶段。期望用CommonJS API写出的应用可以具备跨宿主环境执行的能力，不仅可以利用JavaScript开发富客户端应用，还可以编写以下应用。

  - 服务器端JavaScript应用程序。
  - 命令行工具。
  - 桌面图形界面应用程序。
  - 混合应用。

- CommonJS规范涵盖了模块、二进制、Buffer、字符集编码、I/O流、进程环境、文件系统、套接字、单元测试、Web服务器网关接口、包管理等。

- Node能以一种熟悉的姿态出现，离不开CommonJS规范的影响；CommonJS能以一种寻常的姿态写进各个公司的项目代码中，离不开Node优异的表现

  ![Node和CommonJS关系](https://github.com/GrowLegend/MyNote/blob/master/static/images/深入浅出Nodejs/Node和CommonJS关系.png)

- CommonJS的模块规范

  1. 模块引用：require()方法
  2. 模块定义：对应引入的功能，上下文提供了exports对象用于导出当前模块的方法或者变量，是唯一导出的出口。模块中还存在一个module对象，代表模块自身，而exports是module的属性。Node中一个文件就是一个模块。
  3. 模块标识：传递给require()方法的参数，必须是符合小驼峰命名的字符串，或路径，可以没有文件名后缀.js。

  ```javascript
  // math.js
  exports.add = function() {}
  
  // program.js
  var math = require('math');
  exports.increment = function(val) {
      return math.add(val, 1);
  }
  ```

  模块的意义：将类聚的方法和变量等限定在私有的作用域中，同时支持引入和导出共嗯那个以顺畅地连接上下游依赖。

### Node的模块实现

- 引入模块需要经历：路径分析、文件定位、编译执行

- 模块分类：

  - 核心模块（Node提供的）：在Node源代码的编译过程中，编译进了二进制执行文件。在Node进程启动时，部分核心模块就被直接加载进内存中。引入时，文件定位和编译执行可以省略，并且在路径分析中优先判断，所以加载速度最快。
  - 文件模块（用户编写的）：在运行时动态加载，需要完整的路径分析、文件定位、编译执行过程，速度比核心模块慢。

- 优先从缓存加载：Node对引入过的模块都会进行缓存，以减少二次引入的开销，缓存的是编译和执行之后的对象。require()方法对相同模块的二次加载都一律采用缓存有限的方式，是第一优先级的，核心模块的缓存检查先于文件模块。

- 路径分析和文件定位

  1. 模块标识符分析

     - 核心模块：如http、fs、path等，优先级仅次于缓存加载。

     - 路径形式的文件模块：以.、..和/开始的标识符，require方法会将路径转为真实路径，并以真实路径作为索引，将编译执行后的结果放到缓存中。加载速度慢于核心模块。

     - 自定义模块：可能是一个文件或者包的形式。最费时最慢。

       模块路径：Node在定位文件模块的具体文件时指定的查找策略，具体表现为一个路径组成的数组。生成规则：沿路径向上逐级递归，知道根目录下的node_modules目录。

       在加载的过程中，Node会逐个尝试模块路径中的路径，知道找到目标文件为止。当前文件的路径越深，模块查找耗时会越多，是自定义模块的加载速度最慢的原因。

  2. 文件定位

     - 文件扩展名分析：标识符中不包含扩展名时，Node会按.js、.json、.node的次序补足扩展名，依次尝试。过程中，调用fs模块同步阻塞式地判断文件是否存在。

       诀窍：如果是.node和.json文件，带上扩展名会加快速度。同步配合缓存，可以大幅度环境Node单线程中阻塞式调用地缺陷。

     - 目录分析和包

       ![Node目录分析和包](https://github.com/GrowLegend/MyNote/blob/master/static/images/深入浅出Nodejs/Node目录分析和包.png)

- 模块编译

  - Node中每个文件模块都是一个对象。

    ```javascript
    function Module(id, parent) {
        this.id = id;
        this.exports = {};
        this.parent = parent;
        if (parent && parent.children) {
            parent.children.push(this);
        }
        this.filename = null;
        this.loaded = false;
        this.children = [];
    }
    ```

  - 定位到具体的文件后，Node会新建一个模块对象，然后根据路径载入并编译。对于不同的文件扩展名，其载入方法也有所不同

    - **.js文件**：通过fs模块同步读取文件后编译执行
    - **.node文件**：是用C/C++编写的扩展文件，通过dlopen()方法加载最后编译生成的文件。
    - **.json文件**：通过fs模块同步读取文件后，用JSON.parse()解析返回结果。
    - **其余扩展名文件**：都被当作.js文件载入。

  - 每一个编译成功的模块都会将其文件路径作为索引缓存在Module._cache对象上，以提高二次引入的性能。

  1. JavaScript模块的编译：Node对获取的JavaScript文件内容进行了头尾包装。每个模块文件之间都进行了作用域隔离。包装之后的代码会通过vm原生模块的runInThisContext()方法执行，返回一个具体的function对象。最后将当前模块对象的exports属性、require方法、module、以及在文件定位中得到的完整文件路径和文件目录作为参数传递给这个function执行。这就是这些变量没有定义在每个模块文件中却存在的原因。执行后，exports属性被返回给了调用方。

     ```javascript
     (function(exports, require, module, __filename, __dirname) {
         var math = require('math');
         exports.area = function(radius) {
             return Math.PI * radius * radius;
         };
     })
     ```

  2. C/C++模块的编译：Node调用process.dlopen()方法进行加载和执行。dlopen方法在Windows和*nix平台下分别有不同的实现，通过libuv兼容层进行了封装。不需要编译，只有加载和执行的过程。在执行的过程中，模块的exports对象与.node模块产生联系，然后返回给调用者。优势主要是执行效率方面，劣势则是编写门槛比JavaScript高。

  3. JSON文件的编译：最简单。利用fs模块同步读取JSON文件的内容后调用JSON.parse方法得到对象，将它赋给模块对象的exports，以供外部调用。在用作项目的配置文件时比较有用。不必调用fs模块去异步读取和解析，直接调用require引用即可。

### 核心模块

分为C/C++编写的和JavaScript编写的两部分，其中C/C++文件存放在Node项目的src目录下，JavaScript文件存放在lib目录下。

1. JavaScript核心模块的编译过程：在编译所有C/C++文件之前，编译程序需要将所有的JavaScript模块文件编译为C/C++代码。

   1. 转存为C/C++代码：Node采用了V8附带的js2c.py工具，将所有内置的JavaScript代码（src/node.js和lib/*.js）转换成C++里的数组，生成node_natives.h头文件。在这个过程中，JavaScript代码以字符串的形式存储在node命名空间中，是不可直接执行的。在启动Node进程时，JavaScript代码直接加载进内存中。在加载过程中，JavaScript核心模块经历标识符分析后直接定位到内存中，比普通的文件模块从磁盘中一处一处查找要快很多。

   2. 编译JavaScript核心模块：与文件模块有区别的地方在于获取源代码的方式（核心模块是从内存中加载的）以及缓存执行结果的位置。源文件通过process.binding('natives')取出，编译成功的模块缓存到NativeModule._cache对象上。

      ```javascript
      function NativeModule(id) {
      	this.filename = id + '.js';
          this.id = id;
          this.exports = {};
          this.loaded = false;
      }
      NativeModule._source = process.binding('natives');
      NativeModule._cache = {};
      ```

2. C/C++核心模块的编译过程

   核心模块有些全部由C/C++编写，有些则由C/C++完成核心部分，其他部分由JavaScript实现包装或向外导出，以满足性能需求。C++模块主内完成核心，JavaScript主外实现封装的模式是Node能够提高性能的常见方式，可以在开发速度和性能之间找到平衡点。由C/C++编写的部分统一成为内建模块，通常不被用户直接调用，如buffer、crypto、evals、fs、os等模块。

   - 内建模块的组织形式：每一个内建模块在定义之后，都通过NODE_MODULE宏将模块定义到node命名空间中，模块的具体初始化方法挂载为结构的register_func成员。node_extension.h文件将这些散列的内建模块（node_buffer、node_crypto、node_evals、node_fs、node_http_parser、node_os、node_zlib、node_timer_wrap、node_tcp_wrap、node_udp_wrap、node_pipe_wrap、node_cares_wrap、node_tty_wrap、node_process_wrap、node_fs_event_wrap、node_signal_watcher）统一放进了一个叫node_module_list的数组中。

   - 内建模块的导出：

     Node所有模块类型中存在一种依赖层级关系：文件模块可能依赖核心模块，核心模块可能会依赖内建模块。

     不推荐文件模块直接调用内建模块，如需调用，直接调用核心模块即可。

     Node在启动时会生成一个全局变量process，并提供Binding()方法来协助加载内建模块：先创建一个exports空对象，然后调用get_builtin_module方法取出内建模块对象，通过执行register_func填充exports对象，最后将exports对象按模块名缓存，并返回给调用方完成导出。

3. 核心模块的引入流程

4. 编写核心模块：编写内建模块分为：编写头文件和编写C/C++文件。

### C++扩展模块

- JavaScript的一个典型弱点：位运算。参照Java的位运算实现，但是Java位运算是在int型数字的基础上进行的，而JavaScript只有double型的数据类型，在进行位运算的过程中，需要将double型转换为int型，然后再进行。所以，在JavaScript层面上做位运算的效率不高。
- 应用中会频繁出现位运算的需求，包括转码、编码等过程，如果通过JavaScript来实现，CPU资源将会耗费很多，可以编写C/C++扩展模块来提升性能。
- Node原生模块一定程度上可以跨平台的前提条件是源代码可以支持在*nix和Windows上编译，其中 *nix下通过g++/gcc等编译器编译为动态链接共享对象文件（.so），在Windows下则需要通过Visual C++的编译器编译为动态链接库文件（.dll）。.node的扩展名只是为了看起来自然一点，不会因为平台差异产生不同的感觉。实际上，在Windows下是一个.dll文件，在 *nix下则是一个.so文件。为了实现跨平台，dlopen方法在内部实现时区分了平台，分别用的是加载.so和.dll的方式。
- 一个平台下的.node文件在另一个平台下是无法加载执行的，必须重新用各自平台下的编译器编译为正确的.node文件。

1. 前提条件：深厚的C/C++编程功底

   - GYP（Generate Your Projects）项目生成工具：可以减少编写跨平台项目文件的工作量，Node自身的源码就是通过GYP编译的。专有的扩展构建工具node-gyp。
   - V8引擎C++库。
   - libuv库：是跨平台的一层封装，通过它去调用一些底层操作高效的多。封装的功能包括事件循环、文件操作等。
   - Node内部库。
   - 其他库

2. C/C++扩展模块的编写

3. C/C++扩展模块的编译

4. C/C++扩展模块的加载

   ![require引入.node文件的过程](https://github.com/GrowLegend/MyNote/blob/master/static/images/深入浅出Nodejs/require引入.node文件的过程.png)

### 模块调用栈

### 包与NPM

### 前后端共用模块

### 总结

### 参考资源

## 异步I/O

### 为什么要异步I/O

### 异步I/O实现现状

### Node的异步I/O

### 非I/O的异步API

### 事件驱动与高性能服务器

### 总结

### 参考资源

## 异步编程

### 函数式编程

### 异步编程的优势与难点

### 异步编程解决方案

### 异步并发控制

### 总结

### 参考资源

## 内存控制

### V8的垃圾回收机制与内存限制

### 高效使用内存

### 内存指标

### 内存泄漏

### 内存泄漏排查

### 大内存应用

### 总结

### 参考资源

## 理解Buffer

### Buffer结构

### Buffer的转换

### Buffer的拼接

### Buffer与性能

### 总结

### 参考资源

## 网络编程

### 构建TCP服务

### 构建UDP服务

### 构建HTTP服务

### 构建WebSocket服务

### 网络服务与安全

### 总结

### 参考资源

## 构建Web应用

### 基础功能

### 数据上传

### 路由解析

### 中间件

### 页面渲染

### 总结

### 参考资源

## 玩转进程

### 服务模型的变迁

### 多进程架构

### 集群稳定之路

### Cluster模块

### 总结

### 参考资源

## 测试

### 单元测试

### 性能测试

## 产品化

### 项目工程化

### 部署流程

### 性能

### 日志

### 监控报警

### 稳定性

### 异构共存

### 总结

### 参考资源
