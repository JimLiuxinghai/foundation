## 类型转换

### 1.typeof

```
console.log(typeof 'jim') // string
```
《JavaScript权威指南》中对 typeof 的介绍：

    typeof 是一元操作符，放在其单个操作数的前面，操作数可以是任意类型。返回值为表示操作数类型的一个字符串。

在ES6之前JS一共有6种数据类型： 

Undefined，Null，Boolean，Number，String，Object

但是使用typeof判断时是这样的结果：

undefined、object、boolean、number、string、object

Null和Object都返回object

而且可以检测function类型

```
function a () {}
console.log(typeof a) //function
```
但是Object的其他类型却不能被检测出来

```
var date = new Date();
var error = new Error();
console.log(typeof date); // object
console.log(typeof error); // object
```

### 2.Object.prototype.toString

[ES5规范](https://es5.github.io/#x15.2.4.2)

    When the toString method is called, the following steps are taken:

    1.If the this value is undefined, return "[object Undefined]".
    2.If the this value is null, return "[object Null]".
    3.Let O be the result of calling ToObject passing the this value as the argument.
    4.Let class be the value of the [[Class]] internal property of O.
    5.Return the String value that is the result of concatenating the three Strings "[object ", class, and "]".

当 toString 方法被调用的时候，下面的步骤会被执行：

* 1.如果 this 值是 undefined，就返回 [object Undefined]
* 2.如果 this 的值是 null，就返回 [object Null]
* 3.让 O 成为 ToObject(this) 的结果
* 4.让 class 成为 O 的内部属性 [[Class]] 的值
* 5.最后返回由 "[object " 和 class 和 "]" 三个部分组成的字符串

通过规范，我们至少知道了调用 Object.prototype.toString 会返回一个由 "[object " 和 class 和 "]" 组成的字符串，而 class 是要判断的对象的内部属性。

```
console.log(Object.prototype.toString.call(undefined)) // [object Undefined]
console.log(Object.prototype.toString.call(null)) // [object Null]

var date = new Date();
console.log(Object.prototype.toString.call(date)) // [object Date]
```
这个class值是识别类型的关键

Object.prototype.toString可以识别出14种类型

```
// 以下是11种：
var number = 1;          // [object Number]
var string = '123';      // [object String]
var boolean = true;      // [object Boolean]
var und = undefined;     // [object Undefined]
var nul = null;          // [object Null]
var obj = {a: 1}         // [object Object]
var array = [1, 2, 3];   // [object Array]
var date = new Date();   // [object Date]
var error = new Error(); // [object Error]
var reg = /a/g;          // [object RegExp]
var func = function a(){}; // [object Function]

function checkType() {
    for (var i = 0; i < arguments.length; i++) {
        console.log(Object.prototype.toString.call(arguments[i]))
    }
}

checkType(number, string, boolean, und, nul, obj, array, date, error, reg, func)
```

```
console.log(Object.prototype.toString.call(Math)); // [object Math]
console.log(Object.prototype.toString.call(JSON)); // [object JSON]
```

```
function a() {
    console.log(Object.prototype.toString.call(arguments)); // [object Arguments]
}
a();
```

### Type函数

写一个 type 函数能检测各种类型的值，如果是基本类型，就使用 typeof，引用类型就使用 toString。此外鉴于 typeof 的结果是小写，希望所有的结果都是小写。

考虑到实际情况下并不会检测 Math 和 JSON，所以去掉这两个类型的检测。

注意，在 IE6 中，null 和 undefined 会被 Object.prototype.toString 识别成 [object Object]！

```
var class2type = {};

// 生成class2type映射
"Boolean Number String Function Array Date RegExp Object Error".split(" ").map(function(item, index) {
    class2type["[object " + item + "]"] = item.toLowerCase();
})

function type(obj) {
    // 一箭双雕
    if (obj == null) {
        return obj + "";
    }
    return typeof obj === "object" || typeof obj === "function" ?
        //如果检测不到类型返回object，比如es6新增的Symbol，Set，Map
        class2type[Object.prototype.toString.call(obj)] || "object" :
        typeof obj;
}
```


