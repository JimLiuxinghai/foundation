## 千分位加逗号

### 1.toLocalString()

```javascript
(123456789).toLocaleString('en-US');
```

对于中文场景下，`toLocaleString('en-US')`中的`'en-US'`理论上是可以缺省的，也就是直接`(123456789).toLocaleString()`也是可以得到123,456,789。但是如果你的产品可能海外用户使用，则保险起见，还是保留`'en-US'`。

另外，对于IE edge之前的版本，`Number.toLocaleString()`会自动补上两位小数，如果是不需要的，需要自己额外过滤掉。

### 2.正则

```javascript

var num = 123123.1212
function formatData(num) {
	num += '';
	if (!num.includes('.')) num += '.';
	return num.replace(/(\d)(?=(\d{3})+\.)/g, function($0, $1) {
	  return $1 + ',';
	}).replace(/\.$/, '');
}
```

在`\d`前面加一个非单词边界`\B`，用来表示所匹配的这个空后面不能是一个单词边界，这样就可以把最前面的这个`,`去掉了。

* `g`是表示全局匹配的修饰符，全局匹配指查找所有匹配而非在找到第一个匹配后停止。

* `$`是表示结尾的量词，如`n$`，匹配的是任何以n为结尾的字符串。

* `\d`是查找数字的元字符。

* `n{X}`是匹配包含 X 个 n 的序列的字符串的量词。

* `n+`是匹配任何包含至少一个 n 的字符串的量词。

* `?=n`正向预查，用于匹配任何其后紧接指定字符串 n 的字符串。

* `match()` String对象的方法，作用是找到一个或多个正则表达式的匹配。

* `replace()`String对象的方法，作用是替换与正则表达式匹配的子串。

* `\B`是表示匹配非单词边界的元字符，与其互为补集的元字符是`\b`，表示匹配单词边界。

### 3.数组

```javascript
function toThousands(num) {
    var result = [ ], counter = 0;
    num = (num || 0).toString().split('');
    for (var i = num.length - 1; i >= 0; i--) {
        counter++;
        result.unshift(num[i]);
        if (!(counter % 3) && i != 0) { result.unshift(','); }
    }
    return result.join('');
}
```

