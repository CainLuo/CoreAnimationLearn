# Chapter 27: Frame Animations with UIImageView

现在是时候学习如何创建一种非常特殊的动画了。 帧动画是你小时候喜欢的动画类型，今天可能仍然很喜欢; 这就是迪斯尼的Duck Tales，Hanna-Barbera的Tom和Jerry以及Flintstones漫画的创作方式。

这也是您用来为游戏中的角色制作动画的动画。 仅仅将静态游戏角色从一个位置转换为另一个位置是不够的。 你需要移动角色的脚或旋转飞机的螺旋桨以给出逼真的运动感。

要创建角色移动的效果，可以将动画分解为帧，这些帧是表示动作阶段的静止图像。 当您快速显示帧时，一个接一个地显示帧，看起来您的角色正在移动：



[图片]



本章中的项目设置在南极，玩家可以控制企鹅企鹅。

你将从一个静态的场景开始，并通过这一章来让企鹅走路并在他的世界中滑行。



[图片]



## 项目基础
打开本章的入门项目，然后选择Main.storyboard以查看最初的游戏场景：



[图片]



企鹅和按钮都连接到ViewController.swift中的代码，但如果你点击它们，目前什么也没发生。 打开ViewController.swift并查看代码的内容。

您将看到两个出口：一个用于企鹅图像视图（企鹅），另一个用于幻灯片按钮（slideButton）。 您还有两个属性，walkFrames和slideFrames，它们分别包含步行和滑动动画的UIImage帧。

屏幕左侧的HUD按钮连接到actionLeft（_ :)和actionRight（_ :); 你将为这两种方法添加一些动画代码，让企鹅走路。

屏幕右侧的HUD按钮连接到actionSlide（_ :)，您可以在其中添加一些代码以使企鹅跳跃并在冰上滑动。

打开Images.xcassets并浏览帧图像以查看步行和滑动动画的各个帧，如下所示：



[图片]



你需要走路才能跑步 - 在这个游戏中，你的企鹅需要走路才能滑行！ 当玩家点击HUD按钮时，你的第一个任务是让企鹅走路。

## 设置帧动画
将以下代码添加到ViewController.swift中的loadWalkAnimation（）存根：

```swift
penguin.animationImages = walkFrames
penguin.animationDuration = animationDuration / 3
penguin.animationRepeatCount = 3
```

此代码在UIImageView上设置了许多您可能以前从未见过的属性。 以下是他们每个人依次做的事情：

1. animationImages：存储帧动画的所有帧图像。 walkFrames在代码中作为包含UIImage对象的数组在前面声明，所以在这里你只需要将它分配给penguin.animationImages来设置动画帧。

2. animationDuration：这告诉UIKit动画的一次迭代应该持续多长时间; 因为您将重复动画三次（见下文），所以将其设置为总animationDuration长度的三分之一。

3. animationRepeatCount：它控制动画的重复次数; 在这

例如，动画将重复三次。

最后，您需要从视图控制器中的viewDidLoad（）调用loadWalkAnimation（），以便每当玩家点击左箭头按钮时图像视图就会准备就绪。

将以下代码添加到viewDidLoad（）的底部：

```swift
loadWalkAnimation()
```

建立并运行项目; 点击左箭头按钮将您的企鹅设置为运动：



[图片]



哦 - 点击左箭头按钮时没有任何反应。 你做错了什么吗？ 没有; 您只为帧动画配置了图像视图，但您从未启动过动画。 如果不启动帧动画，图像视图将继续显示其图像属性的内容。

是时候让这只企鹅蹒跚了。

将以下代码添加到actionLeft（_ :)：

```swift
isLookingRight = false
```

要跟踪企鹅面临的方式，您将分别在actionLeft（_ :)和actionRight（_ :)中将isLookingRight设置为false或true。

由于每次更改企鹅的方向时都需要更新企鹅的变换和按钮变换，因此需要向isLookingRight的didSet观察者添加一些代码。

