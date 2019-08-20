# Chapter 18: UINavigationController Custom Transition Animations

通过UINavigationController使用导航堆栈是让应用程序的用户浏览应用程序UI的常用方法。 将新的视图控制器推入导航堆栈或弹出一个视图控制器可以为您提供时尚的动画，而无需您的工作。 一个新的屏幕来自右侧，并略微滞后推开旧的屏幕：



[图片]



上面的屏幕截图显示了iOS如何将新视图控制器推送到设置应用中的导航堆栈：新视图从右侧滑入以覆盖旧视图，新标题淡入，而旧标题标题从视图中消失。

iOS中的导航范例已经成为用户的老头，因为相同的动画已经使用了很多年。 这使您可以在不抛弃用户的情况下修饰导航控制器转换。

与上一章中构建自定义呈现视图控制器的方式大致相同，您可以构建自定义过渡以推送和弹出新视图控制器。

您将使用Logo Reveal项目。 在本章中，您将添加一个自定义透明视图，让用户可以瞥见隐藏在后面的内容：



[图片]



如果您完成了上一章，您会发现自定义导航控制器转换与呈现视图控制器非常相似。

## 介绍Logo Reveal
打开本章的入门项目，然后选择Main.storyboard。 您将看到项目已从标准Master Detail应用程序模板创建。 它具有导航控制器，主视图控制器和详细视图控制器，如下所示：



[图片]



导航已经为您着迷，因此您可以专注于自定义导航控制器。

构建并运行您的项目; 点击默认屏幕（MasterViewController）上的任意位置以显示假期装箱单（DetailViewController）：



[图片]



## 自定义导航过渡
UIKit允许您通过委托模式自定义导航过渡，其方式与呈现视图控制器的方式几乎相同。

您将使MasterViewController类采用UINavigationControllerDelegate协议并将其设置为导航控制器的委托。 每次将视图控制器推送到导航堆栈时，导航控制器都会询问其代理是否应使用内置转换或自定义转换，如下所示：



[图片]



当您按下或弹出视图控制器时，导航控制器会要求其委托为该操作提供动画控制器。

如果从该委托方法返回nil，导航控制器将使用默认转换。 但是，如果返回一个对象，导航控制器将使用它作为自定义过渡动画控制器。 是的 - 这听起来很像前一章，不是吗？

动画控制器应采用您在上一章中使用的相同UIViewControllerAnimatedTransitioning协议。 一旦提供了动画控制器对象（或动画师），导航控制器就会调用以下方法：



[图片]



首先，导航控制器调用transitionDuration（）以找出转换将持续多长时间; 然后它调用animateTransition（），这是您的自定义过渡动画代码将存在的位置。



## 导航控制器委托
在实现委托方法之前，您需要创建动画类的基本框架。

从Xcode的主菜单中选择File \ New \ File ...并选择模板iOS \ Source \ Cocoa Touch Class。

将新类名设置为RevealAnimator并使其成为NSObject的子类。

使新类符合UIViewControllerAnimatedTransitioning协议像这样：

```swift
class RevealAnimator: NSObject, UIViewControllerAnimatedTransitioning {
}
```

现在，您需要实现两个必需的UIViewControllerAnimatedTransitioning方法来解决Xcode的错误消息。

将以下属性添加到类中：

```swift
let animationDuration = 2.0
var operation: UINavigationController.Operation = .push
```

你的动画将持续两秒钟。 在UI导航世界中已经有很长一段时间了，但它会让你以微小的细节看到你的动画。 operation是UINavigationController.Operation类型的属性，用于指示您是在推送还是弹出视图控制器。

现在将以下两个UIViewControllerAnimatedTransitioning方法添加到类中：

```swift
func transitionDuration(using transitionContext:
UIViewControllerContextTransitioning?) -> TimeInterval {
  return animationDuration
}

func animateTransition(using transitionContext:
UIViewControllerContextTransitioning) {
}
```

transitionDuration（）以秒为单位返回动画持续时间，而animateTransition（）是自定义动画的未来主页。 完成导航控制器委托的设置后，您将填充animateTransition（）。

到目前为止，所有Xcode错误都应该已经解决了。 打开MasterViewController.swift，它将作为导航控制器委托。

