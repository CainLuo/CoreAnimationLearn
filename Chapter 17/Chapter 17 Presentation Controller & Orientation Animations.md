# Chapter 17: Presentation Controller & Orientation Animations

呈现视图控制器是将用户焦点从一个包含的UI模块“移动”到另一个UI的常用方式。 当用户完成使用所呈现的UI时，他们将选择保存他们的工作，取消挂起的更改，或以其他方式主动关闭UI。

无论您是展示摄像机视图控制器，地址簿还是自定义设计的模式屏幕，每次都调用相同的UIKit方法：present（_：animated：completion :)。 此方法将当前屏幕“放弃”到另一个视图控制器。 默认演示动画只是将新视图向上滑动以覆盖当前视图。 下图显示了一个“新联系人”视图控制器向上滑动联系人列表：



[图片]



在本章中，您将创建自己的自定义演示控制器动画，以替换默认的动画，并使本章的项目更加生动。



## 浏览入门项目
打开本章的入门项目，一个名为Beginner Cook的应用程序。 选择Main.storyboard开始游览：



[图片]



第一个视图控制器（ViewController）包含应用程序的标题和主要描述以及底部的滚动视图，其中显示了可用草药的列表。

每当用户点击列表中的一个图像时，主视图控制器就会显示HerbDetailsViewController; 此视图控制器运行背景，标题，描述和一些按钮来信任图像所有者。

ViewController.swift和HerbDetailsViewController.swift中已经有足够的代码来支持基本的应用程序。

构建并运行项目以查看应用程序的外观和感觉：



[图片]



点击其中一个药草图像，然后通过标准垂直覆盖过渡显示详细信息屏幕。 对于您的花园种类的应用程序可能没问题，但您的草药应该更好！

您的工作是为您的应用添加一些自定义演示控制器动画，以使其开花！ 您将使用将点击的草本图像扩展为全屏视图的方式替换当前的股票动画，如下所示：



[图片]



卷起袖子，打开你的开发围裙，为自定义演示控制器的内部工作做好准备！

## 在自定义过渡的幕后
UIKit允许您通过委托模式自定义视图控制器的演示文稿; 您只需使用主视图控制器（或您专门为此目的创建的另一个类）采用UIViewControllerTransitioningDelegate。

每次呈现新的视图控制器时，UIKit都会询问其代理是否应该使用自定义转换。 以下是自定义过渡舞蹈的第一步：



[图片]



UIKit调用animationController（forPresented：presents：source :)来查看是否返回了UIViewControllerAnimatedTransitioning。 如果该方法返回nil，则UIKit使用内置转换。 如果UIKit接收到UIViewControllerAnimatedTransitioning对象，则UIKit将该对象用作转换的动画控制器。

在UIKit可以使用自定义动画控制器之前，舞蹈中还有一些步骤：



[图片]



UIKit首先要求您的动画控制器（简称为动画师）以秒为单位转换持续时间，然后调用animateTransition（使用:)。 这是您的自定义动画成为焦点的时候。

在animateTransition（使用:)中，您可以访问屏幕上的当前视图控制器以及要显示的新视图控制器。 您可以随意淡化，缩放，旋转和操纵现有视图和新视图。

现在您已经了解了自定义演示控制器的工作原理，您可以开始创建自己的演示控制器。

## 实施转型代表
由于委托的任务是管理执行动画的动画制作对象，因此在编写委托代码之前，首先必须为动画制作者类创建存根。

从Xcode的主菜单中选择File \ New \ File ...并选择模板iOS \ Source \ Cocoa Touch Class。

将新类名设置为PopAnimator，确保选中Swift，并使其成为NSObject的子类。

打开PopAnimator.swift并更新类定义，使其符合UIViewControllerAnimatedTransitioning协议，如下所示：

```swift
class PopAnimator: NSObject, UIViewControllerAnimatedTransitioning {
}
```

你会看到来自Xcode的一些抱怨，因为你还没有实现所需的委托方法，所以你接下来会把它们存在。

将以下方法添加到类中：

```swift
func transitionDuration(using transitionContext:
UIViewControllerContextTransitioning?) -> TimeInterval {
return 0 }
```

上面的0值只是持续时间的占位符值; 在您完成项目时，您将在以后用实际值替换它。

现在将以下方法存根添加到类中：

```swift
func animateTransition(using transitionContext:
UIViewControllerContextTransitioning) {
}
```

上面的存根将保存你的动画代码; 添加它应该已经清除了Xcode中的剩余错误。

