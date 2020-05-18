## 数组拍平

```JavaScript
var arr = [1,2,[3,4,5,[6,7,8],9],10,[11,12]];
```

实现输出

```javascript
[1,2,3,4,5,6,7,8,9,10,11,12]
```

### 递归

```javascript
function flat(arr){
　　　let result = []
     arr.forEach((val)=>{
         if(Array.isArray(val)){
             result = result.concat(flat(val))
         }else{
             result.push(val)
         }
      })
      return result
 }
```

### flat方法

```javascript
var arr1 = [1, 2, [3, 4]];
arr1.flat(); 
// [1, 2, 3, 4]

var arr2 = [1, 2, [3, 4, [5, 6]]];
arr2.flat();
// [1, 2, 3, 4, [5, 6]]

var arr3 = [1, 2, [3, 4, [5, 6]]];
arr3.flat(2);
// [1, 2, 3, 4, 5, 6]

//使用 Infinity，可展开任意深度的嵌套数组
var arr4 = [1, 2, [3, 4, [5, 6, [7, 8, [9, 10]]]]];
arr4.flat(Infinity);
// [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

### reduce

```javascript
function flat(arr){
    return arr.reduce((prev,cur)=>{
        return prev.concat(Array.isArray(cur)?flat(cur):cur)
    },[])
}
```

