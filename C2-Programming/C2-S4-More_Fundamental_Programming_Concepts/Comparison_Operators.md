# *__Comparison Operators__*

Variables are frequently used in the conditional statements of the previously explained control structures. These conditional statements are based on some sort of comparison. In C, these comparison operators use a shorthand syntax that is fairly common across many programming languages.

<div align="left" width="100%">
<img src="Comparison_Operators.png?raw=true" alt="Comparison Operators" width="50%">
</div>

Most of these operators are self-explanatory; however, notice that the shorthand for _equal to_ uses double equal signs. This is an important distinction, since the double equal sign is used to test equivalence, while the single equal sign is used to assign a value to a variable. The statement _a = 7_ means _Put the value 7 in the variable __a___, while _a == 7_ means _Check to see whether the variable __a__ is equal to 7_. (Some programming languages like Pascal actually use __:=__ for variable assignment to eliminate visual confusion.) Also, notice that an exclamation point generally means _not_. This symbol can be used by itself to invert any expression.

> !(a < b) is equivalent to (a >= b)

These comparisons operators can also be chained together using shorthand for __OR__ and __AND__.

Boolean_Operators
<div align="left" width="100%">
<img src="Boolean_Operators.png?raw=true" alt="Boolean Operators" width="50%">
</div>

The example statement consisting of the two smaller conditions join ed with OR logic will fire true if __a__ is less than __b__, OR if __a__ is less than __c__. Similarly, the example statement consisting of two smaller comparisons joined with AND logic will fire true if __a__ is less than __b__ AND __a__ is not less than __c__. These statements should be grouped with parentheses and can contain many different variations.

Many things can be boiled down to variables, comparison operators, and control structures. Returning to the example of the mouse searching for food, hunger can be translated into a Boolean true/false variable. Naturally, 1 means true and 0 means false.

<pre style="color: white;">
While (hungry == 1)
{
    Find some food;
    Eat the food;
}
</pre>

Here's another shorthand used by programmers and hackers quite often. C doesn't really have any Boolean operators, so any nonzero value is considered true, and a statement is considered false if it contains 0. In fact, the comparison operator will actually return a value of 1 if the comparison is true and a value of 0 if it is false. Checking to see whether the variable __hungry__ is equal to 1 will return 1 if __hungry__ equals 1 and 0 if __hungry__ equals 0. Since the program only uses these two cases, the comparison operator can be dropped altogether.

<pre style="color: white;">
While (hungry)
{
    Find some food;
    Eat the food;
}
</pre>

A smarter mouse program with more inputs demonstrates how comparison operators can be combined with variables.

<pre style="color: white;">
While ((hungry) && !(cat_present))
{
    Find some food;
    If(!(food_is_on_a_mousetrap))
        Eat the food;
}
</pre>

This example assumes there are also variables that describe the presence of a cat and the location of the food, with a value of 1 for true and 0 for false. Just remember that any nonzero value is considered true, and the value of 0 is considered false.