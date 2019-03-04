title: 简谈ES6学习心得
date: 2017-3-13 20:08:29
tags: 经验

---

### let和const

 - let和const具有块级作用域，只在当前块级作用域能访问到
 - let和const都没有变量声明提升，会有暂时性死区，不可重复声明。
```    
test = 'abc';
let test;   //报错
```
 - const声明的是常量，在声明之后无法修改它的值，但是声明对象可以修改其属性。
```
const obj = {};
obj.a = 1;
```
 - 典型用例：for循环
```
for(let i=0; i<10; i++) {
    //...
}
console.log(i) //报错
```

### 变量结构赋值
1.解构不成功，变量值则为undefined
2.解构允许有默认值（解构右边值不严格等于undefined，默认值不生效）：

    let [test = 1] = [undefined]; //test为1
    let [test = 1] = [null];  //test为null

3.函数参数的解构赋值+默认值有助于阅读代码和增加代码可维护性：

    function test({x = 0, y=0} = {}) {
        return [x, y];
    }
    function test({x, y} = {x:0, y:0}) {
        return [x, y];
    }
    第一种写法是给参数赋默认值，等号右边传入参数再进行解构赋值；
    第二种写法是在没传参数情况下，整个参数对象有默认值，否则等号右边引用传入对象进行解构赋值。
    
### Promise对象
Promise异步编程解决回调地狱~
Promise特点：
1.状态不受影响，有pending，resolved，rejected三种状态，并且状态一旦改变就不会再变化，在变化以后再给Promise增加回调函数也能获取到一样的结果。
2.Promise对象无法取消，在创建时就立即执行，而且Promise内部错误不会反应到外部，除非设定回调函数，另外，Promise在Pending时无法得知是刚开始还是即将结束。

用法：

    var promise new Promise(function(resolve, reject) {
        if(...) {
            resolve(value);
        }else {
            reject(error);
        }
    });
    promise.then(function(value) {
        //resolve
    }, function(error) {
        //reject
    }).catch(function(error) {
        //处理前面发生的错误
    }).done() //确保抛出错误，因为catch出错它不会自己抓取自己的错误，可提供onFulfilled和Rejected回调函数
    .finally() //无论如何都会执行的操作
    
## 箭头函数-this
ES6里边比较大头的一个家伙~
首先箭头函数可以简便函数的写法，

    function test(arg) {
        ...
    }
    可写成 test = arg => {...}
    function test(x, y) {
        ...
    }
    可写成 test = (x, y) => {...}

但是箭头函数里边比较不同的一点是this的指向：
**箭头函数的this是在定义的时候确定的，而不是在运行时灵活变动的，而箭头函数本身是没有this的，它的this也就是定义时它的上一层函数（或全局变量）的this。**
**例如：**

    var a = 'window';
    let obj = {
        a : 'abc',
        test : function() {
            console.log(this.a);
        }
    }
    obj.test() //'abc'
    
    let obj1 = {
        a : 'abc',
        test : () => {
            console.log(this.a);
        }
    }
    obj1.test() //'window'
    
##ES6模块与CommonJS模块

 1. ES6模块通过import引入，通过export(或exports.name)将模块接口暴露出去，当然也有合并写法，直接exports
    ... from ... (引入并暴露出去)。CommonJS通过require引入模块，通过module.exports暴露接口。
 2. ES6是预编译的，而CommonJS是在运行时确定值的，只是一个值的拷贝，所以使用require(...).name是不保险的，当执行到这句的时候，name不一定存在，就会报错，而ES6则只是一种引用，只要引用存在就不会报错。
 3. CommanJS模块在加载完后会生成一个对象：
```
{
    id: 'name',
    exports: {...},
    loaded: true,
    ...
}
```
当多次执行require命令时，也不会多次执行同一个模块，只从模块的exports中取值，除非手动清除系统缓存。
 4. CommanJS在遇到循环加载时，只会输出已经执行的部分，还未执行部分则不输出，a引用b，b又引用a，则在执行b的过程中，会只获得a的部分引用，执行完b再将执行权交还给a执行，也就会出现**第2点**中不安全现象。
 5. ES6在遇到循环加载时，由于ES6的import不是获取到值，而是一种引用，所以当a引用b，b又引用a时，a执行中遇到引用b，执行b，b引用a发现a已经执行，继续执行b，直至执行完b再将执行权限交给a（此过程如果b引用a中还没执行完的变量，则为undefined）。
 6. 引用阮一峰大神的ES6电子书中的一个例子：
```
// a.js
import {bar} from './b.js';
export function foo() {
  console.log('foo');
  bar();
  console.log('执行完毕');
}
foo();
// b.js
import {foo} from './a.js';
export function bar() {
  console.log('bar');
  if (Math.random() > 0.5) {
    foo();
  }
}
```
ES6在这段代码中能正常执行，而CommonJS会报错，因为b在执行时，foo为null，foo()即会报错。
 7. 当然，两者还是可以兼容互用的，当一个模块使用ES6的方式暴露出来，比如export default name，在CommonJS中可以使用require(...).default属性获取到暴露的接口。

    

 
