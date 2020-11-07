

# 第一章

Chrome浏览器中除了V8作为Javascript引擎外，还有一个Webkit布局引擎，并支持HTML5

![image-20201107103343170](/home/yrh/.config/Typora/typora-user-images/image-20201107103343170.png)

## Node特点：异步I/O，事件与回调函数，单线程，跨平台

Node作为后端Javascript的运行平台，保留了前端浏览器Javascript中那些熟悉的接口，没有改写语言本身的任何特性，依旧基于作用域和原型连，区别在于它将前端中广泛运用的思想迁移到了服务端。

在Node中，绝大多数的操作都是以异步的方式进行调用（异步I/O）。

Node将前端浏览器中应用广泛且成熟的事件引入后端，配合异步I/O，将事件点暴露给业务逻辑。

与其他的Web后端语言相比，Node除了异步和事件外，回调函数是一个特色

Node保持了Javascript在浏览器中单线程的特点，而且在Node中，Javascritp与其余线程是无法共享任何状态的，单线程的最大好处是不用像多线程编程那样处处在意状态的同步问题，没有死锁，也没有线程上下文交换多带来的性能上的开销，单线程的缺点有无法利用多核CPU，错误会引起整个应用退出，应用的健壮性值得考验，大量计算占用CPU导致无法继续调用异步I/O。

浏览器中Javascript与UI共用一个线程，HTML5定制了Web Workers标准，创建工作线程来进行计算，以解决Javascript大计算阻塞UI渲染的问题，工作线程通过消息传递的方式来传递运行结果，工作线程不能访问到主线程中的UI。

Node采用了与 Web Works相同的思路来解决单线程中大量计算的问题：child_process，通过将计算分发到各个子进程，将大量计算分解掉，然后通过进程之间的事件消息来传递结果。

起初，Node只能在Linux平台上运行，在windows上基于libuv实现跨平台，libuv已经成为许多系统实现跨平台的基础组件

![image-20201107102304441](/home/yrh/.config/Typora/typora-user-images/image-20201107102304441.png)

## Node应用场景：I/O密集型，CPU密集型

关于Node讨论最多的主要有I/O密集型和CPU密集型的场景

Node面向网络且擅长并行I/O，能够有效的组织起更多的硬件资源，I/O密集型的优势主要在于Node利用事件循环的处理能力，而不是启动每一个线程为每一个请求服务，资源占用极少。

在 Node.js 12.11.0，worker_threads 模块正式进入稳定版

至此，Nodejs算是了真正的多线程能力。进程是资源分配的最小单位，线程是CPU调度的最小单位。

**1. Nodejs多线程种类**

Node.js 中有三类线程 (child_process 和 cluster 的实现均为进程)

1. event loop的主线程

2. libuv的异步I/O线程池

3. worker_threads的线程

工作线程对于执行 CPU 密集型的 JavaScript 操作非常有用。 它们在 I/O 密集型的工作中用途不大。 Node.js 的内置的异步 I/O 操作比工作线程效率更高。

与 `child_process` 或 `cluster` 不同， `worker_threads` 可以共享内存。 它们通过传输 `ArrayBuffer` 实例或共享 `SharedArrayBuffer` 实例来实现。

# 第二章：模块

Javascript网页脚本，从表单验证，网页特效，到用户体验，到大型应用开发的变迁，从Web网页到Web应用

![image-20201107105516814](/home/yrh/.config/Typora/typora-user-images/image-20201107105516814.png)

CommonJS规范为JavaScript制定了一个美好的愿景，希望Javascript能够在任何地方运行。

最初，JavaScript主要在前端发挥作用，ECMAScript是JavaScript的官方规范，包含了语言的基本要素，核心库等，在实际应用中，JavaScript能力取决于宿主环境中API的支持，随着HTML5的推进以及 浏览器厂商对规范的支持，浏览器中出现了更多更强大的API供JavaScript使用，对于JavaScript自身而言，它的规范薄弱，后端JavaScript尤其，对JavaScript自身有以下缺陷：

- 没有模块系统
- 标准库较少，对于文件系统，I/O等需求没有标准API
- 没有标准接口，没有定义过如Web服务器或者数据库之类的标准统一接口
- 缺乏包管理系统，JavaScript应用中没有自动加载和安装依赖的能力

![image-20201107110853738](/home/yrh/.config/Typora/typora-user-images/image-20201107110853738.png)

Node借鉴CommonJS的Modules规范实现了一套易用的模块系统

## CommonJS的模块规范

主要分为模块引用，模块定义，模块标识3个部分

1.模块引用，通过require()方法，接受模块标识，引入模块的API到当前上下文中

```
var math = require('math');
```

2.模块定义

在模块中，上下文通过require()方法引入外部模块，同时通过exports对象用于导出当前模块的方法或变量，并且是唯一的出口，在模块中，还存在module对象，代表模块自身，exports是module的属性，在Node中，一个文件就是一个模块，将方法挂载在exports对象上作为属性导出

3.模块标识，require的参数，可以是小驼峰字符串，相对和绝对路径，可以没有.js后缀

![image-20201107112052204](/home/yrh/.config/Typora/typora-user-images/image-20201107112052204.png)

可以不用考虑变量污染，命名空间等，将方法和变量等限定在私有的作用域中，每个模块独立互不干扰

Node并未完全按照规范实现，进行了一定的取舍，同时增加了自身特性，Node模块一类是Node提供的模块，称为核心模块，另一类是用户编写的模块，称为文件模块，核心模块在Node源码的编译过程中，编译进了二进制执行文件，在Node进程启动时，被直接加载进内存中，加载速度最快，文件模块动态加载，经过路径分析，文件定位，编译执行过程，速度较快

