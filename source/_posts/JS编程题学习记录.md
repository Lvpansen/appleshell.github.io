---
title: JS编程题学习记录
date: 2020-05-23 11:33:42
tags: Javascript
---

每天一道编程题，学习解决思路，总结题目中的涉及到的知识点。

<!-- more -->

> ### 实现简单的模板字符串替换

[原文地址](https://mp.weixin.qq.com/s/yjGatP7NSnZRAB3v5FslmQ)

<table><tr><td bgcolor=#eee>实现一个render(template, context)方法，将template中的占位符用context填充</td></tr></table>

示例：

    ```js
      var obj = {name: '小明', age: '12'}
      var str = '{{name}}很厉害，才{{age}}岁'
      function render(template, context) {
        // ...
      }
      render(str, obj)  // 小明很厉害，才12岁
    ```

思路：通过遍历context中的属性，每次用属性名，通过正则去匹配template中的同名变量，然后替换为context中的属性值。

    ```js
      var obj = {name: '小明', age: '12'}
      var str = '{{name}}很厉害，才{{age}}岁'
      function render(template, context) {
        Object.keys(context).forEach(key => {
          let reg = new RegExp(`{{${key}}}`, 'g')
          template = template.replace(reg, context[key])
        })
        return template
      }
      render(str, obj)  // 小明很厉害，才12岁
    ```
知识点：

  1. 正则表达式，这道题中用的是字符串的匹配和替换。
  2. 对象的遍历，上述实现中使用了`Object.keys()`，再通过`forEach`方式实现遍历。另一种遍历对象的方式是`for...in`循环。两者的区别：`Object.keys()`可以获取对象自身的所有可枚举属性，不会遍历对象的原型；而`for...in`循环可以遍历对象自身以及原型上的所有可枚举属性。
  3. `str.replace`方法的第二个参数可以是函数，其返回值用于替换第一个参数匹配到的结果。
  4. 正则的非贪婪匹配模式

> ### 将数组扁平化并去除重复数据，最终得到一个升序且不重复的数组

示例：

  ```js
  var arr = [[2,3,5,3,[78,4,2,10]],[5,7,[23,15,6,[5,3,6,20]]],[3,4,6,7,[23,54,6,7]],[2,4,5,6,[13,6,7,8]],3]

  function getResult(arr) {
    // ...
  }

  getResult(arr) // 输出扁平、去重、升序的数组
  ```
思路：由题目可知，我们需要通过数组的扁平化、去重、排序三个步骤来实现。然后就是分别找出三个步骤的实现方法。扁平化：递归、flat、字符化后再转为数组...；去重：Set、循环比较...；排序：sort。

  ```js
  // 扁平化：
  var res = arr.toString().split(',').map(Number);

  var res = arr.flat(Infinity)

  var res = []
  function flatArr(arr) {  // 递归
      arr.forEach(item => {
          if(Array.isArray(item)) {
              return flatArr(item)
          }
          resa.push(item)
      })
      return res
  }
  // 去重
  Array.from(new Set(res))

  var res1 = []
  res.forEach(item => {
      if(res1.includes(item)){
          return
      }
      res1.push(item)
  })
  // 排序
  res1.sort((a, b) => a - b)
  ```
知识点：数组的扁平化、去重、排序实现方法，有些版本可能会使用数组的合并方法。

> ### 如何实现一个 new

思路：前提一定要清楚new干了什么或者说new的步骤是什么

示例：

  ```js
  new (fn, ...arg) {
    const obj = Object.create(fn.prototype)
    const res = fn.apply(obj, arg)
    return typeof res === 'object' ? res : obj
  }
  ```

知识点：考察new创建对象的步骤。

> ### 合并下面两个数组

`['A1','A2','B1','B2','C1','C2','D1','D2']`和`['A','B','C','D']`，得到`['A1', 'A2', 'A', 'B1', 'B2', 'B', 'C1', 'C2', 'C', 'D1', 'D2', 'D']`

示例：

  ```js
  var arr1 = ['A1','A2','B1','B2','C1','C2','D1','D2']
  var arr2 = ['A','B','C','D']

  var str = arr1.toString()
  arr2.forEach(item => {
      var reg = new RegExp(`${item}2`)
      str = str.replace(reg, `${item}2,${item}`)
  })
  str.split(',')
  ```
> ### 使用迭代的方式实现flatten

  示例：

    ```js
      var arr = [1,2,[3,4,[5,6,7]],8,[9,10,11,[12,13,[14,15],16]],17]
      function flatten(arr) {
        // ...
      }
      flatten(arr) // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17]
    ```

  思路：不考虑限制的话可以使用Array.flat()，借助字符串，递归实现。迭代的方式主要借助while循环。

    ```js
    var arr = [1,2,[3,4,[5,6,7]],8,[9,10,11,[12,13,[14,15],16]],17]
    function flatten(arr) {
      while(arr.some(item => Array.isArray(item))) {
        arr = [].concat(...arr)
      }
      return arr
    }
    flatten(arr)
    ```
  上述写法主要依赖`while循环`和数组的`concat`方法。while循环平时没有for循环，forEach用的多，需要理解其用法。

  concat方法用来合并两个或者多个数组，其参数可以是具体的值或者数组，具体的值会被复制进新数组，数组会被展开一层放进新数组。这里的用法就是`...`展开操作符将目标数组展开成值和数组作为concat的参数，每循环一次就flat一层，直到arr中不存在数组。主要理解点如下：

    ```js
    var alpha = ['a']
    var res = alpha.concat(1, [2, 3])  // 这里可以用...[1, [2,3]]代替
    console.log(res) // ['a', 1, 2, 3]
    ```
  知识点：while循环、...展开操作符、[Array.concat](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/concat)