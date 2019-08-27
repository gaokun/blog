> 老规矩, 本文并不介绍Vuex/Redux/Flux基本用法
> 
> 假定读者了解Flux思想, 会使用Promise, 并有一定的使用场景的经验

#### 一. 引言
Flux 是一种状态管理的架构, 一种思想. Redux, Vuex 在其基础上改进, 属一脉相承, 所以本文以Vuex来表述.

你是否也曾有过疑问: 

> "Vuex/Redux 的文档看了很多, 还是不会用啊, 也不知道什么时候该用它啊"
> 
> "就象高中的公式, 都会背, 但考试的时候就不知道该用哪个"

在刚开始看Flux的时候, 我也认为它就是MVC的一个翻版. 直到找到合适的业务场景: 呃, 真香 ^_^

本文从一个特别的角度切入, 让你 "会记公式, 也会用公式".

**请先赞同这个说法:**  

```
程序的本质: 是用 [代码] 操作 [数据].
```

游戏人物从A点到跑到B点, 是人物位置的变化. 

打怪升级后, 会带来属性的变化. 

而无论玩家如何操作, 都没有任何反馈的游戏, 恐怕没人喜欢玩.

所以对任何输入都做 相同反馈或无反馈 的 程序或功能 是没有意义的.

<br>

**小明**: 那 linux shell > /dev/null 2>&1 呢?

**老师**: gun出去!

---
#### 二. EventBus 与 Vuex 示例

在 Vue 中 Vuex 和 EventBus 是 **组件通讯** 的两种方案.
> 这里的 EventBus 特指在 Vue 中的用法, 即创建一个全局的Vue实例, 用于收发事件.
>
> 我们也不对 EventBus 做过多展开, 写的多而全, 容易偏离主题.

#### 用一个功能来举例: 显示一个Popup.

在 **组件B** 中, 点击button, 可控制 **组件A** 中的popup的显示.

##### EventBus 版实现: 

> EventBus实际就是观察者模式.

首先绑定一个全局的Vue实例

main.js:

```javascript
Vue.prototype.$eventBus = new Vue();
```

然后就可以在其他的Vue实例中使用了

组件A ('收'组件):

```javascript
Vue.component('ComponentA', {
	template: `<popup :visible="visible"></popup>`,
	data() {
		return {
			visible: false
		}
	},
	created() {
		this.$eventBus.$on('setVisible', (event) => {
			this.visible = event.visible;
		});
	}
});
```

组件B ('发'组件):

```javascript
Vue.component('ComponentB', {
	template: `<button @click="togglePopup">Toggle Popup</button>`,
	data() {
		return {
			visible: false
		}
	},
	methods: {
		togglePopup() {
			this.visible = !this.visible;
			this.$eventBus.$emit('setVisible', { visible: this.visible });
		}
	}
});
```


##### Vuex 版实现:

store:

```javascript
{
	state: {
		visible: false
	},
	mutations: {
		setVisible(state, visible) {
			state.visible = visible;
		}
	}
}
```

组件A ('收'组件):

```javascript
Vue.component('ComponentA', {
	template: `<popup :visible="visible"></popup>`,
	computed() {
		visible() {
			return this.$store.state.visible;
		}
	},
});

```

组件B ('发'组件):

```javascript
Vue.component('ComponentB', {
	template: `<button @click="togglePopup">Toggle Popup</button>`,
	computed() {
		visible() {
			return this.$store.state.visible;
		}
	},
	methods: {
		togglePopup() {
			this.$store.commit('setVisible', !this.visible);
		}
	}
});
```
OK, 上面两个例子, 展示了Vue的组件间通讯, 但区别在哪里?

---
#### 三. 干货分析

直接说结论:

#### EventBus方式: 要求 '收' 与 '发' 组件 同时*在场*.

#### Vuex方式: 将 '收' 与 '发' 组件 分离, 到达解耦的目的.

如何理解这里的 **'在场'** 呢?

在 EventBus方式 中, 如果 '收'组件, 当时没有渲染在页面上, 那么 '发'组件发出的 `显示popup` 的消息, 则无人响应, 没有任何效果. 所以说, 要求 '收'组件的实例也要渲染在页面上. 这就是 **'在场'**.

#### 我们结合上一篇提到的 Ajax请求 [Callback方式] 与 [Promise方式] 做个分析: 

**收** 与 **发** 耦合:
> Ajax Callback方式:
>> 请求**发**出时, 要求同时指定 '收' 的逻辑 (指success, failure).
> 
> 组件通讯 EventBus方式:
>> 消息**发**出时, 要求'收'组件在场.

**收** 与 **发** 解耦:
> Ajax Promise方式:
>> 请求可以先**发**出, 不要求同时指定`success`, `failure`, '收' 的逻辑可在之后通过promise `.then`, `.catch`处理.
> 
> 组件通讯 Vuex方式:
>> 消息**发**出时, 只改变state, 不要求'收'组件在场, 之后当'收'组件渲染时, 则会根据state渲染.


这呼应本文标题
> <原来 Vuex/Redux/Flux 和 Promise 都做对了同一件事情>

---
#### 三. 总结

至此, 什么时候应该用 Vuex, 什么时候应该用 EventBus, 想必读者心里也有数了.

对, **当数据变化时 '收'组件 不在场时, 就应该使用Vuex**

举例: 购物车

> 购物车可以收起内容, 只显示数量, 也可以展开来显示商品.
>
> 即 '商品列表' 组件, 在添加商品时, 可能并不 **在场**

如果业务上, 收发组件总是同时出现, 就没有必要非要使用Vuex, 毕竟有个全局污染问题

**PS:**
> 不知道你是否会问: "Vuex毕竟要改一个state, 如果没有state要改呢, 只是想传递一个事件呢?"

请见本文开始提到的

```
程序的本质: 是用 [代码] 操作 [数据].
```
我们要做的, 就是把要 '操作的数据' 提到 state 中. 

--- end ---