在类定义之外添加下面的代码到文件的底部：

```swift
extension MasterViewController: UINavigationControllerDelegate {
}
```

这在新的扩展中采用了UIViewNavigationControllerDelegate协议; 此视图控制器现在可以充当导航控制器委托。

在调用任何segues或将某些内容推送到堆栈之前，您需要在视图控制器生命周期的早期设置导航控制器的委托。

将以下代码添加到viewDidLoad（）：

```swift
navigationController?.delegate = self
```

您的下一个任务是创建一个RevealAnimator实例，并在被要求提供动画控制器时将其传递给导航控制器。

将以下属性添加到MasterViewController：

```swift
let transition = RevealAnimator()
```

这是您用于推动和弹出视图控制器的动画师。

现在您已经拥有了动画师，请将以下方法添加到类扩展中：

```swift
func navigationController(_
  navigationController: UINavigationController,
  animationControllerFor
  operation: UINavigationController.Operation,
  from fromVC: UIViewController,
  to toVC: UIViewController) ->
  UIViewControllerAnimatedTransitioning? {
  transition.operation = operation
  return transition
}
```

这是一个方法名称的怪物，但是如果你切断了噪音，你会发现它归结为以下参数：

* navigationController：当您的对象是多个导航控制器的委托时，这有助于区分导航控制器; 这不太可能，但你仍然需要防范这种可能性。

* operation：这是一个UINavigationController.Operation值，可以是.push或.pop。

* fromVC：这是当前在屏幕上可见的视图控制器; 它通常是导航堆栈中的最后一个视图控制器。

* toVC：这是您将转换到的视图控制器。

如果您支持不同视图控制器的不同过渡，则可以选择要返回的动画对象类型。 为了简化此项目，在设置动画师的操作属性以指示推送或弹出过渡后，您将始终返回RevealAnimator对象。

构建并运行您的项目; 点击第一个视图控制器，您将看到导航栏动画超过两秒钟。



![图片]



请注意，导航栏的更新会持续到您在RevealAnimator中指定的持续时间 - 但此时不会发生任何其他情况。 您的动画师控制转换，但由于您没有在animateTransition（）中编写任何代码，因此不会对内容进行动画处理。

但是，至少这表明导航控制器正在调用您的自定义转换。 现在是时候动画了！

## 添加自定义显示动画
自定义过渡动画的计划相对简单。 您只需在DetailViewController上为蒙版设置动画，使其看起来像RW徽标的透明部分，显示底层视图控制器的内容。

你将不得不处理图层和一些动画任务，但是到目前为止你还没有完成任务。 对于像你这样的动画专业人士来说，创建过渡动画将是一件轻松的事！

打开RevealAnimator.swift并添加以下属性：

```swift
weak var storedContext: UIViewControllerContextTransitioning?
```

由于您要为转换创建一些图层动画，因此您需要将动画上下文存储在某处，直到动画结束并执行委托方法animationDidStop（_：finished :)。 此时，您将从animationDidStop（）中调用completeTransition（）来结束转换。

将以下代码添加到animateTransition（）以存储转换上下文以供以后使用：

```swift
storedContext = transitionContext
```

> 注意：如果您向前跳过，可以在第17章“自定义演示控制器和设备方向动画”中从上下文和动画容器视图中了解有关如何获取转换视图控制器的更多详细信息。

现在将以下初始转换代码添加到animateTransition（）：

```swift
let fromVC = transitionContext.viewController(forKey:
  .from) as! MasterViewController
let toVC = transitionContext.viewController(forKey:
  .to) as! DetailViewController

transitionContext.containerView.addSubview(toVC.view)
toVC.view.frame = transitionContext.finalFrame(for: toVC)
```

由于您最初将处理推送转换，因此您可以对转换的“从”和“到”视图控制器的身份进行假设。

首先，您获取“from”视图控制器（fromVC）并将其转换为MasterViewController; 然后，您将获取到CVC作为DetailViewController。

最后，您只需将VC.view添加到转换容器视图，并将其框架设置为transitionContext中的“final”框架。 这将假期装箱单放在主屏幕上的最终位置。

现在你要创建揭示动画。 揭示动画的秘诀是让一个物体 - 在你的情况下，RW徽标 - 增长到覆盖整个屏幕区域。

