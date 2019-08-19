+++
title = '什么时候使用 useMemo 和 useCallback'
date = 2019-08-19T21:18:32+08:00
categories = ["译文"]
tags = ["React", "useMemo", "useCallback"]
+++

> 原文：[When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback)

性能优化总是会有成本，但并不总是带来好处。我们来谈谈 `useMemo` 和 `useCallback` 的成本和收益。

<!--more-->


这里是一个糖果提售货机：

![](https://lg-gxg3fdgs-1254198195.cos.ap-shanghai.myqcloud.com/blog/20190730212055.png)

（原文中可点击交互，点击 "grab" 按钮后“提取”对应的糖果，对应项会从页面删除；全部提取完后会出现 "refill" 按钮，点击重置所有糖果）

以下是它的实现方式：

```react
function CandyDispenser() {
  const initialCandies = ['snickers', 'skittles', 'twix', 'milky way']
  const [candies, setCandies] = React.useState(initialCandies)
  const dispense = candy => {
    setCandies(allCandies => allCandies.filter(c => c !== candy))
  }
  return (
    <div>
      <h1>Candy Dispenser</h1>
      <div>
        <div>Available Candy</div>
        {candies.length === 0 ? (
          <button onClick={() => setCandies(initialCandies)}>refill</button>
        ) : (
          <ul>
            {candies.map(candy => (
              <li key={candy}>
                <button onClick={() => dispense(candy)}>grab</button> {candy}
              </li>
            ))}
          </ul>
        )}
      </div>
    </div>
  )
}
```

现在我想问你一个问题，我希望你在继续之前好好想想。我要做一个改变，我想让你告诉我哪一个会有更好的性能特征。

我唯一要改变的是在 `React.useCallback` 里包裹 `dispense` 函数：

```react
const dispense = React.useCallback(candy => {
  setCandies(allCandies => allCandies.filter(c => c !== candy))
}, [])
```

这是**原来**的代码：

```react
const dispense = candy => {
  setCandies(allCandies => allCandies.filter(c => c !== candy))
}
```

所以我的问题是，在这个特定的例子中，哪一个对性能更好？原来的还是 `useCallback`？

如果你选择的是 `useCallback`，再好好思考下。



正确答案是：使用原来的代码性能会更好😉



## 为什么 useCallback 更糟糕？！

我们听到很多你应该使用 `React.useCallback` 来提高性能，并且“内联函数可能会对性能造成问题”，那么不使用`callCallback` 是如何变得更好的？

从我们的具体例子中退后一步，甚至从React那里考虑一下：**执行的每行代码都有成本**。让我稍微重构一下 `useCallback` 的例子来更清楚地说明事情（没有实际的改变，只是移动下代码）：

```react
const dispense = candy => {
  setCandies(allCandies => allCandies.filter(c => c !== candy))
}
const dispenseCallback = React.useCallback(dispense, [])
```

这是原来的：

```react
const dispense = candy => {
  setCandies(allCandies => allCandies.filter(c => c !== candy))
}
```

注意到了吗？让我们看一下 diff：

```diff
const dispense = candy => {
    setCandies(allCandies => allCandies.filter(c => c !== candy))
  }
+ const dispenseCallback = React.useCallback(dispense, [])
```

是的，除了`useCallback`版本做了更多的工作之外，它们完全相同。 我们不仅需要定义函数，还要定义一个数组（`[]`）并调用 `React.useCallback`，它本身会设置属性和运行逻辑表达式等。

因此，在这两种情况下，JavaScript 必须在每次渲染中为函数定义分配内存，并且根据 `useCallback` 的实现方式，你可能会获得更多的函数定义内存分配（实际情况并非如此，但重点还在这里）。这就是我试图通过我的 Twitter 民意调查得到的：

![](https://lg-gxg3fdgs-1254198195.cos.ap-shanghai.myqcloud.com/blog/20190730215347.png)

我还想提一下，在组件的第二次渲染中，原来的 `dispense` 函数被垃圾收集（释放内存空间），然后创建一个新的 `dispense` 函数。 但是使用 `useCallback` 时，原来的 `dispense` 函数不会被垃圾收集，并且会创建一个新的 `dispense` 函数，所以从内存的角度来看，这会变得更糟。

作为一个相关的说明，如果你有其它依赖，那么React很可能会挂起对前面函数的引用，因为 **memoization** 通常意味着我们保留旧值的副本，以便在我们获得与先前给出的相同依赖的情况下返回。 特别聪明的你会注意到，这意味着React还必须挂在对这个等式检查依赖项的引用上（由于闭包，这种情况可能会偶然发生，但无论如何它都值得一提）。



## useMemo 虽然不同，但却是相似的？

`useMemo` 类似于 `useCallback`，除了它允许你将 **memoization** 应用于任何值类型（不仅仅是函数）。 它通过接受一个返回值的函数来实现这一点，然后**只在**需要检索值时调用该函数（通常这只有在每次渲染中依赖项数组中的元素发生变化时才会发生一次）。

所以，如果我不想在每次渲染时初始化那个 `initialCandies` 数组，我可以做这个改变：

```diff
- const initialCandies = ['snickers', 'skittles', 'twix', 'milky way']
+ const initialCandies = React.useMemo(
+  () => ['snickers', 'skittles', 'twix', 'milky way'],
+  [],
+ )
```

我可以避免那个问题，但是节省的成本是如此之小，以至于换来使代码更加复杂的成本是不值得的。实际上，这里使用`useMemo` 也可能会更糟，因为我们再次进行了函数调用，并且代码会执行属性赋值等。

在这个特定的场景中，更好的方法是进行这个更改：

```diff
+ const initialCandies = ['snickers', 'skittles', 'twix', 'milky way']
  function CandyDispenser() {
-   const initialCandies = ['snickers', 'skittles', 'twix', 'milky way']
    const [candies, setCandies] = React.useState(initialCandies)
```

但有时你没有那么奢侈，因为这个值要么来源于 `props` 或者函数体内初始化的其它变量。

关键是这两种方式无关紧要，优化这些代码的好处是如此微不足道，以至于你可以更好地花时间来改善产品质量。



## 重点是什么？

重点是：

**性能优化不是免费的。 它们总是带来成本，但这并不总是带来好处来抵消成本。**

因此，负责任地进行优化。



## 所以我应该什么时候使用 useMemo 和 useCallback？

这两个 hooks 内置于 React 都有特别的原因：

1. 引用相等
2. 昂贵的计算



## 引用相等

如果你是 JavaScript 或者编程新手，你很快就会明白为什么会这样：

```js
true === true // true
false === false // true
1 === 1 // true
'a' === 'a' // true

{} === {} // false
[] === [] // false
() => {} === () => {} // false

const z = {}
z === z // true

// NOTE: React actually uses Object.is, but it's very similar to ===
```

我不打算深入研究这个问题，但是当你在React函数组件中定义一个对象时，它跟上次定义的相同对象，引用是不一样的（即使它具有所有相同值和相同属性），这足以说明这个问题。

在React中，有两种情况下引用相等很重要，让我们一个个地来看。



### 依赖列表

让我们来回顾一个例子。

> 警告，你将看到一些人为故意设计的代码。请不要吹毛求疵，只关注概念，谢谢。

```react
function Foo({bar, baz}) {
  const options = {bar, baz}
  React.useEffect(() => {
    buzz(options)
  }, [options]) // we want this to re-run if bar or baz change
  return <div>foobar</div>
}

function Blub() {
  return <Foo bar="bar value" baz={3} />
}
```

这里有问题的原因是因为 `useEffect` 将对每次渲染中对 `options` 进行引用相等性检查，并且由于JavaScript的工作方式，每次渲染 `options` 都是新的，所以当React测试 `options` 是否在渲染之间发生变化时，它将始终计算为 `true`，意味着每次渲染后都会调用 `useEffect` 回调，而不是仅在 `bar` 和 `baz` 更改时调用。

我们可以做两件事来解决这个问题：

```react
// option 1
function Foo({bar, baz}) {
  React.useEffect(() => {
    const options = {bar, baz}
    buzz(options)
  }, [bar, baz]) // we want this to re-run if bar or baz change
  return <div>foobar</div>
}
```

这是个不错的选择，如果这是真的，我就会这么做。

但是有一种情况下：如果 `bar` 或者 `baz` 是（非原始值）对象、数组、函数等，这不是一个实际的解决方案：

```react
function Blub() {
  const bar = () => {}
  const baz = [1, 2, 3]
  return <Foo bar={bar} baz={baz} />
}
```

这正是 `useCallback` 和 `useMemo` 存在的原因。你可以这样解决这个问题（现在都放一起了）：

```react
function Foo({bar, baz}) {
  React.useEffect(() => {
    const options = {bar, baz}
    buzz(options)
  }, [bar, baz])
  return <div>foobar</div>
}

function Blub() {
  const bar = React.useCallback(() => {}, [])
  const baz = React.useMemo(() => [1, 2, 3], [])
  return <Foo bar={bar} baz={baz} />
}
```

> 请注意，同样的事情也适用于传递给 `useEffect`, `useLayoutEffect`*,* `useCallback`, 和  `useMemo` 的依赖项数组。



### React.memo

> 警告，你将看到一些人为故意设计的代码。请不要吹毛求疵，只关注概念，谢谢。

看看这个：

```react
function CountButton({onClick, count}) {
  return <button onClick={onClick}>{count}</button>
}

function DualCounter() {
  const [count1, setCount1] = React.useState(0)
  const increment1 = () => setCount1(c => c + 1)

  const [count2, setCount2] = React.useState(0)
  const increment2 = () => setCount2(c => c + 1)

  return (
    <>
      <CountButton count={count1} onClick={increment1} />
      <CountButton count={count2} onClick={increment2} />
    </>
  )
}
```

每次单击其中任何一个按钮时，`DualCounter` 的状态都会发生变化，因此会重新渲染，然后重新渲染两个`CountButton`。 但是，实际上只需要重新渲染被点击的那个按钮吧？因此，如果你点击第一个按钮，则第二个也会重新渲染，但没有任何变化，我们称之为“不必要的重新渲染”。

**大多数时候，你不需要考虑去优化不必要的重新渲染**。React是非常快的，我能想到你可以利用时间去做很多事情，比起做这些类似的优化要好得多。事实上，我展示给你看的代码很少有优化的需求，以至于我在 PayPal 工作的3年里从未需要这样做，甚至在我使用 React 更长的时间里。

然而，有些情况下渲染可能会花费大量时间（比如重交互的图表、动画等）。多亏 React 的实用性，有一个逃生舱（escape hatch）：

```react
const CountButton = React.memo(function CountButton({onClick, count}) {
  return <button onClick={onClick}>{count}</button>
})
```

现在 React 只会当 `props` 改变时会重新渲染 `CountButton`！  但我们还没有完成，还记得引用相等吗？在 `DualCounter` 组件中，我们组件函数里定义了 `increment1` 和 `increment2` 函数，这意味着每次 `DualCounter` 重新渲染，那些函数会新创建，因此 React 无论如何会重新渲染两个 `CountButton`。

所以这是 `useCallback` 和 `useMemo` 能派上用场的另外一个场景：

```react
const CountButton = React.memo(function CountButton({onClick, count}) {
  return <button onClick={onClick}>{count}</button>
})

function DualCounter() {
  const [count1, setCount1] = React.useState(0)
  const increment1 = React.useCallback(() => setCount1(c => c + 1), [])

  const [count2, setCount2] = React.useState(0)
  const increment2 = React.useCallback(() => setCount2(c => c + 1), [])

  return (
    <>
      <CountButton count={count1} onClick={increment1} />
      <CountButton count={count2} onClick={increment2} />
    </>
  )
}
```

现在我们可以避免 `CountButton` 的所谓“不必要的重新渲染”。



我想重申下，**在没有测量前**，强烈建议不要使用 `React.momo` （或者它的朋友 `PureComponent` 和 `shouldComponentUpdate`），因为优化总会带来成本，并且你需要确保知道会有多少成本和收益，这样你才能决定在你的案例中它是否能真的有帮助（而不是有害的）。正如我们上面所说的那样，**一直保持正确是一件很困难的事情，所以你可能无法获得任何好处**。





## 昂贵的计算

这是 `useMemo` 内置于 React 的另一个原因（注意这个不适用于 `useCallback`）。`useMemo` 的好处是你可以采用如下值：

```js
const a = {b: props.b}

```

然后惰性获取：

```js
const a = React.useMemo(() => ({b: props.b}), [props.b])

```

这对于上面的情况并不是很有用，但是想象一下你有一个计算成本很高的同步计算值的函数（我的意思是有多少应用真实地需要 [像这样计算素数](https://developer.mozilla.org/en-US/docs/Tools/Performance/Scenarios/Intensive_JavaScript)，但这就是一个例子）：

```react
function RenderPrimes({iterations, multiplier}) {
  const primes = calculatePrimes(iterations, multiplier)
  return <div>Primes! {primes}</div>
}

```

使用正确的 `iterations` 或 `multiplier` 可能会非常缓慢，而且你没有太多可以特别做的事情。你不能自动地用户的硬件更快，但是你可以这样做，这样你就不必连续两次计算相同的值，这就是 `useMemo` 为你所做的：

```react
function RenderPrimes({iterations, multiplier}) {
  const primes = React.useMemo(() => calculatePrimes(iterations, multiplier), [
    iterations,
    multiplier,
  ])
  return <div>Primes! {primes}</div>
}
```

可以这样做的原因是，即使你在每次渲染时定义了计算素数的函数（非常快），React只在需要值时才调用该函数。 除此之外，React还会在给定输入的情况下存储先前的值，并在给定跟之前相同输入的情况下返回先前的值。 这是 **memoization** 在起作用。



## 总结

最后，我想说，每个抽象(和性能优化)都是有代价的。应用 [AHA 编程原则](https://kentcdodds.com/blog/aha-programming)，直到确实需要抽象或优化时才去做，这样可以避免承担成本而不会获得收益的情况。

具体来说，`useCallback` 和 `useMemo`的成本是：对于你的同事来说，你使代码更复杂了；你可能在依赖项数组中犯了一个错误，并且你可能通过调用内置的 hook、并防止依赖项和 memoized 值被垃圾收集，而使性能变差。如果你获得了必要的性能收益，那么这些成本都是值得承担的，但**最好先测量一下**。



相关阅读：

- React FAQ: ["Are Hooks slow because of creating functions in render?"](https://reactjs.org/docs/hooks-faq.html#are-hooks-slow-because-of-creating-functions-in-render)
- [Ryan Florence](https://twitter.com/ryanflorence): [React, Inline Functions, and Performance](https://reacttraining.com/blog/react-inline-functions-and-performance)



