# *__Memory Segments in C__*

In C, as in other compiled languages, the compiled code goes into the text segment, while the variables reside in the remaining segments. Exactly which memory segment a variable will be stored in depends on how the variable is defined. Variables that are defined outside of any functions are considered to be global. The _static_ keyword can also be prepended to any variable declaration to make the variable static. If static or global variables are initialized with data, they are stored in the data memory segment; otherwise, these variables are put in the _bss memory segment_. Memory on the _heap memory segment_ must first be allocated using a memory allocation function called _malloc()_. Usually, pointers are used to reference memory on the heap.

Finally, the remaining function variables are stored in the _stack memory segment_. Since the stack can contain many different stack frames, stack variables can maintain uniqueness within different functional contexts. The _memory_segment.c_ program will help explain these concepts in C.

__memory_segments.c__

```c
#include <stdio.h>

int global_var;
int global_initialized_var = 5;

// This is just a demo function.
void function() 
{
    int stack_var; // Notice this variable has the same name as the one in main().
    printf("the function's stack_var is at address 0x%08x\n", &stack_var);
}

int main() 
{
    int stack_var; // Same name as the variable in function()
    static int static_initialized_var = 5;
    static int static_var;
    int* heap_var_ptr;

    heap_var_ptr = (int*) malloc(4);
    
    // These variables are in the data segment.
    printf("global_initialized_var is at address 0x%08x\n", &global_initialized_var);
    printf("static_initialized_var is at address 0x%08x\n\n", &static_initialized_var);
    
    // These variables are in the bss segment.
    printf("static_var is at address 0x%08x\n", &static_var);
    printf("global_var is at address 0x%08x\n\n", &global_var);
    
    // This variable is in the heap segment.
    printf("heap_var is at address 0x%08x\n\n", heap_var_ptr);
    
    // These variables are in the stack segment.
    printf("stack_var is at address 0x%08x\n", &stack_var);
    function();
} 
```

Most of this code is fairly self-explanatory because of the descriptive variable names. The global and static variables are declared as described earlier, and initialized counterparts are also declared. The stack variable is declared both in _main()_ and in _function()_ to showcase the effect of functional contexts. The heap variable is actually declared as an integer pointer, which will point to memory allocated on the heap memory segment. The _malloc()_ function is called to allocate four bytes on the heap. Since the newly allocated memory could be of any data type, the _malloc()_ function returns a void pointer, which needs to be typecast into an integer pointer.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc memory_segments.c
reader@hacking:~/booksrc $ ./a.out
global_initialized_var is at address 0x080497ec
static_initialized_var is at address 0x080497f0

static_var is at address 0x080497f8
global_var is at address 0x080497fc

heap_var is at address 0x0804a008

stack_var is at address 0xbffff834
the function's stack_var is at address 0xbffff814
reader@hacking:~/booksrc $
</pre>

The first two initialized variables have the lowest memory addresses, since they are located in the data memory segment. The next two variables, __static_var__ and __global_var__, are stored in the _bss memory segment_, since they aren't initialized. These memory addresses are slightly larger than the previous variable's addresses, since the bss segment is located below the data segment. Since both of these memory segments have a fixed size after compilation, there is little wasted space, and the addresses aren't very far apart.

The heap variable is stored in space allocated on the heap segment, which is located just below the bss segment. Remember that memory in this segment isn't fixed, and more space can be dynamically allocated later. Finally, the last two __stack_var__s have very large memory addresses, since they are located in the stack segment. Memory in the stack isn't fixed, either; however, this memory starts at the bottom and grows backward toward the heap segment. This allows both memory segments to be dynamic without wasting space in memory. The first __stack_var__ in the _main()_ function's context is stored in the stack segment within a stack frame. The second __stack_var__ in _function()_ has its own unique context, so that variable is stored within a different stack frame in the stack segment. When _function()_ is called near the end of the program, a new stack frame is created to store (among other things) the __stack_var__ for _functions()_'s context. Since the stack grows back up toward the heap segment with each new stack frame, the memory address for the second __stack_var__ (_0xbffff814_) is smaller than the address for the first __stack_var__ (_0xbffff834_) found within _main()_'s context.