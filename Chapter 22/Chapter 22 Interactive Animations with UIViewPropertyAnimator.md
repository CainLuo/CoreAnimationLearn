# Chapter 22: Interactive Animations with UIViewPropertyAnimator

您已经介绍了很多UIViewPropertyAnimator API，例如基本动画，自定义计时和弹簧，以及动画的摘要。但是，与旧式的“即发即忘”API相比，你还没有研究过什么让这个课真正有趣。

UIView.animate（withDuration：...）提供了一种在屏幕上为视图设置动画的方法，但是一旦定义了所需的结束状态，就会发送动画以进行渲染，并且控制权无法控制。
但是如果你想与动画互动怎么办？或者创建动画，这些动画不是静态的，而是由用户手势或麦克风输入驱动的，就像你在书中涉及图层动画的那部分一样？

这是UIViewPropertyAnimator在动画视图方面真正实现的地方。使用此类创建的动画是完全交互式的：您可以启动，暂停它们并改变它们的速度。最后，您可以通过直接设置当前进度来简单地“擦除”动画。

由于UIViewPropertyAnimator可以同时驱动预设动画和交互式动画，因此当谈到动画师当前的状态时，事情会变得有点复杂。本章的下一部分将教你如何处理动画师状态。

如果您已完成上一章中的挑战，请继续处理您的Xcode项目;如果您跳过了挑战，请打开本章提供的入门项目。

您应该让项目具有不同的动画，当您在搜索栏中输入文本，点击图标或展开小部件视图时，这些动画会启动。



[图片]



## 动画状态机
除了处理您的动画外，UIViewPropertyAnimator还展示了状态机的行为，并且可以为您提供有关动画当前状态的许多不同方面的信息。

您可以检查动画是否已开始，是否已暂停或完全停止，或动画是否已反转。 最后，您可以检查动画“完成”的位置，例如从所需的最终状态开始，或者介于两者之间的某个位置。

UIViewPropertyAnimator有三个属性可帮助您找出当前状态：



[图片]



isRunning属性（只读）告诉您动画师的动画当前是否处于运动状态。 默认情况下该属性为false，并在调用startAnimation（）时变为true。 如果您暂停或停止动画，或者您的动画自然完成，它将再次变为false。

默认情况下，isReversed属性为false，因为您始终以向前方向开始动画，即您的动画从其开始状态播放到结束状态。 如果将此属性更改为true，则动画将反转方向并回放到其初始状态。

状态属性（只读）确定动画师是否处于活动状态并且当前是动画还是处于某种其他被动状态。

默认情况下，状态为非活动状态 这通常意味着你刚刚创建了动画师，并且还没有调用任何方法。 请注意，这与将isRunning设置为false不同：isRunning实际上只关注正在播放的动画，而当状态处于非活动状态时，实际上意味着动画师还没有做任何事情。

当你要么：状态变得活跃：

* 调用startAnimation（）以启动动画
* 在没有开始动画的情况下调用pauseAnimation（），
* 将fractionComplete属性设置为“将动画”“回放”到某个位置。 

动画完成后，状态会切换回非活动状态。

如果在动画师上调用stopAnimation（），它会将其state属性设置为stopped。 在这种状态下，你唯一能做的就是完全放弃动画师，或者调用finishAnimation（at :)来完成动画并将动画师带回非活动状态。

正如您可能想到的那样，UIViewPropertyAnimator只能按特定顺序在状态之间切换。 它不能直接从非活动状态直接停止，也不能从停止直接转为活动状态。

在您的控制下还有一个选项：如果您设置了名为pausesOnCompletion的属性，一旦动画师完成了动画的运行而不是自动停止，它将暂停。 这将使您有机会从暂停状态继续使用它。

如果您有疑问，可以随时回到本章的这一部分，并参考下面的状态流程图：



[图片]



如果使用UIViewPropertyAnimator管理状态起初听起来有点复杂，请不要担心。 如果你打电话给一个你不允许在当前状态下打电话的方法，你的应用程序会立即崩溃，这样你就有机会找出出错的地方。

## 交互式3D触摸动画
在本章的这一部分中，您将在iPhone主屏幕上创建类似于3D触摸交互的交互式动画：



[图片]



> 注意：对于本节，您需要兼容3D触摸的iOS设备或用于模拟器的Force Touch触控板。 设备更好，因为3D触摸可以让您获得更好的控制。

当您继续按下主屏幕图标时，您将看到动画在您的手指下以交互方式进展; 背景变得越来越模糊，并且图标中出现了一个光模糊框架。

