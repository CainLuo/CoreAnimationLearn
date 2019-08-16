# Chapter 16: Replicating Animations

在本章中，您将尝试一些全新的东西：使用容器层来复制动画。

让我向您介绍我最喜欢的图层类：CAReplicatorLayer。

CAReplicatorLayer背后的想法很简单。 你创建了一些内容 - 它可以是一个形状，一个图像或任何你可以用图层绘制的东西 - 而CAReplicatorLayer会在屏幕上复制它，如下所示：



[图片]



“为什么我需要克隆形状或图像？”你会问。 你这样问是对的; 你经常不需要克隆任何东西的确切外观。

CAReplicatorLayer的超级大国来自于你可以轻松指示它使每个克隆与其祖先略有不同的事实。

例如，您可以逐步更改每个副本的色调。 原始图层可能是洋红色，而在创建每个副本时将色调向青色方向移动。



[图片]



此外，您可以在副本之间应用转换; 例如，您可以在每个副本之间应用简单的旋转变换以在圆圈中绘制它们，如下所示：



[图片]



但最好的功能是能够设置动画延迟以跟随每个副本。 当您设置0.2秒的instanceDelay并为原始内容添加动画时，第一个副本将以0.2秒的延迟动画，第二个副本将在0.4秒内动画，第三个副本将在0.6秒内动画，依此类推。

您可以使用它来创建引人入胜且复杂的动画，您可以以同步方式为多个元素设置动画。

在本章中，您将使用个人助理应用程序来“倾听”您的问题并回答。 作为Apple自己的私人助理Siri的眨眼，你的项目被命名为Iris。

您将创建两个不同的复制。 首先，你将创建在Iris会话时播放的视觉反馈动画，它看起来很像一个迷幻的正弦波：



[图片]



然后，您将使用CAReplicatorLayer创建交互式麦克风驱动的音频波，在用户说话时提供视觉反馈：



[图片]



这两个动画将向您介绍CAReplicatorLayer的许多功能。 为了涵盖这一层提供的每个功能，它将自己填满整本书！

但你不需要听我说我喜欢用CAReplicatorLayer创建动画多少; 是时候体验自己的魔力了。

## 像兔子一样复制
### 入门项目概述
启动本章的入门项目并打开Main.storyboard。 您会注意到项目设置非常简单：



[图片]



只有一个视图控制器，它具有一个按钮和一个标签。 用户在按下按钮时询问他们的问题; 当他们释放按钮时，Iris会发言回应。 标签将显示麦克风输入电平和Iris的答案。

打开ViewController.swift; 请注意，按钮事件已连接到操作。 当用户触摸按钮时，actionStartMonitoring（）会触发; 当用户抬起手指时，actionEndMonitoring（）会触发。

现在，ViewController做得不多，只需在用户抬起手指时从actionEndMonitoring（）内调用startSpeaking（）。

该项目还有两个超出本章范围的类，但它们将帮助您开发一个有趣和实用的应用程序，同时让您专注于动画部分。 它们如下：

* Assistant：人工智能助理。 它有一个预定义的有趣答案列表，并根据用户的问题说出它们。

* MicMonitor：监控iPhone麦克风上的输入电平，并反复调用您提供的闭合表达式。 您可以在此处更新显示。

一切准备就绪，你可以跳进去添加一些很酷的动画！

### 设置复制器层
打开ViewController.swift并添加以下两个属性：

```swift
let replicator = CAReplicatorLayer()
let dot = CALayer()
```

dot将是使用CALayer基本属性（如背景和边框颜色）绘制的简单形状。 复制器将帮助您在屏幕上获得多个点副本。

接下来，添加以下常量来创建动画：

```swift
let dotLength: CGFloat = 6.0
let dotOffset: CGFloat = 8.0
```

您将使用第一个常量作为点图层的宽度和高度，而第二个常量保持每个点复制之间的偏移。

要完成设置，您必须将复制器层添加到视图控制器的视图中。 将以下内容添加到viewDidLoad（）：

```swift
replicator.frame = view.bounds
view.layer.addSublayer(replicator)
```

在这里，您使复制器图层与视图控制器的视图大小相同，并将其添加为子图层。 如果你此时要运行项目，那么似乎没有任何改变; 那是因为你没有添加任何可见的内容来复制。

下一步是打扮点图层并将其添加到复制器以显示复制。 将以下内容附加到viewDidLoad（）：

