## Set与WeakSet

### Set

#### 定义：

**Set在其他语言中叫集合，是由一堆无序的、有关联的、且不重复的内存结构组成的组合**。

根据定义可以得出：

* 集合中的元素无序且唯一
* 集合中的元素可以是任意类型，无论是原始值还是引用类型

#### Set的实例属性和方法

* size：返回集合中所包含元素的数量
* add(value)：向集合中添加一个新的值
* delete(value)：从集合中删除一个值
* has(value)：如果值在集合中存在，返回true，不存在返回false
* clear()：移除集合中所有的元素
* keys()：返回键名的遍历器
* values()：返回值的遍历器(由于 Set 没有键名只有键值，所有 keys 和 values 效果一致)
* entries()：返回键值对的遍历器
* forEach(): 使用回调函数遍历每个成员

#### Set的基本用法

Set本身是一个构造函数，用来生成Set结构

```javascript
const s = new Set();
[1,2,3,4,4].forEach(i => s.add(i))

for(let i of s) {
  console.log(i)
}

// output: 1 2 3 4
// Set不会添加重复元素
```

上面代码可以看出 Set 用途之一是可以去除数组中的重复元素

Set 构造函数还可以接受一个数组（或具有 iterable 接口的其他数据结构）作为参数，用来初始化一个集合。

```javascript
// 例 1
const s = new Set([1,2,3,4,5,5,4,'5'])

[...s]  // [1,2,3,4,5,'5']  
// 注意 向 Set 中加入值的时候不会发生类型转换, Set内部用 （===） 来判断两个值是否相等

// 例 2

const m = new Map()
m.set('a', 1)
m.set('a', 2)

const s = new Set(m)

[...s] // [['a', 1], ['b', 2]]

Array.from(s) // [['a', 1], ['b', 2]]  Array.from方法可以将 Set 转为数组
```

### WeakSet

#### 定义

**WeakSet 结构与 Set 类似，也是不重复的值的集合**

#### 与Set的区别

* WeakSet中的元素只能是对象，不能是 其他类型的值
* WeakSet中的值都是弱引用，即垃圾回收机制不考虑WeakSet对于对象的引用，如果该对象不再被其他变量引用，则垃圾回收机制就会自动收回该对象所占用的内存，所以只要 WeakSet 成员对象在外部消失，它们在 WeakSet 里面的引用就会自动消失。
* 由于 WeakSet 内部有多少个成员，取决于垃圾回收机制有没有运行，运行前后很可能成员个数是不一样的，而垃圾回收机制何时运行是不可预测的，因此 ES6 规定 WeakSet 不可遍历。