这两个动画告诉用户他们正在通过手势进行操作，并通过该动画向他们提供有关他们进度的反馈。 当您用力按压时，图标框会从图标上分离并变为菜单：



[图片]



这是一个整洁的小型交互式动画，您将在本章中重现。

> 注意：您将不会了解有关使用UIPreviewInteractionDelegate处理3D触摸的详细信息，因为本章是关于创建动画的。 如果您想了解有关UIPreviewInteractionDelegate的更多信息，请查看我们在raywenderlich.com上的iOS 10 by Tutorials一书。

打开WidgetView.swift并在WidgetView上找到符合UIPreviewInteractionDelegate的扩展。 这些是当用户按下窗口小部件视图时UIKit调用的委托方法。

为了让您开始开发动画本身，UIPreviewInteractionDelegate方法已经连接到LockScreenViewController上调用相关方法。

WidgetView中的代码如下所示：

* 3D触摸开始时调用LockScreenViewController.startPreview（for :)。
* 在用户重复调用LockScreenViewController.updatePreview（百分比:)
  压力更大（或更柔软）。
* 当peek交互成功完成时，调用LockScreenViewController.finishPreview（）。
* 最后，如果用户抬起手指而未完成预览手势，则调用LockScreenViewController.cancelPreview（）。

不用多说，让我们开始编码吧！

打开LockScreenViewController.swift并添加这三个属性，您需要这些属性才能创建窥视交互：

```swift
var startFrame: CGRect?
var previewView: UIView?
var previewAnimator: UIViewPropertyAnimator?
```

您将使用startFrame来跟踪动画的开始位置。 previewView将是您图标的快照视图; 你会在动画期间暂时使用它。

previewAnimator将成为驱动预览动画的交互式动画师。

添加一个属性以保持模糊效果以显示图标框（如上面的屏幕截图所示）：

```swift
let previewEffectView = IconEffectView(blur: .extraLight)
```

IconEffectView是初始项目中包含的自定义类。 这是一个包含单个标签的简单模糊视图。 您将使用它来模拟从按下的图标弹出的菜单，如下所示：



[图片]



向下滚动到扩展名LockScreenViewController：WidgetsOwnerProtocol并在其中插入一个新方法：

```swift
func startPreview(for forView: UIView) {
	previewView?.removeFromSuperview()
	previewView = forView.snapshotView(afterScreenUpdates: false)
	view.insertSubview(previewView!, aboveSubview: blurView)
}
```

如前所述，只要用户开始按下图标，WidgetView就会调用startPreview（for :)。 forView参数是用户开始手势的集合单元格图像。

首先删除任何现有的previewView视图，以防万一，确保不在屏幕上留下工件。 然后，您可以创建集合视图图标的快照，最后将其添加到模糊效果视图上方的屏幕上。

您可以立即运行该应用程序并开始按下图标。 您将在左上角看到图标弹出窗口的副本！



[图片]



当然，图标并未覆盖现有图标，因为您尚未设置其位置。 让我们继续构建动画：

```swift
previewView?.frame = forView.convert(forView.bounds, to: view)
startFrame = previewView?.frame
addEffectView(below: previewView!)
```

您在图标副本上设置了正确的位置，以便它完全覆盖现有图标。 然后存储该起始位置和大小以供将来在startFrame中引用。 最后，调用addEffectView（下面:)来添加图标快照下方的模糊框架。

使用下面的代码片段将addEffectView（下面:)的实现添加到LockScreenViewController，以在图标快照下方插入效果：

```swift
func addEffectView(below forView: UIView) {
  previewEffectView.removeFromSuperview()
  previewEffectView.frame = forView.frame
  forView.superview?.insertSubview(previewEffectView, belowSubview: forView)
}
```

这样就完成了动画的设置阶段。 恭喜你完成！

接下来切换到AnimatorFactory.swift以创建动画本身。 将以下方法添加到AnimatorFactory：

```swift
static func grow(view: UIVisualEffectView,
  blurView: UIVisualEffectView) -> UIViewPropertyAnimator {
// 1
  view.contentView.alpha = 0
  view.transform = .identity
// 2
  let animator = UIViewPropertyAnimator(
    duration: 0.5, curve: .easeIn)
  return animator
}
```

您的新工厂方法有两个参数：

* view：动画视图
* blurView将在主动画旁边设置动画的模糊背景。 然后执行以下操作：

1. 首先，它通过淡出视图内容（这是标记“Customize Actions ...”）并重置视图上的变换来基本化视图的当前状态。

2. 然后创建一个新的动画师，持续时间为0.5秒，并具有易于进入的时间曲线。

