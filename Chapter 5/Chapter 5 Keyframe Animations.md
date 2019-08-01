# Chapter 5: Keyframe Animations

关键帧动画是一种特殊的视图动画，它不需要从点a到点B，而是让您创建具有多个里程碑的动画。关键帧是另一个复杂的积木，令人赏心悦目的动画，让您可以升级从简单的一个镜头动画到设计复杂的动画序列。

让我们看看，如果你想把多个简单的动画链接在一起，并以矩形模式移动视图，会是什么样子:



[图片]



要做到这一点，你可以把几个动画和完成闭包链接在一起，像这样:

```swift
UIView.animate(withDuration: 0.5,
  animations: {
    view.center.x += 200.0
  },
  completion: { _ in
    UIView.animate(withDuration: 0.5,
      animations: {
        view.center.y += 100.0
      },
			completion: { _ in
        UIView.animate(withDuration: 0.5,
          animations: {
            view.center.x -= 200.0
          },
          completion: { _ in
            UIView.animate(withDuration: 0.5,
              animations: {
                view.center.y -= 100.0
              }
						) 
					}
				) 
			}
		) 
	}
)
```

看那些嵌套!这是一个简单的矩形路径;你能想象一个更复杂的运动是什么样的吗?



[图片]



相反，您可以将整个动画分割成四个不同的阶段，或者关键帧，然后将各个关键帧组合成一个关键帧动画。



[图片]



第一阶段的动画序列加速飞机在跑道上。第二阶段给飞机一个小的高度和倾斜它向上。第三阶段继续倾斜飞机，以更快的速度将飞机推向天空。

在动画的最后10%中还有第四步，也是最后一步，让飞机淡出视野，就像它在一些低空云层后面移动一样。

完整的动画可能难以创建，但是将动画分成不同的阶段会使其更易于管理。一旦为每个阶段定义了关键帧，就基本上解决了问题。

您将了解关键帧，并如何组装成一个关键帧动画，通过创建一个类似于上述飞机动画-但它将更加复杂。别担心，你完全有能力处理这件事!

### 设置关键帧动画
如果你完成了上一章的项目，你可以把它作为你的起点;如果您没有这样做，或者您想重新开始，您可以使用本章的starter项目。

建立和运行您的项目;你应该看到两个交替的转机:



[图片]



你要让飞机从起始位置起飞，绕一圈，然后着陆并滑行回起始点。每次屏幕在连接航班之间切换时都会运行这个动画。

完整的动画应该是这样的:



[图片]



开放ViewController.swift;这个动画的所有代码都将驻留在一个名为plane()的新类方法中。

找到changeFlight(to:，animated:)，您将在顶部附近看到一个if动画语句，其中包含所有动画更改。在if块的顶部添加以下行(注意将其放入块内):

```swift
planeDepart()
```

现在添加以下初始版本的新动画方法plane()如下所示:

```swift
func planeDepart() {
  let originalCenter = planeImage.center
  UIView.animateKeyframes(withDuration: 1.5, delay: 0.0,
    animations: {
      //add keyframes
},
    completion: nil
  )
}
```

您将飞机的原始位置存储在originalCenter中，这样您就知道动画应该在何处完成。

您还可以使用animateKeyframes(带有duration:delay:options: animation:completion:)来创建第一个关键帧动画。参数列表与您在本书中始终用于创建视图动画的参数列表相同。将持续时间设置为1.5秒，不添加其他选项，并添加一个空动画闭包。

关键帧的选项是不同的;它们来自UIViewKeyFrameAnimationOptions枚举而不是UIViewAnimationOptions。您将在本章稍后了解关键帧选项的详细信息。

### 添加关键帧
现在是添加第一个关键帧的时候了。用以下代码替换//add keyframes注释:

```swift
UIView.addKeyframe(withRelativeStartTime: 0.0, relativeDuration: 0.25,
  animations: {
    self.planeImage.center.x += 80.0
    self.planeImage.center.y -= 10.0
  }
)
```

这是您将要添加的addKeyframe(使用relativestarttime: relativeDuration:animation:)的几个调用中的第一个。

上面的动画关键帧与您目前使用的动画方法没什么不同。

addKeyframe(withRelativeStartTime:relativeDuration:animation:)是您第一次在本书的动画中使用相对持续时间。关键帧的开始时间及其持续时间表示为相对于整个动画持续时间的百分比。例如，0.1是10%，0.25是25%，1.0是整个持续时间的100%。

使用相对值可以指定关键帧的持续时间应该只占总时间的一小部分;UIKit获取每个关键帧的相对持续时间并自动计算出每个关键帧的确切持续时间，为您节省了大量工作。

在上述代码的最后一位中，您将开始时间设置为0.0(表示“立即”)，持续时间设置为0.25，表示动画总时间的25%:



[图片]



由于您将整个动画的持续时间设置为1.5秒，所以第一个关键帧将持续大约0.375秒。稍后，如果您决定将动画的总持续时间修改为不同的值，各个关键帧动画将相应地重新计算它们的持续时间。

建立和运行您的项目，并检查出您的动画的第一帧:



[图片]



运行第一个关键帧将启动动画闭包中的代码:飞机向右移动80个点，向上移动10个点。

当然，这只是创建完整动画序列的第一步。要创建第二个关键帧，请在前一个addKeyframe调用下面的关键帧块中添加以下代码:

