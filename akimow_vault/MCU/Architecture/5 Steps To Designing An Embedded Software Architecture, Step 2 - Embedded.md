---
title: "5 Steps To Designing An Embedded Software Architecture, Step 2 - Embedded"
source: "https://www.embedded.com/5-steps-to-designing-an-embedded-software-architecture-step-2/"
author:
  - "[[Stephen Evanczuk]]"
published: 2022-09-28
created: 2025-02-10
description: "Embedded.com Explores The First Step In How To Design An Embedded Software Architecture - Identify & Trace Data Assets. Visit To Learn More."
tags:
  - "clippings"
---
In this continuation of the article series, we explore the second step to designing an embedded software architecture: identifying and tracing data assets.

Designing an [embedded software architecture](https://www.embedded.com/embedded/firmware-architecture-in-five-easy-steps/) is not a trivial endeavor that can be thought through in a single afternoon. A software architecture will also not miraculously emerge from just focusing on writing the code. There are five steps teams can use to develop and evolve their software architecture:

- [Separate the software architecture](https://www.embedded.com/embedded/5-steps-to-designing-an-embedded-software-architecture-step-1/)
- Identify and trace data assets
- Decompose the system
- Interface and component design
- Simulate, iterate, and scale

The last post discussed the [five steps to designing an embedded software architecture](https://www.embedded.com/embedded/5-steps-to-designing-an-embedded-software-architecture-step-1/).  We explored the benefits of viewing and embedded software systems as two architectures: a hardware-independent application business architecture and a hardware-dependent real-time architecture.

In today’s post, we will explore the second step to designing an embedded software architecture: identifying and tracing data assets.

## **Step #2 – Identify and trace data assets**

When I work with teams to architect their embedded software, I find that engineers have two tendencies; First, they want to get into the hardware from day one. Engineers act as if the low-level code interacting with the hardware is the final product, and there isn’t a moment to waste. Unfortunately, this is an antiquated view of embedded software development. In most systems, the real value is in the application code, that hardware-independent part of the architecture.

Second, engineers start throwing out architectural ideas around design patterns for interrupts, messaging, RTOS interactions, and so forth almost immediately. While it is great to brainstorm and get ideas out to be evaluated, how do you know what architectural concepts and design patterns match the design problems on day one? The answer is that you don’t!

Once a team has progressed through step one, separating the software architecture, the second step in designing an embedded software architecture is identifying and tracing data assets in the system. A data asset is any data used by the system to help it perform its function. For example, an embedded system might have data assets like:

- Encryption keys
- A unique identification number
- Pixel maps
- Controller states
- Etc

Many more data assets go into a system, but I think you understand. When we boil down everything, we do to design and build a real-time embedded system, the core of what we do is manage data.

## **The first principle of embedded software design**

The first principle for modern embedded software design is “Data Dictates Design.” We can create all kinds of exciting and unique architectures for accomplishing a given task. Still, in my experience, the most efficient architecture is the one that is designed around the system’s data assets. This is because all the extraneous junk that often finds its way into architecture because it’s the latest and greatest or because we think it is elegant gets thrown away.

When we focus on the data, the architecture can become hyper-focused on what it should be doing with the data. As it turns out, only a few operations can be done with data. First, a system can input data. For example, a user could press a button or receive serial data through a communication interface. Second, a system could output data. For example, display a pixel map to a display or drive a motor. Third, a system could process the data. For example, maybe serial data comes into the system in a packet format that is then decoded. Processing occurs to validate the packet and then unpack the stored data. Finally, a system can store data in either volatile or non-volatile memory.\[1\]

Identifying the data assets and the operations that can be performed on that data can dramatically help a team design its embedded software architecture. Data assets are like red flags waving at the architect that indicate an architectural need in the design. Unfortunately, too many teams ignore the data and instead chase modern design patterns before they understand the data problems they are trying to solve with their system.

## **What does a data-centric architecture mean?**

I know many embedded software developers will find the idea that “Data Dictates Design” to be a little weird. I find this surprising given that in object-oriented programming, we focus on creating objects which are collections of data with operations on that data. In all honesty, a data-centric view of architecture design isn’t new. However, viewing the architecture from a data standpoint has many advantages.

First, a data-centric approach to architecture meshes well with teams whose systems have security concerns. If you are interested in securing your embedded system, you must perform a threat model security analysis (TMSA). That analysis requires developers to identify various data assets in their system and determine the levels of protection they need. A TMSA needs to be performed before architecting the system, which means many details required for an architecture dictated by data will already be available. Two birds with one stone.

Second, identifying the data assets can help us to determine how the system might be broken down at the component level. Suppose we are being good architects and following the [single responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle) (SRP). In that case, there’s a good chance that each data asset we identity will be wrapped in its module with functions that operate on it.

Third, tracing how the various data assets interact can begin to hint at how the system might be architected at the application level. For example, suppose I can see data being input into the system and its high frequency compared to the rate at which it will be processed. In that case, I can start to bound the interaction with an input activity to receive the data and an activity to process the data. In the early architectural phases, I won’t select whether the high-frequency data inputs are received using interrupts, a buffer, or direct memory access (DMA). I would just bound the problem and push off the final decision until it is necessary. I would not immediately jump to say that the processing activity is an RTOS task. The activity might be a task or maybe something else. It’s too early to tell. While an engineer often strives to lock in as many details as possible as quickly as possible, a good software architect pushes off making decisions as long as possible to maximize flexibility.

### **Conclusions**

The second step in designing an embedded software architecture is identifying and tracing data assets in the system. Being hyper-focused on the data can seem strange to an embedded software engineer who is typically hardware focused. One way to help change our mindsets is to modify the definition of embedded software to:

“Embedded software is code designed and constructed to run deterministically, often with real-time deadlines that manage data through inputs, processing, outputs, and storage in various forms.”\[2\]

The great embedded software architecture allows data to dictate the design. Identifying data and then tracing how it interacts with other data in the system can help a software architect see how the architecture may emerge. The architectural picture often starts at a high, 30,000-foot view, with hints of where components and tasks might make sense. At this stage, though, we just want to identify the data assets and the operations they are involved in. The architectural picture will begin to emerge next time we look at the third step, decomposing the system.

**Notes**

\[1\] One could maybe add a fifth which would be to transfer data. However, transferring could be viewed as a processing operation, so I’ve left it out for this discussion.

\[2\] https://www.beningo.com/the-secret-embedded-software-definition-experts-use/

---

![](https://www.embedded.com/embedded/wp-content/uploads/sites/2/2022/05/EMB_Jacob_Beningo.jpg)***Jacob Beningo** is an embedded software consultant who specializes in real-time, microcontroller-based systems. He actively promotes software best practices through numerous articles, blogs, and webinars on topics from software architecture design, embedded DevOps, and implementation techniques. Jacob has 20 years of experience in the field and holds three degrees including a Masters of Engineering from the University of Michigan.*

---

**Related Contents:**

- [5 steps to designing an embedded software architecture, Step 1](https://www.embedded.com/embedded/5-steps-to-designing-an-embedded-software-architecture-step-1/)
- [5 embedded software architecture pitfalls](https://www.embedded.com/embedded/5-embedded-software-architecture-pitfalls/)
- [Fitting software architectures to security and safety demand](https://www.embedded.com/embedded/fitting-software-architectures-to-security-and-safety-demand/)
- [A decision-tree approach to picking a multicore software architecture](https://www.embedded.com/embedded/a-decision-tree-approach-to-picking-a-multicore-software-architecture/)

*For more Embedded, [subscribe to Embedded’s weekly email newsletter](https://aspencore.dragonforms.com/loading.do?omedasite=EmbeddedSubscribe).*