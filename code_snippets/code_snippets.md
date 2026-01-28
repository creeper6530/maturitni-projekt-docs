# Scopes, shadowing and mutability

Fails to compile (pic 3, error msg: pic 4):

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

<hr>Compiles successfully (pic 5).

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

<hr>Compiles with warning, acts unexpectedly (pic 6).

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

<hr>Fails to compile (pic 7, error msg w/o warning: pic 8).

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

<hr>Compiles successfully (pic 9).

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

Compiles successfully (pic 10).

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

<hr>These fail to compile because of overflowing literal (error msg of latter: pic 11):

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

<hr>These compile successfully (latter: pic 12)

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

Compiles successfully (pic 13):

```rust
#![allow(unused)]
fn main() {
    let a: char = 'a';
    let b: char = '💜';
    let c: bool = true;
    let d: &str = "Hello";
    let e: [u8; 5] = [1, 2, 3, 4, 5];
    let mut f: [bool; 3] = [true, true, true];
    f[1] = false;
    let g: (u16, i64) = (20, -50_000);
    let mut h: (f64, &str, bool) = (
        3.1415926535,
        "Pi",
        true
    );
    h.0 = 3.0;
    
    let mut i: [[u8; 5]; 5] = [
        [00, 00, 11, 00, 00],
        [00, 11, 11, 11, 00],
        [11, 00, 11, 00, 11],
        [00, 00, 11, 00, 00],
        [00, 00, 11, 00, 00],
    ];
    i[1][3] = 22;
}
```

# Initialisation

Compiles (pic 14):

```rust
fn main() {
    let a;
    let b: bool;
    let mut c: u8;

    {
        a = 20;
        b = false;
        c = 15;
    };
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

<br>Fails to compile (pic 15, error msg: pic 16):

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

Fails to compile (pic 17, error msg: pic 18):

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

<hr>Compiles successfully (pic 19):

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

<hr>Fails to compile (pic 20, error msg: pic 21):

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

<hr>Fails to compile (pic 22, error msg: pic 23):

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

<hr>Fails to compile (pic 24, error msg: pic 25):

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

<hr>Compiles successfully (pic 26):

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

# Structs and enums

Compiles successfully (pic 31):

```rust
#![allow(unused)]
enum Smer {
    Vpred,
    Vzad,
    Vlevo,
    Vpravo
}

fn main() {
    let forwards = Smer::Vpred;
    let left = Smer::Vlevo;
}
```

<hr>Compiles successfully (pic 33):

```rust
#![allow(unused)]
enum Smer {
    Vpred,
    Vzad,
    Vlevo,
    Vpravo
}

enum Akce {
    NedelejNic,
    OtevriInventar,
    Rekni(String),
    JdiNaSouradnice(i32, i32),
    PodivejSeNa { predmet: String, smer: Smer }
}

fn main() {
    let akce1 = Akce::Rekni(String::from("Ahoj!"));
    let akce2 = Akce::JdiNaSouradnice(20, 45);
    let akce3 = Akce::PodivejSeNa {
        predmet: String::from("Meč sv. Jiří"),
        smer: Smer::Vlevo
    };
    let akce4 = Akce::OtevriInventar;
}
```

# Error handling

Compiles successfully, panics (pic 36):

```rust
#![allow(unused)]
use std::fs::File;

fn main() {
    let greeting_file_1 = File::open("hello1.txt").unwrap();
    let greeting_file_2 = File::open("hello2.txt")
        .expect("hello.txt should be included in this project");
}
```

<hr>Compiles successfully (pic 37):

```rust
#![allow(unused)]
use std::fs::File;

fn main() {
    let file_result: Result<_, _> = File::open("hello.txt");

    let file = match file_result {
        Ok(mut file) => { // Unnecessary mut
            println!("File found, opening.");
            file
        },
        Err(error) => match error.kind() {
            ErrorKind::NotFound => {
                println!("File not found, creating file.");
                File::create("hello.txt")
                    .expect("Failed to create file")
            }
            _ => panic!("Problem opening the file: {error:?}")
        },
    };
}
```

<hr>Compiles successfully (pic 42):

```rust
#![allow(unused)]

