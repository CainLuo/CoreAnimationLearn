# Section I: View Animations

这五章将向您介绍UIKit的动画API。 此API专门用于帮助您轻松制作视图动画，同时避免Core Animation的复杂性，Core Animation在幕后运行动画。

虽然易于使用，但UIKit动画API为您提供了大量灵活性和强大功能，可以处理大多数（如果不是全部）动画要求。

动画是可见的，屏幕效果适用于用户界面中的所有视图或可见对象：



[图片]



您可以在屏幕上为最终继承自UIView的任何对象设置动画; 这包括UILabel，UIImageView，UIButton以及您可能自己创建的任何自定义类。

在本节关于视图动画的五章中，您将学习如何使用动画来改进虚构的航空公司应用程序Bahama Air，为其UI元素添加各种动画。

首先，您将向登录屏幕添加动画：



[图片]



您将在以下章节中使用此屏幕：

* 第1章，入门：您将学习如何移动，缩放和淡化视图。 您将创建许多不同的动画，以适应Swift和基本的UIKit API。
* 第2章，Springs：您将以线性动画的概念为基础，使用弹簧驱动的动画创建更引人注目的效果。Boiiing！：]
* 第3章，过渡：您将了解UIKit中的几种类方法，它们可以帮助您在屏幕内外显示视图。 这些单行API调用使转换效果易于实现。

在登录屏幕上完成工作后，您将转到Bahama Air航班状态屏幕。 您将使用现有屏幕，通过添加登录屏幕主题后面的动画，使其更加精彩。



[图片]



您将从屏幕的静态版本开始，添加一些引人注目的高级动画以改善用户体验：

* 第4章，在实践中查看动画：您已经了解了UIKit中有关动画的大部分知识。 本章将教您如何以创造性的方式结合您熟悉的技术，以构建更酷的动画。
* 第5章，关键帧动画：您将使用关键帧动画来解锁令人印象深刻的UI的最终成就：创建从许多不同阶段构建的精细动画序列。

一旦您完成了本节中的所有章节，您将获得一些关于动画的深入体验，您可以继续阅读本书的其余部分。

本节向您展示了为视图添加动画是多么容易 - 请继续阅读第1章开始！