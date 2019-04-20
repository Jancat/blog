+++
title = 'Vue MVVM 实现 （Proxy 篇）'
date = 2019-04-20T10:32:43+08:00
categories = ["Vue"]
tags = ["vue", "mvvm"]
isCJKLanguage = true
+++

[上一篇](https://jancat.github.io/post/2019/vue-mvvm/)实现 MVVM **数据劫持**是基于 ES5 的 `Object.defineProperty()` API，新的轮子将使用 `Proxy` 替代，follow Vue 3；在原有轮子的基础上做些改造，适配 `Proxy`。

`Object.defineProperty()` 和 `Proxy` 实现 MVVM 数据劫持的区别是：`Object.defineProperty()` 需要为每个 key 定义 getter 和 setter，也就是需要遍历 `data` 下的全部子属性；而 `Proxy` 只需要为 `data` 本身和内部嵌套的对象创建代理，每个对象统一代理访问内部属性，对外提供代理的引用。



## MVVM Proxy 原理实现

效果（同上一个轮子）：

<iframe src="https://codesandbox.io/embed/w2204y1jm5?autoresize=1&fontsize=14&hidenavigation=1&view=preview" title="vue-mvvm-proxy" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>



**index.html** 和 **Compiler** 模块不需要改动，详情见 [上一篇](https://jancat.github.io/post/2019/vue-mvvm/)



### MVVM 模块

<iframe src="https://codesandbox.io/embed/w2204y1jm5?autoresize=1&fontsize=16&hidenavigation=1&module=%2FMVVM.js&view=editor" title="vue-mvvm-proxy" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

**MVVM** 初始化时需要代理 `this.xxx` 到 `this.$data.xxx` 或者 `options.computed.xxx`，所以给 `this` 创建一个 `Proxy` 代理，判断 `key` 在 `this` 本身、`data`、`methods`、`computed` 中，分别通过 `Reflect.get()` 在对应的 object 上获取 key 值。

```js
_proxyThis() {
    const { $options } = this
    const { computed } = $options
    return new Proxy(this, {
      get(target, key, receiver) {
        // 访问 MVVM 实例属性
        if (key in target) return Reflect.get(target, key, receiver)
        // 访问 data 属性
        if (key in $options.data)
          return Reflect.get(target.$data, key, receiver)
        // 访问 method
        if (key in $options.methods)
          return Reflect.get($options.methods, key, receiver)
        // 访问 computed 属性
        return typeof computed[key] === 'function'
          ? computed[key].call(target._vm)
          : Reflect.get(computed, key, receiver)
      },
      set(target, key, value, receiver) {
        // 设置 data 属性
        if (!target[key]) {
          return Reflect.set(target.$data, key, value, receiver)
        }
        return Reflect.set(target, key, value, receiver)
      },
    })
  }
```

`new Proxy(this, handler)` 创建的 `this` 代理需要作为 `MVVM` 构造函数的实例对象返回，因为提供给外部访问的是代理，即 `this._vm`：

```js
constructor(options = {}) {
  // ...
  this._vm = this._proxyThis()
	// ...
  return this._vm
}
```

然后将 `methods` 函数中的 `this` 绑定到 `this._vm` 上:

```js
_bindMethods() {
  const methods = this.$options.methods
  Object.keys(methods).forEach(method => {
    methods[method] = methods[method].bind(this._vm)
  })
}
```

调用 `Observer` 给 `options.data` 和嵌套对象创建代理：

```js
this.$data = observe(options.data)
```

最后还是一样调用 `Compiler` 解析模板：

```js
this.$compile = new Compiler(options.el || document.body, this._vm)
```





### Dep 模块

<iframe src="https://codesandbox.io/embed/w2204y1jm5?autoresize=1&fontsize=16&hidenavigation=1&module=%2FDep.js&view=editor" title="vue-mvvm-proxy" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

**Dep** 模块存储结构跟上个轮子的 Dep 不同，因为使用 `Proxy` 代理后，只能为每个对象绑定一个 `Dep` 实例，一个 `Dep` 实例包含该对象内部**所有属性**的订阅者，而不是之前的一个属性一个 `Dep` 实例。

为了保证存储订阅者的数据结构的唯一性，使用 **Set** 结构保证订阅者不会重复；外层使用 **Map** 结构存储 **key - Set** 的映射关系。

订阅中心提供的每个接口，都需要 `key` 和 `sub` 参数，根据 `key` 找出 **Map** 中映射的 **Set**，在订阅者 **Set** 中进行操作。



### Observer 模块

<iframe src="https://codesandbox.io/embed/w2204y1jm5?autoresize=1&fontsize=16&hidenavigation=1&module=%2FObserver.js&view=editor" title="vue-mvvm-proxy" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

**Observer** 模块使用 `Proxy` 代理 `data` 中嵌套对象的访问。

导出的 `observe` 函数递归遍历 `data` 内部属性，遇到对象就创建一个 `Proxy` 覆盖原始对象值，最后返回 `data` 对象的代理：

```js
export function observe(data) {
  // data 不是对象无法劫持，忽略
  if (!data || typeof data !== 'object') {
    return data
  }
  // 深度监听
  Object.keys(data).forEach(key => {
    data[key] = observe(data[key])
  })

  return Observer(data)
}
```



`proxyObject` 函数为 `data` 中的每个嵌套对象创建代理和绑定的 `Dep` 实例。跟上个轮子不同，在 getter 中，这里简化了添加订阅者的逻辑，如果判断 `Dep.target` 存在，直接通过闭包在绑定的 `Dep` 实例上把订阅该 `key` 的 `Watcher` 实例添加到订阅者中。

```js
const proxyObject = obj => {
  const dep = new Dep()
  return new Proxy(obj, {
    get: function(target, key, receiver) {
      // 如果订阅者存在，直接添加订阅
      if (Dep.target) {
        dep.addSub(key, Dep.target)
      }
      return Reflect.get(target, key, receiver)
    },
    set: function(target, key, value, receiver) {
      if (Reflect.get(receiver, key) === value) {
        return
      }
      const res = Reflect.set(target, key, observe(value), receiver)
      dep.notify(key)
      return res
    },
  })
}
```



### Watcher 模块

<iframe src="https://codesandbox.io/embed/w2204y1jm5?autoresize=1&fontsize=16&hidenavigation=1&module=%2FWatcher.js&view=editor" title="vue-mvvm-proxy" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

因为添加订阅者的逻辑已经在 `Observer` 模块做了，所以 `Watcher` 模块删除了 `applyDep()` 方法，其它不变：

```js
// applyDep(dep) {
//   if (!this.depIds.hasOwnProperty(dep.id)) {
//     dep.addSub(this)
//     this.depIds[dep.id] = dep
//   }
// }
```



### 新的模块关系图

![image-20190419211202655](https://ws3.sinaimg.cn/large/006tNc79gy1g2893dkdxlj30xm0u0n18.jpg)



## 总结

Vue 3.0 使用了 **Proxy** 后，将会消除之前  Vue 2.x 中基于 `Object.defineProperty` 的实现所存在的一些限制：无法监听 **属性的添加和删除**、**数组索引值和长度的变更**。



GitHub 地址：<https://github.com/Jancat/vue-mvvm-proxy>



## 参考

<https://github.com/xiaomuzhu/proxy-vue>