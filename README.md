# vortex-docs
TAKEN FROM THE VORTEX TEX FILE

# The Vortex programming language

## Daniel "q66" Kolesa

## October 18, 2018

Abstract
Vortex is a scripting language created to explore the possibilities of using Lua as an inter-
mediate language. While compiling to performant Lua (comparable to handwritten code),
Vortex tries to simplify programming by introducing new features (such as lambda expres-
sions, macros, lists and objects) that Lua lacked before (while not introducing anything
that is not present in regular Lua). Vortex gains inspiration from well known programming
languages mainly in the functional paradigm, such as OCaml, F#, Scheme and Rust besides
Lua itself. The language itself tries to offer the programmer multiple paradigms – func-
tional, procedural, object oriented (prototype based – delegative with multiple inheritance)
and metaprogramming. Vortex does not try to create a “preprocessor” for Lua (akin to
attempts like CoffeeScript/MoonScript).


## Contents


- 1Introduction
   - 1.1 Disclaimer
   - 1.2 Conventions
- 2Concepts
   - 2.1 Variables and values
   - 2.2 Metaprogramming
   - 2.3 Modules
   - 2.4 Tables and metatables
   - 2.5 Coroutines
- 3Lexical
   - 3.1 Encoding
   - 3.2 Whitespace
   - 3.3 Comments
   - 3.4 Identifiers
   - 3.5 Keywords
   - 3.6 Other tokens
   - 3.7 Number literals
   - 3.8 String literals
- 4Expressions
   - 4.1 Main scope
   - 4.2 Blocks
   - 4.3 Return and result expressions
   - 4.4 Binary and unary expressions
   - 4.5 Tables and lists
   - 4.6 Primary expressions
   - 4.7 Suffixed expressions
   - 4.8 Let expression
   - 4.9 Set expression
   - 4.10 With expression
   - 4.11 Functions
   - 4.12 If expression
   - 4.13 Loops
   - 4.14 Pattern matching
   - 4.15 Objects
   - 4.16 Coroutines
   - 4.17 Sequences
   - 4.18 Enumerations
   - 4.19 Other expressions
- 5Macros
- 6Theruntimeandthestandardlibrary
- 7TheREPL
- 8Appendix:Styleguide
   - 8.1 Indentation
   - 8.2 Whitespace
   - 8.3 Blocks
   - 8.4 Naming style
   - 8.5 Examples
- 9Appendix:ThecompletesyntaxofVortex
- 10 Appendix: Influences


## 1Introduction

This document attempts to create a reference manual for the Vortex programming language.
It tries to cover:

- Various aspects of the language, including lexical analysis and all of the language’s
    structures. The standard library is not covered (and at the point of writing, not yet
    designed).
- Rationale for many of the language features.
- Examples of usage.
It tries to be just documentation, not a tutorial. For a tutorial, check out other sources
(such as the Vortex wiki).

### 1.1 Disclaimer

Vortex is still in heavy development. This document will change as the language changes

- do not use it in production. While the basic idea and style is given, language features
may appear, change and vanish. The implementation may not reflect the actual state of
the language. Some features described here are not yet implemented at the time of writing.
Unimplemented features will be clearly marked as such in their section headers.

### 1.2 Conventions

The formal grammar of Vortex specified here is written using an extended dialect of BNF.
The BNF strings may contain/regexes/.Examplecodeiswrittenusingmonospaceblocks.
Character ranges may be used for obvious things, such asa-Zor0-9.

## 2Concepts

Vortex shares most of the basic concepts with Lua.

### 2.1 Variables and values

Vortex is an untyped language, or unityped to be precise – there is a single data type in
the language, a value, similar to Lua, Python, JavaScript and so on. The values are tagged.
The variable holds the data type and its value holds the tag.
Vortex type tags are shared with Lua, being represented bynil,boolean,number,string,
function,userdata,threadandtable.Theniltag can have a single value,nil.The
booleantag can have thetrueandfalsevalues. The only two values that evaluate as false
in conditions arefalseandnil.Thetagnumberrepresents a floating point number (by
default double precision), depending on the underlying C. The tagstringis an immutable
sequence of bytes. Vortex is like Lua 8-bit clean. Strings can contain any byte, including
embedded zeros.
The taguserdatahas the same meaning as in Lua. The managed kind represents a managed
(garbage collected) block of memory that can have a metatable. The light kind represents
arawpointer.Thetagthreadis represented by coroutines which are identical with Lua’s.
Functions are represented using the tagfunction.They’retransparentclosures.
And finallytableis the ultimate do-it-all data structure of Vortex. It represents an associa-
tive array (unordered hash map). It’s heterogenous, being able to hold any arbitrary value
except nil in both key and value positions.
Vortex also has lists. These are similar to lists in Lisp or other functional languages. They
don’t hold their own tag – they’re tables with specific behavior.
Functions, threads, full userdata and tables are always accessed by reference, everything else
by value (copied). All values are first class values. You can store them, pass them around


and return from functions.
There are two types of variables in Vortex, locals and table fields. Global variables are
merely fields of the global table called_G.Localandglobalvariableshavetobedeclared
before they can be assigned or used, which is different from Lua, where assignment to unde-
clared variable creates a global and reading an undeclared variable results in thenilvalue.
You declare and define variables using theletexpression.

### 2.2 Metaprogramming

Vortex features two types of metaprogramming – static and dynamic. Dynamic metapro-
gramming is identical to Lua’s and is represented by reflection over tables and by metatables.
Vortex however also introduces static metaprogramming (performed at compile time). That
is represented by macros. Macros work at AST level. They’re hygienic – identifiers inside
them cannot escape unless explicitly desired.
As Vortex’s module system is dynamic, evaluated already past the macro expansion step,
there is a system of language “extensions” that allow macros across modules.

### 2.3 Modules

As said above, Vortex features a dynamic module system. It’s taken from Lua itself and it’s
compatible. It’s further documented in the standard library part.

### 2.4 Tables and metatables

As said before, the table is the ultimate do-it-all and the only data structure in Vortex and
Lua. Vortex also has lists, but they’re simply tables with syntax. The real power of tables
comes from metatables.
Every table can have a metatable, or any value actually. In case of tables and userdata,
each piece of userdata and each table can have (but doesn’t have to have) its own, unique
metatable. With other values, the metatable is specific to the type tag (for example all
strings share a single metatable). Of course, you can share metatables between tables, but
it’s all explicit.
Metatables are ordinary tables. A metatable contains metamethods. Metamethods define
fallbacks for operations on the value. What does this mean? It means you can change the
semantics on tables and other values and extend the language this way. For example, a
metamethod__adddefines what happens when you add the value together with some other
value.
Metamethods have an intuitive naming scheme based on the name of the event they handle.
They’re prefixed with two underscores. You can get a metatable of a value using theget_mt
function. You can set a metatable usingset_mt.
The metamethod handling is identical to that of Lua. Please refer to its reference manual
for more information.
__add :the+operator.
This metamethods is called when you try to add something to the value or if something
that has no__addtries to add he value to itself. It takes two arguments representing
the left side of the operation and the right side. It returns the addition result.
__sub :the-operator.
__mul:the*operator.
__div:the/operator.
__mod:the%operator.
__pow :the^operator.
__unm: unary minus, takes just one argument.
__concat :the~operator.


```
__len :the#unary operator.
You can get the value’s length using the raw_lenfunction without invoking this
metamethod.
__eq:the==operator.
You can compare two values using theraw_eqfunction without invoking this metamethod.
__lt:the<operator.
The>operator behavior is defined by reversing the operand order.
__le :the<=operator.
The>=operator behavior is defined by reversing the operand order. When the<=
metamethod is absent,<is used, assuminga<=bis equivalent tonot (b < a).
__index:indexing.
This metamethod is called onval[key].It’sonlycalledwhenkeyis not present in
the table (it’s a fallback operation). The first argument is the value you’re indexing
on and the second argument is the key. It’s supposed to return the value you want to
get.
If you want to retrieve a member without ever invoking this metamethod (e.g. for use
inside this metamethod so that it doesn’t call recursively), useraw_get.
__newindex:indexassignment.
This metamethod is called onval[key] = value.It’sonlycalledwhenkeyis not
present in the table. The firstargument is the value itself, the second argument is the
key, the third argument is the value. Returns nothing, or ignores any return values.
If you want to set a member without invoking this metamethod, useraw_set.
__call :avaluecall.
Called onmyval(arglist). The first argument is the value that’s being called, followed
by the list of arguments to the call.
__gc:calledongarbagecollectionofthevalue.
Useful for tables and userdata. As Vortex uses a garbage collector, tables are not freed
immediately – instead they’re marked for collection, and then during collection cycle
freed. The object finalizers are called in the reverse order that they were marked.
```
### 2.5 Coroutines

