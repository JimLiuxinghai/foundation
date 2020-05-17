## Call & apply & bind

### call

> call() 方法在使用一个指定的 this 值和若干个指定的参数值的前提下调用某个函数或方法。

注意：

* 1.call改变了this的指向
* 2.执行绑定函数
* 3.注意参数length不确定
* 4.this参数可以传null，当为null的时候，视为指向window
* 5.函数是可以有返回值得

```javascript
Function.prototype.call2 = function (context) {
    context = context || window;
    context.fn = this;
    let args = [];
    for(let i = 1; i < arguments.length; i ++) {
        args.push(arguments[i])
    }
    var result = context.fn(...args);
    delete context.fn;
    return result
}
```

### apply

根据call的实现，可以很轻松实现一个apply

```javascript
Function.prototype.apply2 =  function (context, arr) {
    context = Object(context) || window;
    context.fn = this;
    if(!arr) {
        result = context.fn();
    }
    else {
        result = context.fn(...arr)
    }
    delete context.fn
    return result;
}
```

### bind

> bind()方法会创建一个新的函数。当这个新函数被调用时，bind()的第一个参数将作为它运行时的this，之后的一序列参数将会在传递的实参前传入作为它的参数。

注意：

* 返回一个函数

* 可以传入参数

* 参数可以定义时传递，也可在使用时传递

* 调用方非函数报错

* 一个绑定函数也能使用new操作符创建对象：这种行为就把原函数当做构造器。提供的this值被忽略，同时调用时的参数被提供给模拟函数

  举个例子：

  ```javascript
  var value = 2;
  
  var foo = {
      value: 1
  };
  
  function bar(name, age) {
      this.habit = 'shopping';
      console.log(this.value);
      console.log(name);
      console.log(age);
  }
  
  bar.prototype.friend = 'kevin';
  
  var bindFoo = bar.bind(foo, 'daisy');
  
  var obj = new bindFoo('18');
  // undefined
  // daisy
  // 18
  console.log(obj.habit);
  console.log(obj.friend);
  // shopping
  // kevin
  ```

  尽管在全局和foo中都声明了value值，最后依然返回了undifined，说明绑定的this失效了

  
  
  不考虑构造函数：
  
  ```javascript
  Function.prototype.bind2 = function (context) {
  
      var self = this;
      // 获取bind2函数从第二个参数到最后一个参数
      var args = Array.prototype.slice.call(arguments, 1);
  
      return function () {
          // 这个时候的arguments是指bind返回的函数传入的参数
          var bindArgs = Array.prototype.slice.call(arguments);
          return self.apply(context, args.concat(bindArgs));
      }
  
  }
  ```
  
  
  
考虑构造函数
  
  > 一个绑定函数也能使用new操作符创建对象：这种行为就像把原函数当成构造器。提供的 this 值被忽略，同时调用时的参数被提供给模拟函数。
  
  也就是说当 bind 返回的函数作为构造函数的时候，bind 时指定的 this 值会失效，但传入的参数依然生效。举个例子：
  
  ```javascript
  var value = 2;
  
  var foo = {
      value: 1
  };
  
  function bar(name, age) {
      this.habit = 'shopping';
      console.log(this.value);
      console.log(name);
      console.log(age);
  }
  
  bar.prototype.friend = 'kevin';
  
  var bindFoo = bar.bind(foo, 'daisy');
  
  var obj = new bindFoo('18');
  // undefined
  // daisy
  // 18
  console.log(obj.habit);
  console.log(obj.friend);
  // shopping
  // kevin
  ```
  
  
  
  实现：
  
  ```javascript
  
  Function.prototype.bind2 = function (context) {
      var self = this;
      var args = Array.prototype.slice.call(arguments, 1);
  
      var fBound = function () {
          var bindArgs = Array.prototype.slice.call(arguments);
          // 当作为构造函数时，this 指向实例，此时结果为 true，将绑定函数的 this 指向该实例，可以让实例获得来自绑定函数的值
          // 以上面的是 demo 为例，如果改成 `this instanceof fBound ? null : context`，实例只是一个空对象，将 null 改成 this ，实例会具有 habit 属性
          // 当作为普通函数时，this 指向 window，此时结果为 false，将绑定函数的 this 指向 context
          return self.apply(this instanceof fBound ? this : context, args.concat(bindArgs));
      }
      // 修改返回函数的 prototype 为绑定函数的 prototype，实例就可以继承绑定函数的原型中的值
      fBound.prototype = this.prototype;
      return fBound;
  }
  ```
  
  优化：
  
  ```javascript
  Function.prototype.bind2 = function (context) {
  
      if (typeof this !== "function") {
        throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
      }
  
      var self = this;
      var args = Array.prototype.slice.call(arguments, 1);
      var fNOP = function () {};
  
      var fbound = function () {
          self.apply(this instanceof self ? this : context, args.concat(Array.prototype.slice.call(arguments)));
      }
  
      fNOP.prototype = this.prototype;
      fbound.prototype = new fNOP();
  
      return fbound;
  
  }
  ```
  
  



