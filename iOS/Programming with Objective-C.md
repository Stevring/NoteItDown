[文档地址](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Introduction/Introduction.html)

# Introduction

OC中的基本概念: 

- Category
- Protocol
- Values and Collections (NSNumber, NSArray ...)
- Blocks
- Error Objects (used for runtime problems)
- Code Conventions

# Defining Classes

1. OC中, 类本身就是一种对象, 在类方法中的 `self` 关键字指向类对象

2. OC是一门动态语言体现在
   1. 对象一般都是动态创建的
   2. 某个实例所属的对象是在运行时确定的
   3. 调用方法的地址是在运行时确定的

# Working with Objects

# Encapsulating Data

1. OC对象的一个重要作用是保存数据, 数据一般声明为对象的属性:

   ```objective-c
   @interface SomeObject: NSObject
   @property NSString *name;
   @end
   ```

2. 属性的综合

  编译器会为每个属性综合出 accessor methods 来获取或设置属性的值:
  
  ```objective-c
  - (NSString)name {...}
  - (void)setName: (NSString *)name {...}
  ```

3. 以及一个实例变量 (instance variable)来保存属性的值, 实例变量是真正保存数据的地方, 实例变量一般以下划线+属性名组成`_name`, 也可以自行设定实例变量的名字:

  ```objective-c
@implementation SomeObject
  @synthesize name = ivar_name; // 设定实例变量的名字为ivar_name
  @synthesize name; // 设定实例变量的名字与属性名相同
  @end
  ```
  
3. 不具有属性的实例变量

   可以声明一个不具有对应属性的实例变量, 外界无法获取和设置该实例变量, 类似于c++里的private

   ```objective-c
   @interface SomeObject: NSObject {
     NSString *privateVar;
   }
   @property NSString *name;
   @end
   ```

   

4. 初始化方法

   1. 在初始化方法中, 访问属性时应该直接使用实例变量, 因为此时对象还未完成初始化

5. 实例变量的综合

   1. 如果自定义了某个 readwrite 属性的 getter 和 setter 方法或者自定义了某个 readonly 属性的 getter 方法, 则编译器不会自动synthesize其实例变量

   2. 因为此时编译器认为编程人员 takes full control over this property

   3. 如果确实需要一个实例变量, 可以手动综合:

      1. ```objective-c
         @implementation SomeObject
         @synthesize name = _name;
         @end
         ```

6. 使用weak属性需注意测试其是否为nil

   1. ```objective-c
      NSObject *cachedObject = self.someWeakProperty; //保留一个强引用避免weak属性失效
      if (cachedObject) { //检查是否为nil
        [someObject doSomethingImportantWith:cachedObject];
      }                                                        
      cachedObject = nil; // 废弃强引用   
      ```

7. copy属性

   1. 使用copy修饰的属性在赋值时会复制一份源数据, 因此源数据所属的类必须支持 NSCopying 协议

   2. 对于copy修饰的属性, 在初始化时使用实例变量必须也要copy源数据

      1. ```objective-c
         - (id)initWithObject:(SomeObject)obj
         {
         	...
           _objCopyProperty = [obj copy];
           ...
         }
         ```

# Customizing Existing Classes

1. Category

   1. Category可以扩展某个类的方法, 使用category声明的方法在该类的所有实例对象上都可以调用

   2. Category的声明和实现通常不在原有类文件中

   3. Category中适用于添加实例方法和类方法, 但不太适用于添加属性

      1. 添加属性是合法语法, 但由于无法在Category中添加实例变量
      2. 所以编译器无法为Category中新增的属性 synthesize 实例变量和 accessor methods
      3. 当然可以自己实现这个属性的 accessor methods, 前提是使用到的实例变量已经在原来的类中生成了.

   4. Category方法名冲突

      1. Best Practice: 在方法名前增加前缀:

         1. ```objective-c
            @interface NSSortDescriptor (XYZAdditions)
            + (id)xyz_sortDescriptorWithKey:(NSString *)key ascending:(BOOL)ascending; // 前缀应该小写, 然后加下划线
            @end
            ```

2. Class Extension

   1. 在Extension中增加的方法必须在原来类的 `@implementation` 块中实现, 因此Extension要求有源码
   2. 在Extension中也可以添加实例变量
   3. Extension和原来的类一起编译

   4. Extension的语法:

      ```objective-c
      @interface XYZPerson () // 括号中写了标识符是Category, 不写是Extension
      @property (readwrite) NSString *uniqueIdentifier;
      @end
       
      @implementation XYZPerson
      ...
      @end
      ```

# Working with Protocols

1. 编译器不会为 protocol 里声明的属性进行综合

# Values and Collections

1. Collections的fastEnumeration:

   1. ```objective-c
      for (id obj in array) {
        do something
      }
      ```




# Common Questions

1. `instancetype` 和 `id` 的区别
   1. 二者都代表任意指针, 但前者只能作为函数返回值, 后者可用于任何地方.
   2. 若`obj1`的某方法返回值类型为`instancetype`, 则编译器认为该方法返回的对象类型是`obj1`所属的类. 如果返回类型是`id`,则编译器只知道返回的是一个对象, 无法进行静态检查.

