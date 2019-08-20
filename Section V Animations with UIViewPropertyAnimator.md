# Section V: Animations with UIViewPropertyAnimator

UIViewPropertyAnimator是iOS10中引入的一个类，它可以帮助开发人员创建交互式，可中断的视图动画。

既然你在这本书中做到了这一点（巨大的恭喜），你已经知道如何利用Core Animation提供的大部分功能。而且由于UIKit中的所有API都包含了较低级别的功能，因此在查看UIViewPropertyAnimator时不会有太多惊喜。

但是，这个类确实使某些类型的视图动画更容易创建，因此绝对值得研究。

最值得注意的是，当您通过动画师运行动画时，您可以动态调整这些动画 - 您可以暂停，停止，反转和更改已经运行的动画的速度。

如上所述，您可以通过使用图层和视图动画的组合来完成上述所有操作，但UIViewPropertyAnimator可以方便地将多个API包装在同一个类中，这样更容易使用。

此外，这个新类完全取代了UIView.animate（withDuration ...）API集和使用CAAnimation创建的动画，因此您仍然需要经常将这些与UIViewPropertyAnimator动画结合使用。

在本书的这一部分中，您将学习一个具有大量不同视图动画的项目，您将使用UIViewPropertyAnimator实现这些动画。



[图片]



* 第20章，UIViewPropertyAnimator入门 - 了解如何创建基本视图动画和关键帧动画。您将研究使用超出内置缓动曲线的自定义时序。
* 第21章，使用UIViewPropertyAnimator的中级动画 - 在本章中，您将学习如何使用具有自动布局的动画师。此外，您将学习如何反转动画或制作添加动画，以便在此过程中实现更平滑的更改。
* 第22章，使用UIViewPropertyAnimator进行交互式动画 - 了解如何根据用户的输入以交互方式驱动动画。为了获得额外的乐趣，您将了解基本和关键帧动画的交互性。
* 第23章，UIViewPropertyAnimator视图控制器转换 - 使用UIViewPropertyAnimator创建自定义视图控制器转换以驱动转换动画。您将创建静态和交互式过渡。

完成所有这些章节后，您肯定会在家中使用UIViewPropertyAnimator为您的应用程序中的所有类型的动画！