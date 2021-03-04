<span style="color:darkblue;font-size:30px;">Introduction </span>

Increase the visibility of a field from private to package, package to protected or protected to public.

Example:

![increasemethod](increasemethod.png)

<span style="color:darkblue;font-size:30px;">Pre and Post Conditions </span>

<span style="color:MidnightBlue;font-size:20px;">Pre Conditions: </span>

1. User must enter the field's name, and the source class's name for the refactoring
   in order to increase the target field's visibility.

<span style="color:MidnightBlue;font-size:20px;">Post Conditions: </span>

No specific Post Condition

<span style="color:darkblue;font-size:30px;">Code</span>
<Pre>
<Code style="display: block; border: 1px solid #999;">
import os
import understand
import networkx as nx

from antlr4 import *
from antlr4.TokenStreamRewriter import TokenStreamRewriter

from gen.java.JavaParser import JavaParser
from gen.javaLabeled.JavaLexer import JavaLexer
from gen.javaLabeled.JavaParserLabeled import JavaParserLabeled
from gen.javaLabeled.JavaParserLabeledListener import JavaParserLabeledListener
from refactorings.Refactoring_action_module_for_big_project import Main_Refactors_Action_for_big_project


class IncreaseFieldVisibilityRefactoringListener(JavaParserLabeledListener):
    <span style="color:green;">
    """
    To implement Increase Field Visibility refactoring based on its actors.
    Detects the required field and increases/changes its visibility status.
    """
    </span>
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
        self.detected_field = None
        self.detected_method = None
        self.TAB = "\t"
        self.NEW_LINE = "\n"
        self.code = ""
        self.tempdeclarationcode = ""

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
        # field_identifier = ctx.variableDeclarators().getText().split(",")
        field_identifier = ctx.variableDeclarators().variableDeclarator(0).variableDeclaratorId().IDENTIFIER().getText()
        if self.field_name in field_identifier:
            if grand_parent_ctx.modifier() == []:
                self.token_stream_rewriter.replaceRange(
                    from_idx=ctx.typeType().start.tokenIndex,
                    to_idx=ctx.typeType().stop.tokenIndex,
                    text='private ' + ctx.typeType().getText()
                )
            elif grand_parent_ctx.modifier(0).getText() == 'public':
                self.token_stream_rewriter.replaceRange(
                    from_idx=grand_parent_ctx.modifier(0).start.tokenIndex,
                    to_idx=grand_parent_ctx.modifier(0).stop.tokenIndex,
                    text='private')
            elif grand_parent_ctx.modifier(0).getText() != 'private':
                self.token_stream_rewriter.replaceRange(
                    from_idx=grand_parent_ctx.modifier(0).start.tokenIndex,
                    to_idx=grand_parent_ctx.modifier(0).stop.tokenIndex,
                    text='private ' + grand_parent_ctx.modifier(0).getText())
            # generate accessor and mutator methods
            # Accessor body
            new_code = '\n\t'
            new_code += 'public ' + ctx.typeType().getText() + ' get' + str.capitalize(self.field_name)
            new_code += '() { \n\t\t return this.' + self.field_name + ';' + '\n\t}'

            # Mutator body
            new_code += '\n\t'
            new_code += 'public void set' + str.capitalize(self.field_name)
            new_code += '(' + ctx.typeType().getText() + ' ' + self.field_name + ') { \n\t\t'
            new_code += 'this.' + self.field_name + ' = ' + self.field_name + ';' + '\n\t}\n'

            self.token_stream_rewriter.insertAfter(ctx.stop.tokenIndex, new_code)

        print("Finished Processing...")


