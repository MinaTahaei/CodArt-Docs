<span style="color:darkblue;font-size:30px;">Introduction </span>

Although it was planned to use a field universally for all classes, in reality the field is used only in
some subclasses. This situation can occur when planned features fail to pan out, for example.
because of this, we push down the field from the superclass into its related subclass.

Example:

![pushdownfield](pushdownfield.png)

<span style="color:darkblue;font-size:30px;">Pre and Post Conditions </span>

<span style="color:MidnightBlue;font-size:20px;">Pre Conditions: </span>

1. There should exist a corresponding child and parent in the project.
2. The field that should be pushed down must be valid.

<span style="color:MidnightBlue;font-size:20px;">Post Conditions: </span>

1. The changed field's usages and callings will also change respectively.
2. There will be children and parents having their desired fields added or removed.

<span style="color:darkblue;font-size:30px;">Code</span>