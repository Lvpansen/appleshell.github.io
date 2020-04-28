---
title: 学习react-typescript-cheatsheet
date: 2020-04-05 16:23:46
tags: typescript
categories: typescript & react
---

学习如何把react和typescript结合起来使用，[教程地址](https://github.com/typescript-cheatsheets/react-typescript-cheatsheet)，记录学习的知识点。

<!-- more -->
[React中使用Typescript官方文档](https://reactjs.org/docs/static-type-checking.html#typescript)

[react typescript实践](https://blog.bitsrc.io/react-typescript-cheetsheet-2b6fa2cecfe2)


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

1. useState

    - 大都数情况下不用专门标记类型，类型推断可以运行的很好
    ```ts
    const [val, toggle] = React.useState(false); // val的类型被推断为boolean，toggle只接受boolean类型的值
    ```

    - 当你需要一个复杂的推断类型，并且需要在其他地方复用时，你可能会显式的去定义这个类型（types/interfaces）,然后导出它们。通过使用`typeof`，就不用这么麻烦。
    ```ts
    const [state, setState] = React.useState({
      foo: 1,
      bar: 2,
    }); // state的类型被推断为{foo: number, bar: numbr}

    const someMethod = (obj: typeof state) => {
      // 使用state的类型，虽然它的类型是被推断出来的
      setState(obj)
    }
    ```

2. useEffect

    使用useEffect时，要注意的是其中的函数return的值只能使一个函数或者`undefined`，否则Typescript和React都会提示你。使用箭头函数的时候要特别注意。
    ```ts
    function DelayedEffect(props: { timerMs: number }) {
      const { timerMs } = props;
      // setTimeout隐式的返回一个数字，所以这里的写法有问题。可以给箭头函数加上大括号来解决。
      useEffect(
        () => 
          setTimeout(() => {
            /* do stuff */
          }, timerMs),
        [timerMs]
      );
      return null
    }
    ```

3. useRef

    - 使用useRef，当创建的ref容器没有初始值时有两种方式：
    ```ts
    const ref1 = useRef<HTMLElement>(null!);
    const ref2 = useRef<HTMLElement | null>(null);
    ```
    第一种方式会使`ref1.current`是只读的，这个会由React进行操作（因为React会设置current的值）

    第二种方式，`ref2.current`是可变的（mutable），由你自己操作。

    - 一个例子
    ```tsx
    function TextInputWithFocusButton() {
      // 初始值是null，但告诉Typescript我们要的是HTMLInputElement。
      const inputEl = React.useRef<HTMLInputElement>(null);
      const onButtonClick = () => {
        // 需要检查inputEl和current是否存在，一旦current存在，它的类型是HTMLIpuntElement，因此就有focus方法
        if (inputEl && inputEl.current) {
          inputEl.current.focus();
        }
      };
      return (
        <>
        // inputEl只能用在input标签上
          <input ref={inputEl} type="text" />
          <button onClick={onButtonClick}>Focus the input</button>
        </>
      )
    }
    ```
4. useReducer

  使用[Discriminate Unions](https://www.typescriptlang.org/docs/handbook/advanced-types.html#discriminated-unions)来给reducer的action参数标记类型。

  ```tsx
  interface SetOne {
    type: 'SET_ONE',
    payload: string
  };
  interface SetTwo {
    type: 'SET_TWO',
    payload: number
  };
  type AppState = {};
  type Action = SetOne | SetTwo;

  exprot function reducer(state: AppState, action:Action): AppState {
    switch(action.type) {
      case 'SET_ONE':
        return {
          ...state,
          one: action.payload, // payload is string
        };
      case 'SET_TWO':
        return {
          ...state,
          two: action.payload, // payload is number
        };
      default:
        return state
    }
  }
  ```

5. 自定义Hook

    - 如果你在自定义hook总返回了一个数组，而且想避免使用推断类型（因为Typescript会推断出联合类型），那么可以使用const断言。
    ```ts
    export function useLoading() {
      const [isLoading, setState] = React.useState(false);
      const load (aPromise: Promise<any>) => {
        setState(true);
        return aPromise.finally(() => setState(false));
      };
      return [isLoading, load] as const; // 正确的推断出[boolean, typeof load]，而不是(boolean | typeof load)[]
    }
    ```

## 类组件（Class Components）
  1. 在Typescript中，React.Component通过设置prop和state的类型来标记类型，即`React.Component<PropType, StateType>`

  2. 类组件中的方法和普通的函数的定义没有区别，记住被函数的每个参数标记类型

  3. 类里面的实例属性可以先定义，并标记类类型，但可以不赋值。
  ```tsx
  class App extends React.Compoent<{message: sting}> {
    pointer: number; // 可以先不赋值
    componentDidMount() {
      this.pointer = 3
    }
    render() {
      return (
        <div>
          {this.props.message} and {this.pointer}
        </div>
      )
    }
  }
  ```

## Type and Interface

在Typescript中，type和interface是不同的，但是用法很相似。下面是使用场景：

  - 当创建一个库或者第三方库的外围类型，使用interface来定义公共的API的类型。
  - React组件的props和state的类型，可以使用type定义。

type适合定义联合类型（例如：`type Mytype = TypeA | TypeB`），而interface适合定义字典形式的类型，然后被实现或扩展（inplementling or extending）

下图是两者之间一些qubie
![comparison between type and interface](https://camo.githubusercontent.com/f00cd1e1d40c197e5cdb82c383952241d7e0dc10/68747470733a2f2f7062732e7477696d672e636f6d2f6d656469612f4477562d6f4f7358634149637432712e6a7067)

## type定义props的示例
```ts
type AppProps = {
  message: sting;
  count: number;
  disabled: boolean;
  /** 数组类型 */
  names: string[];
  /** 具体的字符串精确的知名字符串类型的值，通过联合类型将他们组合起来 */
  status: 'waiting' | 'success';
  /** any object as long as yong don't use its properties (not common) */
  obj: object;
  obj2: {}; // 和object效果一样
  /** 定义了属性的对象（推荐） */
  obj3: {
    id: string;
    title: string
  };
  /** 对象数组（常见） */
  objArr: {
    id: string;
    title: string;
  }[];
  /** any function as long as you don't invoke it (not recommended) */
  onSomething: Function;
  /** 没有返回值的函数（很常见） */
  onClick: () => void;
  /** function with named prop (很常见) */
  onChange: (id: number) => void;
  /** 使用事件的函数（很常见） */
  onClick(event: React.MouseEvent<HTMLButtonElement>): void;
  /** 可选择属性（很常见） */
  optional?: OptionsalType;
}
```

## 常见的React属性类型定义示例
```ts
export declare interface AppProps {
  children1: JSX.Element; //不推荐，没有包含数组形式
  children1: JSX.Element | JSX.Element[]; // 也不太好，不能接受字符串
  children3: React.ReactChildren; // 不太合适
  children4: React.ReactChild[]; // 比上面好一点
  children: React.ReactNode; // 做好的方式，接受任何值
  functionChildren: (name: string) => React.ReactNode; // 推荐，函数作为渲染属性
  style?: React.CSSProperties;
  onChange?: React.FormEventHandler<HTMLInputElement>; // 表单事件
  props: Props & React.PropsWithoutRef<JSX.IntrinsicElements["button"]>; // to impersonate all the props of a button element without its ref
}
```

## 表单事件
1. 表单事件定义在行内，可以直接使用类型推断和上下文类型
```ts
const el = (
  <button
    onClick={(event) => {
      /*...*/
    }}
  />
)
```
2. 如果要单独定义事件函数，可以像下面这样
```ts
onChange = (e: React.FormEvent<HTMLInputElement>): void => {
  this.setState({ text: e.currentTarget.value });
};
```
  也可以像下面这样，给函数变量定义类型
```ts
onChange: React.ChangeEventHandler<HTMLInputElement> = (e) => {
  this.setState({ text: e.currentTarget.value });
}
```
3. 上面第一种方式使用的是方法的推断类型`(e: React.FormEvent<HTMLElement>): void`，第二种方式使用的是`@tyoe/react`中提供的类型。
4. 如果你不关心event的类型，可以使用`React.SyntheticEvent`。