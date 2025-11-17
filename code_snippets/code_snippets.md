# Scopes and mutability

Fails to compile:

```rust
fn main() {               // ---------+-- A
    let x = 20;           //          |
                          //          |
    {                     // -+-- B   |
        println!("x: {x}");//         |
        let y = 5;        //  |       |
    }                     // -+       |
                          //          |
    println!("y: {y}");   //          |
}                         // ---------+
```

```
error[E0425]: cannot find value `y` in this scope
 --> .\test_code.rs:9:19
  |
9 |     println!("y: {y}");   //          |
  |                   ^ help: a local variable with a similar name exists: `x`

error: aborting due to 1 previous error
```

<hr>Compiles with warning, acts unexpectedly.

```rust
fn main() {               // ---------+-- A
    let x = 20;           //          |
    let y = 2048;         //          |
                          //          |
    {                     // -+-- B   |
        let y = 5;        //  |       |
    }                     // -+       |
                          //          |
    println!("x: {x}");   //          |
    println!("y: {y}");   //          |
}                         // ---------+
```

```
warning: unused variable: `y`
 --> .\test_code.rs:6:13
  |
6 |         let y = 5;        //  |       |
  |             ^ help: if this is intentional, prefix it with an underscore: `_y`
  |
  = note: `#[warn(unused_variables)]` on by default

warning: 1 warning emitted
```

```
x: 20
y: 2048
```

<hr>Fails to compile.

```rust
fn main() {               // ---------+-- A
    let x = 20;           //          |
    let y = 2048;         //          |
                          //          |
    {                     // -+-- B   |
        y = 5;            //  |       |
    }                     // -+       |
                          //          |
    println!("x: {x}");   //          |
    println!("y: {y}");   //          |
}                         // ---------+
```

```
warning: value assigned to `y` is never read
 --> .\test_code.rs:3:9
  |
3 |     let y = 2048;         //          |
  |         ^
  |
  = help: maybe it is overwritten before being read?
  = note: `#[warn(unused_assignments)]` on by default

error[E0384]: cannot assign twice to immutable variable `y`
 --> .\test_code.rs:6:10
  |
3 |     let y = 2048;         //          |
  |         - first assignment to `y`
...
6 |          y = 5;        //  |       |
  |          ^^^^^ cannot assign twice to immutable variable
  |
help: consider making this binding mutable
  |
3 |     let mut y = 2048;         //          |
  |         +++

error: aborting due to 1 previous error; 1 warning emitted
```

```
x: 20
y: 2048
```

<hr>Compiles successfully.

```rust
fn main() {               // ---------+-- A
    let x = 20;           //          |
    let mut y = 2048;     //          |
                          //          |
    {                     // -+-- B   |
        y = 5;            //  |       |
    }                     // -+       |
                          //          |
    println!("x: {x}");   //          |
    println!("y: {y}");   //          |
}                         // ---------+
```

```
x: 20
y: 5
```

# Types and inference

Compiles successfully.

```rust
fn main() {
    let a: usize = 98_168_416_450;
    let b: i64 = -2025;
    let c: f32 = 3.14;
    let d = true;
    let e = 20;
    let f = 2.7;
}
```

# References

Fails to compile:

```rust
fn main() {               // ---------+-- A
    let ref_x;            //          |
                          //          |
    {                     // -+-- B   |
        ref_x = &x;       //  |       |
        let x = 5;        //  |       |
    }                     // -+       |
                          //          |
    println!("r: {ref_x}"); //        |
}                         // ---------+
```

```
error[E0425]: cannot find value `x` in this scope
 --> .\test_code.rs:5:18
  |
5 |         ref_x = &x;       //  |       |
  |                  ^ not found in this scope
```

<hr>Fails to compile:

```rust
fn main() {               // ---------+-- A
    let ref_x;            //          |
                          //          |
    {                     // -+-- B   |
        let x = 5;        //  |       |
        ref_x = &x;       //  |       |
    }                     // -+       |
                          //          |
    println!("r: {ref_x}"); //        |
}                         // ---------+
```

```
error[E0597]: `x` does not live long enough
 --> .\test_code.rs:6:17
  |
5 |         let x = 5;        //  |       |
  |             - binding `x` declared here
6 |         ref_x = &x;       //  |       |
  |                 ^^ borrowed value does not live long enough
7 |     }                     // -+       |
8 |                           //          |
9 |     println!("r: {ref_x}"); //        |
  |                   ----- borrow later used here
```

<hr>Fails to compile:

```rust
fn main() {               // ---------+-- A
    let ref_x;            //          |
    let x;                //          |
                          //          |
    {                     // -+-- B   |
        ref_x = &x;       //  |       |
        x = 5;            //  |       |
    }                     // -+       |
                          //          |
    println!("r: {ref_x}"); //        |
}                         // ---------+
```

```
error[E0381]: used binding `x` is possibly-uninitialized  
 --> .\test_code.rs:6:17
  |
3 |     let x;                //          |
  |         - binding declared here but left uninitialized
...
6 |         ref_x = &x;       //  |       |
  |                 ^^ `x` used here but it is possibly-uninitialized

error[E0506]: cannot assign to `x` because it is borrowed
  --> .\test_code.rs:7:9
   |
 6 |         ref_x = &x;       //  |       |
   |                 -- `x` is borrowed here
 7 |         x = 5;            //  |       |
   |         ^^^^^ `x` is assigned to here but it was already borrowed
...
10 |     println!("r: {ref_x}"); //        |
   |                   ----- borrow later used here
```

<hr>Compiles successfully:

```rust
fn main() {               // ---------+-- A
    let ref_x;            //          |
    let x;                //          |
                          //          |
    {                     // -+-- B   |
        x = 5;            //  |       |
        ref_x = &x;       //  |       |
    }                     // -+       |
                          //          |
    println!("r: {ref_x}"); //        |
}                         // ---------+
```