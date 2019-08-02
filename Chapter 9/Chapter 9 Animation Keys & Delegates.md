# Chapter 9: Animation Keys & Delegates

棘手的部分关于UIKit关闭动画和相应的语法是,如果你不使用UIViewPropertyAnimator等一些新的api(你将在以后的章节),一旦你创建并运行一个视图动画无法暂停,停止,或者以任何方式访问它。

然而，使用Core Animation，您可以很容易地检查正在层上运行的动画，并在需要时停止它们。此外，您甚至可以在动画上设置委托对象并对动画事件作出响应。与您在视图动画中看到的完成块相反，您可以在动画开始和结束(或中断)时接收委托回调。

在本章中，您将继续与巴哈马航空登录项目，并使用动画委托使您的动画更具互动性。

## 介绍动画的代表



[图片]



CAAnimation及其子类CABasicAnimation实现委托模式，并允许您响应动画事件。
CAAnimationDelegate提供了两个方法，如果你需要它们中的一个或两个，你可以实现:

```swift
func animationDidStart(_ anim: CAAnimation)
func animationDidStop(_ anim: CAAnimation, finished flag: Bool)
```

您可以打开本章的启动项目，或者继续您自己的项目(如果您已经完成了上一章及其挑战)。

您的第一个任务是使用animationDidStop()使每个表单元素在到达屏幕上的最终位置时跳动一次。

打开ViewController.swift并将以下代码添加到viewWillAppear()中，就在将动画添加到标题层的行之前:

```swift
flyRight.delegate = self
```

然后将委托方法添加到视图控制器扩展名中的类中:

```swift
extension ViewController: CAAnimationDelegate {
  func animationDidStop(_ anim: CAAnimation,
    finished flag: Bool) {
    print("animation did finish")
  }
}
```

这只是向控制台输出一行代码，以显示您实际上正在调用委托方法。

建立和运行您的项目;您应该会在Xcode控制台看到以下输出:



[图片]



好了，你的委托被调用了——但是你如何辨别哪个动画在上面的代码中停止了呢?记住，您向三个不同的层添加了相同的动画!下面的小节将向您展示如何使用键值对来管理它。

## 键值编码合规
CAAnimation类及其子类是用Objective-C编写的，并且与键值编码兼容，这意味着您可以像对待字典一样对待它们，并在运行时向它们添加新的属性。

您将使用此机制为flyRight动画分配一个名称，以便从其他活动动画包中识别它。

回滚到viewWillAppear()，并在设置flyRight委托的行后面添加以下代码:

```swift
flyRight.setValue("form", forKey: "name")
flyRight.setValue(heading.layer, forKey: "layer")
```

在上面的代码中，您在flyRight动画上创建键名并将其设置为form。现在，您可以从委托回调中检查这个name键，以将动画标识为属于表单控件之一。

您还可以分配标题。图层到图层键，这样你就有了一个对动画所属图层的引用。

您可以保持名称不变，但是您应该在每次将动画添加到图层时更新图层键。

找到添加动画到用户名层的行，并在它之前添加以下内容:

```swift
flyRight.setValue(username.layer, forKey: "layer")
```

类似地，找到将动画添加到密码层的行，并在它之前添加以下内容:

```swift
flyRight.setValue(password.layer, forKey: "layer")
```

现在，flyRight的每个副本都带有对其运行所在层的引用。

### 开启键值
现在动画上已经设置了键，可以在animation delegate方法中检查它们。

在animationDidStop的底部添加以下代码:

```swift
guard let name = anim.value(forKey: "name") as? String else {
	return
}
if name == "form" {
//form field found
}
```

在上面的代码中，您使用value(forKey:)从动画中获取name的值，并尝试将该值转换为String。使用这个保护只是因为value(forKey:)在没有为提供的键分配值的情况下返回一个可选的值，或者如果向下转换失败，则返回一个可选的值。

可选绑定语句将为您处理这些检查。如果您可以打开该值，那么您就有了一个可以测试的值。如果没有，则继续执行代码。

现在，当表单动画完成时，您可以运行一些代码。将comments //表单字段替换为以下内容:

```swift
let layer = anim.value(forKey: "layer") as? CALayer
anim.setValue(nil, forKey: "layer")
let pulse = CABasicAnimation(keyPath: "transform.scale")
pulse.fromValue = 1.25
pulse.toValue = 1.0
pulse.duration = 0.25
layer?.add(pulse, forKey: nil)
```

