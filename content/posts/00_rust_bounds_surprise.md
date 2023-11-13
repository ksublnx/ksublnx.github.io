+++
title = 'Rust Surprise #1: Compile-Time Array Bounds Checking Works Only in a Limited Context'
date = 2023-11-11T14:50:04+05:30
type = "post"
tags = ["rust-lang"]
+++

The following code compiles unmolested, and panics at runtime. The out-of-bounds
accesses in lines 11 and 15 are not flagged at compile time.

```rust {linenos=true, linenostart=1, style=autumn}
fn main () -> ()
{
    let xs: [i32; 5] = [1, 2, 3, 4, 5];

    for x in xs {
        print!("{} ", x);
    }
    println!("");

    for i in 0..6 {
        println!("xs[{}] = {}", i, xs[i]);
    }

    let idx2: usize = 5;
    println!("xs[{}] = {}", idx2, xs[idx2]);

    // This code does not compile
    //println!("xs[{}] = {}", 5, xs[5]);
}
```

The panic message is very specific and helpful, however:

````
index out of bounds: the len is 5 but the index is 5
````

## Do bounds checks work the same way with other collections, say `Vec<T>`?
Let's find out.

```rust {linenos=true, linenostart=1, style=autumn}
use std::ops::Range;

fn main() -> ()
{
    let mut vec: Vec<i32> = Vec::new();
    const VRANGE: Range<i32> = 1..6;
    const IRANGE_OOB: Range<usize> = 0..7;

    for x in VRANGE {
        vec.push(x);
    }

    for i in IRANGE_OOB {
        print!("{} ", vec[i]);
    }
    println!("");
}
```

... and sure enough, we panic at runtime with the same error message:

```
index out of bounds: the len is 5 but the index is 5
```

Slices work in the same way too.

## Why was I surprised?
I was under the impression that _all_ array accesses are checked against bounds
at compile time in Rust. This does not seem to be the case.

## Questions
Hopefully, I should be able to find answers to the questions below, and detail
them in a future post:

1. Is the helpful panic message just a simple print of the array length and the
   index, or is there something more complicated going on at runtime?
2. How does the Rust compiler implement bounds checks? 
3. How difficult would it be to implement bounds checks that catch the
   out-of-bounds accesses above at compile time?

## Related Reading
1. [The Rust Performance Book](
    https://nnethercote.github.io/perf-book/bounds-checks.html
)
2. [The Bounds Check Cookbook](
    https://shnatsel.medium.com/how-to-avoid-bounds-checks-in-rust-without-unsafe-f65e618b4c1e
)