```swift
dot.frame = CGRect(
  x: replicator.frame.size.width - dotLength,
  y: replicator.position.y,
  width: dotLength, height: dotLength)

dot.backgroundColor = UIColor.lightGray.cgColor
dot.borderColor = UIColor(white: 1.0, alpha: 1.0).cgColor
dot.borderWidth = 0.5
dot.cornerRadius = 1.5
```

首先将点图层定位到复制器的右边缘，因此，屏幕的右边缘。 然后设置图层的背景颜色并添加边框。

此时，图层将如下所示：



[图片]



要在屏幕上显示它，请添加以下内容：

```swift
replicator.addSublayer(dot)
```

这为复制器添加了点。 运行你的项目，你会看到它出现：



[图片]



到现在为止还挺好。 您所看到的是您可以将任何图层添加到任何其他图层。

“但你向我们承诺的魔力在哪里？”你在问。 保持紧张 - 你快到了！

您将使用三个CAReplicatorLayer属性来帮助您访问该复制魔法：

* instanceCount：设置所需的份数
* instanceTransform：设置要在副本之间应用的转换
* instanceDelay：设置副本之间的动画延迟

您希望复制填满屏幕，因此您必须将屏幕宽度除以副本之间的偏移量（dotOffset），以获得填充宽度所需的点数。 这将使您在Plus尺寸的iPhone上获得更多复制，而在iPhone SE上则更少。

将以下行添加到viewDidLoad（）以设置副本数（包括原始副本）：

```swift
replicator.instanceCount = Int(view.frame.size.width / dotOffset)
```

在一个5.5英寸的屏幕上，instanceCount最终为51; 在4.7英寸的屏幕46上，在4英寸的屏幕上，40。

再次运行项目; 嘿，所有的副本在哪里？



[图片]



不，复印机不再是弗里茨。 你的所有副本都在那里，但它们都是彼此重叠的。 您需要应用变换来显示所有变换。

将以下内容添加到viewDidLoad（）的末尾：

```swift
 replicator.instanceTransform = CATransform3DMakeTranslation(-dotOffset, 0.0, 0.0)
```

在上面的代码中，您首先在每个复制之间设置转换转换。 然后，您可以将dotOffset的负值用于X轴上的平移，因为您希望副本从右向左进行。

复制者将考虑点并从其位置减去8点; 此时将出现第一个副本。 第二个副本将出现在左边的另外8个点，依此类推。

运行你的项目; 您将看到所有副本在屏幕中间排列，每个复制从前一个复制转换8个点：



[图片]



### 你的第一个复制动画
要了解instanceDelay的作用，您将添加一个小的测试动画来点。 将以下内容添加到viewDidLoad（）的末尾：

```swift
// This is a test animation, you're going to delete it
let move = CABasicAnimation(keyPath: "position.y")
move.fromValue = dot.position.y
move.toValue = dot.position.y - 50.0
move.duration = 1.0
move.repeatCount = 10
dot.add(move, forKey: nil)
// This is the end of the code you're going to delete
```

这个动画只是将点移动50点并重复该动作10次。 运行该项目，您将看到所有克隆顺从地移动到屏幕上，就像许多同步的杀戮机器人一样。

接下来，您将为动画添加一些延迟。 插入以下代码行：

```swift
replicator.instanceDelay = 0.02
```

再次运行您的项目并观察所有副本如何遵循原始点动画，但只有在不断增加的延迟之后：



[图片]



现在您已经掌握了基础知识，现在是时候开始使用更酷的动画了。

在继续之前，删除刚刚添加的测试动画（使用上面的注释作为指导）; 你将在它的位置创建更好的动画。

> 注意：确保将代码行保留在设置instanceDelay属性的位置 - 您将需要它来实现您即将构建的效果。

## 复制多个动画
CAReplicatorLayer以您指示的方式复制您创建的内容和动画。 但是你可以想出一些很酷的动画，复制后看起来会更酷。

在本节中，您将处理在Iris讲话时播放的动画。 为此，您将结合使用具有不同延迟的多个简单动画来产生最终效果。

### 缩放动画

首先，您将连续缩放点图层以产生一波点。

找到startSpeaking（）并添加以下缩放动画：

```swift
let scale = CABasicAnimation(keyPath: "transform")
scale.fromValue = NSValue(caTransform3D: CATransform3DIdentity)
scale.toValue = NSValue(caTransform3D:
  CATransform3DMakeScale(1.4, 15, 1.0))
scale.duration = 0.33
scale.repeatCount = .infinity
scale.autoreverses = true
scale.timingFunction = CAMediaTimingFunction(name: .easeOut)
dot.add(scale, forKey: "dotScale")
```