```
Lua offers coroutines and so does Vortex. Coroutines are used to do collaborative multi-
threading. A coroutine is a thread – it has its own stack, but it’s not an OS thread. Unlike
Lua, Vortex offers specialized coroutine syntax.
A coroutine is somewhat similar to a function. When you create a coroutine from a func-
tion, you get a thread object. The thread object is by default in a suspended state. When
you resume the thread object, the function runs. If it’s a regular function you could call
normally, the thread object dies and can’t be resumed again.
There is a language construct in Vortex calledyield.Yieldingacoroutineresultsinthe
thread object being suspended at the point of yield. You can then again resume the corou-
tine and it’ll run until the next yield.
You can pass one or more expressions toyield.Thegrammarisdefinedinthelanguage
section. Theresumefunction will return values of the expressions passed toyield.Italso
returns the return value of the function when the thread object dies. Yieldthus has the
same semantics asreturnwhen it comes to resuming, except that it only suspends the
coroutine instead of making it die.
You can also pass expressions toresume.They’llbereturnedinsidethecoroutinebythe
yield. An example:
```
1 // special syntax for coroutines providedby Vortex
2 // coroutinescan also becreated from functions using
3 // special callswith the same effect
4 let my_coroutine =coro a, bdo
5print("AB",a,b)


6 let (c, d) =yield (5, 10)
7print("CD",c,d)
8 return (15, 20)
9 end
10 print ( resume ( my_coroutine , 2 , 3))
11 print ( resume ( my_coroutine , 6 , 7))
12 print ( resume ( my_coroutine , 8 , 9))

```
This will print something like
```
```
AB 2 3
true 5 10
CD 6 7
true 15 20
false cannot resume dead coroutine
```
```
What exactly happened here? First, we resumed the coroutine, passing values 2 and 3 to
it. Here it was like a function call. The passed values becameaandb.Thecoroutinethen
printed them and yielded – suspended itself, passing back values 5 and 10 and expecting
canddfor the next resume. Theresumefunction then printed true (resumereturns the
state the coroutine was in – whether it was dead or alive as the first result) plus the yielded
values.
Then we resumed once again. The values passed toresumebecamecanddas expected.
The coroutine printed the values and returned more values. The coroutine has died by now.
Theresumecall once again returnedtrue(because it was alive at the point of execution)
and the return values. A third resume returnedfalse,becausethecoroutinewasalready
dead, plus an error message.
That’s coroutines in a nutshell. There are two variants of coroutines in Vortex, regular
coroutines and generators. There’s a subtle difference in the amount of power they give you
and in the simplicity of writing. Please refer to the later sections.
```
## 3Lexical

```
Vortex utilizes a fully free form syntax. Whitespace of any form is ignored, used only to
delimit tokens where it would otherwise be ambiguous.
```
### 3.1 Encoding

```
halphai::=‘a-zA-Z’
```
```
hdigiti::= ‘0-9’
```
```
hhexdigiti::=‘0-9a-fA-F’
```
```
The input is a sequence of bytes. Vortex does not handle Unicode in any way, but it’s UTF-
8clean. AllofthegrammarisconfinedtotheASCII.Becauseit’sUTF-8clean,Unicode
strings and such are allowed and passed to the output without any processing. If an UTF-
BOM is found, it’s skipped automatically. If a shebang line is found in the beginning, it’s
skipped as well.
```
### 3.2 Whitespace


```
hwhitespacei::=‘’
|‘n’
|‘r’
|‘t’
|‘f’
|‘v’
```
The Vortex lexer does not care about whitespace for anything else than token separation.
Whitespace is not needed to separate tokens; the source is simply read character by character
and when a token ending is found, it just goes on to the next one. The BNF above shows
all possible forms of whitespace in Vortex. As you can see, newlines are treated as regular
whitespace as well (with the exception of incrementing the line number). Newline and
carriage return can be used in pairs no matter what order (handles the common cases ’\n’
and ’\r\n’ plus ’\n\r’ for rare platforms).

### 3.3 Comments

```
hcommenti::=‘//’/.*$/
|‘/*’{/./|hcommenti}‘*/’
```
Vortex uses C(++) style comments. Short comments comment out everything until the end
of the line. Long comment can span multiple lines and are enclosed between delimiters.
Unlike C(++), Vortex allows for nesting of comments (thus it requires you to keep them
balanced). Comments do not get past lexical analysis.

### 3.4 Identifiers

```
hidenti::=(‘_’|halphai){‘_’|halphai|hdigiti}
```
```
hidentlisti::=hidenti{‘,’hidenti}
```
Vortex identifiers (names) can consist of alphanumeric characters (ASCII only) and un-
derscores. They can’t start with a digit, but a digit can be present anywhere else in the
identifier. Identifiers starting with an underscore followed by an uppercase character are
reserved. This is by convention and not enforced by the compiler.

### 3.5 Keywords

```
hopkeywordi::=‘band’
|‘bor’
|‘bxor’
|‘asr’
|‘bsr’
|‘bsl’
|‘and’
|‘or’
```
```
hopkeywordassi::=‘band=’
|‘bor=’
|‘bxor=’
|‘asr=’
```

```
|‘bsr=’
|‘bsl=’
```
```
hkeywordi::=‘as’
|‘break’
|‘case’
|‘cfn’
|‘clone’
|‘coro’
|‘cycle’
|‘do’
|‘else’
|‘end’
|‘enum’
|‘false’
|‘fn’
|‘for’
|‘glob’
|‘goto’
|‘if’
|‘in’
|‘let’
|‘loop’
|‘macro’
|‘match’
|‘module’
|‘new’
|‘nil’
|‘quote’
|‘rec’
|‘redo’
|‘result’
|‘return’
|‘seq’
|‘set’
|‘true’
|‘unquote’
|‘when’
|‘while’
|‘with’
|‘yield’
|‘__FILE__’
|‘__LINE__’
| hopkeywordi
| hopkeywordassi
| hunopkeywordi
```
```
hunopkeywordi::=‘not’
|‘bnot’
```
These are “reserved words” in Vortex. They cannot be used as identifiers. Some of these
specified inopkeywordhave an assignment form. Vortex is case sensitive, the keywords only
are keywords in lowercase. For example,matchis a keyword butMATCHorMatchcan be used
as valid identifiers.


### 3.6 Other tokens

```
hbinopi::=‘=’
|‘==’
|‘>’
|‘>=’
|‘<’
|‘<=’
|‘!=’
|‘%’
|‘’
|‘+’
|‘++’
|‘*’
|‘**’
|‘-’
|‘/’
|‘::’
| hopkeywordi
```
```
hassopi::=‘=’
|‘+=’
|‘++=’
|‘*=’
|‘**=’
|‘-=’
|‘/=’
|‘::=’
|‘%=’
| hopkeywordassi
```
```
hunopi::=‘-’
|‘#’
| hunopkeywordi
```
```
hothertoki::= ‘(’
|‘)’
|‘->’
|‘.’
|‘..’
|‘...’
|‘;’
|‘;;’
|‘$’
|‘$(’
```
Here you can see all the other tokens used by Vortex with separate categories for binary,
unary and assignment operators.

### 3.7 Number literals

```
hhexnumi::= (‘0x’|‘0X’) {hhexdigiti}[‘.’]{hhexdigiti}[(‘p’|‘P’) [ ‘+’|‘-’]
hdigiti]
```

```
hdecnumi::={hdigiti}[‘.’]{hdigiti}[(‘e’|‘E’) [ ‘+’|‘-’]hdigiti]
```
```
hnumliterali::= hhexnumi
| hdecnumi
```
Vortex does not make a difference between integers and floats. Numbers follow the Lua
format. Hexadecimal numbers start with0xor0X(the former is better). In numbers like
0.6you can omit the zero and write just.6.Hexadecimalconstantsbehavesimilarly.
Optional decimal exponent is marked witheorE(binary exponent in hex constants ispor
P).

### 3.8 String literals

```
hstrliterali::= hstrprefixi(hstrlongi|hstrshorti)
```
```
hstrprefixi::= /[eErR]*/
```
```
hstrshorti::=‘"’hstrshortelemi‘"’
|‘'’hstrshortelemi‘'’
```
```
hstrlongi::=‘"""’hstrlongelemi‘"""’
|‘'''’hstrlongelemi‘'''’
```
```
hstrshortelemi::=?shortstringcontents?
| hstresci
```
```
hstrlongelemi::=?longstringcontents?
| hstresci
```
```
hstresci::=‘a’
|‘b’
|‘f’
|‘n’
|‘r’
|‘t’
|‘v’
|‘z’
|‘"’
|‘'’
|‘’
```
String literals in Vortex are similar to Python’s. They are represented by the<string>
token in the final stream. There are two types of string literals, short and long literals.
Short literals are delimited with either single or double quotes and typically hold a single
line. They interpret escape sequences. A backslash at the end of the line can be used to
make them span multiple lines. You need to escape nested single or double quotes depending
on the used delimiter (e.g. single quote delimited strings need to escape nested single quotes
but not double quotes).
Long literals behave similarly. They’re delimited by three repeated either single or double
quotes. Escape sequences work the same, and long literals can span multiple lines without
backslashes. You don’t need to escape quotes except when three subsequent quotes are used
(because otherwise they’d terminate the string).
You can prefix both types of literals with eithereorr(and uppercase versions – no difference
there). The former enables interpolation on that string – it will interpret$varand$(expr)
in the string, wherevaris any Vortex variable you can access at that point andexpris any
Vortex expression. For example


