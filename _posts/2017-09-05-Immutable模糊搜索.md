---
layout: post
title: "Immutable模糊搜索"
date: 2017-09-05
---

### 情景

开发中遇到的问题，Immutable.js数据很复杂，List嵌套Map，Map又有一个bList的值，bList是由bMap组成的，要过滤出bMap下某一value是否包含所键入的string。

大致简化模型如下：

<img src="http://p9qv3iwy5.bkt.clouddn.com/markdown/1504593772808.png" width="600"/>



<img src="http://p9qv3iwy5.bkt.clouddn.com/markdown/1504593812214.png" width="600"/>

<img src="http://p9qv3iwy5.bkt.clouddn.com/markdown/1504593309640.png" width="600"/>



相关的mock数据和keynote可以在[百度云](https://pan.baidu.com/s/1c2xqd8G)上查看，密码:`xxge`。



### 解决方法



```javascript
filteInputChange = (event) => {
  const key = event.target.value;
  const after = this.props.data.filter(
    e => e.get('students')
          .find(
            content => content.get('name').includes(key),
          ),
  ).map(
    e => e.update(
      v => v.set(
        'students',
        v.get('students')
         .filter(
           content => content.get('name').includes(key),
         ),
       ),
     ),
  );
  this.setState({ data: after });
}
```



1. 首先筛选出符合条件的section
2. 再更新section下的students(`array`)



这里的update方法详细见[官方介绍](https://facebook.github.io/immutable-js/docs/#/Map/update)

> If no key is provided, then the `updater` function return value is returned as well.

```javascript
const aMap = Map({ key: 'value' })
const result = aMap.update(aMap => aMap.get('key'))
// "value"
```

