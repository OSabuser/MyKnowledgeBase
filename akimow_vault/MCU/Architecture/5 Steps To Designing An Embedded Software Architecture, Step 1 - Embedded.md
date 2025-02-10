---
title: "5 Steps To Designing An Embedded Software Architecture, Step 1 - Embedded"
source: "https://www.embedded.com/5-steps-to-designing-an-embedded-software-architecture-step-1/"
author:
  - "[[Jacob Beningo]]"
published: 2022-09-16
created: 2025-02-10
description: "Embedded.com Explores The First Step In How To Design An Embedded Software Architecture - Separate The Software Architecture. Visit To Learn More."
tags:
  - "clippings"
---
Some think that a clean software architecture can just emerge from coding efforts but that’s like believing that pouring a package of spaghetti into boiling water will result in cooked lasagna noodles.

Software architecture is the fundamental organization of a system embodied in its components, their relationship to each other and the environment, and the principles guiding its design and evolution \[1\]. Software architecture is not meant to be created once and set in stone. Instead, software architecture should evolve and change throughout a product’s lifetime. Over the years, I’ve heard engineers and managers discuss that software architecture should just emerge from coding efforts, as if by magic. Believing in an emergent, clean architecture is like believing that pouring a package of spaghetti into boiling water will result in cooked lasagna noodles.

A software architecture that is thought through and then evolves during implementation has many benefits, such as:

- Acting as a road map to what is being built
- Providing a software picture that can be used to train engineers and explain the software to management and stakeholders
- Minimizing unnecessary rework
- Decreased development costs

A common problem that I often see is that teams struggle to figure out how to define their software architecture. There are five steps teams can use to develop and evolve their software architecture:

