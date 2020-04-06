# 事件


## EventTarget

DOM 的事件操作（监听和触发），都定义在EventTarget接口。所有节点对象都部署了这个接口，其他一些需要事件通信的浏览器内置对象（比如，XMLHttpRequest、AudioNode、AudioContext）也部署了这个接口。

* addEventListener  //添加事件监听
* removeEventListener //移除
* dispatchEvent //触发事件（多作为触发自定义事件）

#### addEventListener

```
target.addEventListener(type, listener[, useCapture]);
```
参数：
* type 事件名称，大小写敏感。
* listener：监听函数。事件发生时，会调用该监听函数。
* useCapture：布尔值，表示监听函数是否在捕获阶段（capture）触发（参见后文《事件的传播》部分），默认为false（监听函数只在冒泡阶段被触发）。该参数可选。

listener 函数也可以是一个具有 handlerEvent的方法
```
buttonElement.addEventListener('click', {
  handleEvent: function (event) {
    console.log('click');
  }
});
```

第三个参数可以是一个对象，对象中有三个参数 默认都为false

* capture 布尔值  监听函数是否在捕获阶段触发
* once 布尔值 监听函数是否只调用一次
* passive 布尔值 监听函数不会调用preventDefault方法

addEventListener方法可以为针对当前对象的同一个事件，添加多个不同的监听函数。这些函数按照添加顺序触发，即先添加先触发。如果为同一个事件多次添加同一个监听函数，该函数只会执行一次，多余的添加将自动被去除（不必使用removeEventListener方法手动去除）

IE8及以下兼容

```
function addEventHandler(obj, eventName, handler) {
    if (document.addEventListener) {
        obj.addEventListener(eventName, handler, false);
    }
    else if (document.attachEvent) {
        obj.attachEvent("on" + eventName, handler);
    }    
    else {
        obj["on" + eventName] = handler;
    }
}
```

#### removeEventListener

参数类型与 addEventListener 相同。
当三个参数都与addEventListener相同时才可以移除。（需要注意的是，绑定事件的回调函数不能是匿名函数，必须是一个已经被声明的函数，因为解除事件绑定时需要传递这个回调函数的引用。）

IE8及一下兼容

```
function removeEventHandler(obj, eventName, handler) {
    if (document.removeEventListener) {
        obj.removeEventListener(eventName, handler, false);
    }
    else if (document.detachEvent) {
        obj,detachEvent("on" + eventName, handler);
    }
    else {
        obj["on" + eventName] = null;
    }
}
```

#### dispatchEvent

在当前节点触发指定事件，从而触发监听函数的执行。该方法返回一个布尔值，只要有一个监听函数调用了。
可以判断事件是否被销毁
```
var canceled = !cb.dispatchEvent(event);
if (canceled) {
  console.log('事件取消');
} else {
  console.log('事件未取消');
}
```
[dispathEvent应用场景](https://juejin.im/entry/5ad070b56fb9a028db591d49)


## 事件触发过程

事件流描述了页面接收事件的顺序。现代浏览器（指 IE6-IE8 除外的浏览器，包括 IE9+、FireFox、Safari、Chrome 和 Opera 等）事件流包含三个过程，分别是捕获阶段、目标阶段和冒泡阶段，下图形象地说明这个过程：

![](https://segmentfault.com/img/bVvdpE)

### 捕获阶段

当我们对 DOM 元素进行操作时，比如鼠标点击、悬浮等，就会有一个事件传输到这个 DOM 元素，这个事件从 Window 开始，依次经过 docuemnt、html、body，再不断经过子节点直到到达目标元素，从 Window 到达目标元素父节点的过程称为捕获阶段，注意此时还未到达目标节点。

### 目标阶段

捕获阶段结束时，事件到达了目标节点的父节点，最终到达目标节点，并在目标节点上触发了这个事件，这就是目标阶段。

需要注意的是，事件触发的目标节点为最底层的节点。比如下面的例子：
```
<div>
    <p>你猜，目标在这里还是<span>那里</span>。</p>
</div>
```
当我们点击“那里”的时候，目标节点是span，点击“这里”的时候，目标节点是p，而当我们点击p区域之外，div区域之内时，目标节点就是div。

### 冒泡阶段

当事件到达目标节点之后，就会沿着原路返回，这个过程有点类似水泡从水底浮出水面的过程，所以称这个过程为冒泡阶段。

针对这个过程，wilsonpage 做了一个 [DEMO](http://jsbin.com/exezex/4/edit?css,js,output)，可以非常直观地查看这个过程。

### 事件委托

上面我们讲了事件的冒泡机制，我们可以利用这一特性来提高页面性能，减少页面事件绑定，事件委托便事件冒泡是最典型的应用之一。

何谓“委托”？在现实中，当我们不想做某件事时，便“委托”给他人，让他人代为完成。JavaScript 中，事件的委托表示给元素的父级或者祖级，甚至页面，由他们来绑定事件，然后利用事件冒泡的基本原理，通过事件目标对象进行检测，然后执行相关操作。看下面例子：

```
// HTML
<ul id="list">
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
    <li>Item 4</li>
    <li>Item 5</li>
</ul>

// JavaScript
var list = document.getElementById("list");
list.addEventListener("click", function(e) {
    console.log(e.target);
});
```

* 减少事件绑定。上面的例子中，也可以分别给每个列表项绑定事件，但利用事件委托的方式不仅省去了一一绑定的麻烦，也提升了网页的性能，因为每绑定一个事件便会增加内存使用。
* 可以动态监听绑定。上面的例子中，我们对 5 个列表项进行了事件监听，当删除一个列表项时不需要单独删除这个列表项所绑定的事件，而增加一个列表项时也不需要单独为新增项绑定事件。

### 阻止事件冒泡

事件委托是事件冒泡的一个应用，但有时候我们并不希望事件冒泡。比如下面的例子：

```
const ele = document.getElementById("ele");
ele.addEventListener("click", function() {
    console.log("ele-click");
}, false);

document.addEventListener("click", function() {
    console.log("document-click");
}, false);
```
我们不希望ele的点击事件冒泡到document上
```
const ele = document.getElementById("ele");
ele.addEventListener("click", function() {
    console.log("ele-click");
}, false);

document.addEventListener("click", function() {
    console.log("document-click");
}, false);
```
我们还能用 e.cancelBubble = true 来替代 e.stopPropagation()。网上的说法是 cancelBubble 仅仅适用于 IE，而 stopPropagation 适用于其他浏览器。但根据我实验的结果，现代浏览器（IE9 及 以上、Chrome、FF 等）均同时支持这两种写法。为了保险起见，我们可以采用以下代码：
```
function preventBubble(e) {
    if (!e) {
        const e = window.event;
    }
    e.cancelBubble = true;
    if (e.stopPropagation) {
        e.stopPropagation();
    }
}
```

## 注意

1.事件执行的顺序除了和绑定的顺序有关外，还和冒泡与捕获阶段有关

2.e.stopPropagation()不只是组织事件冒泡，还组织了事件捕获，就是阻止了事件的进一步传播

参考

* [js事件机制](https://www.barretlee.com/blog/2014/02/17/cb-problem-javascript-event/)