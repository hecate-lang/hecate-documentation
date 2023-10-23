# Hecate-v0.0.1 <!-- omit in toc -->

This is the first specification of a subset of the Hecate programming language. It includes functions, numeric primitive types and basic statements / expressions. Please note that this specification is **highly experimental and subject to change**. 

## Table of Contents <!-- omit in toc --> 
- [Primitive Types](#primitive-types)
  - [Integer Types](#integer-types)
  - [Floating-point Types](#floating-point-types)
  - [Boolean Type](#boolean-type)
- [Functions](#functions)
  - [return Keyword](#return-keyword)
  - [Entry Function](#entry-function)
- [Statements](#statements)
  - [Let Bindings](#let-bindings)
    - [Name Shadowing](#name-shadowing)
  - [Expressions as statements](#expressions-as-statements)
- [Expressions](#expressions)
  - [Binary Operators](#binary-operators)
  - [Unary Operators](#unary-operators)
  - [Type Casts](#type-casts)
  - [Operator Precedence](#operator-precedence)
  - [Function Calls](#function-calls)
  - [Scope Expressions](#scope-expressions)
  - [If Expression](#if-expression)
- [Unit Type](#unit-type)
- [Never Type](#never-type)
- [Comments](#comments)
- [List of Keywords / Reserved Types](#list-of-keywords--reserved-types)
- [Formal Grammar of Hecate](#formal-grammar-of-hecate)

## Primitive Types

This section describes the various primitive types of Hecate. Primitive types are copy by default. 

### Integer Types
Hecate supports a whole range of different signed and unsigned integer types. These are the signed types `i8, i16, i32, i64, i128` and the unsigned types `u8, u16, u32, u64, u128`. The number of a type denotes its bit size. Supported integer formats are decimal, hexadecimal, binary and octal.

### Floating-point Types
Like in most programming languages, two differently sized floating-point numbers are available, namely `f32` and `f64`. Both decimal and fractional part are currently required. The scientific notation is also supported.

### Boolean Type
Booleans are named `bool` in Hecate. 

## Functions

```
// a function mapping an i32 to an i32
fun foo(bar: i32) -> i32 {
    bar * bar
}

// a function without arguments and return type
fun foobar() {
}
```

Functions are defined using the `fun` keyword followed by the function name, function args (put in parenthesis), return type and finally the function body. If no return type is desired, it can simply be omitted. Note that a type must be specified for arguments. In case the function body consists of a single expression only, one can use an alternative formatting:

```
fun foo(bar: i32) -> i32 = bar * bar;
```

Please note that Hecate does not support function overloading.
Function bodies are considered scope expressions. For more detailed information on them refer to the [corresponding section](#scope-expressions)

### return Keyword

To return early form a function use `return`:

```
fun foo(bar: i32) -> i32 {
    if bar < 0 {
        return 0;
    }
    bar
}

fun foobar(bar: i32) {
    if bar == 0 {
        return;
    }
    print(bar);
}
```

### Entry Function
Every Hecate program needs an `entry` function as an entry point of execution. This function does not take any arguments and does not 
return anything:

```
fun entry() {
  // Code execution starts here
  print("Hello World!");
}
```

## Statements

### Let Bindings

You can use let bindings to define a variable. Note that all variables are immutable by default. A type annotation is currently required.

```
let an_int: i32 = -1;
let big_uint: u128 = 100;
let some_bool: bool = true;
```

#### Name Shadowing
Name shadowing is supported, therefore the following code segment is considered valid Hecate syntax:
```
let a_var: i32 = 10;
let a_var: bool = true;
```

### Expressions as statements
Adding a semicolon to the end of an expression turns it into a statement which discards its resulting value. A block expression, as can be found with an if, loop or simply a scope block functions automatically as a statement and may not have a semicolon. Please note that assigning the value of a block to a variable does require a semicolon, which belongs to the assignment statement and not to the block.

## Expressions

### Binary Operators
Currently the following binary operators are supported: `+, -, *, /, %, <, >, <=, >=, ==, !=, &&, ||`.

Compatibility with types:
|         Type         | Sum(`+`/`-`) | Product(`*`/`/`/`%`) | Ordering(`>`/`<`/`>=`/`<=`) | Equation(`==`/`!=`) | Conjunction(`&&`) | Disjunction(`\|\|`) |
| :------------------: | :----------: | :------------------: | :-------------------------: | :-----------------: | :---------------: | :-----------------: |
|    Integer Types     |      ✓       |          ✓           |              ✓              |          ✓          |         ✗         |          ✗          |
| Floating-point Types |      ✓       |          ✓           |              ✓              |          ✓          |         ✗         |          ✗          |
|       Boolean        |      ✗       |          ✗           |              ✓              |          ✓          |         ✓         |          ✓          |

### Unary Operators
Currently the following unary operators are supported:  
- `-` or `+`:  negative or positive sign of a number respectively 
- `!`:            negation of a boolean value

Compatibility with types:
|         Type         | Sign(`+`/`-`) | Negation(`!`) |
| :------------------: | :-----------: | :-----------: |
|    Integer Types     |       ✓       |       ✗       |
| Floating-point Types |       ✓       |       ✗       |
|       Boolean        |       ✗       |       ✓       |

### Type Casts
Hecate doesn't allow implicit type conversions. Therefore you have to explicitly specify which type you want to convert to. This is simply done by using the type-cast operator(:) to supply a type annotation for a value:
```
let a: i32 = 10 + (0.5: i32); // casts from f32 / f64 to i32
```


### Operator Precedence

Operators with a higher precedence are applied before operators with a lower precedence.

|      Operators       | Precedence |
| :------------------: | :--------: |
|         \|\|         |     0      |
|          &&          |     1      |
|        ==, !=        |     2      |
|     <, >, <=, >=     |     3      |
|         +,-          |     4      |
|       *, /, %        |     5      |
|    unary +, -, !     |     6      |
| type-cast operator : |     7      |

### Function Calls
A function can be called by entering the function name followed by a comma seperated list of argument expressions enclosed by parenthesis:
```
fun foo(first: i32, second: i32) -> i32 {
  first + second
}

fun entry() {
  let a: i32 = foo(5, 5); // binds the value 10 to a
}
```

The only exception is the `entry` function. As this represents the entry point of the program, it can not be called inside the program itself.

### Scope Expressions
Scope expressions are defined as a sequence of statements embraced by curly parenthesis. Every return value of a scope expression must have the same type. One can also optionally supply an expression at the end of a scope expression which serves as an implicit return statement. 

```
{
  let a = 10;
  let b = 100;
  return a + b;
}

{
  let a = 10;
  a // implicit return
}
```

Scope expressions that don't return anything return the unit type by default.

### If Expression
To create an if expression, enter the `if` keyword followed by the condition and the scoped block that is executed when the condition succeeds:

```
// if expression returning the unit type ()
if a == 0 {
  print("zero");
}
```

You can also use `else` in combination with more if expressions or scoped blocks in order to choose the branch depending on multiple conditions:

```
// if expression returning an integer
if a == 0 {
  0
} else if a > 0 {
  1
} else {
  -1
}
```

All branches of the if expression are required to produce the same type. Therefore, the following code is not allowed:

```
// mismatched types between integer and bool
if a == 0 {
  10
} else {
  true
}
```

If expressions that do not have an else case implicitly return the unit type. This implies that all branches of the if expression have to return the unit type too.

## Unit Type
The unit type is returned by expressions and functions that do not return anything. It is denoted as the empty tuple `()`. The unit can be used in the same way as any other type.

## Never Type
The never type can never be constructed. It is mainly used for compile-time type checks and promises that code is unreachable. The never type is represented by `!`. Even though currently not in use, it is reserved for future versions.

## Comments
Hecate uses C-style comments like so:
```
// This is a single line comment

/*
    This is a comment
    over multiple lines
*/
``` 

## List of Keywords / Reserved Types
```
i8, i16, i32, i64, i128, u8, u16, u32, u64, u128, f32, f64, bool, fun, if, else, return, let
```

## Formal Grammar of Hecate
This sections aims to provide a correct formal grammar for Hecate. Symbols can be seperated by an abitrary amount of whitespaces.

```
program = program function_definition | ε;

function_definition = "fun" identifier "(" argument_signatures ")" scope_expression;
function_definition = "fun" identifier "(" argument_signatures ")" "->" type scope_expression;
function_definition = "fun" identifier "(" argument_signatures ")" "->" type "=" expression ";";
function_definition = "fun" identifier "(" argument_signatures ")" "=" expression ";";
function_definition = "fun" identifier "(" ")" scope_expression;
function_definition = "fun" identifier "(" ")" "->" type scope_expression;
function_definition = "fun" identifier "(" ")" "->" type "=" expression ";";
function_definition = "fun" identifier "(" ")" "=" expression ";";

argument_signatures = argument_signatures "," argument_signature | argument_signature;
argument_signature = identifier ":" type;
type = "i8" | "i16" | "i32" | "i64" | "i128" | "u8" | "u16" | "u32" | "u64" | "u128" | "bool" | "()" | "!" | "f32" | "f64";

expression = disjunction;
disjunction = disjunction "||" conjunction | conjunction;
conjunction = conjunction "&&" equation | equation;
equation = equation "==" ordering | equation "!=" ordering | ordering;
ordering = ordering "<" product | ordering "<=" product | ordering ">" product | ordering ">=" product | product;
product = product "*" sum | product "/" sum | product "%" sum | sum;
sum = sum "+" unary | sum "-" unary | unary;
unary = "!" unary | "-" unary | "+" unary | cast_expr;
cast_expr = value ":" type;
value = identifier | "(" expression ")" | scope_expression | if_expression | typed_integer | boolean | scientific_rational | function_call | "()";

scope_expression  = "{" statements "}" | "{" statements expression "}";
if_expression = "if" expression scope_expression | "if" expression scope_expression "else" if_expression | "if" expression scope_expression "else" scope_expression; 
function_call = identifier "(" expression_sequence ")" | identifier "(" ")";
expression_sequence = expression_sequence "," expression | expression;

statments = if_expression | scope_expression | expression ";" | let_binding;
let_binding = "let" identifier ":" type "=" expression ";";

typed_integer = integer | integer type;
integer = "0b" bin_integer | "0o" oct_integer | "0d" decimal_integer | decimal_integer | "ox" hex_integer;
bin_integer = bin_integer bin_digit | bin_digit;
oct_integer = oct_integer oct_digit | oct_digit;
dec_integer = dec_integer dec_digit | dec_digit;
hex_integer = hex_integer hex_digit | hex_digit;
bin_digit = "0" | "1";
oct_digit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7";
dec_digit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9";
hex_digit = dec_digit | "A" | "B" | "C" | "D" | "E" | "F" | "a" | "b" | "c" | "d" | "e" | "f";

boolean = "true" | "false";
scientific_rational = rational | rational "e" integer | rational "e" "-" integer;
rational = integer "." integer;

identifier = alpha_underscore alpha_numeric_seq;
alpha_numeric_seq = alpha_numeric_seq alpha_numeric | ε;
alpha_numeric = alpha_underscore | dec_digit;
alpha_underscore = alpha | "_";
alpha = "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j" | "k" | "l" | "m" | "n" | "o" | "p" | "q" | "r" | "s" | "t" | "u" | "v" | "w" | "x" | "y" | "z" | "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J" | "K" | "L" | "M" | "N" | "O" | "P" | "Q" | "R" | "S" | "T" | "U" | "V" | "W" | "X" | "Y" | "Z";
```