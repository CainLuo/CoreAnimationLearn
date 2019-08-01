# Chapter 3: Transitions

转换是可以应用于视图的预定义动画。这些预定义的动画不会试图在视图的开始和结束状态之间插入(就像您在前两章中创建的动画一样)。相反，您将使用转换API来设计动画，以便UI中的各种更改看起来很自然。

## 转换示例
为了更好地理解什么时候使用转换，本节将介绍各种可以使用转换动画的动画场景。

### 添加新视图



[图片]



要使屏幕上添加的新视图具有动画效果，可以调用与前几章中使用的方法类似的方法。这次的不同之处在于，您将选择预定义的转换效果之一，并对动画容器视图进行动画处理。

转换将使容器视图动画化，在动画运行时，作为子视图添加到其中的任何新视图都会出现。

为了更好地解释如何动画容器视图，以及何时执行子视图之间的转换，请阅读下面的代码片段(您不需要在任何地方键入此代码):

```swift
var animationContainerView: UIView!
override func viewDidLoad() {
  super.viewDidLoad()
  //set up the animation container
  animationContainerView = UIView(frame: view.bounds)
  animationContainerView.frame = view.bounds
  view.addSubview(animationContainerView)
}
override func viewDidAppear(_ animated: Bool) {
  super.viewDidAppear(animated)
  //create new view
  let newView = UIImageView(image: UIImage(named: "banner"))
  newView.center = animationContainerView.center
  //add the new view via transition
  UIView.transition(with: animationContainerView,
    duration: 0.33,
    options: [.curveEaseOut, .transitionFlipFromBottom],
    animations: {
      self.animationContainerView.addSubview(newView)
    },
    completion: nil
  )
}
```

在这种假设的情况下，您将在视图控制器的viewDidLoad()中创建一个名为animationContainerView的新视图。然后定位该容器并将其添加到视图中。

稍后，当你想创建一个动画过渡时，你创建一个新的视图来动画;这里它被命名为newView。

要创建转换，可以调用转换(使用:duration: options:animation:completion:)。这几乎与标准的UIView动画方法相同，但在本例中，您提供了一个额外的参数视图，作为转换动画的容器视图。

这里有一个新的动画选项。transitionflipfrombottom，你还没有看到。这是本章介绍中讨论的预定义转换之一。transitionflipfrombottom翻转视图，视图的底部边缘作为视图翻转的“铰链”。

最后，将子视图添加到动画块中的动画容器中，这将导致在转换期间出现子视图。
预定义的过渡动画选项的完整列表如下:

* .transitionFlipFromLeft

* .transitionFlipFromRight

* .transitionCurlUp 
* .transitionCurlDown
* .transitionCrossDissolve
* .transitionFlipFromTop
* .transitionFlipFromBottom 

### 删除一个视图



[图片]



使用转换动画从屏幕上删除子视图的工作原理与添加子视图非常相似。要使用一个过渡动画来实现这一点，你只需在动画闭包表达式中调用removeFromSuperview()，如下所示(同样，这是一个例子，你不需要输入这段代码):

```swift
//remove the view via transition
UIView.transition(with: animationContainerView, duration: 0.33,
  options: [.curveEaseOut, .transitionFlipFromBottom],
  animations: {
    someView.removeFromSuperview()
  },
  completion: nil
)
```

包装器转换将执行翻转动画，并在动画结束时消失一些视图。

### 隐藏/显示一个视图

到目前为止，在本章中，您只了解了更改视图层次结构的转换。这就是为什么您需要一个用于转换的容器视图—这将层次结构更改放在上下文中。

相反，您不需要担心设置一个容器视图来隐藏和显示视图。在本例中，转换使用视图本身作为动画容器。

考虑以下代码来隐藏使用转换的子视图:

```swift
//hide the view via transition
UIView.transition(with: someView, duration: 0.33,
  options: [.curveEaseOut, .transitionFlipFromBottom],
  animations: {
    someView.alpha = 0
  },
  completion: { _ in
    someView.isHidden = true
  }
)
```