现在，在行返回动画师之前，您可以为此动画师添加动画和完成。 为此，请插入以下内容：

```swift
// 3
animator.addAnimations {
  blurView.effect = UIBlurEffect(style: .dark)
  view.transform = CGAffineTransform(scaleX: 1.5, y: 1.5)
}
// 4
animator.addCompletion { _ in
  blurView.effect = UIBlurEffect(style: .dark)
}
```

3. 在此处添加的动画中，在模糊视图上设置效果属性将创建一个很好的模糊过渡。 您已经在前面的章节中完成了这一点，但这次模糊将以交互方式发生，具体取决于用户按下屏幕的难度。 最后，通过简单地调整其变换属性，可以在图标框架上放大模糊。
4. 完成方法显式设置模糊视图的最终状态。 这些带有UIViewPropertyAnimator的交互式动画有时会有点错误，所以只要你的动画完成，UIKit就会调用你的完成，其中的代码将确保你的UI处于你想要的状态。

你几乎准备好了你的成长动画。 当用户按下图标时，您只需要以交互方式擦除它。

返回LockScreenViewController.swift并将以下内容追加到startPreview（）：

```swift
previewAnimator = AnimatorFactory.grow(view: previewEffectView, blurView: blurView)
```

请注意，与前几章不同，您可以创建和配置动画师，但是您不会启动动画。 这一次，您将基于3D触摸输入以交互方式驱动动画进度。

为了完成动画，请实现updatePreview（percent :)方法。 这是WidgetView将使用当前触摸力重复调用的一个：

```swift
func updatePreview(percent: CGFloat) {
  previewAnimator?.fractionComplete =
    max(0.01, min(0.99, percent))
}
```

要理解的重要方面是限制fractionComplete在0.01和0.99范围内。 如果将fractionComplete设置为0.0或1.0，则动画制作者将完成，您不希望在updatePreview内部发生这种情况。 您将从指定的方法完成或取消动画。

您现在可以尝试交互式动画！

运行应用程序，然后轻轻按下其中一个图标。 当您开始按下时，您将看到模糊框架开始“长出”图标。 模糊效果在背景中显得微妙：



[图片]



当你越来越努力地按压时，框架会不断增长，模糊变得更加突出：



[图片]



一旦您施加足够的力来完成预览手势，您将感受到手指下的触觉反馈，动画将停止在此状态。



[图片]



为什么动画会停止？ 不，这不是你的错 - 窥视手势完成，一旦它这样做，它就会停止发送更新; 也就是说，它停止调用updatePreview（percent :)方法。

接下来，您将实现取消或完成交互的方法。

你会（惊喜！）需要更多的动画师。 打开AnimatorFactory.swift并添加一个动画师，它可以解除你的“成长”动画师所做的一切。

您需要此动画师的一种情况是用户取消手势时。 当您需要清理UI时，另一个是成功交互的最后阶段。

添加新工厂方法：

```swift
static func reset(frame: CGRect, view: UIVisualEffectView,
  blurView: UIVisualEffectView) -> UIViewPropertyAnimator {
  return UIViewPropertyAnimator(duration: 0.5,
    dampingRatio: 0.7) {
        view.transform = .identity
		    view.frame = frame
    		view.contentView.alpha = 0
		    blurView.effect = nil
  }
}
```

此方法接受原始动画的起始帧，动画视图和背景blurView。 动画块将重置交互开始之前状态中的所有属性。

切换回LockScreenViewController.swift并在WidgetsOwnerProtocol扩展中添加一个新方法：

```swift
func cancelPreview() {
  if let previewAnimator = previewAnimator {
    previewAnimator.isReversed = true
    previewAnimator.startAnimation()
  }
}
```

这是WidgetView将在用户突然抬起手指时调用的方法，或者如果FaceTime调用在屏幕上弹出，并取消正在进行的手势。

到目前为止，你还没有开始你的动画师。 您一直在重复设置fractionComplete，这会以交互方式驱动动画。

但是，一旦用户取消交互，您就无法继续以交互方式驱动动画，因为您没有更多输入。 相反，您可以通过将isReversed设置为true并调用startAnimation（）来将动画播放到其初始状态。 现在这是UIView.animate（withDuration：...）无法做到的事情！

另外尝试交互。 按下动画的一半，然后继续测试cancelPreview（）。



[图片]



当您抬起手指时动画会正确播放，但最终黑暗模糊会突然重新出现。

这个问题植根于你的成长动画师的代码。 切换回AnimatorFactory.swift并查看grow中的代码（view：UIVisualEffectView，blurView：UIVisualEffectView） - 更具体地说，这部分：