这是一个简单的图层动画，就像本书这一部分中的许多其他动画一样。 您将点图层垂直缩放15倍，并连续来回运行动画。

运行项目并点击灰色按钮; 这会调用actionStartMonitoring（），actionEndMonitoring（），最后调用startSpeaking（）中的代码。 您应该看到原始点图层有效，所有副本都会跟随各自的延迟：



[图片]



恭喜 - 你和CAReplicatorLayer一起开始了！

如果你想从这个动画中获得一些额外的动作，请尝试更改动画的计时功能，看看你可以创建的其他很酷的波形。 例如，这里是一个缓入时间如何塑造最终效果：



[图片]



### 不透明动画
接下来，您将使原始点图层淡入淡出。 这将使波形成一定的尺寸，并在其增长和缩小时改变α以模拟光照条件。 它看起来很像旋转的丝带糖果：



[图片]



将以下淡入淡出动画添加到startSpeaking（）：

```swift
let fade = CABasicAnimation(keyPath: "opacity")
fade.fromValue = 1.0
fade.toValue = 0.2
fade.duration = 0.33
fade.beginTime = CACurrentMediaTime() + 0.33
fade.repeatCount = .infinity
fade.autoreverses = true
fade.timingFunction = CAMediaTimingFunction(name: .easeOut)
dot.add(fade, forKey: "dotOpacity")
```

在缩放动画的持续时间内，您将点图层从不透明度1.0淡化为0.2。 这一次，你以0.33秒的延迟开始动画; 当波浪充分发挥作用时，这会开始淡出效果。

当两个动画同时运行时，运行您的项目并享受新效果：



[图片]



### 色彩动画
如果你发挥你的想象力（并稍微眯一下），你可以想象波浪在你的屏幕上四处扭转。 如果你对它的色调进行动画处理，那么这种印象会更加清晰，就像波浪的每一面都有不同的颜色一样。

这应该是一个足够简单的任务 - 您所要做的就是为点的背景颜色设置动画。

添加第三个动画到startSpeaking（）：

```swift
let tint = CABasicAnimation(keyPath: "backgroundColor")
tint.fromValue = UIColor.magenta.cgColor
tint.toValue = UIColor.cyan.cgColor
tint.duration = 0.66
tint.beginTime = CACurrentMediaTime() + 0.28
tint.fillMode = .backwards
tint.repeatCount = .infinity
tint.autoreverses = true
tint.timingFunction = CAMediaTimingFunction(name: .easeInEaseOut)
dot.add(tint, forKey: "dotColor")
```

此动画将点的色调从品红色更改为青色并返回。 你使用0.66秒的持续时间; 这是缩放动画频率的两倍，并给人的印象是每次波浪“扭曲”时颜色都会发生变化。

你还给动画延迟了0.28秒; 这使得色彩色调动画在波形中出现“扭曲”之前就开始了。 这种微妙的效果在波浪“扭曲”之前提供了下一种颜色的暗示，好像有一些反射正在进行。

运行项目以检查新效果：



[图片]



## 动画CAReplicatorLayer属性

到目前为止，您通过动画复制器层中的内容来创建非常令人眼花缭乱的效果。但由于CAReplicatorLayer本身就是一个图层，因此您也可以为其自身的一些属性设置动画。

您可以为CAReplicatorLayer的基本属性（如position，backgroundColor或cornerRadius）设置动画，但是，您可以通过设置此图层中其他图层中不存在的某些特殊属性来创建一些有趣的效果。

CAReplicatorLayer独有的可动画属性包括：

* instanceDelay：为实例之间的延迟量设置动画
* instanceTransform：动态更改复制之间的转换
* instanceColor：更改用于所有实例的混合颜色
* instanceRedOffset，instanceGreenOffset，instanceBlueOffset：应用增量以应用于每个实例颜色组件
* instanceAlphaOffset：更改应用于每个实例的不透明度增量

在本节中，您将为实例转换设置动画，使语音波更加迷幻！在startSpeaking（）的末尾添加一个动画：

```swift
let initialRotation = CABasicAnimation(keyPath:
  "instanceTransform.rotation")
initialRotation.fromValue = 0.0
initialRotation.toValue   = 0.01
initialRotation.duration = 0.33
initialRotation.isRemovedOnCompletion = false
initialRotation.fillMode = .forwards
initialRotation.timingFunction = CAMediaTimingFunction(name: .easeOut)
replicator.add(initialRotation, forKey: "initialRotation")
```

