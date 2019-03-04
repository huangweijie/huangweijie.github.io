title: Object对象
date: 2015-07-30 19:02:41
tags: 经验
---
## Object()
Object本身当作工具方法使用时，可以将任意值转为对象。

> Object() // 返回一个空对象
Object(undefined) // 返回一个空对象
Object(null) // 返回一个空对象
Object(1) // 等同于 new Number(1)
Object('foo') // 等同于 new String('foo')
Object(true) // 等同于 new Boolean(true)
Object([]) // 返回原数组
Object({}) // 返回原对象
Object(function(){}) // 返回原函数

如果Object函数的参数是一个对象，它总是返回原对象。利用这一点，可以写一个判断变量是否为对象的函数。
```
function isObject(value) {
    return value === Object(value);
}
```
## Object.keys()，Object.getOwnPropertyNames()
Object.keys方法和Object.getOwnPropertyNames方法很相似，一般用来遍历对象的属性。它们的参数都是一个对象，都返回一个数组，该数组的成员都是对象自身的（而不是继承的）所有属性名。它们的区别在于，Object.keys方法只返回可枚举的属性，Object.getOwnPropertyNames方法还返回不可枚举的属性名。
```
var o = {
    p1: 123,
    p2: 456
}; 

Object.keys(o)
// ["p1", "p2"]

Object.getOwnPropertyNames(o)
// ["p1", "p2"]
```
涉及不可枚举属性时，就有不一样的结果：
```
var a = ["Hello", "World"];

Object.keys(a) 
// ["0", "1"]

Object.getOwnPropertyNames(a)
// ["0", "1", "length"]
```
由于JavaScript没有提供计算对象属性个数的方法，所以可以用这两个方法代替:

> Object.keys(o).length
Object.getOwnPropertyNames(o).length

一般情况下，几乎总是使用Object.keys方法，遍历数组的属性。
## Object.observe()
Object.observe方法用于观察对象属性的变化:
```
var o = {};

Object.observe(o, function(changes) {
  changes.forEach(function(change) {
    console.log(change.type, change.name, change.oldValue);
  });
});

o.foo = 1; // add, 'foo', undefined
o.foo = 2; // update, 'foo', 1
delete o.foo; // delete, 'foo', 2
```
上面代码表示，通过Object.observe函数，对o对象指定回调函数。一旦o对象的属性出现任何变化，就会调用回调函数，回调函数通过一个参数对象读取o的属性变化的信息。
但该方法非常新，只有Chrome浏览器的最新版本才部署。
## Object实例对象的方法

> valueOf()：返回当前对象对应的值。
toString()：返回当前对象对应的字符串形式。
toLocalString()：返回当前对象对应的本地字符串形式。
hasOwnProperty()：判断某个属性是否为当前对象自身的属性，还是继承自原型对象的属性。
isPrototypeOf()：判断当前对象是否为另一个对象的原型。
propertyIsEnumerable()：判断某个属性是否可枚举。

## Object.prototype.valueOf()
```
var o = new Object();
o.valueOf = function (){return 2;};

1 + o // 3
```
上面代码自定义了o对象的valueOf方法，于是1 + o就得到了3。这种方法就相当于用o.valueOf覆盖Object.prototype.valueOf。
## Object.prototype.toString()
```
var o1 = new Object();
o1.toString() // "[object Object]"

var o2 = {a:1};
o2.toString() // "[object Object]"
```
## toString()的应用：判断数据类型

> 数值：返回[object Number]。
字符串：返回[object String]。
布尔值：返回[object Boolean]。
undefined：返回[object Undefined]。
null：返回[object Null]。
对象：返回"[object " + 构造函数的名称 + "]" 。

