# Chapter 25: Intermediate 3D Animations 

在上一章中，您了解到将透视应用于单个视图并不是一项复杂的任务; 事实上，一旦你知道m34和相机距离的秘密，你就可以创建各种3D动画。

本章以您已经学过的内容为基础，向您展示如何使用多个视图创建令人信服的3D动画。

本章的入门项目是一个简单的飓风图片库。 到本章结束时，您将获得3D效果以获得图库中图像的整体视图：



[图片]



你可以点击一张照片将它全屏显示，然后点击右上角的按钮将所有图像再次打开 - 当然，用一个很酷的动画将你带到两个状态之间！

准备开始了吗？ 坚持你的帽子，准备好被这个“暴风雨”的项目所震撼！

## 探索入门项目
从本章的Resources文件夹中打开starter项目; 构建并运行它以查看您必须开始的内容：



[图片]



你只拥有一个空白屏幕，顶部有两个条形按钮：左边一个显示NASA图像信用，右边一个调用显示或隐藏图库的方法。

首先，您需要在屏幕上显示所有图像并进行设置，以便为“粉丝”动画做好准备。

打开ViewController.swift并检查类代码。 你会看到一个名为images的数组; 此数组包含一些略微自定义的图像视图。 ImageViewCard类继承自UIImageView并添加一个字符串属性标题来保存飓风标题，并添加一个名为didSelect的属性，以便您可以轻松地在图像上设置点击处理程序。

您的第一个任务是将所有图像添加到视图控制器的视图中。 将以下代码添加到viewDidAppear（_ :)的末尾：

```swift
for image in images {
  image.layer.anchorPoint.y = 0.0
  image.frame = view.bounds
  view.addSubview(image)
}
```

在上面的代码中，您循环遍历所有图像，在y轴上将每个图像的锚点设置为0.0并调整每个图像的大小以使其占据整个屏幕。 完成后，您可以添加每个图像进行查看。

设置锚点可让图像围绕其上边缘而不是中心的默认值旋转，如下图所示：



[图片]



这将使在3D空间中散布图像变得更加容易。

构建并运行您的项目，看看到目前为止您取得了哪些成就：



[图片]



嗯 - 你只看到你添加的最后一张图片。 那是因为所有图像都有相同的帧，所以你只看到你添加的最后一张图片：Hurricane Irene。

为了更明显地显示哪个飓风图像，请在viewDidAppear（_ :)的末尾添加以下行：

```swift
navigationItem.title = images.last?.title
```

此代码采用集合中最后一个飓风图像的名称，并将其设置为视图控制器的导航标题，如此。

建立并再次运行以熟悉艾琳：



[图片]



请注意，您没有在图像上设置任何透视变换; 您将直接在视图控制器的视图上设置透视图。

在上一章中，您在单个视图上调整了transform属性，然后在3D空间中旋转它。 但是，由于您当前的项目有更多的个人视图，您需要在3D中进行操作，因此您可以设置其父视图的透视图，而不是为自己节省大量工作。

将以下代码添加到viewDidAppear（_ :)：

```swift
var perspective = CATransform3DIdentity
perspective.m34 = -1.0/250.0
view.layer.sublayerTransform = perspective
```

在这里，您可以使用图层属性sublayerTransform来设置视图控制器图层的所有子图层的透视图。 然后将子层变换与每个单独层的自身变换组合。

这使您可以专注于管理子视图的旋转或平移，而无需担心透视。 您将在下一节中更详细地了解其工作原理。

## 改变画廊
toggleGallery（_ :)连接到右侧的“浏览”栏按钮，您可以将3D变换应用于四个图像。

将以下变量添加到toggleGallery（_ :)：

```swift
var imageYOffset: CGFloat = 50.0
```

由于您不仅可以旋转所有图像，而只需移动它们以生成“扇形”动画，因此可以使用imageYOffset设置每个图像的偏移。

接下来，您需要遍历所有图像并运行其各自的动画。

将以下代码添加到toggleGallery（_ :)：