现在您已经拥有了基本的动画师类，您可以继续在视图控制器端实现委托方法。 打开ViewController.swift并将以下扩展名添加到文件末尾：

```swift
extension ViewController: UIViewControllerTransitioningDelegate {
}
```

此代码表示视图控制器符合转换委托协议。 你马上就会在这里添加一些方法。

在类的主体中找到didTapImageView（_ :)。 在该方法的底部附近，您将看到显示详细信息视图控制器的代码。 herbDetails是新视图控制器的实例; 您需要将其转换委托设置为主控制器。 在调用present（...）的行之前添加以下代码以显示详细信息视图控制器：

```swift
// ...
herbDetails.transitioningDelegate = self // <-- Add this line
present(herbDetails, animated: true, completion: nil)
```

现在每次在屏幕上显示详细信息视图控制器时，UIKit都会向ViewController询问动画对象。 但是，您仍然没有实现任何UIViewControllerTransitioningDelegate方法，因此UIKit仍将使用默认转换。

下一步是实际创建动画对象并在请求时将其返回给UIKit。 将以下新属性添加到ViewController：

```swift
let transition = PopAnimator()
```

这是PopAnimator的实例，它将驱动您的动画视图控制器转换。

您只需要一个PopAnimator实例，因为每次呈现视图控制器时都可以使用相同的对象，因为每次转换都是相同的。

现在将第一个委托方法添加到ViewController中的扩展：

```swift
func animationController(forPresented presented: UIViewController,
presenting: UIViewController, source: UIViewController) ->
UIViewControllerAnimatedTransitioning? {
  return transition
}
```

此方法采用一些参数，使您可以决定是否要返回自定义动画。 在本章中，您将始终返回单个PopAnimator实例，因为您只有一个演示文稿转换。

您已经添加了用于呈现视图控制器的委托方法，但是您将如何处理解雇视图控制器？

添加以下委托方法来处理此问题：

```swift
func animationController(forDismissed dismissed: UIViewController) ->
UIViewControllerAnimatedTransitioning? {
return nil
}
```

上面的方法与前一个方法基本相同：您检查哪个视图控制器被解除并决定是否返回nil并使用默认动画，或者返回自定义过渡动画师并使用它。 在你返回nil的那一刻，因为你不会在以后实施解雇动画。

你最终有一个自定义动画师来处理你的自定义过渡。 但它有效吗？

构建并运行您的项目并点击其中一个药草图像：



[图片]



什么都没发生。 为什么？ 你有一个自定义动画师来推动过渡，但是......哦，等等，你没有向动画师类添加任何代码！ 你将在下一节中处理这个问题。

## 创建过渡动画师
打开PopAnimator.swift; 这是您将添加代码以在两个视图控制器之间进行转换的位置。

首先，将以下属性添加到此类：

```swift
let duration = 1.0
var presenting = true
var originFrame = CGRect.zero
```

您将在多个位置使用持续时间，例如当您告诉UIKit过渡将花费多长时间以及何时创建组成动画时。

您还可以定义演示，告诉动画师课程您是在演示还是解雇视图控制器。 你想要跟踪这一点，因为通常情况下，你会将动画向前运行呈现，反向运行以消除。

最后，您将使用originFrame存储用户点击的图像的原始帧矩形 - 您将需要它从原始帧动画到全屏图像，反之亦然。 稍后当您获取当前选定的图像并将其帧传递给动画制作实例时，请密切关注originFrame。

现在，您可以继续使用UIViewControllerAnimatedTransitioning方法。

用以下内容替换transitionDuration（）中的代码：

```swift
return duration
```

重复使用持续时间属性可以轻松地尝试过渡动画。 您可以简单地修改属性的值，以使转换运行更快或更慢。

### 设置转换的上下文
是时候为animateTransition添加一些魔法了。 此方法有一个类型为UIViewControllerContextTransitioning的参数，通过该参数可以访问转换的参数和视图控制器。 在开始处理代码本身之前，了解动画上下文实际上是什么很重要。

当两个视图控制器之间的转换开始时，现有视图将添加到转换容器视图中，并且新视图控制器的视图已创建但尚未可见，如下所示：



[图片]



因此，您的任务是将新视图添加到animateTransition（）中的过渡容器中，“动画显示”其外观，并在需要时“动画化”旧视图。

默认情况下，转换动画完成后，旧视图将从转换容器中删除。



[图片]



在这个厨房里有太多的厨师之前，你会创建一个简单的过渡动画，看看它是如何工作的，然后再实现一个更酷，虽然更复杂的过渡。

