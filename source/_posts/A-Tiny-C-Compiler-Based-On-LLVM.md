---
layout: a
title: A-Tiny-C-Compiler-Based-On-LLVM
subtitle: 一个类C语言编译器的实现
date: 2020-05-17 17:09:43
tags: LLVM Compiler Linux
---


<font face="黑体" color=black size=6>A Tiny C Compiler based on LLVM</font>

![](RackMultipart20200617-4-yuiech_html_3e6dc1cda4436e6.png)

**本科实验报告**




2020 年 5 月 31 日

# Catalog

[Ⅰ. Introduction 3](#_Toc41853754)

[Ⅱ. lexical Analysis 5](#_Toc41853755)

[Ⅲ. Parsing 7](#_Toc41853756)

[Ⅳ. Semantic Analysis 9](#_Toc41853757)

[Ⅴ. Optimization Considerations 15](#_Toc41853758)

[Ⅵ. Code Generation 16](#_Toc41853759)

[Ⅶ. Test Cases 20](#_Toc41853760)

## Ⅰ. Introduction

1. **Description**

Our work is a compiler of a C-like language. We have implemented some basic characteristics of the C language, including but not limited to variable, function, loop statement, conditional statement, assignment statement.

- **Variable declaration** :

    Syntax:

    e.g.

    int x;

    double y;

    char c;

Three basic types are supported: int, double, char.

- **Function declaration** :

Syntax:

e.g.

int func(int a,int b);

- **Function definition** :

Syntax:

e.g.

int SumOfTwoInt(int a,int b){

int c;

c=a+b;

return c;

}

- **Loop statement** :

&quot;for&quot; loop:

Syntax:

e.g.

for(int i=0;i\&lt;3;i++) printf(&quot;Wa\n&quot;);

for(int i=0;i\&lt;3;i++){

printf(&quot;W&quot;);

printf(&quot;a&quot;);

printf(&quot; Hello, world!\n&quot;);

}

&quot;while&quot; loop:

Syntax:

e.g.

int a;

a=0;

while(a\&lt;3) a++;

while(a\&lt;6){

a++;

printf(&quot;Wa\n&quot;);

}

- **Conditional statement** :

&quot;if-else&quot; statement:

Syntax:

or

e.g.

int a; a=0;

if(a\&lt;0) a=1;

else a=2;

if(a\&lt;2){a=3;a++;}

else{a=4;}

- **Assignment statement** :

Syntax:

e.g.

int a;

a=1;

a+=1; a-=1; a/=1; a\*=1; a%=1;

a\&lt;\&lt;=1; a\&gt;\&gt;=1;

a^=1; a|=1; a&amp;=1;

1. **Files**

**Necessary files:**

**Scanner.l parser.y grammartreenode.h grammartreenode.cpp func.h cmm.out**

**Makefile demo.cpp and some other test files**

**Instructor files:**

**README.h AST.md Show.txt**

**Helpful files:**

**demo.cpp demo.cpp.ll demo.cpp.s demo.cpp.out**

**t1.c t2.c t3.c t4.c**

1. **Jobs assignment**

**Lexical**** ： ****汪俊威，康锦辉，林成楠**

**Grammar**** ： ****汪俊威，康锦辉**

**Senmatic**** ： ****汪俊威，林成楠**

**IR codes**  **：**** 汪俊威，康锦辉**

**Machine codes**** ：汪俊威**

**Report**  **：**** 康锦辉，林成楠，汪俊威**

**PPT :**  **康锦辉**

## Ⅱ. lexical Analysis

The task of lexical analysis is to scan the source program from left to right,Identify each type of word according to the lexical rules of the language and generate the corresponding token.

First, we give a set of formal definitions and state definitions to simplify the lexical rules that follow.

D [0-9] //Digit

L [a-zA-Z] //letter

H [a-fA-F0-9] //hexadecimal

E ([Ee][+-]?{Digit}+) //Scientific count

P ([Pp][+-]?{Digit}+) //Binary scientific counting

FS (f|F|l|L) //Data type definition

IS ((u|U)|(u|U)?(l|L|ll|LL)|(l|L|ll|LL)(u|U)) //Data type definition

Second, identify the identifier and fixed symbol, generate Numbers

| identifier |
| --- |
| auto | sizeof | goto | return | do | while | for | continue |
| break | switch | default | case | if | else | float | double |
| char | void | int | long | bool | const | short | signed |
| unsigned | static | extern | inline | typedef | struct | enum | union |

| Fixed symbols |
| --- |
| ... | \&gt;\&gt;= | \&lt;\&lt;= | += | -= | \*= | /= | %= |
| &amp;= | ^= | |= | \&gt;\&gt; | \&lt;\&lt; | ++ | -- | -\&gt; |
| &amp;&amp; | || | \&lt;= | \&gt;= | == | != | ; | , |
| : | = | . | &amp; | ! | ~ | - | + |
| \* | / | % | \&lt; | \&gt; | ^ | | | ? |
| ( | ) | [|] | { | } |
 |
 |

Third, lexical rules are used to define variables and constants

{L}({L}|{D})\* //variable

0[xX]{H}+{IS}? //int constant

0[0-7]\*{IS}? //intconstant

[1-9]{D}\*{IS}? //int constant

{D}+{E}{FS}? //double constant

{D}\*&quot;.&quot;{D}+{E}?{FS}? //double constant

{D}+&quot;.&quot;{D}\*{E}?{FS}? //double constant

Fourth, skip the comments and Spaces.

&quot;//&quot;[^\n]\* //Handle &quot;//&quot; class comments

&quot;/\*&quot; //handle /\*\*/ class comments with mutiline\_comment() function

[\t\v\n\f] //Ignore the blank space

## Ⅲ. Parsing

1. **LALR(1)**

An LALR parser or a Look-Ahead LR parser is a simplified version of a canonical LR parser, using bottom-up method, to parse (separate and analyze) a text according to a set of production rules specified by a formal grammar for a programming language. And LALR(1) is a specific version of LALR, which means looking ahead for one word.

1. **Yacc**

We use Yacc to generate a syntax analyzer. Yacc is a LALR(1) parsing generator. We define derivation rules and corresponding behaviors in a Yacc file, and after processing, it would generate a C header file and a C++ source file which include tools, or functions, which can be used to parse the syntax. In our work, we use Bison as a distro of Yacc.

1. **Abstract syntax tree**

The result of parsing would be stored in an abstract syntax tree (AST).

The structure of an AST node:

class ASTnode{

public:

std::string content;

std::string name;

int line\_no;

int intval;

double douval;

float floval;

std::string vtype;

bool booval;

ASTnode\* left;

ASTnode\* right;

ASTnode();

};

The attribute is the type of node.

The attribute is the specific content of node, like the name of some variable, some constant, or some symbol.

The attribute is the corresponding line number of the node.

The left child of a node points to its first sub-node at the next level, and the right child of a node points to its neighbored node.

We construct the AST when parsing. As we use Yacc to generate a syntax analyzer, we define the construction process in the behaviors that would be executed when some derivation is matched. In fact, the construction process also occurs when analyzing lexical, since tokens are terminals in derivations and leaf nodes in the abstract syntax tree.

The detail of derivations can be seen in source files.

![](RackMultipart20200617-4-yuiech_html_e3db9eabdffc7c9c.png) ![](RackMultipart20200617-4-yuiech_html_e3db9eabdffc7c9c.png)

## Ⅳ. Semantic Analysis

We analyze semantics from the root of the abstract syntax tree. We analyze the type of the node we meet, and decide the specific semantic according to the structure of the node and its child-nodes, until meeting a node without child-node, which means the mode we meet represents a terminal.

1. **Attributes computing**

We use the attributes between an AST node and its child-nodes to compute and pass the attributes.

In our consideration, only expressions have attributes to pass and compute. For simplicity, we present the our methods to compute attributes with derivations simplified and abbreviated.

1. **Symbol table**

We use symbol tables to store the maps between identifiers and variables. When it comes to a new nest, we create a new table based on the current table, like the structure of a stack. When we need to lookup the symbol tables to find some variable, we search the most recent table, or the table on the top of table stack first, and then the second recent table, and so on. If all tables are searched but no matched variable found, we process this situation as an error.

The special structure of the symbol table:

LLVM::Module::getValueSymbolTable()

Std::map\&lt;std::string,LLVM::Value\*\&gt;

1. **Type check**

We check the type of variables at a lot of positions, like function calls, variables operations, and function definitions. We here display some examples about our type check work with pseudo codes, and more details can be seen in the source code files.

**Function calls** :

We check the return type and parameter type when we process a function call. If the return type doesn&#39;t match the caller, or a parameter type doesn&#39;t match the declaration of the function, we consider the situation an error.

if(node-\&gt;name==&quot;postfix\_expr&quot;

&amp;&amp; node-\&gt;left-\&gt;name==&quot;identifier&quot;

&amp;&amp; node-\&gt;left-\&gt;right-\&gt;name==&quot;(&quot;

&amp;&amp; node-\&gt;left-\&gt;right-\&gt;right-\&gt;name==&quot;arg\_expr\_list&quot;){

//postfix\_expr -\&gt; identifier &#39;(&#39; arg\_expr\_list &#39;)&#39;

func = lookup\_func\_table(node-\&gt;left);

args = get\_args(node-\&gt;left-\&gt;right);

check(args, func.paras){

if(unmatch)error;

}

......

}

**Variable operations** :

We check the types of variables when making variable operations. For example, when we make bit shift operations, we need to check whether the two operands are both integer and report an error if not. In fact, this is a part of work in the attribute computing.

e.g.

if(node-\&gt;name==&quot;shift\_expr&quot;

&amp;&amp; node-\&gt;left-\&gt;name==&quot;shift\_expr&quot;

&amp;&amp; node-\&gt;left-\&gt;right-\&gt;name==OP\_LSHIFT

&amp;&amp; node-\&gt;left-\&gt;right-\&gt;right-\&gt;name==&quot;add\_expr&quot;){

//shift\_expr -\&gt; shift\_expr OP\_LSHIFT add\_expr

tmp1 = compute(node-\&gt;left);

tmp2 = compute(node-\&gt;left-\&gt;right-\&gt;right);

if(tmp1.type!=int|| tmp2.type!=int)error;

......

}

**Function definitions** :

We check the parameter types when we process a function definition by comparing them with the declaration of the function and report an error if there is any unmatched parameter type.

function process\_func\_def(ASTnode\* cur\_node){

paras = get\_paras(cur\_node);

func\_name = get\_func\_name(cur\_node);

decl = get\_decl(func\_name);

check(decl.paras, paras){

if(unmatch)error;

}

......

}

## Ⅴ. Optimization Considerations

The LLVM IR can automatically do some optimization like the following translation:

Main.cpp:

intmain(){

int i = 23 + 14\*3 – 12/2%2

return0;

}

Main.ll

def i32 @main(){

entry:

%i = alloca i32 , align 4

Store i32 63, i32\* %i, align 4

Ret i32 0

}

Automatically calculating the results of 23 + 14\*3 – 12/2%2.

## Ⅵ. Code Generation

1. **Intermediate codes**

We generate the intermedia codes when analyzing the semantics. We use LLVM to generate intermediate codes, which is a is a set of compiler and toolchain technologies, and can be used to develop a front end for any programming language and a back end for any instruction set architecture.

LLVM is designed around a language-independent intermediate representation (IR) that serves as a portable, high-level assembly language that can be optimized with a variety of transformations over multiple passes.

The standard of intermediate codes should be LLVM IR since we deicide to use the frame of LLVM to finish the work from intermediate codes to assembly codes, and the finial executable file. LLVM IR is something like the three-address code, and has a huge amount of specifications. To ensure the specifications of the intermediate codes we would generate, we apply some auxiliary tools given by LLVM to our code generator, like , , , , , , , and so on.

We create a new translation module at the beginning of generating process, and insert functions or some declarations into the module. When we obtain the attributes of some statement from computing, we find an appropriate location and insert codes after sorting out the attributes and the information of nodes. For example, when we process an AST node of additive expression type, we compute the attributes of its child-nodes, insert the codes about the processing procedure, and return the result.

Value\* additive\_exp(node head =nullptr, Module\* m =nullptr){

if(head-\&gt;left-\&gt;right){

Value\* l = additive\_exp(head-\&gt;left,m);

Value\* lval = isptr?builder.CreateLoad(l):l;

Value\* r = multiplicative\_exp(head-\&gt;left-\&gt;right-\&gt;right);

Value\* rval = isptr?builder.CreateLoad(r):r;

string type = head-\&gt;vtype;

isptr =false;

if(head-\&gt;left-\&gt;right-\&gt;name==&quot;+&quot;){

if(type==&quot;int&quot;)

return builder.CreateAdd(lval,rval);

else

return builder.CreateFAdd(lval,rval);

}elseif(head-\&gt;left-\&gt;right-\&gt;name==&quot;-&quot;){

if(type==&quot;int&quot;)

return builder.CreateSub(lval,rval);

else

return builder.CreateFSub(lval,rval);

}

}else{

return multiplicative\_exp(head-\&gt;left,m);

}

}

More detail can be seen in the source code files. Here is a catalog:

intmain(int argc, char\* argv[]){

if(argc\&gt;1){

yyin=fopen(argv[1],&quot;r&quot;);

}else{

yyin=stdin;

}

yyparse();

printf(&quot;\n&quot;);

treePrint(root,0);

gen(root,argv[1]);

std::string cmd = &quot;llc &quot;+argv[1]+&quot; -o &quot;+ argv[1]+&quot;.s &amp; gcc &quot;+argv[1]

+&quot;.s -o&quot;+argv[1]+&quot;.out&quot;

system(cmd.c\_str());

fclose(yyin);

return0;

}

The function main() is the entry of source codes to target codes or IR codes .

The function gen() helps with translating AST to LLVM IR codes , whose definition is located in func.h file. And the gen() starts to call necessary functions to help navigate from the rootNode of the AST to the leafNode . meaning the Program -\&gt; external\_exp -\&gt; --- -\&gt; terminal symbols .

![](RackMultipart20200617-4-yuiech_html_571c357abecc5721.png)

The code generation follows a Top-Down policy , that is generating from the root node of the AST tree.

**gen(root,argv[1])** is the enter of the whole generating procedure.

And then following all the possible ways to the teminal symbols , while calling functions. For examples,

1. gen(root,argv[1]
2. genfunc(node-\&gt;left)
3. genfuncdef(node-\&gt;left)
4. genstmt(node-\&gt;left)
5. expression(node-\&gt;left)
6. assignment\_exp(node-\&gt;left)
7. conditional-\&gt;exp(node-\&gt;left)
8. …
9. primary\_exp(node-\&gt;left)

**You can view the func.h file int the src codes for more details**

1. **Assembly codes**

Fortunately, LLVM provides a quick way to generate assembly codes from LLVM IR by **llc** src.ll -o src.s

Example:

.text

.file &quot;for.c&quot;

.globl main # -- Begin function main

.p2align 4, 0x90

.type main,@function

main: # @main

.cfi\_startproc

# %bb.0: # %entry

movl $0, -4(%rsp)

movl $0, -8(%rsp)

xorl %eax, %eax

retq

.Lfunc\_end0:

.size main, .Lfunc\_end0-main

.cfi\_endproc

# -- End function

.section &quot;.note.GNU-stack&quot;,&quot;&quot;,@progbits

1. **Executive codes**

Any C compiler gcc or clang can compile asm codes (.s files) to target machine codes.

By **clang src.s -o src.out**

## Ⅶ. Test Cases

**Case 1** : t1.c

Source code:

int ans;intgcd(int a,int b){

if(b==0){

return a;

}else{

returngcd(b,a%b);

}

}

intmain(){

ans = gcd(9,36)\*gcd(3,6);

println(&quot;ans = %d&quot;,ans);

return0;

}

Result:

![](RackMultipart20200617-4-yuiech_html_1872fe407de2be21.png)

![](RackMultipart20200617-4-yuiech_html_80bfcbd9b65b354a.png)

**Case 2** : t2.c

Source code:

int f,k;

intgo(int b,int a){

int fk;

double t;

if(a\&gt;0){

return a\*go(b,a-1);

}else{

return1;

}

}

intmain(){

k = 0;

f = go(k,5);

println(&quot;%d\n&quot;,f);

println(&quot;%d\n&quot;,k);

return0;

}

Result:

![](RackMultipart20200617-4-yuiech_html_835061118e1f8f42.png)

![](RackMultipart20200617-4-yuiech_html_ee02274515ba97ab.png)

**Case 3** : t3.c

Source code:

intmain(){

int a = 13%5;

a += 3+2\*2-12/4;

println(&quot;a = %d&quot;,a);

int b = a==4 ? 1 : 2;

a+= a,b,1,2,3,4;

println(&quot;a = %d&quot;,a);

while(a\&lt;100){

b+=a;

a+=1;

}

println(&quot;a = %d&quot;,a);

println(&quot;b = %d&quot;,b);

return0;

}

Result:

![](RackMultipart20200617-4-yuiech_html_67eeb83ce3ab0693.png)

![](RackMultipart20200617-4-yuiech_html_77e65c0a30f4f72.png)