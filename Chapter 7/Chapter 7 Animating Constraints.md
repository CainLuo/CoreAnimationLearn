# Chapter 7: Animating Constraints

在前一章中，您学习了如何使用自动布局为打包清单项目创建响应性用户界面。在本章中，您将进一步完善它，并为您的应用程序添加一些有弹性的动画。您已经了解到，为了让自动布局正常工作，您不能直接修改视图的frame或center属性。相反，您必须使用布局约束来创建所需的动画。

到目前为止，您已经了解了如何对视图属性进行动画:您可以将数值属性(如alpha)从一个浮点值动画到另一个浮点值。与center属性的情况类似，CGPoint的实例可以逐步修改，直到中心值到达目标位置。很自然，您的下一个问题将是“这很好——但是我如何将约束动画化?”

动画约束并不比动画属性更困难;只是有点不同。通常，您只需用一个新的约束替换一个现有的约束，并让Auto Layout在这两种状态之间使UI具有动画效果。



[图片]



唯一的例外是，当您只需要更改约束方程中的一个属性时，例如常量。在这种情况下，您只需在代码中直接修改约束并使更改具有动画效果。

在本章中，您将添加一个动画来展开打包列表菜单栏，并显示一个项目列表;用户可以点击一个物品，将其添加到他们的装箱单中，如下图所示:



[图片]



这种互动，以及一些更多的视觉享受，将推动流体和引人注目的动画。

你还在等什么?该收拾行李了!

## 动画化接口生成器约束
如果你完成了上一章的项目，你可以继续上一章的内容;否则，您可以使用本章中的starter项目。

您的第一个任务是在用户单击+按钮时展开菜单。为了做到这一点，您需要通过动画菜单栏的高度约束来改变它的高度。

### 让菜单展开
打开ViewController.swift并滚动到类的顶部。在其他出口添加以下代码行:

```swift
@IBOutlet weak var menuHeightConstraint: NSLayoutConstraint!
```

NSLayoutConstraint是表示在Interface Builder中创建的约束的类。与任何其他按钮、图像视图或标签一样，您也可以为约束创建一个outlet。

开放的主要。故事板，并选择菜单视图:



[图片]



打开Size Inspector选项卡，双击表示Height =: 44的约束。这将打开约束方程的熟悉视图:



[图片]



切换到右边的最后一个选项卡-连接检查器:



[图片]



从这里，您可以在ViewController中将约束连接到它的outlet。

从新建引用出口旁边的小圆圈拖动到视图控制器对象:



[图片]



从弹出菜单中，选择唯一可用的选项:menuHeightConstraint。您现在已经创建了您的outlet，如下图所示:



[图片]



打开ViewController.swift并向actionToggleMenu()添加以下三行代码:

```swift
isMenuOpen = !isMenuOpen
menuHeightConstraint.constant = isMenuOpen ? 184.0 : 44.0
titleLabel.text = isMenuOpen ? "Select Item" : "Packing List"
```

在上面的代码中，首先切换Boolean变量isMenuOpen，它跟踪菜单当前是展开还是折叠。

接下来，根据isMenuOpen的状态，将约束的常量属性修改为184.0 pt或44.0 pt，使菜单根据需要展开或折叠。

在最后一行代码中，您可以在Packing List和Select Item之间切换菜单标题。

建立和运行您的项目;点击几次+按钮，菜单就会按照你的计划展开和收缩:



[图片]



要使布局更改具有动画效果，您需要老朋友animate(带有duration:animation:)或任何类似的api。

### 动画布局变化
在actionToggleMenu的底部添加以下代码:

```swift
UIView.animate(withDuration: 1.0, delay: 0.0,
  usingSpringWithDamping: 0.4, initialSpringVelocity: 10.0,
  options: .curveEaseIn,
  animations: {
    self.view.layoutIfNeeded()
  },
  completion: nil
)
```

