---
title: 初学Typescript
date: 2020-02-16 22:21:18
tags: Typescript
categories: Typescript
---

记录学习Typescript的思路大纲和要点。

<!-- more -->
[Typescritp中文手册][1]

[Typescritp官方网站][2]

> ## 基础类型和枚举

1. 布尔值

        ```js
        let isDone: boolean = false;
        ```

2. 数字

        ```js
        let num: number = 234;
        ```

3. 字符串

        ```js
        let name: string = 'Jack';
        ```

4. 数组。有两种方式可以定义数组。

    第一种，可以在元素类型后面接上上`[]`，表示由此类型元素组成的一个数组：

        ```js
        let list：number[] = [1, 2, 3];
        ```

    第二种，使用数组泛型，`Array<元素类型>`  

        ```js
        let list: Array<number> = [1, 2, 3]
        ```

5. 元组Tuple，表示一个已知元素数量和元素类型的数组，各元素的类型不必相同。

        ```js
        let x: [number, string];
        x = [1, 'hello']  // OK
        x = ['hello', 1]  // Error
        ```

    当访问越界的元素时，会报错。

        ```js
        x[2] = 'world'  // Error
        x[5].toString()  // Error
        ```

6. 枚举Enum
    枚举类型可以为一组数字类型的值赋予直观友好的名字

        ```js
        enum StatusCode {success, warning, error}
        let c: StatusCode = StatusCode.success
        ```

    默认情况下，枚举的值是从**0**开始给元素赋值的。也可以手动修改元素的赋值。

        ```js
        enum StatusCode {success = 1, warning, error}
        let c: StatusCode = StatusCode.warning  // 此时warning对应的值是2，依此类推。
        ```

    当然，也可以给每个元素全部按照需要全部赋值。
    通过枚举类型的数值可以获取对应的描述名字

        ```js
        enum StatusCode {success = 1, warning = 3, error = 5}
        let statusName = StatusCode[3]
        console.log(statusName)  // warning
        ```

7. Any
    当不确定变量的类型时，可以用any指定，表示任意类型。这种变量的值可能是动态的，比如用户输入或者第三方库。

        ```js
        let notSure:any = 4
        notSure = 'hello'
        notSure = true

        let arr: any[] = [1, true, 'tree']
        ```

8. Void
    void有点与any含义相反的意思，表示没有任何类型。当一个函数没有返回值时，其返回值类型就是viod

        ```js
        function noValue(): void {
          console.log('hello')
        }
        ```

    声明一个void类型的变量没啥用，只能给这个变量赋值null或者undefined。

9. Null和Undefined
    null和undefined两者各有属于自己的类型null和undefined。
    默认情况下，null和undefined是任何类型的子类型，可以赋值给number类型的变量。但是在严格模式下，null和undefined只能赋值给void和它们各自。

10. Never
    never表示的是那些永远不存在的值的类型。例如总是抛出异常或者根本就不会有返回值的函数表达式或者箭头函数表达式的返回值类型。
    never时任何类型的子类型，可以赋值给任何类型。但是没有类型是never的子类型，没有类型可以赋值给never类型（除了never类型本身）。

        ``` js
        // Function returning never must have unreachable end point
        function error():never {
          throw new Error('error')
        }

        function ee():never {
          throw new Error('fff')
        }
        function rr():void {
          console.log('sdf')
        }
        let a:string = ee()  // OK
        let c:string = rr() // Error, Type 'void' is not assignable to type 'string'
        ```

11. Object
    object表示非基础类型，即不是number、string、boolean、bigint、symbol、null或者undefined之外的任何类型。

        ```js
        declare function create(o: object | null): void

        create({ prop: 0 }) // OK
        create(null) // OK

        create(42) // Error
        create(false) // Error
        ```

12. Type assertions 类型断言
    当你清楚的知道一个变量具有比它现有类型更确切的类型时，可以使用断言。Typescript假设你已经进行了必要的检查。
    类型断言有两种方式：

    尖括号语法

        ```js
        let someValue: any = 'this is a string'
        let strLength = (<string>someValue).length
        ```

    as语法

        ```js
        let someValue: any = 'this is a string'
        let strLength = (someValue as string).length
        ```

    两种语法基本等价。当在JSX中使用Typescript时，只能使用as语法。

> ## 接口

TypeScript的核心原则之一就是对值所具备的形状（shape）进行类型检查。

接口命名这些类型，同时为自己的代码或者第三方的代码定义规范。

1. 定义一个接口

    ```js
    interface LabelValue {
        label: string
    }

    function printLabel(labelObj: LabelValue):void {
        console.log(lableObj.label)
    }

    printLabel({ label: 'hello' }) // Ok
    printLabel({ label: 'hello', size: 10}) // Error

    let myObj = { label: 'hello', size: 10}
    printLabel(myObj) // OK
    ```

