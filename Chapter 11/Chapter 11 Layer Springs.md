# Chapter 11: Layer Springs

您使用spring view动画为Bahama Air项目添加了一些非常酷的效果。现在是学习如何为图层创建spring动画的时候了!您可能一直希望为您的图层动画添加一些有趣的方法-现在您将得到准确地做到这一点!

图层的Spring动画与你通过调用UIKit方法为Spring动画创建的动画有点不同。UIKit方法允许您创建一个有点过于简化的类似spring的动画，但是它的核心动画对应物呈现了一个适当的物理模拟，看起来和感觉起来都更加自然。

本章将介绍UIKit和Core Animation spring Animation之间的不同之处，并向您介绍如何向Bahama Air项目添加一些新的图层spring动画。

不过，首先，您将快速浏览一些理论!

## 阻尼谐振子
阻尼谐什么?

UIKit API简化了spring动画的创建;你不需要知道他们是如何在引擎盖下工作的。然而，由于您现在是核心动画专家，您将被期望更深入地研究细节。

考虑一个钟摆的简单例子;你可以想象一下你祖父的钟摆。这个架子太高了，已经在地板上放了90年了。

在一个没有摩擦的完美世界里，当你爷爷放开钟摆，它就会永远摆动。滴答，滴答，滴答，滴答，滴答……



[图片]



如果爷爷把一支无摩擦的笔放在钟摆上，然后慢慢地把一张纸滑到钟摆下面，他会看到如下图:



[图片]



这是谐振子的一个例子:钟摆在它的平衡点(钟摆静止时所处的位置)前后摆动(或振荡)的量相等。没有摩擦力，钟摆就会永远摆下去。

然而，在现实世界中，系统由于摩擦而失去能量，最终会在平衡点处稳定下来:



[图片]



如果爷爷现在把一张纸放到钟摆下面，图表会是这样的:



[图片]



这是一个阻尼谐振子-有一些力作用于(或阻尼)振荡，所以它每次都会慢一点，直到它停止。

没那么糟吧?

钟摆稳定下来所需要的时间长度，以及最终振荡器图形的样子，取决于振荡系统的以下参数:

* 阻尼:这是由于空气摩擦、机械摩擦和作用于系统的其他外部减速力。
* 质量:钟摆越重，它摆动的时间越长。
* 刚度:振荡器的“弹簧”越硬，也就是地球的重力，摆开始摆动的时候就越硬，系统稳定下来的速度也就越快。想象一下，如果你在月球或木星上使用这个钟摆;在低重力和高重力的情况下，运动是完全不同的。

* 初始速度:你爷爷是简单地让钟摆走了，还是他推了一下钟摆?

“这一切都非常有趣，”您可能会想，“但是它与spring动画有什么关系呢?”

问得好，回答得好!阻尼谐振子系统是驱动iOS中spring动画的关键。下一节将更详细地讨论这一点。

## UIKit vs。Core Animation springs
你可能已经注意到阻尼谐振子比一个简单的UIKit spring动画包含更多的变量。

当您使用弹簧阻尼动画方法(使用带有阻尼的弹簧和初始弹簧速度参数)时，唯一与弹簧相关的参数是阻尼和初始速度。

UIKit动态地调整所有其他变量，使系统在给定的持续时间内稳定下来。这就是为什么UIKit spring动画有时感觉有点-嗯，强迫。UIKit动画有点过于跳跃，对于训练有素的人来说，有点不自然。

幸运的是，Core Animation允许您通过CASpringAnimation类为您的层属性创建适当的spring动画。CASpringAnimation在幕后为UIKit创建spring动画，但是当你直接调用它时，你可以设置系统的各种变量，让动画自己安定下来。这种方法的缺点是你不能告诉动画它的持续时间应该是多少;这是由系统本身决定的，给定你提供的变量。

既然你知道阻尼谐振子是如何工作的，那么卡斯普林动画揭示以下特性就不足为奇了:

•阻尼:应用于系统的阻尼
•质量:系统中重量的质量
•刚度:弹簧附加在重物上的刚度•初始速度:施加在重物上的初始推力。

如果在任何时候，您想知道需要调整哪些变量才能使动画按照您想要的方式工作，请简单地回想一下落地钟的例子，这应该可以帮助您理清思路。

现在，这些准备工作已经足够了——是时候打开Xcode并编写一些spring动画代码了。

## 创建您的第一层spring动画
打开本章的启动项目，或者如果您已经完成了上一章的项目，那么您可以继续上一章的内容。

运行该项目，并观察缩放动画应用到文本字段时，他们达到最终目的地:



[图片]



这个缩放动画让用户知道字段是活动的，可以使用了。然而，动画的结尾有些突然。您可以使用适当的spring动画替换现有的动画，从而使它看起来更好。

打开ViewController.swift并找到animationDidStop(_:finished:)。创建缩放动画的代码如下:

