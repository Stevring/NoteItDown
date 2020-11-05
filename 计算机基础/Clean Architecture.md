# Clean Architecture

## Part 1 Introduction 

一直在讲软件架构设计的重要性，可以直接跳过。

## Part 2 Programming Paradigms

Three paradigms:

- structured programming
- object-oriented programming
- functional programming

All three of them takes away something from the programmer (imposes disciplines to programmers).

### Object-Oriented Programming

Three important characteristics about OO:

1. Encapsulation
2. Inheritance
3. Polymorphism



### Functional Programming

1. Variables in functional languages do not vary.
2. Why do we need immutability of variables ?
   1. Because all race conditions, concurrent problems are due to mutable variables.
3. How can we pratically have immutability?
   1. we firstly need unlimited storage and infinite processor speed, which are impossible.



## Part 3 Design Principles

Design principles applies to mid-level components or modules instead of low-level code. They tell us how to organize data and functions into a group, and how each groups should be connected. Therefore  SOLID is not only for OO, but is for every system that has such groups.



### Single Responsibility Principle

