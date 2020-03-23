---
title: 初学Typescript
date: 2020-02-16 22:21:18
tags: Typescript
categories: Typescript
---

记录学习Typescript的思路大纲和要点。

<!-- more -->
[Typescritp官方网站][2]

[Typescritp中文手册][1]

[另一个中文手册][3]

[深入理解 TypeScript][7]


下面是一些关于TypeScript的学习文章

[TypeScript 安利指南][4]

[TS常见问题整理][5]，包含一些TS本身的问题，项目中tsconfig.json配置问题，ts+react使用中的一些问题。

[TypeScript进阶 之 重难点梳理][8]
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

    相比对象的key-value, 只能通过key去访问value，不能通过value访问key。在枚举中，正反都可以当作key来使用。

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

    函数类型包括两部分：参数类型和返回值类型。
    
    ```js
    // 命名函数
    function add(x: number, y: number): number {
        return x + y
    }

    // 匿名函数
    let myAdd = function(x: number, y: number): number {
        return x + y
    }
    ```
    TypeScript能够根据返回语句推断出返回值类型，因此返回值类型通常可以省略。

    函数的完整类型

    ```js
    let myAdd: (x: number, y: number) => number = function(x: number, y: number): number { return x + y }
    ```
    这种形式是匿名函数的方式，`=>`在这里不是指箭头函数，它的左侧是参数类型，右侧是返回值类型。

    函数没有返回值，其返回值类型是`void`


2. 可选参数和默认参数

    JavaScript中定义了函数，调用时其参数都是可选的，可传可不传，不传参时，其值就是undefined。

    在Typescript中，函数定义了参数，调用时就必须传。

    ```js
    function getName(firstName: string, lastName: string) {
        return firstName + ' ' + lastName
    }

    getName('Jim') // Error
    getName('Jim', 'Jackson', 'Sr.') // Error
    getName('Jim', 'Jackson') // OK
    ```

    在TypeScript中在参数名后加`?`来实现可选参数。可选参数必须写在必传参数之后。

    ```js
    function getName(firstName: string, lastName?: string) {
        if (lastName) {
            return firstName + ' ' + lastName
        } else {
            return lastName
        }
    }

    getName('Jim') // OK
    getName('Jim', 'Jackson', 'Sr.') // Error
    getName('Jim', 'Jackson') // OK
    ```

    在必选参数后面的有默认值的参数，与可选参数一样，调用时可以省略传值。

    但是如果默认参数放在必选参数前面，如果不想传值，则必须明确传入`undefined`来获取默认值。

    可选参数与末尾的默认参数共享参数类型：

    ```js
    function buildName(firstName: string, lastName?: string) {
        // ...
    }
    ```
    和
    ```js
    function buildName(firstName: string, lastName = "Smith") {
        // ...
    }
    ```
    这两个函数共享同样的类型`(firstName: string, lastName?: string) => string`


3. 剩余参数（rest）

    在Javascript中，当你不知道函数会有多少参数传进来的时候，我们用`arguments`来访问所有传入的参数。

    TypeScript中，我们可以用`...`符号来获取剩余参数

    ```js
    function buildName(firstName: string, ...restOfName: strin[]) {
        return `${firstName} ${restOfName.join(" ")}`
    }

    let myName = buildName('Joseph', 'Samuel', 'Lucas', 'MacKinzie')

    // 定义函数类型时也可以用到剩余参数
    let buildNameFn: (fname: string, ...rest: string[]) => string = buildName
    ```

4. this

    重点理解JavaScript中的this知识。

5. 函数重载（overload）
    JavaScript中经常会碰到根据参数的类型返回不同类型的值，在TypeScript中通过为同一个函数定义多个函数类型来进行函数重载

    ```js
    function getValue(x: string) : string

    function getValue(x: number): number

    function getValue(x): any {
        if(typeof x === 'string') {
            return '返回字符串'
        } else if(typeof x === 'number') {
            return x**2
        }
    }
    ```
    用JavaScript和Typescript实现这个函数的逻辑是相同的，只是在TypeScript中要定义多个类型，清晰展示传入值类型和返回值类型之间的关系。
    注意：`function getValue(x): any`并不是重载列表的一部分，这里只有两个重载。

