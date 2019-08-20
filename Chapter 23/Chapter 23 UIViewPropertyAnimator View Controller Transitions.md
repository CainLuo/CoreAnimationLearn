# Chapter 23: UIViewPropertyAnimator View Controller Transitions

在完成第17章到第19章的过程中，您学习了如何创建自定义视图控制器转换。 你看到了它们的灵活性和强大性，所以你可能很自然地想知道如何使用UIViewPropertyAnimator来创建它们。

好消息 - 使用动画师进行过渡很容易，而且几乎没有惊喜。

在本章中，您将查看构建自定义过渡动画并为Widgets项目创建静态和交互式过渡。

完成本章后，您的用户将能够通过下拉窗口小部件表来显示设置视图控制器。

如果您参与了上一章的挑战，请继续研究同一个项目; 如果您跳过了挑战，请打开本章提供的入门项目。

## 静态视图控制器转换
目前，当用户点击“编辑”按钮时，体验非常陈旧。 该按钮在当前按钮上显示一个新的视图控制器，只要您点击该第二个屏幕中的任何可用选项，它就会消失。

让我们调高一点！

创建一个新文件并将其命名为PresentTransition.swift。 将其默认内容替换为：

```swift
import UIKit

class PresentTransition: NSObject,
  UIViewControllerAnimatedTransitioning {
    
  func transitionDuration(using transitionContext:
    UIViewControllerContextTransitioning?) -> TimeInterval {
		return 0.75 
  }
    
  func animateTransition(using transitionContext:
    UIViewControllerContextTransitioning) {
	} 
}
```

您熟悉UIViewControllerAnimatedTransitioning协议，因此您应该熟悉这段代码。

>  注意：如果您跳过本书的View Controller Transitions部分，我建议您退后一步并至少完成第17章“自定义演示控制器和设备方向动画”。

在本章的这一部分中，您将创建一个过渡动画，动画模糊图层并在其上移动新的视图控制器。



[图片]



在已打开的同一文件中添加以下方法，以便为转换创建动画制作工具：

```swift
func transitionAnimator(using transitionContext:
  UIViewControllerContextTransitioning) -> UIViewImplicitlyAnimating {
  let duration = transitionDuration(using: transitionContext)
  
  let container = transitionContext.containerView
  let to = transitionContext.view(forKey: .to)!
  container.addSubview(to)
}
```

在上面的代码中，您为视图控制器转换做了所有必要的准备工作。 首先获取动画持续时间，然后获取目标视图控制器的视图，最后将此视图添加到过渡容器中。

接下来，您可以设置动画并运行它。 将此代码添加到transitionAnimator（使用:)以准备过渡动画的UI：

```swift
to.transform = CGAffineTransform(scaleX: 1.33, y: 1.33)
  .concatenating(CGAffineTransform(translationX: 0.0, y: 200))
to.alpha = 0
```

这会向上扩展并向下移动目标视图控制器的视图并将其淡出。 现在它已经准备好在屏幕上动画了。

在to.alpha = 0之后添加动画师来运行转换：

```swift
let animator = UIViewPropertyAnimator(duration: duration,
curve: .easeOut)
animator.addAnimations({
  to.transform = CGAffineTransform(translationX: 0.0, y: 100)
}, delayFactor: 0.15)
animator.addAnimations({
  to.alpha = 1.0
}, delayFactor: 0.5)
```

在此代码中，您将创建一个包含两个动画块的动画师：

1. 第一个动画将目标视图控制器的视图移动到其最终位置。

2. 第二个动画将内容从0到1的alpha消失。

与前面的章节一样，您永远不应忘记完成转换。 向动画师添加完成：

```swift
animator.addCompletion { _ in
  transitionContext.completeTransition(
    !transitionContext.transitionWasCancelled
  )
}
```

动画完成后，让UIKit知道您已完成转换。 在您的方法结束时，只需返回动画师：

```swift
return animator
```

