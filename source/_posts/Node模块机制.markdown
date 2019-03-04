title: Node模块机制
date: 2016-5-20 17:11:19
tags: 经验

---
    js规范缺陷：
          没有模块系统；
          标准库较少；
          没有标准接口；
          缺乏包管理系统。
CommonJS规范提出弥补当前javascript没有标准的缺陷。

CommonJS模块引用：require()

上下文提供了exports对象用于导出当前模块的方法或变量：

    //main.js
    exports.add = function() {
        
    }
    //program.js
    var math = require('math');
    exports.increment = function(val) {
        return math.add(val,1);
    }

错误：

    exports = function(){
        ...
    };

exports对象是通过形参方式传入，直接赋值会改变形参的引用，但不能改变作用域外的值

    var change = function(a) {
        a = 100;
        console.log(a); //100
    };
    var a = 10;
    change(a);
    console.log(a); //10

如果要达到require引入一个类的效果，赋值给module.exports对象，这个迂回的方案不改变形参的作用。

node模块分为node提供的核心模块跟用户编写的文件模块。
核心模块在node源代码编译时已编译，node启动时即加入内存，省去文件定位和编译执行步骤，在路径分析中优先判断，故加载速度最快。
文件模块运行时动态加载，需完整路径分析、文件定位、编译执行过程，速度比较慢。

不论核心模块还是文件模块，对相同模块二次加载一律缓存优先，这是第一优先级的，不同之处在与核心模块的缓存检查优先于文件模块的。

不可以加载一个与核心模块标识符相同的自定义模块。

模块路径是node在定位文件模块的具体文件时制定的查找策略，其类似于javascript的原型链或者作用域链，先在当前文件目录下的node_modules目录查找，然后沿路径向上逐级递归，这是自定义模块加载速度是最慢的原因。

require()在分析标识符的过程中，会出现标识符中不包含文件扩展名的情况，这时按照.js,.json,.node次序查找。

.js文件：通过fs模块同步读取文件后编译执行；
.node文件：是c/c++编写的扩展文件，通过dlopen()方法加载最后编译生成的文件；
.json文件：通过fs模块同步读取文件后，用JSON.parse()解析返回结果；
其余扩展名文件：当作js文件载入。

编译过程中，node会对js文件内容进行包装：

    (function(exports,requiere,module,_filename,_dirname){
        var math = require('math');
        exports.area = function(radius) {
            return Math.PI*radius*radius;
         };
    });

这样每个模块文件之间进行作用域隔离，不污染全局变量。

**包与npm**
---
在一个具体的包目录下执行npm test时，会运行test指向的脚本。
npm init 生成package.json文件
npm adduser 注册账号，npm必须要使用仓库账号才能将包发布到仓库中
npm publish . 上传包
npm owner 管理包的所有者（npm owner ls/npm owner add/npm owner rm）
npm ls 这个命令可以分析当前路径下可以通过模块路径找到的所有包

包质量和安全问题：
口碑效应，在npm模块首页依赖榜可以说明模块的质量和可靠性；
GitHub中模块项目的观察者数量和分支数量也能侧面反应其可靠性和流行度；
包的测试用例，没有单元测试和文档的包基本上是无法被信任的。

AMD规范：

    define(['dep1','dep2'],function(dep1,dep2){
        return function(){};
    });

需要在声明时指定所有依赖
CMD规范：

    define(function(require,exports,module){
        //...
    });

require,exports,module通过形参传递给模块，在需要依赖模块时，随时调用require()引用


兼容多种模块规范
将hello()方法定义到不同环境，兼容node，AMD，CMD及常见浏览器环境：

    ;(function(name,definition){
        //检测上下文环境是否为AMD或CMD
        var hasDefine = typeof define === 'function',
        //检测上下文环境是否为node
        hasExports = typeof module !== 'undefined' && module.exports;
        if(hasDefine){
              //AMD环境或CMD环境
              define(definition);
         }else if(hasExports){
              //定义为普通node模块
              module.exports = definition();
         }else {
              //将模块执行结果挂在window变量中
              this[name] = definition();
         }
    })('hello',function(){
        var hello = function(){};
        return hello;
    });




