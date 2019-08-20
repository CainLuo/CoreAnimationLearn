# Chapter 20: Getting Started with UIViewPropertyAnimator

UIViewPropertyAnimator是在iOS 10中引入的，它解决了能够创建易于交互，可中断和/或可逆的视图动画的需求。

在iOS 10之前，创建基于视图的动画的唯一选择是UIView.animate（withDuration：...）一组API，它们没有为开发人员暂停或停止已经运行的动画提供任何手段。此外，为了反转，加速或减慢动画，开发人员必须使用基于图层的CAAnimation动画。

UIViewPropertyAnimator使得创建上述所有内容变得更加容易，因为它是一个允许您保持运行动画的类，允许您调整当前运行的动画，并为您提供有关动画当前状态的详细信息。

UIViewPropertyAnimator距离iOS 10之前的“即发即忘”动画还有很大的进步。话虽这么说，UIView.animate（withDuration：...）API仍然在创建iOS动画方面发挥了重要作用;这些API简单易用，通常你只想开始一个简短的淡出或简单的移动，你真的不需要中断或反转它们。在这些情况下使用UIView.animate（withDuration：...）就好了。

请记住，UIViewPropertyAnimator并未实现UIView.animate（withDuration：...）必须提供的所有内容，因此有时您仍需要依赖旧的API。

## 基本动画
打开并运行本章的入门项目。 您应该会看到类似于iOS中的锁定屏幕的屏幕。 初始视图控制器在底部显示搜索栏，单个窗口小部件和编辑按钮：



[图片]



已经为您实现了一些与动画无关的应用程序功能。 例如，如果点击“显示更多”，您将看到窗口小部件展开并显示更多项目。 如果点击编辑，您将看到另一个视图控制器，用于编辑窗口小部件列表。

当然，该应用程序只是模拟iOS中的锁定屏幕。 它实际上并不执行任何操作，并且专门为您设置了一些UIViewPropertyAnimator动画。 我们走吧！

首先，您将在应用程序最初打开时创建一个非常简单的动画。 打开LockScreenViewController.swift并向该视图控制器添加一个新的viewWillAppear（_ :)方法：

```swift
override func viewWillAppear(_ animated: Bool) {
  tableView.transform = CGAffineTransform(scaleX: 0.67, y: 0.67)
  tableView.alpha = 0
}
```

LockScreenViewController中的表视图在该屏幕上显示小部件，因此，为了创建简单的缩放和淡入淡出视图动画过渡，首先缩小整个表视图并使其透明。

如果您现在运行该项目，您将看到日期和下面的一些空白区域：



[图片]



接下来，在视图控制器的视图出现在屏幕上时创建动画制作工具。 将以下内容添加到LockScreenViewController：

```swift
override func viewDidAppear(_ animated: Bool) {
  let scale = UIViewPropertyAnimator(duration: 0.33,
    curve: .easeIn)
}
```

在这里，您使用UIViewPropertyAnimator的一个便利初始化器。 你将尝试所有这些，但你将从最简单的一个开始：UIViewPropertyAnimator（持续时间：，曲线:)。

此初始化程序生成动画实例并设置动画的总持续时间和时间曲线。 后一个参数的类型为UIViewAnimationCurve，这是一个带有以下基于曲线的选项的枚举：

* easeInOut
* easeIn
* easeOut
* linear

这些与您使用的计时选项相匹配UIView.animate（withDuration：...）API，它们产生类似的结果。

既然你已经创建了一个动画师对象，那么让我们来看看你能用它做些什么。

### 添加动画
将动画代码添加到viewDidAppear（_ :)：

```swift
scale.addAnimations {
  self.tableView.alpha = 1.0
}
```

您可以使用addAnimations添加代码块，这些代码块执行所需的动画，就像使用UIView.animate（withDuration：...）一样。 使用动画师时的不同之处在于您可以添加多个动画块。 例如，您可以包含逻辑以有条件地向同一个动画师添加更多或更少的动画。

除了能够有条件地构建复杂的动画外，您还可以添加具有不同延迟的动画。 有一个addAnimations版本，它采用以下两个参数：

* animation，是要执行动画的块，
* 和delayFactor，这是动画开始前的延迟。

请注意，后一个参数不称为延迟，具体为：delayFactor。 这是因为您不提供以秒为单位的绝对值，而是提供动画师剩余持续时间的因子（介于0.0和1.0之间）。

向同一个动画师添加第二个动画，但有一些延迟：

```swift
scale.addAnimations({
  self.tableView.transform = .identity
}, delayFactor: 0.33)
```

