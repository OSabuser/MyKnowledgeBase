---
title: "A Practical Guide to Dynamic Polymorphism in C Programming | HackerNoon"
source: "https://hackernoon.com/a-practical-guide-to-dynamic-polymorphism-in-c-programming"
author:
  - "[[Nikola Savic]]"
published: 2024-05-26
created: 2025-02-18
description: "Explore how dynamic polymorphism enhances code flexibility and maintainability in C programming"
tags:
  - "clippings"
---
**Dynamic polymorphism** is a programming paradigm that enhances code flexibility and maintainability by allowing objects to be treated uniformly while exhibiting different behaviors. While C is not inherently object-oriented like C++ or Java, dynamic polymorphism can be implemented using **function pointers** and **structures**. In this article, we compare multiple implementations of a toy simulation in C to demonstrate the benefits of dynamic polymorphism.

## Initial Implementation without Polymorphism

Here we define a `Toy` struct with a `name` field and create three instances representing Barbie and Superman toys. Each toy is initialized, and an array of pointers to these toy instances is created. The `main` function iterates through the array and prints the sound associated with each toy based on its name.

```clike
#include <stdio.h>
#include <stdlib.h>

// Function prototypes for the sounds made by different toys
char const* barbieSound(void){return "Love and imagination can change the world.";}
char const* supermanSound(void){return "Up, up, and away!";}

// Struct definition for Toy with an character array representing the name of the toy
typedef struct Toy{
    char const* name;  
}Toy;

int main(void){
    //Allocate memory for three Toy instances
    Toy* toy1 = (Toy*)malloc(sizeof(Toy));
    Toy* toy2 = (Toy*)malloc(sizeof(Toy));
    Toy* toy3 = (Toy*)malloc(sizeof(Toy));

    // Initialize the name members of the Toy instances
    toy1->name = "Barbie";
    toy2->name = "Barbie";
    toy3->name = "Superman";
    
    // Create an array of pointers to the Toy instances
    Toy* toys[] = {toy1, toy2, toy3};
    
    // Output the corresponding sound of each toy given its name
    for(int i=0; i < 3; i++){
        if (toys[i]->name == "Barbie"){printf("%s\n",toys[i]->name,barbieSound());}
        if (toys[i]->name == "Superman"){printf("%s\n",toys[i]->name,supermanSound());}
    }

    // Free the allocated memory for the Toy instances
    free(toy1);
    free(toy2);
    free(toy3);
    
    return 0;
}
```

While this is functional, it doesn't scale. Whenever we want to add a new toy we need to update the code to handle a new toy type and its sound function which could raise maintenance issues.

## Enhanced Implementation of Dynamic Polymorphism:

The second code sample uses dynamic polymorphism for a more flexible, scalable application. Here `Toy` has its function pointer for the sound function. Factory functions(`createBarbie()`, `createSuperMan()`) are used to create Barbie and Superman instances, assigning the appropriate sound function to each toy. The `makeSound()` function demonstrates dynamic polymorphism by calling the appropriate sound function for each toy at run time.

```cpp
#include <stdio.h>
#include <stdlib.h>

// Function prototypes for the sounds made by different toys
char const* barbieSound(void){return "Love and imagination can change the world.";}
char const* supermanSound(void){return "Up, up, and away!";}

// Struct definition for Toy with a function pointer for the sound function
typedef struct Toy {
    char const* (*makeSound)();
} Toy;

// Function to call the sound function of a Toy and print the result
// Demonstrates dynamic polymorphism by calling the appropriate function for each toy
void makeSound(Toy* self) {
    printf("%s\n", self->makeSound());
}

// Function to create a Superman toy
// Uses dynamic polymorphism by assigning the appropriate sound function to the function pointer
Toy* createSuperman() {
    Toy* superman = (Toy*)malloc(sizeof(Toy));
    superman->makeSound = supermanSound; // Assigns Superman's sound function
    return superman;
}

// Function to create a Barbie toy
// Uses dynamic polymorphism by assigning the appropriate sound function to the function pointer
Toy* createBarbie() {
    Toy* barbie = (Toy*)malloc(sizeof(Toy)); 
    barbie->makeSound = barbieSound; // Assigns Barbie's sound function
    return barbie;
}

int main(void) {
    // Create toy instances using factory functions
    Toy* barbie1 = createBarbie();
    Toy* barbie2 = createBarbie();
    Toy* superman1 = createSuperman();
    
    // Array of toy pointers
    Toy* toys[] = { barbie1, barbie2, superman1 };
    
    // Loop through the toys and make them sound
    // Dynamic polymorphism allows us to treat all toys uniformly
    // without needing to know their specific types
    for (int i = 0; i < 3; i++) {
        makeSound(toys[i]);
    }
    
    // Free allocated memory
    free(barbie1);
    free(barbie2);
    free(superman1);

    return 0;
}
```