```swift
animator.addCompletion { _ in
  blurView.effect = UIBlurEffect(style: .dark)
}
```

既然动画可以向前或向后播放，你需要在完成块中处理这个问题。

addCompletion（）的闭包所采用的参数是UIViewAnimatingPosition类型。 它的值可以是.start，.end或.current。

如果您的动画自然完成，或者以其他方式达到其结束状态，您将在完成闭包中获得.end值。 如果您反转动画，它将在.start位置完成。 最后，如果你在中途停止动画并在那里完成，你的完成块将获得.current值。

因此，要处理完成或取消预览手势的可能性，请删除现有的完成块并将其替换为：

```swift
animator.addCompletion { position in
  switch position {
    case .start:
      blurView.effect = nil
    case .end:
      blurView.effect = UIBlurEffect(style: .dark)
    default: break
  }
}
```

如果动画反转，则删除模糊效果。 如果成功完成，则明确将效果调整为暗模糊。

尝试几次调整动画; 确保一切按预期进行。 它应该主要是这样做的。

现在有一个新问题。 如果您取消某个图标上的按键，则无法再按下它！

这是因为图标快照仍然位于原始图标上方，并且它会吞下所有触摸。 要解决该问题，您需要在重置动画制作完成后立即删除快照。

让我们将这个代码添加到LockScreenViewController.swift中的cancelPreview（），就在previewAnimator.startAnimation（）下面：

```swift
previewAnimator.addCompletion { position in
  switch position {
    case .start:
      self.previewView?.removeFromSuperview()
      self.previewEffectView.removeFromSuperview()
    default: break
  }
}
```

请记住，对addCompletion（_ :)的调用不会替换现有的完成块，而是添加第二个。

目标是检查动画是否已被反转; 如果是这样，请从视图层次结构中删除快照和图标框。 这就是为什么你感兴趣的唯一案例是当职位是.start。

再次尝试该应用程序，您将看到取消手势后图标再次交互。万岁！ 你快到了。

让我们再添加一个动画师来显示图标菜单。 切换到AnimatorFactory.swift并添加到它：

```swift
static func complete(view: UIVisualEffectView) -> UIViewPropertyAnimator
{
  return UIViewPropertyAnimator(duration: 0.3,
    dampingRatio: 0.7) {
    view.contentView.alpha = 1
    view.transform = .identity
    view.frame = CGRect(
      x: view.frame.minX - view.frame.minX/2.5,
      y: view.frame.maxY - 140,
      width: view.frame.width + 120,
      height: 60
		)
  }
}
```

这次你创建一个简单的弹簧动画师。 对于动画师，您可以执行以下操作：

* 淡入“自定义操作”菜单项。
* 重置转换。
* 将视图框架直接设置为图标正上方的位置。

菜单的位置根据用户按下的图标而变化。

您将水平位置设置为view.frame.minX  -  view.frame.minX / 2.5，如果图标位于屏幕左侧，则显示右侧菜单，如果图标位于左侧，则显示左侧菜单 在屏幕的右侧。 看下面的差异：



[图片]



动画师已准备就绪，因此打开LockScreenViewController.swift并在WidgetsOwnerProtocol扩展中添加最后一个必需的方法：

```swift
func finishPreview() {
  // 1
  previewAnimator?.stopAnimation(false)
  // 2
  previewAnimator?.finishAnimation(at: .end)
	// 3
  previewAnimator = nil
}
```

当用户推动3D触摸手势时，就会在您感觉到触觉反馈时调用finishPreview（）。

1. stopAnimation（_ :)停止当前在屏幕上运行的动画，并根据您传入的布尔参数有两种不同的行为。

2. 当你调用stopAnimation（false）时，你将动画师置于停止状态。 等待你稍后再调用finishAnimation（at :)。 如果你调用stopAnimations（true），这将清除所有动画并将动画师置于非活动状态而不调用你的完成块。 使用此选项可完全取消动画制作者的当前动画。

一旦你将动画师置于停止状态，你就有了一些选择。 你在finishPreview（）中追求的是告诉动画师完成它的最终状态。 因此，您调用finishAnimation（at：.end）; 这将使用计划动画的目标值更新所有视图并调用您的完成。

3. 此手势不再需要previewAnimator，因此您可以将其删除。

您可以使用以下方法之一调用finishAnimation（at :)：

* start：将动画重置为初始状态。
* current：从动画的当前进度更新视图的属性并完成。

调用finishAnimation（at :)后，您的动画师处于非活动状态。