要计算出以秒为单位的实际延迟，请使用delayFactor并将其乘以动画师的剩余持续时间。 由于您尚未启动动画，因此剩余持续时间等于总持续时间。

所以在上面的情况：

```swift
delayFactor(0.33) * remainingDuration(=duration 0.33) = delay of 0.11
seconds
```

为什么第二个参数不是几秒钟内的简单值？

好吧，想象你的动画师已经在运行了，你决定在中途添加一些新的动画。 在这种情况下，剩余持续时间将不等于总持续时间，因为自启动动画以来已经过了一段时间。



[图片]



在这种情况下，delayFactor将允许您根据剩余可用时间安排延迟动画。 此外，这可确保您无法将延迟设置为超过剩余运行时间。



[图片]



### 添加完成
现在添加一个完成块，就像你习惯使用UIView.animate（withDuration：...）一样：

```swift
scale.addCompletion { _ in
  print("ready")
}
```

在这个简单的例子中，你只是打印到控制台，但你可以做任何你想要的事情; 清理一些临时视图，重置移动的视觉元素的位置，等等。

与addAnimations（_ :)一样，您可以多次调用addCompletion（_ :)来添加更多完成处理程序。 这些将一个接一个地执行，并按照您将它们添加到动画师的顺序执行。

最后但并非最不重要的，你需要启动动画师。

在调用startAnimations（）之前，屏幕上不会发生任何事情，因此在习惯使用UIViewPropertyAnimator时请记住，如果您在屏幕上看不到动画，则可能忘记启动它们。

在viewWillAppear（_ :)的末尾添加：

```swift
scale.startAnimation()
```

现在运行项目，并在屏幕上弹出应用程序时享受平滑过渡动画：



[图片]



## 摘录动画
您可能已经注意到，就像图层动画一样，使用UIViewPropertyAnimator的动画添加了相当多的代码。

使用不是“一劳永逸”的对象，可以很容易地将一些动画代码提取到一个单独的类中。 由于您将在本书的这一部分为项目创建大量动画，因此您将在单独的文件中提取大部分动画。

创建一个名为AnimatorFactory.swift的新文件，并将其默认内容替换为：

```swift
import UIKit
class AnimatorFactory {
}
```

然后添加一个方法，其中包含您刚刚编写的动画代码，但默认情况下不运行动画，而是返回动画师作为结果：

```swift
static func scaleUp(view: UIView) -> UIViewPropertyAnimator {
  let scale = UIViewPropertyAnimator(duration: 0.33,
    curve: .easeIn)
  scale.addAnimations {
    view.alpha = 1.0
  }
  scale.addAnimations({
    view.transform = CGAffineTransform.identity
  }, delayFactor: 0.33)
  scale.addCompletion {_ in
    print("ready")
  }
  return scale
}
```

该方法将视图作为其参数，并在该视图上创建所有动画。 最后它返回准备好的动画师。

切换到LockScreenViewController.swift并将viewDidAppear（_ :)替换为：

```swift
override func viewDidAppear(_ animated: Bool) {
  AnimatorFactory.scaleUp(view: tableView)
    .startAnimation()
}
```

那更好，更短，更清洁！

到本节结束时，您将非常欣赏AnimatorFactory，因为它将从您的视图控制器中删除大量代码。

> 注意：在您自己的项目中，您可能希望使用枚举或结构来访问抽象动画师。 在本书中，您将使用静态类方法。

## 运行动画师
此时你可能会问自己“创建一个动画师对象有什么意义，如果它的唯一目的是立即开始？”

这是个好问题！

如果您需要运行单个动画块并且不再需要更改，请继续使用UIView.animate（withDuration：...）。您决定使用哪个API的转折点取决于您是想简单地运行动画 - 还是运行动画并最终与之交互。

如果你想使用UIViewPropertyAnimator但你仍然只有一个动画和完成块，并想立即运行它会怎么样？是否有更简化的方式来创建这样的动画？

为什么，是的，有！我很高兴你问。这就是本章的这一部分被称为运行动画师的原因。在UIViewPropertyAnimator上有一个类方法，它创建一个动画师并立即为你启动它。

接下来，当用户使用搜索栏时，您将淡入模糊图层（blurView），并在用户完成搜索时将其淡出。

打开LockScreenViewController.swift并向LockScreenViewController类添加一个新方法：

```swift
func toggleBlur(_ blurred: Bool) {
  UIViewPropertyAnimator.runningPropertyAnimator(
    withDuration: 0.5, delay: 0.1, options: .curveEaseOut,
    animations: {
      self.blurView.alpha = blurred ? 1 : 0
    },
    completion: nil
  )
}
```