### 添加淡入淡出过渡
您将从简单的淡入淡出过渡开始，以了解自定义过渡。 将以下代码添加到animateTransition（）：

```swift
let containerView = transitionContext.containerView
let toView = transitionContext.view(forKey: .to)!
```

首先，您将获得动画将在其中进行的容器视图，然后获取新视图并将其存储在toView中。

转换上下文对象有两个非常方便的方法，可以让您访问转换播放器：

* view（forKey :)：这使您可以分别通过参数UITransitionContextViewKey.from或UITransitionContextViewKey.to访问“旧”和“新”视图控制器的视图。

* viewController（forKey :)：这使您可以分别通过参数UITransitionContextViewControllerKey.from或UITransitionContextViewControllerKey.to访问“旧”和“新”视图控制器。

此时，您同时拥有容器视图和要显示的视图。 接下来，您需要将要作为子项呈现的视图添加到容器视图中，并以某种方式为其设置动画。

将以下内容添加到animateTransition（）：

```swift
containerView.addSubview(toView)
toView.alpha = 0.0
UIView.animate(withDuration: duration,
  animations: {
    toView.alpha = 1.0
  },
  completion: { _ in
    transitionContext.completeTransition(true)
  }
)
```

请注意，您在动画完成块的转换上下文中调用completeTransition（）; 这告诉UIKit您的过渡动画已经完成，并且UIKit可以自由地结束视图控制器转换。

构建并运行您的项目; 点击列表中的一种草药，你会看到草药概述在主视图控制器上淡入：



[图片]



过渡是可以接受的，你已经看到了在animateTransition中做了什么 - 但你要添加更好的东西！

### 添加弹出过渡
您将以稍微不同的方式构造新转换的代码，因此使用以下内容替换animateTransition（）中的所有代码：

```swift
let containerView = transitionContext.containerView
let toView = transitionContext.view(forKey: .to)!
let herbView = presenting ? toView :
  transitionContext.view(forKey: .from)!
```

containerView是您的动画所在的位置，而toView是要呈现的新视图。 如果您要呈现，herbView只是toView; 否则它将从上下文中获取。 对于呈现和解雇，herbView将始终是您动画的视图。 当您显示详细信息控制器视图时，它将逐渐占用整个屏幕。 当被解雇时，它将缩小到图像的原始帧。

将以下内容添加到animateTransition（）：

```swift
let initialFrame = presenting ? originFrame : herbView.frame
let finalFrame = presenting ? herbView.frame : originFrame

let xScaleFactor = presenting ?
  initialFrame.width / finalFrame.width :
  finalFrame.width / initialFrame.width

let yScaleFactor = presenting ?
  initialFrame.height / finalFrame.height :
  finalFrame.height / initialFrame.height
```

在上面的代码中，您可以检测初始和最终动画帧，然后计算在每个视图之间设置动画时需要在每个轴上应用的比例因子。

现在您需要仔细定位新视图，使其显示在拍摄图像的正上方; 这将使得看起来像点击的图像扩展以填充屏幕。

将以下内容添加到animateTransition（）：

```swift
let scaleTransform = CGAffineTransform(scaleX: xScaleFactor,
                                            y: yScaleFactor)
if presenting {
  herbView.transform = scaleTransform
  herbView.center = CGPoint(
    x: initialFrame.midX,
    y: initialFrame.midY)
  herbView.clipsToBounds = true
}
```

在显示新视图时，您可以设置其比例和位置，使其与初始帧的大小和位置完全匹配。

现在将最后的代码添加到animateTransition（）：

```swift
containerView.addSubview(toView)
containerView.bringSubviewToFront(herbView)
UIView.animate(withDuration: duration, delay:0.0,
  usingSpringWithDamping: 0.4, initialSpringVelocity: 0.0,
  animations: {
    herbView.transform = self.presenting ?
      CGAffineTransform.identity : scaleTransform
    herbView.center = CGPoint(x: finalFrame.midX, y: finalFrame.midY)
  },
  completion: { _ in
		transitionContext.completeTransition(true)
  }
)
```

这将首先将toView添加到容器中。接下来，您需要确保herbView位于顶部，因为这是您动画的唯一视图。请记住，在解散时，toView是原始视图，因此在第一行中，您将在其他所有内容之外添加它，除非您将herbView带到前面，否则您的动画将被隐藏起来。

然后，您可以启动动画。在这里使用弹簧动画会给它一些反弹。

在动画表达式中，您可以更改herbView的变换和位置。在呈现时，您将从底部的小尺寸变为全屏，因此目标变换只是身份变换。在解除时，您可以对其进行动画处理以缩小以匹配原始图像大小。

