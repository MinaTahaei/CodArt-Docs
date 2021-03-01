<span style="color:darkblue;font-size:30px;">Introduction </span>

When subclasses grow and get developed separately, identical (or nearly identical) fields and methods appear. 
Pull up method refactoring removes the repetitive field from subclasses and moves it to a superclass.

Example:

![pullupfield](pullupfield.png)

<span style="color:darkblue;font-size:30px;">Pre and Post Conditions </span>

<span style="color:MidnightBlue;font-size:20px;">Pre Conditions: </span>

1. There should exist a corresponding child and parent in the project.
2. The field that should be pulled up must be valid.

<span style="color:MidnightBlue;font-size:20px;">Post Conditions: </span>

1. The changed field's usages and callings will also change respectively.
2. There will be children and parents having their desired fields added or removed.

<span style="color:darkblue;font-size:30px;">Code</span>