2. 可选属性

    ```js
    interface Config{
        color?: string
        width?: number
    }

    ```

3. 只读属性

    ```js
    interface Config{
        readonly x: number
        readonly y: number
    }

    ```

4. 额外的属性检查

    ```js
    interface Config{
        color?: string
        width?: number
    }

    function createValue(config:Config) {
        console.log(config.color + config.width)
    }

    createValue({ width: 12, name: 'jack'}) // Error

    ```

    javascript中这段代码没问题，但TypeScript会认为这段代码有bug。

    对象字面量赋值给变量或者作为参数传递的时候，会被特殊对待，而且会经过**额外属性校验**。如果对象字面量存在任何‘目标类型’不包含的属性时，就会报错。

    ```js
    // error: 'name' not expected in type 'Config'
    createValue({ width: 12, name: 'jack'})
    
    ```

    绕开这些检查有几种方式：

    最简便的是类型断言

    ```js
    createValue({ width: 12, name: 'jack'} as Config)
    ```

    最好的方式是添加一个字符串索引签名，前提是这个对象可能有其他的额外属性。

    ```js
    interface Config{
        color?: string
        width?: number
        [propName: string]: any
    }
    ```

    最后一种方式是将这个对象赋给一个变量，因为这个变量不会经过额外属性检查

    ```js
    let myObj = { width: 12, name: 'jack'}
    createValue(myObj) // OK
    ```

5. 函数类型

    用接口表示函数类型，我们给接口定义一个调用签名（call signature）

    ```js
    interface SearchFn {
        (source: string, subString: string): boolean
    }

    let mySearch: SearchFn
    mySearch = function(source: string, subString: string) {
        let result = source.search(subString);
        return result > -1;
    }
    ```

    函数的参数会逐个进行检查，要求对应位置上的参数类型是兼容的。

6. 可索引类型

    可索引类型描述的是可以“通过索引获取”的类型，例如`a[10]`或`obj["name"]`
    可索引类型有一个索引签名，描述对象索引的类型，还有相应的索引返回值的类型。

    ```js
    interface StringArray {
      [index: number]: string
    }

    let myArr: StringArray = ['red', 'blue']
    let myStr: string = myArr[0]
    ```

    支持索引类型的签名有两种：数字和字符串。可以同时使用这两种类型的索引，但是**注意**，数字类型索引的返回值的类型必须是字符串类型索引的返回值的子类型。这是因为数字类型的索引，JavaScript会先把数字转成字符串，然后再去索引对象。即，用`100`去索引和用`"100"`去索引效果一样。

    ```js
    class Animal {
      name: string
    }
    class Dog extends Animal {
      breed: string
    }
    // Error: 数字字符串做索引时，可能会得到Animal类型，Animal类型不是Dog的子类型。
    interface Wrong {
      [x: number]: Animal
      [x: string]: Dog
    }
    // OK
    interface Wrong {
      [x: number]: Dog
      [x: string]: Animal
    }
    ```

    字符串类型的索引签名能够很好的描述dictionary模式，而且它会强制所有属性的类型与其返回值的类型匹配。

    ```js
    interface NumberDictionary {
        [index: string]: number
        length: number  // Ok
        name: string  // Error
    }

    // 索引签名是个联合类型时，其他属性的类型可以不同
    interface NumberDictionary {
        [index: string]: number | string
        length: number  // Ok
        name: string  // OK
    }
    ```
    可以给索引签名设置`readonly`，防止属性被赋值

    ```js
    interface ReadonlyStringArray {
        readonly [index: number]: string
    }
    let myArray: ReadonlyStringArray = ["Alice", "Bob"]
    myArray[2] = "Mallory"; // Error
    ```

7. 类类型
    
    类实现接口使用的关键字是`implements`。接口描述了类的公共部分，它不会检查类是否具有某些私有成员。

8. 接口继承

    像类一样，接口也可以继承，这样就可以提取可重用的接口模块。

    ```js
    interface Shape {
        color: string
    }

    interface Square extends Shape {
        length: number
    }

    let s: Square = { color: 'red', length: 12}
    ```

    还可以继承多个接口

    ```js
    interface Shape {
        color: string
    }

    interface Stroke {
        width: number
    }

    interface Square extends Shape, Stroke {
        length: number
    }

    let s: Square = { color: 'red', width: 34, length: 12}
    ```
9. 混合类型

10. 接口继承类

> ## 函数

1. 函数类型

2. 可选参数和默认参数

3. 剩余参数（rest）

4. this

5. 函数重载（overload）

> ## 类
> ## 泛型
> ## 模块
> ## 命名空间
> ## 装饰器

[1]: https://typescript.bootcss.com/
[2]: https://www.typescriptlang.org/docs/home.html