class PropagationIncreaseFieldVisibilityRefactoringListener(JavaParserLabeledListener):

    def __init__(self, common_token_stream: CommonTokenStream = None, using_field_name=None, object_name=None,
                 propagated_class_name=None):

        if using_field_name is None:
            self.using_field_name = []
        else:
            self.using_field_name = using_field_name

        if object_name is None:
            self.object_name = []
        else:
            self.object_name = object_name

        if propagated_class_name is None:
            self.propagated_class_name = []
        else:
            self.propagated_class_name = propagated_class_name

        if common_token_stream is None:
            raise ValueError('common_token_stream is None')
        else:
            self.token_stream_rewriter = TokenStreamRewriter(common_token_stream)

        self.is_class = False

    def enterClassDeclaration(self, ctx: JavaParserLabeled.ClassDeclarationContext):
        # print("Propagation started, please wait...")
        class_identifier = ctx.IDENTIFIER().getText()
        if class_identifier == self.propagated_class_name:
            self.is_class = True
            print("Propagation started, please wait...")
        else:
            self.is_class = False

    def enterVariableDeclarator(self, ctx: JavaParserLabeled.VariableDeclaratorContext):
        if not self.is_class:
            return None
        usingfieldidentifier = ctx.variableDeclaratorId().IDENTIFIER().getText()
        grand_child_ctx = ctx.variableInitializer().expression()
        if usingfieldidentifier in self.using_field_name:
            objectidentifier = grand_child_ctx.expression(0).primary().IDENTIFIER().getText()
            if objectidentifier in self.object_name:
                self.token_stream_rewriter.replaceRange(
                    from_idx=grand_child_ctx.start.tokenIndex,
                    to_idx=grand_child_ctx.stop.tokenIndex,
                    text=grand_child_ctx.expression(0).primary().IDENTIFIER().getText() + '.'
                         + 'get' + str.capitalize(grand_child_ctx.IDENTIFIER().getText()) + '()'
                )

    def enterExpression(self, ctx: JavaParserLabeled.ExpressionContext):
        if not self.is_class:
            return
        if ctx.expression(0) != None:
            if ctx.expression(0).primary() != None:
                if ctx.expression(0).primary().IDENTIFIER().getText() in self.object_name:
                    parent_ctx = ctx.parentCtx
                    count = parent_ctx.getChildCount()
                    if count == 3:
                        expressiontext = parent_ctx.children[2].getText()
                        self.token_stream_rewriter.replaceRange(
                            from_idx=parent_ctx.start.tokenIndex,
                            to_idx=parent_ctx.stop.tokenIndex,
                            text=ctx.expression(0).primary().IDENTIFIER().getText() +
                                 '.' + 'set' + str.capitalize(ctx.IDENTIFIER().getText()) + '(' + expressiontext + ')'
                        )


class PropagationIncreaseFieldVisibility_GetObjects_RefactoringListener(JavaParserLabeledListener):

    def __init__(self, common_token_stream: CommonTokenStream = None, source_class=None,
                 propagated_class_name=None):

        if source_class is None:
            self.source_class = []
        else:
            self.source_class = source_class

        if propagated_class_name is None:
            self.propagated_class_name = []
        else:
            self.propagated_class_name = propagated_class_name

        if common_token_stream is None:
            raise ValueError('common_token_stream is None')
        else:
            self.token_stream_rewriter = TokenStreamRewriter(common_token_stream)

        self.is_class = False
        self.current_class = ''
        self.objects = list()

    def enterClassDeclaration(self, ctx: JavaParserLabeled.ClassDeclarationContext):
        # print("Propagation started, please wait...")
        class_identifier = ctx.IDENTIFIER().getText()
        if class_identifier in self.propagated_class_name:
            self.is_class = True
            print("Propagation started, please wait...")
            self.current_class = class_identifier
        else:
            self.is_class = False

    def enterVariableDeclarator(self, ctx: JavaParserLabeled.VariableDeclaratorContext):
        if not self.is_class:
            return None
        grand_parent_ctx = ctx.parentCtx.parentCtx
        if grand_parent_ctx.typeType().classOrInterfaceType() != None:
            className = grand_parent_ctx.typeType().classOrInterfaceType().IDENTIFIER(0).getText()
            if className in self.source_class:
                objectname = ctx.variableDeclaratorId().IDENTIFIER().getText()
                self.objects.append(objectname)


