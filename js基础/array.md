## 数组方法

### 创建数组

#### 字面量
```
let arr = [1]
```

#### 构造器

```
let arr = new Array(1,2,3,4) //[1,2,3,4]
let arr1 = Array(1,2,3,4) //[1,2,3,4]
let arr2 = Array(3) //[,,]
```

#### ES6 Array.of 

定义：返回由所有参数值组成的数组，如果没有参数，就返回一个空数组。
目的：Array.of() 出现的目的是为了解决上述构造器因参数个数不同，导致的行为有差异的问题。

```
 let arr = Array.of(3, 11, 8); // [3,11,8]
 let arr1 = Array.of(3); // [3]
```

#### ES6 Array.from

定义：用于将两类对象转为真正的数组（不改变原对象，返回新的数组）。

参数：

第一个参数(必需):要转化为真正数组的对象。

第二个参数(可选): 类似数组的map方法，对每个元素进行处理，将处理后的值放入返回的数组。

第三个参数(可选): 用来绑定this。

```
    // 1. 对象拥有length属性
    let obj = {0: 'a', 1: 'b', 2:'c', length: 3};
    let arr = Array.from(obj); // ['a','b','c'];
    // 2. 部署了 Iterator接口的数据结构 比如:字符串、Set、NodeList对象
    let arr1 = Array.from('hello'); // ['h','e','l','l','o']
    let arr2 = Array.from(new Set(['a','b'])); // ['a','b']
```

### 数组方法

数组原型提供了非常多的方法，这里分为三类来讲，一类会改变原数组的值，一类是不会改变原数组，以及数组的遍历方法。

### 改变原数组的方法(9个)

```
    let a = [1,2,3];
    ES5:
     a.splice()/ a.sort() / a.pop()/ a.shift()/  a.push()/ a.unshift()/ a.reverse()
    ES6:
    a.copyWithin() / a.fill
```

#### splice

定义： splice() 方法向/从数组中添加/删除项目，然后返回被删除的项目

语法：array.splice(index, howmany, item1, ...itemX)

参数：

* 1.index：必需。整数，规定添加/删除项目的位置，使用负数可从数组结尾处规定位置。
* 2.howmany： 可选。要删除的项目数量。如果设置为 0，则不会删除项目。
* 3.item： 可选。向数组添加的新元素。

返回值: 如果有元素被删除,返回包含被删除项目的新数组。

eg1：删除
```
    let a = [1, 2, 3, 4, 5, 6, 7];
    let item = a.splice(0, 3); // [1,2,3]
    console.log(a); // [4,5,6,7]
    // 从数组下标0开始，删除3个元素
    let item = a.splice(-1, 3); // [7]
    // 从最后一个元素开始删除3个元素，因为最后一个元素，所以只删除了7
```

eg2：删除并添加
```
    let a = [1, 2, 3, 4, 5, 6, 7];
    let item = a.splice(0,3,'添加'); // [1,2,3]
    console.log(a); // ['添加',4,5,6,7]
    // 从数组下标0开始，删除3个元素，并添加元素'添加'
     let b = [1, 2, 3, 4, 5, 6, 7];
    let item = b.splice(-2,3,'添加1','添加2'); // [6,7]
    console.log(b); // [1,2,3,4,5,'添加1','添加2']
    // 从数组最后第二个元素开始，删除3个元素，并添加两个元素'添加1'、'添加2'

```
eg3：不删除只添加
```
    let a = [1, 2, 3, 4, 5, 6, 7];
    let item = a.splice(0,0,'添加1','添加2'); // [] 没有删除元素，返回空数组
    console.log(a); // ['添加1','添加2',1,2,3,4,5,6,7]
    let b = [1, 2, 3, 4, 5, 6, 7];
    let item = b.splice(-1,0,'添加1','添加2'); // [] 没有删除元素，返回空数组
    console.log(b); // [1,2,3,4,5,6,'添加1','添加2',7] 在最后一个元素的前面添加两个元素

```
结论：

* 1.数组如果元素不够，会删除到最后一个元素为止
* 2.操作元素包含开始的那个元素
* 3.可以添加多个元素
* 4.添加是从开始元素之前添加

#### sort

定义: sort()方法对数组元素进行排序，并返回这个数组。

参数可选: 规定排序顺序的比较函数。

默认情况下sort()方法没有传比较函数的话，默认按字母升序，如果不是元素不是字符串的话，会调用toString()方法将元素转化为字符串的Unicode(万国码)位点，然后再比较字符。