现在您已经拥有了动画工厂方法，您还必须使用它。 向上滚动到animateTransition（使用:)并插入以下代码：

```swift
transitionAnimator(using: transitionContext).startAnimation()
```

这将获取一个随时可用的动画师，并通过startAnimation（）开始。

那应该暂时做到。 让我们将视图控制器连接到过渡动画师并尝试动画。

打开LockScreenViewController并定义以下常量属性：

```swift
let presentTransition = PresentTransition()
```

当UIKit要求您提供演示动画和交互控制器时，您将向UIKit提供此对象。 为此，请向LockScreenViewController添加UIViewControllerTransitioningDelegate一致性：

```swift
extension LockScreenViewController:
  UIViewControllerTransitioningDelegate {
  func animationController(
    forPresented presented: UIViewController,
    presenting: UIViewController, source: UIViewController) ->
    UIViewControllerAnimatedTransitioning? {
    return presentTransition
  }
}
```

animationController（forPresented：presents：source :)方法是您有机会告诉UIKit您计划产生新的自定义视图控制器转换的地方。 您从该方法返回presentTransition，UIKit使用它来跟随动画。

现在进行最后一步 - 您需要将LockScreenViewController设置为演示文稿委托。 滚动到presentSettings（_ :)，然后在调用present（_：animated：completion :)之前将self设置为转换委托。

完成的代码应如下所示：

```swift
settingsController = storyboard?.instantiateViewController(
  withIdentifier: "SettingsViewController") as? SettingsViewController
settingsController.transitioningDelegate = self
present(settingsController, animated: true, completion: nil)
```

这应该是它！ 运行应用程序并点击“编辑”按钮尝试转换。

最初的结果并非令人兴奋（至少还没有！）。 设置控制器似乎有点偏：



[图片]



你会想要处理一些粗糙的边缘，但你的工作几乎已经完成了。

要纠正的第一件事是目标视图控制器不需要纯色背景。 打开Main.storyboard（它在Assets项目文件夹中）并选择设置视图控制器视图。



[图片]



将视图的背景更改为“清除颜色”，您应该看到故事板反映了这样的更改：



[图片]



再试一次这种转变。 这次您应该看到设置视图控制器的内容直接显示在锁定屏幕上：



[图片]



看起来这种过渡可以用更多的动画来做。 例如，淡化小部件顶部的模糊，以便用户可以更好地看到顶部的模态视图控制器，这不是很好吗？

既然你已经是专业人士了，那就让我们做一些新的事 - “动画注入”！ （不需要看那个术语 - 我刚刚为本章想出了它）。

您将向动画师添加一个新属性，允许您将任何自定义动画注入过渡。 这将允许您使用相同的过渡类来生成稍微不同的动画。

切换到PresentTransition.swift并添加一个新属性：

```swift
var auxAnimations: (()-> Void)?
```

在返回之前将此附加到transitionAnimator（使用:)的底部：

```swift
if let auxAnimations = auxAnimations {
  animator.addAnimations(auxAnimations)
}
```

如果您已向对象添加任意任意动画块，它们将被添加到动画制作者动画的其余部分。

这允许您根据具体情况在转换中添加自定义动画。 例如，让我们为当前转换添加模糊动画。

打开LockScreenViewController并在presentSettings（）的顶部插入以下内容：

```swift
presentTransition.auxAnimations = blurAnimations(true)
```

这会将您在很多章节前创建的模糊动画添加到视图控制器转换中！

再试一次过渡，看看这一行如何改变它：



[图片]



重复使用动画简直太神奇了吗？

现在，当用户解除显示的控制器时，您还需要隐藏模糊。 SettingsViewController已经有一个didDismiss属性，所以你只需要将该属性设置为一个动画模糊的块。

在settingsController出现之前的倒数第二行的presentSettings（_ :)中，插入：

```swift
settingsController.didDismiss = { [unowned self] in
  self.toggleBlur(false)
}
```