if __name__ == '__main__':
    print("Increase Field Visibility")
    udb_path = "/home/ali/Documents/compiler/Research/xerces2-j/xerces2-j3.udb"
    class_name = "ListNode"
    field_name = "uri"

    file_list_to_be_propagate = set()
    propagate_classes = set()
    file_list_include_file_name_that_edited = ""
    mainfile = ""
    db = understand.open(udb_path)
    for field in db.ents("public variable"):
        if (str(field) == str(class_name + "." + field_name)):
            # get path file include this field.
            print(field)
            if (field.parent().parent().relname() is not None):
                mainfile = field.parent().parent().longname()
                print(mainfile)
                print(field.parent().parent().longname())
            else:
                for ref in field.refs("Definein"):
                    mainfile = (ref.file().longname())
                    print(mainfile)
                    print(ref.file().relname())
            # get propagate class and their file
            for ref in field.refs("Setby , Useby"):
                if not (str(ref.ent()) == str(field.parent())
                        or str(ref.ent().parent()) == str(field.parent())):
                    propagate_classes.add(str(ref.ent().parent()))
                    file_list_to_be_propagate.add(ref.file().longname())

    file_list_to_be_propagate = list(file_list_to_be_propagate)
    propagate_classes = list(propagate_classes)
    flag_file_is_refatored = False
    corpus = open(
        r"../filename_status_database.txt", encoding="utf-8").read()
    if corpus.find("name:" + mainfile) == -1:
        with open("../filename_status_database.txt", mode='w', encoding="utf-8", newline='') as f:
            f.write(corpus + "\nname:" + mainfile)
            f.flush()
            os.fsync(f.fileno())
        file_list_include_file_name_that_edited += mainfile + "\n"
    else:
        flag_file_is_refatored = True
        print("file already edited")
    print(mainfile)
    stream = FileStream(mainfile, encoding='utf8')
    # Step 2: Create an instance of AssignmentStLexer
    lexer = JavaLexer(stream)
    # Step 3: Convert the input source into a list of tokens
    token_stream = CommonTokenStream(lexer)
    # Step 4: Create an instance of the AssignmentStParser
    parser = JavaParser(token_stream)
    parser.getTokenStream()
    parse_tree = parser.compilationUnit()
    my_listener = IncreaseFieldVisibilityRefactoringListener(common_token_stream=token_stream,
                                                             source_class=class_name,
                                                             field_name=field_name)
    walker = ParseTreeWalker()
    walker.walk(t=parse_tree, listener=my_listener)

    with open(mainfile, "w") as f:
        f.write(my_listener.token_stream_rewriter.getDefaultText())

    print(file_list_to_be_propagate)
    for file in file_list_to_be_propagate:
        flag_file_edited = False
        corpus = open(
            r"filename_status_database.txt", encoding="utf-8").read()
        if (corpus.find("name:" + file) == -1):
            with open("filename_status_database.txt", mode='w', encoding="utf-8", newline='') as f:
                f.write(corpus + "\nname:" + file)
                f.flush()
                os.fsync(f.fileno())
            file_list_include_file_name_that_edited += file + "\n"
        else:
            flag_file_edited = True
        print(file)
        stream = FileStream(file, encoding='utf8')
        # input_stream = StdinStream()
        # Step 2: Create an instance of AssignmentStLexer
        lexer = JavaLexer(stream)
        # Step 3: Convert the input source into a list of tokens
        token_stream = CommonTokenStream(lexer)
        # Step 4: Create an instance of the AssignmentStParser
        parser = JavaParser(token_stream)
        parser.getTokenStream()
        parse_tree = parser.compilationUnit()

        # get object
        my_listener_get_object = PropagationIncreaseFieldVisibility_GetObjects_RefactoringListener(token_stream,
                                                                                                   source_class=class_name,
                                                                                                   propagated_class_name=propagate_classes)
        walker = ParseTreeWalker()
        walker.walk(t=parse_tree, listener=my_listener_get_object)

        my_listener = PropagationIncreaseFieldVisibilityRefactoringListener(common_token_stream=token_stream,
                                                                            using_field_name=field_name,
                                                                            object_name=my_listener_get_object.objects,
                                                                            propagated_class_name=propagate_classes)
        walker = ParseTreeWalker()
        walker.walk(t=parse_tree, listener=my_listener)

        with open(file, "w") as f:
            f.write(my_listener.token_stream_rewriter.getDefaultText())
 </Code>
</Pre>