# *__Structs__*

Sometimes there are multiple variables that should be grouped together and treated like one. In C, __structs__ are variables that can contain many other variables. Structs are often used by various system functions and libraries, so understanding how to use structs is a prerequisite to using these functions.

A simple example will suffice for now. When dealing with many time functions,
these functions use a time struct called __tm__, which is defined in _/usr/include/
time.h_. The struct’s definition is as follows.

```c
struct tm {
int     tm_sec;     /* seconds */
int     tm_min;     /* minutes */
int     tm_hour;    /* hours */
int     tm_mday;    /* day of the month */
int     tm_mon;     /* month */
int     tm_year;    /* year */
int     tm_wday;    /* day of the week */
int     tm_yday;    /* day in the year */
int     tm_isdst;   /* daylight saving time */
};
```

After this struct is defined, `struct tm` becomes a usable variable type, which can be used to declare variables and pointers with the data type of the tm struct. The _time_example.c_ program demonstrates this. When _time.h_ is included, the __tm__ struct is defined, which is later used to declare the __current_time__ and __time_ptr__ variables.

__time_example.c__

```c
#include <stdio.h>
#include <time.h>

int main() 
{
    long int seconds_since_epoch;
    struct tm current_time, *time_ptr;
    int hour, minute, second, day, month, year;

    seconds_since_epoch = time(0); // Pass time a null pointer as argument.
    printf("time() - seconds since epoch: %ld\n", seconds_since_epoch);

    time_ptr = &current_time; // Set time_ptr to the address of
                              // the current_time struct.
    localtime_r(&seconds_since_epoch, time_ptr);

    // Three different ways to access struct elements:
    hour = current_time.tm_hour; // Direct access
    minute = time_ptr->tm_min; // Access via pointer
    second = *((int*) time_ptr); // Hacky pointer access

    printf("Current time is: %02d:%02d:%02d\n", hour, minute, second);
}
```

The _time()_ function will return the number of seconds since January 1, 1970. Time on Unix systems is kept relative to this rather arbitrary point in time, which is also known as the __epoch__. The _localtime_r()_ function expects two pointers as arguments: one to the number of seconds since epoch and the other to a __tm__ struct. The pointer _time_ptr_ has already been set to the address of current_time, an empty __tm__ struct. The address-of operator is used to provide a pointer to __seconds_since_epoch__ for the other argument to _localtime_r()_, which fills the elements of the __tm__ struct. The elements of structs can be accessed in three different ways; the first two are the proper ways to access struct elements, and the third is a hacked solution. If a struct variable is used, its elements can be accessed by adding the elements’ names to the end of the variable name with a period. Therefore, `current_time.tm_hour` will access just the __tm_hour__ element of the __tm__ struct called __current_time__. Pointers to structs are often used, since it is much more efficient to pass a four-byte pointer than an entire data structure. Struct pointers are so common that C has a built-in method to access struct elements from a struct pointer without needing to dereference the pointer.