在上面的代码中，您创建了一个spring动画(正如您在第2章“spring”中了解到的)，并从动画闭包中强制更新布局。这就是动画化约束修改所需要的全部内容。

对上面发生的事情还不清楚吗?这里有更多关于动画工作原理的背景知识。

当您在动画闭包中修改相关视图属性时，它们将像您所期望的那样被动画化，自动布局在完成计算后仍然会设置视图的边界和中心。

在本例中，您已经更新了约束值，但是iOS还没有机会更新布局。通过在animation闭包中调用layoutifrequired()，可以设置布局中涉及的每个视图的中心和边界。就是这样——背景中没有魔法发生!

如果您没有调用layoutifrequired ()， UIKit无论如何都会执行布局，因为您更改了一个约束，该约束将布局标记为dirty。

重新构建和运行您的项目;点击+按钮，您将看到菜单展开和收缩与一个有弹性的动画。

你甚至可以看到菜单标题暂时覆盖了系统菜单在它的方式返回:



[图片]



当然，一旦动画稳定下来，一切看起来都很好。

您是否注意到table view也随着菜单一起收缩和增长?菜单并没有覆盖表，而是在展开时将表视图推开。

这是因为您在前一章中添加的约束将表的顶部附加到菜单的底部。当菜单增长时，表会收缩以满足约束。两个动画的价格为一个!

为了增加一些趣味，现在您将把约束动画和一些非约束动画混合起来，看看可以创建哪些新效果。

您的下一个任务是旋转+按钮45度时，菜单展开，使它类似于x -即关闭按钮。

### 旋转视图动画
既然您已经知道如何通过调整视图的变换来旋转视图，那么在最终的动画闭包中添加以下代码:

```swift
let angle: CGFloat = self.isMenuOpen ? .pi / 4 : 0.0
self.buttonMenu.transform = CGAffineTransform(rotationAngle: angle)
```

当菜单的扩张,将旋转的角度为45度(或π/ 4弧度);当它收缩时，只需将旋转设置回0。然后更新按钮上的转换以设置视图的运动状态。

建立和运行您的项目;点击+按钮，查看旋转动画在调用layoutifrequired()时的效果:



[图片]



动画看起来很华丽;您可以临时将动画持续时间设置为5-6秒，以查看+符号如何旋转成x。您还可以看到按钮围绕其中心跳动，这要感谢驱动约束和旋转动画的spring动画。

## 检查和动画约束
以可视化方式使用outlet是连接outlet的相对简单的方法，但有时不能使用Interface Builder将UI的所有部分连接到outlet。您可能会从代码中添加约束，或者您只是不想control -拖动并创建大量的outlet !

在这些情况下，您需要在运行时检查现有的约束，并在代码中修改希望动画的约束。

幸运的是，UIView类有一个名为constraints的属性，它给你一个影响给定视图的所有约束的列表。这样方便吗?

将以下代码添加到actionToggleMenu()的顶部:

```swift
titleLabel.superview?.constraints.forEach { constraint in
  print(" -> \(constraint.description)\n")
}
```

这个单行程序循环遍历所有影响菜单栏视图的约束，并将它们逐个打印到Xcode的输出控制台。

建立和运行您的项目;点击+按钮，可以看到所有的约束整齐地列出如下:



[图片]



它看起来有点乱，但是仔细阅读输出，您将能够找出每个约束的作用。以以下约束为例:

`UILabel:...'Select Item'.centerX == UIView:…centerX`

很明显这是UIView和UILabel之间的一个约束;描述还包括标签的当前文本。centerX也被提到过几次…啊哈!这必须是将标题水平居中置于菜单栏中的约束。是时候让这个坏男孩动起来了。

### 动画UILabel约束
找到actionToggleMenu(_:)顶部附近的以下行:

```swift
isMenuOpen = !isMenuOpen
```

然后在这一行下面添加以下代码:

