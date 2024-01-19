---
layout: post
title:  "Dymystifying declarations in C language."
date:   2024-01-14 6:00:00 +0900
categories: jekyll update
---

# Introduction

Nice to meet you, my name is Kuma. I am currently working on my own C formatter called [mic](https://github.com/wo4mei3/mic). I intend to develop it into my own original c dialect to raw c transpiler someday. I chose OCaml as my language of choice because I feel that functional languages are cool. There is still a lot of work that needs to be done, and I am already feeling a bit overwhelmed, but I want to finish it.

Now, I would like to explain how to read declarations in C language.
The only two things that would be desirable to know when reading this article are the BNF notation and the rightmost derivation, but you don't have to know them at all since they will be explained step by step later.

After reading this article, you will be able to mechanically decipher any complex declarations using a unified and theoretical methodology such as rightmost derivation, without having to deal with the concept of precedence, which is often introduced in documents around the world.

However, it is necessary to be familiar with the syntax rules of C declarations, especially declarator and direct-declarator.

However, the syntax rules themselves are simple, so you should be able to get used to them quickly.

The syntax rules in this article are a modified version of the ones in K&R C, the bible for C programmers, which is often cited as a reference for C syntax rules.

# BNF Notation and Rightmost Derivation
(Supplement: This chapter is written based on the following article. I think these aratcles are easier to understand)
[https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form)
[https://en.wikipedia.org/wiki/Context-free_grammar#Derivations_and_syntax_trees](https://en.wikipedia.org/wiki/Context-free_grammar#Derivations_and_syntax_trees)

BNF notation is a notation used to define the grammar of formal languages such as programming languages, and consists of a collection of syntax rules with a non-terminal symbol (surrounded by <>) on the left side and multiple symbols (including both terminal and non-terminal symbols) on the right side.

For example, the BNF notation for binary numbers is defined as follows.
```
(1) <bin_num> ::= <bin_seq>
(2) <bin_seq> ::= <bin_digit>
(3) <bin_seq> ::= <bin_digit> <bin_seq>
(4) <bin_digit> ::= 0
(5) <bin_digit> ::= 1
```

An arbitrary binary number can be obtained by replacing the non-terminal symbols on the left-hand side of each of the syntax rules from (1)~(5) with the sequence of symbols on the right-hand side. The goal is reached when the replacement is completed starting from the starting symbol \<bin_num> and ending with the symbol sequence of terminal symbols 0,1.

For example, the binary number 101 can be obtained as follows.
```
<bin_num> 
-> <bin_seq> #(1) applied
-> <bin_digit> <bin_seq> #(3) applied
-> 1 <bin_seq> #(5) is applied
-> 1 <bin_digit> <bin_seq> #(3) is applied
-> 1 0 <bin_seq> #(4) is applied
-> 1 0 <bin_digit> Apply #(2)
-> 1 0 1 #Apply (5)
```
Derivation is the process of applying a syntax rule to a non-terminal symbol, called a start symbol, to obtain an arbitrary expression, as described above.

There are two strategies for derivation. One is leftmost derivation, in which syntax rules are applied sequentially starting from the left-most non-terminal symbol and replacing the symbol, and the other is rightmost derivation, in which syntax rules are applied sequentially starting from the right-most non-terminal symbol.

Now let's look at how to get the binary number 101 with rightmost derivation.
```
<bin_num>
-> <bin_seq> #apply (1)
-> <bin_digit> <binseq> #apply (3)
-> <bin_digit> <bin_digit> <bin_seq> #(3) is applied
-> <bin_digit> <bin_digit> <bin_digit> <bin_digit> #(2) is applied
-> <bin_digit> <bin_digit> 1 #Apply (5)
-> <bin_digit> 0 1 #Apply (4)
-> 1 0 1 #Apply (5)
```
You got it successfully!

In this article, we will use this latter rightmost derivation to derive C declarations.


# Syntax of the declarative part of the C language
Now, let's see how the BNF definition of the actual declaration part of the C language looks like.
It is as follows.
```
(1) <declaration> ::= <declaration-specifier-list> <declarator>

(2) <declaration-specifier-list> ::= <declaration-specifier>
(3) <declaration-specifier-list> ::= <declaration-specifier-list> <declaration-specifier>
(4) <declaration-specifier> ::= <identifier>


(5) <declarator> ::= * <declarator>
(6) <declarator> ::= * const <declarator>
(7) <declarator> ::= <direct-declarator>

(8) <direct-declarator> ::= <identifier>
(9) <direct-declarator> ::= ( <declarator> )
(10) <direct-declarator> ::= <direct-declarator> [ <integer> ]
(11) <direct-declarator> ::= <direct-declarator> ( <parameter-type-list> )

(12) <parameter-type-list> ::= #empty
(13) <parameter-type-list> ::= <declaration
(14) <parameter-type-list> ::= <parameter-type-list> , <declaration>

```
Ugh, that's a long story.
It looks complicated, but there are only a few rules we need to be aware of to figure out the type of a declaration.
The important syntax rules here are (1), (5), (6), (8), (9), (10), and (11).

The left-hand side\<declaration> of syntax rule (1) is the starting symbol here. We start the replacement from here.

When syntax rules (5),(6) are applied, we are reading pointer types.

Syntax rule (8), here we can get the name of the variable being declared.

Syntax rule (9), thanks to this rule we have a nested syntax rule called \<declarator>.

Once syntax rule (10) is applied, we are reading an array type.

When syntax rule (11) applies, we are reading function types.

Of these, only three rules are particularly noteworthy: (5) for pointer types, (10) for array types, and (11) for function types.
 
When these syntax rules are applied, we will have a stack for each declarations to keep track of what rules were applied.

This stack will contain "pointer of" when a rule corresponding to a pointer type is read ("const pointer of" if it is a const pointer type), "array [ " + integer + " ] of" when a rule corresponding to an array type is read, "array of" when a rule corresponding to a function type When a rule corresponding to a function type is read, various strings such as "function ( " + parameter-type-list + " ) returning" are inserted.

After all the replacements of the part of the \<declarator> in the syntax rule (1) are done, the strings in the stack are taken out from the top here and arranged from left to right adding the variable name obtained by the syntax rule (8) for the leftmost position.
Finally, by adding the symbols obtained in the declaration-specifier-list to the rightmost position, we have the English readings corresponding to the declarations.

Now, let's try out the actual right-most derivation of various C declarations in the next section.

# Right-most derivation of declarations
### First, let's try rightmost-deriving the following declaration. 
>char *const ptr [4]

```
<declaration>
-> <declaration-specifier-list> <declarator> # Apply (1)
-> <declaration-specifier-list> * const <declarator> # Apply (6) Put "const pointer of" in the stack.
-> <declaration-specifier-list> * const <direct-declarator> [ 4 ] #(10) apply "array [ 4 ] of" to the stack.
-> <declaration-specifier-list> * const ptr [ 4 ] #Apply (8)
-> <declaration-specifier-list> * const ptr [ 4 ] #Apply (2),(4).
```
Here the stack looks like this from the top.
```
"array [ 4 ] of"
"const pointer of"
```
This is taken from the top and arranged from left to right.

"array [ 4 ] of" "const pointer of"

This is then connected to the "ptr" from the syntax rule (8), and then to the "char" from the rest of the declaration specifier list.
The result is as follows

"ptr array [ 4 ] of const pointer of char".

We now have a simple English notation for the declaration. I think this was done very simply and mechanically.
### Next we will look at a slightly modified version of above declaration.
>char (*const ptr)[4]

```
<declaration>
-> <declaration-specifier-list> <declarator> #(1) applied
-> <declaration-specifier-list> <direct-declarator> [ 4 ] # Apply (10) Put "array [ 4 ] of" in the stack.
-> <declaration-specifier-list> ( <declarator> ) [ 4 ] #(9) applies
-> <declaration-specifier-list> ( * const <declarator> ) [ 4 ] # Apply (6) Put "const pointer of" in the stack.
-> <declaration-specifier-list> ( * const <direct-declarator> ) [ 4 ] #Apply (7)
-> <declaration-specifier-list> ( * const ptr ) [ 4 ] # Apply (8)
-> char * const ptr [ 4 ] #Apply (2),(4).
```
Here the stack looks like this from the top.
```
"const pointer of"
"array [ 4 ] of"
```
This is taken from the top and arranged from left to right.

"const pointer of" "array [ 4 ] of"

This is then connected to the "ptr" from the syntax rule (8), and then to the "char" from the rest of the declaration specifier list.
The result is as follows

"ptr const pointer of array [ 4 ] of char".

You can see that the order of reading pointers and arrays has changed from the previous declaration because the declarator is nested.

### Next, let's look at a more complex example.
>bool (*(*p)[10])(int a, int b)


```
<declaration>
-> <declaration-specifier-list> <declarator> #(1) applied
-> <declaration-specifier-list> <direct-declarator> #(7) applies
-> <declaration-specifier-list> <direct-declarator> ( <parameter-type-list> ) #(11) applies
-> <declaration-specifier-list> <direct-declarator> ( <parameter-type-list> , <declaration> ) #Look separately at the <declaration> that appears here.
```

Since the \<declaration> newly appeared at the end of the \<parameter-type-list>, let's prepare a new stack here and derive ```int b```.

```
<declaration>
-> <declaration-specifier-list> <declarator> #(1) applies
-> <declaration-specifier-list> <direct-declarator> # apply (7)
-> <declaration-specifier-list> b # apply (8)
-> int b # apply (2), (4)
```
There is nothing in the stack, so this reads as is,

"b int".

Let's go back.

```
<declaration>
-> <declaration-specifier-list> <declarator> #(1) applied
-> <declaration-specifier-list> <direct-declarator> #apply (7)
-> <declaration-specifier-list> <direct-declarator> ( <parameter-type-list> ) #(11) applies
-> <declaration-specifier-list> <direct-declarator> ( <parameter-type-list> , int b ) #I was able to derive int b.
-> <declaration-specifier-list> <direct-declarator> ( <declaration> , int b ) #apply (13)
-> <declaration-specifier-list> <direct-declarator> ( int a , int b ) # Put "function ( a int , b int ) returning" in the same stack as for the previous <declaration>.
-> <declaration-specifier-list> ( <declarator> ) ( int a , int b ) # Apply (9).
-> <declaration-specifier-list> ( * <declarator> ) ( int a , int b ) #(5) is applied Put "pointer of" in the stack.
-> <declaration-specifier-list> ( * <direct-declarator> ) ( int a , int b ) #(7) is applied
-> <declaration-specifier-list> ( * <direct-declarator> [10] ) ( int a , int b ) # Apply (10) Put "array [ 10 ] of" in the stack.
-> <declaration-specifier-list> ( * ( <declarator> ) [10] ) ( int a , int b ) #apply (9)
-> <declaration-specifier-list> ( * ( * <declarator> ) [10] ) ( int a , int b ) #(5) is applied Put "pointer of" in the stack.
-> <declaration-specifier-list> ( * ( * <direct-declarator> ) [10] ) ( int a , int b ) #(7) applies
-> <declaration-specifier-list> ( * ( * p ) [10] ) ( int a , int b ) #(8) is applied
-> bool ( * ( * p ) [10] ) ( int a , int b ) #apply (2), (4)
```
Here the stack looks like this from the top.
```
"pointer of"
"array [ 10 ] of"
"pointer of"
"function ( a int , b int ) returning"
```
This is taken from the top and arranged from left to right.


"pointer of array [ 10 ] of pointer of function ( a int , b int ) returning"




This is followed by the "p" from the syntax rule (8), and then by the "bool" from the rest of the declaration specifier list.
The result is as follows


"pointer of array [ 10 ] of pointer of function ( a int , b int ) returning bool".


We now have a simple English notation for the declaration. From now on, we no longer have to be afraid of complicated declarations.


### Finally, let's look at the infamous function declarations of the standard C library.
>void (*signal(int sig, void(*func)(int a)))(int b)


```
<declaration>
-> <declaration-specifier-list> <declarator> #(1) applies.
-> <declaration-specifier-list> <direct-declarator> #(7) applies
-> <declaration-specifier-list> <direct-declarator> ( <parameter-type-list> ) #(11) applies
-> <declaration-specifier-list> <direct-declarator> ( <declaration> ) #(13) applies
-> <declaration-specifier-list> <direct-declarator> ( int b ) #I was able to derive int b. Put "function ( b int ) returning" in the stack.
-> <declaration-specifier-list> ( <declarator> ) ( int b ) #apply (9)
-> <declaration-specifier-list> ( * <declarator> ) ( int b ) #(5) is applied Put "pointer of" in the stack.
-> <declaration-specifier-list> ( * <direct-declarator> ) ( int b ) #(7) is applied
-> <declaration-specifier-list> ( * <direct-declarator> ( <parameter-type-list> ) ) ( int b ) #(11) applies
-> <declaration-specifier-list> ( * <direct-declarator> ( <parameter-type-list> , <declaration> ) ) ( int b ) #(14) applies
```
Let's derive the ```void(*func)(int a)``` part from the ```<declaration>``` that appears here. We will also prepare a new stack.
```
<declaration>
-> <declaration-specifier-list> <declarator> #apply (1)
-> <declaration-specifier-list> <direct-declarator> #(7) applied
-> <declaration-specifier-list> <direct-declarator> ( <parameter-type-list> ) #(11) applies
-> <declaration-specifier-list> <direct-declarator> ( <declaration> ) #(13) applies
-> <declaration-specifier-list> <direct-declarator> ( int a ) #int b could be derived. Put "function ( a int ) returning" in the stack.
-> <declaration-specifier-list> ( <declarator> ) ( int a ) #apply (9)
-> <declaration-specifier-list> ( * <declarator> ) ( int a ) #(5) is applied Put "pointer of" in the stack.
-> <declaration-specifier-list> ( * <direct-declarator> ) ( int a ) #(7) applies
-> <declaration-specifier-list> ( *func ) ( int a ) #(8) is applied
-> void ( *func ) ( int a ) #(2), (4) apply
```
Here the stack looks like this from the top.
```
"pointer of"
"function ( a int ) returning"
```
This is taken from the top and arranged from left to right.

"pointer of function ( a int ) returning"


This is connected after the "func" from the syntax rule (8), and after that, the "void" from the rest of the declaration-specifier-list.
The result is as follows

"func pointer of function ( a int ) returning void"

Now that we have the English notation for the ```void(*func)(int a)``` part, let's discard the stack we used and go back to the original part.

```
<declaration>
-> <declaration-specifier-list> <declarator> #(1) applied
-> <declaration-specifier-list> <direct-declarator> #apply (7)
-> <declaration-specifier-list> <direct-declarator> ( <parameter-type-list> ) #(11) applies
-> <declaration-specifier-list> <direct-declarator> ( <declaration> ) #(13) applies
-> <declaration-specifier-list> <direct-declarator> ( int b ) #I was able to derive int b. Put "function ( b int ) returning" in the stack.
-> <declaration-specifier-list> ( <declarator> ) ( int b ) #apply (9)
-> <declaration-specifier-list> ( * <declarator> ) ( int b ) #(5) is applied Put "pointer of" in the stack.
-> <declaration-specifier-list> ( * <direct-declarator> ) ( int b ) #(7) applies
-> <declaration-specifier-list> ( * <direct-declarator> ( <parameter-type-list> ) ) ( int b ) #(11) applies.
-> <declaration-specifier-list> ( * <direct-declarator> ( <parameter-type-list> , <declaration> ) ) ( int b ) #(14) is applied
-> <declaration-specifier-list> ( * <direct-declarator> ( <parameter-type-list> , void(*func)(int a)) ) ( int b ) #we could derive void(*func)(int a)
-> <declaration-specifier-list> ( * <direct-declarator> ( <declaration>, void(*func)(int a)) ) ( int b ) #(13) applied
-> <declaration-specifier-list> ( * <direct-declarator> ( int sig, void(*func)(int a)) ) ( int b ) #I could derive int sig. Put "function ( sig int , func pointer of function ( a int ) returning void ) returning" in the stack.
-> <declaration-specifier-list> ( * signal ( int sig, void(*func)(int a)) ) ( int b ) #(8) is applied
-> void ( * signal ( int sig, void(*func)(int a)) ) ( int b ) #Apply (2), (4)
```

Here, the stack looks like this from top to bottom:
```
"function ( sig int , func pointer of function ( a int ) returning void ) returning"
"pointer of"
"function ( b int ) returning"
```

These are taken from the top and arranged from left to right.

"function ( sig int , func pointer of function ( a int ) returning void ) returning pointer of function ( b int ) returning"


This is connected after the "signal" from the syntax rule (8), and after that, the "void" from the rest of the declaration-specifier-list.
The result is as follows

"signal function ( sig int , func pointer of function ( a int ) returning void ) returning pointer of function ( b int ) returning void"Now you have read it correctly.# End of the article
Based on this article, I wrote a program in Python to convert C declarations into simple English notation. The source is [here](https://github.com/wo4mei3/decl_reader).

The usage is as follows.When you run main.py, you will be prompted as follows.
``` 
decl_reader $ python3 main.py
>
```
Enter any declaration, followed by a semicolon, and a simple English reading of it will be returned.However, structs, unions, enums, etc. are not supported. It throws an error.
```
> char *const ptr [4] ;

ptr array [ 4 ] of const pointer of char> char (*const ptr)[4];

ptr const pointer of array [ 4 ] of char

> bool (*(*p)[10])(int a, int b);

p pointer of array [ 10 ] of pointer of function ( a int , b int ) returning bool

> void (*signal (int sig, void(*func)(int a)))(int b);

signal function ( sig int , func pointer of function ( a int ) returning void ) returning pointer of function ( b int ) returning void
```

That concludes this article. Thank you for reading this long article.
I am looking forward to the day when you will adopt the method described in this article in your C language declarations decoding method. Exciting.

# References

B.W. Kernihan (Author), D.M. Ritchie (Author) (1988) Prentice Hall "Programming Language C 2nd Edition ANSI Standard Compliant" 

# See also

[https://learn.microsoft.com/en-us/cpp/c-language/interpreting-more-complex-declarators](https://learn.microsoft.com/en-us/cpp/c-language/interpreting-more-complex-declarators)