1 for k, vin pairs ([ 5, 10, 15 ])>print(e"$k>$(v+2)")

```
will print
```
```
1 >
2 >
3 >
```
```
Interpolated strings allow you to escape the dollar sign to prevent interpolation.
The latter prefix turns the strings into raw strings. That means no escape sequences are
interpreted and instead they apparear in the result verbatim. If you escape quotes, the
backslashes will be visible in the string. The same applies about backslashes used to escape
newlines in short literals.
There is one non-standard escape sequence,\z. It skips the following span of whitespace
characters (including newlines) in both short and long literals. That is useful to break a
short string literal into multiple lines and indent the lines without actually including the
newlines and indentation in the string.
You can also insert an arbitrary byte in the string. That can be done either with an escape
sequence\xXX,whereXXis a sequence of two hexadecimal digits (e.g. \x4Ffor uppercase
O) or with an escape sequence\dddwheredddis a sequence of up to three decimal digits
(\79would be the uppercase O).
```
## 4Expressions

```
hexpi::=hstatexpi
```
```
hexplisti::=[hexpi{‘,’hexpi}]
```
```
Vortex is a language that consists purely of expressions. No statements are present in the
language in that sense an expression of any type can appear in any context. One exception
is block scopes. Only expressions that can cause some sort of side effect can appear there
(e.g. variable assignment, function call and so on, in the BNF they’re calledstatexp). The
rationale for this is that doing it otherwise doesn’t really make sense – allowing inclusion of
arbitrary expressions in blocks would result in code doing nothing, and that’s better filtered
out at compile time.
```
### 4.1 Main scope

```
hchunki::={hstatexpi[‘;’]}[(hretexpi|hresexpi)[‘;’]]
```
```
hmainchunki::= {(hstatexpi|hmacroi)[‘;’]}[(hretexpi|hresexpi)[‘;’]]
```
```
The parsing of Vortex begins in the main scope. The main scope is a chunk. A chunk is a
sequence of side effect based expressions optionally separated with semicolons. It may be
ended with either areturnor aresultexpression. Having another expression after one of
these results in syntax error.
The main chunk can unlike any other chunk define macros, which are described in their own
section later.
```
### 4.2 Blocks


```
hblockendi::= ‘end’|‘;;’
```
```
hblocki::=‘do’hchunkihblockendi
```
```
hexpscopei::= ‘->’hexpi|hblocki
```
```
hstatscopei::=‘->’hstatexpi|hblocki
```
```
hstatexpi::= hblocki
```
```
Blocks represent chains of expressions. A block consists of adokeyword, a chunk (see above)
and either anendkeyword or two semicolons. The semicolon ending is useful in inline blocks
or in Lisp style formatted code. Blocks themselves are expressions. Theresultexpression
can be used to specify their value.
```
### 4.3 Return and result expressions

```
hexporlisti::=(hexpi|‘(’hexplisti‘)’)
```
```
hretexpi::= ‘return’hexporlisti
```
```
hresexpi::=‘result’hexporlisti
```
```
hstatexpi::= hretexpihresexpi
```
```
These two expressions are used to manipulate values. Thereturnexpression jumps out of
a function, making it return value(s) (in the main scope it makes the module return a value
later used withrequire).
Theresultexpression is very similar at first. It however works on scope level – it basically
sets a value the current block will evaluate to. With regular functions, this is pretty much
the same (as a block is typically a function return value) but you can see the difference when
working with expression blocks. For example:
```
1 // this function returns 7,x is 5.
2 fn foo()do
3 letx=do
4 result 5
5 end
6 return x+
7 end
8
9 // this function returns 5,never reaching the final return.
10 fnbar() do
11 letx=do
12 return 5
13 end
14 return x+
15 end

### 4.4 Binary and unary expressions

```
hassexpi::=(hidenti|hindexpi)hassopihexpi
```
```
hbinexpi::=hexpihbinopihexpi
```

```
hunexpi::=hunopihexpi
```
```
hexpi::=hbinexpi
| hunexpi
```
```
hstatexpi::= hassexpi
```
Binary expressions are expressions that consist of two operands and an operator. The
operator is in infix form. The BNF here does not describe operator precedences. Assignment
operators are treated differently as it’s required to ensure that the left operand is an lvalue
(you can’t assign an arbitrary expression). Assignment expressions can also be used in
statement form, unlike any other binary expression.
Unary expressions are expressions that consist of an operand and an operator in prefix form.
Here you can see operator precedences for binary and unary operators in Vortex.

```
Operator Precedence Associativity
=, +=, -=, *=, /=, %=, ̃=, ++=, ::=, **=, 1 right
band=, bor=, bxor=, asr=, bsr=, bsl=
or 2 left
and 3 left
==, != 4 left
<, <=, >, >= 5 left
̃ 6 right
bor 7 left
bxor 8 left
band 9 left
asr, bsr, bsl 10 left
+, - 11 left
*, /, % 12 left
++ 13 left
:: 14 right
-, not, #, bnot 15 unary
** 16 right
```
Most of the operators should be clear when it comes to meaning. Assignment operators in
formlhs op= rhsare equivalent tolhs = lhs op rhs.~means concatenation,++is a join
operator (for tables),::is a cons operator (as in Lisp),#retrieves the length of the given
expression,**raises lhs to a power of rhs. band,bor,bxor,asr,bsr,bslmean bitwise
AND, bitwise OR, bitwise XOR, arithmetic right shift, bitwise right shift and bitwise left
shift respectively. The rest of the operators is functionally equivalent to those in C.

### 4.5 Tables and lists

```
htableitemi::=[([‘$’]hidenti|‘$(’hexpi‘)’) ‘:’]hexpi
```
```
htableexpi::=‘{’[htableitemi{‘,’htableitemi}]‘}’
```
```
hlistexpi::=‘[’hexpi{‘,’hexpi}]‘]’
```
Vortex provides two types of built-in data structures. They’re tables and lists.
Tables work in the same way as in Lua. They are something between an array, an associative
array (unordered hash map) and an object. Array and hash elements can be both present
in a single table. Tables are already described in their own section above.
Syntactically, table literals are enclosed in curly braces. If you want a value that has a key,
you use the colon syntax. For example:


1 let array = { 5, 10, 15 }//an array of 3 elements
2 assert(array[1] == 5and array [3] == 15)
3
4 // containskeys "foo","abcd","bar"
5 let assarray = { foo :"bar",$("ab" ~ "cd"):"baz",$bar:"xyz" }
6
7 //combined
8 let comb = { 5 , 10 , hello : "world",15}

