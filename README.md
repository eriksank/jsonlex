# Jsonlex

## What is jsonlex?

Jsonlex is a text scanner (also called "lexer" or "tokenizer"). It does pretty much does what [flex](http://flex.sourceforge.net) does. Under the hood it uses the [PCRE](http://www.pcre.org) library.

In the process of parsing (structured) text, a text scanner will group letters into qualified words. Quite often, the purpose for doing this, is to supply a stream of qualified words to a parser, which will group words into sentences and subsentences. Example, the text:

```
if(a>0) then z=x+y
```

becomes after lexing:

|type|value
|---|---
| IF | if
| LEFT_BRACKET | (
| IDENTIFIER | a
| GREATER_THAN | >
| INTEGER | 0
| RIGHT_BRACKET | )
| THEN | then
| IDENTIFIER | z
| ASSIGN | =
| IDENTIFIER | x
| SUM | +
| IDENTIFIER | y


## Installation

The scripts are written in php. First install `php5-cli`:

	$ sudo apt-get install php5-cli

You can install jsonlex itself with the script `install.sh`:

	$ sudo ./install.sh

You can uninstall jsonlex with the script `uninstall.sh`:

	$ sudo ./uninstall.sh

## Example

	$ cd examples/example-jsonlex-1

	$ cat input.txt | jsonlex def=def.txt

### input.txt

```
if(a==b) {
	x=5;
} else {
	x=25;
}
```

### def.txt

```
#-----------------------------
# keywords
#-----------------------------

IF:if
ELSE:else

#-----------------------------
# punctuation:
#-----------------------------

LEFT_BRACKET:\(
RIGHT_BRACKET:\)
LEFT_CURLY_BRACKET:\{
RIGHT_CURLY_BRACKET:\}
SEMICOLON:;

#-----------------------------
# operators
#-----------------------------

EQUALS:==
ASSIGN:=

#-----------------------------
# the following are greedy/ungreedy unbounded strings
# better to put them at the end
#-----------------------------

NUMBER_INTEGER:[0-9]+
IDENTIFIER:[a-zA-Z_][a-zA-Z0-9_]*

#-----------------------------
# whitespace
#-----------------------------

WHITESPACE:\s+ 
```
A rule must be on one line, consist of a rulename, followed by a semicolon, followed by a regular expression. For the regular expressions, you can use all the constructs available in PCRE. Note that you need to escape PCRE regular expression syntax control characters as appropriate.

### output

By default jsonlex outputs the text analysis in textual format:

```
==TOKENS==
0 1 1 IF if
2 1 3 LEFT_BRACKET (
3 1 4 IDENTIFIER a
4 1 5 EQUALS ==
6 1 7 IDENTIFIER b
7 1 8 RIGHT_BRACKET )
8 1 9 WHITESPACE  
9 1 10 LEFT_CURLY_BRACKET {
10 1 11 WHITESPACE \n	
12 2 2 IDENTIFIER x
13 2 3 ASSIGN =
14 2 4 NUMBER_INTEGER 5
15 2 5 SEMICOLON ;
16 2 6 WHITESPACE \n
17 3 1 RIGHT_CURLY_BRACKET }
18 3 2 WHITESPACE  
19 3 3 ELSE else
23 3 7 WHITESPACE  
24 3 8 LEFT_CURLY_BRACKET {
25 3 9 WHITESPACE \n	
27 4 2 IDENTIFIER x
28 4 3 ASSIGN =
29 4 4 NUMBER_INTEGER 25
31 4 6 SEMICOLON ;
32 4 7 WHITESPACE \n
33 5 1 RIGHT_CURLY_BRACKET }
34 5 2 WHITESPACE \n\n
```

* Column 1: the absolute position in the text (as a string) starting from position 0
* Column 2: the line number, starting from 1
* Column 3: the column number, starting from 1
* Column 4: the token type matched
* Column 5: the actual value of the token matched

Note that:

* newlines are represented by: `\n`
* backslashes (\\) are represented by: `\s`

In order to unescape the embedded control characters, you will have to translate again:

* `"\n"` -> `\n`
* `\s` -> `\`

This simple text format has a common line discipline and is absolutely suitable for use with shell scripts.


### Output in json

	$ cat input.txt | jsonlex def=def.txt format=json

```
{
    "tokens": [
        {
            "name": "IF",
            "value": "if",
            "position": 0,
            "lineno": 1,
            "column": 1
        },
        {
            "name": "LEFT_BRACKET",
            "value": "(",
            "position": 2,
            "lineno": 1,
            "column": 3
        },
        {
            "name": "IDENTIFIER",
            "value": "a",
            "position": 3,
            "lineno": 1,
            "column": 4
        }, ...
```

The advantage of using the json format is that you can feed the output of jsonlex to another script of which the scripting engine supports json. It should be able to parse this output automatically.
 
## Gaps

Imagine that you add the following line at the bottom of the input:

```
	x=5;
if(a==b) {
} else {
	x=25;
}

+++
```

There is no rule to match the sequence "+++". Therefore, jsonlex will add a section before the ordinary output saying:

```
==GAPS==
36 7 1 _GAP_ +++
```

In the json output, jsonlex will add :

```
    "gaps": [
        {
            "name": "_GAP_",
            "value": "+++",
            "position": 36,
            "lineno": 7,
            "column": 1
        }
    ]
```

Of course, it is up to you to decide if the presence of gaps means that there is an error in the input. For programming languages this is always the case. The existence of unqualified tokens in the input will halt the compilation process there and then. In other text analysis situations, gaps are perfectly well acceptable.

## lex-extract

This is a small program that removes the non-programming code from program code typically embedded in html or a similar format. For example to remove the html out of a PHP program:

```
$ cat input.php | lex-extract start=<?php end=?>
```

All html will have been replaced by spaces.

## Why jsonlex?

Flex generates a function in C that you are supposed to use out of C program. Therefore, flex is tightly married to C. It would take you additional effort to create bindings to another language. Of all languages, C is obviously not the worst one to be married to, but still. With jsonlex, there is no need to write a C program. If all you want to do, is to prototype things, and you just need the the output based on input and rules without being dictating to what language you should be using, jsonlex is more suitable.

PCRE regular expressions are also much more powerful than the ones you can use in flex. You can correctly parse comments, strings, heredocs and other multiline syntax with PCRE regular expressions. You cannot do that with flex. You would need to painstakingly write additional functions to tackle the problem manually. By the way, in example 2, you can find PCRE regular expressions for parsing comments and strings.

A C program is supposed to be faster, but because the bulk of the work is done by PCRE, which is also a C library, even that may not necessarily be true. If PCRE systematically generates better internal (DFA) tables than flex, you could actually be better off with PCRE in terms of speed. Flex will precompile the tables to improve performance, but PCRE will cache them too. Therefore, that particular performance gain may not be spectacular either.

Jsonlex is a rapid prototyping lexing tool that allows you to do what would otherwise take quite a bit of effort with flex. 

## Copyright

Written by Erik Poupaert

February 2015

Licensed under the General Public License (GPL)

