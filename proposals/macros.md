# Macros
Macros provide several benefits:
- they allow auto generated boilerplate code
- they provide a clear hint that the compiler is doing something out of the ordinary (see `line!()` or `column!()` rust macro)
- allow for added features without wasting more keywords or complicating the parsing/ast if the feature is situational (a line or column function makes no sense and a keyword would be _too_ extra)
=> even if those only act as a compiler plugin and are not user-definable in hecate/as a macro_rules-like feature they provide many benefits

## Syntax
### Macros applied to the next item
```rs
[<macro_ident>:<any_tokens>]
fun foo() = ();

[<macro_ident>(<any_tokens>)]
fun foo() = ();

#[<macro_ident>(<any_tokens>)]
fun foo() = ();

![<macro_ident>(<any_tokens>)]
fun foo() = ();

!<macro_ident>(<any_tokens>)
fun foo() = ();

#<macro_ident>(<any_tokens>)
fun foo() = ();

$<macro_ident>(<any_tokens>)
fun foo() = ();

<macro_ident>$(<any_tokens>)
fun foo() = ();

<macro_ident>!(<any_tokens>)
fun foo() = ();

// TODO: should we allow {} and [] same as with expression macros?
```
### Expression/statement macros
```rs
<macro_ident>!(<any_tokens>);
#<macro_ident>(<any_tokens>);
$<macro_ident>(<any_tokens>);
<macro_ident>$(<any_tokens>);
#<macro_ident>[<any_tokens>];
#<macro_ident>{<any_tokens>};
```

## Difference to rust
Other than like rust, `{}`, `[]` and `()` should not be usable interchagably. 
Hygene/passing down identifiers to bind is important (same as rust) since we have name shadowing.
We should however consider adding the functionality to create new identifiers (`concat_ident!()` in rust may not be used in a declaring context).
Different from rust, a macro should be any token until the matching closing paren is reached.
### TODO
- allow `<>` as paren pair?
- sanitize block comments or not?

## Implementation
Macros are compile time evaluated and modify the ast. Most likely this will be done very early on after the very first ast generation and before any checking has taken place.
After the ast has been modified, the macro is removed and the ast contnues down the compilation path.

Implementation can be done in several ways
#### loading of compiler plugins from hecate/rust by precompiling a module and dynamically loading it
##### pro: 
- flexible, full accss to ast
- if codable in hecate it can be done "natively"
##### con:
- high entry barrier
#### marco_rules like feature
##### pro:
- fully accessible
- can even be coded in compiler plugin, potentially no extra work needed!
- if done natively, tokens could be traced back to input allowing "free" linting/syntax highlights
##### con:
- custom syntax and potentially clunky behavior (see rust)