现在点击设置屏幕中的一个选项将忽略它。 然后模糊将消失，用户将成功恢复到第一个视图控制器：



[图片]



本章的这一部分到此结束。 您的视图控制器转换准备就绪！

## 交互视图控制器转换
作为本书UIViewPropertyAnimator部分的最后一个主题，您将创建一个交互式视图控制器转换。 您的用户将通过下拉窗口小部件表来推动转换。

首先，让我们使用强大的UIPercentDrivenInteractionTransition类来启用视图控制器转换的交互性。

打开PresentTransition.swift并替换：

```swift
class PresentTransition: NSObject, UIViewControllerAnimatedTransitioning
```

为:

```swift
class PresentTransition:
  UIPercentDrivenInteractiveTransition,
  UIViewControllerAnimatedTransitioning
```

UIPercentDrivenInteractiveTransition是一个定义基于“百分比”的转换方法的类，例如：

* update(_ :)以回退过渡。
* cancel（）取消视图控制器转换。
* finish（）播放转换直到完成。

您可能还记得第19章“交互式UINavigationController转换”中的这些内容，但您没有看到一些专门用于使用UIViewPropertyAnimator的高级API。

一些新功能构建的动画过渡更容易包括：

* timingCurve：如果您的用户以交互方式驱动转换，并且在需要播放转换时直到结束，则可以通过设置此属性为动画提供自定义时序曲线。 这可以是立方体，弹簧或其他自定义计时提供者。
* wantsInteractiveStart：默认情况下这是真的，因为您可能主要将此类用于交互式转换。 但是，如果将该属性设置为false，则转换将以非交互方式启动，您可以暂停它并稍后转到交互模式。
* pause（）：调用此方法暂停非交互式转换并切换到交互模式。

向PresentTransition添加一个新方法：

```swift
func interruptibleAnimator(using transitionContext:
  UIViewControllerContextTransitioning) -> UIViewImplicitlyAnimating {
  return transitionAnimator(using: transitionContext)
}
```

这是UIViewControllerAnimatedTransitioning协议上的一个方法。 它允许您向UIKit提供可中断的动画师，它将用于您的过渡动画。

您的过渡动画师课程现在有两种不同的行为：

1. 如果以非交互方式使用它（当用户按下编辑按钮时），UIKit将调用animateTransition（使用:)来设置转换动画。
2. 如果以交互方式使用它，UIKit将调用interruptibleAnimator（使用:)，让你的动画师，并用它来推动这种过渡。



[图片]



切换到LockScreenViewController.swift并在UIViewControllerTransitioningDelegate扩展中添加此新方法：

```swift
func interactionControllerForPresentation(using animator:
  UIViewControllerAnimatedTransitioning)
  -> UIViewControllerInteractiveTransitioning? {
  return presentTransition
}
```

这将让UIKit知道您在视图控制器演示期间计划一些有趣的交互性。

接下来，仍然在您打开的文件中，添加两个新属性; 您将需要它们来跟踪用户的手势：

```swift
var isDragging = false
var isPresentingSettings = false
```

当用户拉下表时，你会将isDragging标志设置为true，一旦用户拉得足够远，你将依次将isPresentingSettings设置为true。

要跟踪用户拉取表视图的距离，您需要实现一些滚动视图委托方法。 添加新扩展并插入第一个方法：

```swift
extension LockScreenViewController: UIScrollViewDelegate {
  func scrollViewWillBeginDragging(_ scrollView: UIScrollView) {
    isDragging = true
  }
}
```

这可能看起来有点多余，因为UITableView已经有一个属性来跟踪它当前是否被拖动，但这次你要自己做一些自定义跟踪。

接下来添加委托方法以跟踪用户的进度：

```swift
func scrollViewDidScroll(_ scrollView: UIScrollView) {
  guard isDragging else {
		return
	}
  
  if !isPresentingSettings && scrollView.contentOffset.y < -30 {
    isPresentingSettings = true
    presentTransition.wantsInteractiveStart = true
    presentSettings()
		return
	} 
}
```