> ## 类(Classes)
1. 类的定义

    关于类的详细知识，请查看阮一峰老师ES6教程中的讲解，很详细。TypeScript中类的使用是一样的，只是要在定义属性和方法时要注意类型的定义。

2. public、private、pretected修饰符

    这三种修饰符的作用是用来说明类中的属性和方法的使用范围：

    ```js
    class Test {
        constructor() {}
        // 公有实例属性
        name = 'Jack';
        // 公有方法
        say() {
            console.log(this.name)
        }
        public hello () {
            this.say()
            this.proFn()
        }
        // 私有属性
        private hide() {}
        // 受保护属性
        protected proFn() {}
        // 静态属性
        static fn() {}
    }

    class subTest extends Test {
        subFn() {
            console.log(this.name) // OK
            this.proFn() // OK
        }
    }
    ```
    
    public：表示这个属性或者方法在类，子类（派生类），类的外部都可访问到；

    以`name`属性为例，在`Test`类中可以访问，在派生类`subTest`中也可以访问。在类外部也可以访问，如下：

    ```js
    const test = new Test()
    consle.log(test.name)  // OK
    ```

    private: 表示这个属性或者方法只能在类内部可以访问。

    `hide`方法只能在`Test`类中访问，在`subTest`类中不能访问，通过`test`实例也不能访问。

    protected：表示这个属性或者方法可以在类和子类中访问。

    `proFn`方法可以在`Test`类和`subTest`访问。

    注意：当省略修饰符的时候，默认表示的是public。

3. readonly修饰符

    顾名思义，`readonly`修饰符修饰的属性是只读的，不能被修改。只读属性必须在声明或者构造函数里初始化。


4. 静态属性

    public、private、protected和readonly修饰符针对的都是实例属性和方法。

    类中定义静态属性和静态方法，只能通过类本身获取或者调用，不能通过实例化后的对象使用。其中要注意，静态方法中不能获取类中非静态的属性。

    ```js
    class Person1 {
        static name1:string = 'ff'
        static sayHello():void {
            console.log(this.name1 + 'hello')
        }
    }

    Person1.sayHello()
    ```

5. 存取器

    类似于Object.defineProperty方法

    ```js
    class Name {
        private _name: string

        get myname() {
            return this._name
        }
        set myname(value:string) {
            this._name = value
        }
    }

    let nameInfo = new Name()

    nameInfo.myname
    nameInfo.myname = 'gfg'
    ```

6. 抽象类（Abstract Classes）

    抽象类，定义一个标准，是其他类继承的基类，抽象类中定义的抽象方法，子类必须实现。

    不能直接被实例化, 不能new Animal1()
    抽象方法只能定义在抽象类中，抽象方法不包含具体实现。

    ```js
    abstract class Aniaml1 {
        public name:string
        constructor(name:string){
            this.name = name
        }
        abstract eat():void // 这个方法必须在子类中实现
        run():void{ // 这个不是抽象方法，所以子类中可以不实现。

        }
    }
    ```

> ## 泛型

1. 通俗来讲，泛型解决的是类、接口、方法的复用性问题，以及对不特定数据类型的支持。在C语言和Java中，使用泛型创建可支持多种类型的组件，这样就允许用户根据自己的类型来使用组件。

2. 泛型函数

    ```js
    // 定义普通函数，指定了类型，所以没办法复用
    function getData(value: string):string {
        return value
    }

    // 给函数参数和返回值指定泛型，就可以不受类型限制而可以复用
    function getData<T>(value: T): T {
        return value
    }

    getData<number>(12)
    getData<string>('acb')

    // 下面这个泛型函数接收类型参数T，参数arg，是个元素类型为T的数组，返回元素类型为T的数组。
    function getArray<T>(arg: T[]): T[] {
        console.log(arg.length)
        return arg
    }

    ```

