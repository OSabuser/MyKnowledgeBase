---
title: "5 Tips for using callbacks with interrupts - EDN"
source: "https://www.edn.com/5-tips-for-using-callbacks-with-interrupts/"
author:
  - "[[Jacob Beningo]]"
published: 2015-11-23
created: 2025-02-25
description: "Callbacks are references to executable code that higher levels of software pass into a function. These callbacks have the ability to greatly increase the"
tags:
  - "clippings"
---
Callbacks are references to executable code that higher levels of software pass into a function. These callbacks have the ability to greatly increase the portability and reuse of embedded software, but by their very definition they require the use of function pointers and can be extremely dangerous if not used carefully. Here are five tips for creating and using callbacks safely in an embedded system, particularly for using them with interrupts.

Tip #1 – **Create a set or register method** — Callbacks are most often seen as function pointers being passed into a function but they can also be used in a portable system to set the function that will be called by an interrupt service routine (ISR). When an interrupt fires, the interrupt handler will contain a reference to the function that should be executed. In order to set the called function in a reusable manner, it can be helpful to create an interface that allows the callback function to be registered with the module. Creating a register or set function allows the ISR handler to be generic, encapsulated, and even be compiled as part of a peripheral library. The executable function for the interrupt will get set at run-time.

Tip #2 – **Initialize the callback to be NULL or default** — Creating a peripheral driver interface that contains a callback register or set function is a great step towards creating portable interrupts. But there is one problem: what happens if a callback never gets registered yet the interrupt becomes enabled and then fires? The interrupt service routine needs some way to tell if a callback function has been registered. The easiest way to provide such verification is to first initialize the callback function pointer to NULL. A simple check against NULL in the ISR will then prevent any handler from executing. An alternative to using NULL would be to initialize the pointer to a generic and empty interrupt handler. When an uninitialized interrupt callback gets fired then the default handler will run.

Tip #3 – **Verify callback before use** — Setting the initial value of the callback pointer to NULL or to a generic handler allows the interrupt to validate that an interrupt handler has been set. When using function pointers, though, it is always a good idea to first verify that the referenced location in memory is not NULL and does exist within the system. Function pointers can be very dangerous and care should be taken to ensure that the pointed to location is valid before making the call.

Tip #4 – **Use callbacks to add features to lower levels of firmware** — Callback functions are very useful for adding features to lower level drivers and application code in a generic and application-specific manner. In order to produce firmware that is loosely coupled and has high cohesion, developers can pass interrupt vectors and references to other peripheral modules into a driver through the use of a callback. The callback allows the developer to add application specific features at design time without having to continually modify driver code to get the desired behavior. Callbacks can be used in this way to produce very clean, portable and reusable firmware and interfaces.

Tip #5 – **Create abstract callback functions** — Callback functions often need to take an unknown number of parameters and they may or may not also return data, depending on the application needs. Yet the developer's goal is to write callback functions that are abstract. For callbacks related to interrupts, where the callback is just being called as part of the interrupt service routine, the function can take a void parameter and return void. Any data that needs to be shared with the application would be handled the same way it always is for interrupts. For callbacks into the driver that might relate to error handling or other custom features, the easiest way to handle parameters and return values is to pass a pointer and return a pointer.

**Final Thoughts**

Callbacks can find a wide array of uses within an embedded system, especially when it comes to developing portable and reusable firmware. This article has provided five examples of how a developer might think about and implement callbacks for interrupts, but they are just a few of many.

*Jacob Beningo is a Certified Software Development Professional (CSDP) whose expertise is in embedded software. He works with companies to decrease costs and time to market while maintaining a quality and robust product. Feel free to contact him at jacob@beningo.com, at his website www.beningo.com, and sign-up for his monthly Embedded Bytes Newsletter [here](http://www.beningo.com/814-2/).*