---
title: "5 tips for designing a smart watchdog - EDN"
source: "https://www.edn.com/5-tips-for-designing-a-smart-watchdog/"
author:
  - "[[Jacob Beningo]]"
published: 2015-07-14
created: 2025-02-25
description: "Recovering from a system failure or a software glitch can be no easy task. The longer the fault exists the harder it can be to identify and recover. An"
tags:
  - "clippings"
---
Recovering from a system failure or a software glitch can be no easy task. The longer the fault exists the harder it can be to identify and recover. An external watchdog can help systems react quickly to such failures.

The external watchdog is an important and critical tool in the embedded systems engineer toolbox. They must be designed properly, however, in order to catch system failures and not cause them. There are five tips that should be taken into account when designing a watchdog system.

**Tip #1 – Monitor a heartbeat**  
The simplest function that an external watchdog can have is to monitor a periodic “heartbeat” signal that the primary application processor produces, and to generate an error signal when the heartbeat fails. Monitoring of the heartbeat should serve two distinct purposes. First, the microcontroller should only generate the heartbeat after functional checks have been performed on the software to ensure that the software is functioning. Second, the heartbeat should be able to reveal if the real-time response of the system has been jeopardized.

Monitoring the heartbeat for software functionality and real-time response can be done using a simple, “dumb” external watchdog. The external watchdog should have the capability to assign a heartbeat period along with a window that the heartbeat must appear within. The purpose of the heartbeat window is to allow the watchdog to detect when the real-time response of the system has been compromised. In the event that either functional or real-time checks fail the watchdog then attempts to recover the system through a reset of the application processor.

**Tip #2 – Use a low capability MCU**  
Simple timer-based external watchdogs that can monitor a heartbeat are relatively low cost but can severely limit the capabilities and recovery possibilities of the watchdog system. A low capability microcontroller can cost nearly the same amount as an external watchdog timer, so why not add some intelligence to the watchdog and use a microcontroller? The microcontroller firmware can be developed to fulfill the windowed heartbeat monitoring with the addition of so much more.

A “smart” watchdog like this is sometimes referred to as a supervisor or safety watchdog and has actually been used for many years in different industries such as automotive. Generally a microcontroller-based watchdog has been used primarily in safety critical applications. Given the development tools now available and the low cost of the hardware, though, such designs can be cost effective in other applications as well.

![](https://www.edn.com/wp-content/uploads/media-1229599-esc-15-logo.jpg)Attend [ESC Boston 2015](http://www.embeddedconf.com/boston/events/?-mc=we-edn-le-escsv-01) for a live session with Jacob on squeezing more battery life out of an ARM Cortex-M. Register for the event [here](http://www.embeddedconf.com/boston/?-mc=we-edn-le-escsv-01).

**Tip #3 – Supervise critical system functions**  
The decision to use a small microcontroller as a watchdog opens nearly endless possibilities of how the watchdog can be used. One of the first roles added to a smart watchdog is usually to supervise critical system functions such as a system current or sensor state. One example of how a watchdog could supervise a current would be to take an independent measurement of the current and then provide that value to the application processor. The application processor could then compare its own reading to that of the watchdog. If there were disagreement between the two then the system would execute a fault tree that was deemed to be appropriate for the application.

**Tip #4 – Observe a communication channel**  
Sometimes an embedded system can appear to the watchdog and the application processor to be operating as expected, but to an external observer it is in a non-responsive state. In such cases it can be useful to tie the smart watchdog to a communication channel such as a UART. When the watchdog is connected to a communication channel it can not only monitor channel traffic but can receive commands that are specific to the watchdog.

A great example of this is a watchdog designed for a small satellite that monitors radio communications between the flight computer and ground station. If the flight computer becomes non-responsive to the radio, a command could be sent to the watchdog that is then executed and used to reset the flight computer.

**Tip #5 – Consider an externally timed reset function**  
Using a microcontroller to implement a watchdog with extra features adds some complexity and a new software element to the system design. Thus, the question of who is watching the watchdog in such systems is undoubtedly on the minds of many engineers. In the event that the watchdog itself goes off into the weeds how is the watchdog going to recover?

One option would be to use the dumb external watchdog timer that was discussed earlier. The smart watchdog would generate a heartbeat to keep itself from being reset by the dumb watchdog timer. Another option would be to have the application processor act as the watchdog for the watchdog. Careful thought needs to be given to the best way to ensure both processors remain functioning as intended.

**Conclusion**  
The purpose of the smart watchdog is to monitor the system and the primary microcontroller to ensure that they operate as expected. During the design of a system watchdog it can be very tempting to allow the number of features the watchdog supports to creep upward. Developers need to keep in mind that as the complexity of the smart watchdog increases so does the probability that the watchdog itself will contain potential failure modes and bugs. Keeping the watchdog simple and to the minimum necessary feature set will ensure that it can be exhaustively tested and proven to work.

*Jacob Beningo is a Certified Software Development Professional (CSDP) whose expertise is in embedded software. He works with companies to decrease costs and time to market while maintaining a quality and robust product. Feel free to contact him at jacob@beningo.com, at his website www.beningo.com, and sign-up for his monthly Embedded Bytes Newsletter [here](http://www.beningo.com/814-2/).*

---

*![](https://www.edn.com/wp-content/uploads/media-1225059-esc-15-logo.jpg)*

*Join over 2,000 technical professionals and embedded systems hardware, software, and firmware developers at [ESC Silicon Valley](http://www.embeddedconf.com/silicon-valley/?-mc=we-edn-le-escsv-01) July 20-22, 2015 and learn about the latest techniques and tips for reducing time, cost, and complexity in the embedded development process.*

[Passes](http://www.embeddedconf.com/silicon-valley/attend/?-mc=we-edn-le-escsv-01) for the [ESC Silicon Valley 2015 Technical Conference](http://www.embeddedconf.com/silicon-valley/conference/?-mc=we-edn-le-escsv-01) are available at the conference’s official site with discounted advance pricing until July 17, 2015. The Embedded Systems Conference and EDN are owned by UBM Canon.