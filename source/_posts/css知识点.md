---
title: css知识点
date: 2020-03-03 21:58:35
tags: css
---

记录一下 css 只是要点。

<!-- more -->

## CSS 工作规则

### CSS 的三种分类

行内样式、嵌入样式、链接样式。

还有一种是在 css 文件中引入其他 css 样式文件

```css
@import url(css/styles2.css);
```

### css 选择符

1. 元素选择符

   这没什么好讲，就是应用 HTML 元素当选择符。

2. 上下文选择符，又可称为后代组合式选择符（descendant combinator selector）。

   html 代码

   ```html
   <section>
     <h2>An H2 Heading</h2>
     <p>This is paragraph 1</p>
     <p>Paragraph 2 has <a href="#">a link</a> in it.</p>
     <a href="#">Link</a>
     <div>
       <span>a span content</span>
     </div>
   </section>
   ```

   - 普通的上下文选择符
     ```css
     section p {
       color: red;
     }
     ```
   - 子选择符
     ```css
     section > h2 {
       color: red;
     }
     ```
   - 紧邻同胞选择符
     ```css
     h2 + p {
       color: red;
     }
     ```
   - 一般同胞选择符
     ```css
     h2 ~ a {
       color: red;
     }
     ```
   - 通配符

     ```css
     * {
       color: red;
     }

     section * a {
       color: green;
     } // 选中的是section下孙子元素为a标签的元素
     ```

3. ID 和类选择符

   - 类选择符

     普通类选择符、标签带类选择符、多类选择符

   - ID 选择符

     普通 ID 选择符、标签带 ID 选择符

   - 何时用 ID，何时用类

     ID 的用途是在页面中唯一地标识一个元素。

     类的目的是为了标识一组具有相同特征的元素。

4. 属性选择符

   - 属性名选择符，标签名[属性名]

     ```css
     img[title] {
       width: 100px;
     }
     ```

     选中有 title 属性的 img 元素

   - 属性值选择符，标签名[属性名="属性值"]

5. 伪类和伪元素

   - UI 伪类

     UI 伪类会基于特定 HTML 元素的状态应用样式。

   - 结构化伪类

     结构化伪类可以根据标记的结构应用样式，比如根据某元素的父元素或前面的同胞元素是什么。

    * 伪元素
      例如::after，::before等。

### 继承

CSS 中有很多属性是可以继承的，其中相当一部分都跟文本有关，比如颜色、字体、字号。

然而，也有很多 CSS 属性不能继承，因为继承这些属性没有意义。这些不能继承的属性主要涉及元素盒子的定位和显示方式，比如边框、外边距、内边距

### 层叠

通俗理解就是你写的样式，哪个个才是最终起作用的。

规则一：包含 ID 的选择符胜过包含类的选择符，包含类的选择符胜过包含标签名的选择符。

规则二：如果几个不同来源都为同一个标签的同一个属性定义了样式，行内样式胜过嵌入样式，嵌入样式胜过链接样式。在链接的样式表中，具有相同特指度的样式，后声明的胜过先声明的。

规则一胜过规则二。换句话说，如果选择符更明确（特指度更高），无论它在哪里，都会胜出。

规则三：设定的样式胜过继承的样式，此时不用考虑特指度（即显式设定优先）。

## 定位元素

### 盒模型

W3C标准盒模型：content-box，content即内容范围。
IE盒模型：border-box，border即边框范围。

### 浮动

浮动和清除浮动

### 定位

position: static、relative、absolute、fixed。

relative、absolute、fixed定位上下文。

### 显示属性

即display属性，取值和含义。例如inline、inline-block、block、none、table、flex、inherit等等。

## 背景

background-color

background-image

background-repeat

background-position

background-size

background-attachment

background（简写）

元素可以设置多个背景图片, 先列出的图片显示在最上层。
```css
div {
  background: 
  url(images/turq_spiral.png) 30px -10px no-repeat, 
  url(images/pink_spiral.png) 145px 0px no-repeat, 
  url(images/gray_spiral.png) 140px -30px no-repeat,#ffbd75;
}
```

背景渐变：

线性渐变，linear-gradient()
放射性渐变，radial-gradient()


## 字体和文本

字体相关的属性主要是font或以font开头的一些属性。

文本属性主要涉及文本的间距、缩进、装饰、对齐、行高等样式。

## 布局

要考虑是固定宽度布局还是弹性布局。

布局时，一般是不控制布局高度，而是让内容撑开，除非需要滚动条。而布局宽度则是要仔细考虑的。

三栏布局的实现：

固定宽度很简单，写死就好。

弹性布局：

float实现，position实现，flex实现。

## 媒体查询

## 动画