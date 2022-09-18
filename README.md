

http://web.mit.edu/tibbetts/Public/inside-c/www/

> **Note**
> Stored in a pointer means the direct value (address) stored in memory marked by the pointer variable. 
> Pointed to means things stored in heap starting at address stored in a pointer.


# Basics
## #Include Command
### Overview
Include is as simple as copy pasting what is included to the location where it appears. For example, including a <stdio.h> file at the beginning of program is equivalent of copying the whole stdio.h file to the first line. 

### <> and ""
<> files are system files that are already there for you. Include your own file with "". They basically search for different locations in the file system: <> will make the compiler search somewhere like std/bin whereas "" the current directory.

### Examples
```c
#include "completelyValidCProgram.txt"
```
This is our main.c and it's equavelent to this
```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    printf("hello world");
    return 0;
}
```
and to this
```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    #include "cSnippet.txt"
    return 0;
}
```
You can see the assembly file is exactly the same. We can also assume the parsing of #include happens very early during compilation (pre-compiling) since other compiling steps depends on a complete c program string.

## Using .h Header Files
### Overview


### Including .c Directly
```c
#include <stdio.h>
#include "sum.c"

int main(int argc, char const *argv[])
{
    #include "cSnippet.txt"
    printf("%d", sum(1, 2));
    return 0;
}
```
This is valid when you compile only this file since including sum.c just copy pastes the function on top of main function.

However, this won't work when compiling both files (gcc main2.c sum.c -o main3) since there will be 2 sum functions and linker will fail.

The standard process for using external source files is this:

```c
#include <stdio.h>
#include "sum.h"

int main(int argc, char const *argv[])
{
    printf("%d", sum(1, 2));
    return 0;
}
```
and compile with "gcc main3.c sum.c -o main3". sum.h tells the compiler that function sum exists. Two object files will be generated and non can execute alone. Linker will link these files together.

Note: it's possible that sum function isn't implemented at all and in this case it can compile but linker will fail.

## Data Types
https://en.wikipedia.org/wiki/C_data_types#Main_types
### Primitive
integer and float
- bool: 8 bit
- char: 8 bit
- short: 16 bit
- int: >= 16 bit (int on all 64 bit machine is 32 bit)
- long: >= 32 bit (long on 64 bit Mac is 64 bit)
- long long: >= 64 bit

### Address
- pointer
- struct
- enum
- union

## Printf & Scanf
### Format Specifier
- d: signed decimal integer
- ld: signed long decimal integer
- u: unsinged decimal integer
- o: unsigned octal integer
- x: unsinged hex integer
- c: char
- s: array of char, ended by \0
- f: float
- e: float in scientific notation
- p: pointer

### Other Control
1. You can add a number after % to control output width. e.g. %4d means this number will occupy at least 4 digits when printed.
2. %.2f means float with 2 digits

## Memory Layout

> **Note**
> This section uses examples using gcc compiler on x8664 architecture producing Mach-O executable files targeting MacOS  


https://developer.apple.com/library/archive/documentation/Performance/Conceptual/CodeFootprint/Articles/MachOOverview.html
http://math-atlas.sourceforge.net/devel/assembly/MachORuntime.pdf
https://en.wikipedia.org/wiki/Data_segment
https://docs.oracle.com/cd/E26502_01/html/E28388/eoiyg.html
https://www.objc.io/issues/6-build-tools/mach-o-executables/

> **Note**
> Useful commands: xxd, file, otool, c++filt.
> - otool -v -t: disassemble text section
> - otool -v -s __TEXT __cstring
> - otool -v -h: header
> - otool -v -l

### Basics of Data Segments & Section
- Text
- - __text: The compiled machine code for the executable
- - __const: The general constant data for the executable
- - __cstring: Literal string constants (quoted strings in source code)

- Data
- - __data: Initialized global variables (for example int a = 1; or static int a = 1;).
- - __const: Constant data needing relocation (e.g. pointer) (for example, char * const p = "foo";).
- - __bss (block started by symbol): Uninitialized static variables (for example, static int a;).
- - __common: Uninitialized external globals, will be merged to bss during linking (for example, int a; outside function blocks).
https://stackoverflow.com/questions/16835716/bss-vs-common-what-goes-where

### Static
- Static variables are stored on data segment. Data section if modifiable and initialized. Const if constant but its content is not determined at compile time. Bss if modifiable and zero/uninitialized. Common if in class, modifiable and zero/uninitialized.
- For global functions and variables, Static variable/function limits its scope to current file (remove .globl directive)
- For C++ class static members, static doesn't limit scope.


### Extern
Extern means somewhere else someone has instantiated this variable. Usually this introduces variables from other files. Extern just declares the variable to make compiler know it is there, like function declarations.

### Global Variable
Global variables means .globl data. It makes the assembler/linker know it's visible to all files being assembled/linked. Common global data include methods, variables declared in global scope, and in [C++] static members of class. Declaring in global scope doesn't guarantee it to be global (static).

### Const
```c
// const means what's right to it is unmodifiable
//*p1 is const, p1 is not
const int *p1 = &a;
// same as first
int const *p2 = &a;
// p3 is const, *p3 is not
int *const p3 = &a;
// p3 is const, *p3 is const
const int *const p4 = &a;
// p3 is const, *p3 is const
int const *const p5 = &a;
```
Global constants are stored in Text,const section. Const keyword doesn't affect assembly too much. When static and const are used together, static behavior mentioned before will be slightly affected.

### Where Every Type Of Data/Variable is Stored

> **Note**
> Basically, `static` keyword move variables to TEXT/DATA segment instead of stack/heap. Unmodifiable/unchanged things go to TEXT/Data,const section. This is usually indicated by `const` keyword. Uninitialized global static variables go to bss section. Other global variables go to DATA section.

> **Note**
> Compiler manages where variable goes by using assembler directives. This is an example of how assembly is NOT equivalent to machine code. Not all directives will be projected to a change in machine code as some are used by assembler like variable names and syntactic sugars are consumed by compiler. e.g.
> The .globl directive only declares the symbol to be global in scope, it does not define the symbol

