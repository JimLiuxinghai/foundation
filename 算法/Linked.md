## 链表

除了对数据的随机访问，链表几乎可以用在任何可以使用一维数组的地方

链表是由一系列的节点组成的集合，每个节点都使用一个对象的引用指向它的后继，指向另一个节点的引用叫链。

### 单链表

#### 单链表操作

因为单链表包含节点，这两者的构造函数可以是两个独立的构造函数，所以我们需要些构造函数：`Node` 和 `SinglyList`

### **Node**

- `data` 存储数据
- `next` 指向链表中下一个节点的指针

### LList

- `_length` 用于表示链表中的节点数量
- `head` 分配一个节点作为链表的头
- `insert(value)` 向链表中添加一个节点
- `find(position)` 找到在列表中指定位置 n 上的节点
- `remove(position)` 删除指定位置的节点

#### 单链表操作

在实现时，我们首先定义一个名为`Node`的构造函数，然后定义一个名为`SinglyList`的构造函数。

`Node` 的每个实例都应该能够存储数据并且能够指向另外一个节点。 要实现此功能，我们将分别创建两个属性：`data`和`next`。

**读取**

单向链表（单链表）是链表的一种，其特点是链表的链接方向是单向的，对链表的访问要通过顺序读取从头部开始。

![read](../images/algorithm/linked1.png)

**插入**

向单向链表插入一个节点，需要修改它前面的节点（前驱）使其指向新加入的节点，而新加入的节点则指向原来前驱指向的节点

![add](../images/algorithm/linked2.png)

```javascript
/**
 * 一个简单的单向链表
 * @constructor
 */
// 节点
function Node(element) {
  this.element = element;   //当前节点的元素
  this.next = null;         //下一个节点链接
}
//链表类
function LList () {
  this.head = new Node( 'head' ); //头节点
  this.find = find;               //查找节点
  this.insert = insert;           //插入节点
  this.remove = remove;           //删除节点
  this.findPrev = findPrev;       //查找前一个节点
  this.display = display;         //显示链表
}
//查找节点
function find(item) {
  var currNode = this.head
  while(currNode.element !== item) {
    currNode = currNode.next
  }
  return currNode
}
//插入节点
function insert(newElement, item) {
  var newNode = new Node(newElement)
  var currNode = this.find(item)
  newNode.next = currNode.next
  currNode.next = newNode
}
//删除节点
function remove(item) {
  var prevNode = this.findPrev(item)
  var currNode = this.find(item)
  if(prevNode.next !== null) {
    prevNode.next = prevNode.next.next
    currNode.next = null
  }

}
//查找前一个节点
function findPrev(item) {
  var currNode = this.head
  while(currNode.next !== null && currNode.next.element !== item) {
    currNode = currNode.next
  }
  return currNode
}
//显示链表
function display() {
  var currNode = this.head
  while(currNode.next !== null) {
    currNode = currNode.next
    console.log(currNode.element)
  }
}
var llist = new LList()
llist.insert('小黑', 'head')
llist.insert('小白', '小黑')
llist.insert('小紫', '小白')
llist.insert('小蓝', '小紫')
llist.display()
llist.remove('小白')
llist.display()
```

### 双向链表

双向链表和普通链表的区别在于，在链表中，一个节点只有链向下一个节点的链接，而在双向链表中，链接是双向的：一个链向下一个元素，另一个链向前一个元素。 
双向链表提供了两种迭代列表的方法：从头到尾，或者反过来。我们也可以访问一个特定节点的下一个或前一个元素。在单向链表中，如果迭代列表时错过了要找的元素，就需要回到列表起点，重新开始迭代。这是双向链表的一个优点。

![linked4](../images/algorithm/linked4.png)

**插入**

双向链表的插入方法与单链表相似，但需要设置新节点的 previous 属性，尾节点插入，需将其的previous指向其前驱，其next指向它的前驱的next，其前驱的next指向本身，普通节点的插入，多了一步，将其后继的previous指向其本身

![link5](../images/algorithm/link5.png)

**删除**

从双向链表删除一个元素，分尾元素，和普通元素，尾元素删除，需将其的previous指向null，和其前驱的next指向null，普通元素删除，和单向链表没有什么区别，只需将其前驱的next指向它的后继，将其后继的previous指向它的前驱，需要注意的一点是需将删除元素的next、previous都指向null

![](../images/algorithm/link6.png)

```javascript
  /**
     * 一个简单的双向链表
     * @constructor
     */
    // 节点
    function Node(element) {
      this.element = element;   //当前节点的元素
      this.next = null;         //下一个节点链接
      this.previous = null;         //上一个节点链接
    }

//链表类
    function LList() {
      this.head = new Node('head');     //头节点
      this.find = find;                   //查找节点
      this.insert = insert;              //插入节点
      this.remove = remove;          //删除节点
      this.display = display;             //显示链表
      this.lastFind = lastFind;         //查找尾节点
      this.displReverse = displReverse;                   //反序
    }

    //查找节点
    function find(item) {
      var currNode = this.head
      while (currNode.element !== item) {
        currNode = currNode.next
      }
      return currNode
    }

    //插入节点
    function insert(newElement, item) {
      var newNode = new Node(newElement)
      var currNode = this.find(item)
      newNode.next = currNode.next
      newNode.previous = currNode
      currNode.next = newNode
      if (newNode.next !== null) {
        newNode.next.previous = newNode
      }
    }

    //删除节点
    function remove(item) {
      var currNode = this.find(item)
      if (currNode.next !== null) {
        currNode.previous.next = currNode.next
        currNode.next.previous = currNode.previous
        currNode.next = null
        currNode.previous = null
        console.log(currNode)
      } else {
        currNode.previous.next = null
        currNode.previous = null
      }
    }

    //显示链表
    function display() {
      var currNode = this.head
      while (currNode.next !== null) {
        currNode = currNode.next
        console.log(currNode.element)
      }
    }

    //查找尾节点
    function lastFind() {
      var currNode = this.head
      while (currNode.next !== null) {
        currNode = currNode.next
      }
      return currNode
    }

    //显示反序
    function displReverse() {
      var currNode = this.lastFind()
      while (currNode.previous !== null) {
        console.log(currNode.element)
        currNode = currNode.previous
      }
    }

    var llist = new LList()
    llist.insert('小黑', 'head')
    llist.insert('小白', '小黑')
    llist.insert('小紫', '小白')
    llist.insert('小蓝', '小紫')
    llist.display()
    llist.remove('小白')
    llist.display()
    console.log('========反序=========')
    llist.displReverse()
```