模块标识符分类：核心模块（fs, http等），相对绝对路径，非路径模块自定义模块

模块的加载过程，优先从缓存加载，浏览器缓存的是静态脚本文件，Node缓存的是编译和执行后的对象，Node对引入过的对象进行缓存，加载优先级：缓存加载>核心模块>文件模块

模块路径是Node在定位文件模块的具体文件时的查找策略，是一个路径组成的数组，路径规则是由当前路径下的node_modules沿路径向上逐级递归，直到根目录下的node_modules

```
console.log(modules.paths);
```

![image-20201107114305741](/home/yrh/.config/Typora/typora-user-images/image-20201107114305741.png)

在标识符中不存在扩展名时，Node会按.js .json .node的次序补足扩展名查找

可能require查找得到的是一个目录，这在引入自定义模块和逐个模块路径进行查找时经常出现，此时Node将目录当做一个包来处理，首先在当前目录下查找package.json(CommonJS包规范定义的包描述文件)，JSON.parse解析出包描述对象，从中取出main属性指定的文件名定位，若没有package.json或main失败，Node会将index当做默认文件名，然后依次查找index.js index.json index.node

![image-20201107121414672](/home/yrh/.config/Typora/typora-user-images/image-20201107121414672.png)

![image-20201107121126315](/home/yrh/.config/Typora/typora-user-images/image-20201107121126315.png)

![image-20201107121516637](/home/yrh/.config/Typora/typora-user-images/image-20201107121516637.png)

![image-20201107122552284](/home/yrh/.config/Typora/typora-user-images/image-20201107122552284.png)

至此，require, exports, module流程已经完整，这就是Node对CommonJS模块规范的支持

![image-20201107123034243](/home/yrh/.config/Typora/typora-user-images/image-20201107123034243.png)

## 2.3核心模块

c/c++扩展模块，.node文件在windows中是.dll文件，在linux中是.so文件，javascript一个典型弱点是位运算，它中只有double型的数据类型，一个平台下的.node文件在另一个平台下是无法加载的，必须在各自平台下编译为正确的.node文件

GYP项目生成工具，Generate Your Projects，基于GYP提供了一个專有的扩展构建工具node-gyp，通过npm install -g node-gyp安装，.gyp文件是项目文件，node-gyp约定文件是bings.gyp，

然后调用node-gpy configure在当前目录下生成build目录，生成系统相关的项目文件，node-gyp build编译，build/Release/*.node文件生成

node内建模块，需要放置在node命名空间中，并编译进node中，C/C++扩展模块不需要，只需要通过dlopen()方法动态加载

![image-20201107133658834](/home/yrh/.config/Typora/typora-user-images/image-20201107133658834.png)

![image-20201107133758287](/home/yrh/.config/Typora/typora-user-images/image-20201107133758287.png)

NPM与包

![image-20201107134031996](/home/yrh/.config/Typora/typora-user-images/image-20201107134031996.png)

CommonJS包规范，定义package.json为包描述文件，npm的所有行为跟它有关，字段有dependencies当前包所依赖的包列表，npm通过这个属性帮助自动加载依赖的包，scripts脚本说明对象，主要被包管理器用来安装，编译，测试和卸载等，engine支持的javascritp引擎列表

CommonJS包规范是理论，npm是其中的一种实践，对于Node而言，npm帮助完成了第三方模块的发布，安装和依赖等

安装依赖包的全局模式安装如npm install express -g，并不是将一个模块包安装为一个全局包的意思，并不意味着可以从任何地方通过require来引用它，实际上，-g是将一个包安装为全局可用的可执行命令，它根据包描述文件中bin字段的配置，将实际脚本链接到与Node可执行文件相同的路径下

如果Node的可执行文件位置位于/usr/local/bin/node，那么模块目录就是/usr/local/lib/node_modules，最后通过软连接的方式将bin字段配置的可执行文件链接到Node的可执行目录下

npm从本地安装需要指明package.json所在位置，从非官方源安装npm install xxxx --registry=https:xxxxx，npm config set registry https://xxxx 指定默认源

npm init帮助生成package.json

npm adduser注册包仓库帐号，npm publish .将目录打包为一个存档文件上传包， npm ls分析包，npm owner包权限管理

npm开源，可以搭建自己的npm仓库

![image-20201107145547295](/home/yrh/.config/Typora/typora-user-images/image-20201107145547295.png)

npm平台上，包质量良莠不齐，Node代码运行在服务器端，需要考虑安全问题

## 2.7前后端共用模块

一些模块可以在前后端共用，Node模块的引入几乎都是同步的，如果前端也采用同步的方式引入会影响用户体验，鉴于网络原因，CommonJS为后端JavaScript制定的规范不完全适合前端应用场景，AMD规范产生，Asynchronous Module Definition异步模块定义。

AMD规范是CommonJS模块规范的一个延伸，AMD和CMD工作中好像没有使用过

这些规范的目的都是为了 JavaScript 的模块化开发，特别是在浏览器端的。目前这些规范的实现都能达成浏览器端模块化开发的目的。

# 第三章：异步I/O

PHP语言从头到脚都是以同步阻塞的方式执行，Node是首个将异步作为主要编程方式和设计理念，单线程，异步I/O，事件驱动，是node的基调，与Node的事件驱动，异步I/O设计理念比较相近的一个产品是Nginx

利用单线程，远离多线程的死锁，状态同步等问题，异步I/O，让单线程远离阻塞，以更好的利用CPU

![image-20201107160836913](/home/yrh/.config/Typora/typora-user-images/image-20201107160836913.png)

![image-20201107161139042](/home/yrh/.config/Typora/typora-user-images/image-20201107161139042.png)