# Chapter 4: View Animations in Practice

查看可用动画api的参考资料，很容易陷入这样的思维陷阱:视图动画仅限于淡入或淡出并在屏幕上移动视图。在本章中，您将通过三个结合不同动画的实际示例来创建更复杂、更重要的视觉效果。

话虽如此，请注意本章是可选的。如果您想继续学习新的api，请随意跳到下一章—没有什么不好的感觉!

在本章中，你将添加一些很酷的动画来装饰飞行摘要屏幕，如下图所示:



[图片]



在这一章中，你会发现一些新的效果，这些效果是建立在你在前几章所学到的动画基础上的:

1. 交叉淡入动画:将一幅图像与另一幅图像混合的动画。
2. 立方体转换动画:创建一个人造3D转换效果的转换动画。
3. 淡入和弹跳过渡:一个稍微不同的组合，简单的动画，辅助视图和其他一切你学到的这一点。

这一章的代码很有趣，所以直接进入下一节开始吧!

## 淡入淡出的动画
从本章的Resources文件夹中打开starter项目。构建并运行您的项目，以熟悉您必须使用的内容。该应用程序目前显示从伦敦到罗马的航班摘要，中途在巴黎停留。

航班摘要屏幕在两个中转航班之间切换，显示目的地、航班号、登机口号等信息:



[图片]



你的第一个任务是平滑过渡，或混合，在两个背景图像。

你的第一反应可能只是简单地淡入当前的图像，然后淡入新的图像。但是这种方法会在alpha接近0时显示图像背后的内容;您希望在整个动画期间，此屏幕后面的内容保持隐藏。

将图像淡出，然后再次将其淡入将导致一个奇怪的过渡，如下所示:



[图片]



显然，您需要一种不同的方法来解决这个问题。

打开ViewController.swift并添加以下新方法来处理新的交叉淡出效果:

```swift
func fade(imageView: UIImageView, toImage: UIImage, showEffects: Bool) {
  UIView.transition(with: imageView, duration: 1.0,
    options: .transitionCrossDissolve,
    animations: {
      imageView.image = toImage
    },
    completion: nil
  )
  UIView.animate(withDuration: 1.0, delay: 0.0,
    options: .curveEaseOut,
    animations: {
      self.snowView.alpha = showEffects ? 1.0 : 0.0
    },
    completion: nil
  )
}
```

该方法采用以下三个参数:
1. imageView:这是你将要淡出的图像视图。
2. toImage:这是您希望在动画结束时可见的新图像。
3. showEffects:这是一个布尔标志，指示场景应该显示还是隐藏降雪效果。降雪动画应该只在第一个转机航班的屏幕上播放，因为那里的机场可能正在经历恶劣的天气。

这次使用的过渡动画类型是。transitioncrossmelt——这允许你简单地将图像视图的图像更改为作为参数提供的图像，UIKit会自动创建一个交叉淡入过渡。

此外，您还可以根据showEffects的值将snowView淡入或淡入与动画的其他部分并行。

> 注意:降雪效果看起来很酷，对吧?您将在第22章“粒子发射器”中学习如何产生类似的效果。

你从哪里触发淡入(imageView:，toImage:，showEffects:) ?目前changeFlight(to:)切换两个连接航班的所有数据。

因为你最终将在这个屏幕上动画所有的UI元素，更新该方法的签名如下:

```swift
func changeFlight(to data: FlightData, animated: Bool = false) {
```

上面，您为新的动画参数提供了一个默认值false，这样现有的对这个方法的调用就可以像以前一样工作，而不需要动画。

接下来，找到changeFlight(to:，animated:)中的代码，它同时更新了图像视图和雪视图:

```swift
bgImageView.image = UIImage(named: data.weatherImageName)
snowView.isHidden = !data.showWeatherEffects
```

将其封装在一个if语句中，该语句可选地激活转换，如下所示:

```swift
if animated {
  fade(imageView: bgImageView,
    toImage: UIImage(named: data.weatherImageName)!,
    showEffects: data.showWeatherEffects)
} else {
  bgImageView.image = UIImage(named: data.weatherImageName)
  snowView.isHidden = !data.showWeatherEffects
}
```

当动画为真并从数据中传入适当的参数时，此方法调用淡入(imageView:， toImage:， showEffects:)。

如果您想查看数据的内部内容，请查看FlightData.swift。FlightData结构是表示单个航班的模型;FlightData中已经预先定义了其中两个航班。斯威夫特:伦敦toparis和parisToRome。

回到ViewController.swift中，在看到屏幕上的交叉淡入效果之前，您还需要再做一次更改，您需要在调用changeFlight(to:， animated:)的行中添加动画参数。

在changeFlight(to:animated:)中将调用更改为self。方法末尾的changeFlight在延迟调用中如下所示:

```swift
self.changeFlight(to: data.isTakingOff ? parisToRome : londonToParis, animated: true)
```

构建并运行您的应用程序;你现在应该看到图像视图转换顺利:



[图片]



这些图像彼此完美地融合在一起，由于你同时淡化了雪的效果，动画看起来是无缝的。在罗马，你甚至可以在一瞬间看到下雪。

您已经学习了一项重要的技术:转换可以用于对视图的非动画属性进行动画化更改。

> 注意:如果你想找点乐子，你也可以尝试前一章中的一些过渡效果。记住，像。transitionflipfromleft这样的过渡对于当前的项目来说太分散注意力了。transitioncross是一个微妙的“背景”效果，它只会增强动画效果，而动画只会出现在前景中。

这可以处理图像视图，但是航班号和登机口号的文本标签看起来可以使用一些创造性的动画:



[图片]



接下来，您将使用伪3d转换来处理这些问题。

## 多维数据集转换
您将在本节中构建的效果使其看起来像是飞行和登机口信息位于围绕其中心旋转以显示下一个值的立方体的相邻边。完成后，你的动画将如下图所示:



[图片]



这不是一个真正的3D效果，但它看起来非常接近，这是一个伟大的机会，为您尝试动画具有辅助视图。

您的方法是添加一个临时标签，同时对两个标签的高度进行动画处理，删除临时标签，最后自己清理。

这也是您创建的第一个有方向的动画，因为您将向前和向后播放它。该方向将确定临时标签是出现在顶部还是底部。

首先在ViewController类中添加以下枚举:

```swift
enum AnimationDirection: Int {
  case positive = 1
  case negative = -1
}
```

您的动画方法将接受一个设置动画方向的参数。接下来，添加该方法的初始版本，如下所示:

```swift
func cubeTransition(label: UILabel, text: String, direction:
AnimationDirection) {
  let auxLabel = UILabel(frame: label.frame)
  auxLabel.text = text
  auxLabel.font = label.font
  auxLabel.textAlignment = label.textAlignment
  auxLabel.textColor = label.textColor
  auxLabel.backgroundColor = label.backgroundColor
}
```

该方法取三个参数:

1. 标签:你想要动画的标签。

2. 文本:要显示在标签上的新文本。
3. 方向:使新文本标签具有动画效果的位置;这是视图的顶部或底部。

首先，您将创建一个新的标签auxLabel并将现有标签的所有关键属性复制到它，包括框架、字体和对齐方式。

两个标签之间的唯一区别是它们各自包含的文本:辅助标签将包含新文本。

让这两个标签出现在相同的位置不是您想要的。相反，您需要将辅助标签从现有视图中移开，使其变得非常小。

继续过渡!添加以下代码到cubeTransition的末尾:

```swift
let auxLabelOffset = CGFloat(direction.rawValue) *
  label.frame.size.height/2.0
auxLabel.transform =
  CGAffineTransform(translationX: 0.0, y: auxLabelOffset)
  .scaledBy(x: 1.0, y: 0.1)
)
label.superview?.addSubview(auxLabel)
```

