---
title: "Technique: Inheritance and Polymorphism in C - Embedded Artistry"
source: "https://embeddedartistry.com/fieldatlas/technique-inheritance-and-polymorphism-in-c/"
author:
  - "[[Embedded Artistry]]"
published: 2020-02-10
created: 2025-02-18
description: "10 February 2020 by Phillip Johnston • Last updated 8 November 2023While C is not an object-oriented language, objects, inheritance, and polymorphism can be implemented in C. Table of Contents: Inheritance Polymorphism Further Reading Inheritance The core idea behind inheritance relies on structures. A base class or API can be represented by a struct definition that includes all relevant … Continue reading \"Technique: Inheritance and Polymorphism in C\""
tags:
  - "clippings"
---
[10 February 2020](https://embeddedartistry.com/blog/2020/02/10/) • Last updated 8 November 2023

While [C](https://embeddedartistry.com/fieldmanual-terms/c/) is not an object-oriented language, [objects](https://embeddedartistry.com/fieldatlas/technique-objects-in-c/), inheritance, and polymorphism can be implemented in C.

**Table of Contents:**

1. [Inheritance](https://embeddedartistry.com/fieldatlas/technique-inheritance-and-polymorphism-in-c/#Inheritance)
2. [Polymorphism](https://embeddedartistry.com/fieldatlas/technique-inheritance-and-polymorphism-in-c/#Polymorphism)
3. [Further Reading](https://embeddedartistry.com/fieldatlas/technique-inheritance-and-polymorphism-in-c/#Further-Reading)

The core idea behind inheritance relies on structures. A base class or [API](https://embeddedartistry.com/fieldmanual-terms/application-programming-interface/) can be represented by a `struct` definition that includes all relevant interfaces and data members.

For example, consider [this example of an event base “class” from an Electron Vector article](http://www.electronvector.com/blog/event-based-interfaces-for-testability):

To simulate inheritance in C, we define additional event structures **which include the base structure as the first element.** The base structure **must always be the first element**.

The key detail is that a pointer to the derived structures (`event_button_pressed_t`) is also a pointer to the base structure (`event_t`). You can safely cast between the two structures, and any derived type can be used with APIs that require only the base type.

Just remember this foundational pattern, outlined in [Dmitry Frank’s article](https://dmitryfrank.com/articles/oop_in_c):

## Polymorphism

Polymorphism is the ability to substitute objects using matching interfaces for one-another at runtime. With [C++](https://embeddedartistry.com/fieldmanual-terms/cpp/), polymorphism is handled through virtual interfaces and the virtual table (vtable). The example code used below is adapted from [Miro Samek’s excellent overview](https://www.state-machine.com/doc/AN_OOP_in_C.pdf).

For example, we can define a base “class” for a shape, along with a vtable which contains function pointers. Derived classes populate the vtable pointers with functions that are specific to the derived class.

Non-virtual functions for the base class are implemented as a standard functions which take a base class pointer as the first parameter:

One such required non-virtual function is a constructor, whose most important job is to supply the proper vtable for the class:

Any base class or derived function can reference the virtual functions using the vtable:

To derive from the base class, we simply use the base class `Shape` as the first element of the derived structure:

As a result, we can cast `Rectangle` so it can be used with `Shape`interfaces. We can also create `Rectangle`\-specific interfaces.

Of course, our derived class needs its own constructor:

Because we defined a new class, we need `Rectangle`\-specific implementations of the virtual functions. These are added to a new vtable.

Whenever a `Rectangle` is constructed, the base class constructor is called first. Then, the `vtable` is set to `Rectangle`’s table. Any virtual function calls will dereference the proper `Rectangle`\-specific implementation, *even when using base class interfaces*.

This is the basic overview of implementing polymorphism and virtual functions in C. For a detailed overview of implementing full polymorphism, see the links in Further Reading.

As a final note, if you are relying on these concepts in your C programs, you will likely benefit from switching to C++, if only for the fact that the compiler handles so much of this overhead for you. Additionally, you can rely on optimizations from the C++ compiler that won’t work in the C implementation.

## Further Reading

For more on [Inheritance](https://embeddedartistry.com/fieldmanual-terms/generalization-specialization-relationship/), Polymorphism, and OOP in C:

- [Technique: Objects in C](https://embeddedartistry.com/fieldatlas/technique-objects-in-c/)
- [Technique: Encapsulation and Information Hiding in C](https://embeddedartistry.com/fieldatlas/encapsulation-and-information-hiding-in-c/)
- [Programming embedded systems: polymorphism in C](https://www.embedded.com/programming-embedded-systems-polymorphism-in-c-2/) by Miro Samek
- [Programming embedded systems: inheritance in C and C++](https://www.embedded.com/programming-embedded-systems-inheritance-in-c-and-c/) by Miro Samek
- [Object-Oriented Programming in C](https://www.state-machine.com/doc/AN_OOP_in_C.pdf), by Miro Samek, walks through using [encapsulation](https://embeddedartistry.com/fieldmanual-terms/encapsulation/), inheritance, and polymorphism in C. [Companion code](https://sourceforge.net/projects/qpc/) is also available.
- [Object-oriented Techniques in C](https://dmitryfrank.com/articles/oop_in_c) by Dmitry Frank provides an example implementation of polymorphism
- Nathan Jones has a take on [Simple Inheritance](https://github.com/nathancharlesjones/Comparison-of-OOP-techniques-in-C/tree/main/2_Simple-inheritance) in his [Comparison of OOP Techniques in C](https://github.com/nathancharlesjones/Comparison-of-OOP-techniques-in-C/) project. The rest of the project focuses on different polymorphism techniques that can be applied beyond what is described here.
- One of the most detailed guides for implementing objects in C is [From C Function Pointers to C++ Objects](https://www.cs.bham.ac.uk/~hxt/2014/c-plus-plus/objects-in-c.pdf) by Hayo Thielecke. This paper will walk you through the creation of objects using structures and function pointers.
- [Event-based Interfaces for Testability](http://www.electronvector.com/blog/event-based-interfaces-for-testability) shows a practical use of inheritance using event-oriented design
- [How-To Implement Subtype Polymorphism in C](https://blog.melard.fr/post/32822011020/how-to-implement-subtype-polymorphism-in-c) provides another example