```
As you can see, keys can be arbitrary expressions. Keys can be any value exceptnil.
Arrays count from 1. They’re basically associative arrays with keys that are numbers,
however, Lua optimizes this by storing array elements in their own section. You can assign
to a table as you need. Array length is retrieved using the#operator. For example, this
way you can append:
```
1 let array = { 5, 10, 15 }
2array[#array+1]=

```
Tables can have metatables as mentioned in the section above.
Lists are another data structure of Vortex. They use square brackets. Vortex lists are singly
linked lists in concept similar to Lisp lists. You can construct a list multiple ways:
```
1 // using the list syntax
2 let lst = [ 5, 10, 15, 20 ]
3 // using the cons binaryoperator
4 let tsl = 5 :: 10 :: 15 :: 20 :: nil

```
Both ways are equivalent. A list consists of the “head” element and the “tail” element. Here,
the first head is 5, the tail is another list where the head is 10, the tail is another list, the
head again is 25, followed by another list where the head is 20 and the tail isnil.
Sometimes lists are more efficient than just tables. It depends on the use. Lists are imple-
mented using tables in the runtime.
```
### 4.6 Primary expressions

```
hprimaryexpi::=‘(’hexpi‘)’
| htableexpi
| hlistexpi
|‘$(’hexpi‘)’
|‘$’hidenti
| hidenti[‘!’‘(’hexplisti‘)’]
| hnumliterali
| hstrliterali
|‘nil’
|‘true’
|‘false’
```
```
hexpi::=hprimaryexpi
```
```
Primary expressions are simple expressions that can have a suffix. A suffix is for example
aparameterlist(acall),bracketswithanexpression(indexing)oradot(simpleindexing).
Primary expressions typically don’t have a side effect, thus they can’t be used in statement
form unless postfixed as a call. You can wrap any arbitrary expression in parens to get a
primary expression. All types of simple literals as well as table and list literals and macro
expansions are primary expressions.
```

### 4.7 Suffixed expressions

```
hfcallsuffixi::=‘(’hexplisti‘)’
| htableexpi
| hstrliterali
```
```
hmcallsuffixi::= ‘:’hidentihfcallsuffixi
```
```
htcallsuffixi::=hfcallsuffixi
| hmcallsuffixi
```
```
hindexsuffixi::= ‘.’hidenti
|‘[’hexpi‘]’
```
```
hexpsuffixi::= [hexpsuffixi](hindexsuffixi|htcallsuffixi)
```
```
hsuffixedexpi::=hprimaryexpihexpsuffixi
```
```
hexpi::=hsuffixedexpi
```
```
hstatexpi::= hsuffixedexpihtcallsuffixi
```
Suffixed expressions are primary expressions with a suffix. A suffix represents either a call
or indexing. Suffixes can be chained. In statement form, only calls are allowed (not indexing
alone). Indexing can be represented in two forms. The simpler form consists of an expres-
sion, a dot and a name. The name must be a valid indentifier. The more comprehensive
form consists of an expression and an index enclosed in brackets (the index can be an ar-
bitrary expression). Writingfoo.baris equivalent tofoo["bar"]assuming"bar"contains
no non-identifier characters.
Calls can be either method callsobj:mname(args)or regular callsfuncname(args).The
former is just syntactic sugar forobj.mname(obj, args).Ifthesoleargumentisata-
ble or string literal, you can omit the parens. That means writingfuncname "foo"and
funcname { 5, 10, 15 }is equivalent tofuncname("foo")andfuncname({ 5, 10, 15 })
respectively.

### 4.8 Let expression

```
hlettypei::= ‘rec’
|‘glob’
```
```
hletexpi::= ‘let’[hlettypei](hpatterni|‘(’hpatternlisti‘)’) ‘=’hexporlisti
```
```
hstatexpi::= hletexpi
```
Theletexpression provides means to declare and define variables. It consists of the keyword,
apatternorapatternlistenclosedinparens,anassignmentoperatorandeitherasingle
expression or an expression list enclosed in parens.
You can also optionally provide modifier after the keyword. The modifier can currently
be eitherrecorglob. The former is best used with functions – it makes it possible for a
variable to access itself and that way you can define recursive functions. Normally functions
can’t access themselves. The latter is used to define a global variable – by default, all Vortex
variables are local.
You can’t normally declare a variable without definition. However, you can assign nil to it,
which is the same in meaning.
For patterns, look up the section about pattern matching. Note that only patterns that never
fail to match can be used withlet.Forconvenience,thetablepatternalwaysmatchesinlet


```
but doesn’t have to in regular pattern matching (that is because in let you sometimes want
to extract just a few elements of an array, but in pattern matching you want it precise).
An example of let expression:
```
1 let x=5;// local variable x
2 let globy=10;// global variabley
3 fn foo()> [5,10,15]
4 let [a,b]=foo()//a is 5,b is 10 patternusage
5 fnrec bar()>bar() // recursive

### 4.9 Set expression

```
hsetexpi::=‘set’(hsuffixedexpi|‘(’hexplisti‘)’)hassopihexporlisti
```
```
hstatexpi::= hsetexpi
```
```
Thesetexpression assigns values to variables. To assign a single variable, you can use the
regular assignment binary expression. Thesetexpression is used to set multiple variables
at once. Like an assignment expression, the expressions on the left have to be lvalues. This
expression looks quite similar to theletexpression.
The main benefit of setting multiple variables at once is for e.g. swapping values. Consider
this:
```
1 let a=
2 let b=
3 //now let’s swapa andb
4 let tmp = a
5a=b
6b=tmp

```
This doesn’t look too good. Isn’t there a better way? Of course there is.
```
1 let a=
2 let b=
3 //andnow let’s swap
4 set (a, b) = (b, a)

```
Much better, right? Thesetexpression evaluates to its left side after assignment.
```
### 4.10 With expression

```
hwithexpi::= ‘with’(hpatterni|‘(’hpatternlisti‘)’) ‘=’hexporlistihexpscopei
```
```
hwithstati::=‘with’(hpatterni|‘(’hpatternlisti‘)’) ‘=’hexporlistihstatscopei
```
```
hstatexpi::= hwithstati
```
```
hexpi::=hwithexpi
```
```
This expression is sort of similar tolet.Itrepresentsascope-boundvariable.Unlikelet,
it doesn’t evaluate to its left side, but instead to its expression. Consider this:
```
1 let x=with y=a()do
2print("helloworld!")
3 result 5
4 end

```
Here, the variableyis bound to thewithexpression scope, being invisible from anywhere
else. The variablexwill be 5 .Notethatthescopewillalwaysevaluate,evenwhenyisnil.
```

### 4.11 Functions

```
hdefarglisti::=hidenti‘=’hexpi{‘,’hidenti‘=’hexpi}
```
```
harglisti::=[hidentlisti][hdefarglisti][[hidenti]‘...’]
```
```
hfnscopei::= ‘->’(hexpi|hmatchbodyi)|hblocki
```
```
hfnliterali::=‘fn’(harglisti|‘(’harglisti‘)’)hfnscopei
```
```
hfndefi::=‘fn’([hlettypei]hidenti|hidenti[‘.’hidenti]) ‘(’harglisti‘)’hfnscopei
```
```
hstatexpi::= hfndefi
```
```
hexpi::=hfnliterali
```
```
Vortex features first class functions. That means a function can be treated as a first class
citizen – you can pass it as an argument or return it from another function. Like in Lua,
functions are passed by reference – you never access the function value directly, you only
access its reference.
You can create a function two ways. The first way is an anonymous function. Because
anonymous functions don’t have names, you need to assign it to a variable. It looks like
this:
```
1 let add =fna, b>a+b
2print(add(5,10))

```
You can optionally put parens around the argument list. The second way is a named
function. Given the previous example, you can rewrite your function as:
```
1 fnadd(a , b)>a+b

```
Named functions can be defined as table members. For example:
```
1 let tbl = {}
2 fn tbl .foo()>"hello world"

```
They can also have modifiers, the same ones as theletexpression (unless defined as table
members – then it doesn’t make sense). Named function definitions follow the same rules
as theletexpression. By default, named functions are local.
```
1 let foo =fn>foo() //won’t worknonrecursive
2 let rec bar =fn>bar() //worksbar previously declared
3 fn foo()>foo() //won’t work
4 fnrec bar()>bar() //works

```
Named functions can be used as statements. Anonymous functions can’t.
Now, if you look at the examples, you can see they’re pretty much lambda expressions.
They take inputs and they return the value specified after the arrow. You can combine that
with blocks. Bothreturnandresultcan be used to specify the function return value. The
former simply jumps out of the function and makes it return the value, the latter specifies
the value of the block which the function then returns. The result is pretty much the same.
```
1 fn foo()>do
2...
3 end

```
The arrow feels kinda superfluous. Vortex allows you to omit it:
```
1 fn foo()do
2...
3 end


```
Function arguments in Vortex can have default values. After you specify the first default
value, every argument after that one must specify a default value. Example:
```
1 //you can reference the previous arguments too
2 fn foo(a, b = 5, c = b + 2, d = c + 3)do
3...
4 end

```
You can end the argument list with an ellipsis argument. That means the function will be
variadic and you can access the remaining arguments passed to the function again using an
ellipsis. Example:
```
1 fn printf(fmt, ...) do
2print(fmt:format(...))
3 end

```
Note that to pass all the variadic arguments to a function, the ellipsis must be the last
argument of the function call. Otherwise just the first value of the tuple will be passed! You
can make it pass a single value anywhere by capturing it inlet,forexample
```
1print("Only the first value", let_= (...))

```
Lua allows you to do this by wrapping the ellipsis in parens. I consider that quite dangerous
and bug-prone, thus Vortex doesn’t allow this. While wrapping the ellipsis in parens in
legal, it’ll always evaluate to multiple values, no matter what.
You can name the vararg tuple and pass it around as a table like this:
```
1 fn foo(a, b...) do
2print(a,b[1],b[2])
3 end

```
That is functionally equivalent to:
```
1 fn foo(a, ...)do
2 letb={ ... }
3print(a,b[1],b[2])
4 end

```
Vortex functions have a shorthand pattern matching variant. Writing
```
1 fn arglist>
2|patternlist>exp
3|...

```
is equivalent to
```
1 fn arglist>match arglist>
2|patternlist>exp
3|...

### 4.12 If expression

```
hifblocki::=‘do’hchunki
```
```
helseopti::= ‘else’[‘->’]exp
```
```
helsestatopti::= ‘else’[‘->’]statexp
```
```
hifexpi::= ‘if’hexpi(‘->’hexpi[helseopti]|hifblocki(‘end’|helseopti))
```

```
hifstati::=‘if’hexpi(‘->’hstatexpi[helsestatopti]|hifblocki(‘end’|helsestatopti))
```
```
hstatexpi::= hifstati
```
```
hexpi::=hifexpi
```
```
Theifexpression allows you to do structured programming by incorporating conditionals.
Everyifexpression begins with the keyword, followed by a condition. The condition eval-
uates to eithertrueorfalse.Ifitevaluatestotrue,iteitherevaluatestotheexpression
that follows the condition (when in expression form) or simply executes the expression in
statement form (in that case, only statement form expressions are allowed).
There can be an optionalelsepart. That part is evaluated when the condition is not met
and has the same semantics as the former. Note that the arrow is optional withelse,even
with regular expressions.
When usingifwith blocks and without arrows, theelsekeyword implicitly terminates the
block scope. Usingendexplicitly ends theifexpression.
```
### 4.13 Loops

```
hloopcondi::=‘while’hexpi
```
```
hloopexpi::=‘loop’[hloopcondi]hstatscopei[hloopcondi]
```
```
hnumforexpi::= ‘for’hidenti‘=’hexpi‘..’hexpi[‘,’hexpi]hstatscopei
```
```
hgenforexpi::= ‘for’hidentlisti‘in’hexplistihstatscopei
```
```
hstatexpi::= hloopexpi
| hnumforexpi
| hgenforexpi
```
```
hexpi::=‘break’
|‘cycle’
|‘redo’
```
```
Vortex has three types of loops. The simplest one is thelooploop. By default, it loops like
this:
```
```
1 loopdo
2print("I’m infinite!")
3 end
```
```
You can add two kinds of conditions to this kind of loop, a regular condition and a postcon-
dition. Both work similarly, but with the postcondition the loop iterates at least once before
evaluating the condition (it evaluates AFTER iteration) while with the regular condition it
may never start (it evaluates BEFORE iteration). You can use both at once. Example:
```
1 let i=
2 let keep_iteratng =false
3
4 fn check_iterate()>true
5
6 loopwhile i<=10do
7print("I’mno longer infinite...")
8i+=
9keep_iterating=check_iterate()
10 end while keep_iterating


```
Then there is the numericforloop. It iterates using a numeric range. Both the start and
the end of the range are inclusive. For example:
```
```
1 for i=1 .. 10do
2print(i)
3 end
```
```
This prints numbers from one to ten. There is an optional third
step expression. By default it’s 1.
```
```
1 //0, 2,4,6,8, 10
2 for i=0 .. 10,2do
3print(i)
4 end
```
```
The loop never re-evaluates the input expressions – they’re evaluated once when the loop
starts. Thestepis particularly useful for backwards iteration. Writing
```
```
1 for i=start, stop, step>statexp
```
```
is equivalent to
```
1 do
2 let (start , stop, step) =
3(tonum(start),tonum(stop),tonum(stepor 1))
4 ifnot (startand stopand step)>error()
5 loop while(step > 0and start <= stop)
6 or (step <= 0and start >= stop)do
7 let v=start
8statexp
9start+=step
10 end
11 end

```
Finally, there is the genericforloop. It uses iterators and is compatible with Lua iterators.
You have a list of identifiers (you can have as many as you want as long as the iterator
handles them all). Then you have the expression list, which is typically a single expression
(an iterator). Each iteration the iterator is called, returning a new set of values mapping to
the inputs. Writing
```
```
1 for k, vin pairs(tbl)>statexp
```
```
is equialent to
```
```
1 do
2 let (fun, s, var) = pairs(tbl)
3 loopdo
4 let (k, v) = fun(s , var)
5 if k==nil>break
6var=k
7statexp
8 end
9 end
```
```
All types of loops accept both arrow notation and blocks, but even in arrow notation and
expression form the expression past the arrow ALWAYS must be a statement. A loop is an
expression, but it evaluates tonil.
You can control the loop using three expressions, break,cycleandredo.Usingbreak
you can stop the iteration at that point. Usingcycleyou can skip to the next iteration,
incrementing counters or calling iterator as needed. Usingredoyou can achieve a similar
```

thing, but no counter is ever incremented (or iterator called), effectively restarting the
current iteration. Thecyclekeyword is equivalent tocontinuein several other languages
(such as C, C++ or JavaScript).

### 4.14 Pattern matching

```
hobjectpatitemi::=([ ‘$’]hidenti|‘$(’hexpi‘)’) [ ‘:’([‘$’]hidenti|‘$(’hexpi‘)’) ]
```
```
hobjectpatbodyi::=hobjectpatitemi{‘,’hobjectpatitemi}
```
```
htablepatitemi::=[([‘$’]hidenti|‘$(’hexpi‘)’) ‘:’]hpatterni
```
```
hprimarypatterni::=‘(’hpatterni‘)’
| hstrliterali
| hnumliterali
|‘true’
|‘false’
|‘nil’
|‘_’
|[‘$’]hidenti[‘(’[hobjectpatbodyi]‘)’]
|‘$(’hexpi‘)’[‘(’[hobjectpatbodyi]‘)’]
|‘{’htablepatitemi{‘,’htablepatitemi}‘}’
```
```
hsuffixedpatterni::= hprimarypatterni
| hsuffixedpatterni‘when’hexpi
| hsuffixedpatterni‘as’hexpi
```
```
hpatternopi::= ‘and’
|‘or’
|‘::’
```
```
hpatterni::=hsuffixedpatterni[hpatternopihsuffixedpatterni]
```
```
hmatchexpi::=‘match’hexplisti‘->’hmatchbodyi
```
```
hmatchstati::=‘match’hexplisti‘->’hmatchbodystati
```
```
hmatcharmi::=(‘|’|‘case’)hpatternlistihexpscopei
```
```
hmatcharmstati::=(‘|’|‘case’)hpatternlistihstatscopei
```
```
hmatchbodyi::=hmatcharmi{hmatcharmi}
```
```
hmatchbodystati::=hmatcharmstati{hmatcharmstati}
```
```
hexpi::=hmatchexpi
```
```
hstatexpi::= hmatchstati
```
Vortex provides pattern matching similarly to e.g. the MLs, Rust or Haskell. Pattern
matching works like a generalizedswitchstatement – you have a list of input expressions in
thematchexpression and then you have some arms - arms start either with\|orwithcase,
followed by a list of patterns (one pattern for each input, if you don’t provide a pattern for
some input then it always matches the input), then followed by either an expression after
an arrow or adoblock.


The same rules as with e.g.ifapply – whenmatchis used in a statement form, only state-
ment form expressions are allowed.
The arms are evaluated from the top. The first arm where all patterns match is evaluated
and then the evaluation stops. As you can see, it’s similar to aswitch,butthereisno
fallthrough. Also unlike for example OCaml, non-exhaustive patterns are NOT detected.
Pattern matching can be used for decomposition of data structures, besides regular mathch-
ing. For that purpose, lots of patterns are provided.

Variable pattern
This is the simplest kind of pattern. It’s just a name (an identifier) and it is used to
capture the input into a variable (local to the specific arm). You can then use the
input using that variable. It never fails to match.

Wildcard pattern
This one is similar to variable pattern, except that no variable capture is made (it
also never fails to match). It’s an underscore in code.

Expression pattern
This pattern is represented either as a string literal, a number literal,true,false,
nil,anidentifierprefixedby$or an expression enclosed in$().Itdoesn’tcapture
and it may fail to match (it tests the input for equality with the given expression).

Table pattern
This pattern matches tables. It looks like a table literal. It can match both array and
hash members, where array members are written in the same way as expressions in an
array literal (except that they’re patterns) and hash members are written in the same
as well, where the key is an expression and the value is a pattern. When matching a
table, you have to match all the array members, but you don’t have to match all the
hash members. It may fail to match.

Object pattern
With this pattern you can match objects. It consists of an identifier or a$or a
$()expression followed by parens that contain a list of object member captures. A
capture can be a simple identifier (then it will capture a member of that name) or
a$or a$()expression followed by a colon and then by an identifier – in that case
it’ll match a member with the key the expression evaluates to into a variable after the
colon. You don’t have to capture all object members. It may fail to match.

Cons pattern
This binary pattern decomposes an input into two parts, head and tail. It has a
reversed meaning to the cons operator. The input has to be either a table or a list.
On a table this is slow, on a list this is fast (in case of table it has to actuall slice the
table). On invalid input it fails to match.
Binary patterns have their precedence and associativity. The cons pattern has the
highest precedence and is right associative. It looks the same as the cons operator.

And, or patterns
Binary patterns. Theorpattern has the lowest precedence, theandpattern is above
it, followed by the cons pattern. These are all binary patterns in Vortex. Bothand
andorpatterns are left associative. They may fail to match.

When pattern
This pattern is a conditional pattern. It consists of a pattern followed by the keyword
whenfollowed by a condition (which is an expression). It may obviously fail to match.

As pattern
This pattern essentially captures a pattern. It consists of a pattern followed by theas
keyword followed by another pattern. It’s particularly useful if you have for example
an expression pattern that doesn’t capture the input and you still want to capture it
besides checking its equality with the expression.


Some example code for pattern matching:
1 let x=5
2 // prints 5  variable pattern
3print(matchx>|y>y)
4
5 // prints 10, expressionand wildcardpatterns
6print(match 4 >
7|1> 2
8|2> 3
9|3> 4
10 | _>10)
11
12 // prints 5
13 print (match 4 >
14 | $ (2 + 3) asx>x
15 | $ (2 ∗ 2) asy>y+1)
16
17 // prints nil, doesn’t match
18 print (match 10 >
19 | xwhenx==5>x)
20
21 let tbl = { 5, 10, 15, foo: "bar",bar:"baz" }
22 // prints "baz", firstarm is incomplete
23 print (match tbl>
24 | { a , b , foo : c }>a
25 | { a , b , c , bar : d }>d)
26
27 let list = [ 5, 10, 15, 20 ]
28 // reversed list
29 let x=match list>
30 | a : : b : : c : : d>[d,c,b,a])