首先，检查是否启用了isDragging标志; 否则你不想跟踪表视图的偏移量。 然后检查用户是否拉得足够远以开始转换。

如果两个条件都为真，则准备转换设置。 将isPresentingSettings设置为true，将过渡动画设置为交互模式，最后调用presentSettings（）。

presentSettings（）负责在交互模式下启动视图控制器转换，因为您事先将wantsInteractiveStart设置为true。

接下来，您需要添加代码以交互方式更新它。 将以下内容追加到scrollViewDidScroll（_ :)的末尾：

```swift
if isPresentingSettings {
  let progress = max(0.0, min(1.0, ((-scrollView.contentOffset.y) - 30) /
90.0))
  presentTransition.update(progress)
}
```

您可以根据用户拉出表视图的距离计算0.0到1.0范围内的进度，并在过渡动画师上调用update（_ :)以将动画定位到当前进度。

立即尝试过渡，当您向下拖动时，您将看到表格视图逐渐模糊。



[图片]



您还需要注意完成和取消过渡。 添加到与以前相同的扩展名：

```swift
func scrollViewWillEndDragging(_ scrollView: UIScrollView, withVelocity
velocity: CGPoint, targetContentOffset: UnsafeMutablePointer<CGPoint>) {
  let progress = max(0.0, min(1.0, ((-scrollView.contentOffset.y) - 30) /
90.0))
  if progress > 0.5 {
    presentTransition.finish()
} else {
    presentTransition.cancel()
  }
  isPresentingSettings = false
  isDragging = false
}
```

此代码看起来应该类似; 它与您在第19章“交互式UINavigationController转换”中使用的方法相同。 如果用户已经超过了一半的距离（您决定“足够远”），则认为转换成功并将动画播放到最后。 如果用户未拖动超过一半的距离，则取消转换。 无论哪种方式，您重置两个标志的值，并且转换的交互部分结束。

然而，这种转变尚未完成 - 在生产准备就绪之前，还有很多事情需要改进。

切换到PresentTransition.swift并找到transitionAnimator（使用:)。 在完成块中，忽略该参数并始终使用相同的值调用completeTransition（_ :)。

您可以通过检查动画师完成的位置并提供相关值来帮助UIKit。 将现有的addCompletion（...）替换为：

```swift
animator.addCompletion { position in
  switch position {
  case .end:
    transitionContext.completeTransition(!
transitionContext.transitionWasCancelled)
default:
    transitionContext.completeTransition(false)
  }
}
```

实际上，只有动画师在其.end位置完成时，视图控制器转换才成功。 任何其他情况都意味着转换已被取消，因此您可以直接调用completeTransition（false）。 建立并运行并尝试上下摆动桌子 - 甜蜜！

但是有一个问题。 想一想你的非交互式过渡。 点按“编辑”。 出了点问题！

每当用户点击按钮时，您需要更改代码以将视图控制器转换显式设置为非交互式。

切换回LockScreenViewController.swift并找到小部件表数据源方法tableView（_：cellForRowAt :)。 您将看到代码为“编辑”按钮分配了一个闭包，如下所示：

```swift
cell.didPressEdit = {[unowned self] in
  self.presentSettings()
}
```

就在self.presentSettings（）行之前，插入：

```swift
self.presentTransition.wantsInteractiveStart = false
```

这可确保您以非交互方式呈现设置视图控制器。 再次运行应用程序并尝试转换。

## 可中断的过渡动画
接下来，您将考虑在过渡期间在非交互模式和交互模式之间切换。 UIViewPropertyAnimator与视图控制器转换的集成旨在解决用户开始转换到另一个控制器的情况，但在中途改变他们的想法。

在本章的这一部分中，您将添加代码以在点击编辑后开始显示设置控制器，但如果用户在动画期间再次点击屏幕，则暂停转换。