fn main() {
    let num: Option<u8> = None;
    let message = Some("Hello!");
    maybe_say(message);
}

fn maybe_say(input: Option<&str>) {
    match input {
        None => return,
        Some(msg) => println!("{}", msg)
    }
}
```

# Generics and traits

Fails to compile (pic 43, error msg: pic 44):

```rust
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
    }}

    largest
}

fn main() {
    let result_num = largest(&[1, 0, 15, -20, 89]);
    println!("The largest number is {result_num}");

    let result_char = largest(&['a', 'Z', 'g', 'P']);
    println!("The largest char is {result_char}");
}
```

```
error[E0369]: binary operation `>` cannot be applied to type `&T`
 --> test.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- &T
  |            |
  |            &T
  |
help: consider restricting type parameter `T` with trait `PartialOrd`
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T {
  |             ++++++++++++++++++++++

error: aborting due to 1 previous error

For more information about this error, try `rustc --explain E0369`.
```

<hr>Compiles successfully (pic 46):

```rust
#![allow(unused)]

fn main() {}

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: PartialOrd> Pair<T> {
    fn cmp_display(&self) -> bool{
        if self.x >= self.y { true } else { false }
    }
}
```

<hr>Compiles successfully (pic 47):

```rust
fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> Option<&T> {
    if list.len() == 0 { return None };

    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }

    Some(largest)
}

fn main() {
    println!("The largest number is {:?}", largest(&[1, 0, 15, -20, 89]));
    println!("The largest char is {:?}", largest(&['a', 'Z', 'g', 'P']));

    println!("The largest float is {:?}", largest::<f64>(&[]));
}
```

<hr>Compiles successfully (unused):

```rust
#[derive(Clone, Copy, Debug)]
struct Copiable {}

fn main() {
    let x = Copiable{};
    let y = x;

    println!("{:?}", x);
    println!("{:?}", y);
}
```

<hr>Compiles successfully (pic 48; not entirely the same):

```rust
#![allow(unused)]

trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}

struct NewsArticle {
    headline: String,
    location: String,
    author: String,
    content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }

    fn summarize_author(&self) -> String {
        format!("By {}", self.author)
    }
}

struct SocialPost {
    username: String,
    content: String,
    reply: bool,
    repost: bool,
}

impl Summary for SocialPost {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}

fn main() {
    let article = NewsArticle {
        headline: String::from("Penguins win the Stanley Cup Championship!"),
        location: String::from("Pittsburgh, PA, USA"),
        author: String::from("John Doe"),
        content: String::from("The Pittsburgh Penguins once again are the best hockey team."),
    };
    println!("{}", article.summarize());
    let post = SocialPost {
        username: String::from("horse_ebooks"),
        content: String::from("The best way to learn Rust is to write Rust."),
        reply: false,
        repost: false,
    };
    println!("{}", post.summarize());
}
```

```
Penguins win the Stanley Cup Championship!, by John Doe (Pittsburgh, PA, USA)
(Read more from @horse_ebooks...)
```

<hr>Compiles successfully (pic 49):

```rust
#![allow(unused)]
#![allow(nonstandard_style)]

#[derive(Debug)]
enum Freq {
    Hz(u32),
    kHz(u32),
    MHz(u32),
    GHz(u32)
}

impl std::fmt::Display for Freq {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            Freq::Hz(val) => write!(f, "{} hertz", val),
            Freq::kHz(val) => write!(f, "{} kilohertz", val),
            Freq::MHz(val) => write!(f, "{} megahertz", val),
            Freq::GHz(val) => write!(f, "{} gigahertz", val),
        }
    }
}

fn main() {
    let freq = Freq::kHz(201);
    println!("{}", freq);
    println!("{:?}", freq);
}
```

```
201 kilohertz
kHz(201)
```

<hr>Does not compile (pic 50):

```rust
#![allow(unused)]

