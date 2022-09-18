

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
![heap you lock keep key.png]
(https://user-images.githubusercontent.com/36484215/190881940-3c891f7e-cc05-4059-bd37-93e4edd8e433.png)

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
