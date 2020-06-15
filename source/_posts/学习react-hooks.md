---
title: 学习react hooks
date: 2020-03-11 18:09:41
tags: react
categories: react
---

记录一下学习React Hooks的相关知识。

<!-- more -->

React的函数组件，在早期必须是纯函数，不能包含状态，也不支持生命周期方法。而React Hooks则加强了函数组件的功能，函数组件可以拥有自己的状态，也可以实现生命周期函数。

## useState()

`useState()`用于为函数引入状态（state）。

  ```js
  import React, { useState } from 'react'
  function App() {
    const [info, setInfo] = useState({
      count: 0,
      name: 'jack'
    })

    return (
      <div>
        <span>Count: {info.count}</span>
        <button onClick={() => { setInfo({ ...info, count: info.count + 1 }) }}>+</button>
      </div>
    )
  }
  ```

`useState()`的参数是状态的初始值，返回数组，分别是指向当前状态的变量和更新状态的函数。这个方法约定以`set`开头。

注意：这里要注意改变状态的方法和类组件中setState的区别。setState是非覆盖式更新，即只需传入你要改变的状态。useState的更新状态方法是覆盖式更新，需要传入全部的值，例如上例用解构的方式更新状态。

## useEffect()

`useEffect()`用于处理函数组件的副作用的操作，最常见的是想服务器发送数据请求。放在`componentDidMount`中的代码，可以放在`useEffect()`中。

  ```js
  import React, { useState } from 'react'
  function App() {
    const [count, setCount] = useState(0)
    useEffect(() => {
      console.log(`点击次数是${count}`)
    }, [count])
    return (
      <div>
        <span>Count: {info.count}</span>
        <button onClick={() => { setCount(count + 1) }}>+</button>
      </div>
    )
  }
  ```
`useEffect()`的第一个参数是个函数，用来执行副作用操作，比如请求数据。第二个参数是个数组，可以理解为副作用监听的状态数组，当对应状态发生变化就会触发第一个参数执行。

第一个参数中的函数如果有返回值（例如返回一个函数），那么这个返回值会在组件销毁前执行。

第二个参数如果不传，相当于副作用函数监听所有状态。

第二个参数如果为`[]`，即空数组，相当于副作用函数不监听任何状态，副作用函数只在组件初始化时执行。

我们第一个参数传入的函数中有返回值，第二个参数传入`[]`，就可以实现`componentDidMount`和`componentWillUnmount`

  ```js
  import React, { useState } from 'react'
  function App() {
    const [count, setCount] = useState(0)
    useEffect(() => {
      console.log('组件挂载时打印')
      return () => {
        console.log('组件销毁前打印')
      }
    }, [])
    return (
      <div>
        <span>Count: {info.count}</span>
        <button onClick={() => { setCount(count + 1) }}>+</button>
      </div>
    )
  }
  ```

我们不传入第二个参数，可以实现`componentDidUpdate`

  ```js
  import React, { useState } from 'react'
  function App() {
    const [count, setCount] = useState(0)
    const [num, setNum] = useState(0)
    useEffect(() => {
      console.log('num和count任意一个变化时都会打印')
    })
    return (
      <div>
        <span>Count: {info.count}</span>
        <button onClick={() => { setCount(count + 1) }}>+</button>
        <button onClick={() => { setCount(num + 1) }}>num+</button>
      </div>
    )
  }
  ```
  仔细学习[这篇文章](https://overreacted.io/a-complete-guide-to-useeffect/)，才是学习`useEffect`的最好途径，以上的总结只是一些tricks，而且还不完全对。

## useContext()

需要先熟悉[context API][3]，才能更好理解`useContext()`

## useReducer
当函数组件中有多个状态（基于相同的逻辑）时，可以用useReducer进行state的管理。

[参考文章](https://medium.com/trabe/react-usereducer-hook-2b1331bb768)

## useCallback

## useMemo

## useRef

## useImperativeHandle

## useLayoutEffect

## useDebugValue

## 自定义hooks

自定义Hook的作用是提取组件间的可重用逻辑到函数中，这个函数的函数名以use开头。所以说自定义hook就是一个普通的函数。
试想，如果想在两个或多个函数中使用一段相同的逻辑，我们通常的做法是将这段逻辑封装成一个函数。而组件和hook都是函数，同样适用这种方式。


-----------------------------------分割线-----------------------------

[react作者介绍Hooks](https://dev.to/dan_abramov/making-sense-of-react-hooks-2eib)

[比较详细的的介绍了Hooks的诞生背景](https://www.robinwieruch.de/react-hooks)，介绍了Hooks带来的变化。








react Hooks学习的文章：

[十个案例学会 React Hooks][1]

[看完这篇，你也能把 React Hooks 玩出花][2]

[阮一峰 React Hooks 入门教程][4]

[use Hooks](5)

[1]: https://github.com/happylindz/blog/issues/19
[2]: https://mp.weixin.qq.com/s/t6oGCZ_DEfIlykQD-wCjUg
[3]: https://zh-hans.reactjs.org/docs/context.html
[4]: https://www.ruanyifeng.com/blog/2019/09/react-hooks.html
[5]: https://usehooks.com/