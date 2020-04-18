## 节流与防抖

### 防抖

原理：你尽管触发事件，我只在事件触发后n秒执行，如果你在一个事件触发的 n 秒内又触发了这个事件，那我就以新的事件的时间为准，n 秒后才执行，总之，就是要等你触发完事件 n 秒内不再触发事件，我才执行。

```
 <div id="container"></div>
```
```
var count = 1;
var container = document.getElementById('container');

function getUserAction(e) {
    console.log(e)
    container.innerHTML = count++;
};

container.onmousemove = getUserAction;
```
当鼠标滑过会不停的执行 getUserAction,我们给getUserAction添加防抖。

#### 第一版代码

```
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

#### 第二版代码

```
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

#### 第三版代码

```
function debounce (func, wait) {
    var timeout;
    return function () {
        var that = this;
        clearTimeout(timeout);
        timeout = setTimeout(function () {}, wait) {
            
        }
    }
}
```