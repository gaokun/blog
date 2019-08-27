## 你可能没听过的Promise解读

> 本篇文章并不介绍Promise基本用法
> 
> 假定读者会使用Promise, 并有一定的使用场景的经验.

#### 一. 请思考这样一个对话:

> jQuery ajax请求, 之前都是callback方式, 后来也支持了Promise.

**砖家**: JQ既然添加了新特性, 那一定是哪里优秀吧?

**菜鸟**: 费话, 那当然了

**砖家**: 那是哪里不一样呢?

**菜鸟**: callback方式需要指定 `success` 与 `failure` 的回调函数

```javascript
$.get(url, function (data) {
	console.log('success', data);
}, function (error) {
	console.error('error', error);
});
```
而 promise方式 使用 `.done`, `.fail` 就行了

```javascript
$.get(url).done(function (data) {
	console.log('success', data);
}).fail(function (error) {
	console.error('error', error);
});
```
**砖家**: done, fail 的参数不是回调函数吗? 同样都用了回调函数, promise 反而多了一层, 不是更费事了呢?

**菜鸟**: 呃, 好像是这样

**砖家**: 那到底好在哪呢?

在各种 Promise 的文档中, 都是象上面那样介绍 Promise 用法, `.then`, `.catch`, 再来点语法糖, `.finally`, `.all`, `.race` 多方便啊.

抱歉, 其实这些都是 Promise 的"错误"用法.

> 郑重声明: promise还可以解决同步问题, 有很多其他的文章讲这个的用法, 本文并不做赘述

#### 二. 那 Promise 到底是为何而生呢?
**Conclusion first**:

Promise解决的核心问题不是`回调地狱(callback hell)`, 这只是语法糖.

Promise真正解决的是`数据依赖`问题, 采用的方法依然是我们常挂嘴边的, "老掉牙"的 **`解耦`**.

解的是 `数据产生(发)` 与 `使用(收)` 的耦, 也就是 `发` 与 `收` 的解耦.

回到刚才说的 `callback 与 Promise 在ajax请求上到底有什么不同` 这个问题.

**如何理解 ajax 里 Promise 的`解耦`呢? 和 callback 有什么本质不同吗?**

> ajax callback方式, 在请求发出去, 需要指定`success`, `failure`的回调函数, 
> 
> 在请求发出后, 则**失去控制权**. 

**举个例子**: 

```
美芽让小新去买鱼, 如果有粗面, 也买一点.

小新出发后, 妈妈又想起来鱼应该让店家加工成鱼丸,

但小新已经出发了(失去了控制权), 只能等他回来, 再继续处理下面的事情.
```

此时聪明的你会想到, 如果小新带个手机, 是不是就ok了, 妈妈可以给打电话呀.

> 对, 这就是Promise

**Promise 可以在请求过程中或者请求结束后, 添加新的处理逻辑, 保留对结果的处理权**.

[图]
[图]

那 Promise 是如何做到的呢? 
> 1. 增加请求状态判断
> 
> 2. 缓存请求结果

Promise版美芽 甚至可以让小新先出门去鱼店, 因为走路需要时间嘛.

然后要对鱼如何处理, 还没想好, 可以等小新出发后再打电话(`.then`)告诉他.

终上所述Promise的 `发` 与 `收` 是分开的,

所以说下面的示例, 是Promise的 "错误" 用法, 它并没有体现promise的解耦与灵活的数据依赖
> 请正确理解这里所说的 "错误"

```javascript
$.get(url).done(function (data) {
	console.log('success', data);
}).fail(function (error) {
	console.error('error', error);
});
```

#### 三. 正确的Promise用法

用单页面应用(SPA)很常见的场景, 多个子页面都需要user info数据, 这个ajax要如何做?

[图]

父页面: 将userInfoPromise变量传递到需要的子页面 

```javascript
const userInfoPromise = $.get('/userinfo');
```

子页面1: 

```javascript
userInfoPromise.then((userInfo) => {
	console.log('I got u', userInfo); // 逻辑1
});
```

子页面2: 

```javascript
userInfoPromise.then((userInfo) => {
	console.log('Show yourself', userInfo); // 逻辑2
});
```

**效果**: 
> 1. 直接从浏览器进入页面1, SPA先加载父页面, 请求userInfo
> 2. 子页面1加载, userInfo还在加载, 但可以添加处理逻辑了
> 3. 然后浏览完, 用户跳转到子页面2, 需要另一种处理逻辑, 此时userInfo已经加载完毕, 可以直接响应处理, 而不需要再发请求.

同样的, 如果用户先打开子页面2, 再浏览子页面1, 也是一样的. userInfo request只请求一次.

一套代码逻辑, 完美匹配业务需求. 

> 你可以尝试用callback方式实现上面的需求

当我们把 promise 传递出去的时候, 正如它的名字, 传递的是 **承诺**.

#### 四. 总结
上高中的时候, 数理化有辣么多的公式, 不知道你可曾有过这样的感觉.

每次考试就是不会, 出来一对答案: `"呀, 这公式我会啊, 怎么当时没想到呢?"`

这就是 只是记住了公式, 而对它的使用场景不明确, 本质理解的不清晰.

**最后再总结一下**:

Promise解决的核心问题是 `数据依赖`, 采用的手法是对 `发` 与 `收` 的解耦.

**`当我们 [有些逻辑] 需要依赖 [其他逻辑的结果]`** 时, 请使用 Promise, 用的不爽你打死我.


#### 下一篇预告:
**<原来 Vuex 和 Promise 是同一个爹>**

我们学习技术, 要学习他们解决问题的手法, 这样才能举一反三.

如果我说Vuex/Reduct/Flux和Promise的解决问题的手法一样, 你信吗?


> "那Vuex的文档看了很多, 还是不会用啊, 也不知道什么时候该用它啊"

如果你也有此疑问, 请期待下篇文章, 让你 "会记公式, 也会用公式".


--- end ---