在这里，您将传递您打算显示或隐藏的视图作为要转换的第一个参数(使用:duration:options:animation:completion:)。之后你要做的就是在动画块中设置视图的alpha属性，这样过渡动画就开始了。

### 用另一个视图替换视图



[图片]



用另一个视图替换一个视图也是一个简单的过程。您只需将现有视图作为第一个参数传入，并将toView: parameter设置为您希望替换它的视图，如下所示:

```swift
//replace via transition
UIView.transition(from: oldView, to: newView, duration: 0.33,
  options: .transitionFlipFromTop, completion: nil)
```

很明显UIKit为你做了多少繁重的工作!
在本章的其余部分中，您将通过转换来显示和隐藏UI元素，并学习一些可以引入到您自己的项目中的新的动画技能!

## 混合过渡
您将继续在本章的巴哈马航空登录屏幕项目;您已经为这个屏幕上的视图创建了许多引人注目的动画，以便为登录表单添加一些活力，并使按钮对点击作出反应。

接下来，您将模拟一些用户身份验证，并动画化几个不同的进度消息。一旦用户点击登录按钮，你将向他们显示包括“正在连接……””、“授权……”和“失败”。



[图片]



如果您还没有读完前几章，可以从本章参考资料文件夹中的starter项目开始。如果您已经在自己的项目中遵循了前几章中的示例，那么做得很好!您可以继续使用现有的项目。

打开ViewController.swift，查看viewDidLoad()。该方法的一部分添加了存储在类变量状态中的隐藏图像视图。然后代码创建一个文本标签，并将其作为子视图添加到status。

您将使用status向用户显示进度消息。消息来自消息数组，这是starter项目中包含的另一个类变量。

添加以下方法到ViewController:

```swift
func showMessage(index: Int) {
  label.text = messages[index]
  UIView.transition(with: status, duration: 0.33,
    options: [.curveEaseOut, .transitionCurlDown],
    animations: {
      self.status.isHidden = false
    },
    completion: {_ in
      //transition completion
		} 
	)
}
```

此方法接受一个名为index的参数，您可以使用该参数将label的值设置为基于索引的消息内容。

接下来调用transition(使用:duration:options:animation: completion:)使视图具有动画效果。在动画块中将isHidden设置为false，以便在转换时显示横幅。

上面还有一个新的动画选项:. transitioncurldown。这个转换使视图像一张纸一样被翻过来放在一个合法的pad上，看起来像这样:



[图片]



现在是时候练习新的showMessage(index:)方法了。在login()中找到以下代码块:

```swift
animations: {
  self.loginButton.bounds.size.width += 80.0
}, completion: nil)
```

这是您现有的动画块，当用户点击登录按钮时，它会弹出。您将向这个调用showMessage(index:)的动画添加一些新内容—一个完成闭包。

用下面的闭包表达式替换补全的nil参数值:

```swift
completion: { _ in
  self.showMessage(index: 0)
}
```

闭包接受一个Bool参数，该参数告诉您动画是成功完成还是在运行到完成之前被取消。

> 注意:上面的completion closure只有一个参数表示completed。因为您不关心它是否完成，Swift允许您通过在参数的位置放置“_”来跳过绑定。

在闭包中，只需调用索引为0的showMessage来显示消息数组中的第一个消息。

建立和运行您的项目;点击Log In按钮，您将看到状态横幅出现在您的第一个进度消息中。



[图片]



你看到横幅像一张纸一样卷了下来吗?这是一种很好的方式，可以让人们注意到通常只显示为静态文本标签的消息。

> 注意:您的一些动画似乎运行得非常快，不是吗?有时很难确保动画在正确的位置以正确的顺序发生。或者你只是想让事情发生得慢一些，这样你就能欣赏到效果!
>
> 要在不更改代码的情况下降低应用程序中的所有动画的速度，请从iPhone模拟器菜单中选择Debug/Toggle slow animation in Frontmost app。现在点击你的应用程序中的登录按钮，享受生动的慢动作动画和过渡!

对于下一个动画，首先需要保存横幅的初始位置，这样就可以将下一个横幅放在正确的位置。

在viewDidLoad()的末尾添加以下代码，将横幅的初始位置保存到名为statusPosition的属性中:

