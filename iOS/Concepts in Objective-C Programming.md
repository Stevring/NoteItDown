[toc]

# Concepts in Objective-C Programming

## Design Patterns

key-value observing

receptionist

delegation

target-action

## Object Allocation

1. allocates enough memory (page-aligned)
2. sets retain count = 1
3. initializes *isa* instance variable to be the object's class
4. initializes all other instance variables to zero

## Object Initialization

> Initialization sets the instance variables of an object to reasonable and useful initial values

### Basic Rules

1. initializer begins with *init* 

2. initializer may return different object than the one begin allocated:
   1. if the object has one singleton
   2. if the object is identified as unique
   
   therefore, always rely on the initialized instance instead of the “raw” just-allocated one.
   
3. do not reinitialize objects

### Implement an Initializer

1. invoke [super init] first. The order of initialization accounts.

## Model-View-Controller

### Types of Cocoa Controller Objects

1. mediating controller:

   > facilitate—or mediate—the flow of data between view objects and model objects

2. coordinating controller:

   > oversee—or coordinate—the functioning of the entire application or of part of the application


## Target-Action
## Object Modeling

### Key-Value coding

accessing a model object's attributes without knowing its implementation.

1. **Always return an object for a given key.** If the return type or the data type for the specific accessor method or instance variable used to supply the value for a specified key is not an object, then an `NSNumber` or `NSValue` object is created for that value and returned in its place.
2. **Always supply an object for a specific key.**  When you set a value using key-value coding, if the data type required by the appropriate accessor or instance variable for the specified key is not an object, then the value is extracted from the passed object using the appropriate `-`*type*`Value` method.
3. **Key paths**. If one node in the key path is nil, the key path traversal stops and returns nil.
