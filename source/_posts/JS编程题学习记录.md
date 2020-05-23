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