```c
struct Person
{
    const int id = 1; // instance member
    const char *name = "ddsds"; // instance member
    static const int age; // TEXT,const
    static const char *const staticStr; // DATA,const
    static int count; // DATA,data
};

// .section	__DATA,__data
static int globalStatic = 1;
// .zerofill __DATA,__bss,_globalUninitOrZeroStatic,4,2 ## @globalUninitOrZeroStatic
static int globalUninitOrZeroStatic = 0;
// __TEXT, __const
const int globalConstant = 1;

// https://stackoverflow.com/questions/496448/how-to-correctly-use-the-extern-keyword-in-c
// this declares a, it should be instantiated somewhere else
extern int a;

// .section	__TEXT,__text,regular,pure_instructions
// .globl _sum
int sum(int a, int b)
{
    // .zerofill __DATA,__bss,_sum.functionStatic,4,2 ## @sum.functionStatic
    static int functionStatic = 0;
    printf("address of functionStatic: %p\n", &functionStatic);
    printf("address of a: %p\n", &a);
    printf("address of b: %p\n", &b);
    return a + b;
}

// .section	__TEXT,__text,regular,pure_instructions
// note this will not have globl directive, meaning it can't be used by other files by the linker.
static void staticFunction()
{
}

// .section	__TEXT,__text,regular,pure_instructions
// .globl _main
int main()
{
    // leaq	_sum(%rip), %rax
    int (*p)(int a, int b) = sum; 
    register int registerInt = 3;
    int result = (*p)(1, registerInt);

    // string literals are in .section	__TEXT,__cstring,cstring_literals
    printf("address of pointer to function: %p\n", &p);
    printf("address of sum: %p\n", p);
    printf("address of result: %p\n", &result);
    printf("address of globalStatic: %p\n", &globalStatic);
    printf("address of globalUninitOrZeroStatic: %p\n", &globalUninitOrZeroStatic);
    printf("address of global: %p\n", &a);
    // printf("address of registerInt: %p", &registerInt);
    return 0;
}

// this instantiates a
// .comm	_a,4,2 which is in common section of data segment
int a;
```
## Shenanigans 
### Declaration vs Instantiation vs Initialization
- Declaration won't affect the actual program (assembly). It merely tells the compiler that something exists. e.g. `extern int a` `int sum(int a, int b);`
- Instantiation allocates actual space somewhere for something. e.g. `int a` `int a = 1` `int magic() {return 1};`
- Initialization is somewhat vague. Usually it refers to manual assignment of value to something. 


### Why Default Argument Has to be in Function Declaration 
Remember what a function declaration is. It adds a function name to a list somewhere kept by the compiler and when you use a function, the compiler looks the name up in its list. A function name has a name and arguments. So function with default arguments basically creates multiple functions. When you, say, use a function add(int a = 0, int b = 0) without giving argument, you are actually calling add() (compiler will directly initialize a and b in a normal procedure). So add() has to be declared, and it's done with a default argument function declaration. If it were put in implementation, compiler has no way to know if add() and add(int a) exist or not.

### Variadic Argument
printf(const char* str, ...)

This is compiler thing where in gcc these arguments will be sent as normal arguments and retrieved as normal arguments (not an array). 

## Comma Expression
```c
int a = 9;
int b = 10;

a = a+1 , b = 3*4;
```
now a = 10, b = 12
```c
int a = 2;
int b = 0;
int c;

c = (++a, a *= 2, b = a * 5); 

printf("c = %d", c);
```
This will print c = 30. Don't miswrite it as this:
```c
c = ++a, a *= 2, b = a * 5;
printf("c = %d", c);
```
This will print c = 3.

# Pointer
## How To Understand Pointer
### Overview
To understand pointer, you have to first understand assembly and how a program actually works. Then you have to know how c compiler actually inteprets what you write to assembly code. The rest is about getting used to it.

A good starting point for Intel AT&T assembly: https://flint.cs.yale.edu/cs421/papers/x86-asm/asm.html

### Nature of Variable
A pointer is a variable, so let’s see how a variable declaration is interpreted by our compiler. 
```c
int a = 10;
int b = 20;
```
This translates to this:
```s
movl	$10, -20(%rbp)
movl	$20, -24(%rbp)
```
So declaring a variable basically means moving a value, either from an immediate or from a register, to a chunk of memory on stack.

Do remember that the name "a" and "b" don't exist in machine code. It's just a value at a memory location. Finding out which value corresponds to which variable you write is the compiler's job.

### Nature of Pointer
Luckily, a pointer is a variable, so the nature of a pointer is exactly the same as a variable. It's still a chunk of memory (8 byte) storing a "number" and compiler decides how to interpret this number. 

```c
int *p1 = 10;
int *p2 = a;
int *realP = &a;
```
This is translated to this:
```s
movl	$10, %eax
movq	%rax, -32(%rbp)

movslq	-20(%rbp), %rax
movq	%rax, -40(%rbp)

leaq	-20(%rbp), %rax
movq	%rax, -48(%rbp)
```
movslq means move sign extended long to quadword. leaq means "load effective address" to register

The compiler first calculate what needs to be stored in the pointer and store it in a register then move value of this register to stack. 

In the third example where we declared a pointer in a "normal" way, we can see it's nothing special. A value is pasted to rax and rax moved to stack. The point is pointer is really not a completely new thing from an int or char, and you shouldn't think of it differently. 

### & and *
As shown in the previous example, & followed by a variable will first evaluate the variable, finds the memory location, and `leaq` it to a temporary register. &pointer is no different.

But why do we create this pointer type at all if it's just a normal variable? Can't we just store addresses with long long type? The problem is accessing the variable stored in the address stored in a pointer. 

```c
int a = 10;
int *realP = &a;
long isThisAPointer = &a;
printf("%p\n", realP);
printf("%p\n", isThisAPointer);
printf("%d\n", *realP);
// printf("%d\n", *isThisAPointer);
```
The last printf won't work because * must be applied on a pointer. This is a compiler thing (compiler maintains a symbol table showing types of variable and complains if type is wrong) and it's possible that the designer allows * followed by any data type but it's dangerous. The printf with %p format specifier however, doesn't seem to care, the results from first 2 printf are the same.

Let's examine how * works in assembly:
```s
// printf("%d\n", *realP);
movq	-48(%rbp), %rax
movl	(%rax), %esi
leaq	L_.str.1(%rip), %rdi
movb	$0, %al
callq	_printf
```
So *p basically means (p).

Note: pointer type in assembly really just means the specifier like q, l, w, b, this is similar in primitive types

### Why Pointer
Note that not many language has explicit pointer type and many programmers don't really use it. There are several reasons to use pointer: 1) function call where the argument is big 2) referencing another data structure 3) directly manipulating memory address. But these are essentially the same thing: call data by their address instead of names (like a delivery service where the courier doesn't have to know you, just the address is enough)

## Array
### Nature of Array Variable
```c
int array1[] = {1, 2, 3};
printf("%p\n", array1);
printf("%p\n", &array1);
printf("%x\n", array1);
printf("%x\n", &array1);
```
All these are almost equavelent and will print the same address. 