```
As previously mentioned, theletexpression also makes use of patterns. The pattern use is
limited though – only patterns that never fail to match can be used. The table pattern is
modified appropriately to allow use withlet–youcanmatchpartialcontentsofanarray
with it.
```
### 4.15 Objects

```
hobjectparentsi::=hsuffixedexpi
|‘(’hexplisti‘)’
```
```
hobjectimplctori::=‘[’[hidentlisti]‘]’
```
```
hobjectitemi::=hfndefi
|([‘$’]hidenti|‘$(’hexpi‘)’) ‘:’hexpi
```
```
hobjectbodyi::=hobjectimplctori[‘do’{hobjectitemi}‘end’]
|‘do’{hobjectitemi}‘end’
```
```
hobjectexpi::= ‘clone’[hobjectparentsi]hobjectbodyi
```
```
hnewexpi::=‘new’(hprimaryexpi|‘(’hexplisti‘)’)
```
```
hexpi::=hobjectexpi
| hnewexpi
```
```
Vortex features a builtin object system. It’s prototype based, delegative and supports mul-
```

```
tiple inheritance. It also has operator overloading. The object system is based around the
cloneexpression. It clones a parent object (or a set of parent objects in case of multiple
inheritance). If you don’t provide a parent object, it inherits from the internal Object table
(which provides the basic stuffconcerning objects). That means Object is the base for every
user defined object.
When inheriting from multiple parents, non-existent members are looked up from parents
from left to right depth-first.
Thecloneexpression cannot be used as a statement. Objects are first class values and the
cloneexpression evaluates to them, so you useclonein combination withlet.
Objects feature constructors, but they don’t have destructors. There are GC finalizers
provided in the same manner as the__gcmetamethod with tables. There is an implicit
constructor syntax – you can provide a list of identifiers in square brackets that represent
member names and Nth constructor argument will become Nth member in the brackets.
Constructors are never called when cloning an object.
There is also thenewexpression, which creates an “instance” of the object – that means, it
clones the object and calls the parent constructor on it. That allows you to pretty much
closely emulate classes without losing the flexible prototypal nature of Vortex’s object sys-
tem.
Vortex provides thesuperfunction in the standard library. It returns a proxy object on
which one can call parent methods. You can provide either one or two arguments. If you
have an objectx,whichisaninstanceofBar,whichisacloneofFoo,callingsuper(x):abc()
calls a method ofFoo(instead ofBar,whichisaparentofx)anditclalsitonx.Ifyou
provide two arguments, the first one is an object – a clone, the other one is the object on
which we want to call. Callingsuper(Bar, x)andsuper(x)is equivalent in this case. Some
examples:
```
1 let Foo =clonedo
2 fn __init(self,a,b)do
3print("I’ma constructor!")
4 set (self.a, self.b) = (a, b)
5 end
6
7 fn foo()do
8print("I’ma method without self, all alone")
9 end
10
11 fn bar(self)>print("I’ma method!", self.a, self.b)
12 end
13
14 let Bar =cloneFoodo
15 fn xyz(self)>Foo. bar (self)
16 end
17
18 let Baz =clone [a,b,c]
19
20 let inst =newBaz(5 , 10 , 15)
21 print ( i n s t. a , i n s t. b , i n s t. c )

