+++
title = '设计一对通用的消息提示组件 —— Alert & Toast'
date = 2020-12-26T22:02:07+08:00
categories = ["UI Component"]
tags = ["组件库", "Alert", "Toast"]
+++

![](https://lg-gxg3fdgs-1254198195.cos.ap-shanghai.myqcloud.com/images/20201226221536.png)

在浏览器页面中，通用的消息提示组件一般可以分为**静态局部提示**和**动态全局提示**，用于反馈用户需要关注的信息，使用频率较高。
<!--more-->

虽然这两种提示从视觉交互上来看比较相似，但使用场景不同，组件对外暴露的接口也有一点区别，所以一般拆分成两个 UI 组件提供给业务方使用。


代码未动，设计先行，先考虑接口，再考虑实现。本文介绍如何设计这两个组件（展示、交互、API、实现思路），满足业务方的定制化和组件的通用化，提供简单易用、视觉交互友好的消息提示组件。

以下的设计思路整理总结自 People Design、 Ant Design 、Material-UI 的消息提示组件 ，主要针对 PC 端设计展示和交互，使用 React 语法 ；本文主要关注组件基础设计思路，不涉及：具体代码实现、无障碍设计 WAI-ARIA、主题切换、移动端适配等。



Alert & Toast ↓↓



## Alert 警告提示

静态局部提示组件在 Antd 和 Material-UI 中命名为 **Alert 警告提示**



### 使用场景

页面局部展示一段简短且重要的信息，在不影响用户操作的同时能够吸引用户的注意力。



### 展示交互

消息框宽度默认水平撑满当前容器，以非浮层的静态展现形式，始终展现，不会自动消失，用户可以点击关闭，可以提供自定义的操作按钮。



### 标题和描述

Alert 提示可以指定标题，表示提示的类型或主题

长文本提示建议增加标题，用户可快速理解主要内容，内容建议不超过 4 行。

![](https://lg-gxg3fdgs-1254198195.cos.ap-shanghai.myqcloud.com/images/20201226221001.png)

*Material-UI Alert 带标题和内容*

![](https://lg-gxg3fdgs-1254198195.cos.ap-shanghai.myqcloud.com/images/20201226221028.png)

*AntD Alert 带标题和内容*



#### API 设计

| 参数    | 说明                     | 类型      | 默认值 |
| ------- | ------------------------ | --------- | ------ |
| message | 警告提示内容，一般是文本 | ReactNode | -      |
| title   | 在内容上方展示的消息标题 | ReactNode | -      |



```jsx
<Alert title="标题" message="提示内容" />
```



### 提示类型

Alert 按照功能默认提供四种类型的提示，分别是：

- **普通提示 info**：用于展示背景条件、政策信息、规范要求、当前状态等客观内容；
- **成功提示 success**：用于展示已完成操作的成功状态；
- **警告提示 warning**：用于展示可能会导致某种后果的警示性文本；
- **错误提示 error**：用于展示当前操作或者整体状态有错误，提示用户修正或展示错误相关信息。

![](https://lg-gxg3fdgs-1254198195.cos.ap-shanghai.myqcloud.com/images/20201226221039.png)

*AntD Alert 四种提示类型*

![](https://lg-gxg3fdgs-1254198195.cos.ap-shanghai.myqcloud.com/images/20201226221048.png)

*Material-UI Alert 四种提示类型*

每种提示类型都提供对应的图标展示在消息开头，合适的图标让信息类型更加醒目，不带图标适合单调简单的提示。



#### API 设计

| 参数 | 说明                                                         | 类型                 | 默认值           |
| ---- | ------------------------------------------------------------ | -------------------- | ---------------- |
| type | 提示类型，共有四种选择 `success`、`info`、`warning`、`error` | string               | `info`           |
| icon | 辅助图标，可以指定 `false` 不展示图标，或者提供自定义图标    | ReactNode \| boolean | type 对应的 icon |



```jsx
<Alert message="info tips" type="info" />
<Alert message="success tips" type="success" icon={false} />
<Alert message="warning tips" type="warning" icon={<CustomIcon />} />
<Alert message="error tips" type="error" />
```



### 变体

提示框有描边（outlined）和填充（filled）这两种变体可以使用，以便匹配不同的设计风格。

标准（默认）是浅色填充无边框，填充（filled）是深色填充，描边（outlined）只有边框无填充。



![](https://lg-gxg3fdgs-1254198195.cos.ap-shanghai.myqcloud.com/images/20201226221058.png)

*Material-UI Alert 变体示例*



#### API 设计

| 参数    | 说明                                                    | 类型   | 默认值     |
| ------- | ------------------------------------------------------- | ------ | ---------- |
| variant | 变体类型，共有三种选择 `standard`、`filled`、`outlined` | string | `standard` |



```jsx
<Alert message="info tips" type="info" />
<Alert message="success tips" type="success" variant="filled" />
<Alert message="warning tips" type="warning" variant="outlined" />
```





### 可关闭

当用户接收到 Alert 提供的信息后，可能不希望再被 Alert 吸引注意力，影响对其它信息的处理；此时可以配置关闭按钮，允许用户主动关闭 Alert。

关闭后添加 leave 的过渡动效，平滑自然地收起 Alert。

![](https://lg-gxg3fdgs-1254198195.cos.ap-shanghai.myqcloud.com/images/20201226221111.png)

*AntD Alert 自定义关闭示例*



#### API 设计

| 参数       | 说明                              | 类型       | 默认值 |
| ---------- | --------------------------------- | ---------- | ------ |
| closeText  | 自定义关闭按钮，替换默认的 x 图标 | ReactNode  | `-`    |
| closable   | 默认不可关闭                      | boolean    | false  |
| afterClose | 关闭后触发的回调函数              | () => void | -      |



```jsx
<Alert message="info tips" closable closeText="关闭" afterClose={() => console.log('closed')} />
```





### 操作

提示中可以配置操作按钮，在尾部展示（关闭按钮之前）。文本消息不超过右侧操作按钮区域，间隔一定间距换行。

![](https://lg-gxg3fdgs-1254198195.cos.ap-shanghai.myqcloud.com/images/20201226221121.png)

*AntD Alert 操作示例*

消息内容中也可以自定义文本链接，点击会跳转到其它页面。

![](https://lg-gxg3fdgs-1254198195.cos.ap-shanghai.myqcloud.com/images/20201226221131.png)

*People Design 常驻提示操作示例*



#### API 设计

| 参数   | 说明         | 类型      | 默认值 |
| ------ | ------------ | --------- | ------ |
| action | 自定义操作项 | ReactNode | `-`    |



```jsx
<Alert
  message="Success Tips"
  type="success"
  action={
    <button onClick={() => console.log('Action clicked.')}>UNDO</button>
  }
/>
```





## Toast 全局提示

> 有些组件库命名为 Message、Snackbar 等

全局提示是操作后的轻量级短时反馈提示，不会打断用户操作。



### 使用场景

用于展示操作后系统对该行为作出的反馈，如成功、警示、错误、提示信息等。



### 展示交互

位于页面顶部中央，距离顶部有固定间距。消息框宽度随内容自适应，超过最大宽度后文本换行。不随页面滚动，有浮层阴影效果。

展示一段时间后自动消息（可以设置不自动关闭），对内容干扰降到最低。

当不同操作触发了多条全局提示，按照时间先后顺序在页面上方中央依次堆叠出现，将未消失提示推至下方。

消息出现（从上至下掉落或从小放大）和消失（相反动作）都会有过渡。

当指针悬浮在 Toast 上时，认为用户此时正在关注此提示，所以清除 Toast 自动关闭的定时器，避免 Toast 突然在用户关注焦点中消失；当指针离开 Toast 后，重新创建 Toast 的关闭定时器。



### 和 Alert 的对比

- Alert 作为局部组件嵌入；Toast 页面最上层浮动展示，暗示全局性；
- Alert 跟随容器一起静态出现； Toast 在某个操作后动态浮现；
- Alert 默认常驻；Toast 出现后默认自动消失；
- Alert 在模板中声明式引用；Toast 在全局 JavaScript 作用域内命令式静态方法调用，可以看作是工具方法，在事件中触发；
- Alert 每个实例独立，配置不共享；Toast 实例间共享全局配置，可以统一管理，配置默认的样式和行为；
- Alert 提供 action prop；Toast 为轻交互，不提供额外操作，但可以在 content 中自行嵌入（基本约束，灵活定制）；
- Alert 提供 title prop；Toast 为轻提示，一般不需要标题，但同样可以在 content 中自行嵌入。



### 提示类型

Toast 除了 Alert 的四种提示类型外，还有一种 loading 类型：

- **加载中 loading**：用于展示当前操作正在处理中或者内容正在加载中

![](https://lg-gxg3fdgs-1254198195.cos.ap-shanghai.myqcloud.com/images/20201226221143.png)

*AntD message 示例*



### 变体

> 同 Alert



### 静态方法 API

> API 基本参照 AntD message

Toast 作为特殊的工具组件，提供每个提示类型的静态方法调用：

- `Toast.success(content, [duration], onClose)`
- `Toast.error(content, [duration], onClose)`
- `Toast.info(content, [duration], onClose)`
- `Toast.warning(content, [duration], onClose)`
- `Toast.loading(content, [duration], onClose)`

| 参数     | 说明                                        | 类型                | 默认值 |
| :------- | :------------------------------------------ | :------------------ | :----- |
| content  | 提示内容                                    | ReactNode \| config | -      |
| duration | 自动关闭的延时，单位秒。设为 0 时不自动关闭 | number              | 3      |
| onClose  | 关闭时触发的回调函数                        | function            | -      |



#### 返回 promise

静态方法调用后返回 promise，可以使用 `.then()`、 `await` 语法，以同步的写法在之后执行 Toast 关闭后的逻辑：

```
Toast[level](content, [duration]).then(afterClose)
```

```js
await Toast[level](content, [duration])
afterClose()
```



#### 实现思路

```js
function openToast(configs) {
	return new Promise(resolve => {
    new Toast({ ...configs, onClose={() => {
      resolve(true)
      configs.onClose?.()
    }}})
  })
}

function Toast() {}

['success', 'error', 'info', 'warning', 'loading'].forEach(type => {
  Toast[type] = (...configs) => openToast({ type, ...configs })
})
```





### 配置对象传参

除了上面的简略参数外，还可以使用对象的形式传递参数，用来定制更具体的 Toast：

- `Toast.success(config)`
- `Toast.error(config)`
- `Toast.info(config)`
- `Toast.warning(config)`
- `Toast.loading(config)`

`config` 对象属性如下：

| 参数      | 说明                                                         | 类型             | 默认值 |
| :-------- | :----------------------------------------------------------- | :--------------- | :----- |
| className | 自定义 CSS class                                             | string           | -      |
| content   | 提示内容                                                     | ReactNode        | -      |
| duration  | 自动关闭的延时，单位秒。设为 0 时不自动关闭，会展示关闭图标，也可以通过 `Toast.destroy(key)` 关闭 | number           | 3      |
| icon      | 自定义图标                                                   | ReactNode        | -      |
| key       | 当前提示的唯一标志                                           | string \| number | -      |
| style     | 自定义内联样式                                               | CSSProperties    | -      |
| onClose   | 关闭时触发的回调函数                                         | function         | -      |



### 全局配置

提供 `Toast.config(options)` 配置 Toast 的默认样式和行为，在引入 Toast 的入口文件中配置。`options` 属性如下：

| 参数         | 说明                                           | 类型              | 默认值              |
| :----------- | :--------------------------------------------- | :---------------- | :------------------ |
| duration     | 默认自动关闭延时，单位秒                       | number            | 3                   |
| getContainer | 配置渲染节点的输出位置                         | () => HTMLElement | () => document.body |
| maxCount     | 最大显示数, 超过限制时，最早的消息会被自动关闭 | number            | -                   |
| top          | 消息距离顶部的位置                             | number            | 24                  |



### 关闭提示

- `Toast.destroy()` 关闭当前所有的 Toast；
- `Toast.destroy(key)` 如果在 `config` 对象中配置了一个 Toast 的唯一 `key`，可以传递 `key` 参数关闭指定的 Toast。



### 通过 key 更新 Toast

可以通过唯一的 `key` 来动态更新 Toast 内容

```js
const key = 'updatable'

const openMessage = () => {
  Toast.loading({ content: 'Loading...', key });
  setTimeout(() => {
    message.success({ content: 'Loaded!', key, duration: 2 });
  }, 1000);
};
```



### 通过 Hooks 获取上下文

当需要 context 信息（例如父级组件向下传递的共享状态）时，可以通过 `Toast.useToast()` 创建支持读取 context 的 `contextHolder`，方法会返回 `api` 实体以及 `contextHolder` 节点。将其插入到需要获取 context 的位置即可：

```jsx
const Context = React.createContext();

function Demo() {
  const [toast, contextHolder] = Toast.useToast();
  const info = () => {
    toast.info({
      // Access the closest Provider for this context above in the tree
      content: <Context.Consumer>{({ name }) => `Hello, ${name}!`}</Context.Consumer>,
      duration: 1,
    });
  };

  return (
    <Context.Provider value={{ name: 'Toast hook' }}>
      {/* Receive the upper context value */}
      {contextHolder}
      <button onClick={info}>
        Display normal message
      </button>
    </Context.Provider>
  );
}
```



#### 实现思路

直接调用 Toast 方法，会动态创建新的 Toast 实例，默认渲染在 body 底部的容器中，此时在 Toast 中访问的是容器的 context。

要在 Toast content 内访问父级组件的 context，需要把 content 挂载到 `<Context.Provider>` 的 React 子树中，这样 content 中的 `<Context.Consumer>` 才能接收到绑定的 `value`。但是如果直接把 Toast 渲染到父级组件下，即使是 `fixed` 定位，多少也会受到父组件的干扰（样式、生命周期等），所以为了避免父组件的影响，同时自动应用过渡动画，需要在一个专用的 Toast 容器环境中让 Toast 和普通调用一样渲染，即默认的 body 底部容器。

跨组件树异地渲染正是 `React.createPortal()` 的用武之地。

通过 `Toast.useToast()` 返回的 `toast` api，在事件中触发提示后，会动态创建 Toast 实例（作为 contextHolder 插入到父组件子树），Toast render 中判断是 hook 触发则利用 `React.createPortal(<ToastContent />, bodyContainer)` 把 **ToastContent** 渲染到默认容器中，这样 ToastContent 即成为了 `<Context.Provider>` 子树中的组件，实现访问父级 context，又能在默认容器中正常渲染。

使用代码简略说明：

```js
function useToast() {
	const [element, setElement] = useState(null)

  function toast(props) {
    // Toast 容器已经挂载到 body 底部
    const containerCallback = container => {
      // 动态创建 Toast 实例
      const toastElem = <Toast container={container} />
      setElement(toastElem)
    }
    addToast({ ...props, containerCallback })
  }
  return [toast, element]
}

function addToast({ containerCallback }) {
  if (containerCallback) {
    // hook 调用，在 body 底部动态创建 Toast 容器，mount 后传递 DOM 节点
    return (
      <div ref={div => div && containerCallback(div)} />
    )
  } else {
    return <Toast />
  }
}

function Toast({ container }) {
  if (container) {
    // hook 调用，不渲染到父组件下
    return React.createPortal(<ToastContent />, container)
  } else {
    // 普通调用，正常渲染
    return <ToastContent />
  }
}
```



## 总结

通过设计 Alert 和 Toast 的展示交互和 API，能够从普通用户、组件使用方、组件提供方的角度考虑如何做好一个通用组件；不同的组件库有不同的风格，但沉淀出来的优秀实践都大同小异。

有了设计方案后才考虑怎么优雅地实现这些功能，关联组件库、解决兼容问题、处理各种细节等，逐步打磨成熟。



## 参考

[Ant Design Message 全局提示](https://ant-design.gitee.io/components/message-cn/)

[Ant Design Alert 警告提示](https://ant-design.gitee.io/components/alert-cn/)

[People Design 全局提示](https://people-design.bytedance.net/docs/component/coponent-feedback-global-prompt-cn)

[People Design 常驻提示](https://people-design.bytedance.net/docs/component/coponent-feedback-warning-cn)

[Material-UI Alert](https://material-ui.com/components/alert/)

[Material-UI Snackbar](https://material-ui.com/components/snackbars/)




