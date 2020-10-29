# Introduction

# Collection View Basics

1. 展示一个 Collection View 需要的对象有
   1. **Top-level containment and management**: UICollectionView, UICollectionViewController
   
   2. **Content management**: UICollectionViewDataSource, UICollectionViewDelegate
   
   3. **Presentation**: UICollectionReusableView, UICollectionViewCell
   
   4. **Layout**
   
      UICollectionViewLayout
   
      UICollectionViewLayoutAttributes
   
      UICollectionViewUpdateItem
   
      UICollectionViewFlowLayout：提供的是全局layout information
   
      UICollectionViewDelegateFlowLayout：动态提供一些layout information
2. Collection View的动画
   1. 当删除、插入一个cell时，UICollectionView会自动animate该操作
   2. 但是如果调用 layout 的invalidate方法，则不会自动显示动画

# Designing Your Data Source and Delegate

1. highlight state 和 selected state 的区别

学了 Custom layout之后尝试 Transitioning Between Layouts

学了 Text Programming Guide 之后 尝试 EditMenu for a Cell

# Using the Flow Layout

# Incoporating Gesture Support



# Creating Custom Layouts

## Subclassing UICollectionViewLayout

1. **collection view layout process**。当 collection view 需要布局信息时，它直接调用layout object的方法。使用 layout object 的 invalidataLayout 方法手动让 collection view 更新布局。collection view 调用 layout object 的方法过程如下。

   1. `prepareLayout`：初始化计算
   2. `collectionViewContentSize`：计算collection view的content大小
   3. `layoutAttributesForElementsInRect:`：计算与该rectangle有交集的所有view的layout information

   **Note**：调用layout object的`invalidateLayout`方法并不会立刻重新开始layout process，而是会等到下一个view update cycle。

2. **Providing Layout Attributes On Demand**

   除了上面的常规layout process外，有时候collection view需要知道某个元素的layout信息，layout object需要实现以下方法

   - `layoutAttributesForItemAtIndexPath:` (required)
   - `layoutAttributesForSupplementaryViewOfKind:atIndexPath:` (if has supplementary views)
   - `layoutAttributesForDecorationViewOfKind:atIndexPath:` (if has decoration views)

   

## Making Your Custom Layouts More Engaging

1. supplementary views （以下简称SV）

   1. 是什么？SV 与 collection cell 一样，都由 data source 提供，其主要目的是辅助显示内容，（比如 section header，section footer）。

   2. 有什么性质？SV 也是重复利用的，因此需要继承`UICollectionViewResuableView`

   3. 怎么用？

      1. 在 collection view 注册SV 

         `registerClass:forSupplementaryViewOfKind:withReuseIdentifier:`

      2. 在 data source 中提供SV ：

         `collectionView:viewForSupplementaryElementOfKind:atIndexPath:`

         使用

         `dequeueReusableSupplementaryViewOfKind:withReuseIdentifier:forIndexPath:`获取SV实例（和Cell相同）

      3. 在 layout object中为SV生成layout信息

         `layoutAttributesForElementsInRect:`

      4. 在layout object中实现`layoutAttributesForSupplementaryViewOfKind:atIndexPath:`

2. decoration views （以下简称DV）

   1. 是什么？仅用于装饰性的视图，与data source无关。由layout object管理。

   2. 性质？由于DV仅仅是装饰性的，因此在初始化时（`initWithFrame:`）就应完成所有配置。当layout object提供的layoutAttributes中含有DV时，collection view会自动创建DV。DV也需要继承`UICollectionViewResuableView`，因为layout object采用了重用机制。

   3. 怎么用？

      1. 在layout object注册

         `registerClass:forDecorationViewOfKind: `

      2. 在layout object中为DV生成layout信息

          `layoutAttributesForElementsInRect:`

      3. 在layout object中实现

         `layoutAttributesForDecorationViewOfKind:atIndexPath:`

      4. (optional) 在layout object中实现`initialLayoutAttributesForAppearingDecorationElementOfKind:atIndexPath:` and `finalLayoutAttributesForDisappearingDecorationElementOfKind:atIndexPath:`来处理DV的显示和消失

         see [Making Insertion and Deletion Animations More Interesting](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/CreatingCustomLayouts/CreatingCustomLayouts.html#//apple_ref/doc/uid/TP40012334-CH5-SW13)

   4. 

   







https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/CreatingCustomLayouts/CreatingCustomLayouts.html#//apple_ref/doc/uid/TP40012334-CH5-SW1



Question

1. Decoration View 和 Supplementary View 的区别

   - Decoration View仅仅是装饰性的view，由layout object决定它是哪个类（registerClass: forDecorationViewOfKind:）并提供位置信息，由collection view隐式创建，程序员无法获取到其引用（目测是使用了layout object的register信息）。

   - Supplementary View更多的与Data相关，由collection view决定是哪个类（registerClass:forSupplementaryViewOfKind:），layout object提供位置信息，同时由Data Source创建(collectionView:dequeReusableSuppplementaryViewOfKind:withIndexPath:)。

   see [UIcollectionview decoration view VS supplementary view](https://stackoverflow.com/questions/19175995/uicollectionview-decoration-view-vs-supplementary-view)

   



