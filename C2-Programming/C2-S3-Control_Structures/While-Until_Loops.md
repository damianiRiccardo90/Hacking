# *__While/Until Loops__*

Another elementary programming concept is the while control structure, which is a type of loop. A programmer will often want to execute a set of instructions more than once. A program can accomplish this task through looping, but it requires a set of conditions that tells it when to stop looping, lest it continue into infinity. A __while loop__ says to execute the following set of instructions in a loop _while_ a condition is true. A simple program for a hungry mouse could look something like this:

<pre style="color: white;">
While (you are hungry)
{
    Find some food;
    Eat the food;
}
</pre>

The set of two instructions following the while statement will be repeated _while_ the mouse is still hungry. The amount of food the mouse finds each time could range from a tiny crumb to an entire loaf of bread. Similarly, the number of times the set of instructions in the while statement is executed changes depending on how much food the mouse finds.

Another variation on the while loop is an until loop, a syntax that is available in the programming language Perl (C doesn't use this syntax). An __until loop__ is simply a while loop with the conditional statement inverted. The same mouse program using an until loop would be:

<pre style="color: white;">
Until (your are not hungry)
{
    Find some food;
    Eat the food;
}
</pre>

Logically, any until-like statement can be converted into a while loop. The driving directions from before contained the statement _Continue on Main Street until you see a church on your right_. This can easily be changed into a standard while loop by simply inverting the condition.

<pre style="color: white;">
While (there is not a church on the right)
    Drive down Main Street;
</pre>