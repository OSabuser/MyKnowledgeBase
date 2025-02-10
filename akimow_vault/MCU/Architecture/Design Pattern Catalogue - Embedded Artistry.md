---
title: "Design Pattern Catalogue - Embedded Artistry"
source: "https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/"
author:
  - "[[Embedded Artistry]]"
published: 2019-10-25
created: 2025-02-07
description: "25 October 2019 by Phillip Johnston • Last updated 9 September 2024Design patterns are valuable concepts to add to your mental library. These are non-obvious solutions to scenarios and problems frequently encountered in the programming world. Exposure to patterns shows you how many of your problems have been previously solved by others, freeing you to … Continue reading \"Design Pattern Catalogue\""
tags:
  - "clippings"
---
[25 October 2019](https://embeddedartistry.com/blog/2019/10/25/) • Last updated 9 September 2024

Design patterns are valuable concepts to add to your mental library. These are *non-obvious* solutions to scenarios and problems frequently encountered in the programming world.

Exposure to patterns shows you how many of your problems have been previously solved by others, freeing you to focus your limited time and energy reserves on those problems which are truly novel for your product. You will also develop a mental library of patterns that you can work toward in refactoring efforts.

**Table of Contents:**

- [Architecture](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Architecture)
- [General Software](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#General-Software)
- [Embedded Software](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Embedded-Software)
- [Distributed Systems](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Distributed-Systems)
- [Security](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Security)
- [Safety](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Safety)
- [Data Integrity](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Data-Integrity)
- [Testing](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Testing)
- [Cloud Computing](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Cloud-Computing)
- [Field Atlas Entries](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Field-Atlas-Entries)

## Architecture

[Architectural patterns](https://en.wikipedia.org/wiki/Architectural_pattern) are general, reusable solutions to software architecture problems. They have broader scope than software [design patterns](https://embeddedartistry.com/fieldmanual-terms/design-pattern/) and are typically intended to resolve software engineering issues.

General information about architectural patterns:

- [Wikipedia: List of Software Architecture Styles and Patterns](https://en.wikipedia.org/wiki/List_of_software_architecture_styles_and_patterns)
- *[Pattern-Oriented Software Architecture: A System of Patterns](https://amzn.to/2l4W7y2)* by Frank Buschmann et al.
- *[Software Architecture Patterns](https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/)* by Mark Richards

Common architectural patterns for embedded systems include:

- [Layered Architecture](https://en.wikipedia.org/wiki/Abstraction_layer), which organizes the various software components into n-*tiers* or *layers*, each with a specific role
- Extremely common architectural pattern, especially for embedded systems
- Embedded layers might consist of: [HAL](https://embeddedartistry.com/fieldmanual-terms/hardware-abstraction-layer/)/[BSP](https://embeddedartistry.com/fieldmanual-terms/board-support-package/), Drivers/Middleware, Business Logic
- [Layered Architecture](https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/ch01.html) by Mark Richards
- [Presentation Domain Data Layering](https://martinfowler.com/bliki/PresentationDomainDataLayering.html) by Martin Fowler
- [Layers, Hexagons, Features, and Components](http://www.codingthearchitecture.com/2016/04/25/layers_hexagons_features_and_components.html) by Simon Brown
- [Hexagonal Architecture](https://en.wikipedia.org/wiki/Hexagonal_architecture_\(software\)) (also known as Ports & Adapters), which involves creating loosely coupled components that can be connected to each other through ports and adapters
- [Hexagonal != Layers](http://tpierrain.blogspot.com/2016/04/hexagonal-layers.html) by Thomas Pierrain
- [The Pattern: Ports and Adapters](https://web.archive.org/web/20180408231827/http://alistair.cockburn.us/Hexagonal+architecture) by Alistair Cockburn
- [Layers, Hexagons, Features, and Components](http://www.codingthearchitecture.com/2016/04/25/layers_hexagons_features_and_components.html) by Simon Brown
- [Event Driven Architecture](https://en.wikipedia.org/wiki/Event-driven_architecture), which involves building a system around the production, detection, consumption, and reaction to events.
- [Software Architecture Patterns: Event Driven Architecture](https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/ch02.html) by Mark Richards
- [Microkernel](https://en.wikipedia.org/wiki/Microkernel)
- [Publish-subscribe](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern), which decouples components that publish data on a given topic from the components which want to receive that data
- [Domain-Specific Language](https://en.wikipedia.org/wiki/Domain-specific_language) (DSL), which involves the creation of a language or language dialect suitable for a specific problem domain
- [Pipes and Filters](https://en.wikipedia.org/wiki/Pipeline_\(software\)), which consists of components (filters) that transform or filter data and pass it to other components by way of connectors (pipes), with potential buffering between stages

## General Software

**Subsections:**

1. [General Software Design Patterns](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#General-Software-Design-Patterns)
2. [Decoupling Patterns](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Decoupling-Patterns)
3. [Asynchronous and Event-Driven Patterns](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Asynchronous-and-Event-Driven-Patterns)
4. [Patterns for Displacing Legacy Systems](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Patterns-for-Displacing-Legacy-Systems)
5. [Multi-threading Patterns](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Multi-threading-Patterns)
6. [Refactoring Patterns](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Refactoring-Patterns)

### General Software Design Patterns

For generally useful software architecture patterns, see:

- *[Design Patterns: Elements of Reusable Object-Oriented Software](https://amzn.to/2l6aZfB)* by Erich Gamma et al.
- [Small Memory Software](http://smallmemory.charlesweir.com/), by Charles Wier and James Noble, provides patterns for systems with limited memory

Generally useful programming patterns:

- [Look-up Table](https://embeddedartistry.com/fieldmanual-terms/look-up-table/)

### Decoupling Patterns

- [Pointer Array Pattern](https://embeddedartistry.com/fieldmanual-terms/pointer-array-pattern/) – Write generic drivers by accessing registers through a table lookup.
- [Callback Function](https://embeddedartistry.com/fieldmanual-terms/callback-function/) – Provide a reference to a function (or function-like object) that will be invoked when a specific condition holds true.
- [Facade Pattern](https://embeddedartistry.com/fieldmanual-terms/facade-pattern/) – Provide a unified interface to a set of interfaces in a subsystem. [Facade](https://embeddedartistry.com/fieldmanual-terms/facade-pattern/) defines a higher-level interface that makes the subsystem easier to use.
- [Managing Complexity with the Mediator and Facade Patterns](https://embeddedartistry.com/blog/2020/07/06/managing-complexity-with-the-mediator-and-facade-patterns/) provides descriptions and examples of those patterns
- [Mediator Pattern](https://embeddedartistry.com/fieldmanual-terms/mediator-pattern/) – Define an object that encapsulates how a set of objects interact. [Mediator](https://embeddedartistry.com/fieldmanual-terms/mediator-pattern/) promotes loose [coupling](https://embeddedartistry.com/fieldmanual-terms/coupling/) by keeping objects from referring to each other explicitly, and it lets you vary their interaction independently.
- [Main Pattern](https://embeddedartistry.com/fieldmanual-terms/main-pattern/) – decouple modules (and keep as many as possible independent of the underlying platform) by making the application responsible for the connections between and configuration of each module. We consider this a variation of the Mediator pattern.
- [Managing Complexity with the Mediator and Facade Patterns](https://embeddedartistry.com/blog/2020/07/06/managing-complexity-with-the-mediator-and-facade-patterns/) provides descriptions and examples of those patterns
- [Template Method Pattern](https://embeddedartistry.com/fieldmanual-terms/template-method-pattern/) – Defines the skeleton of an algorithm or operation in high-level steps. Users or subclasses can override or implement the behavior of specific steps within the algorithm, but are not able to modify the general algorithm flow itself.
- [Generation Gap](https://embeddedartistry.com/fieldmanual-terms/generation-gap-pattern/) – Separate generated code from non-generated code through the [Template Method pattern](https://embeddedartistry.com/fieldmanual-terms/template-method-pattern/). This ensures that customizing the generated code does not require users to modify generated code.
- [Non-Virtual Interface](https://embeddedartistry.com/fieldmanual-terms/non-virtual-interface-pattern/) – controls how methods in a base class are overridden. Base classes use public, non-virtual functions that can be called by clients. Overridable methods are defined as `protected`, `virtual` members.
- [Adapter](https://embeddedartistry.com/fieldmanual-terms/adapter-pattern/) – The [adapter pattern](https://embeddedartistry.com/fieldmanual-terms/adapter-pattern/) allows the interface of an existing module to be used as/with another interface. This is done by adding in a thin “adapter” module that maps the desired interface onto the existing interface.
- [Configuration Table Pattern](https://embeddedartistry.com/fieldmanual-terms/configuration-table-pattern/) – store configuration and initialization information inside of a table, and pass the table to an initialization routine that iterates over the table entries.

### Asynchronous and Event-Driven Patterns

- [Active Object](https://embeddedartistry.com/fieldmanual-terms/active-object/) – decouples method execution from method invocation for objects that each reside in their own thread of control. Typically, an active object is constructed using an internal thread and a queue of operations or events that will be executed on the active object’s thread.
- [Message Passing](https://embeddedartistry.com/fieldmanual-terms/message-passing/) – invoke behavior by sending messages to communicate what you want done (in contrast to the default approach of directly invoking functions provided by a module/object).
- [Message Queue](https://embeddedartistry.com/fieldmanual-terms/messsage-queue/) – components used for communication in software systems that represent a queue of messages or events that are awaiting processing.
- [Event Loop](https://embeddedartistry.com/fieldmanual-terms/event-loop/) – waits for and processes (or dispatches) events or messages in a program.

### Patterns for Displacing Legacy Systems

- [Patterns of Legacy Displacement](https://martinfowler.com/articles/patterns-legacy-displacement/), by Ian Cartwright, Rob Horn, and James Lewis, is a collection of patterns that can be used to modernize legacy systems. The primary link contains a general discussion on updating legacy systems, as well as an example for removing a Middleware
- [Critical Aggregator](https://martinfowler.com/articles/patterns-legacy-displacement/critical-aggregator.html) – Combine data from different parts of the business to support making critical decisions
- [Divert the Flow](https://martinfowler.com/articles/patterns-legacy-displacement/divert-the-flow.html) – First divert cross-organization activities away from legacy
- [Extract Product Lines](https://martinfowler.com/articles/patterns-legacy-displacement/extract-product-lines.html) – Identify and separate systems by product line.
- [Feature Parity](https://martinfowler.com/articles/patterns-legacy-displacement/feature-parity.html) – Replicate existing functionality of a legacy system using a new technology stack.
- [Legacy Mimic](https://martinfowler.com/articles/patterns-legacy-displacement/legacy-mimic.html) – New system interacts with legacy system in such a way that the old system is not aware of any changes.
- [Transitional Architecture](https://martinfowler.com/articles/patterns-legacy-displacement/transitional-architecture.html) – Software elements installed to ease the displacement of a legacy system that we intend to remove when the displacement is complete.

### Multi-threading Patterns

- Singleton
- [Using Singletons in Embedded C++](https://blog.stratifylabs.dev/device/2021-11-29-Using-Singletons-in-embedded-cpp/) by Stratify Labs
- - [Multi-threaded Singleton Access in embedded C++](https://blog.stratifylabs.dev/device/2022-01-27-Multithread-Singleton-Access-in-embedded-cpp-copy/) by Stratify Labs

### Refactoring Patterns

- Martin Fowler’s [Refactoring](https://refactoring.com/) website documents a number of refactoring patterns. However, the patterns are fully documented in the [corresponding book](https://amzn.to/3OeuX0N).
- [*Refactoring to Patterns*](https://amzn.to/3KLCSAs) by Joshua Kerievsky provides a pattern-directed refactoring approach.

## Embedded Software

For a general discussion of embedded software patterns, see:

- *[Practical UML Statecharts in C/C++: Event-Driven Programming for Embedded Systems](https://amzn.to/2l66Rw7)*, by Miro Samek, which provides a selection of [state machine](https://embeddedartistry.com/fieldmanual-terms/finite-state-machine/)patterns for event-driven embedded systems
- [Free Download](https://www.state-machine.com/psicc2/%3E%5B)\]([https://sourceforge.net/projects/qpc/files/doc/PSiCC2.pdf/download](https://sourceforge.net/projects/qpc/files/doc/PSiCC2.pdf/download))
- [Small Memory Software](http://smallmemory.charlesweir.com/), by Charles Wier and James Noble, provides patterns for systems with limited memory
- [The full book is available for free online](http://smallmemory.charlesweir.com/book.html)
- [Summary of Patterns](http://smallmemory.charlesweir.com/PatternSummaries.pdf)
- [Thinking Small: The Processes for Creating Small Memory Software](http://smallmemory.charlesweir.com/ThinkingSmall.pdf), by James Noble and Charles Wier

**Subsections:**

- [General](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#General)
- [Watchdog Timers](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Watchdog-Timers)
- [Hardware Interfaces](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Hardware-Interfaces)
- [State Machines](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#State-Machines)
- [Memory Allocation](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Memory-Allocation)
- [Managing Limited Memory](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Managing-Limited-Memory)
- [Small Data Structures](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Small-Data-Structures)
- [Compression](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Compression)
- [Secondary Storage](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Secondary-Storage)
- [Fail-Safe Data Storage](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Fail-Safe-Data-Storage)
- [User Interfaces](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#User-Interfaces)

### General

- [Monitor-Actuator Pair Design Pattern](https://betterembsw.blogspot.com/2014/04/monitor-actuator-pair-design-pattern.html)
- How can we avoid a single-point of failure in a safety-critical component?
- Use two independent hardware components operating as a “monitor-actuator” CPU pair. The actuator CPU is the component that actually performs the computation or control function. An independent monitor Chip is used to detect when the actuator has failed and mitigate the failure (e.g., by resetting the actuator or triggering an emergency stop)
- [Digital twin](https://en.wikipedia.org/wiki/Digital_twin) – a virtual representation that serves as the real-time digital counterpart of a physical object or process.
- [Desired state vs. actual state](https://blog.golioth.io/better-iot-design-patterns-desired-state-vs-actual-state/) – an application of a “digital twin” to a connected embedded device

### Watchdog Timers

For general watchdog timer usage patterns:

- [Proper Watchdog Timer Use](https://betterembsw.blogspot.com/2014/05/proper-watchdog-timer-use.html), by Phil Koopman

### Hardware Interfaces

- [Protection Class: A Solution for Resolving I2C Communication Issues in Complex Systems](https://www.irnas.eu/protection-class-a-solution-resolving-i2c-communication-issues-in-complex-systems/) – describes a pattern for handling intermittent [I2C](https://embeddedartistry.com/fieldmanual-terms/inter-ic-communication/) communication failures “under the hood” without alerting the higher application layers.
- [High Speed Serial Port Design Pattern](http://www.eventhelix.com/RealtimeMantra/PatternCatalog/high_speed_serial_port.htm) – describes how to interface with high-speed serial communication devices that require the use of [DMA](https://embeddedartistry.com/fieldmanual-terms/direct-memory-access/). The main objective is to encapsulate the interface with the device and provide a hardware-independent interface to the high-speed serial port.

### State Machines

- [Ultimate Hook](https://www.state-machine.com/doc/Pattern_UltimateHook.pdf)
- How can you provide a consistent policy for handling events, while allowing clients to override every aspect of the default behavior?
- Hierarchical state nesting – a composite state can define the default behavior (the common look and feel) and supply an “outer shell” for nesting client substates
- [Reminder](https://www.state-machine.com/doc/Pattern_Reminder.pdf)
- How can you prevent loosely related functions of a system from becoming tightly coupled by a common event?
- Consider, for example, periodic data acquisition, in which a sensor producing the data needs to be polled at a predetermined rate
- Assume that a periodic `TIMEOUT` event is dispatched to the system at the desired rate to provide the stimulus for polling the sensor.
- Because the system has only one external event (the `TIMEOUT` event), it seems that this event needs to trigger both the polling of the sensor and the processing of the data
- We don’t want that
- Invent a stimulus to propagate a reminder that data is ready for processing
- Moreover, you can use state nesting to arrange these two functions in a hierarchical relation, which gives you even more control over the behavior
- [Deferred Event](https://www.state-machine.com/doc/Pattern_DeferredEvent.pdf)
- How can we defer an event (which can be postponed within a certain limit) that is received at an inconvenient moment until the system is finished processing the current activity?
- Defer the new request and handle it at a more convenient time, which effectively leads to altering the sequence of events presented to the [state machine](https://embeddedartistry.com/fieldmanual-terms/finite-state-machine/)
- [Orthogonal Component](https://www.state-machine.com/doc/Pattern_Orthogonal.pdf)
- How can we model objects consisting of relatively independent parts with their own state behavior *without* using orthogonal reasons?
- Use object [composition](https://embeddedartistry.com/fieldmanual-terms/whole-part-relationship/). Concurrency virtually always arises within objects by [aggregation](https://embeddedartistry.com/fieldmanual-terms/whole-part-relationship/); that is, multiple states of the components can contribute to a single state of the composite object.
- [Transition to History](https://www.state-machine.com/doc/Pattern_History.pdf)
- How can we handle a high-priority event during a state transition in a composite state, and then return to the most recent substate of the composite state after processing completes?
- Store the most recently active leaf substate as a dedicated “history” data member, and use that member during a “transition to history” staet
- [State-Local Storage](https://www.state-machine.com/doc/Pattern_SLS.pdf)
- How can we reduce the runtime memory requirements for complex state machines, where we have multiple complex states that don’t require use of all of the extended state variables needed by other states?
- Allow state machine designer to reduce the memory footprint of a state machine by providing variables local to states. As the state machine transitions from one state to another, the SLS mechanism automatically overlays the extended-state variables for the target state configuration on top of the no-longer needed variables for the source state configuration. This results in a lower memory footprint.

### Memory Allocation

- How do you allocate memory to store your data structures?
- Choose the simplest allocation technique that meets your need.
- [Small Memory Software Chapter: Memory Allocation](http://smallmemory.charlesweir.com/6_AllocationChapter.pdf)
- [Fixed Allocation](http://smallmemory.charlesweir.com/6_AllocationChapter.pdf)
- How can you ensure you will never run out of memory?
- Pre-allocate objects during initialization.
- [Variable Allocation](http://smallmemory.charlesweir.com/6_AllocationChapter.pdf)
- How can you avoid unused empty space?
- Allocate and deallocate variable-sized objects as and when you need them.
- [Memory Discard](http://smallmemory.charlesweir.com/6_AllocationChapter.pdf)
- How can you allocate temporary objects?
- Allocate objects from a temporary workspace and discard it on completion.
- [Pooled Allocation](http://smallmemory.charlesweir.com/6_AllocationChapter.pdf)
- How can you allocate a large number of similar objects?
- Pre-allocate a pool of objects, and recycle unused objects.
- [Compaction](http://smallmemory.charlesweir.com/6_AllocationChapter.pdf)
- How do you recover memory lost to fragmentation?
- Move objects in memory to remove unused space between them.
- [Reference Counting](http://smallmemory.charlesweir.com/6_AllocationChapter.pdf)
- How do you know when to delete a [shared object](https://embeddedartistry.com/fieldmanual-terms/shared-library/)?
- Keep a count of the references to each [shared object](https://embeddedartistry.com/fieldmanual-terms/shared-library/), and delete each object when its count is zero.
- [Garbage Collection](http://smallmemory.charlesweir.com/6_AllocationChapter.pdf)
- How do you know when to delete shared objects?
- Identify unreferenced objects, and de-allocate them.

### Managing Limited Memory

- How can you manage memory use across a whole system?
- Make every component responsible for its own memory use.
- [Small Memory Software Chapter: Small Architecture](http://smallmemory.charlesweir.com/2_ArchitectureChapter.pdf)
- [Memory Limit](http://smallmemory.charlesweir.com/2_ArchitectureChapter.pdf):
- How can you share out memory between multiple competing components?
- Set limits for each component and fail allocations that exceed the limits.
- [Small Interfaces](http://smallmemory.charlesweir.com/2_ArchitectureChapter.pdf)
- How can you reduce the memory overheads of component interfaces?
- Design interfaces so that clients control data transfer.
- [Captain Oates](http://smallmemory.charlesweir.com/2_ArchitectureChapter.pdf)
- How can you fulfill the most important demands for memory?
- Sacrifice memory used by less vital components rather than fail more important tasks.
- [Read-Only Memory](http://smallmemory.charlesweir.com/2_ArchitectureChapter.pdf)
- What can you do with read-only code and data?
- Store read-only code and data in read-only memory.
- [Hooks](http://smallmemory.charlesweir.com/2_ArchitectureChapter.pdf)
- How can you change information in read-only storage?
- Access read-only information through hooks in writable storage and change the hooks to give the illusion of changing the information.

### Small Data Structures

- How can you reduce the memory needed for your data?
- Choose the smallest structure that supports the operations you need.
- [Small Memory Software Chapter: Small Data Structures](http://smallmemory.charlesweir.com/5_DataStructuresChapter.pdf) \*[Packed Data](http://smallmemory.charlesweir.com/5_DataStructuresChapter.pdf)
- How can you reduce the memory needed to store a data structure?
- Pack data items within the structure so that they occupy the minimum space. \*[Sharing](http://smallmemory.charlesweir.com/5_DataStructuresChapter.pdf)
- How can you avoid multiple copies of the same information?
- Store the information once, and share it everywhere it is needed. \*[Copy-on-Write](http://smallmemory.charlesweir.com/5_DataStructuresChapter.pdf)
- How can you change a shared object without affecting its other clients?
- Share the object until you need to change it, then copy it and use the copy in future. \*[Embedded Pointer](http://smallmemory.charlesweir.com/5_DataStructuresChapter.pdf)
- How can you reduce the space used by a collection of objects?
- Embed the pointers maintaining the collection into each object. \*[Multiple Representations](http://smallmemory.charlesweir.com/5_DataStructuresChapter.pdf)
- How can you support several different implementations of an object?
- Make each implementation satisfy a common interface.

### Compression

- How can you fit a quart of data into a pint pot of memory?
- Use a compressed representation to reduce the memory required.
- [Small Memory Software Chapter: Compression](http://smallmemory.charlesweir.com/4_CompressionChapter.pdf)
- [Table Compression](http://smallmemory.charlesweir.com/4_CompressionChapter.pdf)
- How do you compress many short strings?
- Encode each element in a variable number of bits so that the more common elements require fewer bits.
- [Difference Coding](http://smallmemory.charlesweir.com/4_CompressionChapter.pdf)
- How can you reduce the memory used by sequences of data?
- Represent sequences according to the differences between each item.
- [Adaptive Compression](http://smallmemory.charlesweir.com/4_CompressionChapter.pdf)
- How can you reduce the memory needed to store a large amount of bulk data?
- Use an adaptive compression algorithm.

### Secondary Storage

- What can you do when you have run out of primary storage?
- Use secondary storage as extra memory at runtime.
- [Small Memory Software Chapter: Secondary Storage](http://smallmemory.charlesweir.com/3_SecondaryStorageChapter.pdf)
- [Application Switching](http://smallmemory.charlesweir.com/3_SecondaryStorageChapter.pdf)
- How can you reduce the memory requirements of a system that provides many different functions?
- Split your system into independent executables, and run only one at a time.
- [Data File](http://smallmemory.charlesweir.com/3_SecondaryStorageChapter.pdf)
- What can you do when your data doesn’t fit into main memory?
- Process the data a little at a time and keep the rest on secondary storage.
- [Resource Files](http://smallmemory.charlesweir.com/3_SecondaryStorageChapter.pdf)
- How can you manage lots of configuration data?
- Keep configuration data on secondary storage, and load and discard each item as necessary.
- [Packages](http://smallmemory.charlesweir.com/3_SecondaryStorageChapter.pdf)
- How can you manage a large program with lots of optional pieces?
- Split the program into packages, and load each package only when it’s needed.
- [Paging](http://smallmemory.charlesweir.com/3_SecondaryStorageChapter.pdf)
- How can you provide the illusion of infinite memory?
- Keep a system’s code and data on secondary storage, and move them to and from main memory as required.

### Fail-Safe Data Storage

- [How to Protect Non-Volatile Data](https://barrgroup.com/Embedded-Systems/How-To/Protect-Non-Volatile-Data) by Niall Murphy
- [Ensuring fail-safe data storage in battery-powered IoT sensor nodes](https://www.embedded.com/ensuring-fail-safe-data-storage-in-battery-powered-iot-sensor-nodes/) by Nilesh Badodekar and Girija Chougala

### User Interfaces

- [User Involvement: Techniques for Handling Memory Constraints in the UI](http://smallmemory.charlesweir.com/UserInvolvement.pdf), by Charlies Wier and James Noble

## Distributed Systems

- [Distributed Embedded Scheduling](https://users.ece.cmu.edu/~koopman/lectures/ece649/16_scheduling.pdf)
- [Distributed Time](https://users.ece.cmu.edu/~koopman/lectures/ece649/17_embedded_internet_security.pdf)
- [Circuit Breaker Pattern](https://lab.scub.net/architecture-patterns-the-circuit-breaker-8f79280771f1)

> The Circuit Breaker pattern is designed to detect failures and encapsulates the logic of preventing a system from executing an operation that’s set to fail. Instead of repeatedly making requests to a service that is likely unavailable or facing issues, the circuit breaker stops all attempts for a while, giving the troubled service time to recover.
- [Patterns of Distributed Systems](https://martinfowler.com/articles/patterns-of-distributed-systems/), a project by Unmesh Joshi, documents a number of patterns. The primary page also describes general problem situations where these patterns can help, as well as how patterns can be sequenced together.
- [Clock-Bound Wait](https://martinfowler.com/articles/patterns-of-distributed-systems/clock-bound.html) – Wait to cover the uncertainty in time across cluster nodes before reading and writing values so values can be correctly ordered across cluster nodes.
- [Consistent Core](https://martinfowler.com/articles/patterns-of-distributed-systems/consistent-core.html) – Maintain a smaller cluster providing stronger consistency to allow large data cluster to coordinate server activities without implementing quorum based algorithms.
- [Emergent Leader](https://martinfowler.com/articles/patterns-of-distributed-systems/emergent-leader.html) – Order cluster nodes based on their age within the cluster to allow nodes to select a leader without running an explicit election.
- [Fixed Partitions](https://martinfowler.com/articles/patterns-of-distributed-systems/fixed-partitions.html) – Keep the number of partitions fixed to keep the mapping of data to the partition unchanged when size of a cluster changes.
- [Follower Reads](https://martinfowler.com/articles/patterns-of-distributed-systems/follower-reads.html) – Serve read requests from followers to achieve better throughput and lower latency
- [Generation Clock](https://martinfowler.com/articles/patterns-of-distributed-systems/generation.html)) – A monotonically increasing number indicating the generation of the server
- [Gossip Dissemination](https://martinfowler.com/articles/patterns-of-distributed-systems/gossip-dissemination.html) – Use random selection of nodes to pass on information to ensure it reaches all the nodes in the cluster without flooding the network
- [Heartbeat](https://martinfowler.com/articles/patterns-of-distributed-systems/heartbeat.html) – Show a server is available by periodically sending a message to all the other servers.
- [High-water Mark](https://martinfowler.com/articles/patterns-of-distributed-systems/high-watermark.html) – An index in the write ahead log showing the last successful replication.
- [Hybrid Clock](https://martinfowler.com/articles/patterns-of-distributed-systems/hybrid-clock.html) – Use a combination of system timestamp and logical timestamp to have versions as date-time, which can be ordered
- [Idempotent Receiver](https://martinfowler.com/articles/patterns-of-distributed-systems/idempotent-receiver.html) – Identify requests from clients uniquely so they can ignore duplicate requests when client retries
- [Key-Range Partitions](https://martinfowler.com/articles/patterns-of-distributed-systems/key-range-partitions.html) – Partition data in sorted key ranges to efficiently handle range queries.
- [Lamport Clock](https://martinfowler.com/articles/patterns-of-distributed-systems/lamport-clock.html) – Use logical timestamps as a version for a value to allow ordering of values across servers
- [Leader and Followers](https://martinfowler.com/articles/patterns-of-distributed-systems/leader-follower.html) – Have a single server to coordinate replication across a set of servers.
- [Lease](https://martinfowler.com/articles/patterns-of-distributed-systems/time-bound-lease.html) – Use time bound leases for cluster nodes to coordinate their activities.
- [Low-water Mark](https://martinfowler.com/articles/patterns-of-distributed-systems/low-watermark.html) – An index in the write ahead log showing which portion of the log can be discarded.
- [Paxos](https://martinfowler.com/articles/patterns-of-distributed-systems/paxos.html) – Use two consensus building phases to reach safe consensus even when nodes disconnect
- [Quorum](https://martinfowler.com/articles/patterns-of-distributed-systems/quorum.html) – Avoid two groups of servers making independent decisions, by requiring majority for taking every decision.
- [Replicated Log](https://martinfowler.com/articles/patterns-of-distributed-systems/replicated-log.html) – Keep the state of multiple nodes synchronized by using a write-ahead log that is replicated to all the cluster nodes.
- [Request Batch](https://martinfowler.com/articles/patterns-of-distributed-systems/request-batch.html) – Combine multiple requests to optimally utilise the network.
- [Request Pipeline](https://martinfowler.com/articles/patterns-of-distributed-systems/request-pipeline.html) – Improve latency by sending multiple requests on the connection without waiting for the response of the previous requests.
- [Request Waiting List](https://martinfowler.com/articles/patterns-of-distributed-systems/request-waiting-list.html) – Track client requests which require responses after the criteria to respond is met based on responses from other cluster nodes.
- [Revert to Source](https://martinfowler.com/articles/patterns-legacy-displacement/revert-to-source.html) – Identify the originating source of data and integrate to that
- [Segmented Log](https://martinfowler.com/articles/patterns-of-distributed-systems/log-segmentation.html) – Split log into multiple smaller files instead of a single large file for easier operations
- [Single Socket Channel](https://martinfowler.com/articles/patterns-of-distributed-systems/single-socket-channel.html) – Maintain order of the requests sent to a server by using a single TCP connection
- [Singular Update Queue](https://martinfowler.com/articles/patterns-of-distributed-systems/singular-update-queue.html) – Use a single thread to process requests asynchronously to maintain order without blocking the caller
- [State Watch](https://martinfowler.com/articles/patterns-of-distributed-systems/state-watch.html) – Notify clients when specific values change on the server
- [Two Phase Commit](https://martinfowler.com/articles/patterns-of-distributed-systems/two-phase-commit.html) – Update resources on multiple nodes in one atomic operation.
- [Versioned Value](https://martinfowler.com/articles/patterns-of-distributed-systems/versioned-value.html) – Store every update to a value with a new version, to allow reading historical values.
- [Version Vector](https://martinfowler.com/articles/patterns-of-distributed-systems/version-vector.html) – Maintain a list of counters, one per cluster node, to detect concurrent updates
- [Write-Ahead Log](https://martinfowler.com/articles/patterns-of-distributed-systems/wal.html) – Provide durability guarantee without the storage data structures to be flushed to disk, by persisting every state change as a command to the append only log

## Security

For general discussion of security patterns:

- [SEI: Secure Design Patterns](https://resources.sei.cmu.edu/asset_files/TechnicalReport/2009_005_001_15110.pdf)
- [Black Hat Briefings: Security Design Patterns](https://www.blackhat.com/presentations/bh-federal-03/bh-fed-03-peterson-up.pdf) (presentation slides)
- [ACM: Software-Security Patterns: Degree of Maturity](https://dl.acm.org/citation.cfm?id=2855364), by Michaela Bunke

Security anti-patterns:

- [Embedded Systems Security Pitfalls](https://www.youtube.com/playlist?list=PL_DQiOR0jhbU_Kt9UY_kArGVRJ73ymPb0), by Phil Koopman (lecture)
- [Slides](http://course.ece.cmu.edu/~ece642/lectures/40_securitypitfalls.pdf)
- [Top 16 Embedded Security Pitfalls](https://betterembsw.blogspot.com/2011/10/embedded-security-pitfalls.html) by Phil Koopman covers common pitfalls to avoid when designing your security plan.
- [Secrecy vs. Integrity and Why Encryption Might Be the Wrong Choice](https://betterembsw.blogspot.com/2013/10/secrecy-vs-integrity-and-why-encryption.html) by Phil Koopman shows that encryption doesn’t solve all of our security problems. In many cases, authentication and integrity are more important.
- [Security Pitfalls in Cryptography](https://www.schneier.com/essays/archives/1998/01/security_pitfalls_in.html) contains some lessons learned that you can apply to avoid creating a bad cryptographic system.

## Safety

For a general discussion of safety architectural patterns:

- [Safety Architecture Patterns](https://users.ece.cmu.edu/~koopman/lectures/ece642/35_safetyarchpatterns.pdf) by Phil Koopman
- [YouTube lecture](https://www.youtube.com/watch?v=JA5wdyOjoXg&list=PL_DQiOR0jhbUurEbjmy7IOhUeifZpCrFs)

**Subsections:**

- [Avoiding Single Points of Failure](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Avoiding-Single-Points-of-Failure)
- [Critical System Isolation](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Critical-System-Isolation)
- [Redundancy Management](https://embeddedartistry.com/fieldatlas/design-pattern-catalogue/#Redundancy-Management)

### Avoiding Single Points of Failure

For a general discussion on patterns for single points of failure:

- [Single Points of Failure](https://users.ece.cmu.edu/~koopman/lectures/ece642/31_singlepointfailures.pdf) by Phil Koopman ([YouTube lecture](https://www.youtube.com/watch?v=_t3uWa3Q4dc&list=PL_DQiOR0jhbX8ZZXpoLNH7feXMOXOcwnq))
- [Safety Requires No Single Points of Failure](https://betterembsw.blogspot.com/2014/03/safety-requires-no-single-points-of.html) by Phil Koopman
- [Self-Monitoring and Single Points of Failure](https://betterembsw.blogspot.com/2014/04/self-monitoring-and-single-points-of.html) by Phil Koopman

Patterns:

- [Monitor-Actuator Pair Design Pattern](https://betterembsw.blogspot.com/2014/04/monitor-actuator-pair-design-pattern.html)
- How can we avoid a single-point of failure in a safety-critical component?
- Use two independent hardware components operating as a “monitor-actuator” CPU pair. The actuator CPU is the component that actually performs the computation or control function. An independent monitor Chip is used to detect when the actuator has failed and mitigate the failure (e.g., by resetting the actuator or triggering an emergency stop)

### Critical System Isolation

For a general discussion on critical system isolation patterns:

- [Critical System Isolation](http://course.ece.cmu.edu/~ece642/lectures/32_sil_isolation.pdf) (slides) by Phil Koopman
- [YouTube lectures](https://www.youtube.com/watch?v=PX_ccAaSgx4&list=PL_DQiOR0jhbVt_isWkRlLz1281251oI_N)

### Redundancy Management

For a general discussion on redundancy management patterns:

- [Redundancy Management](http://course.ece.cmu.edu/~ece642/lectures/33_redundancymanagement.pdf) (slides)
- [YouTube lecture](https://www.youtube.com/watch?v=PaD9T9AILog&list=PL_DQiOR0jhbXAHi5OHoa3BI4ffApxy75S)
- [Redundancy Patterns Exercise](http://course.ece.cmu.edu/~ece642/weekly/week11.html#patterns_exer)
- [Redundant Input Processing for Safety](https://betterembsw.blogspot.com/2014/04/redundant-input-processing-for-safety.html), by Phil Koopman

## Data Integrity

For a general overview on data integrity patterns:

- [Data Integrity](http://course.ece.cmu.edu/~ece642/lectures/35_dataIntegrity.pdf) (slides only)
- [CRC Selection Exercise](http://course.ece.cmu.edu/~ece642/weekly/week11.html#crc_exer)

## Testing

For a general overview of testing patterns, see:

- *[xUnit Test Patterns](https://amzn.to/2N8k8jo)*, by Gerard Meszaros
- [xUnit Test Patterns](http://xunitpatterns.com/) (website)

Design for test patterns:

- [Dependency Injection](http://xunitpatterns.com/Dependency%20Injection.html)
- How do we design the system-under-test so that we can replace its dependencies at run time?
- The client provides the depended-on object to the system-under-test
- [Dependency Lookup](http://xunitpatterns.com/Dependency%20Lookup.html)
- How do we design the system-under-test so that we can replace its dependencies at run time?
- The system-under-test asks another object to return the depended-on object before it uses it
- [Humble Object](http://xunitpatterns.com/Humble%20Object.html)
- How can we test a component that is tightly coupled to a framework which we cannot easily integrate into our test environment?
- We extract all the logic from the hard-to-test component into a component that is testable via synchronous tests. As a result, the Humble Object component becomes a very thin [adapter](https://embeddedartistry.com/fieldmanual-terms/adapter-pattern/) layer that contains very little code. Each time the Humble Object is called by the framework, it delegates to the testable component.
- [Test Hook](http://xunitpatterns.com/Test%20Hook.html)
- How can we make code testable when it is coupled with hard-coded dependencies?
- We modify the behavior of the system-under-test or its dependencies to support testing by putting adding a functional hook or testing flag that can be changed during runtime

## Cloud Computing

For a general overview of cloud computing patterns, see:

- [Wikipedia: Cloud Computing](https://en.wikipedia.org/wiki/Cloud_computing), which discusses a variety of deployment, service, and architectural models

## Field Atlas Entries