此动画仅影响实例变换的旋转分量; 也就是说，它保留了为viewDidLoad（）中的实例设置的转换组件，并且仅为旋转设置动画。

您可以将实例之间的旋转设置为0.0弧度到0.01弧度。 与其邻居相比，每个复制都会略微旋转。

运行你的项目; 在一个全新的层面上享受你的复制动画 - 或者我应该说曲线？



[图片]



上面的instanceRotation动画看起来不错 - 但是当我说迷幻时，我的意思是精神错乱！ 如果你要结合所有正在运行的复制动画的效果并同时扭转并旋转波浪怎么办？

添加下面的动画以完成效果：

```swift
let rotation = CABasicAnimation(keyPath: "instanceTransform.rotation")
rotation.fromValue = 0.01
rotation.toValue   = -0.01
rotation.duration = 0.99
rotation.beginTime = CACurrentMediaTime() + 0.33
rotation.repeatCount = .infinity
rotation.autoreverses = true
rotation.timingFunction = CAMediaTimingFunction(name:  .easeInEaseOut)
replicator.add(rotation, forKey: "replicatorRotation")
```

在这里，您在instanceTransform.rotation上运行第二个动画，该动画在第一个动画完成后启动。 然后，您可以将变换旋转从0.01弧度（第一个动画的最终值）设置为-0.01弧度并返回。

这使得语音动画成为最后的疯狂推动：



[图片]



这是一个出色的（如果不是催眠诱导的）动画，你通过组合几个选择的动画创建它。 诀窍是知道要设置动画的属性以及如何选择正确的延迟和持续时间以获得您正在寻找的效果。

> 注意：就个人而言，我认为如果我将头部向右旋转45度，效果最好。 但要小心不要看太长时间 - 它会让你头晕目眩！

剩下的就是给你不那么有用的助手，Iris，倾听和回应的能力。

首先，你将赋予Iris演讲的力量。 名为Assistant的入门项目类将帮助您完成此任务。 将以下内容添加到startSpeaking（）的顶部:

```swift
meterLabel.text = assistant.randomAnswer()
assistant.speak(meterLabel.text!, completion: endSpeaking)
speakButton.isHidden = true
```

您首先从Assistant类中获得一个随机答案，并通过meterLabel将其可视化。 然后你打电话给助手讲话（_：完成:); 将endSpeaking（）作为完成参数传递给讲话结束时调用。

接下来，您将在endSpeaking（）中添加代码，删除所有正在运行的动画，并优雅地将wave返回到其初始状态。

将以下内容插入endSpeaking（）：

```swift
replicator.removeAllAnimations()
```

这将删除复制器的instanceTransform上的动画。

接下来，您需要将点图层设置为原始比例的动画。 加：

```swift
let scale = CABasicAnimation(keyPath: "transform")
scale.toValue = NSValue(caTransform3D: CATransform3DIdentity)
scale.duration = 0.33
scale.isRemovedOnCompletion = false
scale.fillMode = .forwards
dot.add(scale, forKey: nil)
```

对于此动画，您不指定fromValue; 这意味着Core Animation将从当前值开始动画，并将变换设置为CATransform3DIdentiy。

这是必要的，因为在任何给定时间，每个复制的实例都具有不同的变换。 省略fromValue会将每个复制从其当前比例设置为动画的最终值。

最后，添加以下代码以从dot中删除当前正在运行的其余动画并重置说话按钮状态：

```swift
dot.removeAnimation(forKey: "dotColor")
dot.removeAnimation(forKey: "dotOpacity")
dot.backgroundColor = UIColor.lightGray.cgColor
speakButton.isHidden = false
```

再次运行项目; 这一次，Iris会在你身边随便回答！ 当Iris完成讲话时，波浪将优雅地动画回到其初始状态：



[图片]



## 交互式复制动画
现在你需要每次按下发言按钮才能看到和听到Iris的回答。 但你实际上并没有问她什么，说实话，这是非常有趣的部分。

在最后一节中，您将创建一个动画，在您向Iris询问问题时显示麦克风输入。

> 注意:如果iOS模拟器中的麦克风不起作用，请使用物理设备在本章的这一部分中测试您的应用程序。

将以下内容添加到actionStartMonitoring（）：

```swift
dot.backgroundColor = UIColor.green.cgColor
monitor.startMonitoringWithHandler { level in
  self.meterLabel.text = String(format: "%.2f db", level)
}
```