这听起来像是规模转换的工作！ 将以下内容添加到animateTransition（）：

```swift
let animation = CABasicAnimation(keyPath: "transform")
animation.fromValue =
  NSValue(caTransform3D: CATransform3DIdentity)
animation.toValue =
  NSValue(caTransform3D:
  CATransform3DConcat(
    CATransform3DMakeTranslation(0.0, -10.0, 0.0),
    CATransform3DMakeScale(150.0, 150.0, 1.0)
  )
)
```

此动画将徽标的大小增加150倍，同时将其向上移动一点。 为什么？ 徽标形状不均匀，您希望后面的视图控制器通过RW形状的“孔”显示。 将其向上移动一点意味着缩放图像的底部将更快地覆盖屏幕。

下图显示了缩放动画的工作方式：



[图片]



如果你使用像圆形或椭圆形的对称形状，你就不会有这个问题，但你的动画不会那么酷。

现在将以下行添加到animateTransition（）以稍微优化动画：

```swift
animation.duration = animationDuration
animation.delegate = self
animation.fillMode = .forwards
animation.isRemovedOnCompletion = false
animation.timingFunction = CAMediaTimingFunction(name:
.easeIn)
```

首先，设置动画的持续时间以匹配转换持续时间。 然后，您将动画师设置为委托，并配置动画模型以将动画保留在屏幕上; 这样可以避免在转换过程中出现故障，因为无论如何都会隐藏RW徽标。 最后，添加缓动以使揭示效果随着时间的推移而加速。

RevealAnimator目前不是动画委托，因此跳转到文件的顶部并将CAAnimationDelegate协议添加到类定义中，如下所示：

```swift
class RevealAnimator: NSObject, UIViewControllerAnimatedTransitioning,
CAAnimationDelegate {
...
}
```

这应该清除您当前的编译器错误。

您的动画已完成 - 但应该应用哪个图层？

将以下代码添加到animateTransition的末尾（使用:)：

```swift
let maskLayer: CAShapeLayer = RWLogoLayer.logoLayer()
maskLayer.position = fromVC.logo.position
toVC.view.layer.mask = maskLayer
maskLayer.add(animation, forKey: nil)
```

这将创建一个应用于DestinationViewController的CAShapeLayer。 maskLayer与MasterViewController上的“RW”徽标位于相同的位置。 然后，您只需将maskLayer设置为视图控制器视图的掩码。

然后，您可以将动画添加到遮罩层 - 这意味着您可以测试转换的当前状态。

构建并运行您的项目以查看目前的情况：



[图片]



不错，不错; 你的曝光正在运行，但动画有点笨拙，一旦你将包列表推到最顶层，你就无法回到主屏幕。 是时候解决这些问题！

## 照顾粗糙的边缘
您可能已经注意到您仍然可以看到缩放显示徽标背后的原始徽标。 处理此问题的最简单方法是在原始徽标上运行显示动画。 你已经有了动画，所以无论重复使用它。 这将使原始徽标与面具一起生长，完全匹配其形状，因此不会妨碍。

将以下内容添加到animateTransition（）：

```swift
fromVC.logo.add(animation, forKey: nil)
```

再次构建并运行您的项目，以验证原始徽标不再处于闲置状态。



[图片]



现在，一个稍微难以解决的问题：在第一次推送转换后导航不再工作的原因是什么？

如果你看一下你到目前为止所做的事情，你会发现你从未真正完成过渡。 你计划在动画结束时调用completeTransition（） - 但是从来没有开始实现该代码。

RevealAnimator被设置为您的揭示动画的代表。 因此，您需要覆盖animationDidStop（_：finished :)并完成该方法中的转换。

将以下代码添加到RevealAnimator：

```swift
func animationDidStop(_ anim: CAAnimation, finished flag: Bool) {
  if let context = storedContext {
    context.completeTransition(!context.transitionWasCancelled)
//reset logo
}
  storedContext = nil
}
```

在这里，检查是否有存储的转换上下文; 如果是这样，你可以调用completeTransition（）。 这将球传回导航控制器以完成UIKit侧的过渡。

在方法结束时，您只需将转换上下文的引用设置为nil。

由于揭示动画在完成后不会自动删除，因此您需要自己处理。