在类定义顶部附近找到以下属性：

```swift
var isLookingRight = true
```

...并将其替换为以下代码：

```swift
var isLookingRight: Bool = true {
  didSet {
    let xScale: CGFloat = isLookingRight ? 1 : -1
    penguin.transform = CGAffineTransform(scaleX: xScale, y: 1)
    slideButton.transform = penguin.transform
  } 
}
```

此代码将企鹅和滑动按钮的x轴刻度设置为1或-1，具体取决于isLookingRight的值。 然后设置该变换将翻转视图，以便企鹅可以面向正确的方向：



[图片]



现在你需要调用动画代码。 将以下内容添加到actionLeft（）的底部：

```swift
penguin.startAnimating()
```

当您调用startAnimating（）时，图像视图会在您配置动画时播放动画：animationImages数组中的每个帧按顺序显示，总共超过1秒。

动画完成后，图像视图将再次显示其初始内容 - 图像属性。

构建并运行您的项目; 点击左箭头按钮，看看你的企鹅在行动：



[图片]



你冷酷的朋友正在走路 - 但他无处可去！ 当步行动画播放时，您需要将企鹅移动到屏幕上。

### 翻转你的视图
将以下代码添加到actionLeft（_ :)：

```swift
UIView.animate(withDuration: animationDuration, delay: 0,
  options: .curveEaseOut,
  animations: {
    self.penguin.center.x -= self.walkSize.width
  },
  completion: nil
)
```

这是熟悉的代码，在本书的这一点上几乎不需要解释; 您只需在屏幕上为视图设置动画即可。 您可以使用步行图像的宽度来确定企鹅在动画播放时间内移动的距离。

再次运行项目; 现在看起来怎么样？



[图片]



这还差不多！ 走向另一个方向的代码非常相似。

将以下代码添加到actionRight（_ :)：

```swift
isLookingRight = true
penguin.startAnimating()

UIView.animate(withDuration: animationDuration, delay: 0,
  options: .curveEaseOut,
  animations: {
    self.penguin.center.x += self.walkSize.width
  },
  completion: nil
)
```

它与以前的代码几乎相同：您可以更改角色方向，播放帧动画并将其移向屏幕的右边缘。

构建并运行您的项目并玩游戏。 你能想到角色可以执行的其他帧动画吗？

## 播放不同的帧动画
现在Pengui可以四处走动，是时候跳跃和滑行了。 您将在本节中面对一些新的帧动画问题，并学习如何以优雅的方式解决它们。

要使图像视图播放不同的帧动画，您需要更改animationImages属性的内容。 方便的是，ViewController的slideFrames属性已经包含了滑动动画的图像。

将以下代码添加到loadSlideAnimation（）以加载新的帧序列：

```swift
penguin.animationImages = slideFrames
penguin.animationDuration = animationDuration
penguin.animationRepeatCount = 1
```

在上面的代码中，将slideFrames数组指定给animationImages，然后将动画持续时间设置为预定义的1秒时间跨度。 最后，将重复计数设置为1。

看起来您已经处理了上面所需的所有内容，因此您可以移动代码来加载和播放动画。

将以下代码添加到actionSlide（_ :)：

```swift
loadSlideAnimation()
penguin.startAnimating()
```

构建并运行您的项目; 点击屏幕右侧的滑动按钮，你会看到Pengui跳到他的肚子上并反弹回他的脚。



[图片]



嗯，这个动画中有一些奇怪的事情。 当动画运行时，Pengui得到了真正的胖乎乎的; 事实上，当他躺在地上时，他看起来完全圆了！ 是什么赋予了？

那么，首先检查最明显的事情：幻灯片图像是否正确？ 看看slide01和slide02，你会发现企鹅的形状和大小都是正确的。 所以你的项目中的某些东西必须破坏Pengui的身材。 无论是那个，或者他最近一直在吃太多鱼。

