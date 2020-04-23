#### 回顾

ES6增加了箭头函数：

```javascript
let func = value => value;
```

相当于：

```javascript
let func = function (value) {
    return value;
};
```

如果需要给函数传入多个参数：

```javascript
let func = (value, num) => value * num;
```

如果需要返回一个对象：

```javascript
let func = (value, num) => ({total: value * num})
```

很多时候，你可能想不到要这样用，所以再来举个例子，比如在 React 与 Immutable 的技术选型中，我们处理一个事件会这样做：

```javascript
handleEvent = () => {
  this.setState({
    data: this.state.data.set("key", "value")
  })
}
```

改进为：

```javascript
handleEvent = () => {
  this.setState(({data}) => ({
    data: data.set("key", "value")
  }))
}
```



#### 比较

与普通函数的区别：

##### 1.没有this

**箭头函数没有 this，所以需要通过查找作用域链来确定 this 的值。**

这就意味着如果箭头函数被非箭头函数包含，this绑定的就是最近一层非箭头函数的this。

模拟一个实际开发中的例子：

我们的需求是点击一个按钮，改变该按钮的背景色。

为了方便开发，我们抽离一个Button组件，当需要使用的时候，直接：

```javascript
//传入元素id值即可绑定该元素点击时改变背景色的时间
new Button('button')
```

HTML代码如下：

```html
<button id="button">点击变色</button>
```

JS代码：

```javascript
function Button (id) {
  this.element = document.querySelector('#' + id);
  this.bindEvent();
}

Button.prototype.bindEvent = function () {
  this.element.addEventListener("click", this.setBgColor, false);
}

Button.prototype.setBgColor = function() {
    this.element.style.backgroundColor = '#1abc9c'
};
var button = new Button('button');
```

看着好像没有问题，结果却是报错 `Uncaught TypeError: Cannot read property 'style' of undefined`

这是因为当使用addEventListener()为一个元素注册事件的时候，事件函数里的this值是该元素的引用。

所以如果我们在 setBgColor 中 `console.log(this)`，this 指向的是按钮元素，那 this.element 就是 undefined，报错自然就理所当然了。

也许你会问，既然 this 都指向了按钮元素，那我们直接修改 setBgColor 函数为：

```javascript
Button.prototype.setBgColor = function () {
  this.style.backgroundColor = '#1abc9c';
}
```

这样确实可以解决这个问题

但是实际开发中，我们可能会在setBgColor中调用其他函数，比如写成:

```javascript
Button.prototype.setBgColor = function() {
    this.setElementColor();
    this.setOtherElementColor();
};
```

所以我们希望setBgColor中的this还是指向实例对象的，这样就可以调用其他的函数。

利用es5 我们可以使用bind

```javascript
Button.prototype.bindEvent = function() {
    this.element.addEventListener("click", this.setBgColor.bind(this), false);
};
```

为避免 addEventListener的影响，使用bind强制绑定setBgColor()的this为实例对象

使用ES6，我们可以更好的解决这个问题：

```javascript
Button.prototype.bindEvent = function() {
    this.element.addEventListener("click", event => this.setBgColor(event), false);
};
```

由于箭头函数没有this，所以会向外层查找this的值，即bindEvent中的this，此时this指向实例对象，所以可以正确的调用this.setBgColor方法，而this.setBgColor中的this也会正确的指向实例对象。

在这里再额外提一点，就是注意 bindEvent 和 setBgColor 在这里使用的是普通函数的形式，而非箭头函数，如果我们改成箭头函数，会导致函数里的 this 指向 window 对象。

最后因为箭头函数没有this，所以也不能用call()，apply()，bind() 这些方法改变this的指向。

##### 2.没有arguments

箭头函数没有自己的arguments对象，使箭头函数可以访问外围函数的arguments对象：

```javascript
function constant() {
    return () => arguments[0]
}

var result = constant(1);
```

##### 3.不能通过new关键字调用

JavaScript函数中有两个内部方法[[Call]]和[[Construct]]。

当通过new调用函数时，执行[[Construct]]方法，创建一个实例对象，然后再执行函数体，将this绑定到实例上。

当直接调用的时候，执行[[Call]]方法，直接执行函数体。

箭头函数并没有[[Construct]]方法，不能被用作构造函数，如果通过new方式调用，会报错。

```javascript
var Foo = () => {};
var foo = new Foo(); // TypeError: Foo is not a constructor
```

##### 4.没有new.target

因为不能使用new调用，所以也没有new.target值。可以参考[阮老师的文章](http://es6.ruanyifeng.com/#docs/class#new-target-属性)

##### 5.没有原型

由于不能使用new调用箭头函数，所以也没有构建原型的需求，于是箭头函数也不存在prototype这个属性。

```
var Foo = () => {};
console.log(Foo.prototype); // undefined
```

##### 6.没有super

连原型都没有，自然也不能通过 super 来访问原型的属性，所以箭头函数也是没有 super 的，不过跟 this、arguments、new.target 一样，这些值由外围最近一层非箭头函数决定。

#### 总结

最后，关于箭头函数，引用MDN的介绍就是：

箭头函数表达式的语法比函数表达式更短，并且不绑定自己的this，arguments，super或 new.target。这些函数表达式最适合用于非方法函数(non-method functions)，并且它们不能用作构造函数。

那么什么是 non-method functions 呢？

我们先来看看 method 的定义：

> A method is a function which is a property of an object.

对象属性中的函数就被称之为 method，那么 non-mehtod 就是指不被用作对象属性中的函数了，可是为什么说箭头函数更适合 non-method 呢？

让我们来看一个例子就明白了：

```javascript
var obj = {
  i: 10,
  b: () => console.log(this.i, this),
  c: function() {
    console.log( this.i, this)
  }
}
obj.b();
// undefined Window
obj.c();
// 10, Object {...}
```

