# Higher ranked lifetimes

Higher ranked lifetimes allow specifying that all posisible lifetimes may be
used. This is useful when the exact lifetime isn't known or cannot be named,
such as inferred lifetimes in another function body.

## Higher ranked lifetimes in functions

Consider the following function

```rust
fn values(fun: fn(&i32) -> &i32, x: i32) -> (i32, &'static i32) {
    static ZERO = 0;
    (*fun(&x), fun(&X))
}
```

`fun` is called with both the `'static` lifetime and the short infered lifetime
of the reference to `x`. The ability of certain lifetime parameters of a type
to be chosen at each point the type is used is called *late binding*.

If we try to explicitly specify the lifetime like so then we can only call
`fun` with a single lifetime specified by the caller, so neither call is valid.

```rust,compile_fail,E0597
fn values<'a>(fun: fn(&'a i32) -> &'a i32, x: i32) -> (i32, &'static i32) {
    static ZERO = 0;
    (*fun(&x), fun(&X))
}
```

Late bound lifetimes can be written out using the `for<'a>` syntax. In this
case as

```rust
fn values(fun: for<'a> fn(&'a i32) -> &'a i32, x: i32) -> (i32, &'static i32) {
    static ZERO = 0;
    (*fun(&x), fun(&X))
}
```

In function pointers a late bound lifetime may not appear only in the return
type, so `for<'a> fn() -> &'a i32` is not a valid type.

[Function item types] can also have late-bound lifetimes. As function item types
cannot be named, which lifetime parameters are late bound is inferred. Lifetime
bounds are late-bound if

* It is contrained by a parameter type.
* It does not appear in a where clause.

A type constrains all of the lifetimes that appear in it apart from any inputs
to asociated type projections. In other words, `<&'a T as Trait<'b>>::Foo` does
not constrain `'a` or `'b`.

## Higher ranked lifetimes in traits

Higher ranked lifetimes can be used in both trait bounds and trait objects. For
example, if we wanted to instead use the `Fn` trait in the example above we can
write

```rust
# {
fn values<F>(fun: F) -> i32 where F: Fn(&i32) -> &i32 {
    static ZERO = 0;
    (*fun(&x), fun(&X))
}
# } {
// Expanded
fn values<F>(fun: F) -> i32 where F: for<'a> Fn(&'a i32) -> &'a i32 {
    static ZERO = 0;
    (*fun(&x), fun(&X))
}
# } {
// or
fn values<F>(fun: F) -> i32 where for<'a> F: Fn(&'a i32) -> &'a i32 {
    static ZERO = 0;
    (*fun(&x), fun(&X))
}
// Using a trait object
# } {
fn values(fun: for<'a> for Fn(&'a i32) -> &'a i32, x: i32) -> (i32, &'static i32) {
    static ZERO = 0;
    (*fun(&x), fun(&X))
}
// Expanding the trait object type:
# } {
fn values(fun: for<'a> for<'a> Fn(&'a i32) -> &'a i32, x: i32) -> (i32, &'static i32) {
    static ZERO = 0;
    (*fun(&x), fun(&X))
}
# }
```

A higher ranked lifetime must not only appear in associated types of a trait.
So `for<'a> Fn() -> &'a i32` and `for<'b> Foo<AType = Bar<'b>>` are not valid
types.

The higher ranked lifetime can also be used in a type in a where clause:

```rust
# trait SomeTrait {}
fn f<T>(x: T) where for<'a> &'a T: SomeTrait {}
```
