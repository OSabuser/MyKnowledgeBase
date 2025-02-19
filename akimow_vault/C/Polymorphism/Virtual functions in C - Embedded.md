---
title: "Virtual functions in C - Embedded"
source: "https://www.embedded.com/virtual-functions-in-c/"
author:
  - "[[Dan Saks]]"
published: 2012-08-08
created: 2025-02-18
description: "Although C doesn’t provide native support for virtual functions, you can emulate virtual functions in C if you attend to all the details. Last month, I"
tags:
  - "clippings"
---
**Although C doesn’t provide native support for virtual functions, you can emulate virtual functions in C if you attend to all the details.**

Last month, I explained how C++ compilers typically implement virtual functions by illustrating how using virtual functions affects the storage layout for objects.(Saks, Dan. [“Storage layout for polymorphic objects,”](http://www.eetimes.com/discussion/programming-pointers/4390835) Embedded.com, July 21, 2012. ) This month, I’ll continue by showing how to implement virtual functions in C in a way that generates machine code very similar to what you get from C++.

As before, my sample classes represent an assortment of two-dimensional geometric shapes such as `circle` , `rectangle` , and `triangle` , all derived from a common base class called `shape` . In C++, the definition for the base class `shape` looks like:

```cpp
class shape {public:    shape();                
	// constructor    
	virtual double area() const;    
	virtual double perimeter() const;
	private:    
	coordinates position;    
	color outline, fill;
};
```

The `area` and `perimeter` member functions are virtual. A class, such as `shape` , with a least one virtual function is a *polymorphic type* . C++ compilers typically add a hidden pointer to each polymorphic type. That pointer is commonly called a *vptr* and it points to a table of function pointers called a *vtbl* .

You can implement a polymorphic `shape` type in C using the following declarations:

```cpp
// shape.h - base class for shapes

#ifndef SHAPE_H_INCLUDED
#define SHAPE_H_INCLUDED
typedef struct shape shape;
typedef struct shape_vtbl shape_vtbl;
struct shape_vtbl {    
	double (*area)(shape const *s);    
	double (*perimeter)(shape const *s);
};
struct shape {    
	shape_vtbl *vptr;   
	coordinates position;    
	color outline, 
	fill;
};

#endif
```

A class derived from a polymorphic base class will be polymorphic as well, and it inherits the base class’s `vptr` . The `vptr` must have the same offset in the base class subobject (the base class portion) of a derived class object as it does in a base class object.

In C++, the definition for a `circle` class derived from `shape` looks like:

```cpp
class circle: public shape {
public:    circle(double r);       
// constructor    
virtual double area() const;    
virtual double perimeter() const;
private:    double radius;
};
```

In C, the declarations for a polymorphic `circle` type look like:

```cpp
// circle.h – circle derived from shape
#ifndef CIRCLE_H_INCLUDED
#define CIRCLE_H_INCLUDED
#include "shape.h"
typedef struct circle circle;
struct circle {    
	shape base;     // the base class sub-object    
	double radius;
};
void circle_construct(circle *c, double r);
#endif
```

The `base` member of the `circle` structure above includes all the members inherited from the `shape` base class, including `vptr` . The `circle_construct` function initializes a `circle` , including its `vptr` . I’ll cover the details of initializing the `vptr` in an upcoming column.

As I showed last month, the C++ definition for a `rectangle` class derived from `shape` looks a lot like the definition for `circle` :

```cpp
class rectangle: public shape {
	public:    rectangle(double h, double w);    
	virtual double area() const;    
	virtual double perimeter() const;
	private:    double height, width;
};
```

Similarly, the C declarations for a polymorphic `rectangle` type look a lot like the declarations for `circle` :

```cpp
// rectangle.h - rectangle interface
#ifndef RECTANGLE_H_INCLUDED
#define RECTANGLE_H_INCLUDED
#include "shape.h"
typedef struct rectangle rectangle;
struct rectangle {    
	shape base;     // the base class sub-object    
	double height, width;
};
void rectangle_construct(rectangle *t, double h, double w);
#endif
```

In C++, a call to the virtual `area` function applied to a `shape` looks exactly like a non-virtual call, as in:

```cpp
shape *s;
s->area();
```

If `s` points to a `circle` (the dynamic type of `*s` is `circle` ), then the call above calls `circle::area` . If `s` points to a `rectangle` , then the call above calls `rectangle::area` .

C doesn’t provide explicit support for classes with member functions. In C, you simply use ordinary functions to emulate member functions. For example:

```cpp
circle_area(&c);
```

applies the `circle_area` function to `circle c` . You get the same runtime performance as a C++ member function call, but without any compile-time checking to enforce access control.

In C, virtual function calls look unlike any other kind of function call. For example, a call to the virtual `area` function applied to a `shape` looks like:

```cpp
shape *s;
s->vptr->area(s);
```

In this case, if `s` points to a `circle` (the dynamic type of `*s` is `circle` ), then the call above calls `circle_area` . If `s` points to a `rectangle` , then the call above calls `rectangle_area` .

As I hinted earlier, this works only if you initialize the `vptr` properly, something I’ll cover in an upcoming column.

***Dan Saks** is president of Saks & Associates, a C/C++ training and consulting company. For more information about Dan Saks, visit his website at [www.dansaks.com](http://www.dansaks.com/). Dan also welcomes your feedback: e-mail him at .  
*