By using a function pointer within the `Toy` struct, the code can easily accommodate new toy types without modifying the core logic.

**Factory function** **is a** **design pattern** used to create objects without specifying the exact class or type of the object that will be created. In the context of C programming and especially in embedded software, a factory function helps in managing the creation and initialization of various types of objects (e.g., sensors, peripherals) in a modular and flexible manner. This pattern is particularly useful when dealing with dynamic polymorphism, as it abstracts the instantiation logic and allows the system to treat objects uniformly

**A function pointer** is a variable that stores the address of a function, allowing the function to be called through the pointer. This enables dynamic function calls, which are particularly useful when the function to be executed needs to be determined at runtime.

#### Defining a Function Pointer

A function pointer is defined using the following syntax:

```clike
<return type> (*<pointer name>)(<parameter types>);
```

#### Example

For example, to declare a pointer to a function that returns an `int` and takes two `int` parameters, you would write:

```clike
int (*functionPointer)(int, int);
```

### Performance Considerations

- **Memory Overhead**: Each structure that uses function pointers requires additional memory to store these pointers. This can slightly increase your program's memory footprint, especially if many such structures are used.
- **Function Call Overhead**: Indirect function calls through pointers can introduce a slight performance overhead compared to direct function calls. This is due to the additional level of indirection required to resolve the function pointer at runtime.

### How the Overheads Occur

When a function is called directly, the assembly code typically contains a direct jump to the function's address. For example, let’s take a simple C program:

```clike
#include <stdio.h>

void function(){
    printf("Hello Barbie");
}

int main(void){
    function();
    return 0; 
}
```

The assembly code of the C code compiled with an x86-64 GCC 14.1 compiler looks like this:

```
.LC0:
        .string "Hello Barbie"
function:
        push    rbp
        mov     rbp, rsp
        mov     edi, OFFSET FLAT:.LC0
        mov     eax, 0
        call    printf
        nop
        pop     rbp
        ret
main:
        push    rbp        # Save the base pointer of the previous stack frame by pushing it on the stack
        mov     rbp, rsp   # Set up a new stack frame for the current function
        mov     eax, 0     # Clear the eax register
        call    function   # Call the function 'function'
        mov     eax, 0     # Set eax register to 0, indicating the return value of the main function
        pop     rbp        # Pop the saved value from stack into rbp, restoring the previous stack frame
        ret                # Return from main function
```

We can see that in this case, calling the function takes only two instructions in the assembly code (`call function`, `mov eax, 0`).

If we replace function with a function pointer called `myFunction` to point to the function:

```clike
int main(void){
    void (*myFunction)() = function; // function pointer called myFunction to point to function
    myFunction();                    // calling the function pointed by myFunction pointer
    return 0; 
}
```

The assembly code of the main function will look like this:

```
main:    
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16      # Allocate 16 bytes on the stack for local variables 
        mov     QWORD PTR [rbp-8], OFFSET FLAT:function  # Store the address of function into the memory location [rbp-8]
        mov     rdx, QWORD PTR [rbp-8]                   # Load the function pointer stored at [rbp-8] into rdx register
        mov     eax, 0                                   # Clear the eax register before the function call
        call    rdx                                      # Call the function stored in rdx
        mov     eax, 0                                   # Set the return value of main to 0
        leave                                            # Release the stack space used by the current stack frame
        ret
```

