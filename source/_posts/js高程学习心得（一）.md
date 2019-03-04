title: js高程学习心得（一）
date: 2017-01-05
tags: 经验
---

"use strict" //使用严格模式

null //空对象指针

typeof操作符检测基本类型（Null,Undefined,Boolean,String,Number）

instanceof检测引用类型(Object,Array,Date,RegExp,Function)和基本包装类型（类似引用类型，不过是瞬间的，String,Number,Boolean）

typeof 未定义变量 = undefined

typeof null = object

NaN == NaN //false

parseInt()第二个参数带进制

parseFloat()只识别十进制

Number() //null返回0，undefined返回NaN

解析对象都是先调用对象的valueOf()，若为NaN则继续调用toString()

标识符开头：字母，_，或者$

toString()可带参数，表示进制

    var num = 10;
    num.toString(2)  //"1010"

null和undefined没有toString方法

Infinity * 0 = NaN
Infinity / Infinity = NaN
0 / 0 = NaN
~ / 0 = Infinity或-Infinity

label语句可使break,continue返回到指定位置
如：

    start: for(var i = 0; i < 10; i++) {
        for(var j = 0; j < 10; j++) {
            break start;  //将会结束两个循环
        }
    }

with(环境) //不推荐，性能不好

switch(a < b) //这里的比较使用全等比较,不转换类型

arguments对象类型数组，可用[]访问，但不是Array实例，它还有一个属性callee指向arguments所属函数的函数名

**ECMAScript无函数签名，其参数是以一个包含0或多个值的数组形式传递（不能重载但类似）**

**对象按值传递：**

    var person = new Object();
    function a(obj) {
        obj.name = "abc";
        obj = new Object();
        obj.name = "def";
    }
    a(person);
    alert(person.name) //"abc"
    //按值传递，但按引用来引用同一对象

**每个执行环境都有一个与之关联的变量对象（环境中变量+函数）
变量对象形成作用域链，作用域链前端是当前环境变量对象
若环境是函数，则把其活动对象作为变量对象，函数活动对象最初只含有arguments对象**

## 数组方法
pop() //数组尾部移除一项，返回移除项
push() //数组尾部增加项，返回数组长度
shift() //数组头部移除一项，返回移除项
unshift() //数组头部增加项，返回数组长度
slice() //两参数表示起始和结束索引，返回这段索引表示的数组，不影响原数组
splice() //可删除可插入，2参数删除，3参数以上插入

indexOf("项"，起点) //查找项在数组中的索引，默认从0向后查
lastIndexOf与indexOf相反，默认从后向前查
若没找到则返回-1

## 数据，字符处理
toFixed() //字符串形式返回指定个数小数
toExponential() //字符串形式返回指定个数小数指数表示法
toPrecision()  //返回小数格式或指数格式（根据传入数字，参数表示所有数字位数，不包括指数部分）
charAt() //返回指定位置字符
charCodeAt() //返回指定位置字符字符编码

slice(起，末) //参数负时加长度
substring(起，末) //参数负时取0
substr(起，个数) //负时起始位置加长度，个数取0
以上三函数是取出指定位置字符串

trim() //去前置后缀空格
toUpperCase() //大写 toLocaleUpperCase() 针对地区
toLowerCase() //小写 toLocaleLowerCase()

split() //将字符串用指定分隔符分割，存入数组

encodeURI()
decodeURI() //解析
encodeURIComponent()
decodeURIComponent() //解析

    var uri="http://www.wrox.com/illegal value.htm#start";
    encodeURI(uri) //"http://www.wrox.com/illegal%20value.htm#start"
    encodeURIComponent(uri) //"http%3A%2F%2Fwww.wrox.com%2Fillegal%20value.htm%23start"

## 属性描述符
数据属性：
[[Enumerable]],
[[Writable]],
[[Value]],
[[Configurable]] //改为false就不能改true
Object.defineProperty()
Object.defineProperties() //定义多个属性

    var person = {};
    Object.defineProperty(person,"name",{
        writable : false,
        value : "Jacy"
    })
    alert(person.name) //"Jacy"
    person.name = "jacy";
    alert(person.name) //"Jacy"

访问器属性：
[[Enumerable]],
[[Configurable]],
[[Get]]
[[Set]]
Object.getOwnPropertyDescriptor(object,"name")

    Object.getOwnPropertyDescriptor(person,"name")
    //Object {value: "Jacy", writable: false, enumerable: false, configurable: false}
    
Object.getPrototypeOf() //获取对象的原型

    Object.getPrototypeOf(person)
    //Object {}
    
Object.keys() //返回对象实例可枚举属性（数组）
Object.getOwnPropertyNames() //返回对象实例所有属性（数组）
