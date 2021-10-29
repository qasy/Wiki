Для того чтобы отключить форматирование для части кода (полезно для хедеров, потому что важен порядок иногда):  
>**// clang-format off**  
>#include <glad/glad.h>  
>#include <GLFW/glfw3.h>  
>#include <iostream>  
>**// clang-format on**  
  
ну или более понятный пример:  
>**// clang-format off**  
>#include <b.h>  
>#include <a.h>  
>#include <c.h>  
>**// clang-format on**  
>#include <d.h>  
>#include <e.h>  

Путь к файлу прописывается в **settings.json**:  
Например:  
>"editor.formatOnSave": true,  
>   "clang-format.assumeFilename": "/home/aleksey/Documents/CppProjects/RPGame/.vscode/.clang-format",  
>   "[cpp]": {  
>       "editor.defaultFormatter": "xaver.clang-format"  
>   },  
  
  
##Пример файла форматирования:##  
########## Note: let's keep the options sorted in alphabetical order.

AccessModifierOffset: -4  
AlignAfterOpenBracket: true  
AlignConsecutiveAssignments: true  
AlignEscapedNewlinesLeft: true  
AlignOperands: true  
AlignTrailingComments: true  
AllowAllArgumentsOnNextLine: true  
AllowAllConstructorInitializersOnNextLine: true  
AllowAllParametersOfDeclarationOnNextLine: true  
AllowShortBlocksOnASingleLine: false  
AllowShortCaseLabelsOnASingleLine: false  
AllowShortFunctionsOnASingleLine: None  
AllowShortIfStatementsOnASingleLine: false  
AllowShortLoopsOnASingleLine: false  
AlwaysBreakAfterDefinitionReturnType: None  
AlwaysBreakBeforeMultilineStrings: false  
AlwaysBreakTemplateDeclarations: true  
  
########## Note: if BinPackArguments is "true", lambdas may be formatted in a bit ugly way, e.g.  
########## foo(bla, [](OOO)  
########## {  
########## },  
########## [](AAA)  
########## {  
########## });  
########## With "false" the "[](OOO)" part occupies a separate line.  
##########  
########## There is a bug report about incorrect formatting of lambdas similar to  
########## the example above: http://llvm.org/bugs/show_bug.cgi?id=20450  
BinPackArguments: false  
  
BinPackParameters: false  
  
BreakBeforeBinaryOperators: None  
BreakBeforeBraces: Allman  
BreakBeforeTernaryOperators: true  
BreakConstructorInitializersBeforeComma: true  
ColumnLimit: 120  
  
########### A regular expression that describes comments with special meaning, which should not be split into lines or otherwise changed.  ##########  
#CommentPragmas:  
  
ConstructorInitializerAllOnOneLineOrOnePerLine: false  
ConstructorInitializerIndentWidth: 4  
ContinuationIndentWidth: 4  
Cpp11BracedListStyle: true  
DerivePointerAlignment: false  
ExperimentalAutoDetectBinPacking: false  
ForEachMacros: [BOOST_FOREACH]  
IndentCaseLabels: true  
IndentWidth: 4  
IndentWrappedFunctionNames: false  
########## There is a bug report about empty lines at the beginning of  
########## namespaces: http://llvm.org/bugs/show_bug.cgi?id=20449  
KeepEmptyLinesAtTheStartOfBlocks: true  
Language: Cpp  
##########A regular expression matching macros that start/stop a block.  ##########
##########MacroBlockBegin  
##########MacroBlockEnd  
MaxEmptyLinesToKeep: 1  
NamespaceIndentation: All  
ObjCBlockIndentWidth: 4  
ObjCSpaceAfterProperty: true  
ObjCSpaceBeforeProtocolList: true  
  
########## Note: these options specify the penalties for certain formatting decisions; for each chunk of code clang-format is
########## supposed to try all available combinations and select the one with minimum total penalty.
########## Note: it's rather hard to predict the effects that each individual option will have on the code readability in general,
########## hence we only use some of the options with big values to effectively disable the corresponding behaviour.
##########PenaltyBreakBeforeFirstCallParameter:
##########PenaltyBreakComment:
##########PenaltyBreakFirstLessLess:
##########PenaltyBreakString:
##########PenaltyExcessCharacter:
PenaltyReturnTypeOnItsOwnLine: 10000

PointerAlignment: Right  
SortIncludes: false  
SpaceAfterCStyleCast: false  
SpaceBeforeAssignmentOperators: true  
SpaceBeforeParens: ControlStatements  
SpaceInEmptyParentheses: false  
SpacesBeforeTrailingComments: 1  
SpacesInAngles: false  
SpacesInCStyleCastParentheses: false  
SpacesInContainerLiterals: false  
SpacesInParentheses: false  
SpacesInSquareBrackets: false  
Standard: c++17  
TabWidth: 4  
UseTab: Never  
 ##########----------------------------------
  
