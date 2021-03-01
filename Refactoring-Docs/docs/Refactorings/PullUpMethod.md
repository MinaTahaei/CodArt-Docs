<span style="color:darkblue;font-size:30px;">Introduction </span>

When subclasses grow and get developed separately, your code may have methods that perform similar work.
Pull up method refactoring removes the repetitive method from subclasses and moves it to a superclass.

Example:

![pullupmethod](pullupmethod.png)

<span style="color:darkblue;font-size:30px;">Pre and Post Conditions </span>

<span style="color:MidnightBlue;font-size:20px;">Pre Conditions: </span>

1. The source package, class and method should exist.
2. If the method uses attributes and methods that are defined in the body of the classes, The refactoring should not be done.

<span style="color:MidnightBlue;font-size:20px;">Post Conditions: </span>

No specific Post Condition

<span style="color:darkblue;font-size:30px;">Code</span>