```
Object.prototype.toString.call(2) // "[object Number]"
Object.prototype.toString.call('') // "[object String]"
Object.prototype.toString.call(true) // "[object Boolean]"
Object.prototype.toString.call(undefined) // "[object Undefined]"
Object.prototype.toString.call(null) // "[object Null]"
Object.prototype.toString.call(Math) // "[object Math]"
Object.prototype.toString.call({}) // "[object Object]"
Object.prototype.toString.call([]) // "[object Array]"
```
可以利用这个特性，写出一个比typeof运算符更准确的类型判断函数。
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
在上面这个type函数的基础上，还可以加上专门判断某种类型数据的方法。
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
## Object.getOwnPropertyDescriptor()&&属性的attributes对象
在JavaScript内部，每个属性都有一个对应的attributes对象，保存该属性的一些元信息。使用Object.getOwnPropertyDescriptor方法，可以读取attributes对象。
```
var o = { p: 'a' };

Object.getOwnPropertyDescriptor(o, 'p') 
// Object { value: "a", 
//         writable: true, 
//         enumerable: true, 
//         configurable: true
// }
```
attributes对象包含如下元信息：

> value：表示该属性的值，默认为undefined。
  writable：表示该属性的值（value）是否可以改变，默认为true。
  enumerable： 表示该属性是否可枚举，默认为true，也就是该属性会出现在   for...in和Object.keys()等操作中。
  configurable：表示“可配置性”，默认为true。如果设为false，表示无法删   除该属性，也不得改变attributes对象（value属性除外），也就是configur   able属性控制了attributes对象的可写性。
  get：表示该属性的取值函数（getter），默认为undefined。
  set：表示该属性的存值函数（setter），默认为undefined。
  
## Object.defineProperty()，Object.defineProperties()
Object.defineProperty方法允许通过定义attributes对象，来定义或修改一个属性，然后返回修改后的对象。它的格式如下：
> Object.defineProperty(object, propertyName, attributesObject)
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
如果一次性定义或修改多个属性，可以使用Object.defineProperties方法。
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
```
var o = {};

Object.defineProperty(o, "p", {
    value: 123, 
    get: function() { return 456; } 
});
// TypeError: Invalid property. 
// A property cannot both have accessors and be writable or have a value,
```
上面代码同时定义了get属性和value属性，结果就报错。
Object.defineProperty() 和Object.defineProperties() 的第三个参数，是一个属性对象。它的writable、configurable、enumerable这三个属性的默认值都为false。

## 可枚举性（enumerable）
如果一个属性的enumerable为false，下面三个操作不会取到该属性。

 - for..in循环
 - Object.keys方法
 - JSON.stringify方法
 
 enumerable可以用来设置“秘密”属性:
```
var o = {a:1, b:2};

o.c = 3;
Object.defineProperty(o, 'd', {
  value: 4,
  enumerable: false
});

o.d
// 4

for( var key in o ) console.log( o[key] ); 
// 1
// 2
// 3

Object.keys(o)  // ["a", "b", "c"]

JSON.stringify(o // => "{a:1,b:2,c:3}"
```

上面代码中，d属性的enumerable为false，所以一般的遍历操作都无法获取该属性，使得它有点像“秘密”属性，但还是可以直接获取它的值。

至于for...in循环和Object.keys方法的区别，在于前者包括对象继承自原型对象的属性，而后者只包括对象本身的属性。如果需要获取对象自身的所有属性，不管enumerable的值，可以使用Object.getOwnPropertyNames方法。
```
var car = {
  id: 123,
  color: red,
  owner: 12
};

var owner = {
  id: 12,
  name: Javi
};

Object.defineProperty( car, 'ownerOb', {value: owner} );
car.ownerOb // {id:12, name:Javi}

JSON.stringify(car) //  '{id: 123, color: "red", owner: 12}'
```
上面代码中，owner对象作为注释，加入car对象。由于ownerOb属性的enumerable为false，所以JSON.stringify最后正式输出car对象时，会忽略ownerOb属性。
## Object.getOwnPropertyNames()
Object.getOwnPropertyNames方法返回直接定义在某个对象上面的全部属性的名称，而不管该属性是否可枚举。
```
var o = Object.defineProperties({}, {
        p1: { value: 1, enumerable: true },
        p2: { value: 2, enumerable: false }
});

Object.getOwnPropertyNames(o)
// ["p1", "p2"]
```
一般来说，系统原生的属性（即非用户自定义的属性）都是不可枚举的。
// 比如，数组实例自带length属性是不可枚举的

