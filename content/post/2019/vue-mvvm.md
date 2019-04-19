+++
title = 'Vue MVVM (defineProperty) 实现'
date = 2019-04-19T23:33:14+08:00
categories = ["Vue"]
tags = ["vue", "mvvm"]
+++

![Vue MVVM Poster](https://cn.vuejs.org/images/data.png)

<!--more-->



## Vue 响应式原理

Vue 2.x 的响应式原理是通过 **数据劫持** 和 **发布-订阅**模式实现的。

初始化 Vue 实例时内部就会定义数据劫持和创建发布-订阅模式。

![Vue MVVM](https://ws1.sinaimg.cn/large/006tKfTcgy1g1pnu3komuj31z40m0wn6.jpg)

### 数据劫持

**数据劫持** 通过 `Object.defineProperty()` 劫持传入的 `data`，对 `data` 属性(包括嵌套对象) 的操作进行监听（通过 getter/setter），从而触发一系列响应。

在 [官方文档-深入响应式原理](https://cn.vuejs.org/v2/guide/reactivity.html) 中也说明了 Vue 在初始化实例后 **无法检测到对象属性的添加或删除**，需要通过 `Vue.set(vm.someObject, 'b', 2)` 或者 `this.$set(this.someObject,'b',2)` 的方式动态添加新的响应式属性，即给新的属性添加 getter/setter。



### 发布-订阅

**发布-订阅**模式是数据劫持之后的响应，订阅中心 **Dep** 收集对 **Data** 各个属性的依赖，**Watcher** 作为订阅者订阅这些依赖；依赖项的 setter 触发时，会通知对应的 **Watcher**，从而触发更新。



### MVVM 各模块职责

- **MVVM**: Vue 实例初始化，调用 **Observer** 数据劫持，调用 **Compiler** 解析模板；
- **Observer**: 劫持 data 全部属性，定义 setter 、getter，添加和通知订阅者；
- **Compiler**: 解析模板初始化视图，收集模板中的数据依赖，创建订阅者订阅变化，绑定更新函数；
- **Dep**：订阅中心，提供添加、移除、通知订阅的接口；
- **Watcher**: data 属性的订阅者，收到变化通知后调用更新函数更新视图。



## MVVM 原理实现

为了理解 Vue MVVM 内部原理，自己造一个简单的 MVVM 伪轮子，能够实现模板解析、数据响应式等基本功能，忽略细节和明显的 bug。同时，为了更容易理解和更好的 coding 体验，使用现代浏览器支持的 ES6+ 语法实现，划分各个模块。

这个轮子中借助 ES5 的 `Object.defineProperty()` 实现数据劫持；在下个轮子中，将会使用 ES6 的 `Proxy` 实现，这也是 Vue 3 的计划之一。

效果：

<iframe src="https://codesandbox.io/embed/q7j083qmyq?autoresize=1&fontsize=14&hidenavigation=1&view=preview" title="vue-mvvm" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>



### index.html

入口文件，创建渲染模板，通过 `v-model` 指令双向绑定 `data`，同理 `v-class`、`v-text`、`v-html` 指令插入 `data` 值，`v-on:click` 绑定点击事件监听器 `method`。

在 `module script` 中导入并实例化 MVVM，定义 `el`、`data`、`computed`、`methods`、`$watch`





### MVVM 模块

<iframe src="https://codesandbox.io/embed/q7j083qmyq?autoresize=1&fontsize=16&hidenavigation=1&view=editor" title="vue-mvvm" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

MVVM 构造函数中把 `options.data`、`options.computed`、`options.methods` 中的属性全部绑定到 MVVM `this` 上，这样就能在 `methods` 中通过 `this.xxx` 代理到各自的 key 中，暂时不考虑重复的 key。

初始化 `this` 代理后，调用 `Observer` 劫持 `data` 中的全部属性。

最后，调用 `Compiler` 解析模板。

MVVM 实例也提供了 `$watch` 方法，调用 `Watcher` 监听指定属性的变化。



### Dep 模块

<iframe src="https://codesandbox.io/embed/q7j083qmyq?autoresize=1&fontsize=16&hidenavigation=1&module=%2FDep.js&view=editor" title="vue-mvvm" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

**Dep** 模块作为**发布-订阅**模式的订阅中心，每个 `data` 中的属性(包括嵌套) 都绑定了一个 Dep 实例(通过闭包引用)，提供添加、通知、移除订阅者的接口。

一个某属性绑定的 Dep 实例可以包含多个订阅该属性的 Watcher 实例，属性变化后，将通知这些 Watcher 实例。



### Observer 模块

<iframe src="https://codesandbox.io/embed/q7j083qmyq?autoresize=1&fontsize=16&hidenavigation=1&module=%2FObserver.js&view=editor" title="vue-mvvm" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

**Observer** 模块依赖 **Dep** 模块。

Observer 构造函数递归劫持 `data` 中的全部属性，定义 getter 和 setter，同时为每个属性绑定一个 Dep 实例。

在属性的 getter 中判断当前 `Dep.target` 是否指向一个 `Watcher` 实例，如果是，则把该 `Watcher` 实例作为属性订阅者添加到 `Dep` 中：

```js
get() {
  // 访问该 key 时如果 Dep.target 指向 Watcher 实例，把该 key 对应的 Dep 实例传递给 Watcher 实例
  // 也可以直接 dep.addSub(Dep.target)
  Dep.target && dep.depend()
  return val
}
```

在属性的 setter 中当值变化后通过闭包引用的 `Dep` 实例通知订阅者：

```js
set(newVal) {
  if (newVal === val) {
    return
  }
  val = newVal
  // 新的值是object的话，进行监听
  observe(newVal)
  // 通知订阅该 key 的 Watcher 实例
  dep.notify()
}
```



#### Observer named export

`Observer` 模块导出的不是 `Observer class`，而是 `observe` 函数，用于递归实例化 `Observer` 劫持嵌套的 `data key` 对象。相当于 `data` 中的每个嵌套对象都对应一个 `Observer` 实例，包括 `data` 本身。不过在这个轮子中没有用到 `Observer` 实例的引用。



### Watcher 模块

<iframe src="https://codesandbox.io/embed/q7j083qmyq?autoresize=1&fontsize=16&hidenavigation=1&module=%2FWatcher.js&view=editor" title="vue-mvvm" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

**Watcher** 模块依赖 **Dep** 模块。

`Watcher` 在初始化时将会把 `Dep.target` 指向自身的 `this`，然后访问监听的属性，触发 `Observer` 中定义的属性 getter，从而把当前 `Watcher` 实例添加到属性订阅者中。

```js
// 获取 Wacher 实例监视的值
_getValue() {
  // 访问监听的属性时把 Dep.target 指向自身，从而在 Observer 中把当前实例添加到属性订阅者中
  Dep.target = this
  const value = this.getter(this.vm)
  // 获取属性值后置空 Dep.target
  Dep.target = null
  return value
}
```



#### 监听嵌套属性

如果 `Watcher` 实例监听的是嵌套属性，比如 `a.b.c`，关注的是 `c` 值的变化。则在访问 `a` 时，`a` 绑定的 `Dep` 实例会把当前 `Watcher` 实例添加到订阅者中，同理 `b`、`c`，即一个 `Watcher` 实例的引用同时存在于嵌套属性的父属性 `Dep` 订阅者中；当父属性值变化时，也会通知到子属性的订阅者。



#### 重复的 Dep

```js
applyDep(dep) {
  if (!this.depIds.hasOwnProperty(dep.id)) {
    dep.addSub(this)
    this.depIds[dep.id] = dep
  }
}
```

假如相应属性的 `dep.id` 已经在当前 `watcher` 的 `depIds` 里，说明不是一个新的属性，仅仅是改变了其值而已，则不需要将当前 `watcher` 添加到该属性的 `dep` 里。

假如相应属性是新的属性，则将当前 `watcher` 添加到新属性的`dep`里，如通过 `vm.child = {name: 'a'}` 改变了 `child.name` 的值，`child.name` 就是个新属性，则需要将当前`watcher(child.name)`加入到新的 `child.name` 的`dep`里。因为此时 `child.name` 是个新值，之前的 `setter`、`dep` 都已经失效，如果不把 `watcher` 加入到新的 `child.name` 的`dep`中，通过 `child.name = xxx` 赋值的时候，对应的 `watcher` 就收不到通知，等于失效了。



### Compiler 模块

<iframe src="https://codesandbox.io/embed/q7j083qmyq?autoresize=1&fontsize=16&hidenavigation=1&module=%2FCompiler.js&view=editor" title="vue-mvvm" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

**Compiler** 模块依赖 **Watcher** 模块。

#### DocumentFragment

`Compiler` 实例化时先把模板节点的子节点全部转移到 `DocumentFragment` 中进行节点操作，避免频繁 reflow，影响性能；在模板解析完后再把 `DocumentFragment` 转移到根节点下。

```js
function nodeToFragment(node) {
  const fragment = document.createDocumentFragment()

  // 将原生节点移动到fragment
  let child = node.firstChild
  while (child) {
    fragment.appendChild(child)
    child = node.firstChild
  }

  return fragment
}
```



#### 指令解析

`Compiler` 解析模板中的 `v-` 前缀的指令和 `{{ exp }}` 文本插值表达式，该轮子忽略 `JavaScript` 插值表达式。能够解析的指令有 `v-model`、 `v-class`、`v-text`、`v-html`、`v-on`，这些指令都是引用 `data` 中的属性，触发 `Observer` 定义的 getter 和 setter。

```js
// 指令处理集合
const compilerUtils = {
  text() {},
  html() {},
  // ...
}
```



#### v-model 双向绑定

`v-model` 指令绑定的属性，`View` 和 `Model` 中的值变化互相影响，即双向绑定。

监听 `v-model` 指令所在的 `node input` 事件，`input` 变化后通过 `vm.xxx` 改变对应的 `model` 属性值，触发该属性定义的 setter，即实现了 `View` 影响 `Model`。

```js
model(node, vm, exp) {
  this.bind(node, vm, exp, 'model')

  let val = this._getVMVal(vm, exp)
  node.addEventListener('input', e => {
    const newValue = e.target.value
    if (val === newValue) {
      return
    }
    this._setVMVal(vm, exp, newValue)
    val = newValue
  })
}
```



#### 监听属性变化

每个指令引用的 `data` 属性都需要实例化 `Watcher` 进行监听，并同时绑定更新函数给 `Watcher` 收到变化通知后调用，更新视图：

```js
bind(node, vm, exp, dir) {
  const updaterFn = updater[dir + 'Updater']

  updaterFn && updaterFn(node, this._getVMVal(vm, exp))

  new Watcher(vm, exp, (value, oldValue) => {
    updaterFn && updaterFn(node, value, oldValue)
  })
}
```



#### 属性值更新

监听的属性值变化后，每个指令都对应不同的更新操作，比如替换 `textContent`、`innerHTML`、`value` 等：

```js
const updater = {
  textUpdater(node, value) {
    node.textContent = typeof value === 'undefined' ? '' : value
  },
  htmlUpdater() {}
  // ...
}
```

在这个轮子中，更新 DOM 是同步执行的，而且不涉及 Virtual DOM。在 Vue 中会对 Virtual DOM 更新然后缓存在异步更新队列，在下一个事件循环 **tick** 中 flush 到 DOM。



### 回顾

**MVVM** 模式降低了 **View** 与 **Model** 的耦合，由 **ViewModel** 自动处理 View 与 Model 的数据同步，解决了数据频繁更新（操作 DOM）的问题。View 通过使用**模板语法**来**声明式**的将数据渲染进 DOM，当 ViewModel 对 Model 进行更新的时候，会通过数据绑定更新到 View。Model (在本文中就是 `data`) 仅仅关注数据层本身，不关心任何行为；Model 的数据模型结合视图和业务实体。

~~宠着~~，画了个各模块的关系图帮助理解：

![image-20190419115645881](https://ws3.sinaimg.cn/large/006tNc79gy1g27t19vigyj30tq0wd42d.jpg)



最后，本轮子 GitHub 地址：<https://github.com/Jancat/vue-mvvm>

**Follow** or **Star**，你选哪个？



（~~不，小孩子才做选择，我全都要~~）



## 参考

本轮子基于下面的仓库代码改造，感谢！

<https://github.com/DMQ/mvvm>