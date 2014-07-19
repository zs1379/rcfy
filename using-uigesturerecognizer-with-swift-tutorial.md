原文：http://www.raywenderlich.com/76020/using-uigesturerecognizer-with-swift-tutorial

更新提示：这篇教程已经由Caroline Begbie为适配IOS8及Swift做了更新。原帖由Ray Wenderlich发布。<br />

假如你想要在你的应用中检测手势，例如点击，捏合，平移，或者旋转，用Swift和内建的UIGestureRecognizer类实现是非常容易的。<br />

在这篇教程中，你将学会如何简单地在你的应用中增加手势识别，不管是在Xcode的故事板中或者用编程方式。你将创建一个简单的应用程序，你可以移动一只猴子及一个香蕉，在手势识别的帮助下进行拖动，捏合。<br />

你还可以尝试一些很酷的方式，例如：<br />
* 加入运动感应器<br />
* 在手势识别间建立依赖关系<br />
* 创建一个自定义的UIGestureRecognizer，然后你可以给猴子挠痒<br />

本教程假设你已经熟悉了故事板的基础概念。假如你刚刚接触故事板，可以先看看我们故事板的教程。<br />

我觉得猴子已经给我们竖起了大拇指，所以让我们开始把:]<br />

		注：在写这篇教程的时候，我们的理解是我们不能发布Xcode6的截图，
		因为它还处于测试阶段，因而我们将不会在本Swift教程中进行截图，直到我们确认它是OK的。
		
### 入门<br />

打开Xcode6，然后通过iOS\Application\Single View应用模板创建一个项目。应用名称直接填写MonkeyPinch，语言选择Swift，然后设备选择iphone。点击下一步，选择目录保存你的项目，然后点击创建。<br />

在你进行任何的下一步前，下载本项目的资源，然后将这六个文件放到你的项目中。勾选目标文件夹：如果需要的话拷贝它们，选择创建组，然后点击完成。<br />

接着，打开故事板。试图控制器现在设为默认，这样就可以为多个设备使用一个故事板。通常你将使用constraints和size类布局你的故事板。但是因为这个应用将只有iphone版本，因此你可以禁用size类。在文件检查器面板(视图菜单>工具>显示文件检查器)，将使用size类前的勾选去掉。选择保留size类数据：iphone，然后点击禁用size类。<br />

你的视图现在将显示iphone 5的尺寸及比例。<br />

拖动图片视图到视图控制器。设置图片为monkey.png，并且通过选择(编辑菜单>匹配内容大小)调整图片的大小去匹配图片。当拖进第二张图片，设置它为banana.png，同样调整它的大小。按照你喜欢的方式排列图片视图。此时，你应该拥有类型下面的东东：<br />

![排列视图中的图像](http://cdn2.raywenderlich.com/wp-content/uploads/2014/07/uigesture-arrange.png)

这是这个应用的UI-现在你将增加一个手势识别去拖动这些形象。<br />

### UIGestureRecognizer概述<br />

在开始之前，先看一份如何使用UIGestureRecognizers以及为什么它们如此得心应手。<br />

在以前的UIGestureRecognizers，如果你想要检测一个手势，例如移动，你必须注册UIView上所有点击上的通知-例如touchesBegan, touchesMoves, 和touchesEnded。每个程序员写了稍微不同的代码来检测触摸，从而导致了整个程序的小错误以及不一致。<br />

在IOS 3.0，苹果公司就开始拯救UIGestureRecognizer类！它提供了默认的常见手势，例如点击，捏合，旋转，平移，按，长按。通过使用它们，不仅仅为你节约了很多代码，并且能让你的程序运行得很好！当然你还可以使用旧的点击事件替代，假如你的应用需要它们。<br />

使用UIGestureRecognizers是非常简单的。你只需要执行以下的步骤：<br />

* 1：创建一个手势识别。当你创建一个手势识别，你可以指定一个回调函数，因而手势识别可以给你发送更新消息，当手势开始，修改或者结束。<br />
* 2：创建一个手势识别到一个视图。每个手势识别将与一个（也只能有一个）视图相关联。当触摸发生在该视图上时，手势识别器将去搜索是否有匹配的触摸类型，如果有的话则调用回调函数。<br />

你可以通过编程来执行这两个步骤（你将会在这个教程后面做到），但是使用故事板来增加手势识别就更简单了。因此现在添加你的第一个手势识别到该项目吧！

### UIGestureRecognizer<br />

依然打开故事板，看里面的滑动手势识别引用对象，并且拖动它到猴子图像视图的上面。这同时也创建了滑动手势识别，并与猴子图像视图相关联。你可以通过点击猴子的图像视图来检查链接，查看链接检查器（视图>工具>显示链接检查器），然后确认滑动手势识别在gestureRecognizers的Outlet集合中。<br />

你也许相知道为什么它关联到图像视图而不是视图本身。这两种方法都是好的，这只是看哪种更适合你的项目。既然你把它放在猴子上，你知道所有的点击都玉猴子相关联。这种方法不好的地方在于你可以想要点击在它之外的地方。在这种情况下，你可以将它关联到视图本身，但是你需要写代码去检查用户点击是否在猴子的范围内或者香蕉的范围内，并且做出相应的反应。<br />

现在你创建了滑动手势识别并且将它绑定到了图像视图，你只需要去写回调事件函数去处理当事件发生时需要做的事情。<br />

打开ViewController.swift并且添加以下的函数到ViewController类：<br />

		@IBAction func handlePan(recognizer:UIPanGestureRecognizer) {
  		let translation = recognizer.translationInView(self.view)
  		recognizer.view.center = CGPoint(x:recognizer.view.center.x + translation.x,
                		                   y:recognizer.view.center.y + translation.y)
  		recognizer.setTranslation(CGPointZero, inView: self.view)
		}

UIPanGestureRecognizer将会调用这个函数当触摸事件第一次发生，当用户持续触摸时会多次调用，最后一次当触摸完成(一般是用户将手指拿开)。<br />

UIPanGestureRecognizer将自身当做一个参数传到函数里。你可以通过调用translationInView:函数去获得用户移动手指的量。<br />

当你完成时将translation设置回0是非常重要的。否则的话translation将一直保持，然后你会看到你的猴子移出屏幕！<br />

记住不要硬编码猴子图像视图到函数中，你通过recognizer.view取得一个猴子图像视图的引用。这使得你的代码更加通用，因此你下次可以使用相同的香蕉图像视图。<br />

好了，现在这个函数完成了，你将把它挂钩到UIPanGestureRecognizer。在故事板，将它从滑动手势感应器拖向视图控制器。一个弹出窗口将出现，选择handlePan:。<br />

还有一件事。如果你编译并且运行，并且尝试拖动猴子，这将不会正常工作。原因是因为在视图上的点击默认情况下是被禁用的，类似图像视图。因此选择所有的图像视图，打开属性检查器，并且检查用户交互选择框是否处于勾选状态。<br />

再次编译和运行，然后这个时候你可以在屏幕中拖动猴子！<br />

记住你无法拖动香蕉。这是因为手势识别器只连接到了一个视图(也只能一个)。因此通过以下的步骤为香蕉也创建一个手势识别器：<br />

* 拖动滑动手势识别到香蕉图像视图上。<br />
* 拖动新的滑动手势识别到视图控制器并且链接handlePan：函数。<br />

去尝试下，你现在可以在屏幕上拖动两个图像视图。非常简单地实现这么一个很酷很好玩的效果，不是么？<br />