在上面的代码中，首先计算辅助标签的垂直偏移量。方向的原始值是1或-1，这将为您提供正确的垂直偏移量来定位临时标签。

接下来，调整辅助标签的转换，创建一个仿透视效果。当你把文本单独缩放到y轴上时，它看起来会被压扁，就像你在看边缘上的文本平面:



[图片]



最后，在与现有标签相同的层次结构中添加新创建的标签。

接下来在cubeTransition的末尾添加以下动画代码:

```swift
UIView.animate(withDuration: 0.5, delay: 0.0, options: .curveEaseOut,
  animations: {
    auxLabel.transform = .identity
    label.transform =
      CGAffineTransform(translationX: 0.0, y: -auxLabelOffset)
          .scaledBy(x: 1.0, y: 0.1)
    )
  },
  completion: { _ in
    label.text = auxLabel.text
    label.transform = .identity
    auxLabel.removeFromSuperview()
  }
)
```

在动画块中重置auxLabel的转换;这使得新文本的高度增加，并将其精确地置于旧文本之上。

说到旧文本，您还可以应用一个转换来对标签进行缩放，并将其移动到与新文本出现相反的方向。

在if语句中插入changeFlight(to:animated:)中的以下代码，就在调用fadeImageView的地方:

```swift
let direction: AnimationDirection = data.isTakingOff ?
  .positive : .negative
cubeTransition(label: flightNr, text: data.flightNr, direction:
direction)
cubeTransition(label: gateNr, text: data.gateNr, direction: direction)
```

在相同的方法中，移动设置else语句中的flightStatus、flightNr、gateNr、departingFrom、arrivingTo text的现有代码，否则这些语句将立即更改标签的文本并破坏您的动画。

完成的else块现在应该是这样的:

```swift
} else {
  bgImageView.image = UIImage(named: data.weatherImageName)
  snowView.isHidden = !data.showWeatherEffects
  flightNr.text = data.flightNr
  gateNr.text = data.gateNr
  departingFrom.text = data.departingFrom
  arrivingTo.text = data.arrivingTo
  flightStatus.text = data.flightStatus
}
```

建立和运行您的项目;享受你的劳动成果，因为你看着门号和航班号动画通过你的幻想假三维过渡。



[图片]



请注意，向前和向后运行动画会给人一种真实的印象，即您正在切换两个备用状态。这个屏幕变得越来越生动，感谢你!

你还没有完成;您将通过在屏幕上的三个字母的机场代码中添加一个复合动画来结束本章，从而使屏幕转换感觉更自然、更敏捷!

## 渐变和弹跳过渡
本章的最后一个动画使用了标签、辅助视图以及到目前为止您所学到的所有内容。如果你愿意，可以把它看作是一章回顾!

首先为新的过渡动画添加以下新方法:

```swift
func moveLabel(label: UILabel, text: String, offset: CGPoint) {
  let auxLabel = UILabel(frame: label.frame)
  auxLabel.text = text
  auxLabel.font = label.font
  auxLabel.textAlignment = label.textAlignment
  auxLabel.textColor = label.textColor
  auxLabel.backgroundColor = .clear
  auxLabel.transform = CGAffineTransform(translationX: offset.x, y:
offset.y)
  auxLabel.alpha = 0
  view.addSubview(auxLabel)
}
```

新方法采用三个参数:
1. 标签:你想要动画的标签。
2. 文本:要显示的新文本。
3. 偏移量:用于使辅助标签具有动画效果的任意偏移量。

上面的代码非常简单:正如您在上一节中所做的那样，您创建一个辅助标签，并将所有属性从现有的标签复制到它。要创建标签转换，只需使用偏移参数。

最后，在将新标签添加到视图控制器视图之前，通过将其alpha属性设置为0.0来隐藏它。