```swift
titleLabel.superview?.constraints.forEach { constraint in
  if constraint.firstItem === titleLabel &&
     constraint.firstAttribute == .centerX {
    constraint.constant = isMenuOpen ? -100.0 : 0.0
    return
	} 
}
```

在这里，您将遍历影响菜单栏视图的约束列表，但这一次您将寻找要调整的特定约束。

你还记得水平中心约束的方程吗?它看起来像这样:

```swift
Superview.CenterX = 1.0 * UILabel.CenterX + 0.0
```

NSLayoutConstraint属性以一种非常直观的方式映射到上面的方程:



[图片]



现在，当您回头查看添加的最后一段代码时，if条件更有意义了:对于每个约束，检查secondItem是否是标题标签，以及约束是否与标题的CenterX对齐。

找到正确的约束后，将常数调整为100 pt，以便在菜单打开时将标题向左推。

> 注意:动态查找现有约束并使用它的方法稍微简单一些;你下一个会看到。

建立和运行您的项目;点击+按钮查看新的约束逻辑是如何工作的:



[图片]



请记住，您正在从spring animation API中调用layoutifrequired()，因此标题animation会弹一下。

> 注意:如果你的动画没有启动，在标签上的父视图约束中检查水平居中;确保标签是第一项，菜单栏视图是第二项。

你的UI看起来很酷，但你知道你可以更进一步。下一节将向您展示如何替换约束来创建一些整洁的动画。

## 通过替换约束进行动画
在本章的这一点上，您只修改了约束的常量属性。具有讽刺意味的是，常量属性是NSLayoutConstraint类中的一个可变属性!

如果您想修改乘数，或者以任何其他方式更改约束，您需要删除约束，然后在其位置上添加一个新的约束。

要学习如何做到这一点，您将使菜单标题的垂直对齐具有动画效果，以便在菜单打开时向上移动一点。这应该会在菜单底部留下足够的空白来显示更多的内容，您将在本章稍后添加这些内容。

这一次，您将使用一种不同的技术来确保您得到了正确的约束。

在Interface Builder中，您可以为每个约束分配一个标识符，这可以帮助您在运行时轻松地获得它。

开放的主要。并找到标题标签的对齐中心Y约束:



[图片]



双击约束，在标识符文本框中输入TitleCenterY:



[图片]



回到ViewController.swift，在for循环的末尾，在actionToggleMenu的代码中找到以下位置:



[图片]



在上述位置插入以下代码:

```swift
if constraint.identifier == "TitleCenterY" {
  constraint.isActive = false
  //add new constraint
  return
}
```

检查约束的标识符是否与要替换的标识符相同，如果是，则删除约束。你可以通过将isActive设置为false来做到这一点;这将导致视图层次结构删除约束。如果没有对约束对象的引用，则约束对象将从内存中删除。

建立和运行您的项目;点击+观察发生了什么:



[图片]



由于不再有保持视图对齐的约束，标题只会跳转到其父视图的顶部。“+”按钮顺从地跟在后面，因为它自己的“CenterY”附在标题的“CenterY”上。

有趣的是，您需要添加一个新的约束并修复布局。

### 以编程方式添加约束
当菜单被收回时，你想要在菜单视图中垂直居中的标题如下:



[图片]



或用其方程表示约束:

```swift
Title.CenterY = Menu.CenterY * 1.0 + 0.0
```

但是当菜单展开时，你想把标题向上移动一点，为后面的打包项目列表腾出空间:



[图片]



下面是详细列出的约束方程:

```swift
Title.CenterY = Menu.CenterY * 0.67 + 0.0
```

在刚刚添加的代码中找到占位符comment //add new constraint，并替换为:

```swift
let newConstraint = NSLayoutConstraint(
  item: titleLabel,
  attribute: .centerY,
  relatedBy: .equal,
  toItem: titleLabel.superview!,
  attribute: .centerY,
  multiplier: isMenuOpen ? 0.67 : 1.0,
  constant: 0)
newConstraint.identifier = "TitleCenterY"
newConstraint.isActive = true
```

