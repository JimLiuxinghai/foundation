## 节流与防抖

防抖是虽然事件持续触发，但只有等事件停止触发后 n 秒才执行函数，节流是持续触发的时候，每 n 秒执行一次函数

### 防抖

原理：你尽管触发事件，我只在事件触发后n秒执行，如果你在一个事件触发的 n 秒内又触发了这个事件，那我就以新的事件的时间为准，n 秒后才执行，总之，就是要等你触发完事件 n 秒内不再触发事件，我才执行。

```
 <div id="container"></div>
```
```javascript
var count = 1;
var container = document.getElementById('container');

function getUserAction(e) {
    console.log(e)
    container.innerHTML = count++;
};

container.onmousemove = getUserAction;
```
当鼠标滑过会不停的执行 getUserAction,我们给getUserAction添加防抖。



```javascript
function debounce(func, wait) {
    var timeout;
    return function () {
        clearTimeout(timeout)
        timeout = setTimeout(func, wait);
    }
}
```

```
container.onmousemove = debounce(getUserAction, 1000);
```

##### this

当我们不使用debounce时，this的值为
```
<div id="container"></div>
```
使用后 this的值为window，我们需要将tihs指向正确的对象



```javascript
function debounce (func, wait) {
    var timeout;
    return function() {
        var that = this;
        clearTimout(timeout);
        timeout = setTimout(function() {
            func.apply(that);
        }, wait)
    }
}
```

##### Event 对象

如果我们不使用debounce时

```
function getUserAction(e) {
    console.log(e);
    container.innerHTML = count++;
};
```
e会是MouseEvent
但是我们使用时，e是undefined



```javascript
function debounce(func, wait) {
    var timeout;

    return function () {
        var context = this;
        var args = arguments;

        clearTimeout(timeout)
        timeout = setTimeout(function(){
            func.apply(context, args)
        }, wait);
    }
}
```

解决了this与Event对象的问题

##### 立即执行


需要先立即执行一次方法，然后等待n秒后执行,加一个immediate参数判断是否需要立即执行


```
function debounce (func, wait, immediate) {
    var timeout;
    return function () {
        var that = this;
        var args = argumets;
        if(timeout) clearTimoute(timeout);
        if(immediate) {
            var carllNow = !timeout;
            timeout = setTimeout(function(){
                timeout = null;
            }, wait)
            if (callNow) func.apply(that, args)
        }
        else {
            timeout = setTimeout(function(){
                func.apply(that, args);
            }, wait)
        }
    }
}
```

### 节流

原理：如果你持续触发事件，每隔一段时间，只执行一次事件。

根据首次是否执行以及结束后是否执行，效果有所不同，实现的方式也有所不同。
我们用 leading 代表首次是否执行，trailing 代表结束后是否再执行一次。

关于节流的实现，有两种主流的实现方式，一种是使用时间戳，一种是设置定时器。

### 使用时间戳

当触发事件的时候，我们取出当前的时间戳，然后减去之前的时间戳(最一开始值设为 0 )，如果大于设置的时间周期，就执行函数，然后更新时间戳为当前的时间戳，如果小于，就不执行。

```
function throttle (func, wait) {
    var that, args;
    var previous = 0;
    return function () {
        var now = + new Date();
        that = this;
        args = arguments;
        if(now - previos > wait) {
            func.apply(that, args);
            previous = now;
        }
    }

```

```
container.onmousemove = throttle(getUserAction, 1000);
```

### 使用定时器
当触发事件的时候，我们设置一个定时器，再触发事件的时候，如果定时器存在，就不执行，直到定时器执行，然后执行函数，清空定时器，这样就可以设置下个定时器。

```
function throttle(func, wait) {
    var timeout, that;
    var previous = 0;

    return function() {
        that = this;
        args = arguments;
        if (!timeout) {
            timeout = setTimeout(function(){
                timeout = null;
                func.apply(that, args)
            }, wait)
        }

    }
}
```

比较以上两种方式：
    1.第一种事件会立刻执行，第二种事件会在n秒后执行；
    2.第一种事件停止触发后没有办法再执行事件，第二种事件停止触发后依然会再执行一次事件

结合以上两种模式

```
/**
    options = {
        leading：false 表示禁用第一次执行
        trailing: false 表示禁用停止触发的回调
    }
*/
function throttle(func, wait, options) {
    var timeout, that, args, result;
    var previous = 0;
    if (!options) options = {};

    var later = function() {
        previous = options.leading === false ? 0 : new Date().getTime();
        timeout = null;
        func.apply(that, args);
        if (!timeout) that = args = null;
    };

    var throttled = function() {
        var now = new Date().getTime();
        if (!previous && options.leading === false) previous = now;
        var remaining = wait - (now - previous);
        that = this;
        args = arguments;
        if (remaining <= 0 || remaining > wait) {
            if (timeout) {
                clearTimeout(timeout);
                timeout = null;
            }
            previous = now;
            func.apply(that, args);
            if (!timeout) that = args = null;
        } else if (!timeout && options.trailing !== false) {
            timeout = setTimeout(later, remaining);
        }
    };
    return throttled;
}
```