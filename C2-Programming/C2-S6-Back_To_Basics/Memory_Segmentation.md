# *__Memory Segmentation__*

A compiled program's memory is divided into five segments: Text, data, bss, heap, and stack. Each segment represents a special portion of memory that is set aside for a certain purpose.

The _text segment_ is also sometimes called the _code segment_. This is where the assembled machine language instructions of the program are located. The execution of instructions in this segment is nonlinear, thanks to the aforementioned high-level control structures and functions, which compile into branch, jump, and call instructions in assembly language. As a program executes, the _EIP_ is set to the first instruction in the the text segment. The processor then follows an execution loop that does the following:

1. Reads the instruction that _EIP_ is pointing to
2. Adds the byte length of the instruction to _EIP_
3. Executes the instruction that was read in step 1
4. Goes back to step 1

Sometimes the instruction will be a jump or a call instruction, which changes the _EIP_ to a different address of memory. The processor doesn't care about the change, because it's expecting the execution to be nonlinear anyway. If _EIP_ is changed in step 3, the processor will just go back to step 1 and read the instruction found at the address of whatever _EIP_ was changed to.

Write permission is disabled in the text segment, as it is not used to store variables, only code. This prevents people from actually modifying the program code; any attempt to write to this segment of memory will cause the program to alert the user that something bad happened, and the program will be killed. Another advantage of this segment being read-only is that is can be shared among different copies of the program, allowing multiple executions of the program at the same time without any problems. It should also be noted that this memory segment has a fixed size, since nothing ever changes in it.

The _data_ and _bss_ segments are used to store global and static program variables. The _data segment_ is filled with the initialized global and static variables while the _bss segment_ is filled with their uninitialized counterparts. Although these segments are writable, they also have a fixed size. Remember that global variables persist, despite the functional context (like the variable _j_ in the previous examples). Both global and static variables are able to persist because they are stored in their own memory segments.

The _heap segment_ is a segment of memory a programmer can directly control. Blocks of memory in this segment can be allocated and used for whatever the programmer might need.On notable point about the heap segment is that it isn't of fixed size, so it can grow larger or smaller as needed. All of the memory within the heap is managed by allocator and deallocator algorithms, which respectively reserve a region of memory in the heap for use and remove reservations to allow that portion of memory to be reused for later reservations. The heap will grow and shrink depending on how much memory is reserved for use. This means a programmer using the heap allocation functions can reserve and free memory on the fly. The growth of the heap moves downward toward higher memory addresses.

The _stack segment_ also has variable size and is used as a temporary scratch pad to store local function variables and context during function calls. This is what GDB's backtrace command looks at. When a program calls a function, that function will have its own set of passed variables, and the function's code will be at a different memory location in the text (or code) segment. Since the context and the _EIP_ must change when a function is called, the stack is used to remember all of the passed variables, the location the _EIP_ should return to after the function is finished, and all the local variables used by that function. All of this information is stored together on the stack in what is collectively called a _stack frame_. The stack contains many stack frames.

In general computer science terms, a _stack_ is an abstract data structure that is used frequently. It has _first-in_, _last-out_ (__FILO__) ordering, which means the first item that is put into a stack is the last item to come out of it. Think of it as putting beads on a pice of string that has a knot on one end, you can't get the first bead off until you have removed all the other beads. When an item is placed into a stack, it's known as _pushing_, and when an item is removed from a stack, it's called _popping_.

As the name implies, the stack segment of memory is, in fact, a stack data structure, which contains stack frames. The _ESP_ register is used to keep track of the address of the end of the stack, which is constantly changing as items are pushed onto the into and popped off of it. Since this is very dynamic behavior, it makes sense that the stack is also not of a fixed size. Opposite to the dynamic growth of the heap, as the stack changes in size, it grows upward in a visual listing of memory, toward lower memory addresses.

The _FILO_ nature of a stack might seem odd, but since the stack is used to store context, it's very useful. When a function is called, several things are pushed to the stack together in a _stack frame_. The _EBP_ register, sometimes called the _frame pointer_ (__FP__) or _local base pointer_ (__LB__), is used to reference local function variables in the current stack frame. Each stack frame contains the parameters to the function, its local variables, and two pointers that are necessary to put things back the way they were: The saved frame pointer (_SFP_) and the return address. The _SFP_ is used to restore _EBP_ to its previous value, and the _return address_ is used to restore _EIP_ to the next instruction found after the function call. This restores the functional context of the previous stack frame.

The following _stack_example.c_ code has two functions: _main()_ and _test_function()_.

__stack_example.c__

```c
void test_function(int a, int b, int c, int d)
{
    int flag;
    char buffer[10];

    flag = 31337;
    buffer[0] = 'A';
}

int main()
{
    test_function(1, 2, 3, 4);
}
```

This program first declares a test function that has four arguments, which are all declared as integers, _a_, _b_, _c_, and _d_. The local variables for the function include a single character called _flag_ and a 10-character buffer called _buffer_. The memory for these variables is in the stack segment, while the machine instructions for the function's code is stored in the text segment. After compiling the program, its inner workings can be examined with GDB. The following output shows the disassembled machine instructions for _main()_ and _test_function()_. The _main()_ function starts at _0x08048357_ and _test_function()_ starts at _0x08048344_. The first few instructions of each function (shown in bold below) set up the stack frame. These instructions are collectively called the _procedure prologue_ or _function prologue_. They save the frame pointer on the stack, and they save stack memory for the local function variables. Sometimes the function prologue will handle some stack alignment as well.