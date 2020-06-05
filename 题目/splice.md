## 模拟实现Array.splice

定义： splice() 方法向/从数组中添加/删除项目，然后返回被删除的项目

语法：array.splice(index, howmany, item1, ...itemX)

参数：

* 1.index：必需。整数，规定添加/删除项目的位置，使用负数可从数组结尾处规定位置。
* 2.howmany： 可选。要删除的项目数量。如果设置为 0，则不会删除项目。
* 3.item： 可选。向数组添加的新元素。

返回值: 如果有元素被删除,返回包含被删除项目的新数组



```javascript
Array.prototype._splice = function (start, deleteCount, ...addList) {
  if (start < 0) {
    if (Math.abs(start) > this.length) {
      start = 0
    } else {
      start += this.length
    }
  }

  if (typeof deleteCount === 'undefined') {
    deleteCount = this.length - start
  }

  const removeList =  this.slice(start, start + deleteCount)

  const right = this.slice(start + deleteCount)

  let addIndex = start
  addList.concat(right).forEach(item => {
    this[addIndex] = item
    addIndex++
  })
  this.length = addIndex

  return removeList
}
```

