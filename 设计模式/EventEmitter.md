## 观察者模式

观察者模式（observer）广泛的应用于javascript语言中，浏览器事件（如鼠标单击click，键盘事件keyDown）都是该模式的例子。设计这种模式背后的主要原因是促进形成低耦合，在这种模式中不是简单的对象调用对象，而是一个对象“订阅”另一个对象的某个活动，当对象的活动状态发生了改变，就去通知订阅者，而订阅者也称为观察者。



假设有一个发布者Jack，它每天出版报纸杂志，订阅者Tom将被通知任何时候发生的新闻。

Jack要有一个subscribers属性，它是一个数组类型，订阅的行为将会按顺序存放在这个数组中，而通知意味着调用订阅者对象的某个方法。因此，当用户Tom订阅信息的时候，该订阅者要向Jack的subscribe()提供他的一个方法。当然也可以退订，我不想再看报纸了，就调用unsubscribe()取消订阅。



一个简单的观察者模式应有以下成员：

- subscribes 一个数组
- subscribe() 将订阅添加到数组里
- unsubscribe() 把订阅从数组中移除
- publish() 迭代数组，调用订阅时的方法

这个模式中还需要一个type参数，用于区分订阅的类型，如有的人订阅的是娱乐新闻，有的人订阅的是体育杂志，使用此属性来标记。

```javascript
var Jack = {
  subscribers: {
    'any': []
  },
  //添加订阅
  subscribe : function (type = 'any', fn) {
    if(!this.subscribers[type]) {
      this.subscribers[type] = [];
    }
    this.subscribers[type].push(fn); //将订阅方法保存在数组里
  },
  //取消订阅
  unsubscribe : function (type = 'any', fn) {
    this.subscribers[type] = this.subscribers[type].filter(function (item) {
      return item !== fn;
    }); 
  },
  //发布订阅
  publish : function (type = 'any', ...args) {
    console.log(...args, 'argssss')
    this.subscribers[type].forEach(function (item) {
      item(...args);	//根据不同的类型调用相应的方法
    });
  } 

}
var Tom = {
  readNews: function (info) {
    console.log(info);
  }
};

//Tom订阅Jack的报纸
Jack.subscribe('娱乐', Tom.readNews);
Jack.subscribe('体育', Tom.readNews);

//Tom 退订娱乐新闻：
Jack.unsubscribe('娱乐', Tom.readNews);

//发布新报纸：
Jack.publish('娱乐', 'S.H.E演唱会惊喜登台')
Jack.publish('体育', '欧国联-意大利0-1客负葡萄牙');
```