在toggleBlur（_ :)中，您使用UIViewPropertyAnimator.runningPropertyAnimator（withDuration：delay：options：animations：completion）来创建已在运行的动画师。

你肯定注意到了UIViewPropertyAnimator。 runningPropertyAnimator（withDuration：...）采用与UIView.animate（withDuration：...）完全相同的参数，以便您更轻松地使用此新API。

尽管看起来这可能是一种“即发即忘”的API，但请注意它确实会返回一个动画实例。 您可以添加更多动画，更多完成块，并且通常与当前正在运行的动画进行交互。

现在让我们看看淡入淡出动画的样子。 LockScreenViewController已设置为搜索栏的委托，因此您只需实现所需的方法即可在正确的时间触发动画。

添加一个新的LockScreenViewController扩展：

```swift
extension LockScreenViewController: UISearchBarDelegate {
  func searchBarTextDidBeginEditing(_ searchBar: UISearchBar) {
    toggleBlur(true)
  }
  func searchBarTextDidEndEditing(_ searchBar: UISearchBar) {
    toggleBlur(false)
  }
}
```

当用户点击搜索字段时，您会淡化模糊，并在用户使用完搜索栏时将其淡出。 要为用户提供更多取消搜索的方法，还要添加以下两种方法：

```swift
func searchBarResultsListButtonClicked(_ searchBar: UISearchBar) {
  searchBar.resignFirstResponder()
}
func searchBar(_ searchBar: UISearchBar, textDidChange searchText:
String) {
  if searchText.isEmpty {
    searchBar.resignFirstResponder()
  }
}
```

这将允许用户通过点击右侧按钮或删除其搜索查询来解除搜索。 立即运行该应用，然后点按搜索栏文本字段。



[图片]



你会看到小部件在模糊效果视图下消失。 当您点击搜索栏右侧的按钮时，模糊视图会淡出。

## 基本关键帧动画
之前您学习了如何向同一个动画师添加更多动画块，并让它们以给定的延迟因子开始。 这很方便，但它与您习惯使用视图关键帧动画的方式完全不同。

UIView.animateKeyframes API非常强大，因为它允许您以任何方式对动画进行分组，具有任何延迟和持续时间。

准备一些好消息！

实际上，您可以在UIViewPropertyAnimator动画块中使用UIView.animate和UIView.animateKeyframes。

因此，如果你想创建一个复杂的关键帧动画但仍然意识到创建动画师的好处，比如可以暂停或反转，你可以！

在本章的这一部分中，您将创建一个简单的抖动关键帧动画。 您将在用户点击的任何图标上播放该动画，以便为他们提供一些视觉点击反馈：



[图片]



切换到AnimatorFactory.swift并添加一个新方法：

```swift
static func jiggle(view: UIView) -> UIViewPropertyAnimator {
  return UIViewPropertyAnimator.runningPropertyAnimator(
    withDuration: 0.33, delay: 0, animations: {
    },
    completion: {_ in
    }
	) 
}
```

你的微动画动画将运行0.33秒，但仍然没有做太多。 在动画块中添加以下内容：

```swift
UIView.animateKeyframes(withDuration: 1, delay: 0,
  animations: {
    UIView.addKeyframe(withRelativeStartTime: 0.0,
      relativeDuration: 0.25) {
      view.transform = CGAffineTransform(rotationAngle: -.pi/8)
    }
    UIView.addKeyframe(withRelativeStartTime: 0.25,
      relativeDuration: 0.75) {
      view.transform = CGAffineTransform(rotationAngle: +.pi/8)
    }
    UIView.addKeyframe(withRelativeStartTime: 0.75,
      relativeDuration: 1.0) {
      view.transform = CGAffineTransform.identity
    }
},
  completion: nil
)
```

此代码定义了一个视图关键帧动画，就像您在完成第5章“关键帧动画”时创建的那样。

第一个关键帧将给定视图向左旋转，第二个关键帧将其向右旋转，最后第三个关键帧将其带回原位 - 呃，我的意思是重置它的变换。

要确保图标保持在其初始位置，即使动画被中断，请在完成块中添加：

```swift
view.transform = .identity
```

没有明显的方法来中断动画，但由于您使用的是动画师，因此总是可以添加代码以暂停或停止该特定的动画师。

现在，与UIView.animate（withDuration：...）系列API相比，您对动画的看法有很大差异。 使用动画师时，您的动画总是可以最终成功完成或在中途停止，或者甚至不在最终状态完成，但如果在执行期间被反转则处于启动状态。

