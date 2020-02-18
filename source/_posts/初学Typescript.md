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

> ## 函数

> ## 类

> ## 泛型

> ## 模块

> ## 命名空间

> ## 装饰器

[1]: https://typescript.bootcss.com/
[2]: https://www.typescriptlang.org/docs/home.html