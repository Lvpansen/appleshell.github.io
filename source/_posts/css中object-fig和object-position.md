---
title: css中object-fig和object-position
date: 2019-11-29 18:24:36
tags: css
categories: css
---

`object-fit`属性用来指定`img`元素如何适应宽度和高度确定的框；`object-position`属性用来切换`img`元素在框内的对齐方式。

<!-- more -->

html中使用的图片的实际尺寸大部分都是比较大的，尤其是在移动端，而且越清晰的图片相应的尺寸也会越大，往往会超过手机的屏幕尺寸，通常情况下我们会给`img`元素增加`width`和`height`的css属性，以让图片能够限制在父容器中。

给图片设置宽高需要注意的是图片的显示变形问题，如果我们给图片设置的宽高，或者在图片宽高均为100%时，设置父容器的宽高，就需要按照图片原始的宽高比例来设置，这样相当麻烦，每个图片都要计算其比例，这种方法是行不通的。我通常的做法是，设置好父容器需要的宽和高，然后设置图片：

    max-width: 100%;
    max-height: 100%;

这样就能保证图片即被限制在容器内，又保证了其显示不会变形。

当然上面的属性也可以只设置一个，图片虽然不会变形，但是通常图片内容会显示不完整。

## object-fit属性

图片自适应容器还有一种方法是用`backkground`的方式，通过设置`background-size`属性来控制。而`object-fit`属性的功能就类似于背景图片控制显示的方法来控制`img`的自适应显示。

### 取值

> contain

类似`background-size: contain`的效果，图片将会等比例缩放，直到宽或者高充满容器。如果图片宽高比例跟容器宽高比例不相同，那么未充满容器的一侧就会留白。

> cover

类似`background-size: cover`的效果，图片将会保持宽高比例不变的情况下，覆盖整个容器。如果图片宽高比例与容器宽高比例不同，那么超出容器的一侧就会被裁剪。

> fill

图片将会充满容器，不会被裁剪，也不会有留白。如果图片宽高比例与容器宽高比例不同，那么图片会被拉伸，显示变形。

> scale-down

图片显示的尺寸会与`none`或者`contain`中的一个相同，取决于这两个属性谁得到的图片尺寸更小一些。

> none

图片尺寸没有做任何变化

我自己感觉常用的应该是前三个属性值。这个属性的详细内容点击[这里][1]

[1]:https://developer.mozilla.org/zh-CN/docs/Web/CSS/object-fit

## object-position

这个属性用来控制图片在容器内的位置，确定的是二维位置，所以用两个值来表示，可以使用相对或绝对偏移。

### 取值

第一个值表示横坐标，也就是水平方向的偏移，第二个值表示纵坐标，也就是垂直方向的偏移。

横坐标可以取值：left | center | right | 绝对位置（例如100px）| 相对位置（例如50%）

纵坐标坐标可以取值：top | center | bottom | 绝对位置（例如100px）| 相对位置（例如50%）

这个属性的详细内容点击[这里][2]

[2]:https://developer.mozilla.org/zh-CN/docs/Web/CSS/object-position

*****************

以上两个属性我们是拿`img`元素来举例的，其实不仅仅是图片可以使用这个属性，MDN中的介绍是**可替换元素（replaced element）**，除了`img`元素，`iframe`、`vedio`等一些元素，都是可替换元素，定义请点击[这里][3]

[3]: https://developer.mozilla.org/zh-CN/docs/Web/CSS/Replaced_element