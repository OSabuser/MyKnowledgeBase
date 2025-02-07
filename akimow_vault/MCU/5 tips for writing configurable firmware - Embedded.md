---
title: "5 tips for writing configurable firmware - Embedded"
source: "https://www.embedded.com/5-tips-for-writing-configurable-firmware/"
author:
  - "[[Stephen Evanczuk]]"
published: 2023-04-27
created: 2025-02-07
description: "Configurable firmware might increase complexity and starting costs in some cases, but configurability offers flexibility and scalability needed to meet"
tags:
  - "clippings"
---
Configurable firmware might increase complexity and starting costs in some cases, but configurability offers flexibility and scalability needed to meet customer challenges in the future.

One strategy to rapidly deploy new embedded products into the marketplace is to leverage platforms. You or company probably have a product roadmap that identifies the products that will be released over the next several years. With customers wanting slightly different features, configurations, and customizations, it is not practical to develop one off products. Instead, if you create a product platform with common core software that can be extensible and configurable, you can dramatically decrease costs and development time. Let’s explore five tips for writing configurable firmware that I think will dramatically improve your software.

**Tip #1 – Start with a top-down approach**

Embedded software teams often struggle to write configurable software because they think about software from a bottoms-up approach. Approaching software from the hardware up puts the hardware, not the application or the user at the designs center. The result is often tightly coupled code and has little to no configuration built in. That is exactly what we don’t want.

Changing the approach to think about embedded software from the top down can change everything. The developers will be forced to think about how their customers will use the product. Very often, looking at a product through different customers eyes will immediately identify where configuration is needed. One customer might need motor speed setting A, while another needs motor setting B. On motor stalled fault one customer may want the LED to come on solid, while the other wants it to blink at 5 Hz.

Starting at the top, with the customer(s), and working down to the hardware will improve the configurability of your software.

**Tip #2 – Leverage configuration files**

If you look outside the firmware and microcontroller industry, you’ll find that software developers working on applications use configuration files to adapt their software. A configuration file(s) can help a developer to dictate how they want their application to behave under different configuration settings. Obviously, a configuration file provides a level of customization to the software that improves reuse and may decrease overall costs.

Firmware developers might include some configuration parameters in non-volatile memory to allow customizations; however, these customizations only affect the run-time behavior of the system. What about configurations that completely change the behavior of the core code?

An interesting technique that I often use to improve configurability of my embedded software is to use configuration files that autogenerate code for me. For example, I have a Python toolkit that I wrote that will read in a yaml file with thread configuration information and generate C/C++ code that is compiled into my applications. If a customer requires a custom feature, a new thread can be added to the configuration and automatically included in the build. The idea can be used beyond threads to apply to product specific information such as number of motors, relays, hardware that is present or absent, and so forth.

![An example architecture for using configuration and templates fed into a generator that produces source code based on the configuration settings. (Image Source: Embedded Software Design page 314).](https://www.embedded.com/embedded/wp-content/uploads/sites/2/2023/04/2304jb2_f1.png)  
*Figure 1: An example architecture for using configuration and templates fed into a generator that produces source code based on the configuration settings. (Image Source: Embedded Software Design, page 314).*

**Tip #3 – Create configuration tables in your code**

A technique that I’ve used for most of my career to write configurable code is to create and use configuration tables. A configuration table is usually defined as an array of a structure. For example, if I want to create a configuration table for a digital input/output peripheral, I might define a structure that looks something like the following code, depending on the supported features:

```
typedef struct
{
   DioChannel_t Channel;
   uint8_t Resistor;    
   uint8_t DriveStrength;
   uint8_t PassiveFilter;
   uint8_t Direction;   
   uint8_t State;       
   uint8_t Function;    
}DioConfig_t;
```

The above structure defines the characteristics of the dio pin that would typically be used to initialize it. We can then create an array of the structure with the configuration information for each dio pin something like the following:

```
static Dio_Config_t const Dio_Config[] =
{
// Name,    Resister, Drive, Filer,   Dir,   State,  Function
  {PORTA_1, DISABLED, HIGH, DISABLED, OUTPUT, HIGH, FCN_GPIO},
  {PORTA_2, DISABLED, HIGH, DISABLED, OUTPUT, HIGH, FCN_GPIO},
  {SWD_DIO, PULLUP,   LOW,  DISABLED, OUTPUT, HIGH, FCN_MUX7},
  {PORTA_4, DISABLED, HIGH, DISABLED, OUTPUT, HIGH, FCN_GPIO},
  {PORTA_5, DISABLED, HIGH, DISABLED, INPUT,  HIGH, FCN_GPIO},
  {PORTA_6, DISABLED, HIGH, DISABLED, OUTPUT, HIGH, FCN_GPIO},
  {PORTA_7, DISABLED, HIGH, DISABLED, OUTPUT, HIGH, FCN_GPIO},
  {PORTA_8, DISABLED, HIGH, DISABLED, OUTPUT, HIGH, FCN_GPIO},
};
```

You’ll love these configuration tables for several reasons:

- They are human readable which is great for code reviews
- They can be generated by a script reading a yaml, json, etc file
- They can be manually edited by a human if needed
- Initialization code is simplified by looping over the array
- It’s portable, reusable and scalable
- They can be used for hardware and software configuration

**Tip #4 – Use abstraction layers**

One problem that you might have noticed with a lot of firmware is that it is tightly coupled to the hardware. Firmware developers often tie their application code directly to the hardware which makes porting and reusing the firmware difficult. As you probably learned during the chip shortage, if you need to reconfigure your application to work with a new chip, it can be a nightmare to go back and redo all that code. You need to be able to easily configure your application to work with any hardware.

One solution is to use an abstraction layer. The abstraction layer creates an interface for accessing hardware that drivers must adhere to. The application code calls the abstraction. The idea is known as the dependency inversion principle, which is part of the SOLID principles for object-oriented design. By using an interface between the hardware and the application code, you invert the dependencies to break the hardware/application dependency and make the software and hardware more configurable.

**Tip #5 – Polymorphism is your friend**

If you are a C programmer, you might think that polymorphism is a bad word or something that doesn’t exist in C. On both accounts, you’d be incorrect. Polymorphism is a powerful technique that allows us to configure our application behavior at compile time (static polymorphism) or run-time (dynamic polymorphism). Polymorphism allows a single interface to represent different types of objects.

For example, let’s say that you have a product that blinks an LED. An LED could be connected to hardware through a dio port, to a PWM channel, to an external I/O expander on SPI or I2C, and so on. Depending on the version of hardware or customer requirements, might change how the LED is connected. While this seems like an annoying configuration problem, a developer could use polymorphism to write application code that doesn’t care how the LED is connected. Polymorphism can improve configurability in addition reusability and flexibility.

**Conclusions**

Embedded products today are no longer one-off products that are manufactured for years to come. Innovation and changes in technology are exponential, with teams needing to develop platform code that can be reused to launch many products in the years ahead. To meet this need, you must embrace configurability in firmware. In some instances, configurable firmware will increase complexity and starting costs, and perhaps even be less efficient from a memory and performance perspective. However, configurability will give businesses developing embedded products the flexibility and scalability to meet their customers challenges into the future. 
