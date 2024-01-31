# *__Variable Scoping__*

Another interesting concept regarding memory in C is variable scoping or context, in particular, the contexts of variables within functions. Each function has its own set of local variables, which are independent of everything else. In fact, multiple calls to the same function all have their own contexts. You can use the _printf()_ function with format strings to quickly explore this; check it out in _scope.c_.

__scope.c__

```c
#include <stdio.h>

void func3() 
{
    int i = 11;
    printf("\t\t\t[in func3] i = %d\n", i);
}

void func2() 
{
    int i = 7;
    printf("\t\t[in func2] i = %d\n", i);
    func3();
    printf("\t\t[back in func2] i = %d\n", i);
}

void func1() 
{
    int i = 5;
    printf("\t[in func1] i = %d\n", i);
    func2();
    printf("\t[back in func1] i = %d\n", i);
}

int main() 
{
    int i = 3;
    printf("[in main] i = %d\n", i);
    func1();
    printf("[back in main] i = %d\n", i);
}
```

The output of this simple program demonstrates nested function calls.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc scope.c
reader@hacking:~/booksrc $ ./a.out
[in main] i = 3
        [in func1] i = 5
                [in func2] i = 7
                        [in func3] i = 11
                [back in func2] i = 7
        [back in func1] i = 5
[back in main] i = 3
reader@hacking:~/booksrc $
</pre>

In each function, the variable __i__ is set to a different value and printed. Notice that within the _main()_ function, the variable _i_ is _3_, even after calling _func1()_ where the variable _i_ is _5_. Similarly, within _func1()_ the variable _i_ remains _5_, even after calling _func2()_ where _i_ is _7_, and so forth. The best way to think of this is that eah function call has its own version of the variable _i_.

Variables can also have a global scope, which means they will persist across all functions. Variables are global if they are defined at the beginning of the code, outside of any functions. In the _scope2.c_ example code shown below, the variable _j_ is declared globally and set to _42_. This variable can be read from and written to by any function, and the changes to it will persist between functions.

___scope2__

```c
#include <stdio.h>

int j = 42; // j is a global variable.

void func3() 
{
    int i = 11, j = 999; // Here, j is a local variable of func3().
    printf("\t\t\t[in func3] i = %d, j = %d\n", i, j);
}

void func2() 
{
    int i = 7;
    printf("\t\t[in func2] i = %d, j = %d\n", i, j);
    printf("\t\t[in func2] setting j = 1337\n");
    j = 1337; // Writing to j
    func3();
    printf("\t\t[back in func2] i = %d, j = %d\n", i, j);
}

void func1() 
{
    int i = 5;
    printf("\t[in func1] i = %d, j = %d\n", i, j);
    func2();
    printf("\t[back in func1] i = %d, j = %d\n", i, j);
}

int main() 
{
    int i = 3;
    printf("[in main] i = %d, j = %d\n", i, j);
    func1();
    printf("[back in main] i = %d, j = %d\n", i, j);
}
```

The results of compiling and executing _scope2.c_ are as follows.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc scope2.c
reader@hacking:~/booksrc $ ./a.out
[in main] i = 3, j = 42
        [in func1] i = 5, j = 42
                [in func2] i = 7, j = 42
                [in func2] setting j = 1337
                        [in func3] i = 11, j = 999
                [back in func2] i = 7, j = 1337
        [back in func1] i = 5, j = 1337
[back in main] i = 3, j = 1337
reader@hacking:~/booksrc $
</pre>

In the output, the global variable _j_ is written to in _func2()_, and the change persists in all functions except _func3()_, which has its own local variable called _j_. In this case, the compiler prefers to use the local variable. With all these variables using the same names, it can be a little confusing, but remember that in the end, it's all just memory. The global variable _j_ is just stored in memory, and every function is able to access that memory. The local variables for each function are each stored in their own places in memory, regardless of the identical names. Printing the memory addresses of these variables will give a clearer picture of what's going on, In the _scope3.c_ example code below, the variable addresses are printed using then unary address-of operator.

__scope3.c__

```c
#include <stdio.h>

int j = 42; // j is a global variable.

void func3() 
{
    int i = 11, j = 999; // Here, j is a local variable of func3().
    printf("\t\t\t[in func3] i @ 0x%08x = %d\n", &i, i);
    printf("\t\t\t[in func3] j @ 0x%08x = %d\n", &j, j);
}

void func2() 
{
    int i = 7;
    printf("\t\t[in func2] i @ 0x%08x = %d\n", &i, i);
    printf("\t\t[in func2] j @ 0x%08x = %d\n", &j, j);
    printf("\t\t[in func2] setting j = 1337\n");
    j = 1337; // Writing to j
    func3();
    printf("\t\t[back in func2] i @ 0x%08x = %d\n", &i, i);
    printf("\t\t[back in func2] j @ 0x%08x = %d\n", &j, j);
}

void func1() 
{
    int i = 5;
    printf("\t[in func1] i @ 0x%08x = %d\n", &i, i);
    printf("\t[in func1] j @ 0x%08x = %d\n", &j, j);
    func2();
    printf("\t[back in func1] i @ 0x%08x = %d\n", &i, i);
    printf("\t[back in func1] j @ 0x%08x = %d\n", &j, j);
}

int main() 
{
    int i = 3;
    printf("[in main] i @ 0x%08x = %d\n", &i, i);
    printf("[in main] j @ 0x%08x = %d\n", &j, j);
    func1();
    printf("[back in main] i @ 0x%08x = %d\n", &i, i);
    printf("[back in main] j @ 0x%08x = %d\n", &j, j);
}
```

The results of compiling and executing _scope3.c_ are as follows.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc scope3.c
reader@hacking:~/booksrc $ ./a.out
[in main] i @ 0xbffff834 = 3
[in main] j @ 0x08049988 = 42
        [in func1] i @ 0xbffff814 = 5
        [in func1] j @ 0x08049988 = 42
                [in func2] i @ 0xbffff7f4 = 7
                [in func2] j @ 0x08049988 = 42
                [in func2] setting j = 1337
                        [in func3] i @ 0xbffff7d4 = 11
                        [in func3] j @ 0xbffff7d0 = 999
                [back in func2] i @ 0xbffff7f4 = 7
                [back in func2] j @ 0x08049988 = 1337
        [back in func1] i @ 0xbffff814 = 5
        [back in func1] j @ 0x08049988 = 1337
[back in main] i @ 0xbffff834 = 3
[back in main] j @ 0x08049988 = 1337
reader@hacking:~/booksrc $
</pre>

In this output, it is obvious that the variable _j_ used by _func3()_ is different than the _j_ used by the other functions. The _j_ used by _func3()_ is located at _0xbffff7d0_, while the _j_ used by the other functions is located at _0x08049988_. Also, notice that the variable _i_ is actually a different memory address for each function.

In the following output, GDB is used to stop execution 