当用户按下发言按钮时，上述方法触发。 要指示应用是“正在收听”，请将点图层颜色更改为绿色。 然后在监视器实例上调用startMonitoringWithHandler（）。

> 注意：MicMonitor类非常简单 - 如果您有兴趣了解它是如何获得麦克风级别的话，请查看MicMonitor.swift。

您作为参数提供的闭包块会重复执行，并将当前麦克风级别作为参数获取。

运行应用程序并按住按钮; 与设备通话，您将看到显示当前的麦克风电平。 如果iOS提示您输入麦克风，只需点击系统警报上的确定：



[图片]



到现在为止还挺好; 您现在需要的是使用级别变量来相应地设置复制器内容的动画。

首先，您需要将麦克风级别标准化为可用于动画的内容。 level的值在-160.0 db到0.0 db的范围内，-160.0 db是最安静的，0.0 db意味着非常大的声音。

在处理程序块中添加一行额外的代码，将级别值转换为有用的值，并将其存储在scaleFactor中，以便完整的块看起来像这样：

```swift
monitor.startMonitoringWithHandler { level in
  self.meterLabel.text = String(format: "%.2f db", level)
  let scaleFactor = max(0.2, CGFloat(level) + 50) / 2
}
```

scaleFactor将存储介于0.1和25.0之间的值。 您可以使用此选项将点缩放到合理的大小以表示麦克风输入电平。

将以下实例属性添加到ViewController类：

```swift
var lastTransformScale: CGFloat = 0.0
```

对于缩放动画，您需要将最后一个缩放值保存到此属性。 由于您将不断覆盖运行比例动画，因此您需要跟踪图层应该缩放到的最后一个值。

现在跳回麦克风处理程序闭包并添加以下代码，该代码从最后一个转换动画到您从当前麦克风级别计算的新转换：

```swift
let scale = CABasicAnimation(keyPath: "transform.scale.y")
scale.fromValue = self.lastTransformScale
scale.toValue = scaleFactor
scale.duration = 0.1
scale.isRemovedOnCompletion = false
scale.fillMode = .forwards
self.dot.add(scale, forKey: nil)
```

此图层动画仅在点图层变换的比例分量的y轴上运行。 您可以将比例从最后使用的值设置为当前scaleFactor。

最后，仍然在处理程序内部，添加以下代码以保存下一个处理程序调用的当前scaleFactor值：

```swift
self.lastTransformScale = scaleFactor
```

这段代码应该使复制器层生动起来。 运行项目，按住按钮说话。 您将看到屏幕上显示实际的音频波形，所有这些都由复制器层处理，而您只需为原始点图层设置动画：



[图片]



但是当你放开说话按钮时，结果有些莫名其妙：



[图片]



啊哈 - 您需要重置动画并停止监听麦克风。 你可以在actionEndMonitoring（）中处理它。

在该方法的顶部插入以下内容：

```swift
monitor.stopMonitoring()
dot.removeAllAnimations()
```

在这里，您可以通过调用stopMonitoring（）并删除在dotlayer上运行的所有动画来禁用麦克风监视器。 这样，应用程序可以继续并显示Iris动画。

试一试应用程序。 是不是和Iris说话很开心？

麦克风输入动画有点突然结束，但你将在下面的挑战中解决这个问题。

## 关键点
* 您可以通过CAReplicatorLayer轻松创建复合动画效果，以组合同一动画的多个副本。

* 您可以通过CAReplicatorLayer上的instanceCount，instanceTransform和instanceDelay属性设置动画复制之间的数量和变化。

* 除了在原始动画上设置动画属性外，还可以为复制器图层本身设置属性动画。

## 挑战
### 挑战1：平滑麦克风输入和光圈动画之间的过渡

您的第一个挑战是不仅通过调用dot.removeAllAnimations（）来删除点图层上运行的两个动画，而是将波动画回适合下一个要运行的动画的状态。
分三步采取这一挑战：

* 首先，删除从actionEndMonitoring（）中删除点上运行动画的行。
* 然后，在其位置，将点的比例设置为y轴上的值1.0。 将动画留在屏幕上等待Iris动画开始 - 完成后不要删除它。
* 最后，添加另一个动画，将点色调从绿色更改为洋红色。 对于此动画，请使用.backwards的fillMode  - 如果没有它，复制器层将追溯性地将色调重置为最终值。

记住这两个新动画的持续时间。 它们应该在Iris动画开始之前完成。

当你完成这个挑战时，两个动画应该很好地融合在一起，如下所示：



[图片]