We can see that when using a function pointer, we have one additional assembly instruction for storing the function address in the function pointer and three additional instructions for calling the function pointed to by the function pointer. Also, 16 bytes had to be allocated for storing local variables, while only 8 bytes of the stack memory were used by the function pointer. This is because the stack size typically needs to be a multiple of 16, so if we were to use 20 bytes in our function, the compiler would allocate 32 bytes for the stack frame.

## Virtual tables

Let's take for example a scenario where each Toy, instead of having one sound, has multiple sounds, each represented by a separate function:

```clike
char const* barbieSound1(void) {return "Love and imagination can change the world.";}
char const* barbieSound2(void) {return "You can be anything you want to be.";}
char const* barbieSound3(void) {return "Every girl can make a difference.";}
char const* barbieSound4(void) {return "Dream big and dare to fail.";}

char const* supermanSound1(void) {return "Up, up, and away!";}
char const* supermanSound2(void) {return "There is a superhero in all of us, we just need the courage to put on the cape.";}
char const* supermanSound3(void) {return "Dreams save us. Dreams lift us up and transform us.";}
```

We could try to refactor our Toy structure to have multiple `makeSound` functions:

```clike
typedef struct Toy {
    char const* (*makeSound1)();
    char const* (*makeSound2)();
    char const* (*makeSound3)();
    char const* (*makeSound4)();
} Toy;

Toy* createBarbie() {
    Toy* barbie = (Toy*)malloc(sizeof(Toy)); 
    barbie->makeSound1 = barbieSound1;  
    barbie->makeSound2 = barbieSound2;  
    barbie->makeSound3 = barbieSound3;  
    barbie->makeSound4 = barbieSound4;  
    return barbie;
}
Toy* createSuperman() {
    Toy* superman = (Toy*)malloc(sizeof(Toy)); 
    barbie->makeSound1 = supermanSound1;  
    superman->makeSound2 = supermanSound2;  
    superman->makeSound3 = supermanSound3;  
    superman->makeSound4 = NULL; // supermanSound4 is not defined
    return superman;
}
```

As we can see, as the number of functions increases, managing function assignments and function calling becomes difficult and error-prone.

One possible solution to that problem are virtual tables.

**Virtual tables** (vtables) provide a way to store function pointers in a table, allowing dynamic function calls at runtime. Each structure has a pointer to the vtable, and the vtable contains pointers to the functions that implement the ‘structure’s methods’, in our case the toy structure’s different makeSound functions.

```clike
#include <stdio.h>
#include <stdlib.h>

// Barbie sound functions
char const* barbieSound1(void) { return "Love and imagination can change the world."; }
char const* barbieSound2(void) { return "You can be anything you want to be."; }
char const* barbieSound3(void) { return "Every girl can make a difference."; }
char const* barbieSound4(void) { return "Dream big and dare to fail."; }

// Superman sound functions
char const* supermanSound1(void) { return "Up, up, and away!"; }
char const* supermanSound2(void) { return "There is a superhero in all of us, we just need the courage to put on the cape."; }
char const* supermanSound3(void) { return "Dreams save us. Dreams lift us up and transform us."; }

// Define a type for function pointers
typedef char const* (*PTRFUN)();

// Vtables for Barbie and Superman
PTRFUN barbieVTable[4] = { barbieSound1, barbieSound2, barbieSound3, barbieSound4 };
PTRFUN supermanVTable[3] = { supermanSound1, supermanSound2, supermanSound3 };

// Toy structure with a vtable pointer
typedef struct Toy {
    PTRFUN* vtable;
} Toy;

// Create a Barbie toy and set its vtable
Toy* createBarbie() {
    Toy* barbie = (Toy*)malloc(sizeof(Toy)); 
    barbie->vtable = barbieVTable;
    return barbie;
}

// Create a Superman toy and set its vtable
Toy* createSuperman() {
    Toy* superman = (Toy*)malloc(sizeof(Toy)); 
    superman->vtable = supermanVTable;
    return superman;
}

// Function to make a toy produce a sound
void makeSound(Toy* self, int num) {
    // Check if the index is within bounds of the vtable
    if (self->vtable[num] != NULL) {
        printf("%s\n", self->vtable[num]());
    } else {
        printf("Invalid sound index.\n");
    }
}

int main(void) {
    // Create Barbie and Superman toys
    Toy* barbie = createBarbie();
    Toy* superman = createSuperman();

    // Make sounds using the vtable
    makeSound(barbie, 0);  // Outputs: "Love and imagination can change the world."
    makeSound(barbie, 1);  // Outputs: "You can be anything you want to be."
    makeSound(superman, 0); // Outputs: "Up, up, and away!"
    makeSound(superman, 1); // Outputs: "There is a superhero in all of us, we just need the courage to put on the cape."

    // Clean up
    free(barbie);
    free(superman);

    return 0;
}
```