This is the asm code
```s
# -20(%rbp) is address of a, a, and address of the stored integer 1
# L_.str(.1) is the formatter string
movl	%eax, -12(%rbp)
leaq	-20(%rbp), %rsi
leaq	L_.str(%rip), %rdi
movb	$0, %al
callq	_printf
leaq	L_.str(%rip), %rdi
leaq	-20(%rbp), %rsi
movb	$0, %al
callq	_printf
leaq	-20(%rbp), %rsi
leaq	L_.str.1(%rip), %rdi
movb	$0, %al
callq	_printf
leaq	L_.str.1(%rip), %rdi
leaq	-20(%rbp), %rsi
movb	$0, %al
callq	_printf
```
So an array variable is just like a pointer. Next we'll discuss the differences.

### Nature of Array Declaration
```c
int main(int argc, char const *argv[])
{
    int arr[3];
    arr[1] = 2;
    int *p_arr1 = &arr;
    int *p_arr2 = arr;
    *p_arr1 = 1;
    // *(p_arr2 + 5) = 3; this is fine
    // *(p_arr2 + 2) = 3; this is totally legit
    // *(p_arr2 + 3) = 3; or *(p_arr2 + 4) = 3; gives you abort 6
    printf("%d%d%d", arr[0], arr[1], arr[2]);
    return 0;
}


/*
	subq	$64, %rsp                                   as seen here, stack array declaration must include size so everything is preallocated, note it's 16 byte aligned
	leaq	-20(%rbp), %rax                             load arr address to rax
	movq	___stack_chk_guard@GOTPCREL(%rip), %rcx     get canary
	movq	(%rcx), %rcx                                get canary
	movq	%rcx, -8(%rbp)                              put canary on stack top
	movl	$0, -24(%rbp)                               protect area below array?
	movl	%edi, -28(%rbp) ??
	movq	%rsi, -40(%rbp) ??
	movl	$2, -16(%rbp)                               
	movq	%rax, %rcx                                  pointer init with &
	movq	%rcx, -48(%rbp)                             this is int *p = &arr
	movq	%rax, -56(%rbp)                             this is int *p = arr, pointer init with pointer is one line shorter
	movq	-48(%rbp), %rax
	movl	$1, (%rax)
	movq	-56(%rbp), %rax
	movl	$3, 8(%rax)
	movq	___stack_chk_guard@GOTPCREL(%rip), %rax
	movq	(%rax), %rax
	movq	-8(%rbp), %rcx
	cmpq	%rcx, %rax
	jne	LBB0_2
	retq
LBB0_2:
	callq	___stack_chk_fail
	ud2

*/
```
- The difference between pointer and array declaration is that a pointer is just moving a value into the stack (like any other declarations) while an array means more work for the compiler.
- First it has to make sure the array has a fixed length so it can allocate an fixed amount of space on stack
- Then it must handle some security problems like putting the canary on top of stack
- Some optimization like translating array access directly to a stack location is also seen here (movl $2 -16(%rbp)).
## String
Just an array with 0 ('\0') at the end.
### puts