此时，您已经通过将新视图控制器放置在点击图像上来设置舞台，您在初始帧和最终帧之间进行了动画处理，最后，您调用了completeTransition（）将事物交还给UIKit。现在是时候看到你的代码了！

构建并运行您的项目;点击第一个草本图像，以查看您的视图控制器转换。



[图片]



嗯，它并不完美，但是一旦你处理了一些粗糙的边缘，你的动画将正是你想要的！

目前你的动画从左上角开始; 那是因为originFrame的默认值的原点是（0,0） - 你永远不会将它设置为任何其他值。

打开ViewController.swift并将以下代码添加到animationController的顶部（forPresented :)：

```swift
transition.originFrame =
selectedImage!.superview!.convert(selectedImage!.frame, to: nil)
transition.presenting = true
selectedImage!.isHidden = true
```

这会将转换的originFrame设置为selectedImage的帧，这是您最后一次点击的图像视图。 然后将呈现设置为true并在动画期间隐藏拍摄的图像。

再次构建和运行您的项目; 在列表中点击不同的草药，看看你的过渡如何看待每个。



[图片]



### 添加解雇转换
剩下要做的就是解雇细节控制器。 你实际上已经完成了动画师的大部分工作 - 转换动画代码执行逻辑杂耍设置正确的初始和最终帧，所以你大部分都是向前和向后播放动画。 甜！

打开ViewController.swift并用以下内容替换animationController（forDismissed :)的主体：

```swift
transition.presenting = false
return transition
```

这告诉您的动画制作者对象您正在关闭视图控制器，以便动画代码以正确的方向运行。

构建并运行项目以查看结果。 点击草药，然后点击屏幕上的任意位置以解除它。



[图片]



过渡动画看起来很棒，但请注意您选择的草药已从滚动视图中消失！ 当您关闭详细信息屏幕时，您需要确保重新显示已点击的图像。

打开PopAnimator.swift并向该类添加一个新的闭包属性：

```swift
var dismissCompletion: (()->Void)?
```

这将允许您传递一些代码，以便在解除转换完成时运行。

接下来，在调用completeTransition（）之前，找到animateTransition（）并将以下代码添加到对animateWithDuration（...）的调用中的完成处理程序中：

```swift
if !self.presenting {
  self.dismissCompletion?()
}
```

一旦解雇动画完成，此代码将执行dismissCompletion  - 这是显示原始图像的最佳位置。

打开ViewController.swift并将以下代码添加到viewDidLoad（）：

```swift
transition.dismissCompletion = {
  self.selectedImage!.isHidden = false
}
```

转换动画完成后，此代码显示原始图像以替换药草详细信息视图控制器。

构建并运行您的应用程序，以两种方式享受过渡动画，现在草药不会在途中迷路！



[图片]



过渡动画在边缘看起来仍然有点粗糙，但是你将在本章的挑战中自己完成它。

在你遇到挑战之前，还有一个动画话题要处理：查看用于管理设备方向更改的转换。

## 设备方向转换

> 注意：本章的这一部分是可选的; 如果您对学习如何在视图控制器中处理设备方向的变化不感兴趣，请直接跳到挑战。

您可以将设备方向更改视为从视图控制器到其自身的演示过渡，只是大小不同。

iOS 8中引入的viewWillTransition（to size：coordinator :)为您提供了一种简单直接的方法来处理设备方向的变化。 您不需要构建单独的纵向或横向布局; 相反，您只需要对视图控制器视图的大小进行更改。

打开ViewController.swift并为viewWillTransition添加以下存根（to：with :)：

```swift
override func viewWillTransition(to size: CGSize, with coordinator:
UIViewControllerTransitionCoordinator) {
  super.viewWillTransition(to: size, with: coordinator)
}
```

第一个参数（大小）告诉您视图控制器转换到的大小。 第二个参数（协调器）是转换协调器对象，它允许您访问许多转换的属性。

您需要在此应用程序中执行的操作是减少应用程序背景图像的alpha值，以便在设备处于横向模式时提高文本的可读性。

将以下代码添加到viewWillTransitionToSize：

```swift
coordinator.animate(
  alongsideTransition: { context in
    self.bgImage.alpha = (size.width>size.height) ? 0.25 : 0.55
  },
  completion: nil
)
```

animate（withsideTransition :)允许您指定自己的自定义动画，以便在更改方向时与UIKit默认执行的旋转动画并行执行。