Virtual tables simplify the management of multiple function pointers, reduce redundancy, and make it easier to extend the system with new behaviors.

To show how much the vtable plays a significant role, let’s look at an example where we want to use 10,000 instances of each toy in our program:

```cpp
#include <stdio.h>

typedef void (*FUNPTR)();

struct Superman
{
    FUNPTR fptr1;
    FUNPTR fptr2;
    FUNPTR fptr3;
    FUNPTR fptr4;
    FUNPTR fptr5;
    FUNPTR fptr6;
    FUNPTR fptr7;
    FUNPTR fptr8;
    FUNPTR fptr9;
    FUNPTR fptr10;
}superman;

FUNPTR * barbieVTable[10];

struct Barbie
{
    FUNPTR * vtable;
}barbie;

int main(void){
    printf("Size sf Superman struct: %ld bytes.\n", sizeof(superman));
    printf("Size of Barbie struct: %ld bytes, size of barbieVTable: %ld.\n", sizeof(barbie), sizeof(barbieVTable));
    printf("Alocated memory for 10,000 Superman toys: %ld bytes.\n", 10000 * sizeof(superman));
    printf("Alocated memory for 10,000 Barbie toys: %ld bytes.\n", 10000 * sizeof(barbie) + sizeof(barbieVTable));
    return 0;
}
```
```bash
#expected output:
Size sf Superman struct: 80 bytes.
Size of Barbie struct: 8 bytes, size of barbieVTable: 80.
Alocated memory for 10,000 Superman toys: 800000 bytes.
Alocated memory for 10,000 Barbie toys: 80080 bytes.
```

**Explanation:**

- **struct Superman**:
- This struct has ten function pointers (`FUNPTR`), directly embedded within the struct. Each function pointer typically takes up 8 bytes (on a 64-bit system), resulting in a total size of 80 bytes for each instance of `Superman`.
- **struct Barbie**:
- This struct has a single pointer to a vtable (`barbieVTable`), which contains ten function pointers. The size of the `Barbie` struct is 8 bytes (the size of a single pointer), and the vtable itself is an array of ten function pointers each of size 8 bytes, resulting in 80 bytes total for the vtable.

**Memory Allocation:**

- **For 10,000 Superman instances**:
- Each `Superman` instance is 80 bytes.
- Total memory required = 10,000 \* 80 bytes = 800,000 bytes.
- **For 10,000 Barbie instances**:
- Each `Barbie` instance is 8 bytes.
- Additionally, only one shared vtable of 80 bytes is required.
- Total memory required = (10,000 \* 8 bytes) + 80 bytes = 80,000 + 80 bytes = 80,080 bytes.

**Conclusion:**

This demonstration shows how using a vtable can reduce memory usage when creating multiple instances of objects that share the same set of functions.

## Benefits of Dynamic Polymorphism

- **Flexibility**: Allows new types to be added with minimal changes to existing code.
- **Maintainability**: Reduces the need for extensive conditional logic.
- **Scalability**: Supports the extension of the system by adding new object types and behaviors.

### Practical Use Cases

Dynamic polymorphism is often used in various software design patterns, such as the Strategy Pattern, State Pattern, and Command Pattern. These patterns use polymorphism to allow different behaviors and states to be easily swapped in and out. This makes the code more flexible and easier to maintain. If you have any questions or thoughts on this topic, feel free to leave a comment below!