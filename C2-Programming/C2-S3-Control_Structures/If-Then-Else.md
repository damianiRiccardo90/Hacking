# *__If-Then-Else__*

In the case of our driving directions, Main Street could be under construction. If it is, a special set of instructions needs to address that situation. Otherwise, the original set of instructions should be followed. These types of special cases can be accounted for in a program with one of the most natural control structures: The _if-then-else structure_. In general, it looks something like this:

<pre style="color: white;">
If (condition) then
{
    Set of instructions to execute if the condition is met;
}
Else
{
    Set of instructions to execute if the condition is not met;
}
</pre>

For this book, a C-like pseudo-code will be used, so every instruction will end with a semicolon, and the sets of instructions will be grouped with curly braces and indentation. The if-then-else pseudo-code structure of the preceding driving directions might look something like this:

<pre style="color: white;">
Drive down Main Street;
If (street is blocked)
{
    Turn right on 15th Street;
    Turn left on Pine Street;
    Turn right on 16th Street;
}
Else
{
    Turn right on 16th Street;
}
</pre>

Each instruction is on its own line, and the various sets of conditional instructions are grouped between curly braces and indented for readability.
In C and many other programming languages, the __then__ keyword is implied and therefore left out, so it has also been omitted in the preceding pseudo-code.
Of course, other languages require the _then_ keyword in their syntax, BASIC, Fortran, and even Pascal, for example. These types of syntactical differences in programming languages are only skin dep; the underlying structure is still the same. Once a programmer understands the concepts these languages are trying to convey, learning the various syntactical variations is fairly trivial. Since C will be used the later sections, the pseudo-code used in this book will follow a C-like syntax, but remember that pseudo-code can take on many forms.

Another common rule of C-like syntax is when a set of instructions bounded by curly braces consists of just one instruction, the curly braces are optional. Fro the sake of readability, it's still a good idea to indent these instructions, but it's not syntactically necessary. The driving directions from before can be rewritten following this rule to produce an equivalent piece of pseudo-code:

<pre style="color: white;">
Drive down Main Street;
If (street is blocked)
{
    Turn right on 15th Street;
    Turn left on Pine Street;
    Turn right on 16th Street;
}
Else
    Turn right on 16th Street;
</pre>

This rule is about sets of instructions holds true for all of the control structures mentioned in this book, and the rule itself can be described in pseudo-code.

<pre style="color: white;">
If (there is only one instruction in a set of instructions)
    The use of curly braces to group the instructions is optional;
Else
{
    The use of curly braces is necessary;
    Since there must be a logical way to group these instructions;
}
</pre>

Even the description of a syntax itself can be thought of as a simple program. There are variations of if-then-else, such as select/case statements, but the logic is still basically the same: If this happens do these things, otherwise do these other things (which could consist of even more if-then statements).