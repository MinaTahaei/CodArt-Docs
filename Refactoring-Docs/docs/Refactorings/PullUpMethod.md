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
<Pre>
<Code style="display: block; border: 1px solid #999;">
from antlr4.TokenStreamRewriter import TokenStreamRewriter
from refactorings.utils.utils_listener_fast import TokensInfo, SingleFileElement
from refactorings.pullup_method_get_removemethod import get_removemethods
from refactorings.utils.utils2 import Rewriter, get_program, get_filenames_in_dir


def pullup_method_refactoring(source_filenames: list, package_name: str, class_name: str, method_key: str,
                              filename_mapping=lambda x: x):
    program = get_program(source_filenames)  # گرفتن پکیج های برنامه
    _sourceclass = program.packages[package_name].classes[class_name]
    target_class_name = _sourceclass.superclass_name
    static = 0
    removemethod = get_removemethods(program, package_name, target_class_name, method_key,
                                     class_name)  # متد های مشابه در کلاس های دیگر
    _targetclass = program.packages[package_name].classes[target_class_name]
    _method_name = program.packages[package_name].classes[class_name].methods[method_key]
    tokens_info = TokensInfo(_method_name.parser_context)
    exps = tokens_info.get_token_index(tokens_info.token_stream.tokens, tokens_info.start,
                                       tokens_info.stop)  # لیست متغیر های داخل بدنه کلاس که داخل متد استفاده شده اند
    if _method_name.is_constructor:
        return False
    # if method use param of class body return false
    for token in exps:
        if token.text in _sourceclass.fields:
            return False

    if bool(_method_name.body_method_invocations_without_typename) == True:
        return False

    Rewriter_ = Rewriter(program, filename_mapping)
    for remove in removemethod:
        _methodd = removemethod[remove]
        if _methodd != None:
            _methodds = _methodd[0]
            _method = program.packages[package_name].classes[remove].methods[str(_methodds)]
            _method_token_info = TokensInfo(_method.parser_context)
            Rewriter_.replace(_method_token_info, " ")

    class_tokens_info = TokensInfo(_targetclass.parser_context)
    singlefileelement = SingleFileElement(_method_name.parser_context, _method_name.filename)
    token_stream_rewriter = TokenStreamRewriter(singlefileelement.get_token_stream())
    strofmethod = token_stream_rewriter.getText(program_name=token_stream_rewriter.DEFAULT_PROGRAM_NAME,
                                                start=tokens_info.start,
                                                stop=tokens_info.stop)
    Rewriter_.insert_before(tokens_info=class_tokens_info, text=strofmethod)
    Rewriter_.apply()
    # در کلاس های دیگر هر جا که از این متد استفاده شده باید اپدیت شود
    for package_names in program.packages:
        package = program.packages[package_names]
        for class_ in package.classes:
            _class = package.classes[class_]
            for method_ in _class.methods:
                __method = _class.methods[method_]
                for inv in __method.body_method_invocations:
                    invc = __method.body_method_invocations[inv]
                    method_name = method_key[:method_key.find('(')]
                    if (invc[0] == method_name & package_names == package_name):
                        inv_tokens_info = TokensInfo(inv)
                        if (static == 0):
                            class_token_info = TokensInfo(_class.body_context)
                            Rewriter_.insert_after_start(class_token_info, target_class_name + " " + str.lower(
                                target_class_name) + "=" + "new " + target_class_name + "();")
                            Rewriter_.apply()
                        Rewriter_.replace(inv_tokens_info, target_class_name)
                        Rewriter_.apply()
    return True


if __name__ == "__main__":
    mylist = get_filenames_in_dir('/home/ali/Desktop/code/TestProject/src/')
    print("Testing pullup_method...")
    if pullup_method_refactoring(mylist, "test_package", "AppChild1", "methodToPullUp()"):
        print("Success!")
    else:
        print("Cannot refactor.")
</Code>
</Pre>