NSLayoutConstraint的初始化器接受一车参数，但幸运的是，它们精确地映射到约束方程的所有部分。参数如下:

* 项目:方程中的第一项;在本例中，标题标签。
* attribute:新约束的第一项的属性。
* relatedBy:约束既可以表示数学等式，也可以表示不等式。在这本书中，你只会用到等式表达式，所以这里你用。equal来表示这个关系。
* toItem:约束方程中的第二项;在本例中，它是标题的父视图。
* attribute:新约束的第二项的属性。
* 乘法器:前面讨论过的方程乘法器。
* 常数:方程常数。

作为附加步骤，您将为约束提供标识符TitleCenterY。下次用户切换菜单时，您将使用该标识符来查找和替换刚才创建的约束。

最后，需要将约束上的active属性设置为true，这告诉Auto Layout将其应用于当前布局。

建立和运行您的项目;打开菜单，标题应该移动到菜单栏垂直中心上方的一个点，如下图所示:



[图片]



## 添加菜单内容
您的下一个工作是显示菜单中的项目列表;这些都是你可以添加到你的装箱单上的所有可能的项目。
初始项目附带的HorizontalItemList类将帮助您显示项目列表。

> 注意:本章不详细介绍HorizontalItemList;它的实现与创建动画无关。不过，如果你想知道它是如何工作的，你仍然可以浏览HorizontalItemList.swift !

还在ViewController。swift，滚动到actionToggleMenu底部，添加以下代码:

```swift
if isMenuOpen {
  slider = HorizontalItemList(inView: view)
  slider.didSelectItem = {index in
    print("add \(index)")
    self.items.append(index)
    self.tableView.reloadData()
    self.actionToggleMenu(self)
}
  self.titleLabel.superview!.addSubview(slider)
} else {
  slider.removeFromSuperview()
}
```

如果菜单即将打开，您可以在slider中创建一个新的HorizontalItemList实例来保存新项目，并为didSelectItem分配一个闭包表达式，最后将slider添加到菜单栏中。

didSelectItem在用户点击列表中的图像时运行;在此事件中，将图像索引添加到选定项列表中，并重新加载表视图。

在else分支中，当菜单即将关闭时，只需从其父视图中删除图像列表。

建立和运行您的项目;添加一些项目来查看图片列表是什么样子的:



[图片]



你差不多了;本章的最后一节将介绍如何在代码中动态创建和动画视图。

## 动态创建视图
您的最终任务是创建一个新视图，为视图添加一些约束，并将其动画化到屏幕上，这将使用您到目前为止学到的所有知识，并很好地结束本章。

当您点击表行时，ViewController中的showItem(_:)被调用。你的任务是创建一个点击图像索引的图像视图，并显示在屏幕底部，如下图所示:



[图片]



将以下代码添加到showItem(_:)中，以在所选图像中创建一个图像视图:

```swift
let imageView = UIImageView(image: UIImage(named: "summericons_100px_0\
(index).png"))
imageView.backgroundColor = UIColor(red: 0.0, green: 0.0, blue: 0.0,
alpha: 0.5)
imageView.layer.cornerRadius = 5.0
imageView.layer.masksToBounds = true                  imageView.translatesAutoresizingMaskIntoConstraints = false
view.addSubview(imageView)
```

在上面的代码中，您加载所选的图像并从中创建一个图像视图。你给它一个半透明的黑色背景，稍微圆角。注意，您没有在图像视图的父视图中设置图像视图的位置。

接下来，您将逐一为图像视图创建约束。在刚刚添加的代码下面直接添加以下代码:

```swift
let conX = imageView.centerXAnchor.constraint(equalTo:
view.centerXAnchor)
```