```swift
let pulse = CABasicAnimation(keyPath: "transform.scale")
pulse.fromValue = 1.25
pulse.toValue = 1.0
pulse.duration = 0.25
layer?.addAnimation(pulse, forKey: nil)
```

因为CASpringAnimation是从CABasicAnimation派生出来的，所以您只需要替换类名并设置spring阻尼。

要做到这一点，可以找到以下几行:

```swift
let pulse = CABasicAnimation(keyPath: "transform.scale")
```

用以下案文取代:

```swift
let pulse = CASpringAnimation(keyPath: "transform.scale")
pulse.damping = 2.0
```

运行您的项目，并享受您的新的春天动画!

等等，那个动画有问题。多运行项目几次，仔细观看动画;你会注意到动画在大约0.25秒的时候停止并跳转到最后一帧。

这是一个例子，你的代码做的正是你让它做的:

* 您使用自定义阻尼值和所有其他系统变量的默认值创建了一个spring动画
* 但是通过设置它的duration属性，您还告诉它运行0.25秒。

弹簧系统不能在0.25秒内稳定;您提供的变量意味着动画应该运行几秒钟才会稳定下来。下面是一个可视化演示，演示如何切断spring动画:



[图片]



幸运的是，这是一个简单的修复。一旦你设置了所有的系统变量，如刚度和阻尼，询问你的CASpringAnimation需要多少时间来稳定下来，并将其设置为动画的持续时间。

取代`pulse.duration = 0.25`，具体如下:

```swift
pulse.duration = pulse.settlingDuration
```

结算期限估计系统结算所需的时间;您可以使用该值让Core Animation知道动画应该在屏幕上停留多长时间。

再次运行您的项目，以享受平稳的摇摆轴动画:



[图片]



它看起来好多了，但是它能跑很长时间。在给定的参数下，动画将花费1.93秒来稳定下来——对于一个只打算关闭前一个过渡动画的效果来说，这个时间太长了。

回想一下本章导论中的摆锤示例:您需要增加或减少阻尼来减少动画的持续时间吗?

你是对的-一个更大的阻尼值意味着钟摆会稳定得更快。改变你的动画阻尼为7.5，以获得更微妙的效果:

```swift
pulse.damping = 7.5
```

重新运行项目;这次的结垢效果就像丝绸一样光滑。

## 弹簧动画属性
这就是你的第一个spring动画;你所要做的就是调整阻尼，然后一切就会自行解决。
那么刚度、初速度和质量呢?
CASpringAnimation为它所有的弹性属性提供了预定义的值:

* damping:10.0

* mass:1.0
* stiffness:100.0
* initialVelocity: 0.0

在本节中，您将向文本字段添加输入验证，如果用户输入的字符太少，则使字段跳转。您将使用CASpringAnimation的所有四个属性来产生您想要的精确效果。

滚动到ViewController.swift的最底部，找到使ViewController符合UITextFieldDelegate协议的类扩展。

向扩展体添加以下回调方法:

```swift
func textFieldDidEndEditing(_ textField: UITextField) {
  guard let text = textField.text else { return }
  if text.count < 5 {
    // add animations here
	} 
}
```

委托方法textFieldDidEndEditing(_textfield:)接收刚刚失去焦点的文本字段作为参数。在上面的代码中，检查该字段的文本值是否小于5个字符;如果是，则播放动画以吸引用户对该字段的注意。

在注释//添加动画下面添加以下代码:

```swift
let jump = CASpringAnimation(keyPath: "position.y")
jump.fromValue = textField.layer.position.y + 1.0
jump.toValue = textField.layer.position.y
jump.duration = jump.settlingDuration
textField.layer.add(jump, forKey: nil)
```

你的项目运行;在用户名文本字段中单击(如果在设备上单击)，然后立即在密码字段中单击(或单击)。

这将触发textFieldDidEndEditing(_textfield:)，由于您没有输入任何文本，验证失败动画将播放。

目前，动画只是简单地将字段向下移动一个点，并将其向上移动一个点，使其回到原来的位置。这并不有趣——但是你可以用你的动画忍者技能轻松地解决这个问题!

### 初始速度
此属性允许您指定动画的起始速度。默认值0使动画在开始时没有push;就好像有人只是扛着重担就放手了。

正值使动画朝着平衡点的方向移动，负值使动画开始远离平衡点。

您的跳转动画应该是相当可见的，所以给它一个良好的推动在开始的100.0。在初始化动画对象之后添加以下行(重要的是在计算和设置持续时间之前添加它):

```swift
jump.initialVelocity = 100.0
```

再次检查你的弹簧动画:



[图片]



即使位置增量仅为1点，由于开始时的额外推力，电场也会跳得更高。在稳定下来之前，磁场会稍微振荡一下。

### 质量
它看起来更好，但是跳转动画设置得太快了。增加初始速度将使动画持续更久，但这也意味着场跳得太远。

如果你增加附加的重量的质量，而不是一个动画持续更久?听起来不错!

默认的质量值是1.0(在您的脑海中，您可以选择磅、千克或任何您喜欢的其他测量单位)，您可以使用任何正值来帮助您达到预期的效果。

