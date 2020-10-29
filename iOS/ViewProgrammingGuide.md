

[View Programming Guide for iOS](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009503)

# View and Window Architecture

## View Architecture Fundamentals

每一个`UIView`背后都有一个`CALayer`处理渲染和动画，通过`view.layer`获取`CALayer`层

1. View Drawing Cycle
   1. 当View第一次出现在屏幕时，系统将其内容保存为一个快照，并将快照显示在屏幕上
   2. 在每一个 run loop 结束后，系统查询View是否修改了内容 （使用UIView的`setNeedsDisplay`接口告诉系统需要重新绘制）
      1. 如果没修改，则使用缓存的快照作为其显示内容
      2. 如果修改了，则重新绘制快照 （自定义绘制使用UIView的` drawRect`接口）
2. Content Modes
   1. `view.contentMode`控制的是当`view`的`frame`、`bounds`等信息改变了之后，如何使用缓存的快照适应新的大小。也就是说如果只改变`view`的`frame`、`bounds`等信息，而不改变其具体显示的内容，系统不会重新绘制图层，而是将缓存的内容拉伸、放缩适应新的空间。
   2. 例外：如果view.contentMode == UIViewContentModeRedraw，则无论什么改变都会重新绘制图层。

## View Geometry and Coordinate Systems

默认坐标系以左上角为原点，坐标值是浮点数。每一个View都有两套坐标系统：父view和自己

1. bounds, frame and center

   1. 这三个都可以改变，但改变一个，另外的可能也会受影响
   2. 默认subview的frame超出superview的部分不会裁剪，如果将superview.clipToBounds设置为YES，则会裁剪。无论是否裁剪，只有在superview范围内的subview才能响应用户事件，范围外的点击了无反应。

2. 用view.transform去动态改变一个view的位置（绘图与动画）

3. Points V.S Pixels

   1. iOS中，每个设备屏幕大小都是用的points来度量，一个point不一定对应一个pixel
   2. 对编程者来说，只需要使用point即可

4. 从用户点击界面到系统作出响应，中间发生了什么？

   See [The Runtime Interaction Model for Views](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/WindowsandViews/WindowsandViews.html#//apple_ref/doc/uid/TP40009503-CH2-SW1)

## Tips for Using Views Effectively

1. 尽量不使用自定义图层绘制
2. 利用content mode来拉伸内容
3. 尽量将 view.opaque 设置为YES，否则该view背后的view还需要绘制，占用了系统资源
4. 调整滚动时的View绘制行为
5. 不要在系统提供的UIConrol上增加subview



# Windows



# Views

## Creating and Configuring View Objects

1. UIView的常用property
   1. 透明度：opaque, alpha, hidden
   2. 大小和位置：bounds, frame, transform, center
   3. 自动调整大小：autoresizingMask, autoresizesSubviews
   4. 渲染模式：contentMode, contentScaleFactor
   5. 用户交互：gestureRecognizers, userInteractionEnabled, multipleTouchEnabled, exclusiveTouch
   6. 视图内容：backgroundColor, subviews, drawRect, layer
2. tagging
   1. 设置view.tag的内容后，可以调用包含该view的父view的viewWithTag方法快速得到该view

## Creating and Managing a View Hierarchy

1. 增加、删除、管理 subviews
2. 在View树里定位某个view
3. 平移、缩放和旋转
4. 坐标变换
   1. 注意如果某个view是旋转过的，坐标变化的结果会有差异噢

## Adjusting the Size and Position of Views at Runtime

1. 使用subview.autoresizingMask来控制当superview的大小改变后，subview怎么变化
2. 实现layoutSubviews方法来手动控制subviews的大小位置变化
   1. sample code [ScrollViewSuite](https://developer.apple.com/library/archive/samplecode/ScrollViewSuite/Introduction/Intro.html#//apple_ref/doc/uid/DTS40008904)

## Modifying Views at Runtime

## Interacting with Core Animation Layers

主要参考 Core Animation Programming Guide



# Animation

## Animating Property Changes of one View

主要用到了 [UIView animateWith...]接口

## Creating Animated Transitions Between Views

主要用到了 [UIView transitionWith...]接口