```swift
UIView.addKeyframe(withRelativeStartTime: 0.1, relativeDuration: 0.4) {
  self.planeImage.transform = CGAffineTransform(rotationAngle: -.pi / 8)
}
```

第二个关键帧开始动画的10%，并持续总时间的40%。这个关键帧旋转飞机，因为它离开跑道:



[图片]



现在构建并运行您的项目，看看这两个关键帧在一起运行时是什么样子的:

通过添加第三个关键帧继续构建序列，如下所示:

```swift
UIView.addKeyframe(withRelativeStartTime: 0.25, relativeDuration: 0.25) {
  self.planeImage.center.x += 100.0
  self.planeImage.center.y -= 50.0
  self.planeImage.alpha = 0.0
}
```

在动画的这个阶段，你让飞机继续前进，但开始淡出。

建立和运行您的项目，看看如何在您的动画三个关键帧:



[图片]



飞机飞得看不见了，消失了。现在你需要把它带回来，并在下一个连接飞行屏幕上安全着陆。

在您将飞机显示在屏幕上之前，您必须重置飞机的方向——也就是说，撤消您应用的旋转——并将其移动到可见区域的左侧。

因为飞机是隐形的，alpha值暂时为0，所以你可以在用户看不到任何东西的情况下移动它。添加以下关键帧到您的动画:

```swift
UIView.addKeyframe(withRelativeStartTime: 0.51, relativeDuration: 0.01) {
  self.planeImage.transform = .identity
  self.planeImage.center = CGPoint(x: 0.0, y: originalCenter.y)
}
```

这个关键帧在前三帧全部完成后运行，并且只持续一小部分秒;它重置平面变换并将其移动到屏幕的左侧边缘。

现在你可以添加最后的关键帧和出租车的飞机框架和出租车回到原来的位置:

```swift
UIView.addKeyframe(withRelativeStartTime: 0.55, relativeDuration: 0.45) {
  self.planeImage.alpha = 1.0
  self.planeImage.center = originalCenter
}
```

这个关键帧淡入飞机，同时移动到动画的初始点。

构建并运行您的项目，以查看完整的动画序列如下所示:



[图片]



花一分钟回顾一下您编写的代码。它很容易跟踪和解码;您可以修改、重新排列或更改上述任何部分的时间，而不需要您做太多工作。

将动画想象成一系列独立的动画，中间有延迟，甚至是一组由完成闭包触发的动画，这是一个很短的步骤。关键帧是一种非常有用和灵活的方法来设计和控制你的动画

### 关键帧动画的计算模式
关键帧动画不支持标准视图动画中内置的缓和曲线。这是故意的;关键帧应该在特定的时间开始和结束，并相互流动。

如果你上面的动画的每个阶段都有一个缓和曲线，那么平面就会晃动，而不是平稳地从一个动画移动到下一个动画。如果你能对整个动画应用缓动，那将会导致你的动画持续时间被忽略——这不是你想要的。

相反，您可以选择几种计算模式;每种模式都提供了一种不同的方法来计算动画的中间帧，以及用于平滑移动和均匀节奏的不同优化器。通过搜索UIViewKeyframeAnimationOptions查看文档以获得更多细节。

现在您已经知道了如何使用关键帧动画将任意数量的简单动画组合在一起，您可以构建任何想到的序列。如果您想测试您对关键帧和关键帧动画的知识，请在进入下一节之前尝试一下下面的挑战。

## 要点
* 您可以使用关键帧来“链接”动画，这比在标准api上使用补全闭包来生成序列要容易得多。
* 关键帧动画不仅允许你“链”动画，而且可以“重叠”它们，并按你希望的任何方式对它们进行分组。
* 不要忘记所有关键帧计时值都是相对于完整动画的，而定义完整序列的API使用绝对时间。

## 挑战
### 挑战:激活航班起飞时间

屏幕上仍然有一个UI元素没有动画:屏幕顶部显示航班起飞时间的黑色航班摘要状态栏。



[图片]



您的挑战是添加一个关键帧动画，以便在每次连接航班数据更改时将摘要移出屏幕。然后，你将改变文字，以反映下一趟航班的起飞时间，使其在屏幕上显示为动画，如下图所示:



[图片]



离开摘要标签已经连接到一个名为summary的出口。
如果你需要你的解决方案的一般结构，这里有一个基本的配方，你可以遵循:

* 创建一个新的方法summarySwitch(to: String)来动画摘要标签。
* 在方法内部，定义一个有两个关键帧的关键帧动画——一个用于将文本动画出屏幕，另一个用于将文本动画返回。
* 最后，调用关键帧动画外的delay(seconds:completion:)并在closure参数中更改摘要标签的文本。您需要对文本的更改进行计时，以便准确地在标签离开屏幕时发生更改。
* 从changeFlight的“动画”部分调用summarySwitch(to) (to:，animated:)，不要忘记将现有的非动画调用移动到else语句中。

你做得怎么样?如果你成功了，恭喜你!您已经成功地使项目中的每个UI元素具有了动画效果。

本书的下一部分将深入探讨看似复杂的自动布局和动画世界;您将了解到自动布局并不像您可能认为的那样神秘，而且您将学习动画约束，就像动画视图一样简单!