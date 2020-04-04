## js原型与原型链

### 介绍

每一个函数都有prototype的属性

```
function Person() {

}
// prototype是函数才会有的属性
Person.prototype.name = 'jim';
var person1 = new Person();
var person2 = new Person();
console.log(person1.name) // jim
console.log(person2.name) // jim
```

函数的 prototype 属性指向了一个对象，这个对象正是调用该构造函数而创建的实例的原型，也就是这个例子中的 person1 和 person2 的原型。

那什么是原型呢？你可以这样理解：每一个JavaScript对象(null除外)在创建的时候就会与之关联另一个对象，这个对象就是我们所说的原型，每一个对象都会从原型"继承"属性。

#### constructor

每个原型都有一个 constructor 属性指向关联的构造函数。

```
function Person() {

}
console.log(Person === Person.prototype.constructor); // true
```

#### 原型链

当读取实例的属性时，如果找不到，就会查找与对象关联的原型中的属性，如果还查不到，就去找原型的原型，一直找到最顶层为止。


### 结论

1.只有函数才有prototype属性, 对象没有
![proto]('../images/js/proto.png')

2.其实原型对象就是通过 Object 构造函数生成的，结合之前所讲，实例的 __proto__ 指向构造函数的 prototype.

![proto]('../images/js/prototype5.png')
