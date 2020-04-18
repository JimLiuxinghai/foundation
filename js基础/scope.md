# JS作用域

## 作用域介绍

作用域是指程序源代码中定义变量的区域。
作用域规定了如何查找变量，也就是当前执行环境对变量的访问权限。
Javascript采用词法作用域(lexical scoping)，也就是静态作用域。

* 词法作用域(静态作用域)，函数的作用域在函数定义时就决定了。
```
var value = 1;

function foo() {
    console.log(value);
}

function bar() {
    var value = 2;
    foo();
}

bar();

//结果为1
```

* 动态作用域，函数作用域是在函数执行时才决定

a.bash
```
value=1
function foo () {
    echo $value;
}
function bar () {
    local value=2;
    foo;
}
bar

//结果为2
```

## 执行上下文

当 JavaScript 代码执行一段可执行代码(executable code)时，会创建对应的执行上下文(execution context)。

JavaScript 的可执行代码(executable code)的类型有哪些了？
其实很简单，就三种，全局代码、函数代码、eval代码。
每个执行上下文，都有3个属性

* 变量对象 （Variable Object，VO）
* 作用域链 (Scope chain)
* this

### 变量对象

变量对象是与执行上下文相关的数据作用域，存储了上下文中定义的变量和函数声明。

因为不同的执行上下文下的变量对象稍有不同：全局上下文和函数上下文。

#### 全局上下文

先了解全局对象：

    全局对象是预定义的对象，作为JavaScript的全局函数和全局属性的占位符，通过使用全局对象，可以访问其他所有预定义的对象、函数和属性。

    在顶层JavaScript代码中，可以使用关键字this引用全局对象。因为全局对象是作用域链的头，这以为这所有非限定性的变量和函数名都会作为改对象的属性来查询。

    例如，当JavaScript引用parseInt()函数时，它引用的市全局对象的parseInt属性。全局对象是作用域链的投，还意味着在顶层JavaScript代码中声明的所有变量都将成为全局对象的变量。

1.可以通过this引用，在客户端JavaScript中，全局对象就是window对象。

2.全局对象是由Object构造函数实例化的一个对象。

3.预定义了一对函数和属性 比如Math， Date等

4.作为全局变量的宿主。
```
var a = 1;
console.log(this.a)
```
5.客户端JavaScript中，全局对象有window属性指向自身。

总结：全局上下文中的变量对象就是全局对象。

#### 函数上下文

在函数上下文中，我们用活动对象(activation object, AO)来表示变量对象。

AO 实际上是包含了 VO 的，因为除了 VO 之外，AO 还包含函数的 parameters，以及 arguments 这个特殊对象。也就是说 AO 的确是在进入到执行阶段的时候被激活，但是激活的除了 VO 之外，还包括函数执行时传入的参数和 arguments 这个特殊对象。
AO = VO + function parameters + arguments。变量对象是规范上实现的，不可以在JavaScript环境中访问。只有当其进入执行上下文，才会被激活，变成activation object，而只有被激活的变量对象的各种属性才可以被访问。

#### 执行过程

执行上下文会被分为两个阶段处理 分析和执行

* 进入执行上下文
    进入执行上下文时，代码还没有执行。
    变量对象包括：

    1.函数的所有形参 (如果是函数上下文)
    * 由名称和对应值组成的一个变量对象的属性被创建
    * 没有实参，属性值设为 undefined

    2.函数声明
    * 由名称和对应值组成一个变量对象的属性被创建
    * 如果变量对象属性已存在，则完全覆盖（函数重载）

    3.变量声明

    * 由名称和对应值（undefined）组成一个变量对象的属性被创建；
    * 如果变量名称跟已经已经声明的形式参数或函数相同，则变量对象不会干扰已经存在的这类属性

    举个🌰

    ```
    function foo(a) {
        var b = 2;
        function c() {}
        var d = function() {};

        b = 3;

    }
    foo(1);
    ```
    进入执行上下文后

    ```
    AO = {
        arguments: {
            0: 1,
            length: 1
        },
        a: 1,
        b: undefined,
        c: reference to function c(){},
        d: undefined
    }
    ```
