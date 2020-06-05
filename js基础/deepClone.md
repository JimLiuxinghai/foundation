##  深拷贝

### 深拷贝 VS 浅拷贝

深拷贝和浅拷贝都是针对的引用类型，JS中的变量类型分为值类型（基本类型）和引用类型；对值类型进行复制操作会对值进行一份拷贝，而对引用类型赋值，则会进行地址的拷贝，最终两个变量指向同一份数据。

根据拷贝的层级不同可以分为浅拷贝和深拷贝，浅拷贝就是只进行一层拷贝，深拷贝就是无限层级拷贝。

```javascript
var a1 = {b: {c: {}};

var a2 = shallowClone(a1); // 浅拷贝
a2.b.c === a1.b.c // true

var a3 = clone(a1); // 深拷贝
a3.b.c === a1.b.c // false

function shallowClone(source) {
    var target = {};
    for(var i in source) {
        if (source.hasOwnProperty(i)) {
            target[i] = source[i];
        }
    }

    return target;
}
```

### 1.简单的深拷贝

```javascript
function clone(source) {
    var target = {};
    for(var i in source) {
        if (source.hasOwnProperty(i)) {
            if (typeof source[i] === 'object') {
                target[i] = clone(source[i]); // 注意这里
            } else {
                target[i] = source[i];
            }
        }
    }

    return target;
}
```

问题：

- 没有对参数做检验
- 判断是否对象的逻辑不够严谨
- 没有考虑数组的兼容

```javascript
function isObject(x) {
    return Object.prototype.toString.call(x) === '[object Object]';
}
```

其实递归方法最大的问题在于**爆栈**，当数据的层次很深是就会栈溢出

```javascript
function createData(deep, breadth) {
    var data = {};
    var temp = data;

    for (var i = 0; i < deep; i++) {
        temp = temp['data'] = {};
        for (var j = 0; j < breadth; j++) {
            temp[j] = j;
        }
    }

    return data;
}

createData(1, 3); // 1层深度，每层有3个数据 {data: {0: 0, 1: 1, 2: 2}}
createData(3, 0); // 3层深度，每层有0个数据 {data: {data: {data: {}}}}
```

当clone层级很深的话就会栈溢出，但数据的广度不会造成溢出

```javascript
clone(createData(1000)); // ok
clone(createData(10000)); // Maximum call stack size exceeded

clone(createData(10, 100000)); // ok 广度不会溢出
```

其实大部分情况下不会出现这么深层级的数据，但这种方式还有一个致命的问题，就是循环引用，举个例子

```javascript
var a = {};
a.a = a;

clone(a) // Maximum call stack size exceeded 直接死循环了有没有，/(ㄒoㄒ)/~~
```

关于循环引用的问题解决思路有两种，一直是循环检测，一种是暴力破解。

### 2.JSON

```javascript
function cloneJSON(source) {
    return JSON.parse(JSON.stringify(source));
}
```

下面来测试下cloneJSON有没有溢出的问题，看起来cloneJSON内部也是使用递归的方式

```javascript
cloneJSON(createData(10000)); // Maximum call stack size exceeded
```

既然是用了递归，那循环引用呢？并没有因为死循环而导致栈溢出啊，原来是JSON.stringify内部做了循环引用的检测，正是我们上面提到破解循环引用的第一种方法：循环检测

```javascript
var a = {};
a.a = a;

cloneJSON(a) // Uncaught TypeError: Converting circular structure to JSON
```

### 3.破解递归爆栈

其实破解递归爆栈的方法有两条路，第一种是消除尾递归，但在这个例子中貌似行不通，第二种方法就是干脆不用递归，改用循环，当我提出用循环来实现时，基本上90%的前端都是写不出来的代码的，这其实让我很震惊

举个例子，假设有如下的数据结构

```javascript
var a = {
    a1: 1,
    a2: {
        b1: 1,
        b2: {
            c1: 1
        }
    }
}
```

这不就是一个树吗，其实只要把数据横过来看就非常明显了