由于两个动画之间的帧图像不同，因此滑动动画会关闭：



[图片]



步行帧图像为108 x 96，而幻灯片图像为93 x 75.如果它们的大小相同，则每个图像中的空白区域最终会变大。 想象一个具有五个，六个或更多帧动画的角色; 你最终会得到巨大的图像尺寸，以适应所有可能的动画帧。

> 注意：此问题的简单解决方案是将图像视图的内容模式从其默认值“面积填充”设置为“中心”或“左上角”。 但既然你是一个动画忍者，那么你将实现一个稍微不同且更灵活的解决方案。

在本章的这一部分中，您将手动调整图像视图的大小并重新定位，以创建漂亮流动的精美动画。

首先，您需要在播放任何动画之前设置所需的图像视图帧; 这可以确保框架在屏幕上可见时尺寸正确。

将以下代码添加到actionSlide（_ :)，就在您开始动画的位置之前：

```swift
penguin.frame = CGRect(
  x: penguin.frame.origin.x,
  y: penguinY + (walkSize.height - slideSize.height),
  width: slideSize.width,
  height: slideSize.height)
```

此代码将企鹅图像视图向下移动一点以补偿幻灯片动画的较短帧，并调整图像视图的大小以匹配slideSize。

slideSize包含slide01.png的大小; viewDidLoad（）已包含获取图像的代码。

现在将以下代码添加到actionSlide（_ :)的底部：

```swift
UIView.animate(withDuration: animationDuration - 0.02, delay: 0.0,
  options: .curveEaseOut,
  animations: {
    self.penguin.center.x += self.isLookingRight ?
      self.slideSize.width : -self.slideSize.width
  },
  completion: { _ in
    // animation is complete
  } 
 )
```

在上面的代码中，您将创建一个视图动画来移动企鹅图像视图并模拟跳转动作。 您还可以添加或减去一只企鹅长度（这将是一个“penguimeter”？），具体取决于企鹅所面对的方式。

您会注意到动画持续时间有一个小的调整，而你的缩短时间会缩短。 你会在短时间内看到为什么这是必要的。

构建并运行您的项目; 你的动画在这一点上应该看起来很漂亮。



[图片]



但可怜的Pengui被困在冰上; 进行幻灯片后，即使点击左右箭头按钮，所有动画也会成为幻灯片动画。

哎呀 - 你没有重置图像视图框并重新加载步行框架。 这很容易修复。

使用以下内容替换上面添加的代码段中 //动画是完整注释：

```swift
self.penguin.frame = CGRect(
  x: self.penguin.frame.origin.x,
  y: self.penguinY,
  width: self.walkSize.width,
  height: self.walkSize.height)
self.loadWalkAnimation()
```

这会将图像视图帧重置为其初始大小，并加载步行帧序列。

从动画持续时间中减去的小时间间隔在这里发挥作用; 没有它，框架动画和运动动画同时结束，这有时会导致企鹅在调整框架之前恢复站立的小故障。 添加小偏移量可确保在上一个动画结束之前新帧已准备就绪。

最后一次构建并运行项目; 你应该看到，一旦他完成滑动，Pengui现在会回到他的直立位置。

使用UIImageView的帧动画很简单，但它们是你已经令人印象深刻的动画技能的一个很好的补充。

您可能认为UIKit不适合游戏，但您会惊讶地发现情况并非如此！ 核心动画是一个比许多其他平台更强大的引擎，正如你在本书中看到的，它可以运行粒子系统，多个动画，淡入淡出等等。

## 关键点
* 您可以通过设置其animationImages，animationDuration和animationRepeatCount属性，利用UIImageView的强大功能轻松创建帧动画。

* 通过调用startAnimating（）在UIImageView上启动帧动画，并且可以将其与同时进行的其他视图动画组合。

* 如果要替换图像视图显示的图像，请确保尺寸与当前内容相匹配，以避免在屏幕上看到压扁的图像。