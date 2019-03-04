title: javascript小知识点
date: 2015-07-30 19:42:43
tags: 经验
---
## 标识符
 - 中文是合法的标识符，可以用作变量名。
 - JavaScript有一些保留字，不能用作标识符：arguments、break、case、catch、class、const、continue、debugger、default、delete、do、else、enum、eval、export、extends、false、finally、for、function、if、implements、import、in、instanceof、interface、let、new、null、package、private、protected、public、return、static、super、switch、this、throw、true、try、typeof、var、void、while、with、yield。
 - Infinity、NaN、undefined虽然不是保留字，但是因为具有特别含义，也不应该用作标识符。
 - JavaScript的标识名区分大小写，所以undefined和null不同于Undefined和Null（或者其他仅仅大小写不同的词形），后者只是普通的变量名。

## typeof运算符
undefined返回undefined    利用这一点，typeof可以用来检查一个没有声明的变量，而不报错。如：

```
v
// ReferenceError: v is not defined
typeof v
// "undefined"
// "undefined"
```
实际编程中，这个特点通常用在判断语句:
```
// 错误的写法：
if (v){
    // ...
}
// ReferenceError: v is not defined
// 正确的写法：
if (typeof v === "undefined"){
    // ...
}
```
## instanceof运算符
既然typeof对数组（array）和对象（object）的显示结果都是object，那么怎么区分它们呢？instanceof运算符可以做到:
```
var o = {};
var a = [];
o instanceof Array // false
a instanceof Array // true
```
## Boolean（）
以下六个值的转化结果为false，其他的值全部为true
undefined
null
-0
+0
NaN
''（空字符串）

> Boolean(undefined) // false
Boolean(null) // false
Boolean(0) // false
Boolean(NaN) // false
Boolean('') // false

## javascript自动添加分号
如果一行的起首是“自增”（++）或“自减”（--）运算符，则它们的前面会自动添加分号。
如果continue、break、return和throw这四个语句后面，直接跟换行符，则会自动添加分号。这意味着，如果return语句返回的是一个对象的字面量，起首的大括号一定要写在同一行，否则得不到预期结果。

## 原始类型值的转换规则
    数值：转换后还是原来的值。
    
    字符串：如果可以被解析为数值，则转换为相应的数值，否则得到NaN。空字符串转为0。

    布尔值：true转成1，false转成0。

    undefined：转成NaN。

    null：转成0。
    
## Number函数和parseInt函数
Number函数和parseInt函数区别主要在于parseInt逐个解析字符，而Number函数整体转换字符串的类型。另外，Number会忽略八进制的前导0，而parseInt不会。

> parseInt('011') // 9
parseInt('42 cats') // 42
parseInt('0xcafebabe') // 3405691582
Number('011') // 11
Number('42 cats') // NaN
Number('0xcafebabe') // 3405691582

## 加法运算符
加法运算符的类型转换，可以分成三种情况讨论:
（1）运算子之中存在字符串
两个运算子之中，只要有一个是字符串，则另一个不管是什么类型，都会被自动转为字符串，然后执行字符串连接运算
（2）两个运算子都为数值或布尔值
这种情况下，执行加法运算，布尔值转为数值（true为1，false为0）
（3）运算子之中存在对象
运算子之中存在对象（或者准确地说，存在非原始类型的值），则先调用该对象的valueOf方法。如果返回结果为原始类型的值，则运用上面两条规则；否则继续调用该对象的toString方法，对其返回值运用上面两条规则。

> 1 + {a:1}
// "1[object Object]"

对象{a:1}的valueOf方法，返回的就是这个对象的本身，因此接着对它调用toString方法。({a:1}).toString()默认返回字符串"[object Object]"，所以最终结果就是字符串“1[object Object]”。

有趣的是，如果更换上面代码的运算次序，就会得到不同的值：
```
{a:1} + 1
// 1
```
原来此时，JavaScript引擎不将{a:1}视为对象，而是视为一个代码块，这个代码块没有返回值，所以被忽略。因此上面的代码，实际上等同于 {a:1};+1 ，所以最终结果就是1。为了避免这种情况，需要对{a:1}加上括号。

## 版权符号

> \XXX 用三位八进制数（0到377）代表一些特殊符号，比如\251表示版权符号。
\xXX 用两位十六进制数（00到FF）代表一些特殊符号，比如\xA9表示版权符号。
\uXXXX 用四位十六进制的Unicode编号代表某个字符，比如\u00A9表示版权符号。

## Object.prototype.toString.call方法

> Object.prototype.toString.call(2) // "[object Number]"
Object.prototype.toString.call('') // "[object String]"
Object.prototype.toString.call(true) // "[object Boolean]"
Object.prototype.toString.call(undefined) // "[object Undefined]"
Object.prototype.toString.call(null) // "[object Null]"
Object.prototype.toString.call(Math) // "[object Math]"
Object.prototype.toString.call({}) // "[object Object]"
Object.prototype.toString.call([]) // "[object Array]"

可以利用这个特性，写出一个比typeof运算符更准确的类型判断函数：
```
var type = function (o){
    var s = Object.prototype.toString.call(o);
        return s.match(/\[object (.*?)\]/)[1].toLowerCase();
};
type({}); // "object"
type([]); // "array"
type(5); // "number"
type(null); // "null"
type(); // "undefined"
type(/abcd/); // "regex"
type(new Date()); // "date"
```
在上面这个type函数的基础上，还可以加上专门判断某种类型数据的方法:
```
 ['Null',
 'Undefined',
 'Object',
 'Array',
 'String',
 'Number',
 'Boolean',
 'Function',
 'RegExp',
 'Element',
 'NaN',
 'Infinite'
].forEach(function (t) {
    type['is' + t] = function (o) {
        return type(o) === t.toLowerCase();
    };
});
type.isObject({}); // true
type.isNumber(NaN); // false
type.isElement(document.createElement('div')); // true
type.isRegExp(/abc/); // true
```
## Object.defineProperty方法
Object.defineProperty方法接受三个参数，第一个是属性所在的对象，第二个是属性名（它应该是一个字符串），第三个是属性的描述对象。比如，新建一个o对象，并定义它的p属性，可以这样写：
```
var o = Object.defineProperty({}, "p", {
        value: 123,
        writable: false,
        enumerable: true,
        configurable: false
});
o.p
// 123
o.p = 246;
o.p
// 123
// 因为writable为false，所以无法改变该属性的值
```
如果一次性定义或修改多个属性，可以使用Object.defineProperties方法：
```
var o = Object.defineProperties({}, {
        p1: { value: 123, enumerable: true },
        p2: { value: "abc", enumerable: true },
        p3: { get: function() { return this.p1+this.p2 },
              enumerable:true,
              configurable:true
        }
});
o.p1 // 123
o.p2 // "abc"
o.p3 // "123abc"
```
上面代码中的p3属性，定义了取值函数get。这时需要注意的是，一旦定义了取值函数get（或存值函数set），就不能将writable设为true，或者同时定义value属性，否则会报错。

## Object.getOwnPropertyNames方法
Object.getOwnPropertyNames方法返回直接定义在某个对象上面的全部属性的名称，而不管该属性是否可枚举。
需要注意的是，当使用var命令声明变量时，变量的configurable为false。
而不使用var命令声明变量时（或者使用属性赋值的方式声明变量），变量的可配置性为true。
