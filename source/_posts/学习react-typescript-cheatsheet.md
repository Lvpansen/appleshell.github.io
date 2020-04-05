---
title: 学习react-typescript-cheatsheet
date: 2020-04-05 16:23:46
tags: typescript
categories: typescript & react
---

学习如何把react和typescript结合起来使用，[教程地址](https://github.com/typescript-cheatsheets/react-typescript-cheatsheet)，记录学习的知识点。

<!-- more -->


## 函数组件（Function Components）

1. 函数组件有两种写法：普通函数（normal functions）和`React.FuntionComponent`（简写为`React.FC`）。

  ```tsx
    type AppProps = {message: string}; /* could also use interface */

    // normal version
    const App = ({ message }: AppProps) => <div>{message}</div>

    // React.FC
    const App: React.FunctionComponent<AppProps> = ({ message }) => (<div>{message}</div>)
  ```
2. `React.FC`和普通函数写法的一些不同：

  - `React.FC`显式的说明返回类型，普通函数是隐式的。
  - `React.FC`对静态属性（例如`displayName`、`propTypes`、`defaultProps`）提供类型检查和自动补全（typechecking and autocomplete）
  - `React.FC`隐式定义了`children`属性（如下）。但是最好的能够显式的写出来。
  ```tsx
  const Title: React.FC<{ title: string }> = ({
    children,
    title
  }) => <div title={title}>{children}</div>
  ```
  - 将来`React.FC`可能会自动把`props`标记为只读（`readonly`），虽然当用解构的方式传毒props时这没有什么意义。

3. 注意：

  - 在使用条件渲染时，下面这种写法不允许
  ```tsx
  const MyConditionalComponent = ({shouldRender = false}) =>
    shouldRender ? <div /> : false; // don't do this is JS ether，js中也不要这样写
  const el = <MyConditionalComponent />; // 报错
  ```
  这是因为由于编译器的限制，函数组件不能返回除JSX表达式或者`null`之外的值。

- 如果确实需要返回其他React支持的类型，那就需要用类型断言。

## Hooks

1. 

## 类组件（Class Components）

## 标记默认属性类型（defauProps）

## Type and Interface

## 表单和事件

## Context