> Object.keys([]) // []
Object.getOwnPropertyNames([]) // [ 'length' ]
// Object.prototype对象的自带属性也都是不可枚举的
Object.keys(Object.prototype) // []
Object.getOwnPropertyNames(Object.prototype)
// ['hasOwnProperty',
//  'valueOf',
//  'constructor',
//  'toLocaleString',
//  'isPrototypeOf',
//  'propertyIsEnumerable',
//  'toString']

上面代码可以看到，数组的实例对象（[]）没有可枚举属性，不可枚举属性有length；Object.prototype对象也没有可枚举属性，但是有不少不可枚举属性。
## Object.prototype.propertyIsEnumerable()
判断一个属性是否枚举：
```
var o = {};
o.p = 123;

o.propertyIsEnumerable("p") // true
o.propertyIsEnumerable("toString") // false
```
## 可配置性（configurable）
可配置性（configurable）决定了是否可以修改属性的描述对象。也就是说，当configure为false的时候，value、writable、enumerable和configurable都不能被修改了。
```
var o = Object.defineProperty({}, 'p', {
        value: 1,
        writable: false, 
        enumerable: false, 
        configurable: false
});

Object.defineProperty(o,'p', {value: 2})
// TypeError: Cannot redefine property: p

Object.defineProperty(o,'p', {writable: true})
// TypeError: Cannot redefine property: p

Object.defineProperty(o,'p', {enumerable: true})
// TypeError: Cannot redefine property: p

Object.defineProperties(o,'p',{configurable: true})
// TypeError: Cannot redefine property: p
```
上面代码首先生成对象o，并且定义属性p的configurable为false。然后，逐一改动value、writable、enumerable、configurable，结果都报错。
需要注意的是，writable只有在从false改为true会报错，从true改为false则是允许的。
至于value，只要writable和configurable有一个为true，就可以改动。
```
var o1 = Object.defineProperty({}, 'p', {
        value: 1,
        writable: true,
        configurable: false
});

Object.defineProperty(o1,'p', {value: 2})
// 修改成功

var o2 = Object.defineProperty({}, 'p', {
        value: 1,
        writable: false,
        configurable: true
});

Object.defineProperty(o2,'p', {value: 2}) 
// 修改成功
```
可配置性决定了一个变量是否可以被删除（delete）。
```
var o = Object.defineProperties({}, {
        p1: { value: 1, configurable: true },
        p2: { value: 2, configurable: false }
});

delete o.p1 // true
delete o.p2 // false

o.p1 // undefined
o.p2 // 2
```
当使用var命令声明变量时，变量的configurable为false。
```
var a1 = 1;

Object.getOwnPropertyDescriptor(this,'a1')
// Object {
// value: 1, 
// writable: true, 
// enumerable: true, 
// configurable: false
// }
```
而不使用var命令声明变量时（或者使用属性赋值的方式声明变量），变量的可配置性为true。
```
a2 = 1;
Object.getOwnPropertyDescriptor(this,'a2')
// Object {
// value: 1, 
// writable: true, 
// enumerable: true, 
// configurable: true
// }
```
// 或者写成
```
this.a3 = 1;

Object.getOwnPropertyDescriptor(this,'a3')
// Object {
// value: 1, 
// writable: true, 
// enumerable: true, 
// configurable: true
// }
```
## 可写性（writable）
可写性（writable）决定了属性的值（value）是否可以被改变。
```
var o = {}; 

Object.defineProperty(o, "a", { value : 37, writable : false });

o.a // 37
o.a = 25;
o.a // 37
```
关于可写性，还有一种特殊情况。就是如果原型对象的某个属性的可写性为false，那么派生对象将无法自定义这个属性。
```
var proto = Object.defineProperty({}, 'foo', {
    value: 'a',
    writable: false
});

var o = Object.create(proto);

o.foo = 'b';
o.foo // 'a'
```
有一个规避方法，就是通过覆盖attributes对象，绕过这个限制，原因是这种情况下，原型链会被完全忽视。
```
Object.defineProperty(o, 'foo', { value: 'b' });
o.foo // 'b'
```
## 存取器（accessor）
除了直接定义以外，属性还可以用存取器（accessor）定义。其中，存值函数称为setter，使用set命令；取值函数称为getter，使用get命令。
```
var o = {
  get p() {
    return "getter";
  },
  set p(value) {
    console.log("setter: "+value);
  }
}
```
利用存取器，可以实现数据对象与DOM对象的双向绑定。
```
Object.defineProperty(user, 'name', {
  get: function() {
    return document.getElementById("foo").value;
  },
  set: function(newValue) {
    document.getElementById("foo").value = newValue;
  },
  configurable: true
});
```
## 控制对象状态
JavaScript提供了三种方法，精确控制一个对象的读写状态，防止对象被改变。最弱一层的保护是preventExtensions，其次是seal，最强的freeze。
### Object.preventExtensions方法
Object.preventExtensions方法可以使得一个对象无法再添加新的属性。
```
var o = new Object();

Object.preventExtensions(o);

Object.defineProperty(o, "p", { value: "hello" });
// TypeError: Cannot define property:p, object is not extensible.

o.p = 1;
o.p // undefined
```
不过，对于使用了preventExtensions方法的对象，可以用delete命令删除它的现有属性。
### Object.isExtensible方法
Object.isExtensible方法用于检查一个对象是否使用了preventExtensions方法。也就是说，该方法可以用来检查是否可以为一个对象添加属性。
```
var o = new Object();

Object.isExtensible(o)
// true

Object.preventExtensions(o);
Object.isExtensible(o)
// false
```
### Object.seal方法
Object.seal方法使得一个对象既无法添加新属性，也无法删除旧属性。
```
var o = { p:"hello" };

Object.seal(o);

delete o.p;
o.p // "hello"

o.x = 'world';
o.x // undefined
```
Object.seal还把现有属性的attributes对象的configurable属性设为false，使得attributes对象不再能改变。
```
var o = { p: 'a' };

// seal方法之前
Object.getOwnPropertyDescriptor(o, 'p')
// Object {value: "a", writable: true, enumerable: true, configurable: true}

Object.seal(o);

// seal方法之后
Object.getOwnPropertyDescriptor(o, 'p') 
// Object {value: "a", writable: true, enumerable: true, configurable: false}

Object.defineProperty(o, 'p', { enumerable: false })
// TypeError: Cannot redefine property: p
```
可写性（writable）有点特别。如果writable为false，使用Object.seal方法以后，将无法将其变成true；但是，如果writable为true，依然可以将其变成false。