```swift
for subview in view.subviews {
  guard let image = subview as? ImageViewCard else {
	continue
  }
  // more code here
}
```

在这里，您遍历视图控制器视图的所有子视图，并仅对作为ImageViewCard实例的子视图执行操作。

在上面添加的保护块之后添加以下代码，以替换此处的更多代码注释：

```swift
var imageTransform = CATransform3DIdentity
//1
imageTransform = CATransform3DTranslate(
  imageTransform, 0.0, imageYOffset, 0.0)
//2
imageTransform = CATransform3DScale(
  imageTransform, 0.95, 0.6, 1.0)
//3
imageTransform = CATransform3DRotate(
  imageTransform, .pi/8, -1.0, 0.0, 0.0)
```

首先将标识转换分配给imageTransform，然后对其添加一系列调整。 这是每个单独的调整对图像的作用：

1. 使用CATransform3DTranslate在y轴上移动图像; 这会将图像从其默认的0.0 y坐标偏移，如下所示：



[图片]



之后，您将分别计算每个图像的imageYOffset; 现在所有图像移动的数量都相同，所以你现在仍然只能看到最顶层的图像。

2. 使用CATransform3DScale通过调整变换的比例分量来缩放图像。 您可以在x轴上稍微缩小图像，但是在y轴上将其缩小到60％以丰富旋转3D效果：



[图片]



3. 最后使用CATransform3DRotate将图像旋转22.5度，使其产生一些透视变形，如下所示：



[图片]



请记住，您已经设置了锚点，因此图像围绕其顶部边缘旋转。

现在你看到通过view.layer.sublayerTransform设置m34值的值; 旋转变换只需重新使用子层变换中的m34值，而无需在此处应用它。 那很方便！

现在剩下的就是将变换应用于每个图像。 添加以下行（仍在for body中）：

```swift
image.layer.transform = imageTransform
```

构建并运行您的项目; 点击“浏览”按钮查看转换结果：



[图片]



同样，您只能看到顶部图像，因为它后面的所有其他图像都应用了相同的变换。 现在是为每个图像定制变换的绝佳机会。

将以下行添加到for块的末尾：

```swift
imageYOffset += view.frame.height / CGFloat(images.count)
```

这会根据图像在堆栈中的位置调整每个图像的y偏移。 您将屏幕高度除以图像数量，以便它们在屏幕上均匀分布。

构建并运行您的项目以查看您的视图：



[图片]



甜蜜 - 画廊看起来就像你想要的那样！ 但由于这是一本关于动画的书，如果你没有动画过渡到扇形视图，那将是一种耻辱，不是吗？

## 动画画廊
在toggleGallery（_ :)中找到以下行，您可以在每个图像上设置变换：

```swift
image.layer.transform = imageTransform
```

在该行上方插入以下代码以进行动画转换：

```swift
let animation = CABasicAnimation(keyPath: "transform")
animation.fromValue = NSValue(caTransform3D:
  image.layer.transform)
animation.toValue = NSValue(caTransform3D: imageTransform)
animation.duration = 0.33
image.layer.add(animation, forKey: nil)
```

这段代码非常熟悉：您可以在transform属性上创建一个图层动画，并将其从当前值设置为您之前设计的imageTransform。

再次构建并运行您的项目; 点击“浏览”按钮，欣赏完成的动画：



[图片]



你现在已经完成了画廊; 当您在用户点击“浏览”按钮时添加关闭风扇的功能时，您将在“挑战”部分重新访问它。

## 将图像带到前面
在最后一节中，您将为图像库添加一些交互性：点击图像将使其在其他图像前跳转，以便用户可以更好地查看它ImageViewCard已经具有名为的闭包表达式属性didSelect; 当用户点击图像并将点击的图像视图作为输入参数接收时，这会触发。

要添加此功能，您需要向ViewController添加一个方法，并将其分配给所有图像视图上的didSelect。

首先将以下代码添加到for循环体内的viewDidAppear（）：

```swift
image.didSelect = selectImage
```