接下来，由于动画已完成，您可以使用jiggle（view :)方法获取关键帧动画师并在某些视图上运行它。

打开IconCell.swift（该文件位于Widget子文件夹中）。 这是自定义集合单元类，它在窗口小部件视图中显示每个图标：



[图片]



每当选择其中一个单元格时，您将在其图像上运行抖动动画，为用户提供一些触摸反馈。

在单元格上添加一个新的便捷方法，以在其图像上启动动画师：

```swift
func iconJiggle() {
  AnimatorFactory.jiggle(view: icon)
}
```

现在Xcode抱怨您的AnimatorFactory.jiggle方法返回结果，但您不以任何方式使用它。 幸运的是，这是一个容易解决的问题。

切换到AnimatorFactory.swift并将以下内容添加到静态func jiggle之前的行（view：UIView） - > UIViewPropertyAnimator一个@discardableResult属性，以便Xcode知道您可以选择忽略该方法的结果：

```swift
@discardableResult
static func jiggle(view: UIView) -> UIViewPropertyAnimator
```

不要完全删除返回类型 - 稍后您将使用该方法的结果。

要最终运行动画，请打开WidgetView.swift并找到collectionView（collectionView：didSelectItemAt :)。 这是当用户点击集合视图单元格时调用的集合视图委托方法。

附加以下内容：

```swift
if let cell = collectionView.cellForItem(at: indexPath) as? IconCell {
  cell.iconJiggle()
}
```

再次运行应用程序并尝试点击一些图标; 你很快就会看到它们在你的手指下跳：



[图片]



> 注意：如果您碰巧想知道，目前还没有简化的方法来重复动画动画。 如果你想要一个重复的动画，你将需要观察动画师的运行属性，一旦动画完成，将动画师的进度重置为0％并重新开始。 或者只使用UIView.animate（withDuration：animations）方法。

通过这个动画，你已经完成了基础之旅。 希望您已经了解了使用UIViewPropertyAnimator而不是旧API的一些好处。

然而，你所涵盖的只是UIViewPropertyAnimator可以做的一小部分。 在接下来的章节中，您将研究更有趣的方法来设置动画的时序，交互性和电源视图控制器转换。

## 关键点
* 使用UIViewPropertyAnimator类型时，您可以创建一个对象，为其添加动画和/或完成闭包，并在方便时启动动画。
* 在类实例中包含动画闭包提供了一种通过动画工厂重用动画的全新方法。
* 要创建视图关键帧动画，您仍需要使用旧的api UIView.animateKeyframes（withDuration：delay：keyframes :)。

## 挑战
您已经了解了有关使用UIViewPropertyAnimator的一些基础知识，但在接下来的三章中还有更多需要学习的内容。 在本章的挑战部分，花点时间思考一下你所学到的知识，并体验你第一次遇到属性动画师的状态。

### 挑战1：将模糊动画提取到工厂中
要再次练习抽象动画，请将toggleBlur（_ :)中的模糊动画提取到AnimatorFactory上的静态方法中。

这次，静态工厂方法应该采用两个参数：动画视图以及是否为完全透明或完全不透明状态设置动画。

最后，您应该能够通过使用这个单行程序轻松切换blurView的可见性：

```swift
func toggleBlur(_ blurred: Bool) {
  AnimatorFactory.fade(view: blurView, visible: blurred)
}
```

您是否意识到使用UIViewPropertyAnimator抽象和重复使用动画是多么容易？ 我知道我当然会这样做！

### 挑战2：防止重叠动画
在此挑战中，您将学习如何检查动画师当前是否正在执行其动画。

如果您在同一个图标上反复点击，您将看到它在每次点击时跳回到其初始状态，并且动画看起来不稳定。

现在，你只是忽略了AnimatorFactory.jiggle的结果 - 但如果你没有呢？如果您实际掌握了动画对象并使用它来检查是否存在当前活动的抖动动画，则可以防止在该相同图标上进一步点击。

首先，将一个名为animator的可选属性添加到IconCell类中。
接下来，不是丢弃AnimatorFactory.jiggle的结果，而是将其存储在动画师中。现在，每次用户点击图标时，您都可以检查图标上是否已有动画。

在iconJiggle（）的开头检查是否设置了animator，如果是，请检查其isRunning属性是否为true。 isRunning告诉你动画师当前是否正在运行它的动画 - 即它已经启动但尚未完成。

如果有一个正在运行的动画师，剩下要做的就是退出iconJiggle（）而不创建新的动画。这将解决您的问题，用户可以根据需要在图标上点击多次。

接下来 - 使用UIViewPropertyAnimator制作更复杂的动画！