切换到PresentTranstion.swift。 您需要稍微改变动画师，以便分别处理交互式和非交互式模式，但同时也处理相同的过渡。

再添加两个属性：

```swift
var context: UIViewControllerContextTransitioning?
var animator: UIViewPropertyAnimator?
```

您将使用这些来跟踪转换的当前上下文以及动画师处理其动画。

向下滚动到transitionAnimator（使用:)，并在返回动画线之前的底部，插入以下内容：

```swift
self.animator = animator
self.context = transitionContext
```

每次为转换创建新的动画师时，您还会存储对它的引用。

转换完成后释放这些资源也很重要。 在之前插入的两行之后添加一个新的完成块：

```swift
animator.addCompletion { [unowned self] _ in
  self.animator = nil
  self.context = nil
}
```

现在，您可以向类添加一个方法来中断转换：

```swift
func interruptTransition() {
  guard let context = context else {
		return
  }
  context.pauseInteractiveTransition()
  pause() 
}
```

您可以调用pauseInteractiveTransitioning（）来暂停动画师，并在过渡动画师上暂停（）以使其处于交互模式。

要在非交互式转换期间允许触摸，您必须明确将动画设置为能够处理用户活动。 滚动回transitionAnimator（使用:)并将此行插入底部：

```swift
animator.isUserInteractionEnabled = true
```

您确保过渡动画是交互式的，以便用户在暂停后继续与屏幕交互。

您将允许用户向上或向下滚动以分别完成或取消转换。 为此，切换回LockScreenViewController.swift并添加一个新属性：

```swift
var touchesStartPointY: CGFloat?
```

如果用户在转换期间触摸屏幕，您可以暂停它并存储第一次触摸的位置：

```swift
override func touchesBegan(_ touches: Set<UITouch>,
  with event: UIEvent?) {
  guard presentTransition.wantsInteractiveStart == false,
    presentTransition.animator != nil else {
			return
	}
  touchesStartPointY = touches.first!.location(in: view).y
  presentTransition.interruptTransition()
}
```

您可以检查在非交互式转换期间是否发生了触摸，并且转换的动画师当前正在运行。

在这种情况下，您存储当前触摸位置，然后调用自定义方法interruptTransition（）。

这将暂停转换并使其保持交互模式。

运行应用程序，点击编辑，然后再次快速再次点击。 转换将在屏幕上冻结，如下所示：



[图片]



现在，您需要跟踪用户触摸并查看用户是向上还是向下平移。 添加以下内容：

```swift
override func touchesMoved(_ touches: Set<UITouch>,
  with event: UIEvent?) {
  guard let startY = touchesStartPointY else {
		return
	}
  let currentPoint = touches.first!.location(in: view).y
  if currentPoint < startY - 40 {
    touchesStartPointY = nil
    presentTransition.animator?.addCompletion { _ in
      self.blurView.effect = nil
    }
    presentTransition.cancel()
  } else if currentPoint > startY + 40 {
    touchesStartPointY = nil
    presentTransition.finish()
  }
}
```

有了这么大的代码，您的非交互式转换交互式转换就完成了！

你观察了两种不同的情况。 首先，如果用户将其触摸向下移动超过40个点，则取消转换并重置模糊效果。 如果用户将其触摸向上移动超过40点，则表示您已成功完成转换。

最后一次尝试应用程序。 点击编辑，再次点击以暂停转换，并根据您平移的方向取消或完成转换。

这就是本书的这一部分！

你已经学到了很多关于UIViewPropertyAnimator以及如何充分利用它的知识。 你已经完成了相当冗长的四章，但是你取得了很多成就，而且项目看起来很棒：



[图片]



## 关键点
* 通过结合您对自定义过渡的了解以及使用UIViewPropertyAnimator创建交互式，可中断动画，您可以创建令人惊叹的过渡效果。