至于属性对象的value是否可改变，是由writable决定的。
```
var o = { p: 'a' };
Object.seal(o);
o.p = 'b';
o.p // 'b'
```
### Object.isSealed方法
Object.isSealed方法用于检查一个对象是否使用了Object.seal方法。
```
var o = { p: 'a' };

Object.seal(o);
Object.isSealed(o) // true
```
### Object.freeze方法
Object.freeze方法可以使得一个对象无法添加新属性、无法删除旧属性、也无法改变属性的值，使得这个对象实际上变成了常量。
```
var o = {p:"hello"};

Object.freeze(o);

o.p = "world";
o.p // hello

o.t = "hello";
o.t // undefined
```
### Object.isFrozen方法
Object.isFrozen方法用于检查一个对象是否使用了Object.freeze()方法。
```
var o = {p:"hello"};

Object.freeze(o);
Object.isFrozen(o) // true
```
## 注意
需要注意的是，使用上面这些方法锁定对象的可写性，但是依然可以通过改变该对象的原型对象，来为它增加属性。
```
var o = new Object();

Object.preventExtensions(o);

var proto = Object.getPrototypeOf(o);

proto.t = "hello";

o.t
// hello
```
一种解决方案是，把原型也冻结住。
```
var o = Object.seal(
            Object.create(Object.freeze({x:1}),
                {y: {value: 2, writable: true}})
);

Object.getPrototypeOf(o).t = "hello";
o.hello // undefined
```
