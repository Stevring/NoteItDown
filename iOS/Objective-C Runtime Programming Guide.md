# Interacting with the Runtime

1. OC程序从三个层面与Runtime系统交互
   1. OC源码, 如调用某个对象的方法, 最后会被Runtime转化为消息
   2. NSObject的方法. NSObject 提供了一些与 Runtime 交互的方法, 如 `respondsToSelector:`
   3. Runtime方法. 直接调用Runtime方法获取信息

# Messaging

## The  objc_msgSend Function

1. 所有对方法的调用在 Runtime 都会转化为 `objc_msgSend(receiver, selector, arg1, arg2, ...)`
2. `objc_msgSend`首先找到`selector`的地址, 然后调用该方法后将其返回值作为自己的返回值
3. 寻找`selector`的地址方式如下
   1. 每一个类的实例都有一个`isa`指针指向类对象
   2. 每一个类对象都有一个`dispatch table`, 记录了各个方法对应的内存地址
   3. 首先寻找`obj.isa`指向的类是否有`selector`
   4. 如果没有, 就找`obj.isa.superclass`指向的类, 直到搜寻到根类(NSObject)
   5. 如果找到了方法, 就调用
   6. 否则抛出异常
4. 为了提高寻找`selector`的效率, Runtime 为每个类缓存了方法地址, 查询时首先查看缓存是否有该方法的条目

## Using Hidden Arguments

1. `obj_msgSend`方法在调用具体的方法时, 自动增加了两个参数
   1. `self` 指向实例对象
   2. `_cmd` 是一个SEL, 指向调用的方法的名字

## Getting a Method Address

1. NSObject提供了获取方法地址的方法`methodForSelector:`
2. 一般不会直接调用对象的方法

# Dynamic Method Resolution

1. 可以在运行时给类动态加载方法, 参考[文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtDynamicResolution.html#//apple_ref/doc/uid/TP40008048-CH102-SW1)

# Message Forwarding

## Forwarding

1. `objc_msgSend`如果没有找到某个方法的地址, 会调用该对象的`forwardInvocation`方法
2. 可以自己重载这个方法, 将消息发送到其他对象, 这称为" message forwarding"

## Forwarding and Inheritance

1. Forwarding使得对象A可以调用对象B的方法`sel`, 看起来就像A继承了B
2. 但 Forwarding 和继承的区别在于
   1. 通过Forwarding转发的方法并不属于A, 所以`[A respondsToSelector:sel]` 会返回`NO`
   2. 同时`[A isKindOfClass:[B class]]` 会返回`NO`
   3. `instancesRespondToSelector:` , `conformsToProtocol:`类似
   4. 在某些情况下, 如果要A表现得就是B的子类, 需要手动实现上述方法, 参考[文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtForwarding.html#//apple_ref/doc/uid/TP40008048-CH105-SW12)

# Type Encodings

1. 对于每个方法, 编译器都会将方法的[返回值和各个参数]的类型编码成一个字符串与之对应
2. 在源码中可以使用`@encode()`指令来对类型进行转化

# Declared Properties

1. 主要是一些反射机制, 用于获取类的方法, 属性等等