这个方法使用了新的NSLayoutAnchor类，这使得创建公共约束非常容易。这里，你在图像视图的中心x锚点和视图控制器的视图之间创建一个约束。

接下来，添加下面的代码，给图像视图一个底部约束:

```swift
let conBottom = imageView.bottomAnchor.constraint(equalTo: view.bottomAnchor, constant: imageView.frame.height)
```

此约束设置图像视图的底部与视图控制器的视图的底部匹配，加上图像的高度;这个位置的图像刚刚离开屏幕的底部边缘，这将作为动画的起点。

接下来，要确定图像的宽度。添加以下代码:

```swift
let conWidth = imageView.widthAnchor.constraint(equalTo: view.widthAnchor, multiplier: 0.33, constant: -50.0)
```

将图像宽度设置为屏幕宽度的1/3，小于50pt。目标大小为屏幕的1/3;您将动画移除了50 pt的差异，以便稍后使图像“成长”到合适的位置。

最后，您只需要设置图像的高度。因为图像是正方形的，所以可以将高度设置为图像的宽度。

为最后一个约束添加最后一段代码，然后在一个组中激活它们:

```swift
let conHeight = imageView.heightAnchor.constraint(equalTo:
imageView.widthAnchor)
NSLayoutConstraint.activate([conX, conBottom, conWidth, conHeight])
```

最后一个约束与其他约束稍有不同，因为这是第一次在同一个视图的两个属性之间创建关系。别担心，这样做还是可以的。

建立和运行您的项目;点击表格行，你会看到你的新图像视图如下:



[图片]



当然，这只是动画的起始位置。你现在的工作是让图像从底部弹出。

### 添加额外的动态动画
将以下代码添加到showItem(_:):

```swift
UIView.animate(withDuration: 0.8, delay: 0.0,
  usingSpringWithDamping:  0.4, initialSpringVelocity: 0.0,
  animations: {
    conBottom.constant = -imageView.frame.size.height/2
    conWidth.constant = 0.0
    self.view.layoutIfNeeded()
},
  completion: nil
)
```

在conY上更改常量将图像向上移动，调整到conWidth将图像宽度增加50 pt，使图像返回到原始大小。您不需要设置高度，因为它被自动限制为图像的宽度。

最后调用layoutifrequired()，这将启动动画。建立和运行您的项目;点击几行可以看到你的图像动画:



[图片]



挂在!图像视图从屏幕左上角开始，然后飞到中间!发生了什么事?

考虑一下:您添加了一个视图，设置了一些约束，然后更改了这些约束并激活了布局更改。然而，视图从来没有机会执行它的初始布局，所以您的图像从左上角的默认位置(0,0)开始。啊-这就是它飞进来的原因。

要解决这个问题，您需要确保在动画开始之前完成初始布局。在动画调用之前添加以下代码:

```swift
view.layoutIfNeeded()
```

这将立即设置您的初始布局之前发生的任何事情。在调用layoutifrequired()和下一次调用之间所做的所有约束更改都将成为动画的一部分。

现在就构建并运行您的项目，动画应该能够正常工作。

有些烦人的是，这些图片不断地堆积在一起;您将在本章末尾的挑战中解决这个问题。

不要忘记尝试在不同的模拟器和方向的项目;你的约束动画应该看起来很好，在所有:



[图片]



## 挑战
### 挑战1:将图像从屏幕上动画化
好了-现在你要修正那些烦人的图像视图，停留在屏幕上。

在showItem(_:)中，保持图像可见1秒，然后将其动画移出屏幕。

使用动画方法，可以为动画设置一个延迟，使图像从屏幕上恢复动画效果，最后从动画完成闭包中的视图层次结构中删除图像视图。

这就是你完成挑战所需要知道的。好运！

现在，您已经很好地理解了如何在自动布局项目中创建视图动画。虽然这本书中并不是所有的项目都使用自动布局，但是当你可以的时候，试着使用它来保持你的技能集新鲜!