- Separate the software architecture
- [Identify and trace data assets](https://www.embedded.com/embedded/5-steps-to-designing-an-embedded-software-architecture-step-2/)
- Decompose the system
- Interface and component design
- Simulate, iterate, and scale

So, let’s explore a few steps you can take to design your embedded software architecture.

## **Step #1 – Separate the software architecture**

Many embedded software teams look at their software architecture as a single cohesive architecture that includes both the application code and the hardware interactions. Viewing architecture this way is not surprising because embedded software engineers often look at their system from the hardware up. Embedded software is unique because it must interact with the hardware, which is different from all other software engineering fields. While this is true, a modern software architect will and should separate hardware-dependent and hardware-independent software, as shown in Figure 1.

![](https://www.embedded.com/embedded/wp-content/uploads/sites/2/2022/09/09jb_b1-f1.png)  
*Figure 1 – An embedded software architecture is separated into hardware-dependent and independent architectures that interact through an abstraction layer. (Image Source: Embedded Software Design \[2\]).*

I call this separation a “Tale of Two Architectures”. Traditionally, developers will design and implement their architecture so that the hardware’s independent and dependent layers are tightly coupled. But unfortunately, there are quite a few problems with a tightly coupled architecture.

## **Issues with Tightly Coupled Architectures**

First, they are not very portable. For example, what happens if a microcontroller suddenly becomes unavailable? (Chip shortage, anyone?). If the code is tightly coupled, attempting to move the application code to run on a new microcontroller becomes a herculean effort. Application code is tightly coupled to low-level hardware calls on the microcontroller! I know a lot of companies who have suffered through this recently. If they didn’t update their architecture, they had to go back through all their code and change every line that interacted with the hardware. The companies that updated their architecture broke their architecture coupling through abstractions and dependency injection.

Second, unit testing the application in a development environment rather than on the target hardware is nearly impossible. If the application code makes direct calls to the hardware, a lot of work will go into the test harness to successfully run that test, or the testing will need to be done on the hardware. Testing on hardware is slow and is often a manual rather than an automated process. The result that I’ve seen is that the software is not tested well, and the overall system quality suffers. In addition, it can take longer to deliver the software.

Finally, a tightly coupled architecture will have scalability issues. Tightly coupled systems often share data. As the software system tries to grow and add new features, it becomes more difficult with each new feature to add new code. Interactions between components, the ability to access shared data, and the chance for troublesome defects rise dramatically. As a result, feature development can come to a standstill despite developers working furiously to get the job done.

## **How separating the architecture solve the problem**

Separating the software architecture into hardware-dependent and independent architectures solves all the problems with a tightly coupled architecture. For example, creating an abstraction layer between the hardware-dependent and independent architectures allows the application code to be moved from one microcontroller to the next. The abstraction layer breaks the hardware dependencies; in other words, the application doesn’t know or care about the hardware. Instead, the application depends on the interface. New hardware-dependent components just need to meet the requirements for the interface. That means if we change hardware, only the hardware modules change! Not the entire code base.

Adding an abstraction layer between the two architectures also solves many problems with unit testing. With the abstraction layer in place, it’s easier to create test doubles and mocks that return expected and unexpected data to the application code. We can write all the application code without even having the hardware! I know this sounds absurd to many embedded software developers. However, over the last week, I’ve added several new features to a product I’m working on, and I turned the hardware on only once. All my development was done using unit tests on a host computer!

When we keep our architectures separate and focus on minimizing coupling, it becomes much easier to scale the software. However, just because we broke the architecture up doesn’t mean we can’t create coupling and cohesion issues in each architecture. We still need to ensure that we follow [SOLID](https://en.wikipedia.org/wiki/SOLID) principles in both architectures. The good news is that it allows us to focus on each architecture independently, which means real-time and hardware constraint issues don’t find their way into the core application logic.

The last benefit I want to mention is that by separating the hardware-dependent and independent architectures, we can focus on developing and delivering the application before the hardware is even available. The benefit here is that customers and management can be provided early access to the application and provide feedback. The ability to then iterate over the application and focus on ensuring it meets the real needs becomes much more manageable. Today, too many teams focus on getting the hardware ready first, and the core application is an afterthought. That’s no way to design and build a system.

### **Software Architecture Design Step #1 Conclusions**

Software architecture can help teams get control over their software. Emergent software architectures often result in a big-ball-of-mud and spaghetti code. That doesn’t mean that we’re forced to use a Waterfall methodology. Successful software architecture is often created through iteration and evolution. The first step to designing a software architecture is recognizing that embedded systems don’t have just one architecture. Instead, there are two architectures: hardware-dependent and independent architecture. Purposefully separating these architectures allows developers to leverage modern software techniques and methodologies to improve time to market, quality, and development costs.

**Part 2 of this series can be found here:** [5 Steps To Designing An Embedded Software Architecture, Step 2](https://www.embedded.com/embedded/5-steps-to-designing-an-embedded-software-architecture-step-2/)

**Notes**

\[1\] [https://en.wikipedia.org/wiki/IEEE\_1471](https://en.wikipedia.org/wiki/IEEE_1471). IEEE 1471 has been superseded by IEEE 42010. (I just like the IEEE 1471 definition).

\[2\] Beningo, Jacob. Embedded Software Design. Apress, 2022

---

**Related Contents:**

- [5 Steps To Designing An Embedded Software Architecture, Step 2](https://www.embedded.com/embedded/5-steps-to-designing-an-embedded-software-architecture-step-2/)
- [5 embedded software architecture pitfalls](https://www.embedded.com/embedded/5-embedded-software-architecture-pitfalls/)
- [Fitting software architectures to security and safety demand](https://www.embedded.com/embedded/fitting-software-architectures-to-security-and-safety-demand/)
- [A decision-tree approach to picking a multicore software architecture](https://www.embedded.com/embedded/a-decision-tree-approach-to-picking-a-multicore-software-architecture/)
- [Firmware Architecture In 5 Easy Steps](https://www.embedded.com/embedded/firmware-architecture-in-five-easy-steps/)

*For more Embedded, [subscribe to Embedded’s weekly email newsletter](https://aspencore.dragonforms.com/loading.do?omedasite=EmbeddedSubscribe).*