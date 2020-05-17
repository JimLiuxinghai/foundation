## new

> new 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象类型之一

使用：

```javascript
// Otaku 御宅族，简称宅
function Otaku (name, age) {
    this.name = name;
    this.age = age;

    this.habit = 'Games';
}

// 因为缺乏锻炼的缘故，身体强度让人担忧
Otaku.prototype.strength = 60;

Otaku.prototype.sayYourName = function () {
    console.log('I am ' + this.name);
}

var person = new Otaku('Kevin', '18');

console.log(person.name) // Kevin
console.log(person.habit) // Games
console.log(person.strength) // 60

person.sayYourName(); // I am Kevin
```

注意：

* person可以访问到Otaku构造函数里的属性
* person可以访问到Qtaku.prototype上的属性
* 构造函数可以有返回值

实现步骤：

* 用new Object() 的方式新建了一个对象 obj
* 取出第一个参数，就是我们要传入的构造函数。此外因为 shift 会修改原数组，所以 arguments 会被去除第一个参数
* 将 obj 的原型指向构造函数，这样 obj 就可以访问到构造函数原型中的属性
* 使用 apply，改变构造函数 this 的指向到新建的对象，这样 obj 就可以访问到构造函数中的属性
* 返回 obj

```
function objectFactory() {

    var obj = new Object(),

    Constructor = [].shift.call(arguments);

    obj.__proto__ = Constructor.prototype;

    var ret = Constructor.apply(obj, arguments);

    return typeof ret === 'object' ? ret : obj;

};
```

