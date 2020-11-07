# Clean Architecture

## Part 1 Introduction 

一直在讲软件架构设计的重要性，可以直接跳过。

## Part 2 Programming Paradigms

Three paradigms:

- structured programming
- object-oriented programming
- functional programming

All three of them takes away something from the programmer (imposes disciplines to programmers).

### Structured Programming

1. 背景：希望像推导数学公式一样证明程序的正确性。
2. Conclusion
   1. If a programme only uses sequential，loop and branching  execution, we can decompose it into smaller modules, each of which is easy to prove correctness.
      - Sequential execution: give some input and test whether the output is as expected.
      - Loop: prove the base condition is correct, then use inducing.
      - branching: prove every possible branch is correct.
   2. If a programme uses unlimited `goto` statement, it will be hard to decomposed into provable smaller modules.
   3. So we should constrain our usage of `goto`.

### Object-Oriented Programming

What is OO? Encapsulation, inheritance and polymorphism. An OO language must support these three things.


### Functional Programming

1. Variables in functional languages do not vary.
2. Why do we need immutability of variables ?
   1. Because all race conditions, concurrent problems are due to mutable variables.
3. How can we pratically have immutability?
   1. we firstly need unlimited storage and infinite processor speed, which are impossible.


## Part 3 Design Principles

Design principles applies to mid-level components or modules instead of low-level code. They tell us how to organize data and functions into a group, and how each groups should be connected. Therefore  SOLID is not only for OO, but is for every system that has such groups.

### Single Responsibility Principle

Single Responsibility: each module or class should has only one reason to change.