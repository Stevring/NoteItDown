# Getting Started

1. Key-Value Coding 提供了一种间接访问对象的属性的方法 (通过字符串)
2. 要使得某个类符合 Key-Value Coding 规范, 需使其遵从 NSKeyValueCoding 协议
3. NSObject 遵从了 NSKeyValueCoding 协议, 因此所有继承自 NSObject 的类都遵从该协议

# Key-Value Coding Fundamentals

## Accessing Object Properties

1. 一个对象的 property 属于以下几类

   1. Attributes. 纯粹表示某个值, 如 NSNumber, int, float, NSColor 等
   2. To-one relationships. 一对一关系, 对某个对象的引用
   3. To-many relationships. 一对多关系, 通常NSArray, NSSet 表示对多个对象的引用

2. Key 和 Key Paths

   1. Key 对应着对象的属性, 通常就是属性名称的字符串表示
   2. Key Paths 是使用"."连接的多个key, 表示对对象寻找的路径

3. 使用 Key 和 Key Paths 获取对象属性

   1. `valueForKey:`

   2. `valueForKeyPath:`

   3. `dictionaryWithValuesForKey:`

   4. 如果 key 不存在, 对象会发送`valueForUndefinedKey:`消息给自己, 默认抛出异常

   5. 如果 Key Paths 中存在 To-many relationships, 最后返回的结果也是 Collection

      1. 经过实验, Key Paths 的中间节点不能是 NSDictionary 类型, 否则会返回 nil, 同时不会报错

4. 使用 Key 和 Key Paths 设置对象属性
   1. `setValue:forKey:`
   2. `setValue:forKeyPath:`
   3. `setValuesForKeysWithDictionary:`
   4. 如果 key 不存在, 对象会发送`setValue:forUndefinedKey:`消息给自己, 默认抛出异常
   5. 如果将要设置的值是 `nil`, 对象会发送 `setNilValueForKey:`给自己, 默认抛出异常

## Accessing Collection Properties

1. 如果对象某个属性是集合, 且想要获取或设置集合中的某个元素, 可以参考[文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/AccessingCollectionProperties.html#//apple_ref/doc/uid/10000107i-CH4-SW1), 这些方法更高效

## Using Collection Operators

1. 如果要获取或设置的属性是一个集合, OC提供的一些[集合运算符](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/CollectionOperators.html#//apple_ref/doc/uid/20002176-BAJEAIEE) 比较方便
   - 如 @avg, @count, @max, @min, @sum

## Representing Non-Object Values

1. Key-Value Coding 的对象转换
   1. 由于Collection中不能含有 `nil`元素, 因此获取某个对象的属性如果是 `nil`, 会被转化为 `NSNull`
   2. 同理, 如果将要设置的值是`NSNull`, 则会转化为`nil`再设置给对象
   3. int, float 等 scalar (对象的属性) 和 NSNumber (获取的值或将要设置的值) 也会进行转换
   4. 也就是说Key-Value Coding返回的结果全是对象, 设置的值也全是对象

## Validating Properties

1. Key-Value Coding 还可用于检测对象的属性是否等于某个值, 参考[文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/ValidatingProperties.html#//apple_ref/doc/uid/10000107i-CH18-SW1)

## Accessor Search Patterns

1. Key 和 Key Paths的搜索, 参考[文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/SearchImplementation.html#//apple_ref/doc/uid/20000955-CJBBBFFA)
2. 除非自己实现相关方法, 否则不需要太了解

# Adopting Key-Value Coding

## Achieving Basic Key-Value Coding Compliance

1. 通常来说编译器自动综合的 Accessor methods 和 instance variable 可以支持 Key-Value Coding
2. 如果是自己实现 Accessor methods 和 综合 instance variable, 要注意命名方式
   1. 自己实现 Getter
      1. 普通变量可以使用与属性名相同的方法: `ReturnType <Key>`
      2. 布尔变量可以使用`is`作为前缀: `BOOL is<Key>`
   2. 自己实现 Setter
      1. 使用`void set<Key>`作为方法名
      2. 如果被设置的值有可能是`nil`, 还需要重载方法`setNilValueForKey:`
   3. instance variable
      1. 使用普通的命名: 下划线 + 属性名

## Defining Collection Methods

1. 如果某个属性是Collection类型的, 且要手动实现 Accessor Methods, 则需要实现一些方法让这个属性表现得和NSArray 或者 NSSet 一样, 参考[文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/DefiningCollectionMethods.html#//apple_ref/doc/uid/10000107i-CH17-SW1)

## Handling Non-Object Values

1. 对于某些不是对象的属性, 需要实现`setNilValueForKey:`来对 `nil` 和scalar value之间转换



## Desining for Performance

## Checklist

参考[文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/Compliant.html#//apple_ref/doc/uid/20002172-BAJEAIEE)