### 4.16 Coroutines

```
hcorobodyi::= ‘->’hexpi
| harglistihexpscopei
```
```
hcoroexpi::=(‘coro’|‘cfn’)hcorobodyi
```

```
hyieldexpi::= ‘yield’hexporlisti
```
```
hexpi::=hcoroexpi
```
```
hstatexpi::= hyieldexpi
```
```
Ialreadydescribedcoroutinesabove,hereI’lldescribethemwhenitcomestotheirsyntax.
Vortex provides two variants of coroutines, thecorovariant and thecfnvariant. The for-
mer creates a true coroutine – a thread object that you can resume and so on. The latter
creates a simplified function form – it resumes when you call it, and there is no way to query
whether it’s already dead (except that it returns nil when it is and you call it).
Each of these has two forms again. The arrow form (the keyword followed by an arrow
and an expression) takes an arbitrary expression and makes it into a coroutine. The func-
tion form consists of the keyword, a function argument list (identical with a regular func-
tion) and a function body (an arrow followed by an expression or a block with or with-
out arrow). That one is just a shorthand, for examplecfn a, b -> bodyis the same as
cfn -> fn a, b -> body.
You can yield from a coroutine using theyieldexpression, which looks similar to thereturn
orresultexpression. You can resume a thread object using theresumefunction in the core
library. There is also the Lua library for coroutine manipulation which works just fine.
```
### 4.17 Sequences

```
hseqexpi::= ‘seq’hexpscopei
```
```
hexpi::=hseqexpi
```
```
Vortex features sequences. They evaluate to a tuple of expressions (not a first class value –
it’s similar to a function that returns multiple values). They’re basically coroutines. They
evaluate to the values you yield from them. That you can use for e.g. list comprehensions.
Example:
```
1 // this table isan array ofnumbers from 1 to 10
2 let x={seq>for i=1 .. 10>yield i}
3
4 //with blocks
5 let x={seqdo
6 for i=1 .. 5>yield i
7 yield 6
8 return 7 // also works, terminatesthe sequence
9 end }

### 4.18 Enumerations

```
henumitemi::=hidenti[‘:’hexpi]
```
```
henumexpi::=‘enum’‘(’henumitemi{‘,’henumitemi}‘)’
```
```
hexpi::=henumexpi
```
```
Vortex also features enumerations. They start with theenumkeyword followed by a list of
identifiers in parens. An enumeration represents a sequence of numbers (or custom expres-
sions). It evaluates to an associative array where the identifiers specified here are the keys.
```

```
Each of the keys has an associated value, by default numerical, starting from 0, incrementing
by 1 with each following member. Note that the order when iterating an enum is undefined.
```
1 let Foo =enum (A, B, C)
2 assert(Foo.A == 0andFoo .B = 1 and Foo .C = 2)
3
4 let Bar =enum (FOO: 5 , BAR, BAZ: 10 , BAH)
5 assert(Bar.FOO == 5 and Bar .BAR == 6
6 and Bar .BAZ == 10 and Bar .BAH == 11)

```
The expression you optionally provide when defining an enum member should be incre-
mentable.
```
### 4.19 Other expressions

```
hexpi::=‘__FILE__’
|‘__LINE__’
```
```
There are two other expressions. They’re very simple. The__FILE__expression expands
to the current filename at compile time. The__LINE__expression expands to the line the
expression is on, also at compile time.
```
## 5Macros