```swift
statusPosition = status.center
```

现在您可以开始设计视图动画和转换的混合。添加以下方法，通过标准动画从屏幕上删除状态信息:

```swift
func removeMessage(index: Int) {
  UIView.animate(withDuration: 0.33, delay: 0.0, options: [],
    animations: {
      self.status.center.x += self.view.frame.size.width
    },
    completion: { _ in
      self.status.isHidden = true
      self.status.center = self.statusPosition
      self.showMessage(index: index+1)
    }
	) 
}
```

在上面的代码中，您使用老朋友animate(带有duration:delay:options:animation:completion:)将状态移动到屏幕可见区域之外。

当动画在completionclosure中完成时，您将状态移回其原始位置并隐藏它。最后，再次调用showMessage，但这次传递的是要显示的下一条消息的索引。

将标准动画与转换相结合非常简单:您只需调用适当的API，而UIKit在后台调用相应的核心动画位。

现在，您需要完成showMessage和removeMessage之间的调用链，以模拟真实的身份验证过程。

找到showMessage(index:)并用以下代码替换注释//转换完成:

```swift
delay(2.0) {
  if index < self.messages.count-1 {
    self.removeMessage(index: index)
  } else {
//reset form
	} 
}
```

转换完成后，等待2.0秒并检查是否还有剩余消息。如果是，则通过removeMessage(index:)删除当前消息。然后在removeMessage(index:)的完成块中调用showMessage(index:)来按顺序显示下一条消息。

> 注意:delay(_:completion:)是一个方便的函数，它在经过延时后运行一段代码;它在viewcontroller。swift的顶部定义。这里您使用它来模拟通常的网络访问延迟。

再次构建和运行您的项目;享受由此产生的动画序列，其中更新的认证进程消息如下:



[图片]



转换是动画知识的一个小而重要的子集，要保存在你的具象工具箱中，因为它们是在UIKit中创建3d风格动画的唯一方法。

如果你想学习更精细的3D效果，你将有机会在第六部分“3D动画”中学习，它将详细讨论核心动画和3D层转换。

在进入下一节之前，请尝试一下本章中的挑战。既然你在前三章学了这么多关于动画的知识，一个挑战是不够的——我已经给了你三个!

这些挑战让您有机会在您的巴哈马航空登录屏幕上完成开发工作，并接受您的第一个uber haxx0r挑战。哇!

## 要点
* 苹果(Apple)提供了一组预定义的动画，称为过渡(transition)，你可以用它来处理应用程序UI状态中的特殊变化。
* 转换的目标是在视图层次结构中添加、删除和替换视图。
* 当你在设计动画时，你可以在模拟器中调试完全回到慢动作模式下观察它们。

## 挑战
### 挑战1:选择你最喜欢的过渡
到目前为止，您只看到了一个内置的转换动画。你不好奇看看其他的是什么样子吗?

在这个挑战中，您可以尝试所有其他可用的过渡动画，并使用您喜爱的动画进度消息横幅。

打开ViewController，在showMessage(index:)中找到指定转换动画的行。

```swift
UIView.transitionWithView(status, duration: 0.33, options:
 [.curveEaseOut, .transitionCurlDown], animations: ...
```

用其他可用的转换动画替换. transitioncurldown，然后构建并运行项目，看看它们是什么样子。以下是可用的转换列表:

```swift
.transitionFlipFromLeft
.transitionFlipFromRight
.transitionCurlUp
.transitionCurlDown
.transitionCrossDissolve
.transitionFlipFromTop
.transitionFlipFromBottom
```

你认为哪个动画和屏幕上的其他动画配合得最好?

如果你没有最喜欢的，试试我最喜欢的transition: . transitionflipfrombottom。我认为它非常适合横幅图形:



[图片]



### 挑战2:将表单重置为初始状态
对于这个挑战，您将通过取消单击Log In按钮后运行的所有动画，将表单重置为初始状态。这样，如果登录失败，当用户第二次点击Log In按钮时，将再次看到所有动画。

以下是完成这项挑战所需的一般步骤:

1. 创建一个新的空方法resetForm()，并从占位符comment //reset form所在的代码中调用它。

2. 在resetForm()中，使用transition(with:duration:options: animation:completion:)将状态的可见性设置为hidden，并将center设置为self.statusPosition。这应该将旗帜重置为其初始状态。使用0.2秒的持续时间进行转换。

3.如果隐藏横幅的转换使用与显示横幅的动画完全相反的动画，那就太好了。例如，如果您通过. transitioncurldown显示横幅，那么使用. transitioncurlup隐藏它。transitionflipfrombottom的倒数是。transitionflipfromtop。等等。

4. 接下来，在resetForm()中添加一个对animate(带有duration:delay:options: animation:completion:)的调用。在动画闭包块内进行如下调整:

* 移动自己。旋转器——Log In按钮内的活动指示器——到它原来的位置(-20.0,16.0)。
* 设置self的alpha属性。旋转到0.0隐藏它。
* 将Log In按钮的背景颜色调回其原始值:UIColor(红色:0.63，绿色:0.84，蓝色:0.35,alpha: 1.0)。
* 继续重置对Log In按钮所做的所有更改，并减小limit .size。宽度属性值为80.0。
* 最后，将按钮移回到密码字段下的原始位置，并减小中心。y乘以60。0。

如果您精确地反转了身份验证过程中的所有动画，那么屏幕当所有身份验证消息显示完毕后，动画应如下图所示:



[图片]



做得好!现在是本章的最后一个挑战…

### 挑战3:动画背景中的云
如果背景中的云在屏幕上慢慢移动，然后从另一边重新出现，这不是很酷吗?

是啊，那一定很酷——这就是你的挑战!
4个云图像视图已经连接到ViewController中的4个outlet，所以可以开始了。你可以尝试让云自己移动，使用你新发现的过渡动画知识，或者你可以按照下面的方法:

1. 创建一个带有签名animateCloud(cloud: UIImageView)的新方法，并在其中添加代码。
2. 首先，计算平均云速度。假设云应该在大约60.0秒内穿过整个屏幕。调用该常量cloudSpeed并将其设置为60.0 / view.frame.size.width。

3.接下来，计算动画将云移动到屏幕右侧的持续时间。记住，云不是从屏幕的左边缘开始的，而是从随机的点开始的。您可以通过计算云需要遵循的路径的长度并将结果乘以平均速度(view.frame.size.width - cloud.frame. original .x) * cloudSpeed来计算正确的持续时间。

4. 然后使用上面计算的时间调用animate(带有duration:delay:options:animation:completion:)。您需要从中创建一个TimeInterval实例，因为编译器不会为您决定正确的类型:TimeInterval(duration)。对于选项参数使用.curveLinear;这是少数几次你将使用没有缓动的动画之一。云层自然是在相当远的背景中，所以它们的运动看起来绝对是均匀的。
5. 在动画闭包表达式中设置frame.origin。将云的x属性设置为self.view.frame.size.width。这将云移动到屏幕区域之外。
6. 在completion closure块中，将云从当前位置移动到屏幕的相反边缘。不要忘记像本章前面那样使用“_”跳过闭包参数。要正确定位云，请设置它的frame.origin。x -cloud.frame.size.width。
7. 仍然在完成闭包中工作，向animateCloud()添加一个调用，以便让云在屏幕上重新动画。
8. 最后，在viewDidAppear()的末尾添加以下代码，以启动所有4个云的动画:

```swift
animateCloud(cloud1)
animateCloud(cloud2)
animateCloud(cloud3)
animateCloud(cloud4)
```

这应该使所有的四朵云慢慢地穿过屏幕，以创建一个漂亮的，不引人注目的效果。

如果你完成了本章的挑战，恭喜你!他们很艰难!

在过去的几章中有很多信息需要消化，但是你使用了一个僵硬的静态登录表单，并把它变成了一个吸引眼球的有趣的用户体验:



[图片]



是时候从新的材料和把你所有的视图动画知识测试一下了!在下一章中，您将使用您所学到的各种实用的东西来为Bahama Air应用程序添加一些重要的润色。