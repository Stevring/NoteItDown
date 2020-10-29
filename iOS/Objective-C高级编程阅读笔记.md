[TOC]

# 第一章 自动引用计数

## 1.1 什么是自动引用计数

自动引用计数是指编译器自动进行内存管理，无需手动`retain`或`release`代码

## 1.2 引用计数

1. 持有对象的两种方式
   1. 调用 `alloc/new/copy/mutableCopy` 开头的方法创建的对象，自己自动持有
   2. 调用不以 `alloc/new/copy/mutableCopy` 开头的方法得到的对象，是autorelease对象，自己不会自动持有，但可以使用 `[obj retain]` 持有该对象
2. 不需要自己持有的对象时释放
3. `alloc/retain/retainCount/release` 的实现
   1. GNUstep在对象头部前面一块区域记录该对象的retainCount
   2. Apple使用散列表记录对象的retainCount，散列表的键是对象内存块地址的散列值
4. `autorelease` 的实现
   1. 所有的 `AutoReleasePool` 对象存储在一个类似于栈的结构，每当调用`[obj autorelease]`时都将`obj`放到栈顶的`AutoReleasePool`里
   2. 当调用 `AutoReleasePool`对象的`drain`方法时，会将该对象的`pool`池里的所有对象都`release`

## 1.3 ARC 规则

1. `__strong` 修饰符
   1. 是`id`类型和对象类型的默认修饰符
   2. 持有对象的强引用
2. `__weak` 修饰符
   1. 两个对象互相使用`__strong`修饰的变量引用时，发生了循环引用的问题，即无法release对象
   2. 使用`__weak`修饰符的变量，不会对对象强引用
   3. `__weak`指向的对象被释放后, 该指针会被置为nil
3. `__unsafe_unretained`修饰符
   1. 与`__weak`修饰符类似，但既不持有强引用，也不持有弱引用
   2. 在低版本iOS和OS X中作为`__weak`的替代
   3. 但`__unsafe_unretained` 指向的对象释放后, 该指针仍指向原内存空间 (成为野指针)
4. `__autoreleasing` 修饰符
   1. 在使用ARC时，不能直接调用`[obj autorelease]`，也不能直接生成和使用`AutoReleasePool`对象
   2. 使用`@autoreleasepool{Other Code}`替代`AutoReleasePool`对象
   3. 使用`__autoreleasing` 修饰的对象会自动被注册到autoreleasepool, 等价于调用对象的autorelease方法
   4. 一般来说，会**非显式使用__autoreleasing**
      1. 如果不是 `alloc/new/copy/mutableCopy`开头的方法, 返回的对象会自动注册到`autoreleasepool` 
      2. 访问`__weak`修饰的变量时,因为使用`__weak`修饰的变量指向的对象有可能被废弃，所以每次在使用`__weak`修饰的变量前 编译器会自动生成一个`__autoreleasing`的中间变量并注册到`autoreleasepool`, 因此如果大量使用某个`__weak`变量, 最好将其暂时赋值给某个强引用.
         1. 这一条不对，现在已经不会将weak注册了
      3. 指针的指针. `NSError **error` 会被自动修改为 `NSError *__autoreleasing * error`，`id *obj` 会被自动修改为`id __autoreleasing *obj`

5. C语言指针（Core Foundation框架）与Objective-C对象（Foundation框架）的转换  ` __bridge/ __bridge_retained/__bridge_transfer`

  ```objective-c
  SomeTypeRef p;
  NSObject *x; //假设x持有某个对象 object
  1.
  p = (__bridge SomeTypeRef)x; 
  // object的引用计数仍为1, p不持有该对象
  2.
  p = (__bridge_retained SomeTypeRef)x; // 等价于 p = (CFBridgingRetain SomeTypeRef)x;
  // object的引用计数为2, p持有该对象
  3.
  p = (__bridge_transfer SomeTypeRef)x; // 等价于 p = (CFBridgingRelease SomeTypeRef)x;
  // object的引用计数为1, p持有该对象, x不持有该对象(相当于执行了 [x release]; )
  ```

6. 对象的属性与所有权修饰符的对应关系

  | 属性 (这些都是修饰setter的) | 所有权修饰符          |
  | ----------------- | --------------------- |
  | assign            | `__unsafe_unretained` |
  | copy              | `__strong`            |
  | retain            | `__strong`            |
  | unsafe_unretained | `__unsafe_unretained` |
  | weak                  | `__weak`                      |



7. 数组

   1. 静态数组

      > `__unsafe_unretained`修饰符以外的修饰符 会将静态数组的元素初始化为`nil`,  这与普通变量相同

      这里不太对，经过实验发现`__unsafe_unretained`也会将变量初始化为`nil`

   2. 动态数组
      1. 系统提供的`NSMutable*` 一类动态容器会自动持有追加的对象并进行管理.
      2. > C风格的动态数组 初始化必须为nil: `[id|NSArray|...] __strong *array = nil`. 因为编译器不保证*指针* 类型的变量初始化为nil. 分配成员内存时, 必须用`calloc` 或 `malloc + memset` 将内存块初始化为0. 释放内存时, 必须手动将数组中的元素赋值为nil, 再`free(数组)`, 因为编译器不能确定动态数组的生命周期, 所以无法自动插入释放对象的代码 (静态数组可以).
      
         经过实验，发现并不能使用` NSArray *array = calloc(...)` 这样的代码

