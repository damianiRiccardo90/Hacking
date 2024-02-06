# *__Error-Checked malloc()__*

In _heap_example.c_, there were several error checks for the _malloc()_ calls. Even though the _malloc()_ calls never failed, it's important to handle all potential cases when coding in C. But with multiple _malloc()_ calls, this error-checking code needs to appear in multiple places. This usually makes the code look sloppy, and it's inconvenient if changes need to be made to the error-checking code or if new _malloc()_ calls are needed. Since all the error-checking code is basically the same for every _malloc()_ call, this is a perfect place to use a function instead of repeating the same instructions in multiple places. Take a look at _errorchecked_heap.c_ for an example.

__errorchecked_heap.c__

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Function prototype for errorchecked_malloc()
void* errorchecked_malloc(unsigned int);

int main(int argc, char* argv[]) 
{
    char* char_ptr; // A char pointer
    int* int_ptr; // An integer pointer
    int mem_size;
    
    if (argc < 2)      // If there aren't command-line arguments,
        mem_size = 50; // use 50 as the default value.
    else
        mem_size = atoi(argv[1]);

    printf("\t[+] allocating %d bytes of memory on the heap for char_ptr\n", mem_size);
    char_ptr = (char*) errorchecked_malloc(mem_size); // Allocating heap memory

    strcpy(char_ptr, "This is memory is located on the heap.");
    printf("char_ptr (%p) --> '%s'\n", char_ptr, char_ptr);
    printf("\t[+] allocating 12 bytes of memory on the heap for int_ptr\n");
    int_ptr = (int*) errorchecked_malloc(12); // Allocated heap memory again

    *int_ptr = 31337; // Put the value of 31337 where int_ptr is pointing.
    printf("int_ptr (%p) --> %d\n", int_ptr, *int_ptr);

    printf("\t[-] freeing char_ptr's heap memory...\n");
    free(char_ptr); // Freeing heap memory

    printf("\t[+] allocating another 15 bytes for char_ptr\n");
    char_ptr = (char*) errorchecked_malloc(15); // Allocating more heap memory

    strcpy(char_ptr, "new memory");
    printf("char_ptr (%p) --> '%s'\n", char_ptr, char_ptr);

    printf("\t[-] freeing int_ptr's heap memory...\n");
    free(int_ptr); // Freeing heap memory
    printf("\t[-] freeing char_ptr's heap memory...\n");
    free(char_ptr); // Freeing the other block of heap memory
}

// An error-checked malloc() function
void* errorchecked_malloc(unsigned int size) 
{
    void* ptr;
    ptr = malloc(size);
    if (ptr == NULL) 
    {
        fprintf(stderr, "Error: could not allocate heap memory.\n");
        exit(-1);
    }
    return ptr;
}
```

The _errorchecked_heap.c_ program is basically equivalent to the previous _heap_example.c_ code, except the heap memory allocation and error checking has been gathered into a single function. The first line of code _[void* errorchecked_malloc(unsigned int);]_ is the function prototype. This lets the compiler know that there will be a function called _errorchecked_malloc()_ that excepts a single, unsigned integer argument and returns a _void_ pointer. The actual function can then be anywhere; in this case it is after the _main()_ function. The function itself is quite simple; it just accepts the size in bytes to allocate and attempts to allocate that much memory using _malloc()_. If the allocation fails, the error-checking code displays an error and the program exits; otherwise, it returns the pointer to the newly allocated heap memory. This way, the custom _errorchecked_malloc()_ function can be used in place of a normal _malloc()_, eliminating the need for repetitious error checking afterward. This should begin to highlight the usefulness of programming with functions.