```
hmacroi::=‘macro’hidenti[hidentlisti][‘...’]hexpscopei
```
```
hquoteexpi::= ‘quote’(hsuffixedexpi|hblocki)
```
```
hunquoteexpi::= ‘unquote’(hsuffixedexpi|hblocki)
```
```
hexpi::=hquoteexpi
| hunquoteexpi
```
```
Vortex features an AST based macro system. That means it operates on AST level rather
than on text level – that way it can be context aware and much safer. The currently
implemented macro system is not hygienic (it can capture and create outer identifiers) but
it’s planned.
Amacrostartswiththemacrokeyword followed by the macro name and an argument list.
Then either an arrow followed by an expression or a block follows. This is very similar to
regular functions, but there are no default argument values and no named varargs.
To use a macro, you need to expand it. You do that using themacroname!(macroargs)
syntax (it’s a primary expression). It means that the macro will expand at that point,
substituting the expansion expression with some actual code.
As mentioned previously, a macro works on AST level – the macro has zero or more inputs
(that are serialized AST nodes) and it’s supposed to return an AST node, which is then
injected into the final AST at the expansion point.
Macro arguments are implicitly serialized into AST nodes. The expression you return from
the macro (using regularreturnor using the arrow notation) is not, so you need to serialize
it. For that purpose, there is thequoteexpression. It takes an arbitrary expression and
evaluates to the AST of it, converting any subexpressions into AST as well. For example
```
1 let x=quote (fna, bdo
2print("helloworld")
3 end)


```
results in a function AST node with all the contents serialized as well as the function itself.
There is one other expression,unquote.Itbasicallytellsthequoteexpression to “stop” at
that point. For example:
```
1 let a=quote (x + y)
2 let b=quote (y∗ z)
3
4 // this is a binary expression with
5 // operator/ containingtwosymbols
6 let c=quote (a / b)
7 // this is a binary expression withoperator
8 /// containingtwo binary expressions, a andb
9 let d=quote (unquote a/unquoteb)

```
With these facilities available, we can define a macro:
```
1 macromy_if(cond , texp , fexp )>quote
2(matchnot unquote cond>
3|true >unquote fexp
4|false>unquote texp)
5
6 let x=5
7print(my_if!(x==5,"hello", "world"))

```
Here it prints “hello”, because the condition is true. The unquotes are needed so that we
put the inputs themselves in the result, not just symbols.
```
## 6Theruntimeandthestandardlibrary

```
This reference manual does not cover the runtime nor the standard library. It references
some core functions, but does not define anything – please refer to the standard library
documentation for more.
```
## 7TheREPL

```
Vortex provides an interactive command line and a standalone script runner combined into a
single script. By default it launches an interactive session. There you can input statements.
Local variables are preserved.
```
```
> letx={ 5, 10, { 15, 20 } }
>=x// properoutput
{5,10,{15,20}}
```
```
The Vortex REPL in the default implementation is itself written in Vortex. It requires a
Lua module calledvxutil.ThismoduleisshippedwithVortexandiswritteninC–you
need to compile it and then put into a Lua C module search path (for example the current
directory). The C module provides support for signals,isattyand the readline library. All
functions have their fallbacks. For example, if you want readline support in the REPL (so
that you can move in the history and seek in the current input), you need to compile the C
module withVX_READLINEdefined. For proper function ofisattyyou need to compile the
module with eitherVX_POSIXorVX_WINdefined depending on your platform. If you don’t
define either of these, a fallback will always return true, which means autodetection whether
to launch an interactive session won’t work (whenisattyreturns false, it means something
is piped into the standard input, and the REPL tries to execute that).
```
```
usage : vortex [ options ] [ script [ args ]]
estr runthestring’str’
```

```
ienterinteractivemodeaftertheoptionsarehandled
llib requirelibrary’lib’
vshowversioninformation
 stop handling options
 stop handling options and run stdin
```
```
Here you can see the REPL options you can pass to it. As mentioned above, you can use
the REPL with pipes. For example
```
```
echo ’ print (" hello world")’|<replcommand>
```
```
should work.
If your interactive line starts with=followed by an expression, the REPL prints its value.
You can see an example in the very beginning of this section. Tables get special treatment
```
- they’re serialized before printing. That should allow you to see the values clearly.

## 8Appendix:Styleguide

```
These are recommended conventions for Vortex code. You don’t have to follow them, but
it’s strongly recommended for consistency.
```
### 8.1 Indentation

```
You use 4 spaces to indent each level. Tabs should be avoided. Try not to follow the Lua
convention of 2 spaces.
```
### 8.2 Whitespace

```
Put a space before and after a binary operator. Do not space offunary operators. Do not
put spaces around parens. You should put a space after each comma in the language. Do
not put a space between function name and its argument list. You can use spaces to align
things where needed.
```
### 8.3 Blocks

```
Keep thedokeyword on the same line with the preceeding code.
```
### 8.4 Naming style

```
The Vortex naming rules are simple and encourage readable code.
```
- Values, modules, functions and variables generally aresnake_case.
- Objects areThis_Case.
- Macro names aresnake_case. No need for uppercase because of obvious expansion
    syntax.

1 let foo = 5//Good
2 let Bar = 10 //Bad!
3
4 fnfoo_bar(a, b)>expr() //Good
5 fnfooBar(a, b)>expr() //Bad
6
7 //Good
8 let My_Object =clonedo
9 fnmy_method(self)>self.x


10 end
11 //Bad
12 let MyObject =clonedo
13 fnmyMethod(self)> self.x
14 end
15
16 //Good
17 macro foo(a, b)do ...end
18 print ( foo! ( 5 , 10))
19 //Bad
20 macroFOO( a , b ) do ...end
21 print (FOO! ( 5 , 10))

### 8.5 Examples

1 //Good!
2 let x=a+b
3
4 //Also good.
5 if foodo
6print("boo!")
7 end
8
9 //Alignment isgood
10 foo ( bar () ,
11 baz ( ) )
12
13 //Spacing makes things readable.
14 foo ( bar () , baz () , (5 + 10 ∗ (341) / 150))
15
16 //Functions, the goodway
17 fn foo(a, b)>hello(a, b)
18
19 //But this isbad.
20 let x=a+b
21
22 //This is badtoo.
23 if foo
24 do
25 print ("boo")
26 end
27
28 //Bad alignment
29 foo ( bar () ,
30 baz ( ) )
31
32 // Excessivespacing.
33 foo ( bar () , baz () , ( 5 + 10 ∗ (341)/150))
34
35 //A badlywritten function
36 fn foo (a, b)>hello ( a, b )

## 9Appendix:ThecompletesyntaxofVortex


halphai::=‘a-zA-Z’

hdigiti::= ‘0-9’

hhexdigiti::=‘0-9a-fA-F’

hwhitespacei::=‘’
|‘n’
|‘r’
|‘t’
|‘f’
|‘v’

hcommenti::=‘//’/.*$/
|‘/*’{/./|hcommenti}‘*/’

hidenti::=(‘_’|halphai){‘_’|halphai|hdigiti}

hidentlisti::=hidenti{‘,’hidenti}

hopkeywordi::=‘band’
|‘bor’
|‘bxor’
|‘asr’
|‘bsr’
|‘bsl’
|‘and’
|‘or’

hopkeywordassi::=‘band=’
|‘bor=’
|‘bxor=’
|‘asr=’
|‘bsr=’
|‘bsl=’

hkeywordi::=‘as’
|‘break’
|‘case’
|‘cfn’
|‘clone’
|‘coro’
|‘cycle’
|‘do’
|‘else’
|‘end’
|‘enum’
|‘false’
|‘fn’
|‘for’
|‘glob’
|‘goto’
|‘if’


```
|‘in’
|‘let’
|‘loop’
|‘macro’
|‘match’
|‘module’
|‘new’
|‘nil’
|‘quote’
|‘rec’
|‘redo’
|‘result’
|‘return’
|‘seq’
|‘set’
|‘true’
|‘unquote’
|‘when’
|‘while’
|‘with’
|‘yield’
|‘__FILE__’
|‘__LINE__’
| hopkeywordi
| hopkeywordassi
| hunopkeywordi
```
hunopkeywordi::=‘not’
|‘bnot’

hbinopi::=‘=’
|‘==’
|‘>’
|‘>=’
|‘<’
|‘<=’
|‘!=’
|‘%’
|‘’
|‘+’
|‘++’
|‘*’
|‘**’
|‘-’
|‘/’
|‘::’
| hopkeywordi

hassopi::=‘=’
|‘+=’
|‘++=’
|‘*=’
|‘**=’


#### |‘-=’

#### |‘/=’

#### |‘::=’

#### |‘%=’

```
| hopkeywordassi
```
hunopi::=‘-’
|‘#’
| hunopkeywordi