为您当前的动画增加质量到10.0这样:

```swift
let jump = CASpringAnimation(keyPath: "position.y")
jump.initialVelocity = 100.0
jump.mass = 10.0
```

你的项目运行;上面的更改增加了动画的持续时间——但是额外的质量意味着文本字段的跳转比计划的略高。

别担心——你仍然走在正确的道路上。只要再做几次调整，你就会有效果了!

> 注意:目前调整spring变量可能感觉不是很直观，但是随着您对这些值进行进一步的实验，您将开始了解如何才能最好地实现预期的效果。

### 刚度
弹簧动画由于其高初速度和额外的质量超过了目标。但是，如果给控制动画的弹簧增加一些额外的刚度来控制运动，会怎么样呢?

刚度可以取任何您想要的正值:0创建一个非常软的弹簧(bouncy bouncy)， 100是默认值(bouncy)，超过100的每一个增量都会使弹簧变得越来越硬(弹性越来越小)。

在您的动画中，将刚度增加到1500，以限制跳转到合理的大小;新代码行下划线如下:

```swift
let jump = CASpringAnimation(keyPath: "position.y")
jump.initialVelocity = 100.0
jump.mass = 10.0
jump.stiffness = 1500.0
```

你的项目运行;动画现在跳得恰到好处，感觉很紧;这一定会引起用户的注意。

### 阻尼
动画看起来很棒，但是它看起来确实有点太长了。您将增加系统阻尼，使动画更快地稳定下来。

应用于系统的阻尼系数可以为任意正值;0将使你的动画永远振荡。增加你的动画阻尼到50.0:

```swift
let jump = CASpringAnimation(keyPath: "position.y")
jump.initialVelocity = 100.0
jump.mass = 10.0
jump.stiffness = 1500.0
jump.damping = 50.0
```

现在就运行这个项目，享受一个精美的，微妙的动画，吸引用户的注意，而不惹恼他们:



[图片]



## 特定的层属性
到目前为止，在本章中，您已经为转换和位置属性创建了层spring动画。从技术上讲，您可以使用UIKit spring api创建一个类似的弹性效果，尽管代价是平滑和质量。

为了结束CASpringAnimation，您将在一个层属性上创建一个spring动画，这是您不能单独使用视图动画创建的。

验证动画现在可能有点过于精细和平滑。您将在包含无效输入的文本字段周围添加一个不那么微妙的闪烁红色边框。

在textFieldDidEndEditing(_ textField:)中，在if语句中，在向文本字段添加跳转动画之后，添加以下代码来设置文本字段的边框:

```swift
textField.layer.borderWidth = 3.0
textField.layer.borderColor = UIColor.clear.cgColor
```

这段代码在文本字段周围添加了一个透明边框。接下来，您将使该边框颜色具有动画效果。

在边框上设置透明颜色的行下面添加以下代码:

```swift
let flash = CASpringAnimation(keyPath: "borderColor")
flash.damping = 7.0
flash.stiffness = 200.0
flash.fromValue = UIColor(red: 1.0, green: 0.27, blue: 0.0, alpha:
1.0).cgColor
flash.toValue = UIColor.white.cgColor
flash.duration = flash.settlingDuration
textField.layer.add(flash, forKey: nil)
```

在这里，您创建了一个带有阻尼和刚度值的spring动画，该动画会随着文本字段的跳转同步闪烁边框。

一个简单的CABasicAnimation将使边框颜色从红色变为白色。但是因为您选择了spring动画，所以边框颜色从红色开始，并在最终的白色周围稍微振荡一下。

运行app欣赏结束效果;字段边界闪烁了几次才稳定下来并消失:



[图片]



这就结束了关于图层spring动画的速成课程!

> 注意:在一些iOS版本中，Core Animation删除了文本框的圆角。如果您在阅读本章时遇到这种情况，请在您编写的最后一段代码之后添加这一行:textField.layer。cornerRadius = 5。这将再次确认核心动画，你想要保持字段的圆角无论如何。

## 要点
* 您可以用一个实际的摆锤示例来描述spring动画的属性，该示例可以帮助您理解各种spring动画属性背后的动机。
* 要为您的图层创建弹簧动画，您可以使用CASpringAnimation API，它允许您调整弹簧的阻尼、质量、刚度和初始速度。

## 挑战
到目前为止，您已经对layer spring动画有了相当扎实的理解，所以我将留给您自己来解决这个挑战。

### 挑战:转换角落半径和背景动画到弹簧
您的任务是重新访问tintBackgroundColor(layer:， toColor:)和roundCorners(layer:， toRadius:)中的代码，用spring动画替换现有的代码，并配置动画，这样您就可以清楚地看到圆角反弹，而不会过度。

独自完成这个挑战将给您一些时间来试验CASpringAnimation的属性，并让您了解什么值工作得很好。

当你完成后，你就可以进入下一章，处理图层关键帧动画了。