8. `__weak`修饰的变量, 在该对象废弃时, 要进行一系列操作以将该变量赋值为`nil`, 所以使用`__weak`应该克制一点, 否则会占用大量CPU资源.

# 第二章 Blocks

## 2.1 Blocks概要

1. blocks是带有局部变量（自动变量）的匿名函数

## 2.2 Block模式

1. blocks相关的语法

   ```objective-c
   // 声明一个函数func1，该函数的参数是 (int x)，返回类型是一个block，该block的参数是 (int, int)，block的返回值是int
   int (^func1(int x))(int, int)
   {
       NSLog(@"%d", x);
       return ^int(int count, int y){return y+1;};
   }
   
   // 声明了block，参数是(int, float)，返回值是int，并用blk来指代这种block
   typedef int (^blk)(int, float);
   ```

2. 截获局部变量 
   1. 如果block使用了某个外部的变量，则block截获的是创建block时的变量值，后续更改该变量不会影响block截获的值
   2. 如果要在block中对该变量赋值，则需要将该声明为`__block int x`（在block中使用截获的值不会有问题(例如给截获的Objective-C 对象发送消息)）
   3. 声明为`__block int x`后，在外部改变该变量，block内部也能感知到

## 2.3 Blocks的实现

1. block会被转化为一个类，初始化block时，参数会作为类的初始参数传入，调用block相当于调用该类的一个方法

2. 如果block用到了外部的局部变量，该变量也作为参数传入类的初始化方法，因此在外部更改这些值，block内部无法感知

3. 而全局变量，全局静态变量，在block的方法里可以直接访问，所以不会作为参数传入block类的初始化函数；但局部静态变量会把其指针传入block类的初始化方法，所以在block内部是可以直接更改其值的（不用加`__block`修饰符）

4. 局部变量之所以不像局部静态变量那样传入指针进行block初始化，是因为局部变量在这个函数return之后就失效了，block内部即便有指针也成为了野指针。

5. 局部变量增加`__block`修饰符后，在编译时会被转化为一个结构体，该结构体有一个成员对应该局部变量。在block类初始化时，将该结构体的指针传入，所以block内部也可以修改局部变量。

6. Block的存储

   | Block定义方式                        | 存储区 |
   | ------------------------------------ | ------ |
   | 全局变量、全局静态变量、局部静态变量 | 数据区 |
   | 局部变量                             | 栈     |
   |                                      |        |

   如果某个函数定义了一个局部变量block，且将其作为返回值，则编译器会自动插入代码将该block复制到堆，这就使得即便超出了该block的作用域，block也能使用。

   >p114 上方：向方法或函数的参数中传递Block时，编译器不能判断是否需要将block复制到堆中，因此需要手动复制

   这个地方似乎有问题，经过实验发现不手动复制也不会有问题。

7. `__block`变量的存储

   1. `__block val`原本保存在栈上，且会被转化为一个结构体`s`，该结构体中有一个`__forwarding`指针指向自身，对`__block val`的访问会转化为`s.__forwarding->val`

   2. 如果某个block用了该变量，且该block被复制到堆中了，则该`s`也会被复制到堆中，设其为`t`

   3. 然后将栈上`s.__forwarding`指向`t`，`t.forwarding`也指向t

   4. block中指向`s`的指针转化为指向`t`的指针

   5. 这样就能使得

      > 不管__block变量配置在栈上还是堆上，都能正确访问该变量

   



# 第三章 Grand Central Dispatch

1. GCD中的两种Dispatch Queue: Serial Dispatch Queue和Concurrent Dispatch Queue。前者线性执行追加的任务，后者并行执行追加的任务

2. 创建一个queue

   1. Serial Dispatch Queue: 

      `dispatch_queue_t someQueue = dispatch_queue_create(@"com.queue.name", NULL)`

   2. Concurrent Dispatch Queue:

      `dispatch_queue_t someQueue = dispatch_queue_create(@"com.queue.name", DISPATCH_QUEUE_CONCURRENT)`

3. > Dispatch Queue 没有像Block那样转化为对象，所以必须手动释放 `dispatch_release(someQueue)`对应的, 通过函数或方法获取的Dispatch Queue有必要:`dispatch_retain(someQueue)`,并在不需要时release

   这一点存疑，经过实验发现ARC模式下调用`dispatch_release`和`dispatch_retain`会导致编译错误

4. 使用`dispatch_set_target_queue`更改自己创建的queue的优先级

5. 使用`dispatch_after`设定某个时间后再执行

6. 使用Dispatch Group可以等待多个block完成后执行结束处理.

   ```objective-c
   dispatch_queue_t queue = dispatch_get_global(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
   dispatch_group_t group = dispatch_group_create(); // 用于监听这个group里的block
   dispatch_group_async(group, queue, blk1); //blk会retain这个group
   dispatch_group_async(group, queue, blk2);
   dispatch_group_async(group, queue, blk3);
   
   dispatch_group_notify(group, dispatch_get_main_queue(), blk4); //等待group中的blk执行结束后, 在main_queue中执行blk4, 如果不需要结束后处理某些业务, 可使用dispatch_group_wait(dispatch_group_t, dispatch_time_t)
   
   dispatch_release(group) //结束后释放group
     
   // blk1-3的执行顺序不定, 但blk4一定是最后一个执行的
   ```

