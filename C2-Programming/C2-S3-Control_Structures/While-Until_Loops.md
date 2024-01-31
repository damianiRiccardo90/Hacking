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

### *__For Loops__*

Another looping control structure is the __for loop__. This is generally used when a programmer wants to loop for a certain number of iterations. The driving direction _Drive straight down Destination Road for 5 miles_ could be converted to a for loop that looks something like this:

<pre style="color: white;">
For (5 iterations)
    Drive straight for 1 mile;
</pre>

In reality, a for loop is just a while loop with a counter. The same statement can be written as such:

<pre style="color: white;">
Set the counter to 0;
While (the counter is less than 5)
{
    Drive straight for 1 mile;
    Add 1 to the counter;
}
</pre>

The C-like pseudo-code syntax of a for loop makes this even more apparent:

<pre style="color: white;">
For (i=0; i<5; i++)
    Drive straight for 1 mile;
</pre>

In this case, the counter is called _i_, and the for statement is broken up into three sections, separated by semicolons. The first section declares the counter and sets it to its initial value, in this case 0. The second section is like a while statement using the counter, _While_ the counter meets this conditions, keep looping. The third and final section describes what action should be taken on the counter during each iteration. In this case, _i++_ is a shorthand way of saying, _Add 1 to the counter called i_.

Using all of the control structures, the driving directions from page 6 can be converted into a C-like pseudo-code that looks something like this:

<pre style="color: white;">
Begin going East on Main Street;
While (there is not a church on the right)
    Drive down Main Street;
If (street is blocked)
{
    Turn right on 15th Street;
    Turn left on Pine Street;
    Turn right on 16th Street;
}
Else
    Turn right on 16th Street;
Turn left on Destination Road;
For (i=0; i<5; i++)
    Drive straight for 1 mile;
Stop at 743 Destination Road;
</pre>