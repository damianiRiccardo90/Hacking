# *__Functions__*

Sometimes there will be a set of instructions the programmer knows he will need several times. These instructions can be grouped into a smaller sub-program called a __function__. In other languages, functions are known as sub-routines or procedures. For example, the action of turning a car actually consists of many smaller instructions: Turn on the appropriate blinker, slow down, check for oncoming traffic, turn the steering wheel in the appropriate direction, and so on. The driving directions from the beginning of this chapter require quite a few turns; however, listing every little instruction for every turn would be tedious (and less readable). You can pass variable as arguments to a function in order to modify the way the function operates. In this case, the function is passed the direction of the turn.

<pre style="color: white;">
Function Turn(variable_direction)
{
    Activate the variable_direction blinker;
    Slow down;
    Check for oncoming traffic;
    while(there is oncoming traffic)
    {
        Stop;
        Watch for oncoming traffic;
    }
    Turn the steering wheel to the variable_direction;
    while(turn is not complete)
    {
        if(speed < 5 mph>)
            Accelerate;
    }
    Turn the steering wheel back to the original position;
    Turn off the variable_direction blinker;
}
</pre>

This function describes all the instructions needed to make a turn. When a program that knows about this function needs to turn, it can just call this function. When the function is called, the instructions found within it are executed with the arguments passed to it; afterward, execution returns to where it was in the program, after the function call. Either left or right can be passed into this function, which caused the function to runt in that direction.

By default in C, functions can return a value to a caller. For those familiar with functions in mathematics, this makes perfect sense. Imagine a function that calculates the factorial of a number, naturally, it returns the result.

In C, functions aren't labeled with a "function" keyword; instead, they are declared by the data type of the variable they are returning. This format looks very similar to variable declaration. If a function is meant to return an integer (perhaps a function that calculates the factorial of some number _x_), the function could look like this:

```c
int factorial(int x)
{
    int i;
    for(i = 1; i < x; i++)
        x *= i;
    return x;
}
```

This function is declared as an integer because it multiplies every value from 1 to __x__ and returns the result, which is an integer. The return statement at the end of the function passes back the contents of the variable __x__ and ends the function. This factorial function can then be used like an integer variable in the main part of any program that knows about it.

```c
int a = 5, b;
b = factorial(a);
```

At the end of this short program, the variable __b__ will contain 120, since the factorial function will be called with the argument of 5 and will return 120. Also in C, the compiler must "know" about functions before it can use them. This can be done by simply writing the entire function before using it later in the program or by using function prototypes. A _function prototype_ is simply a way to tell the compiler to expect a function with this name, this return data type, and these data types as its functional arguments. The actual function can be located near the end of the program, but it can be used anywhere else, since the compiler already knows about it. An example of a function prototype for the _factorial()_ function would look something like this:

```c
int factorial(int);
```

Usually, function prototypes are located near the beginning of a program. There's no need to actually define any variable names in the prototype, since this is done in the actual function. The only thing the compiler cares about is the function's name, its return data type, and the data types of its functional arguments.

If a function doesn't have any value to return, it should be declared as __void__, as is the case with the _turn()_ function I used as an example earlier. However, the _turn()_ function doesn't yet capture all the functionality that our driving directions need. Every turn in the directions has both a direction and a street name. This means that a turning function should have two variables: The direction to turn and the street to turn on to. This complicates the function of turning, since the proper street must be located before the turn can be made. A more complete turning function using proper C-like syntax is listed below in pseudo-code.

<pre style="color: white;">
void turn(variable_direction, target_street_name)
{
    Look for a street sign;
    current_intersection_name = read street sign name;
    while(current_intersection_name != target_street_name)
    {
        Look for another street sign;
        current_intersection_name = read street sign name;
    }

    Activate the variable_direction blinker;
    Slow down;
    Check for oncoming traffic;
    while(there is oncoming traffic)
    {
        Stop;
        Watch for oncoming traffic;
    }
    Turn the steering wheel to the variable_direction;
    while(turn is not complete)
    {
        if(speed < 5 mph>)
            Accelerate;
    }
    Turn the steering wheel right back to the original position;
    Turn off the variable_direction blinker;
}
</pre>

This function includes a section that searches for the proper intersection by looking for street signs, reading the name on each street sign, and storing that name in a variable called __current_intersection_name__. It will continue to look for and read street signs until the target street is found; at that point, the remaining turning instructions will be executed. The pseudo-code driving instructions can now be changed to use this turning function.

<pre style="color: white;">
Begin going East on Main Street;
while (there is not a church on the right)
    Drive down Main Street;
if (street is blocked)
{
    Turn(right, 15th Street);
    Turn(left, Pine Street);
    Turn(right, 16th Street);
}
else
    Turn(right, 16th Street);
Turn(left, Destination Road);
for (i = 0; i < 5; i++)
    Drive straight for 1 mile;
Stop at 743 Destination Road;
</pre>

Functions aren't commonly used in pseudo-code, since pseudo-code is mostly used as a way for programmers to sketch out program concepts before writing compilable code. Since pseudo-code doesn't actually have to work, full functions don't need to be written out, simply jotting down _Do some complex stuff here_ will suffice. But in a programming language like C, functions are used heavily. Most of the real usefulness of C comes from collections of existing functions called libraries.