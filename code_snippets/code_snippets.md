# Scopes, shadowing and mutability

Fails to compile:

```rust
fn main() {               // ---------+-- A
    let x = 20;           //          |
                          //          |
    {                     // -+-- B   |
        let y = 5;        //  |       |
        println!("x: {x}");// |       |
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

<hr>Compiles successfully.

```rust
fn main() {               // ---------+-- A
    let x = 20;           //          |
                          //          |
    {                     // -+-- B   |
        let y = 5;        //  |       |
        println!("y: {y}");// |       |
    }                     // -+       |
                          //          |
    println!("x: {x}");   //          |
}                         // ---------+
```

```
y: 5
x: 20
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

These fail to compile because of overflowing literal

```rust
fn main() {
    let a = -2147483649;
}
```

```rust
fn main() {
    let a: u8 = 1000;
}
```

These compile successfully

```rust
fn main() {
    let a: i64 = -2147483649;
}
```

```rust
fn main() {
    let a = -2147483649_i64;
}
```

# Initialisation

Compiles:

```rust
fn main() {
    let a;
    let b: bool;
    let mut c: u8;

    a = 20;
    b = false;
    c = 15;

    c = c * 2;

    println!("a: {a}");
    println!("b: {b}");
    println!("c: {c}");
}
```

```
a: 20
b: false
c: 30
```

<br>Fails to compile:

```rust
fn main() {
    let a;
    println!("a: {a}");

    a = 20;
    println!("a: {a}");
}
```

```
error[E0381]: used binding `a` is possibly-uninitialized
 --> .\test_code.rs:3:19
  |
2 |     let a;
  |         - binding declared here but left uninitialized
3 |     println!("a: {a}");
  |                   ^ `a` used here but it is possibly-uninitialized
  |
  = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)
```

# Basics of references

Fails to compile:

```rust
fn len_by_value(param: String) -> usize { // ---+-- B
    param.len() //                              |
} // -------------------------------------------+

fn main() { // --------------------------+-- A
    let x = String::from("123"); //      |
//                                       |
    let lenght = len_by_value(x); //     |
    println!("x' lenght: {}", lenght); //|
    println!("x: {}", x); //             |
} // ------------------------------------+
```

```
error[E0382]: borrow of moved value: `x`
  --> .\test_code.rs:10:23
   |
 6 |     let x = String::from("123"); //      |
   |         - move occurs because `x` has type `String`, which does not implement the `Copy` trait
 7 | //                                       |
 8 |     let lenght = len_by_value(x); //     |
   |                               - value moved here
 9 |     println!("x' lenght: {}", lenght); //|
10 |     println!("x: {}", x); //             |
   |                       ^ value borrowed here after move
   |
note: consider changing this parameter type in function `len_by_value` to borrow instead if owning the value isn't necessary
  --> .\test_code.rs:1:24
   |
 1 | fn len_by_value(param: String) -> usize { // ---+-- B
   |    ------------        ^^^^^^ this parameter takes ownership of the value
   |    |
   |    in this function
   = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)
help: consider cloning the value if the performance cost is acceptable
   |
 8 |     let lenght = len_by_value(x.clone()); //     |
   |                                ++++++++
```

<hr>Compiles successfully:

```rust
fn len_by_ref(param: &String) -> usize { // --+-- B
    param.len() //                            |
} // -----------------------------------------+

fn mod_by_ref(param: &mut String) { //---+-- C
    param.push('4'); //                  |
} // ------------------------------------+

fn main() { //---------------------------+-- A
    let mut x = String::from("123"); //  |
//                                       |
    mod_by_ref(&mut x); //               |
//                                       |
    let lenght = len_by_ref(&x); //      |
    println!("x' lenght: {}", lenght); //|
    println!("x: {}", x); //             |
} // ------------------------------------+
```

```
x' lenght: 4
x: 1234
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
  |     - `x` dropped here while still borrowed
8 |                           //          |
9 |     println!("r: {ref_x}"); //        |
  |                   ----- borrow later used here

error: aborting due to 1 previous error

For more information about this error, try `rustc --explain E0597`.
```

### TODO: Some better example of exclusion of & and &mut at the same time.

<hr>Fails to compile:

```rust
struct Structure{
    ptr: &u32
}

fn main() {}
```

```
error[E0106]: missing lifetime specifier
 --> .\test_code.rs:2:10
  |
2 |     ptr: &u32
  |          ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 ~ struct Structure<'a>{
2 ~     ptr: &'a u32
  |
```

<hr>Fails to compile:

```rust
struct PtrCapsule<'a>{
    ptr: &'a u32
}

impl<'a> PtrCapsule<'a>{
    fn print(&self){
        println!("Value: {}", self.ptr);
    }
}

fn inc(num: &mut u32){
    *num += 1
}

fn main() {
    let mut x = 20;
    let capsule = PtrCapsule{ ptr: &x };

    println!("x: {}", x);

    inc(&mut x);
    capsule.print();
}
```

```
error[E0502]: cannot borrow `x` as mutable because it is also borrowed as immutable
  --> .\test_code.rs:21:9
   |
17 |     let capsule = PtrCapsule{ ptr: &x };
   |                                    -- immutable borrow occurs here
...
21 |     inc(&mut x);
   |         ^^^^^^ mutable borrow occurs here
22 |     capsule.print();
   |     ------- immutable borrow later used here
```

<hr>Compiles successfully:

```rust
use std::cell::RefCell;

struct RefCapsule<'a>{
    cell: &'a RefCell<u32>
}

impl<'a> RefCapsule<'a>{
    fn print(&self){
        println!("Value: {}", self.cell.borrow());
    }
}

fn inc(num: &RefCell<u32>){
    *num.borrow_mut() += 1
}

fn main() {
    let x = RefCell::new(20);
    let capsule = RefCapsule{ cell: &x };

    println!("x: {}", x.borrow());

    inc(&x);
    capsule.print();
}
```

```
x: 20
Value: 21
```