# Types and Purity

This document aims to lay out a general idea on how the closely interlinked concepts
of types, mutability and function purity should work in hecate.

FUrthermore, in this document there is no distinction between functions and methods, the latter being treated as syntactic sugar for the former.

> [!NOTE]
> Examples in this document will be written n a rust-like syntax but either endorse nor condemn any particular style.

## Strict defaults
An important concept of functions is their pureness (or "constness").
Along with the mutability of types this causes a sort of one-way coercion:
- a mutable value can always be treated immutably
- a pure function can always be treated as one with side effects
A similar point can be made for visibility:
- a public type can always be treated as if it was invisible (aka by not using it)

Strictness (in the sense of purity and mutablilty) allow you to do less thigns with your code but also oftentimes
give valuable promises to the compiler (increasing fucntionality) and the programmer (increasing clarity).

These points lead to a logical conclusion: \
Each of these concepts should have their strictest variant applied by default.
This makes you think about which restrictions _specifically_ need to be relaxed,
at the cost of loosing functionality. Additionally, the compiler throws you an error
if one of your restrictions is not applicable (for example when trying to mutate an immutable variable),
but it does not necessarily when you forget to supply an optional but possible more restriccitve keyword,
like package-private as a default in java.

This has already been done in rust with publicity and mutablity of variables, but not for example with constant functions.
This results in some functions bot being able to be used in cosnt context even though they would work just fine.

## Functions
Functions can have the following levels of "purity", or "outwards interaction".
The level promised is valid when any function called in the body has at least that high of a promise level.
Failing to do so will result in a compile time error.

Levels:
- "true" const
    - ❗ may not have mutable references as params
    - can be evaluated at compile time when called with const parameters
    - can be cached since they have no outwards interaction
    - deliver the highest possible promise of reliabilty possible, can not fail due to any external circumstances except for bad parameters
- pure, "almost const" a.k.a. const with heap allocations
    - ❗ not cleanly defined: the heap uses allocation which technically is an outwards interaction, even though the side effects are - apart from oout of memory - "nonexistent"
    - ❗ almost the same promises as const:
        - ❗ no compile time evaluation
        - ❗ can only be cached when type implements a true deepcopy
    - has higher versatility due to allowing data types with pointers
- impure
    - may have mutable states and side effects
    - runs the same regardless of hardware or execution context
- io
    - can dos tuff like file or socket interaction, read env vars or system time
    - ❗ programmer can no linger fully guarantee validity of the code in every context, a famous example being the [Year 2000 problem](https://en.wikipedia.org/wiki/Year_2000_problem)

Drawbacks:
- this system has to - at the low level - make some naiive assumptions, resulting in some magically "overleveled" functions
- const is hard to debug since it even lacks "print" (could be solved via magic debug macro/function which is not allowed in release mode (or a similar flag))
- downgrading a function means all the call sites would ahve to be downgraded aswell. on the positive side, that course of action would probably be bad code design either way
- generic interfaces or when passing functions as values you usually would want to be as loosely bound as possible (resulting in "io" being the default). This could be fixed with "generic over constness"
```rs
// pseudo-rust
<P> fn map<purity P, Item, Mapped>(func: <P> fn(Item) -> Mapped, iterable: impl Iter<Item>) -> impl Iter<Value> where P >= pure { /* ... */ }

let _ = map<pure, u32, u32>(|x| x+1, vec[1, 2, 3]);
let _ = map<io, String, Result<(), _>>(|f| println!(readlines(f)?), vec["foo.txt", "bar.txt"]);
```
## Types
> [!NOTE]
> Not to be coonfused with mutavilty of _values_ like in rust. SImilar concept but not discussed here

Rust has a way of declaring mutability for references `&T` vs `&mut T`, but not for struct fields or for owned types in a nested way.
- immutable struct fields would only be assignable via struct literal syntax
- a field is mutable if the parent scope is mutable and it has been declared as mutable there


Drawbacks:
- verbosity when declaring leveled mutabilty
- might need "generic over mutabilty" like the function constness

```rs
struct Foo {
    x: i32, // immutable after assignment
    a: mut bool, // mutable if outer type is mutable
    b: mut bool, // mutable if outer type is mutable
}

struct Bar<T> {
    counter: mut usize,
    t: mut T,
    t2: T
}

// x1.counter, x1.t.a, x1.t.b, x2.t.a and x2.t.b are mutable
type X1 = mut Bar<mut Foo>;
// x1.counter, x1.t.a, x1.t.b are mutable
type X1 = mut Bar<Foo>;
// nothing is mutable, compiler warning for "unneccessary mutablitly qualifier"
type X1 = Bar<mut Foo>;
// nothing is mutable
type X1 = Bar<Foo>;

// mutability generic concept
struct MaybeMut<stasis M, T> {
    field: <M> T
}

type SurelyMut<T> = mut MaybeMut<mut, T>;
type NotMut<T> = mut MaybeMut<mut, T>;
```