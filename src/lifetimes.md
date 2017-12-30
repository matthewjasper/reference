# Lifetimes

Every reference in Rust has a _lifetime_, which is the scope where it is valid.
Inside functions lifetimes are inferred, but in function signatures and in
types lifetimes need to be explicitly stated.

Unlike types, there is only one namable concrete lifetime (`'static`) so
lifetimes are almost always written in generic context. Also unlike types,
lifetimes don't affect code generation, they only allow the compiler to check
that a program is memory safe.

## Syntax

Lifetimes are written as a `'` followed by an [identifier] or the `static`
keyoword.

References with lifetimes are written as `&'a T` or `&'a mut T`. Types with
lifetime parameters are written like `Ref<'a, T>`, with the lifetime parameters
always listed before any type parameters.


## `'static`

There is one concrete lifetimes that can be named in Rust: `'static`. It is the
lifetime of anything that lives until the end of the programme. References to
[`static` items], references in [`const` items], string literals and [promoted
statics] are examples of references with this lifetime.



## Lifetimes in user defined types

A type can have lifetime parameters that can be used as lifetime parameters
of the types of it's fields. The lifetime must be used in the type of at least one of
the struct fields to determine the [variance]

```rust
struct Str<'a> {
    s: &'a str
}
```

## Lifetimes in functions

Lifetimes can be introduced in function signatures to relate the return type
and the types of the arguments. For example:

```rust
fn first<'a>(s: &'a [u8]) -> &'a u8 {
    &s[0]
}

fn zero<'a>(s: &'a [u8]) -> &'static u8 {
    static ZERO: u8 = 0;
    &ZERO
}
```

The lifetimes in `first` could have been [elided].

## Lifetime inference

In function bodies lifetimes are  usually infered. Infered lifetimes do not
have any syntax. Lifetimes are inferred when no explicit lifetime is given
and the lifetime elision rules don't determine the lifetime.

```rust
let x = 120;
let y = &x;                 // y is infered to be `&i32`, with the liftime
                            // parameter also infered
let z: &i32 = &x;           // Can omit the lifetime parameter in &i32
                            // becomes an infered lifetime
let f: fn(&i32) = |x| {}    // Elision, expands to `for<'a> fn(&'a i32)`
```

[`const` items]: items.html#constant-items
[elided]: lifetime-elision.html
[function items]: items.html#function-items
[identifier]: identifiers.html
[promoted statics]: expressions.html#temporary-lifetimes
[variance]: subtyping.md#variance
[`static` items]: items.html#constant-items