Xcode会抱怨selectImage不存在，但你会在下一步中解决这个问题。

将以下方法添加到ViewController：

```swift
func selectImage(selectedImage: ImageViewCard) {
  for subview in view.subviews {
    guard let image = subview as? ImageViewCard else {
		continue
    }
    if image === selectedImage {
      //selected image
	} else {
      //any other image
	} 
  }
}
```

这是selectImage的骨架（selectedImage :); 当用户点击其中一个图像时，您可以遍历所有子视图并像以前一样获取ImageViewCard实例。 然后检查每个ImageViewCard以查看它是否是所选图像。

现在您还需要两个动画：一个用于为所选图像设置动画，另一个用于为图库中的所有其他图像设置动画。

你将反过来解决这个问题并首先淡出未选择的图像。

使用以下代码替换selectImage（selectedImage :)中的 //任何其他图像注释：

```swift
UIView.animate(withDuration: 0.33, delay: 0.0,
  options: .curveEaseIn,
  animations: {
    image.alpha = 0.0
  },
  completion: {_ in
    image.alpha = 1.0
    image.layer.transform = CATransform3DIdentity
  }
)
```

这是一个淡出每个图像的简单动画。 在完成块中，将其变换重置为标识变换，将alpha重置为完全不透明。 到上述动画完成时，所选图像将位于所有其他图像的前面 - 如此透明或不透明，您将看不到任何未选择的图像。 在此处重置alpha可以避免在想要再次查看图像时重置它。

构建并运行您的项目; 打开图库并点击图像以查看所有其他未选择的图像从视图中淡出。



[图片]



当该动画完成时，视图会更改为全屏显示顶部图像 - 因为您没有对所选图像执行任何操作！ 你现在就解决这个问题。

您将动画所选图像的变换返回到身份变换，并在动画完成时将图像拉到前面。

使用以下代码替换selectImage（selectedImage :)中的//选定图像注释：

```swift
UIView.animate(withDuration: 0.33, delay: 0.0,
  options: .curveEaseIn,
  animations: {
    image.layer.transform = CATransform3DIdentity
  },
  completion: {_ in
    self.view.bringSubviewToFront(image)
  }
)
```

在这里，您没有对动画进行3D变换，然后确保图像位于视图堆栈的顶部，以便它可见。

最后，将以下代码添加到selectImage（selectedImage :)的末尾：

```swift
self.navigationItem.title = selectedImage.title
```

这将使用当前选定的图像标题更新导航栏。

构建并运行您的项目; 点击各种图像，看看它们如何缩放到全屏。 它看起来怎么样？

我以为你会喜欢这个优雅的小动画！ 这是一个包装; 在下面的挑战部分中有一小部分功能可以清理，但除此之外，您还可以轻松获得令人惊叹的动画效果。 我怀疑你可以想到很多方法在你自己的项目中使用它！

## 关键点
* 通过m34属性设置3D透视并使用生成的变换在图层上设置sublayerTransform时，可以在3D空间中为其所有子图层设置动画。
* 您可以将CATransform3D图层动画与UIKit视图动画相结合，让Core Animation自动组合并在屏幕上渲染它们。

## 挑战
### 挑战1：使用“浏览”按钮切换图库
现在，用户打开图像后必须从图库中选择图像。 在此挑战中，您将使“浏览”按钮像切换一样工作以关闭图库视图。

向ViewController添加一个名为isGalleryOpen的新属性，并将其初始值设置为false。 您需要在代码中的几个位置更新此属性的值：

* 在toggleGallery（_ :)结束时将其设置为true
* 在selectImage（selectedImage :)结束时将其设置为false

在toggleGallery（）的顶部，添加一个检查以查看图库是否已打开。 如果是这样，则遍历所有图像并将其变换设置为原始值。 不要忘记重置isGalleryOpen并返回，以便其余的方法代码也不会执行。

就是这样 - 如果您愿意，可以在这个项目中使用动画，看看您可以自己添加哪些其他华丽元素！