fn main() {}

fn compiles<'a>() -> impl Iterator<Item = &'a u16> {
    let arr: &[u16] = &[1, 1, 2, 3, 5, 8, 13];
    arr.iter()
}

fn compiles_too(maybe: bool) -> impl Iterator {
    let arr: &[i64];
    if maybe {
        arr = &[1, 1, 2, 3, 5, 8, 13];
    } else {
        arr = &[42];
    };
    arr.iter()
}

fn not_compiles(maybe: bool) -> impl Iterator {
    if maybe { "".chars() } else { [].iter() }
}
```

```
error[E0308]: `if` and `else` have incompatible types
  --> .\test_code.rs:21:36
   |
21 |     if maybe { "".chars() } else { [].iter() }
   |                ----------          ^^^^^^^^^ expected `Chars<'_>`, found `Iter<'_, _>`
   |                |
   |                expected because of this
   |
   = note: expected struct `Chars<'_>`
              found struct `std::slice::Iter<'_, _>`
help: you could change the return type to be a boxed trait object
   |
20 - fn not_compiles(maybe: bool) -> impl Iterator {
20 + fn not_compiles(maybe: bool) -> Box<dyn Iterator> {
   |
help: if you change the return type to expect trait objects, box the returned expressions
   |
21 |     if maybe { Box::new("".chars()) } else { Box::new([].iter()) }
   |                +++++++++          +          +++++++++         +
```

<hr>Compiles successfully (unused):

```rust
#![allow(unused)]

fn main() {}

fn fixed(maybe: bool) -> Box<dyn Iterator<Item = char>> {
    if maybe {
        Box::new("".chars())
    } else {
        Box::new(['a'].iter().copied())
    }
}
```

# Closures

Compiles (pic 51):

```rust
#![allow(unused)]

fn main() {
    fn  add_one_v1   (x: u32) -> u32 { x + 1 }
    let add_one_v2 = |x: u32| -> u32 { x + 1 };
    let add_one_v3 = |x|             { x + 1 };
    let add_one_v4 = |x|               x + 1  ;

    // Works fine without
    /*add_one_v1(5_u32);
    add_one_v2(5_u32);*/

    // Needed for the closures to infer their types
    add_one_v3(5_u32);
    add_one_v4(5_u32);
}
```

<hr> Fails to compile (pic 52, error on pic 53):

```rust
#![allow(unused)]

fn main() {
    let example_closure = |x| x;

    let s = example_closure(String::from("hello"));
    let n = example_closure(5);
}
```

```
error[E0308]: mismatched types
 --> .\test_code.rs:7:29
  |
7 |     let n = example_closure(5);
  |             --------------- ^ expected `String`, found integer
  |             |
  |             arguments to this function are incorrect
  |
note: expected because the closure was earlier called with an argument of type `String`
 --> .\test_code.rs:6:29
  |
6 |     let s = example_closure(String::from("hello"));
  |             --------------- ^^^^^^^^^^^^^^^^^^^^^ expected because this argument is of type `String`
  |             |
  |             in this closure call
note: closure parameter defined here
 --> .\test_code.rs:4:28
  |
4 |     let example_closure = |x| x;
  |                            ^
help: try using a conversion method
  |
7 |     let n = example_closure(5.to_string());
  |                              ++++++++++++
```

<hr>Compiles (unused):

```rust
#![allow(unused)]

fn main() {
    let x = 42;
    let no_params = || { x * 2 };

    no_params();
}
```

<hr>Fails to compile (pic 54, error on pic 55):

```rust
#![allow(unused)]

fn main() {
    let mut x = String::from("abc");
    let mut no_params = || {
        x.make_ascii_uppercase();
    };

    println!("{}", x);
    no_params();
}
```

```
error[E0502]: cannot borrow `x` as immutable because it is also borrowed as mutable
  --> .\test_code.rs:9:20
   |
 5 |     let mut no_params = || {
   |                         -- mutable borrow occurs here
 6 |         x.make_ascii_uppercase();
   |         - first borrow occurs due to use of `x` in closure
...
 9 |     println!("{}", x);
   |                    ^ immutable borrow occurs here
10 |     no_params();
   |     --------- mutable borrow later used here
   |
   = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)
```

<hr>Compiles successfully (pic 56):

```rust
#![allow(unused)]

fn main() {
    let mut x = String::from("abc");
    let mut up = false;
    let mut no_params = move || {
        if up {
            x.make_ascii_lowercase();
            up = false;
        } else {
            x.make_ascii_uppercase();
            up = true;
        };
        print!("{} ", x);
    };

    no_params();
    no_params();
    no_params();
    // Would throw "borrow of moved value" error,
    // like seen in picture 18
    //println!("{}", x);
}
```

```
ABC abc ABC 
```

<hr>Compiles successfully (pic 57):

```rust
#![allow(unused)]

fn test(f: impl Fn(u8) -> i32) {
    f(20);
}

fn test2(f: fn (u8) -> i32) {
    f(55);
}

fn main() {
    let clos = |_| { println!("Hello!"); 0 };

    test(clos);
    test2(clos);
}
```

# Patterns

Compiles successfully (pic 41):

```rust
fn main() {
    let x = Some(0);

    match x {
        Some(value) => println!("The value is: {}", value),
        None => println!("No value found."),
    }

    if let Some(value) = x {
        println!("The value is: {}", value);
    } else {
        println!("No value found.");
    }
    //println!("The value is: {}", value);

    // -------------------------------------------------------

    let y = Some(10);

    let num;
    match y {
        Some(value) => { num = value },
        None => {
            println!("Error occurred.");
            return;
        }
    }
    println!("The value is: {}", num);

    let Some(num) = y else {
        println!("Error occurred.");
        return;
    };
    println!("The value is: {}", num);
}
```

```
The value is: 0
The value is: 0
The value is: 10
The value is: 10
```

<hr> Compiles successfully (unused):

```rust
fn main() {
    let x: u32 = 6;
    let y = Some(42);
    let z = "hello world";

    match x {
        0 => println!("x is zero"),
        1 | 6 | 8 | 9 => println!("x is a nice number"),
        2 | 3 | 4 | 5 | 7 => println!("x is an ugly number"),
        10..=99 => println!("x is double digit number"),
        100.. => println!("x is triple digit or more"),
    }
    match y {
        Some(num) if (num % 2 == 0) => println!("The number {num} is even"),
        Some(num) => println!("The number {num} is odd"),
        None => println!("No number provided"),
    }
    match z {
        string if string.starts_with("hello") => println!("Greeting detected: {string}"),
        string => println!("Normal string: {string}"),
    }
}
```

```
x is a nice number
The number 42 is even
Greeting detected: hello world
```


<details>
<summary><h1> Output 3 </h1></summary>

Compiles successfully:

```rust
fn function() {
    println!("called `function()`");
}

mod sister {
    pub fn function() {
        println!("called `sister::function()`");
    }
}

mod ctx {
    mod daughter {
        pub fn function() {
            println!("called `ctx::daughter::function()`");

            // Accessing topmost scope
            crate::sister::function();
            
            // Accessing "grandparent" scope - two levels up
            super::super::function();
        }
    }
    
    fn function() {
        println!("called `ctx::function()`");
    }

    pub fn test_all() {
        // Equivalent
        function();
        self::function();

        daughter::function();

        // Accessing parent scope
        super::function();
    }
}

use ctx::test_all;
use ctx::daughter::function as daughter_function;

fn main() {
    test_all();

    daughter_function();
}
```

```
called `ctx::function()`
called `ctx::function()`
called `ctx::daughter::function()`
called `sister::function()`
called `function()`
called `function()`
called `ctx::daughter::function()`
called `sister::function()`
called `function()`
```
</details>