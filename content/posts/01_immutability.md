+++
title = 'Rust Surprise #2: Subverting Immutability via Raw Pointers'
date = 2023-11-11T17:46:18+05:30
type = "post"
tags = ["rust-lang"]
+++

`*const` pointers can be typecast into `*mut` pointers.  The resulting `*mut`
pointer can be used to modify the memory it points to, _regardless of whether
the original variable was qualified with a `mut`, or not._

However:

 (1) A `*const` cannot be used to modify memory directly; it _must_ be cast into
     a `*mut` to do so.

 (2) It is not possible to derive a `*mut` from (a reference to) an immutable
     variable. A relevant `*const` has to be typecasted into a `*mut` in order to
     subvert immutability.

 (3) The mutation can _only_ happen within an `unsafe` block.

```rust {linenos=true, linenostart=1, style=autumn}
let x: i32 = 42;
let px: *const i32 = &x as *const i32;

println!("x = {}", *px);
unsafe {
    // Thankfully, the code below does not compile.
    *px = 44;
}
println!("x = {}", *px);

// This too is illegal, and will break compilation
let mpx: *mut i32 = &mut x as *mut i32;

//
// ... And finally, we subvert immutability for fun without profit :)
//
let mpx: *mut i32 = px as *mut i32;
unsafe {
    *mpx = 43;
}
println!("*px = {}", *px);

println!("x = {}", x);
```

### Why was I surprised?
The [Rust Book](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#dereferencing-a-raw-pointer)
does mention that raw pointers are allowed to ignore the borrowing rules by
having both immutable and mutable pointers to the same (memory) location.

However, the fact that a `*mut` pointing to an immutable variable (arrived at by
suitable subterfuge as detailed in the code sample above) can modify said
immutable variable, came as a surprise.

### Questions
Too many. At this time, I need to dig around some more in order to arrive at
properly-framed questions.

### Related Reading
1. [The Rust Book on Dereferencing Raw Pointers](
    https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#dereferencing-a-raw-pointer
)