您的动画关闭将接收转换上下文，就像您在呈现视图控制器时使用的那样。在这种情况下，您没有“从”和“到”视图控制器，因为它们是相同的，但您可以获取诸如转换持续时间之类的属性。

在动画闭包中，检查目标尺寸的宽度是否大于高度;如果是这样，您将背景图像的alpha值减小到0.25。这使得背景在转换为横向模式时淡出，在转换为纵向时淡入0.55 alpha。

构建并运行您的应用程序;旋转设备（如果在iPhone模拟器中测试，则按Cmd +向左箭头）以查看正在运行的Alpha动画。

将屏幕旋转到横向模式时，可以清楚地看到背景暗淡。这使得更长的文本行更容易阅读。

但是，如果你点击草药图像，你会发现动画有些混乱。

发生这种情况是因为屏幕具有横向方向，但图像仍具有纵向尺寸。因此，原始图像和拉伸以填满屏幕的图像之间的过渡不是流动的。

不要害怕 - 你可以使用你的新朋友viewWillTransition（to：with :)来解决这个问题。

ViewController上有一个名为positionListItems（）的实例方法，用于调整和定位草本图像。当您的应用首次启动时，将从viewDidLoad（）调用此方法。

在您设置alpha之后，在最后添加的animate（alongsideTransition :)动画块中添加对此方法的调用：

```swift
self.positionListItems()
```

这将在设备旋转时设置草药图像的大小和位置的动画。 一旦屏幕重新定向，草药图像也将调整大小：



[图片]



由于这些图像现在将具有横向布局，因此您的过渡动画也可以正常工作。 试试看！



[图片]



这就是本章的内容; 看看下面的挑战，你将在其中修饰过渡动画的一些剩余粗糙边缘。

## 关键点
* 通过在其中一种类型中实现UIViewControllerTransitioningDelegate协议并使其成为演示文稿委托来启用和配置自定义演示文稿转换。
* 从执行UIViewControllerAnimatedTransitioning协议的转换动画器类执行自定义转换。
* 要创建任何过渡动画，您可以使用本书前几章中已经介绍过的任何标准动画技术。

## 挑战
您的演示动画中有两个微小的缺陷：您可以看到详细视图文本，直到它消失的最后一刻; 此外，最初的药草图像有圆角，这使得动画最终看起来很犀利。

### 挑战1：平滑过渡动画
您的第一个任务是在转换时适当地淡入或淡出草药详细信息视图的内容。 这纠正了细节视图的文本在被解除时消失的尴尬时刻。

看一下storyboard文件; 您将看到详细信息控制器已将其所有文本视图添加到连接到containerView插座的主视图中。 您需要做的就是在进行过渡动画时淡入或淡出containerView。

解决这一挑战的方法有三个步骤：

* 使用转换上下文的viewController（forKey :)方法获取对HerbDetailsViewController的引用。 请记住，密钥会有所不同，具体取决于您是呈现还是解雇，并且您需要将结果转换为HerbViewController。 可供选择的两个键是UITransitionContextViewControllerKey.to和UITransitionContextViewControllerKey.from。
* 在开始动画（已经在animateTranstion中的一个动画）之前，将containerView的alpha设置为零; 它是HerbDetailsViewController的出口属性。 您只需要在呈现视图控制器时执行此操作，而不是在解除视图控制器时执行此操作，因为它在解除时已经可见。
* 在动画闭包中设置containerView的alpha  - 你在其中设置herbView的动画 - 在你呈现时完全不透明（1.0）并在你解散时完全透明（0.0）。

要更仔细地检查更改结果，可以将转换持续时间增加到10秒并详细观察动画（或者从iOS模拟器主菜单中使用“调试/切换最前面应用程序中的慢动画”）。

### 挑战2：为角半径设置动画
最后，您将为细节视图的角半径设置动画，使其与主视图控制器中草本图像的圆角相匹配。

在animateTransition（）结束时，创建并运行图层动画以更改herbView图层的角半径。将herbView.layer.cornerRadius从20.0 / xScaleFactor设置为0.0（如果呈现），反之亦然（如果解除）。您需要考虑比例因子，因为您还在转换视图。将动画的持续时间设置为持续时间/ 2，以便在弹跳开始之前完成，否则看起来会很奇怪！

这是你可能想要使用的一个小技巧。 cornerRadius是UIKit动画API可以设置动画的少数特殊图层属性之一。您可以在动画块中更改角落半径，它只是工作 - 尝试一下！

这包含了演示控制器动画。接下来，导航控制器动画。你会发现两者之间有很多相似之处，所以把草药放在一边然后潜入！