这为您的动画设置了舞台。接下来，您将交换原始标签和辅助标签的位置。这一次，您将为每个标签创建单独的动画，并独立地移动它们，而不是为两个标签使用相同的动画。这将产生一个有趣的和有机的视觉效果。

首先在moveLabel的末尾添加以下代码:

```swift
UIView.animate(withDuration: 0.5, delay: 0.0,
  options: .curveEaseIn,
  animations: {
    label.transform = CGAffineTransform(translationX: offset.x, y:
offset.y)
   label.alpha = 0.0
  },
  completion: nil
)
```

这个动画将标签从原来的位置移开并淡出。接下来，添加以下代码来动画辅助标签:

```swift
UIView.animate(withDuration: 0.25, delay: 0.1, options: .curveEaseIn,
  animations: {
    auxLabel.transform = .identity
    auxLabel.alpha = 1.0
  },
  completion: {_ in
//clean up
	} 
)
```

重新设置上面辅助标签的转换，有效地将其移动到原始位置。您还可以通过动画化文本的alpha值来淡入文本。

注意，这个动画在0.1秒的延迟后开始，并且只持续0.25秒。这意味着这两个标签将在互换位置时重叠，创造出一种微妙但令人愉悦的“幽灵”效果。



[图片]



在测试新动画之前，需要添加代码，以便在动画完成时删除辅助标签。

在刚刚添加的代码的补全块中找到注释//clean up，并用以下代码替换:

```swift
auxLabel.removeFromSuperview()
label.text = text
label.alpha = 1.0
label.transform = .identity
```

这很好地完成了转换，将标签返回到初始状态，并设置标签的新文本。

现在找到changeFlight(to:，animated:)，并在if animated{语句的底部添加以下代码:

```swift
let offsetDeparting = CGPoint(
  x: CGFloat(direction.rawValue * 80),
  y: 0.0)
moveLabel(label: departingFrom, text: data.departingFrom,
  offset: offsetDeparting)
let offsetArriving = CGPoint(
  x: 0.0,
  y: CGFloat(direction.rawValue * 50))
moveLabel(label: arrivingTo, text: data.arrivingTo,
  offset: offsetArriving)
```

这个新方法采用任意偏移量作为参数。这允许您为起飞和到达机场创建两个不同的动画。

对于离开标签，创建一个水平移动;对于到达机场，您创建一个垂直移动。

这就是您需要的所有代码!构建并运行您的应用程序，通过快速的新转换效果查看所有标签的动画。



[图片]



花点时间来欣赏一下，即使在这个令人印象深刻的动画中，在任何给定的视图中，您仍然只动画了一些属性。您已经使用了边界、框架、中心、转换、背景颜色和alpha——但是您已经成功地使用这些属性创建了一些非常令人印象深刻的动画!

将每个属性单独动画化可能并不总是会产生令人兴奋的效果，但是使用一些简单动画的组合以及一些微妙而强大的技术，例如添加辅助视图和转换，可以产生令人印象深刻的视觉效果。

让你的想象力…起飞!

## 要点
* 您不局限于通过一个动画调用来动画一个视图属性;您可以自由地组合和重叠动画。

* 要创建复杂的效果，你可能会用到手头任务需要的所有“技巧”，包括在动画期间创建临时视图。

## 挑战
### 挑战1:动画飞行状态横幅
到目前为止，仍然有一个UI元素，当屏幕在两个连接航班之间切换时没有动画-航班状态:



[图片]



横幅上写着“登”的文字是UILabel;因此，您可以使用在本章中创建的两个label动画中的任何一个来显示备用的飞行状态。

我相信你可以自己弄清楚如何让横幅动起来;使用flightStatus出口访问标签和数据。获取要动画化的新文本。

您可以自由地进行试验，并进一步尝试。你获得的经验越多，就会对UIKit中的动画有更好的理解!



[图片]



现在您已经是动画和过渡方面的专家了，现在是查看动画的最后一个主题——将多个动画步骤和关键帧组合在一起。