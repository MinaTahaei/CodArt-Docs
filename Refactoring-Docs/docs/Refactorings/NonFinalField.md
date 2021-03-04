<span style="color:darkblue;font-size:30px;">Introduction </span>

Add the "final" property to a field, so that it never changes once it's initialized.

Example:

![makefinalnonfield](makefieldnonfinal.png)

<span style="color:darkblue;font-size:30px;">Pre and Post Conditions </span>

<span style="color:MidnightBlue;font-size:20px;">Pre Conditions: </span>

1. User must enter the field's name in order to make it non-final

<span style="color:MidnightBlue;font-size:20px;">Post Conditions: </span>

No specific Post Condition

<span style="color:darkblue;font-size:30px;">Code</span>
<Pre>
<Code style="display: block; border: 1px solid #999;">
import understand
from antlr4 import *
from antlr4.TokenStreamRewriter import TokenStreamRewriter

from gen.java.JavaParser import JavaParser
from gen.javaLabeled.JavaLexer import JavaLexer
from gen.javaLabeled.JavaParserLabeled import JavaParserLabeled
from gen.javaLabeled.JavaParserLabeledListener import JavaParserLabeledListener


class MakeFieldNonFinalRefactoringListener(JavaParserLabeledListener):
    """
    To implement extract class refactoring based on its actors.
    Creates a new class and move fields and methods from the old class to the new one
    """

    def __init__(self, common_token_stream: CommonTokenStream = None, source_class=None, field_name: str = None):

        if field_name is None:
            self.field_name = ""
        else:
            self.field_name = field_name

        if source_class is None:
            self.source_class = ""
        else:
            self.source_class = source_class
        if common_token_stream is None:
            raise ValueError('common_token_stream is None')
        else:
            self.token_stream_rewriter = TokenStreamRewriter(common_token_stream)

        self.is_source_class = False
        self.is_final = False

    def enterClassDeclaration(self, ctx: JavaParserLabeled.ClassDeclarationContext):
        print("Refactoring started, please wait...")
        class_identifier = ctx.IDENTIFIER().getText()
        if class_identifier == self.source_class:
            self.is_source_class = True
        else:
            self.is_source_class = False

    def exitFieldDeclaration(self, ctx: JavaParserLabeled.FieldDeclarationContext):

        if not self.is_source_class:
            return None
        grand_parent_ctx = ctx.parentCtx.parentCtx
        field_identifier = ctx.variableDeclarators().variableDeclarator(0).variableDeclaratorId().IDENTIFIER().getText()
        print("field_identifier :", field_identifier)
        if self.field_name in field_identifier:

            if not (grand_parent_ctx.modifier() == []):
                for i in range(0, len(grand_parent_ctx.modifier())):
                    if grand_parent_ctx.modifier(i).getText() == "final":
                        self.is_final = True
                        break
                print("-----------------------", self.is_final)
                if self.is_final:
                    self.token_stream_rewriter.replaceRange(
                        from_idx=grand_parent_ctx.modifier(i).start.tokenIndex,
                        to_idx=grand_parent_ctx.modifier(i).stop.tokenIndex,
                        text=''
                    )

        print("Finished Processing...")


if __name__ == '__main__':
    db_path = "/home/ali/Documents/compiler/Research/xerces2-j/xerces2-j.udb"
    class_name = "DesignDoc"
    field_name = "GENERATOR_NAME"
    mainfile = ""

    db = understand.open(db_path)
    for cls in db.ents("class"):
        if (cls.simplename() == class_name):
            if cls.kindname() != "Unknown Class":
                mainfile = cls.parent().longname()

    stream = FileStream(mainfile, encoding='utf8')
    # Step 2: Create an instance of AssignmentStLexer
    lexer = JavaLexer(stream)
    # Step 3: Convert the input source into a list of tokens
    token_stream = CommonTokenStream(lexer)
    # Step 4: Create an instance of the AssignmentStParser
    parser = JavaParser(token_stream)
    parser.getTokenStream()
    parse_tree = parser.compilationUnit()
    my_listener = MakeFieldNonFinalRefactoringListener(common_token_stream=token_stream, source_class=class_name,
                                                       field_name=field_name)
    walker = ParseTreeWalker()
    walker.walk(t=parse_tree, listener=my_listener)

    with open(mainfile, mode='w', newline='') as f:
        f.write(my_listener.token_stream_rewriter.getDefaultText())
</Code>
</Pre>