完成之后，将layer的值设置为nil，以删除对原始层的引用。

> 注意:记住value(forKey:)总是返回一个AnyObject?;因此，您必须将结果转换为所需的类型。不要忘记强制转换操作可能会失败，因此必须在上面的示例中使用选项来处理错误条件，例如当name键存在而layer键不存在时。

最后，您创建了一个动画，它将层的比例提高了一点点(1.25倍)，然后将层动画恢复到原来的大小(1.0倍)。

注意，这里使用的是带有图层的可选链接?-这意味着如果动画中没有存储层，add(_:forKey:)调用将被跳过。由于你之前将图层设置为nil，这个脉冲动画只会在表单字段第一次从右边飞进来时发生。

建立和运行您的项目;您应该看到，每当flyRight动画完成时，元素将在停止之前跳动一次:



[图片]



他的动画提供了一个很好的边缘，并吸引用户的注意力到这些控件。它负责处理已经停止的动画;但是如何处理仍然在运行的动画呢?这就是动画键的作用。



## 动画的钥匙



[图片]



您可能已经注意到`add(_:key:)`有两个参数;到目前为止，您只使用了第一个函数来传递动画对象本身。

key参数是一个字符串标识符，它允许您在动画启动后访问和控制动画。

在本部分中，您将创建另一个图层动画，学习如何一次运行多个动画，并发现如何使用动画键来控制正在运行的动画。

你将添加一个新的动画标签到你的表单，给用户一些基本的指令，如下图所示:



[图片]



您的新标签将缓慢地从右向左移动。一旦用户开始输入用户名或密码，此标签将停止移动并直接跳转到其最终位置。一旦用户知道要做什么，就不需要继续动画。

首先，您需要添加一些代码来将标签设置为animate。在ViewController顶部的属性声明末尾添加以下属性:

```swift
let info = UILabel()
```

这是将用于向用户显示指令的标签。

接下来，在viewDidLoad的底部添加以下代码来设置标签的各种属性:

```swift
info.frame = CGRect(x: 0.0, y: loginButton.center.y + 60.0,  width:
view.frame.size.width, height: 30)
info.backgroundColor = .clear
info.font = UIFont(name: "HelveticaNeue", size: 12.0)
info.textAlignment = .center
info.textColor = .white
info.text = "Tap on a field and enter username and password"
view.insertSubview(info, belowSubview: loginButton)
```

这将在login按钮下面添加标签。当用户点击login按钮时，它会向下移动并覆盖指令，因为用户不再需要它们了。

现在，当应用程序第一次打开时，您需要将说明从右边滑进去。找到viewDidAppear()，并将以下代码添加到该方法的末尾:

```swift
let flyLeft = CABasicAnimation(keyPath: "position.x")
flyLeft.fromValue = info.layer.position.x + view.frame.size.width
flyLeft.toValue = info.layer.position.x
flyLeft.duration = 5.0
info.layer.add(flyLeft, forKey: "infoappear")
```

这将使您的标签具有动画效果，就像您的表单字段一样，只有您的标签将从屏幕的右侧滑到左侧。注意，您将动画的键设置为infoappear;您将使用此值查找动画，然后当用户开始在用户名或密码字段中输入文本时停止动画。

### 添加第二层动画
现在，您将向标签添加第二个动画，该动画将在标签滑动时淡入说明。在viewDidAppear的末尾添加以下代码:

```swift
let fadeLabelIn = CABasicAnimation(keyPath: "opacity")
fadeLabelIn.fromValue = 0.2
fadeLabelIn.toValue = 1.0
fadeLabelIn.duration = 4.5
info.layer.add(fadeLabelIn, forKey: "fadein")
```

上面的代码在标签中从几乎不可见的不透明度0.2淡入到完全可见的不透明度1.0。上面的动画运行在一个不同的属性-不透明度-比在以前的动画-位置-有一个完全不同的持续时间。这表明可以在同一层上独立运行多个动画。

> 注意:如果需要同步图层上的动画，那么需要使用动画组来实现。下一章将介绍这种技术。

运行该项目，看看组合动画的行动:



[图片]



到目前为止，一切顺利。现在是棘手的部分:如果用户开始输入用户名或密码，则取消动画。

### 确定运行动画
您需要知道用户何时点击用户名或密码字段。在ViewController.swift的底部添加以下扩展名:

```swift
extension ViewController: UITextFieldDelegate {
  func textFieldDidBeginEditing(_ textField: UITextField) {
    guard let runningAnimations = info.layer.animationKeys() else {
			return
		}
    print(runningAnimations)
  }
}
```

这将添加一个类扩展，以说明ViewController符合UITextFieldDelegate协议。像这样将协议分离到它们自己的扩展中有助于保持文件的组织。

您需要实现的一个方法textfielddidbeginedit将在用户开始编辑文本字段时调用。现在，您只需打印出标签上运行的所有活动动画的列表。

接下来，在viewDidAppear的底部添加以下代码:

```swift
username.delegate = self
password.delegate = self
```

这将使类成为两个文本字段的委托。
建立和运行您的项目;点击用户名或密码字段之前的标签动画完成，你应该看到以下输出在您的Xcode控制台:



[图片]



成功!一旦你知道哪个动画正在运行，你可以用它们做几件事:

1. 调用该层上的removeallanimation()来停止所有正在运行的动画，或者调用removeAnimation(forKey:)只删除一个动画。

2. 枚举animationKeys()返回的动画列表。动画对象是不可变的，因此不能在它们进行时修改动画。
3. 使用animationForKey(_:)键获取动画。与之前一样，返回的animation对象将是不可变的。

为了完成你的“停止动画”效果，你将取消移动标签的动画，但是让淡入动画继续运行，因为如果标签的不透明度直接跳到1.0会显得很奇怪。

添加以下代码到textfielddidbeginedit:

```swift
info.layer.removeAnimation(forKey: "infoappear")
```

这将删除滑入动画，并使信息标签直接跳到屏幕的中心——而不影响淡入动画。

最后一次构建和运行您的项目;点击任意一个文本框，你会看到指令直接跳到屏幕中央，并继续淡入:



[图片]



这是一个包装!您已经学习了委托方法，通过键访问动画，并创建了您的第一个复合层动画作为额外的奖励!

为什么不通过下面的挑战来测试你的知识呢?

## 要点
* 你可以为你的图层动画设置一个委托，并在动画开始或结束时得到通知。

* 动画是键值编码兼容的，所以你可以附加任意的数据片段，给你更多关于动画上下文的信息。

* add(_， forKey:) API允许您选择指定一个forKey字符串参数来给特定的动画命名。

## 挑战
在这个挑战中，您将通过用层动画替换现有的云动画来增强您对动画委托和键值编码的知识。

用以下方法替换animate(cloud: UIImageView)方法:

```swift
func animateCloud(layer: CALayer) {
//1
  let cloudSpeed = 60.0 / Double(view.layer.frame.size.width)
  let duration: TimeInterval = Double(
    view.layer.frame.size.width - layer.frame.origin.x)
    * cloudSpeed
//2
  let cloudMove = CABasicAnimation(keyPath: "position.x")
  cloudMove.duration = duration
  cloudMove.toValue = self.view.bounds.width +
    layer.bounds.width/2
  cloudMove.delegate = self
  cloudMove.setValue("cloud", forKey: "name")
  cloudMove.setValue(layer, forKey: "layer")
  layer.add(cloudMove, forKey: nil)
}
```

这个方法与使用UIKit动画云的方法有类似的实现。实际上，上面//1部分的前几行几乎与UIKit实现相同。

在上述方法的第二部分中，您将为图层移动创建一个CABasicAnimation实例，并在动画上设置一个名称和图层。

接下来，将viewDidAppear()中的调用替换为animateCloud()，如下所示:

```swift
animateCloud(layer: cloud1.layer)
animateCloud(layer: cloud2.layer)
animateCloud(layer: cloud3.layer)
animateCloud(layer: cloud4.layer)
```

您现在的任务是通过实现重置每个云的委托代码来完成动画。
您已经知道如何通过animationDidStop(_: finished:)来实现这一点。遵循以下要点:

* 检查动画名称是否为“cloud”。
* 如果是，从“图层”键获取动画的图层。
* 设置图层的位置。x -layer.bounds.width / 2。
* 等待半秒(使用delay(seconds:)函数)并调用self.animateCloud()，传入当前层。

建立和运行您的项目;检查云一旦离开屏幕就会重新出现:



[图片]



祝贺你-你已经学习了基本的图层动画技术，并准备继续创建更复杂的动画。您将在下一章“组和高级计时”中学习动画组。