* 执行

    在代码执行阶段，会顺序执行代码，根据代码，修改变量对象的值

    还是上面的例子，当代码执行完后，这时候的 AO 是：
    
    ```
    AO = {
        arguments: {
            0: 1,
            length: 1
        },
        a: 1,
        b: 3,
        c: reference to function c(){},
        d: reference to FunctionExpression "d"
    }
    ```

到这里变量对象的创建过程就介绍完了，让我们简洁的总结我们上述所说：

1.全局上下文的变量对象初始化是全局对象
    
2.函数上下文的变量对象初始化只包括 Arguments 对象

3.在进入执行上下文时会给变量对象添加形参、函数声明、变量声明等初始的属性值

4.在代码执行阶段，会再次修改变量对象的属性值

注意： 进入执行上下文时，首先会处理函数声明，其次会处理变量声明，如果如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性。


## 作用域链

在执行上下文中会创建 作用域链

当查找变量时，会先从当前上下文的变量对象查找，如果没有找到，就从父级（词法层面上的父级）执行上下文的变量对象中查找，一直找到全局上下文变量对象，也就是全局对象。这样由多个执行上下文的变量对象构成的链表就叫作用域链。

从函数创建和激活两个时期讲解作用域链是如何创建和变化的：

### 函数创建
函数作用域在函数定义时就已经决定。

因为函数有一个内部属性[[Scope]]，当函数创建的时候，就会保存所有的变量对象到其中，你可以理解[[Scope]]就是所有父变量对象的层级链，但是注意：[[Scope]并不代表完整的作用域链。

```
function a() {
    function b () {
        ···
    }
}
```
函数创建是各自的[[Scope]]为：
```
a.[[Scope]] = [
    globalContext:.VO
]
b.[[Scope]] = [
    aContext.AO,
    globalContext.VO
]
```

### 函数激活

当函数激活时，进入函数上下文，创建VO/AO后，就会将活动对象添加到作用域链的前端。
这时候执行上下文的作用域链，我们命名为Scope

```
Scope = [AO].concat([[Scope]]);
```
至此，作用域链创建完毕。


## 执行上下文栈


当执行到一个函数的时候，就会进行准备工作，这里的“准备工作”，让我们用个更专业一点的说法，就叫做"执行上下文(execution context)"。

接下来问题来了，我们写的函数多了去了，如何管理创建的那么多执行上下文呢？

所以 JavaScript 引擎创建了执行上下文栈（Execution context stack，ECS）来管理执行上下文

为了模拟执行上下文栈的行为，让我们定义执行上下文栈是一个数组：
```
ECStack = [];
```

当Javascript开始执行代码的时候 最先遇到的就是全局代码，所以初始化的时候就会像执行上下文栈中压入一个全局执行上下文(globalContext)，当只有应用程序结束时，ECStack才会被清空，所以程序结束之前 栈底都有一个globalContext
```
ECStack = [
    globalContext
]
```

举个🌰
```
function fun3() {
    console.log('fun3')
}

function fun2() {
    fun3();
}

function fun1() {
    fun2();
}

fun1();
```
当遇到函数执行时，就会创建一个执行上下文，并把该执行上下文推入执行上下文栈中，当函数执行完毕，就会将函数的执行上下文从栈中弹出。

```
// 伪代码

// fun1()
ECStack.push(<fun1> functionContext);

// fun1中竟然调用了fun2，还要创建fun2的执行上下文
ECStack.push(<fun2> functionContext);

// 擦，fun2还调用了fun3！
ECStack.push(<fun3> functionContext);

// fun3执行完毕
ECStack.pop();

// fun2执行完毕
ECStack.pop();

// fun1执行完毕
ECStack.pop();

// javascript接着执行下面的代码，但是ECStack底层永远有个globalContext
```