```
    // 字符串排列 看起来很正常
    var a = ["Banana", "Orange", "Apple", "Mango"];
    a.sort(); // ["Apple","Banana","Mango","Orange"]
    // 数字排序的时候 因为转换成Unicode字符串之后，有些数字会比较大会排在后面 这显然不是我们想要的
    var	a = [10, 1, 3, 20,25,8];
    console.log(a.sort()) // [1,10,20,25,3,8];

```

##### 比较函数的两个参数：

sort的比较函数有两个默认参数，要在函数中接收这两个参数，这两个参数是数组中两个要比较的元素，通常我们用 a 和 b 接收两个将要比较的元素：

* 若比较函数返回值<0，那么a将排到b的前面;
* 若比较函数返回值=0，那么a 和 b 相对位置不变；
* 若比较函数返回值>0，那么b 排在a 将的前面；

对于sort()方法更深层级的内部实现以及处理机制可以看一下这篇文章[深入了解javascript的sort方法](https://juejin.im/entry/59f7f3346fb9a04514635552)

常见排序：

1.升降序
```
 var array =  [10, 1, 3, 4,20,4,25,8];
 // 升序 a-b < 0   a将排到b的前面，按照a的大小来排序的 
 // 比如被减数a是10，减数是20  10-20 < 0   被减数a(10)在减数b(20)前面   
 array.sort(function(a,b){
   return a-b;
 });
 console.log(array); // [1,3,4,4,8,10,20,25];
 // 降序 被减数和减数调换了  20-10>0 被减数b(20)在减数a(10)的前面
 array.sort(function(a,b){
   return b-a;
 });
 console.log(array); // [25,20,10,8,4,4,3,1];

```
2.多条件排序
```
var array = [{id:10,age:2},{id:5,age:4},{id:6,age:10},{id:9,age:6},{id:2,age:8},{id:10,age:9}];
array.sort(function(a,b){
    if(a.id === b.id){// 如果id的值相等，按照age的值降序
        return b.age - a.age
    }else{ // 如果id的值不相等，按照id的值升序
        return a.id - b.id
    }
})
// [{"id":2,"age":8},{"id":5,"age":4},{"id":6,"age":10},{"id":9,"age":6},{"id":10,"age":9},{"id":10,"age":2}] 

```

#### pop()

定义：pop()方法删除一个数组中的最后一个元素，并返回这个元素。
参数：无
```
let  a =  [1,2,3];
let item = a.pop();  // 3
console.log(a); // [1,2]
```

#### shift()

定义：shift()方法删除数组的第一个元素，并返回这个元素
参数：无

```
let a = [1,2,3];
let item = a.shift(); //1
console.log(a) //[2,3]
```

#### push()

定义：push() 方法可向数组的末尾添加一个或多个元素，并返回新的长度。

参数: item1, item2, ..., itemX ,要添加到数组末尾的元素

```
let  a =  [1,2,3];
let item = a.push('末尾');  // 4
console.log(a); // [1,2,3,'末尾']
```

#### unshift()

定义：unshift() 方法可向数组的开头添加一个或多个元素，并返回新的长度。
参数：item1,item2....itemX, 要添加到数组开头的元素。

```
let  a =  [1,2,3];
let item = a.unshift('开头');  // 4
console.log(a); // ['开头',1,2,3]=
```

#### reverse()

定义：用于颠倒数组中元素的顺序
参数：无

```
let  a =  [1,2,3];
a.reverse();  
console.log(a); // [3,2,1]
```

#### ES6: copyWithin() 

定义：在当前数组内部，将指定位置的成员复制到其他位置，并返回这个数组。
语法：
```
    array.copyWithin(target, start = 0, end = this.length)
```
参数：三个参数都是数值，如果不是，会转换为数值

* 1.target （必选）：从该位置开始替换数据。如果为负值，表示倒数。
* 2.start （可选）：从该位置开始读取数据，默认为0。如果为负值，表示倒数。
* 3.end （可选）：到该位置前停止读取数据，默认等于数组长度，如为负值，表示可从数组结尾处规定位置。

```
// -2相当于3号位，-1相当于4号位
[1, 2, 3, 4, 5].copyWithin(0, -2, -1)
// [4, 2, 3, 4, 5]
var a=['OB1','Koro1','OB2','Koro2','OB3','Koro3','OB4','Koro4','OB5','Koro5']
// 2位置开始被替换,3位置开始读取要替换的 5位置前面停止替换
a.copyWithin(2,3,5)
// ["OB1","Koro1","Koro2","OB3","OB3","Koro3","OB4","Koro4","OB5","Koro5"] 

```

* 1.第一个参数开始呗替换的元素位置
* 2.要替换数据的位置范围：从第二个参数开始读取的元素，在第三个参数前面一个元素停止读取
* 3.数组的长度不会改变
* 4.读取几个元素就从开始被替换的地方替换几个元素

#### ES6: fill()

定义：使用给定值，填充一个数组。返回修改后的数组

参数：
* 1.value（必须）：要填充数组的值
* 2.start（可选）：填充的开始位置，默认值为0
* 3.end（可选）：填充的结束位置，默认值为this.length

```
[1, 2, 3].fill(4);               // [4, 4, 4]
[1, 2, 3].fill(4, 1);            // [1, 4, 4]
[1, 2, 3].fill(4, 1, 2);         // [1, 4, 3]
[1, 2, 3].fill(4, 1, 1);         // [1, 2, 3]
[1, 2, 3].fill(4, 3, 3);         // [1, 2, 3]
[1, 2, 3].fill(4, -3, -2);       // [4, 2, 3]
[1, 2, 3].fill(4, NaN, NaN);     // [1, 2, 3]
[1, 2, 3].fill(4, 3, 5);         // [1, 2, 3]
Array(3).fill(4);                // [4, 4, 4]
[].fill.call({ length: 3 }, 4);  // {0: 4, 1: 4, 2: 4, length: 3}

// Objects by reference.
var arr = Array(3).fill({}) // [{}, {}, {}];
// 需要注意如果fill的参数为引用类型，会导致都执行同一个引用类型
// 如 arr[0] === arr[1] 为true
arr[0].hi = "hi"; // [{ hi: "hi" }, { hi: "hi" }, { hi: "hi" }]
```

### 不改变原数组的方法（8个）

```
    ES5：
    slice、join、toLocateString、toStrigin、cancat、indexOf、lastIndexOf、
    ES7：
    includes
```

#### slice()

定义：方法返回一个从开始到结束（不包括结束）选择的数组的一部分浅拷贝到一个新数组对象，且原数组不会被修改。

注意：字符串也有一个slice方法是用来提取字符串的，不要搞混。

参数：
* 1.begin（可选）：索引数值，接受负值，在该索引处开始提取原数组中的元素，默认为0.
* 2.end（可选）：索引数值（不包括）接受负值，在该索引处前结束提取原数组元素，默认值为数组末尾(包括最后一个元素)。

```
let a= ['hello','world'];
let b=a.slice(0,1); // ['hello']
a[0]='改变原数组';
console.log(a,b); // ['改变原数组','world'] ['hello']
b[0]='改变拷贝的数组';
console.log(a,b); // ['改变原数组','world'] ['改变拷贝的数组']
```
如上：新数组是浅拷贝的，元素是简单数据类型，改变之后不会互相干扰。

如果是复杂数据类型(对象,数组)的话，改变其中一个，另外一个也会改变。

```
    let a= [{name:'OBKoro1'}];
    let b=a.slice();
    console.log(b,a); // [{"name":"OBKoro1"}]  [{"name":"OBKoro1"}]
    a[0].name='改变原数组';
    console.log(b,a); // [{"name":"改变原数组"}] [{"name":"改变原数组"}]
    // b[0].name='改变拷贝数组',b[0].koro='改变拷贝数组';
    // [{"name":"改变拷贝数组","koro":"改变拷贝数组"}] [{"name":"改变拷贝数组","koro":"改变拷贝数组"}]

```

原因在定义上面说过了的：slice()是浅拷贝，对于复杂的数据类型浅拷贝，拷贝的只是指向原数组的指针，所以无论改变原数组，还是浅拷贝的数组，都是改变原数组的数据。

#### join()

定义：方法用于把数组中的所有元素通过指定的分隔符放入一个字符串，返回生成的字符串。

参数：str（可选）：指定要使用的分隔符，默认为逗号。

```
let a= ['hello','world'];
let str=a.join(); // 'hello,world'
let str2=a.join('+'); // 'hello+world'
```
所以，join()/toString()方法在数组元素是数组的时候，会将里面的数组也调用join()/toString(),如果是对象的话，对象会被转为[object Object]字符串。

#### toLocakeString()

定义：返回一个标识数组元素的字符串，该字符串有数组中每个元素的toLocalString()返回值经调用join()方法连接（由逗号隔开）组成。

```
let a = [{name: 'jim'}, 23, 'abcd', new Date()];
let str = a.toLocalString(); //[object Object],23,abcd,2018/5/28 下午1:52:20 
```
如上述栗子：调用数组的toLocaleString方法，数组中的每个元素都会调用自身的toLocaleString方法，对象调用对象的toLocaleString,Date调用Date的toLocaleString。

#### toString()

定义： toString() 方法可把数组转换为由逗号链接起来的字符串。

该方法的效果和join方法一样，都是用于数组转字符串的，但是与join方法相比没有优势，也不能自定义字符串的分隔符，因此不推荐使用。

值得注意的是：当数组和字符串操作的时候，js 会调用这个方法将数组自动转换成字符串

```
   let b= [ 'toString','演示'].toString(); // toString,演示
   let a= ['调用toString','连接在我后面']+'啦啦啦'; // 调用toString,连接在我后面啦啦啦
```

#### concat()

定义： 方法用于合并两个或多个数组，返回一个新数组。

参数：

* 1.arrayX （必须）：该参数可以使具体的值，也可以是数组对象，可以是任意多个

```
    let a = [1, 2, 3];
    let b = [4, 5, 6];
    //连接两个数组
    let newVal=a.concat(b); // [1,2,3,4,5,6]
    // 连接三个数组
    let c = [7, 8, 9]
    let newVal2 = a.concat(b, c); // [1,2,3,4,5,6,7,8,9]
    // 添加元素
    let newVal3 = a.concat('添加元素',b, c,'再加一个'); 
    // [1,2,3,"添加元素",4,5,6,7,8,9,"再加一个"]
   // 合并嵌套数组  会浅拷贝嵌套数组
   let d = [1,2 ];
   let f = [3,[4]];
   let newVal4 = d.concat(f); // [1,2,3,[4]]
```

##### ES6 扩展运算符...合并数组：

因为ES6的语法更简洁易懂，所以现在合并数组我大部分采用...来处理，...运算符可以实现cancat的每个栗子，且更简洁和具有高度自定义数组元素位置的效果。

```
let a = [2, 3, 4, 5]
let b = [ 4,...a, 4, 4]
console.log(a,b); //  [2, 3, 4, 5] [4,2,3,4,5,4,4]
```

#### indexOf()

定义：返回数组中可以找到一个给定元素的第一个索引，如果不存在，返回-1

语法：
```
array.indexOf(searchElement, fromIndex);
```

参数：
* 1.searchElement（必须）：被查找的元素
* 2.fromIndex（可选）：开始查找的位置（不能大于等于数组的长度，返回-1），接受负值，默认为0.

严格相等的搜索:

数组的indexOf搜索跟字符串的indexOf不一样,数组的indexOf使用严格相等===搜索元素，即数组元素要完全匹配才能搜索成功。

注意：indexOf()不能识别NaN

```
let a=['啦啦',2,4,24,NaN]
console.log(a.indexOf('啦'));  // -1 
console.log(a.indexOf('NaN'));  // -1 
console.log(a.indexOf('啦啦')); // 0
```

使用场景：
* 1.数组去重
* 2.根据获取的数组下标执行操作，改变数组中的值等。
* 3.判断是否存在，执行操作。

#### lastIndexOf()

定义: 方法返回指定元素,在数组中的最后一个的索引，如果不存在则返回 -1。（从数组后面往前查找）

语法：
```
arr.lastIndexOf(searchElement, fromIndex)
```

参数：

* 1.searchElement（必须）：被查找的元素
* 2.fromIndex（可选）：逆向查找开始位置，默认值数组的长度 -1，即查找整个数组。

关于fromIndex有三个规则：

* 1.正值。如果该值大于或等于数组的长度，则整个数组会被查找。

* 2.负值。将其视为从数组末尾向前的偏移。(比如-2，从数组最后第二个元素开始往前查找)

* 3.负值。其绝对值大于数组长度，则方法返回 -1，即数组不会被查找。

```
 let a=['OB',4,'Koro1',1,2,'Koro1',3,4,5,'Koro1']; // 数组长度为10
 // let b=a.lastIndexOf('Koro1',4); // 从下标4开始往前找 返回下标2
 // let b=a.lastIndexOf('Koro1',100); //  大于或数组的长度 查找整个数组 返回9
 // let b=a.lastIndexOf('Koro1',-11); // -1 数组不会被查找
 let b=a.lastIndexOf('Koro1',-9); // 从第二个元素4往前查找，没有找到 返回-1

```

#### ES7 includes()

定义：返回一个布尔值，表示某个数组是否包含给定的值

语法：
```
arr.includes(searchElement,fromIndex=0)
```
参数：
* 1.searchElement（必须）：被查找的元素
* 2.fromIndex（可选）：默认值为0，参数表示搜索的起始位置，接受负值。正值超过数组长度，数组不会被搜索，返回false。负值绝对值超过长数组度，重置从0开始搜索。

```
let a=['OB','Koro1',1,NaN];
// let b=a.includes(NaN); // true 识别NaN
// let b=a.includes('Koro1',100); // false 超过数组长度 不搜索
// let b=a.includes('Koro1',-3);  // true 从倒数第三个元素开始搜索 
// let b=a.includes('Koro1',-100);  // true 负值绝对值超过数组长度，搜索整个数组
```

### 遍历方法（12个）

js中遍历数组并不会改变原始数组的方法共有12个：
```
    ES5：
    forEach、every 、some、 filter、map、reduce、reduceRight、
    ES6：
    find、findIndex、keys、values、entries

```

#### 关于遍历：

* 关于遍历的效率，可以看一下这篇[详解JS遍历](http://louiszhai.github.io/2015/12/18/traverse/#%E6%B5%8B%E8%AF%95%E5%90%84%E6%96%B9%E6%B3%95%E6%95%88%E7%8E%87)
* 尽量不要在遍历的时候，修改后面要遍历的值
* 尽量不要在遍历的时候修改数组的长度

#### forEach()

定义：按升序为数组中含有效值的每一项执行一次回调函数。

语法：

```
array.forEach(function(currentValue, index, arr), thisValue)
```

参数：

* 1.function（必须）：数组中每个元素需要调用的函数。
* 2.thisValue （可选）：当执行回调函数时this绑定对象的值，默认值为undefined

关于forEach()你要知道：

* 无法中途退出循环，只能用return退出本次回调，进行下一次回调。
* 它总是返回 undefined值,即使你return了一个值。

##### 下面类似语法同样适用这些规则


1. 对于空数组是不会执行回调函数的。
2. 对于已在迭代过程中删除的元素，或者空元素会跳过回调函数。
3. 遍历次数再第一次循环前就会确定，再添加到数组中的元素不会被遍历。
4. 如果已经存在的值被改变，则传递给 callback 的值是遍历到他们那一刻的值。

```
    let a = [1, 2, ,3]; // 最后第二个元素是空的，不会遍历(undefined、null会遍历)
    let obj = { name: 'OBKoro1' };
    let result = a.forEach(function (value, index, array) { 
      a[3] = '改变元素';
      a.push('添加到尾端，不会被遍历')
      console.log(value, 'forEach传递的第一个参数'); // 分别打印 1 ,2 ,改变元素
      console.log(this.name); // OBKoro1 打印三次 this绑定在obj对象上
      // break; // break会报错
      return value; // return只能结束本次回调 会执行下次回调
      console.log('不会执行，因为return 会执行下一次循环回调')
    }, obj);
    console.log(result); // 即使return了一个值,也还是返回undefined
    // 回调函数也接受接头函数写法

```

#### every()

定义：用于检测数组所有元素是否符合函数定义的条件

语法：
```
    array.every(function(currentValue, index, arr), thisValue)
```
参数：（几个方法参数都类似）

* 1.function （必须）：数组中每个元素需要调用的函数

    //回调函数参数
    1. currentValue(必须),数组当前元素的值
    2. index(可选), 当前元素的索引值
    3. arr(可选),数组对象本身

* 2.thisValue（可选）：当执行回调函数时this绑定对象的值，默认值为undefined

方法返回值规则：

* 1.如果数组中检测到有一个元素不满足，则整个表达式返回 false，且剩余的元素不会再进行检测。
* 2.如果所有元素都满足条件，则返回 true。

```
    function isBigEnough(element, index, array) { 
      return element >= 10; // 判断数组中的所有元素是否都大于10
    }
    let result1 = [12, 5, 8, 130, 44].every(isBigEnough);   // false
    let result2 = [12, 54, 18, 130, 44].every(isBigEnough); // true
    // 接受箭头函数写法 
    [12, 5, 8, 130, 44].every(x => x >= 10); // false
    [12, 54, 18, 130, 44].every(x => x >= 10); // true

```

#### some()

定义：数组中的是否有满足判断条件的元素

语法：

```
    array.some(function(currentValue, index, arr), thisValue)
```

参数：同every

方法返回值规则：

* 1.如果有一个参数满足条件，返回true，剩下元素不进行检测
* 2.如果没有元素满足条件，返回false

```
 function isBigEnough(element, index, array) {
   return (element >= 10); //数组中是否有一个元素大于 10
 }
 let result1 = [2, 5, 8, 1, 4].some(isBigEnough); // false
 let result2 = [12, 5, 8, 1, 4].some(isBigEnough); // true

```

#### filter()

定义：返回一个新数组，其包含通过所提供函数实现的测试的所有元素。

语法：
```
    let new_array = arr.filter(function(currentValue, index, arr), thisArg)
```

参数：同上

```
    let a = [32, 33, 16, 40];
    let result = a.filter(function (value, index, array) {
      return value >= 18; // 返回a数组中所有大于18的元素
    });
    console.log(result,a);// [32,33,40] [32,33,16,40]

```

#### map()

定义：创建一个新数组，其结果是该数组中的每个元素都调用一个提供的函数后返回的结果。

语法：
```
    let new_array = arr.map(function(currentValue, index, arr), thisArg)
```
参数：同上

```
let a = ['1','2','3','4'];
let result = a.map(function (value, index, array) {
  return value + '新数组的新元素'
});
console.log(result, a); 
// ["1新数组的新元素","2新数组的新元素","3新数组的新元素","4新数组的新元素"] ["1","2","3","4"]

```

#### reduce()

定义：reduce() 方法对累加器和数组中的每个元素（从左到右）应用一个函数，最终合并为一个值。

语法：
```
    array.reduce(function(total, currentValue, currentIndex, arr), initialValue)

```
参数：

* 1.function（必须）：数组中每个元素需要调用的函数

    // 回调函数的参数
    1. total(必须)，初始值, 或者上一次调用回调返回的值
    2. currentValue(必须),数组当前元素的值
    3. index(可选), 当前元素的索引值
    4. arr(可选),数组对象本身

* 2.initialValue(可选): 指定第一次回调 的第一个参数。

回调第一次执行时:

* 如果 initialValue 在调用 reduce 时被提供，那么第一个 total 将等于 initialValue，此时 currentValue 等于数组中的第一个值；
* 如果 initialValue 未被提供，那么 total 等于数组中的第一个值，currentValue 等于数组中的第二个值。此时如果数组为空，那么将抛出 TypeError。
* 如果数组仅有一个元素，并且没有提供 initialValue，或提供了 initialValue 但数组为空，那么回调不会被执行，数组的唯一值将被返回。

```
    // 数组求和 
    let sum = [0, 1, 2, 3].reduce(function (a, b) {
      return a + b;
    }, 0);
    // 6
    // 将二维数组转化为一维 将数组元素展开
    let flattened = [[0, 1], [2, 3], [4, 5]].reduce(
      (a, b) => a.concat(b),
      []
    );
     // [0, 1, 2, 3, 4, 5]

```

#### reduceRight()

这个方法除了与reduce执行方向相反外，其他完全与其一致，请参考上述 reduce 方法介绍。

#### ES6：find()& findIndex() 

find()定义：用于找出第一个符合条件的数组成员，并返回该成员，如果没有符合条件的成员，则返回undefined。

findIndex()定义：返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回-1。

语法：
```
    let new_array1 = arr.find(function(currentValue, index, arr), thisArg)
    let new_array2 = arr.findIndex(function(currentValue, index, arr), thisArg)
```

参数:(这几个方法的参数，语法都类似)

这两个方法都可以识别NaN,弥补了indexOf的不足。

```
    // find
    let a = [1, 4, -5, 10].find((n) => n < 0); // 返回元素-5
    let b = [1, 4, -5, 10,NaN].find((n) => Object.is(NaN, n));  // 返回元素NaN
    // findIndex
    let a = [1, 4, -5, 10].findIndex((n) => n < 0); // 返回索引2
    let b = [1, 4, -5, 10,NaN].findIndex((n) => Object.is(NaN, n));  // 返回索引4
```