回到Widgets项目。 由于你摆脱了预览动画师，你可以运行完整的动画师来显示菜单。 将以下内容附加到finishPreview（）的末尾：

```swift
 AnimatorFactory.complete(view: previewEffectView)
  .startAnimation()
```

这将完成效果。 只要您运行应用程序并按下图标，您将看到其菜单以交互方式弹出：



[图片]



当动画结束时，菜单将沿着图标很好地定位：



[图片]



恭喜你 - 你应该轻拍一下。 这是一个复杂的发展效应！ 但要支撑自己 - 这只是一个开始！ 在下一章中，您将开始使用交互式视图控制器转换动画！

同时，通过本章提供的挑战，您可以获得更多添加和重新使用动画制作者的经验，以及使用交互式关键帧动画！

## 关键点
* 您可以通过检查state，isRunning和isReversed的值的组合来内省动画师的状态。
* 您可以通过切换isRunning来暂停动画制作者，并通过切换isReversed来反转动画的执行。
* 当您允许反转动画时，请注意在完成闭包中正确包装这些动画。 这是因为它们可能会以预期的最终状态或初始状态结束，具体取决于它们在完成时运行的方向。

## 挑战
### 挑战1：允许用户关闭菜单
一旦用户看到显示菜单的完整动画，他们就无法对该应用程序执行任何其他操作。

在此挑战中，如果用户点击模糊视图或菜单项，您将重置UI。 这将允许他们“关闭”菜单并进一步与应用程序交互。

在finishPreview（）中添加以下代码以准备交互式模糊：

```swift
blurView.effect = UIBlurEffect(style: .dark)
blurView.isUserInteractionEnabled = true
blurView.addGestureRecognizer(UITapGestureRecognizer(target: self,
action: #selector(dismissMenu)))
```

在上方，您确保将模糊效果设置为暗，并在模糊视图本身上启用用户交互。 这将允许用户点击图标周围的任何位置以关闭菜单。

最后，添加一个触摸识别器并将其连接到一个名为dismissMenu（）的方法。

下一步 - 自己添加dismissMenu（）：

* 使用AnimatorFactory.reset（frame：，view：，blurView :)动画制作来消除动画。
* 将startFrame属性用于AnimatorFactory.reset的frame参数（frame：，view：，blurView :)。
* 在启动动画师之前，添加一个完成块，从屏幕上删除previewEffectView和previewView。 还禁用模糊视图上的用户交互性，以便它不会吞下任何其他触摸。

最后，在viewDidLoad（）中，在previewEffectView上添加一个tap识别器，也连接到dismissMenu（）。 这将允许用户点击自定义操作...以关闭菜单。

运行该应用程序并尝试打开并解除菜单几次。 这不是那么快乐的老乐趣吗？

### 挑战2：交互式关键帧动画
在第20章中，您了解了向动画师添加关键帧动画是多么容易。 如果您有一个带关键帧的动画师，您仍然可以使用它来创建交互式动画。 您的用户可以来回擦洗关键帧。

要尝试一下，您将为成长动画添加一个额外的元素 - 在用户按下图标时以交互方式擦洗的动画。

打开AnimatorFactory.swift并找到增长中的位置（视图：UIVisualEffectView，blurView：UIVisualEffectView），向动画师添加动画：

```swift
animator.addAnimations {
  blurView.effect = UIBlurEffect(style: .dark)
  view.transform = CGAffineTransform(scaleX: 1.5, y: 1.5)
}
```

删除整个代码块并将其替换为：

```swift
animator.addAnimations {
  UIView.animateKeyframes(withDuration: 0.5, delay: 0.0, animations: {
    UIView.addKeyframe(withRelativeStartTime: 0.0, relativeDuration: 1.0,
animations: {
      blurView.effect = UIBlurEffect(style: .dark)
      view.transform = CGAffineTransform(scaleX: 1.5, y: 1.5)
    })
      UIView.addKeyframe(withRelativeStartTime: 0.5, relativeDuration: 0.5,
animations: {
      view.transform = view.transform.rotated(by: -.pi/8)
    })
	}) 
}
```

您可以使用两个关键帧创建动画：

* 第一个关键帧用于动画的总持续时间，并运行与之前相同的动画。
* 第二个关键帧在总持续时间的后半部分踢，并略微旋转视图。

动画中的新元素将有助于在用户完成手势时向用户提供反馈。 就在他们用力按压之前，他们会看到图标框架倾斜：



[图片]



这也将为显示菜单的完整动画添加一些趣味性：



[图片]



通过完整的交互和所有动画，您现在可以继续学习下一章，并尝试使用UIViewPropertyAnimator创建视图控制器转换。