使用以下内容替换位于animationDidStop（）中的// reset logo注释：

```swift
let fromVC = context.viewController(forKey: .from)
  as! MasterViewController
fromVC.logo.removeAllAnimations()
```

由于在推送过渡期间唯一需要屏蔽视图控制器内容的时间，因此一旦视图控制器完成转换，您就可以安全地移除屏蔽。

在animationDidStop（）中的removeAllAnimations（）下面添加以下代码：

```swift
let toVC = context.viewController(forKey: .to)
  as! DetailViewController
toVC.view.layer.mask = nil
```

这将在视图出现并转换完成后删除掩码。

应该这样做。 构建并运行您的项目; 将装箱单视图控制器推到屏幕上，然后点击开始返回原始视图。 以下引人注目的崩溃表明您正在调用自定义转换的证据：



[图片]



您正在尝试将“从”视图控制器转换为MasterViewController实例 - 这仅适用于推送转换，但不适用于弹出转换。哎呦。

打开RevealAnimator.swift并找到animateTransition（）。 您需要在条件中包含大部分代码。 在设置storedContext的第一行之后添加以下行：

```swift
if operation == .push {
```

然后，一直滚动到方法的末尾，并为方法的最末端的if添加一个右括号。

在您尝试强制转换之前，该条件会检查您是否正在处理推送转换。 这应该照顾那段崩溃的代码。

构建并运行以再次尝试。 您的推送转换正在运行，但是您的弹出转换将不会执行任何操作，因为您在animateTransition（）中没有任何代码来处理它。

这是你弯曲你的忍者编码肌肉的地方; 您将在下面的挑战部分中自己创建弹出过渡 - 并为沿途的揭示动画添加一些优雅。

## 关键点
* 要启用和自定义自定义导航过渡，请在其中一种类型中采用UINavigationControllerDelegate协议，并将其设置为导航委托。
* 要执行任何自定义导航过渡动画，请使用UIViewControllerAnimatedTransitioning协议作为过渡动画师。
* 只要正确结束转换并在最后调用必要的UIKit方法，就可以成功使用图层动画进行自定义转换。
* 要保留导航转换的上下文并在转换的持续时间内异步访问它，可以将其保留在动画类型中的UIViewControllerContextTransitioning属性中。

## 挑战
### 挑战1：淡入新视图控制器
现在过渡看起来像一个尖锐的镂空; 新视图控制器的内容立即可见，使整个动画看起来有点笨重。

你的挑战是在揭示动画运行时淡入新的视图控制器。

为此，请在CABasicAnimation中创建淡入淡出并将其添加到toVC.view层。 对此新动画使用相同的转换持续时间，并将fromValue和toValue设置为从完全透明到完全不透明的动画。

在将显示动画添加到徽标和遮罩层的位置之后调用新动画。

完成后，您的转换应如下所示：



[图片]



看起来你的剪影片随着它的增长逐渐变得更加透明; 这给过渡带来了神秘的效果。

### 挑战2：添加弹出过渡
要创建弹出过渡，只需在RevealAnimator的animateTransition（）内的if语句中添加一个互补的else分支。在else分支中你可以添加你想要的任何动画，但是当你完成时不要忘记调用completeTransition（）。以下是创建简单收缩过渡的方法：

* 在animateTransition（）内部添加else分支。
* 使用上下文中的viewForKey方法获取“from”和“to”视图。因为这次你没有对视图控制器的属性做任何事情，所以你可以单独获取视图。
* 使用转换容器视图并从View中插入下面的toView。提示：使用insertSubview（_：belowSubview：_）。
* 使用动画从View缩放到0.01。不要使用0.0进行缩放 - 这会让UIKit感到困惑。对于此动画，您可以使用普通视图动画 - 无需创建图层动画。
* 动画结束后，就像之前一样调用转换上下文中的completeTransition（）。

这将导致以下有品味的缩小过渡：



[图片]



您可能已经猜到，您也可以为UITabBarController创建自定义过渡。 你不会在这里介绍它们，但是它们的工作方式与导航控制器转换类似，因此你可以根据你到目前为止学到的东西轻松找出它们。

下一章将过渡到下一个级别，并向您展示如何让您的用户与过渡本身进行交互！