# Other 
## The Heap
![heap you lock keep key.png](https://user-images.githubusercontent.com/36484215/190881940-3c891f7e-cc05-4059-bd37-93e4edd8e433.png)

## Macro
### Overview
This is very similiar to #include. All macros are basically replaced with another thing.

1. All capital letter
2. #undef or end of file marks end of macro lifetime
3. You can use macros to define macros

### Macro with Parameters
#define average(a, b) (a+b)/2 means all appearance of average will be replaced with (a+b)/2 and the parameter list following average will be cast to (a, b)


## Reference Type
### Nature of Reference Type
As shown in assembly, reference is exactly the same as pointer. It makes pointers easier to use in some cases, as in function parameter.

## Namespace 

## C-ASM Mix
### Inline ASM Code
```c
#define outb(value, port) \
	__asm__("outb %%al,%%dx" ::"a"(value), "d"(port))

#define inb(port) ({                 \
	unsigned char _v;                \
	__asm__ volatile("inb %%dx,%%al" \
					 : "=a"(_v)      \
					 : "d"(port));   \
	_v;                              \
})

#define outb_p(value, port)    \
	__asm__("outb %%al,%%dx\n" \
			"\tjmp 1f\n"       \
			"1:\tjmp 1f\n"     \
			"1:" ::"a"(value), \
			"d"(port))

#define inb_p(port) ({                 \
	unsigned char _v;                  \
	__asm__ volatile("inb %%dx,%%al\n" \
					 "\tjmp 1f\n"      \
					 "1:\tjmp 1f\n"    \
					 "1:"              \
					 : "=a"(_v)        \
					 : "d"(port));     \
	_v;                                \
})
```
Note:
1. asm is usually used as macro
2. multiline macro requires "\"
3. use () to transform a statement to an expression. Last expression is the value of whole expression as seen in `_v` in inb_p and inb

Rules:
1. first part is asm code with each line wrapped in ""
2. second part is output starting with :
3. third part is input starting with :
4. you can use %0-9 to refer to registers from input/output, output is by default %0

![Screen Shot 2022-05-22 at 9.09.48 AM.png](https://user-images.githubusercontent.com/36484215/190881958-11e225f7-04ae-4142-9a35-46a068ccd683.png)


### Call C and ASM Function In ASM and C File
Calling c function in asm or vice versa is easy. Just remember it’s all machine code at the end. Call c in asm is just calling the right mangled symbol and call asm in c is just normal function calling, it will be translated to asm later on.


[[toc]]


# Class & Struct
:::tip
`clang -cc1 -std=c++11 -fdump-record-layouts your-file.cpp` will create a list of all objects' properties
:::

## Overview

``` c
struct Person
{
    int m_age = 0;
    int m_id = 0;

    void greet()
    {
        cout << (*this).m_age << " " << (*this).m_id << endl;
    }
};

int main(int argc, char const *argv[])
{
    //Reference type
    int array[] = {1, 2, 3};
    int(&a)[3] = array;
    int *const &b = array;

    printf("address of the variable 'array': %p\n", array); //0x7ffeee9bf2ac
    printf("address of the content stored in 'array': %p\n", &array[0]); //0x7ffeee9bf2ac

    //Objects
    Person personOnStack;
    personOnStack.greet();

    printf("address of the variable 'personOnStack': %p\n", &personOnStack); //0x7ffeee9bf278

    Person *personOnHeap = new Person();
    personOnHeap->greet();

    printf("address of the variable 'personOnHeap': %p\n", &personOnHeap); //0x7ffeee9bf270
    printf("things stored in pointer 'personOnHeap' (address): %p\n", personOnHeap); //0x7f89dc405cc0
    printf("address of the actual 'personOnHeap': %p\n", &personOnHeap->age); //0x7f89dc405cc0
    return 0;
}
```

## Creating / Deleting Memory 
- malloc/calloc/realloc \ free 
- new \ delete
- new [] \ delete []

> **Note**
> new will allocate on heap and return a pointer while `Class class` will allocate on stack. Both will call init method.

### Array Initialization
![Screen Shot 2021-11-09 at 3.00.17 PM.png](https://boostnote.io/api/teams/wE89btYff/files/96bb9c87595545258dc20b51f074ebf05d4dc8208858347fefeef29ac72fbf70-Screen%20Shot%202021-11-09%20at%203.00.17%20PM.png)


## Nature of struct/class

``` cpp
struct Person
{
    int age;
    int id;
    int height;

    void display()
    {
        cout << this->age << this->id << this->height << endl;
    }
};

int main(int argc, char const *argv[])
{
    int a = 100;
    Person person = {10, 20, 30};
    Person *p = (Person *)&person.id;
    person.display(); // 102030
    p->display();     // 2030100

    return 0;
}
```
The type Person is just like any other types to help compiler know how much space should be used and to ensure safety measures.

### This
This is passed as first argument to instance methods. "->" means go to the address stored in this memory and offset by certain value. "." means the instance is found and just offset. this->var is equivalent as (\*this).var

### Class and struct difference
The only difference between a struct and class in C++ is the default accessibility of member variables and methods. In a struct they are public; in a class they are private.

## Constructor
![Screen Shot 2021-11-09 at 3.04.26 PM.png](https://boostnote.io/api/teams/wE89btYff/files/c0d55581159ffe1e6ef07666aa283c2fa63bfa50ee23216eb849cae01744c26d-Screen%20Shot%202021-11-09%20at%203.04.26%20PM.png)

### Assembly
```cpp
struct Person
{
    int age; int id; int height;
    Person() { this->age = 10; }
};

Person globalInstance;

int main(int argc, char const *argv[])
{
    globalInstance.id = 10;

    Person *cInstance = (Person *)malloc(sizeof(Person));
    cInstance->age = 9;

    Person *cppInstance = new Person();
    cppInstance->age = 20;
    return 0;
}
```
```c
___cxx_global_var_init:                 ## @__cxx_global_var_init
	leaq	_globalInstance(%rip), %rdi
	callq	__ZN6PersonC1Ev

__ZN6PersonC1Ev:                        ## @_ZN6PersonC1Ev
	movq	%rdi, -8(%rbp)
	movq	-8(%rbp), %rdi
	callq	__ZN6PersonC2Ev
  
__ZN6PersonC2Ev:
    movq %rdi, -8(%rbp)
    movq -8(%rbp), rax
    movl $10, (%rax)
    
_main:                                  ## @main
	movl	$10, _globalInstance+4(%rip)

	movl	$12, %edi			## sizeof()
	movq	%rdi, -72(%rbp)     ## storing size
	callq	_malloc

	movq	-72(%rbp), %rdi     ## reloading size
	movq	%rax, -24(%rbp)		## cInstance at rbp-24

	movq	-24(%rbp), %rax		## these two instructions are for ->
	movl	$9, (%rax)			## 
	callq	__Znwm				## new
	movq	%rax, %rdi			## new returns address of allocated memory
	movq	%rdi, %rcx			## use this address as "this" parameter
	movq	%rcx, -64(%rbp)                 ## 8-byte Spill
	movq	%rdi, %rcx
	movq	%rcx, -56(%rbp)                 ## 8-byte Spill
Ltmp0:
	callq	__ZN6PersonC1Ev
	movq	-56(%rbp), %rcx                 ## 8-byte Reload
	movq	%rcx, -32(%rbp)
	movq	-32(%rbp), %rcx
	movl	$20, (%rcx)
	retq

```
[Wrong]Only new and delete will call constructor and destructor.->Person person will also call constructor
### Nature of Constructing Instance
```c
struct Person
{
    int age;
    int height;
    int id;
};

struct Person2
{
    int age = 9; // this creates a default constructor with 0 arg
    int height;
    int id;
};
int main(int argc, char const *argv[])
{
    Person person1;            // this basically does nothing
    Person pc = person1;       // this copies memory belonging to person1 to another memory location
    Person person2();          // this is a function declaration
    Person *p1 = new Person;   // this calls new
    Person *p2 = new Person(); // this calls new and memset to 0

    Person2 person21;             // this calls contructor
    Person2 *p21 = new Person2;   // this calls new and contructor
    Person2 *p22 = new Person2(); // this calls new, memset to 0 and contructor
    return 0;
}
```

> **Note**
> Note that something like `Person p;` or `int a` aren't reflected directly in assembly. This just creates metadata (type, address) about a variable for compiler. When other code refers to it, compiler knows if it's legal and how to evaluate its content with this metadata.

Class packages convenient operations around a specified chuck of data. The compiler first needs to know its size so appropriate space is allocated for this chuck of data. Usually you want some predefined value for this data when declaring it, so a constructor is used. Like any member function, a constructor takes in an address as the first argument. It returns nothing. So the workflow for instantiating an object by the compiler is to first allocate space then give the address of this space to a constructor function. Normally, a constructor changes only data within the chuck specified by class. Since constructor is very common, there're many syntactic sugar for it, especially when instantiating an object. It's also possible to have no constructor since it's just a function.

### Default Constructor
A default constructor is provided by the compiler when something needs to be filled into object at instantiation, namely when:
1. member variable has initial value
2. class has virtual function
3. parent is virtual
4. class virtually inherits another class 
5. parent has constructor

### Initialization List
```c
Person(int age, int height): m_age(age), m_height(height) {}
```

This is pure syntax sugar. Note that 1) you can put anything in the parenthesis. 2) evaluation order is variable declaration order in the class. 

### Delegating Constructor (C11)
```c
Person(): Person(10, 20) {}
Person(int age, int height): m_age(age), m_height(height) {}
```

:::note
Though delegating constructor is not available before c11, constructor inheritance is always available 
:::

### Constructor Inheritance
```c
struct Person
{
    int age;
    int height;
    Person(): Person(10, 20) {}
    Person(int age, int height) : age(age), height(height) {}
};

struct NormalPerson : Person
{
    int id;
    NormalPerson(int id) : Person(0, 0), id(id) {}
};
```

### Copy Constructor
```c
struct Person
{
    int age;
    int height;

    Person() : Person(0, 0) { cout << "person init" << endl; }
    Person(int age, int height) : age(age), height(height) { cout << "person init" << endl; }
    Person(const Person &person) : age(person.age), height(person.height) { cout << "person copy" << endl; }
};

struct NormalPerson : Person
{
    NormalPerson() : Person(0, 0) { cout << "normal person init" << endl; }
    NormalPerson(const NormalPerson &normalPerson) : Person(normalPerson) { cout << "normal person copy" << endl; }
};

int main(int argc, char const *argv[])
{
    NormalPerson p1; // person init, normal person init
    NormalPerson p2(p1); // person copy, normal person copy
    NormalPerson p3; // person init, normal person init
    p3 = p2; // this will use default copy mechanism (shallow, non constructor)
    NormalPerson p4 = p2; // person copy, normal person copy

    return 0;
}
```
```c
leaq -24(%rbp), %rdi // this p1
callq __ZN12NormalPersonC1Ev 
leaq -32(%rbp), %rdi // this p2
leaq -24(%rbp), %rsi // p1
callq __ZN12NormalPersonC1ERKS_ 
leaq -40(%rbp), %rdi // this p3
callq __ZN12NormalPersonC1Ev 
movq -32(%rbp), %rax // default copy p2
movq %rax, -40(%rbp) // default copy to p3
leaq -48(%rbp), %rdi // this p4
leaq -32(%rbp), %rsi // p2
callq __ZN12NormalPersonC1ERKS_ 
```
> **Note**
> Use deep copy when content in pointer needs to be copied. Use copy constructor when you need deep copy. Allocate memory on heap and make memory copy of pointers in the given instance.

> **Note**
> The default copy mechanism seen here is also used by value passing seen in function parameter, (e.g. `Person magic(Person p) {}` where Person doesn't have custom copy constructor) whereby the input is passed as discrete values using argument registers or stack (rsp addressing) while the returned Person is the address of the first element of new pereson. 

## Function and Object
### Object as Function Parameter/Return
```c
int sum3(int a, int b, int c) {}
struct ABC
{
    int a, b, c, d, e;
    ABC(int a, int b, int c, int d, int e) : a(a), b(b), c(c), d(d), e(e) {}
    ABC(const ABC &abc) : a(abc.a), b(abc.b), c(abc.c), d(abc.d), e(abc.e) {}
};
ABC magic(int a, ABC abc)
{
    ABC newABC(abc.a + a, abc.b + a, abc.c + a, abc.d + a, abc.e + a);
    return newABC;
}
int main(int argc, char const *argv[])
{
    ABC abc(2, 4, 6, 8, 10);
    ABC newABC = magic(1, abc);
    sum3(newABC.a, newABC.c, newABC.e);
    return 0;
}
```
```c
_main:                          
	leaq	-40(%rbp), %rdi     ## -40 is now "this" for ABC constructor
	movl	$2, %esi			## normal argument feeding
	movl	$4, %edx
.....
    callq	__ZN3ABCC1Eiiiii	## ABC constructor 
	leaq	-88(%rbp), %rdi		## -88 is now "this" for ABC copy constructor
	leaq	-40(%rbp), %rsi		## -40 is the const ABC & argument to be copied
	callq	__ZN3ABCC1ERKS_		## this copy constructor will copy from -40 to -88
	leaq	-64(%rbp), %rdi		## -64 is used for returned object
	movl	$1, %esi			## int a
	leaq	-88(%rbp), %rdx		## ABC abc
	callq	__Z5magici3ABC		## -64 will now store newABC object
	movl	-64(%rbp), %edi
	movl	-56(%rbp), %esi
	movl	-48(%rbp), %edx
	callq	__Z4sum3iii
	retq

__Z5magici3ABC:                         ## @_Z5magici3ABC
	movq	%rdx, %rax			## ABC abc address in rdx and rax
	movq	%rdi, %rcx			## return address in rdi and rcx
	movq	%rcx, -24(%rbp)     ## -24 stores return address
	movq	%rdi, %rcx			## 
	movq	%rcx, -8(%rbp)		## -8 stores return address
	movl	%esi, -12(%rbp)		## -12 stores int a = 1
	movl	(%rax), %esi		## first element of abc to esi (second argument)
	addl	-12(%rbp), %esi		## add a = 1 to abc.a, save to esi
	movl	4(%rax), %edx
	addl	-12(%rbp), %edx
	movl	8(%rax), %ecx
	addl	-12(%rbp), %ecx
.....
    callq	__ZN3ABCC1Eiiiii	## ABC constructor
	movq	-24(%rbp), %rax     ## return address to rax
	retq

__ZN3ABCC1Eiiiii:                       ## @_ZN3ABCC1Eiiiii
	movq	%rdi, -8(%rbp)
	movl	%esi, -12(%rbp)
.....
	movq	-8(%rbp), %rdi
	movl	-12(%rbp), %esi
.....
	callq	__ZN3ABCC2Eiiiii
	retq

__ZN3ABCC2Eiiiii:                       ## @_ZN3ABCC2Eiiiii
	movq	%rdi, -8(%rbp)
	movl	%esi, -12(%rbp)
.....
	movq	-8(%rbp), %rax
	movl	-12(%rbp), %ecx
	movl	%ecx, (%rax)
	movl	-16(%rbp), %ecx
	movl	%ecx, 4(%rax)
.....
  retq

__ZN3ABCC1ERKS_:                        ## @_ZN3ABCC1ERKS_
	movq	%rdi, -8(%rbp)		## function callee saving convention
	movq	%rsi, -16(%rbp)
	movq	-8(%rbp), %rdi
	movq	-16(%rbp), %rsi
	callq	__ZN3ABCC2ERKS_
	retq

__ZN3ABCC2ERKS_:                        ## @_ZN3ABCC2ERKS_
	movq	%rdi, -8(%rbp)		## function callee saving convention
	movq	%rsi, -16(%rbp)
	movq	-8(%rbp), %rax		## rax stores "this" address
	movq	-16(%rbp), %rcx		## rcx stores address of object to be copied
	movl	(%rcx), %ecx		## first element of rcx
	movl	%ecx, (%rax)		## copy it to first element of rax
	movq	-16(%rbp), %rcx
	movl	4(%rcx), %ecx
	movl	%ecx, 4(%rax)
	movq	-16(%rbp), %rcx
	movl	8(%rcx), %ecx
	movl	%ecx, 8(%rax)
.....
    retq
```
- When passing an object as parameter, either default register based copying mechanism or copy constructor is used.
- When returning an object, the caller to this function will prepare some memory on stack for the returned object and pass the address of the first element as the first parameter. 
- Now the returned object must be either created or copied inside the function.
- If the returned object is created inside the function, the return address is automatically used as first argument (this) to object constructor.
- If the returned object is copied, the return address is used as first argument (this) of copy constructor.
- This way, no temp object is needed since the output object is not really returned or copied from an intermediate object, but filled directly by the function with a predefined address.

### Implicit Construction of Object
```c
int sum3(int a, int b, int c) {}
struct A
{
    int a;
    A(int a) : a(a) {}
};

A magicA(A a)
{
    return a.a + 1;
}
int main(int argc, char const *argv[])
{
    A a1 = 8;
    A newA = magicA(9);
    sum3(1, 2, newA.a);
    return 0;
}
```
- Implicit construction relies on a constructor that takes exactly one argument. 
- You can prevent this by adding `explicit` before constructor.
## Member Type
### Static Class Member & Method
- static member variables are stored in global data section. 
- static member variables have to be initialized.
- static member functions obviously don't have "this" thus can't manipulate non static member variables.
- static member functions can't be virtual, constructor, or destructor

### Const
- const member variables have to be initialized either at the declaration or inside the class.
- const member functions can only manipulate const member vars or static vars.
- const members are still instance members.

```c
class Person
{
public:
    const int id = 1; // instance member
    const char *name = "ddsds"; // instance member
    static const int age; // TEXT,const
    static const char *const staticStr; // DATA,const
    static int count; // DATA,data
};

const int Person::age = 10;
const char *const Person::staticStr = "sdds";
int Person::count = 9;

const char *str = "ddsds";

int main(int argc, char const *argv[])
{
    Person *p = new Person;
    printf("%s", p->name);
    printf("%d", p->id);
    printf("%d", p->age);
    printf("%d", p->staticStr);
    printf("%d", p->count);
    printf("%s", str);
    return 0;
}
```

```c
_main:                                  ## @main
	callq	__Znwm
	callq	__ZN6PersonC1Ev
	movq	%rax, %rdi
	movq	-32(%rbp), %rax      // person pointer in rax
	movq	8(%rax), %rsi        // name
	callq	_printf
	movl	(%rax), %esi		 // id
	callq	_printf
	movl	$10, %esi			 // static const int age
	callq	_printf
	leaq	L_.str(%rip), %rsi   // static const char *const staticStr
	callq	_printf
	movl	__ZN6Person5countE(%rip), %esi // static int count
	callq	_printf
	movq	_str(%rip), %rsi
	callq	_printf
	retq

__ZN6PersonC2Ev:                        ## Person Constructor
	movq	%rdi, -8(%rbp)
	movq	-8(%rbp), %rax
	movl	$1, (%rax)             // const int id = 1
	leaq	L_.str.1(%rip), %rcx   
	movq	%rcx, 8(%rax)          // const char *name = "ddsds"; notice it's 8 byte aligned (also notice string literal optimization, stored once)
	retq

	.section	__TEXT,__const .globl	__ZN6Person3ageE      ## person.age
__ZN6Person3ageE:
	.long	10                              ## 0xa

	.section	__TEXT,__cstring,cstring_literals
L_.str:                                 ## @.str
	.asciz	"sdds"

	.section	__DATA,__const
	.globl	__ZN6Person9staticStrE          ## static const char *const staticStr
__ZN6Person9staticStrE:
	.quad	L_.str

	.section	__DATA,__data
	.globl	__ZN6Person5countE              ## @static int count;
__ZN6Person5countE:
	.long	9                               ## 0x9

	.section	__TEXT,__cstring,cstring_literals
L_.str.1:                               ## @.str.1
	.asciz	"ddsds"

	.section	__DATA,__data
	.globl	_str                            ## @str
_str:
	.quad	L_.str.1
```

### Reference Type Member Variable
Has to be initialized inside class. Comes in handy as function parameter.

## Inheritance
### constructor Inheritance 
1. By default, child constructor will call parent constructor without argument.
2. If parent doesn't have any constructor, no parent constructor will be called.
3. If parent has any constructor without having no argument constructor, error.
4. Child can call any parent constructor explicitly in its initialization list
5. You can't use constructor inside constructor body as always

```cpp
struct Person {
  int m_age;
  
  Person(int age) {
    this->m_age = age;
  }
  //or Person(int age): m_age(age) {}
}

struct Student : Person {
  int m_id;
  
  Student(int id, int age): m_id(id), Person(age) {};
}
```

### Method and Member Of the Same Name
If both parent and child class have method or member with same names, we can use `childClass.ParentClass::m_var/m_func` to access any methods or variables. This is because we are guaranteed to have everything the parent method needs in the child object.

## VMT
### Structure
```c
struct Person3
{
    int age;
    virtual void run() {};
    virtual void die() {};
};

void magic(Person3 &p)
{
    p.run();
    p.die();
}
```
```c
__Z5magicR7Person3:                     ## @_Z5magicR7Person3
	movq	%rdi, -8(%rbp)
	movq	-8(%rbp), %rdi
	movq	(%rdi), %rax
	callq	*(%rax)
	movq	-8(%rbp), %rdi
	movq	(%rdi), %rax
	callq	*8(%rax)
	retq

__ZN7Person3C2Ev: ## Person3 constructor
	movq	__ZTV7Person3@GOTPCREL(%rip), %rcx
	addq	$16, %rcx
	movq	%rdi, -8(%rbp)
	movq	-8(%rbp), %rax
	movq	%rcx, (%rax)
	popq	%rbp
	retq
  
	.section	__DATA,__const
	.globl	__ZTV7Person3                
__ZTV7Person3:
	.quad	0
	.quad	__ZTI7Person3
	.quad	__ZN7Person33runEv
    .quad	__ZN7Person33dieEv
    
	.section	__TEXT,__const
	.globl	__ZTS7Person3                   ## @_ZTS7Person3
__ZTS7Person3:
	.asciz	"7Person3"

	.section	__DATA,__const
	.globl	__ZTI7Person3                   ## @_ZTI7Person3
__ZTI7Person3:
	.quad	__ZTVN10__cxxabiv117__class_type_infoE+16
	.quad	__ZTS7Person3  
```

![Screen Shot 2021-08-24 at 11.13.40 PM.png](https://boostnote.io/api/teams/wE89btYff/files/2222eaedff2dcc1b12c8b71da4d6f54255a5be351390b3e76c6631ef1aff7dfc-Screen%20Shot%202021-08-24%20at%2011.13.40%20PM.png)
![Screen Shot 2021-08-24 at 11.18.26 PM.png](https://boostnote.io/api/teams/wE89btYff/files/78ae8e0b28f1d90f8037f73ba7d780c065736477333f50d5854ea6c48d0115ec-Screen%20Shot%202021-08-24%20at%2011.18.26%20PM.png)

> **Important**
> Objects in memory (heap, stack, or global) only store instance variables if no virtual method is declared. If there exists any virtual method (in method chain), at the start of object memory there will be a pointer to the virtual methods table. There is one vmt for each  parent class with virtual methods. You are guaranteed to find the  implementation of a method by using fixed offset. This is true because you can only declare child instance with parent class and not the reverse. This means this instance is guaranteed to have the parent method table at the bottom of its own VMT.
> TL;DR: At compile time, method call uses the first 8 bytes as the VMT address and find the method with the same offset as the declaring class's VMT. At run-time, whenever an object is created, it either link to an existing VMT or create a new one depending on its declaring class and its real class. One class corresponds to one VMT. 

### Rules
1. Without virtual method calling is just normal c function calling
2. Virtual methods in child class are automatically virtual (can omit virtual keyword)
3. Use virtual destructor for polymorphism 
4. Polymorphism is used mainly for interface and function calling with similar effects. 

```cpp
struct Person {
    int age;
    int height;

    virtual void run() {
        cout << "person run" << endl;
    }
};

struct Worker {
    int salary;

    virtual void work() {
        cout << "worker work" << endl;
    }
};

struct Student : Person, Worker {
    int id;

    void run() {
        cout << "student run" << endl;
    }
    void work() {
        cout << "student work" << endl;
    }
};

int main(int argc, char const *argv[])
{
    Person *p = new Person;
    Person *s = new Student;
    Student *ss = new Student;
    Worker *w = new Student;

    p->run();
    s->run();
    ss->work();
    w->work();
  
    long *lp = (long *)(p);
    long *ls = (long *)(s);
    long *lss = (long *)(ss);
    long *lw = (long *)(w);

    cout << *lp << endl
         << *ls << endl
         << *lss << endl
         << *lw << endl
         << w << endl
         << &w->salary << endl;
}
//output:
person run
student run
student work
student work
4507684936
4507684976
4507684976
4507685008
0x7fd4d1c05d40
0x7fd4d1c05d48
```
VMT for person only has run method. VMT for student has both run and work method and run is first method. Worker has only work method in VMT. VMT for student declared using parent class will use the same VMT as student for the first declared parent (Person in this case). Others will use different VMT with their overriden parent method at the front. This way, the compiler just have to find the offset for VMT. 

## Inheritance With Virtual 
### Abstract Class
Any class containing pure virtual method is abstract, including unimplemented pure virtual method from ancestors. Abstract class cannot declare instances.

```c
struct Animal {
  virtual void run() = 0;
}
```

### Multiple Inheritance


### Virtual Inheritance
Virtual inheritance is like virtual method in that it creates a virtual table, but for member variables, not functions. This way it can avoid duplicates of shared ancestor class as seen in deadly diamond problem.

![Screen Shot 2022-05-13 at 6.54.32 PM.png](https://boostnote.io/api/teams/wE89btYff/files/c7ba2f628ac543cc62777fb537ce6f3cfc8b6ddd0d7610ef425e37c74753a964-Screen%20Shot%202022-05-13%20at%206.54.32%20PM.png)

## Other Features
### Friend 
- friend non-member function
- friend class

### Class inside Class
- private:
- protected:
- public:

[[toc]]

## Operator Overload
### List of Overloadable Operators 
- +、-、+=、==、!=、-、++、--、<<、>>
### Optimization
```c
struct Person
{
    int age; int height; int id;

    Person(int age, int height, int id) : age(age), height(height), id(id) {}
    Person(const Person &p) : age(p.age + 1), height(p.height + 1), id(p.id + 1) {}
};

Person operator+(const Person &p1, const Person &p2)
{
    return Person(p1.age + p2.age, p1.height + p2.height, p1.id + p2.id);
}

int main(int argc, char const *argv[])
{
    Person p1 = Person(1, 2, 3);
    Person p2 = Person(2, 4, 6);
    Person p3 = p1 + p2;
    printf("%d", p3.id);
    return 0;
}

// copy constructor 186
// nothing 167
// reference 143
// both 134 this is because copy constructor makes return 
// easier without really writing copy constructor
```
### const
Depending on if the return value of an operator should be changed and if the operator changes operands, const should be added to return value, function, and input accordingly.

```c
Person &operator++()
{
    age+=1; height+=1; id+=1;
    return (*this);
}

const Person operator++(int)
{
    Person old = Person(*this);
    age+=1; height+=1; id+=1;
    return old;
}

const Person operator-() const
{
  return Person(-age, -height, -id);
}
```

## Template
### Basic
```c
template <typename T>
T add(T a, T b)
{
    return a + b;
}

int main(int argc, char const *argv[])
{
    add(1, 2);
    add<double>(1.1, 2.2);
    add<Person>(Person(1, 2, 4), Person(1, 2, 4));
    return 0;
}
```
```c
__Z3addIiET_S0_S0_ ## int add<int>(int, int)
__Z3addIdET_S0_S0_ ## double add<double>(double, double)
__Z3addI6PersonET_S1_S1_ ## Person add<Person>(Person, Person)
```
Basically, template is a syntax sugar to help programmers generate multiple functions with similar implementation by writing only one "template" function.

### Why Template Can't be Implemented in Other Files
Basics of compiling and linking:
A cpp file can be seen as comprising of either functions or codes/implementations. Codes are translated to assembly which can be executed once assembled. Functions are `call` instructions. One file need only guarantee that its codes are correct and functions exists. The implementation of functions is guaranteed by other files. The process of combining all these files and determining which address to call exactly is done in linking (resolving symbols). If the above protocol is violated by any file, linker will fail.

When putting template function in another file, compiler has no idea what assembly to generate thus it won't generate anything for a template. So the solution is to always put together a template and code using the template by including a header/cpp file containing the implementation of template in files using the template. The convention is to call it a "hpp" (header & cpp) file. Remember to use `#progma once`. 


## C11 Features
### auto
Basically a var in swift or java.

### nullptr
Definition of NULL (it's basically just 0):
```
#ifdef __cplusplus
#  if !defined(__MINGW32__) && !defined(_MSC_VER)
#    define NULL __null
#  else
#    define NULL 0
#  endif
#else
#  define NULL ((void*)0)
#endif
```
This is ambiguous when using NULL as a parameter to functions that accepts an integer and pointer.

Functions called when declaring `int *p = nullptr;`:
```
c++filt __ZNSt3__1L15__get_nullptr_tEv
std::__1::__get_nullptr_t()
c++filt __ZNKSt3__19nullptr_tcvPT_IiEEv
std::__1::nullptr_t::operator int*<int>() const
```
nullptr resolves ambiguity of NULL since NULL is literally just 0.

### Lambda
```c
int exec(int a, int b, int (*func)(int, int))
{
    return func(a, b);
}

int add(int a, int b)
{
    return a + b;
}

int main(int argc, char const *argv[])
{
    int x = 10;
    int y = 20;
    auto p = [x, &y](int a, int b) -> int
    {
        return a + b + x + ++y;
    };

    exec(1, 2, add);
    exec(4, 3, ([](int a, int b) -> int
                { return a + b; }));
    printf("%d", p(1, 2) + p(3, 4) + ([y](int a, int b) mutable -> int
                                      { return a + b + ++y; })(3, 4));
    return 0;
}

```
```c
__Z4execiiPFiiiE:                       ## @_Z4execiiPFiiiE
.....
	callq	*%rdx

__Z3addii:                              ## @_Z3addii

_main:                                  ## @main
	movl	$10, -20(%rbp)				## -20 stores 10;  int x = 10;
	movl	$20, -24(%rbp)				## -24 stores 20;  int y = 20;
	movl	-20(%rbp), %eax				
	movl	%eax, -40(%rbp)				## -40 stores 10;  capturing x
	leaq	-24(%rbp), %rax				
	movq	%rax, -32(%rbp)				## -32 stores address of -24(%rbp); capturing &y
	movl	$1, %edi
	movl	$2, %esi
	leaq	__Z3addii(%rip), %rdx
	callq	__Z4execiiPFiiiE			## exec(1, 2, add);
	leaq	-48(%rbp), %rdi
	callq	__ZZ4mainENK3$_0cvPFiiiEEv	## get address of ([](int a, int b) -> int { return a + b; })
	movq	%rax, %rdx					## put address in 3rd argument
	movl	$4, %edi
	movl	$3, %esi
	callq	__Z4execiiPFiiiE			## exec(4, 3, ([](int a, int b) -> int { return a + b; }));
	leaq	-40(%rbp), %rdi				## %rdi stores address of -40(%rbp) which stores first of captured argument for p
	movl	$1, %esi
	movl	$2, %edx
	callq	__ZZ4mainENK3$_1clEii		## p(1, 2)
	movl	%eax, -64(%rbp)             ## stores result in -64
	leaq	-40(%rbp), %rdi
	movl	$3, %esi
	movl	$4, %edx
	callq	__ZZ4mainENK3$_1clEii		## p(3, 4)
	movl	%eax, %ecx					## stores result in %ecx	
	movl	-64(%rbp), %eax             ## move result of p(1, 2) back to %eax
	addl	%ecx, %eax					## add p(1, 2) and p(3, 4)
	movl	%eax, -60(%rbp)             ## store result in -60
	movl	-24(%rbp), %eax				## copy y to %eax
	movl	%eax, -56(%rbp)				## create new variable with value of y; mutable
	leaq	-56(%rbp), %rdi				## move address of this new variable to first argument of lambda
	movl	$3, %esi					
	movl	$4, %edx
	callq	__ZZ4mainEN3$_2clEii		## ([y](int a, int b) mutable -> int { return a + b + ++y; })(3, 4)
	movl	-60(%rbp), %esi                 ## 4-byte Reload
	addl	%eax, %esi					## add result of __ZZ4mainEN3$_2clEii to previous sum
	leaq	L_.str(%rip), %rdi
	movb	$0, %al
	callq	_printf

__ZZ4mainENK3$_1clEii:                  ## @"_ZZ4mainENK3$_1clEii"
	movq	%rdi, -8(%rbp)				## %rdi is address of captured argument
	movl	%esi, -12(%rbp)				## first argument
	movl	%edx, -16(%rbp)				## second argument
	movq	-8(%rbp), %rcx				## %rcx stores address of captured argument
	movl	-12(%rbp), %eax				## %eax stores first argument
	addl	-16(%rbp), %eax				## add first and second argument, stores in %eax
	addl	(%rcx), %eax				## add first captured argument, stores in %eax
	movq	8(%rcx), %rdx				## %rdx stores second captured argument which is a pointer to y
	movl	(%rdx), %ecx				## %ecx stores value of y
	addl	$1, %ecx					## %ecx+=1
	movl	%ecx, (%rdx)				## store y + 1 back to *y. (%rdx) means memory addressed by %rdx
	addl	%ecx, %eax					## add added y to %eax

__ZZ4mainEN3$_2clEii:                   ## @"_ZZ4mainEN3$_2clEii"
    movq	%rdi, -8(%rbp)				## -8 stores address of copied y 
    movl	%esi, -12(%rbp)				## -12 stores first argument 
    movl	%edx, -16(%rbp) 			## -16 stores second argument 
    movq	-8(%rbp), %rdx 
    movl	-12(%rbp), %eax 
    addl	-16(%rbp), %eax 
    movl	(%rdx), %ecx 
    addl	$1, %ecx 
    movl	%ecx, (%rdx) 
    addl	%ecx, %eax 

__ZZ4mainENK3$_0cvPFiiiEEv:             ## @"_ZZ4mainENK3$_0cvPFiiiEEv"
	movq	%rdi, -8(%rbp)
	leaq	__ZZ4mainEN3$_08__invokeEii(%rip), %rax

__ZZ4mainEN3$_08__invokeEii:            ## @"_ZZ4mainEN3$_08__invokeEii"
	movl	%edi, -4(%rbp)
	movl	%esi, -8(%rbp)
	movl	-4(%rbp), %esi
	movl	-8(%rbp), %edx
	callq	__ZZ4mainENK3$_0clEii

__ZZ4mainENK3$_0clEii:                  ## @"_ZZ4mainENK3$_0clEii"
	movq	%rdi, -8(%rbp)
	movl	%esi, -12(%rbp)
	movl	%edx, -16(%rbp)
	movl	-12(%rbp), %eax
	addl	-16(%rbp), %eax
```
- Lambda is just a function with auto generated name and different syntax.
- Capture mechanism creates copies or pointers of surrounding element and stores these variables in a list. The address of first element of the list is provided as first argument to the lambda. Thus it's impossible to use lambda capturing elements as function argument (since that function has no idea how to provide the capture list).
- [&] means capture all with reference. [=] means capture all by copy. 
> **Note**
> Lambda expression creates more waste code than normal function and inline function.