3. 泛型类型

    ```js
    function getInfo<T>(arg: T): T {
        return arg
    }

    let myInfo: <T>(arg: T) => T = getInfo
    ```
    这里，变量myInfo的类型就是个泛型类型，函数getInfo正好能匹配上这个泛型类型。也可以这样写：

    ```js
    let myInfo: {<T>(arg: T): T} = getInfo
    ```
4. 泛型接口

    上面的最后一个写法就可以用泛型接口实现

    ```js
    interface MyInfoFn {
        <T>(arg: T): T
    }

    function getInfo<T>(arg: T): T {
        return arg
    }

    let myInfo: MyInfoFn = getInfo

    myInfo<number>(234)
    ```
    泛型接口也可以这样写：

    ```js
    interface MyInfoFn<T> {
        (arg: T): T
    }

    function getInfo<T>(arg: T): T {
        return arg
    }

    let myInfo: MyInfoFn<string> = getInfo

    myInfo('sdf')
    ```
    注意两种写法的不同。

5. 泛型类

    泛型类和泛型接口写法差不多，

    ```js
    class Info<T> {
        public value: T;
        constructor(value: T) {
            this.value = value
        }

        showInfo(x: T):T {
            return x
        }
    }

    let info = new Info<string>('sdf')
    ```
> ## 类型兼容性

类型兼容性是基于结构类型的，结构类型只使用其成员来描述类型。

结构化类型系统的基本规则是：如果X要兼容y，那么y至少具有与x相同的属性（y要包含x的所有属性）。

    ```js
    interface Person {
        name: string
    }

    let x: Person
    let y = { name：'Jack', age: 20}
    x = y
    ```
       
由于y有一个成员`name: string`匹配Person接口规定的属性，这就意味着x是y的子类型。因此这个复制是合法的。

> ## 模块

TypeScript与ES6一样，任何包含export和import的文件都被当作一个模块。任何声明（比如变量、函数、类、类型别名、接口）都可以通过export导出。

> ## 命名空间

在代码量较大的情况下，为了避免命名冲突，可将相似功能的函数、类、接口等放置到命名空间内。

TypeScript中的命名空间将代码包裹起来，通过export对外暴露需要在外部访问的成员。

    ```js
    namespace Animal {
        let dog:string

        export interface AnimalInfo {
            getInfo(name: string): string
        }

        export class Dog implements AnimalInfo {
            getInfo(name:string) {
                return '名字：' + name
            }
        }
    }
    ```

命名空间和模块的区别：

    命名空间：内部模块，主要用于组织代码，避免命名冲突；

    模块：外部模块的简称，侧重抽取公共代码，实现重用。一个模块中可能会有多个命名空间。

> ## 装饰器

装饰器相关的知识直接学习阮一峰老师的ES6教程中的装饰器知识。

> ## 编写声明文件

[声明文件][6]

[1]: https://typescript.bootcss.com/
[2]: https://www.typescriptlang.org/docs/home.html
[3]: https://zhongsp.gitbooks.io/typescript-handbook/content/doc/handbook/Basic%20Types.html
[4]: https://mp.weixin.qq.com/s/oaGWXcEYAw8ovfcY4nr5dQ
[5]: https://mp.weixin.qq.com/s/KmDLcJ0jhB667ZouDB8tyg
[6]: https://www.cnblogs.com/jiasm/p/9789962.html
[7]: https://jkchao.github.io/typescript-book-chinese/typings/lib.html#%E4%BD%BF%E7%94%A8%E4%BE%8B%E5%AD%90
[8]: https://mp.weixin.qq.com/s/_lO3cd0FcF0Dg3TRnHPdwg