hothertoki::= ‘(’
|‘)’
|‘->’
|‘.’
|‘..’
|‘...’
|‘;’
|‘;;’
|‘$’
|‘$(’

hhexnumi::= (‘0x’|‘0X’) {hhexdigiti}[‘.’]{hhexdigiti}[(‘p’|‘P’) [ ‘+’|‘-’]
hdigiti]

hdecnumi::={hdigiti}[‘.’]{hdigiti}[(‘e’|‘E’) [ ‘+’|‘-’]hdigiti]

hnumliterali::= hhexnumi
| hdecnumi

hstrliterali::= hstrprefixi(hstrlongi|hstrshorti)

hstrprefixi::= /[eErR]*/

hstrshorti::=‘"’hstrshortelemi‘"’
|‘'’hstrshortelemi‘'’

hstrlongi::=‘"""’hstrlongelemi‘"""’
|‘'''’hstrlongelemi‘'''’

hstrshortelemi::=?shortstringcontents?
| hstresci

hstrlongelemi::=?longstringcontents?
| hstresci

hstresci::=‘a’
|‘b’
|‘f’
|‘n’
|‘r’
|‘t’
|‘v’
|‘z’
|‘"’


#### |‘'’

#### |‘’

hchunki::={hstatexpi[‘;’]}[(hretexpi|hresexpi)[‘;’]]

hmainchunki::= {(hstatexpi|hmacroi)[‘;’]}[(hretexpi|hresexpi)[‘;’]]

hmacroi::=‘macro’hidenti[hidentlisti][‘...’]hexpscopei

hblockendi::= ‘end’|‘;;’

hblocki::=‘do’hchunkihblockendi

hexpscopei::= ‘->’hexpi|hblocki

hstatscopei::=‘->’hstatexpi|hblocki

hfnscopei::= ‘->’(hexpi|hmatchbodyi)|hblocki

hifblocki::=‘do’hchunki

helseopti::= ‘else’[‘->’]exp

helsestatopti::= ‘else’[‘->’]statexp

hstatexpi::= hblocki
|(hidenti|hindexpi)hassopihexpi
| hsuffixedexpihtcallsuffixi
|‘if’hexpi(‘->’hstatexpi[helsestatopti]|hifblocki(‘end’|helsestatopti))
|‘let’[hlettypei](hpatterni|‘(’hpatternlisti‘)’) ‘=’hexporlisti
|‘set’(hsuffixedexpi|‘(’hexplisti‘)’)hassopihexporlisti
|‘with’(hpatterni|‘(’hpatternlisti‘)’) ‘=’hexporlistihstatscopei
|‘fn’([hlettypei]hidenti|hidenti[‘.’hidenti]) ‘(’harglisti‘)’hfnscopei
|‘match’hexplisti‘->’hmatchbodystati
| hretexpi
| hresexpi

hretexpi::= ‘return’hexporlisti

hresexpi::=‘result’hexporlisti

hexpi::=hstatexpi
| hexpihbinopihexpi
| hunopihexpi
| hprimaryexpi
| hsuffixedexpi
|‘match’hexplisti‘->’hmatchbodyi
|‘clone’[hobjectparentsi]hobjectbodyi
|(‘coro’|‘cfn’) (‘->’hexpi|harglistihexpscopei)
|‘if’hexpi(‘->’hexpi[helseopti]|hifblocki(‘end’|helseopti))
|‘with’(hpatterni|‘(’hpatternlisti‘)’) ‘=’hexporlistihexpscopei
|‘loop’[hloopcondi]hstatscopei[hloopcondi]
|‘for’hidenti‘=’hexpi‘..’hexpi[‘,’hexpi]hstatscopei
|‘for’hidentlisti‘in’hexplistihstatscopei
|‘fn’(harglisti|‘(’harglisti‘)’)hfnscopei


```
|‘yield’hexporlisti
|‘seq’hexpscopei
|‘new’(hprimaryexpi|‘(’hexplisti‘)’)
|‘quote’(hsuffixedexpi|hblocki)
|‘unquote’(hsuffixedexpi|hblocki)
|‘enum’‘(’hidenti[‘:’hexpi]{‘,’hidenti[‘:’hexpi]}‘)’
|‘break’
|‘cycle’
|‘redo’
|‘__FILE__’
|‘__LINE__’
```
hprimaryexpi::=‘(’hexpi‘)’
| htableexpi
|‘[’hexpi{‘,’hexpi}]‘]’
|‘$(’hexpi‘)’
|‘$’hidenti
| hidenti[‘!’‘(’hexplisti‘)’]
| hnumliterali
| hstrliterali
|‘nil’
|‘true’
|‘false’

hsuffixedexpi::=hprimaryexpihexpsuffixi

htableexpi::=‘{’[htableitemi{‘,’htableitemi}]‘}’

hexplisti::=[hexpi{‘,’hexpi}]

hexporlisti::=(hexpi|‘(’hexplisti‘)’)

hdefarglisti::=hidenti‘=’hexpi{‘,’hidenti‘=’hexpi}

harglisti::=[hidentlisti][hdefarglisti][[hidenti]‘...’]

hlettypei::= ‘rec’
|‘glob’

hloopcondi::=‘while’hexpi

htableitemi::=[([‘$’]hidenti|‘$(’hexpi‘)’) ‘:’]hexpi

hfcallsuffixi::=‘(’hexplisti‘)’
| htableexpi
| hstrliterali

hmcallsuffixi::= ‘:’hidentihfcallsuffixi

htcallsuffixi::=hfcallsuffixi
| hmcallsuffixi

hindexsuffixi::= ‘.’hidenti
|‘[’hexpi‘]’


```
hexpsuffixi::= [hexpsuffixi](hindexsuffixi|htcallsuffixi)
```
```
hobjectparentsi::=hsuffixedexpi
|‘(’hexplisti‘)’
```
```
hobjectimplctori::=‘[’[hidentlisti]‘]’
```
```
hobjectitemi::=hfndefi
|([‘$’]hidenti|‘$(’hexpi‘)’) ‘:’hexpi
```
```
hobjectbodyi::=hobjectimplctori[‘do’{hobjectitemi}‘end’]
|‘do’{hobjectitemi}‘end’
```
```
hobjectpatitemi::=([ ‘$’]hidenti|‘$(’hexpi‘)’) [ ‘:’([‘$’]hidenti|‘$(’hexpi‘)’) ]
```
```
hobjectpatbodyi::=hobjectpatitemi{‘,’hobjectpatitemi}
```
```
htablepatitemi::=[([‘$’]hidenti|‘$(’hexpi‘)’) ‘:’]hpatterni
```
```
hprimarypatterni::=‘(’hpatterni‘)’
| hstrliterali
| hnumliterali
|‘true’
|‘false’
|‘nil’
|‘_’
|[‘$’]hidenti[‘(’[hobjectpatbodyi]‘)’]
|‘$(’hexpi‘)’[‘(’[hobjectpatbodyi]‘)’]
|‘{’htablepatitemi{‘,’htablepatitemi}‘}’
```
```
hsuffixedpatterni::= hprimarypatterni
| hsuffixedpatterni‘when’hexpi
| hsuffixedpatterni‘as’hexpi
```
```
hpatterni::=hsuffixedpatterni[(‘and’|‘or’|‘::’)hsuffixedpatterni]
```
```
hmatcharmi::=(‘|’|‘case’)hpatternlistihexpscopei
```
```
hmatcharmstati::=(‘|’|‘case’)hpatternlistihstatscopei
```
```
hmatchbodyi::=hmatcharmi{hmatcharmi}
```
```
hmatchbodystati::=hmatcharmstati{hmatcharmstati}
```
## 10 Appendix: Influences

Vortex is a language with many influences. It follows the same principles as Lua, on which
is built. Thus, you can find many similarities in Lua’s and Vortex’s syntax and semantics as
well as the core set of features. Vortex and Lua are both built around tables, the ultimate
do-it-all data structure. The reliance on higher order and first class functions is also taken
very seriously in both languages.
The second largest influence for Vortex is OCaml. Vortex takes many primarily syntactic
features from OCaml, including the pattern matching syntax. Vortex is however, unlike
OCaml, an untyped language. The features are thus adjusted accordingly for Vortex.
OCaml’s relative, F#, is also influental for Vortex. The basic idea of sequences in place of


list comprehensions is taken from F#, but unlike F# sequences are not values in Vortex,
instead they’re simply tuples of values that can’t be treated in a first class manner.
The prototype based multiple inheritance object system of Vortex is inspired by Io. Com-
pared to Io, the constructors work differently and Vortex’s object system allows you to pretty
much closely imitate classes (those are handled a bit differently in Io).
Scheme inspired Vortex in how simple a language can be while remaining very powerful.
Along with Elixir, Scheme provided a basis for Vortex AST macros.
Elixir and Ruby influenced Vortex’s syntax. The Rust language provided an example how
terse a syntax can be and several of Vortex’s keywords are or were taken from Rust. The
cyclekeyword is taken from Fortran. String literal syntax is taken from Python.
Other languages that influenced Vortex to some degree are ALGOL, C, CLU, Haskell and
Self.



