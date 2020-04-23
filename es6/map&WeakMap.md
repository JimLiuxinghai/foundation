## Map与WeakMap

### 定于

**类似于对象，也是键值对的集合**

与对象（Object）的区别：

* 对象的键只能是字符串，Map的键可以是任意类型

#### Map的实例属性和方法

* size：返回Map结构的元素总数
* set(key, value)：向Map中加入或更新键值对
* get(key)：读取key对用的值，如果没有，返回undifind
* has(key)：某个键是否在Map对象中，在返回true否则返回false
* delete(key)：删除某个键，返回true，如果删除失败返回false
* clear()：删除所有元素
* keys()：返回键名的遍历器
* values()：返回键值的遍历器
* entries()：返回所有成员的遍历器
* forEach()：遍历Map所有成员

#### Map的基本用法

```javascript
const s = new Map();
m.set('a': 1);
m.set(2: 'a');
m.get('a') //1
m.has(2) //true
m.size() //2
```



```javascript
const map = new Map([
  ['F', 'no'],
  ['T',  'yes'],
])

for (let key of map.keys()) {
  console.log(key)
}
// "F"
// "T"

for (let value of map.values()) {
  console.log(value)
}
// "no"
// "yes"

for (let [key, value] of map) {
  console.log(key, value)
}
// "F" "no"
// "T" "yes"

[...map]  // [['F', 'no'], ['T', 'yes']]

```

### WeakMap

#### 定义

**WeakMap结构与Map结构类似，也是用于生成键值对的集合**

WeakMap与Map的区别：

* WeakMap只接受对象作为键名（不包括null）
* WeakMap键名所指向的对象，不计入垃圾回收机制（类似于WeakSet）
* WeakMap不可遍历