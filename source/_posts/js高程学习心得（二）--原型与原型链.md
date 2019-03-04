title: js高程学习心得（二）--原型与原型链
date: 2017-01-12
tags: 经验

---

## new操作符

    var person = new Person();
    function New(fun) {
        var obj = {'_proto_' : fun.prototype}; //实例原型指向
        return function() {
            fun.apply(obj, auguments);  //修改this指向
            return obj;
        }
    }



![原型与原型链][1]
原型C的实例B作为原型B，原型B的实例A作为原型A

**工厂模式**

    function createPerson(name, age, job) {
        var o = new Object();
        o.name = name;
        o.age = age;
        o.job = job;
        o.sayName = function() {
            console.log(this.name);
        };
        return o;
    }
    var person = createPerson("jacy", 20, "engineer");

**构造函数模式**

    function Person(name, age, job) {
        this.name = name;
        this.age = age;
        this.job = job;
        this.sayName = function() {
            console.log(this.name);
        };
    }
    var person = new Person("jacy", 20, "engineer");
    
**原型模式**

    function Person() {};
    Person.prototype = {
        constructor : Person,
        name : "jacy",
        age : 10,
        job : "engineer"
    }; //重写原型对象，constructor重新指向Person，但是会变成可枚举属性
    var friend = new Person(); //与重写原型对象不可调换，由于实例已经指向原来的原型对象，重写原型对象不会改变实例对原来原型对象的指向
    
单一的原型模式对于引用类型数据会出问题：

    function Person() {};
    Person.prototype = {
        constructor : Person,
        friends : [1,2,3]
    }; 
    var person1 = new Person();
    var person2 = new Person();
    person1.friends.push("4");
    console.log(person2.friends); //"1,2,3,4"

**组合使用构造函数和原型模式**

    function Person(name, age, job) {
        this.friends = [1,2,3];
    }
    Person.prototype = {
        constructor : Person
    }
    var person1 = new Person("jacy", 20, "engineer");
    var person2 = new Person("Jacy", 20, "engineer");
    person1.friends.push("4");
    console.log(person2.friends); //"1,2,3"

**寄生构造函数**

    function SpecialArray() {
        var values = new Array();
        values.push.apply(values, arguments);
        values.toPipedString = function() {
            return this.join('|');
        }; //扩展Array构造函数又不直接修改Array构造函数
        return values;
    }
    var colors = new SpecialArray("red", "blue", "green");
    alert(colors.toPipedString()); //"red|blue|green"
此方法返回的对象与构造函数和构造函数原型属性间没关系

解决原型链中引用类型值带来的问题：
**借用构造函数**

    function SuperType() {
        this.colors = ["red", "blue", "green"];
    }
    function SubType() {
        SuperType.call(this);
    }
    var instance1 = new SubType();
    instance1.colors.push("black");
    alert(instance1.colors); //"red,blue,green,black"
    var instance2 = new SubType();
    alert(instance2.colors); //"red,blue,green"

**组合式继承--原型链和借用构造函数组合**

    function SuperType(name) {
        ...
    }
    SuperType.prototype.sayName = function() {
        ...
    }
    function SubType(name, age) {
        SuperType.call(this,name); //第二次调用
    }
    
    SubType.prototype = new SuperType(); //第一次调用
    SubType.prototype.constructor = Subtype;
    SubType.prototype.sayAge = function() {
        ...
    }
    
**原型式继承**

    function object(obj) {
        function fun() {};
        fun.prototype = obj;
        return new fun();
    }

ES5新增Object.create()与object()行为相同
只有一个参数时与object相同，第二个参数表示为新对象定义的额外属性对象

**寄生式继承**

    function createAnother(original) {
        var clone = object(original); //object函数非必需
        clone.sayHi = function() {
            ...
        }; //扩展对象
        return clone;
    }

**寄生组合式继承**
组合继承两次调用SuperType产生多余属性

    function inheritPrototype(subType, superType) {
        var prototype = object(superType.prototype); //不采用传统的new操作符直接用原构造函数生成，而用暂时的构造函数生成来实现继承
        prototype.constructor = subType;
        subType.prototype = prototype;
    }
    function SuperType(name) {
        ...
    }
    SuperType.prototype.sayName = function() {
        ...
    }
    function SubType(name, age) {
        SuperType.call(this,name); 
    }
    inheritPrototype(SubType, SuperType);
    SubType.prototype.sayAge = function() {
        ...
    }

  [1]: http://7xiuuj.com1.z0.glb.clouddn.com/scope.png