```
    a
  /   \
 a1   a2        
 |    / \         
 1   b1 b2     
     |   |        
     1  c1
         |
         1 
```

用循环遍历一棵树，需要借助一个栈，当栈为空时就遍历完了，栈里面存储下一个需要拷贝的节点

首先我们往栈里放入种子数据，`key`用来存储放哪一个父元素的那一个子元素拷贝对象

然后遍历当前节点下的子元素，如果是对象就放到栈里，否则直接拷贝

```javascript
function cloneDeep (x) {
  const root = {};
  // 栈
  const loopList = [
    {
      parent: root,
      key: undefined,
      data: x,
    }
  ];
  while(loopList.length) {
    // 深度优先
    const node = loopList.pop();
    const parent = node.parent;
    const key = node.key;
    const data = node.data;
    // 初始化赋值目标，key为undefined则拷贝到父元素，否则拷贝到子元素
    let res = parent;
    if (typeof key !== 'undefined') {
      res = parent[key] = {};
    }
    
    for(let k in data) {
      if (data.hasOwnProperty(k)) {
        if (typeof data[k] === 'object') {
          // 下一次循环
          loopList.push({
            parent: res,
            key: k,
            data: data[k],
          });
        } else {
          res[k] = data[k];
        }
      }
    }
  }
  return root;
}
```

改用循环后，再也不会出现爆栈的问题了，但是对于循环引用依然无力应对

### 4.破解循环引用

有没有一种办法可以破解循环应用呢？别着急，我们先来看另一个问题，上面的三种方法都存在的一个问题就是引用丢失，这在某些情况下也许是不能接受的

举个例子，假如一个对象a，a下面的两个键值都引用同一个对象b，经过深拷贝后，a的两个键值会丢失引用关系，从而变成两个不同的对象，o(╯□╰)o

```javascript
var b = {};
var a = {a1: b, a2: b};

a.a1 === a.a2 // true

var c = clone(a);
c.a1 === c.a2 // false
```

如果我们发现个新对象就把这个对象和他的拷贝存下来，每次拷贝对象前，都先看一下这个对象是不是已经拷贝过了，如果拷贝过了，就不需要拷贝了，直接用原来的，这样我们就能够保留引用关系了，✧(≖ ◡ ≖✿)嘿嘿

但是代码怎么写呢，o(╯□╰)o，别急往下看，其实和循环的代码大体一样，不一样的地方我用`// ==========`标注出来了

引入一个数组`uniqueList`用来存储已经拷贝的数组，每次循环遍历时，先判断对象是否在`uniqueList`中了，如果在的话就不执行拷贝逻辑了

`find`是遍历`uniqueList`的方法。

```javascript
// 保持引用关系
function cloneForce(x) {
    // =============
    const uniqueList = []; // 用来去重
    // =============

    let root = {};

    // 循环数组
    const loopList = [
        {
            parent: root,
            key: undefined,
            data: x,
        }
    ];

    while(loopList.length) {
        // 深度优先
        const node = loopList.pop();
        const parent = node.parent;
        const key = node.key;
        const data = node.data;

        // 初始化赋值目标，key为undefined则拷贝到父元素，否则拷贝到子元素
        let res = parent;
        if (typeof key !== 'undefined') {
            res = parent[key] = {};
        }
        
        // =============
        // 数据已经存在
        let uniqueData = find(uniqueList, data);
        if (uniqueData) {
            parent[key] = uniqueData.target;
            continue; // 中断本次循环
        }

        // 数据不存在
        // 保存源数据，在拷贝数据中对应的引用
        uniqueList.push({
            source: data,
            target: res,
        });
        // =============
    
        for(let k in data) {
            if (data.hasOwnProperty(k)) {
                if (typeof data[k] === 'object') {
                    // 下一次循环
                    loopList.push({
                        parent: res,
                        key: k,
                        data: data[k],
                    });
                } else {
                    res[k] = data[k];
                }
            }
        }
    }

    return root;
}

function find(arr, item) {
    for(let i = 0; i < arr.length; i++) {
        if (arr[i].source === item) {
            return arr[i];
        }
    }

    return null;
}
```