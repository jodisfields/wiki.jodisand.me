# Rust Cheatsheet

A comprehensive guide to Rust - a systems programming language focused on safety, speed, and concurrency.

## Table of Contents

- [Installation](#installation)
- [Basics](#basics)
- [Variables & Data Types](#variables--data-types)
- [Control Flow](#control-flow)
- [Functions](#functions)
- [Ownership](#ownership)
- [Structs & Enums](#structs--enums)
- [Collections](#collections)
- [Error Handling](#error-handling)
- [Generics & Traits](#generics--traits)
- [Lifetimes](#lifetimes)
- [Modules](#modules)
- [Cargo](#cargo)
- [Common Patterns](#common-patterns)

## Installation

```bash
# Install Rust using rustup
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Update Rust
rustup update

# Check version
rustc --version
cargo --version

# Install Rust analyzer (LSP)
rustup component add rust-analyzer

# Format code
rustup component add rustfmt

# Linter
rustup component add clippy
```

## Basics

### Hello World

```rust
fn main() {
    println!("Hello, world!");
}
```

### Compile and Run

```bash
# Compile
rustc main.rs

# Run
./main

# Or use cargo (recommended)
cargo new hello_world
cd hello_world
cargo run
```

### Comments

```rust
// Single line comment

/* Multi-line
   comment */

/// Documentation comment for the following item
/// Supports markdown
fn documented_function() {}

//! Documentation comment for the containing item
//! (usually for modules)
```

### Print Macros

```rust
fn main() {
    // println! - with newline
    println!("Hello");

    // print! - without newline
    print!("Hello ");

    // Formatting
    println!("Number: {}", 42);
    println!("Multiple: {} and {}", 1, 2);
    println!("Named: {name}, {age}", name = "Alice", age = 30);

    // Debug print
    println!("{:?}", vec![1, 2, 3]);
    println!("{:#?}", vec![1, 2, 3]); // Pretty print

    // eprintln! - print to stderr
    eprintln!("Error message");

    // format! - create String
    let s = format!("Value: {}", 42);
}
```

## Variables & Data Types

### Variables

```rust
fn main() {
    // Immutable by default
    let x = 5;
    // x = 6; // Error!

    // Mutable variable
    let mut y = 5;
    y = 6; // OK

    // Constants (always immutable, type required)
    const MAX_POINTS: u32 = 100_000;

    // Shadowing
    let z = 5;
    let z = z + 1; // New variable, shadows previous
    let z = "text"; // Can change type
}
```

### Scalar Types

```rust
fn main() {
    // Integers
    let a: i8 = -128;         // 8-bit signed
    let b: u8 = 255;          // 8-bit unsigned
    let c: i32 = -2_147_483_648; // 32-bit signed (default)
    let d: u32 = 4_294_967_295;  // 32-bit unsigned
    let e: i64 = 123;         // 64-bit signed
    let f: i128 = 123;        // 128-bit signed
    let g: isize = 123;       // Architecture-dependent

    // Floating point
    let x: f32 = 3.14;        // 32-bit
    let y: f64 = 3.14;        // 64-bit (default)

    // Boolean
    let t: bool = true;
    let f: bool = false;

    // Character (4 bytes, Unicode)
    let c: char = 'z';
    let emoji: char = '😊';
}
```

### Compound Types

```rust
fn main() {
    // Tuple
    let tup: (i32, f64, u8) = (500, 6.4, 1);
    let (x, y, z) = tup; // Destructuring
    let first = tup.0;   // Access by index

    // Array (fixed size)
    let arr: [i32; 5] = [1, 2, 3, 4, 5];
    let first = arr[0];

    // Array with same value
    let arr = [3; 5]; // [3, 3, 3, 3, 3]

    // Slice (reference to contiguous sequence)
    let slice: &[i32] = &arr[1..3]; // [2, 3]
}
```

### Strings

```rust
fn main() {
    // String slice (immutable, fixed size)
    let s1: &str = "Hello";

    // String (growable, heap-allocated)
    let mut s2 = String::from("Hello");
    s2.push_str(", world!");

    // String methods
    let len = s2.len();
    let is_empty = s2.is_empty();
    let contains = s2.contains("world");

    // String slicing
    let hello = &s2[0..5];

    // Concatenation
    let s3 = s1.to_string() + " there";
    let s4 = format!("{} {}", s1, s2);
}
```

## Control Flow

### If/Else

```rust
fn main() {
    let number = 6;

    if number % 2 == 0 {
        println!("Even");
    } else {
        println!("Odd");
    }

    // If expression (returns value)
    let result = if number > 5 { "big" } else { "small" };

    // Multiple conditions
    if number < 0 {
        println!("Negative");
    } else if number == 0 {
        println!("Zero");
    } else {
        println!("Positive");
    }
}
```

### Loops

```rust
fn main() {
    // Infinite loop
    loop {
        println!("Forever!");
        break; // Exit loop
    }

    // Loop with return value
    let mut counter = 0;
    let result = loop {
        counter += 1;
        if counter == 10 {
            break counter * 2;
        }
    };

    // While loop
    let mut number = 3;
    while number != 0 {
        println!("{}", number);
        number -= 1;
    }

    // For loop
    let arr = [1, 2, 3, 4, 5];
    for element in arr {
        println!("{}", element);
    }

    // Range
    for number in 1..4 {  // 1, 2, 3
        println!("{}", number);
    }

    for number in 1..=4 { // 1, 2, 3, 4
        println!("{}", number);
    }

    // Enumerate
    for (index, value) in arr.iter().enumerate() {
        println!("{}: {}", index, value);
    }
}
```

### Match

```rust
fn main() {
    let number = 7;

    match number {
        1 => println!("One"),
        2 | 3 => println!("Two or three"),
        4..=10 => println!("Four through ten"),
        _ => println!("Something else"), // Default
    }

    // Match expression (returns value)
    let result = match number {
        1 => "one",
        2 => "two",
        _ => "other",
    };

    // Destructuring
    let point = (0, 5);
    match point {
        (0, y) => println!("On y-axis at {}", y),
        (x, 0) => println!("On x-axis at {}", x),
        (x, y) => println!("At ({}, {})", x, y),
    }

    // Match with guards
    let num = Some(4);
    match num {
        Some(x) if x < 5 => println!("Less than 5: {}", x),
        Some(x) => println!("{}", x),
        None => (),
    }
}
```

### If Let

```rust
fn main() {
    let some_value = Some(3);

    // Instead of match
    if let Some(3) = some_value {
        println!("Three");
    }

    // With else
    if let Some(x) = some_value {
        println!("{}", x);
    } else {
        println!("None");
    }
}
```

## Functions

### Basic Functions

```rust
fn main() {
    greet();
    let result = add(5, 3);
}

fn greet() {
    println!("Hello!");
}

fn add(x: i32, y: i32) -> i32 {
    x + y  // No semicolon = return value
    // Or: return x + y;
}

// Multiple return values (using tuple)
fn swap(x: i32, y: i32) -> (i32, i32) {
    (y, x)
}

// References
fn calculate_length(s: &String) -> usize {
    s.len()
}

// Mutable reference
fn change(s: &mut String) {
    s.push_str(", world");
}
```

### Closures

```rust
fn main() {
    // Closure (anonymous function)
    let add = |x, y| x + y;
    println!("{}", add(2, 3));

    // Capturing environment
    let x = 4;
    let equal_to_x = |z| z == x;
    println!("{}", equal_to_x(4));

    // Type annotations (optional)
    let add: fn(i32, i32) -> i32 = |x, y| x + y;

    // Move keyword (take ownership)
    let s = String::from("hello");
    let closure = move || println!("{}", s);
    closure();
    // s is no longer valid here

    // Using closures with iterators
    let numbers = vec![1, 2, 3, 4, 5];
    let doubled: Vec<i32> = numbers.iter().map(|x| x * 2).collect();
}
```

## Ownership

### Ownership Rules

1. Each value has an owner
2. There can only be one owner at a time
3. When the owner goes out of scope, the value is dropped

```rust
fn main() {
    // s is not valid here
    let s = String::from("hello"); // s is valid from this point

    // s is valid here
    takes_ownership(s);
    // s is no longer valid here (moved)

    let x = 5;
    makes_copy(x);
    // x is still valid (copied, not moved)
}

fn takes_ownership(some_string: String) {
    println!("{}", some_string);
} // some_string goes out of scope, drop is called

fn makes_copy(some_integer: i32) {
    println!("{}", some_integer);
} // some_integer goes out of scope, nothing special happens
```

### References & Borrowing

```rust
fn main() {
    let s1 = String::from("hello");

    // Immutable reference (borrowing)
    let len = calculate_length(&s1);
    println!("Length of '{}' is {}", s1, len);

    // Mutable reference
    let mut s2 = String::from("hello");
    change(&mut s2);

    // Multiple immutable references OK
    let r1 = &s1;
    let r2 = &s1;
    println!("{} and {}", r1, r2);

    // Cannot have mutable and immutable references simultaneously
    // let r3 = &s1;
    // let r4 = &mut s1; // Error!
}

fn calculate_length(s: &String) -> usize {
    s.len()
}

fn change(s: &mut String) {
    s.push_str(", world");
}
```

### Slice Type

```rust
fn main() {
    let s = String::from("hello world");

    // String slices
    let hello = &s[0..5];
    let world = &s[6..11];
    let hello = &s[..5];   // Same as [0..5]
    let world = &s[6..];   // Same as [6..len]
    let whole = &s[..];    // Entire string

    // Array slices
    let arr = [1, 2, 3, 4, 5];
    let slice = &arr[1..3]; // [2, 3]
}

fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[..i];
        }
    }

    &s[..]
}
```

## Structs & Enums

### Structs

```rust
// Define struct
struct User {
    username: String,
    email: String,
    age: u32,
    active: bool,
}

impl User {
    // Associated function (like static method)
    fn new(username: String, email: String, age: u32) -> User {
        User {
            username,
            email,
            age,
            active: true,
        }
    }

    // Method (takes self)
    fn is_adult(&self) -> bool {
        self.age >= 18
    }

    // Mutable method
    fn deactivate(&mut self) {
        self.active = false;
    }

    // Takes ownership
    fn consume(self) -> String {
        self.username
    }
}

fn main() {
    // Create instance
    let user1 = User {
        username: String::from("alice"),
        email: String::from("alice@example.com"),
        age: 30,
        active: true,
    };

    // Access fields
    println!("{}", user1.username);

    // Mutable instance
    let mut user2 = User::new(
        String::from("bob"),
        String::from("bob@example.com"),
        25
    );
    user2.age = 26;

    // Struct update syntax
    let user3 = User {
        email: String::from("charlie@example.com"),
        ..user1  // Use remaining fields from user1
    };

    // Tuple structs
    struct Color(i32, i32, i32);
    let black = Color(0, 0, 0);

    // Unit-like structs
    struct AlwaysEqual;
    let subject = AlwaysEqual;
}
```

### Enums

```rust
// Basic enum
enum IpAddrKind {
    V4,
    V6,
}

// Enum with data
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

// Enum with different types
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        match self {
            Message::Quit => println!("Quit"),
            Message::Move { x, y } => println!("Move to ({}, {})", x, y),
            Message::Write(s) => println!("{}", s),
            Message::ChangeColor(r, g, b) => println!("RGB({}, {}, {})", r, g, b),
        }
    }
}

fn main() {
    let home = IpAddr::V4(127, 0, 0, 1);
    let loopback = IpAddr::V6(String::from("::1"));

    let msg = Message::Write(String::from("Hello"));
    msg.call();
}
```

### Option Enum

```rust
fn main() {
    // Option<T> - for values that might not exist
    let some_number = Some(5);
    let some_string = Some("a string");
    let absent_number: Option<i32> = None;

    // Using match
    let x: Option<i32> = Some(2);
    match x {
        Some(i) => println!("{}", i),
        None => println!("Nothing"),
    }

    // Using if let
    if let Some(i) = x {
        println!("{}", i);
    }

    // Unwrap (panics if None)
    let value = some_number.unwrap();

    // Unwrap with default
    let value = absent_number.unwrap_or(0);

    // Map
    let incremented = some_number.map(|x| x + 1);

    // And_then
    let result = some_number.and_then(|x| Some(x * 2));
}
```

### Result Enum

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    // Result<T, E> - for operations that might fail
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => panic!("Problem opening file: {:?}", error),
    };

    // Match on error kind
    let f = File::open("hello.txt");
    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating file: {:?}", e),
            },
            other_error => panic!("Problem opening file: {:?}", other_error),
        },
    };

    // Unwrap (panics on error)
    let f = File::open("hello.txt").unwrap();

    // Expect (panics with custom message)
    let f = File::open("hello.txt").expect("Failed to open file");

    // Question mark operator (propagate error)
    fn read_username() -> Result<String, std::io::Error> {
        let mut f = File::open("username.txt")?;
        let mut s = String::new();
        f.read_to_string(&mut s)?;
        Ok(s)
    }
}
```

## Collections

### Vector

```rust
fn main() {
    // Create vector
    let mut v: Vec<i32> = Vec::new();
    let v = vec![1, 2, 3]; // Using macro

    // Add elements
    v.push(4);
    v.push(5);

    // Access elements
    let third = &v[2];        // Panics if out of bounds
    let third = v.get(2);     // Returns Option<&T>

    match v.get(2) {
        Some(third) => println!("{}", third),
        None => println!("No third element"),
    }

    // Iterate
    for i in &v {
        println!("{}", i);
    }

    // Iterate with mutation
    for i in &mut v {
        *i += 50;
    }

    // Vector methods
    v.pop();              // Remove last element
    v.len();              // Length
    v.is_empty();         // Check if empty
    v.clear();            // Remove all elements
    v.contains(&3);       // Check if contains
    v.reverse();          // Reverse
    v.sort();             // Sort

    // Storing different types (using enum)
    enum SpreadsheetCell {
        Int(i32),
        Float(f64),
        Text(String),
    }

    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Float(10.12),
        SpreadsheetCell::Text(String::from("blue")),
    ];
}
```

### HashMap

```rust
use std::collections::HashMap;

fn main() {
    // Create HashMap
    let mut scores = HashMap::new();

    // Insert
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    // Access
    let team = String::from("Blue");
    let score = scores.get(&team);  // Returns Option<&V>
    let score = scores.get(&team).copied().unwrap_or(0);

    // Iterate
    for (key, value) in &scores {
        println!("{}: {}", key, value);
    }

    // Update
    scores.insert(String::from("Blue"), 25); // Overwrites

    // Insert if not exists
    scores.entry(String::from("Red")).or_insert(50);

    // Update based on old value
    let text = "hello world wonderful world";
    let mut map = HashMap::new();
    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }

    // HashMap methods
    scores.remove(&team);       // Remove entry
    scores.contains_key(&team); // Check if key exists
    scores.len();               // Number of entries
    scores.clear();             // Remove all entries
}
```

### HashSet

```rust
use std::collections::HashSet;

fn main() {
    let mut set = HashSet::new();

    // Insert
    set.insert(1);
    set.insert(2);
    set.insert(3);

    // Check membership
    if set.contains(&1) {
        println!("Set contains 1");
    }

    // Remove
    set.remove(&1);

    // Set operations
    let a: HashSet<_> = [1, 2, 3].iter().cloned().collect();
    let b: HashSet<_> = [2, 3, 4].iter().cloned().collect();

    let union: HashSet<_> = a.union(&b).cloned().collect();
    let intersection: HashSet<_> = a.intersection(&b).cloned().collect();
    let difference: HashSet<_> = a.difference(&b).cloned().collect();
}
```

## Error Handling

### Panic

```rust
fn main() {
    // Unrecoverable error
    panic!("Crash and burn");

    // Causes panic
    let v = vec![1, 2, 3];
    v[99]; // Index out of bounds
}
```

### Result

```rust
use std::fs::File;
use std::io::{self, Read};

// Returning Result
fn read_file(path: &str) -> Result<String, io::Error> {
    let mut file = File::open(path)?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents)
}

// Custom error type
use std::fmt;

#[derive(Debug)]
struct MyError {
    details: String
}

impl MyError {
    fn new(msg: &str) -> MyError {
        MyError { details: msg.to_string() }
    }
}

impl fmt::Display for MyError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}", self.details)
    }
}

impl std::error::Error for MyError {}

// Using custom error
fn do_something() -> Result<(), MyError> {
    Err(MyError::new("Something went wrong"))
}
```

## Generics & Traits

### Generics

```rust
// Generic function
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}

// Generic struct
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

// Specific implementation
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}

// Generic enum
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### Traits

```rust
// Define trait
trait Summary {
    fn summarize(&self) -> String;

    // Default implementation
    fn author(&self) -> String {
        String::from("Unknown")
    }
}

struct Article {
    title: String,
    content: String,
}

// Implement trait
impl Summary for Article {
    fn summarize(&self) -> String {
        format!("{}: {}", self.title, self.content)
    }
}

// Trait as parameter
fn notify(item: &impl Summary) {
    println!("{}", item.summarize());
}

// Trait bound syntax
fn notify<T: Summary>(item: &T) {
    println!("{}", item.summarize());
}

// Multiple trait bounds
fn notify<T: Summary + Display>(item: &T) {}

// Where clause
fn notify<T>(item: &T)
where
    T: Summary + Display,
{}

// Return trait
fn returns_summarizable() -> impl Summary {
    Article {
        title: String::from("Title"),
        content: String::from("Content"),
    }
}

// Derive common traits
#[derive(Debug, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}
```

## Lifetimes

### Lifetime Syntax

```rust
// Lifetime annotations
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

// Struct with lifetime
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }

    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention: {}", announcement);
        self.part
    }
}

// Static lifetime
let s: &'static str = "I have a static lifetime";
```

## Modules

### Module System

```rust
// src/lib.rs or src/main.rs
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}
        fn serve_order() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}

// Using 'use'
use crate::front_of_house::hosting;

pub fn eat_at_restaurant2() {
    hosting::add_to_waitlist();
}

// Re-export
pub use crate::front_of_house::hosting;

// Nested paths
use std::{cmp::Ordering, io};
use std::io::{self, Write};

// Glob operator
use std::collections::*;
```

### External Files

```rust
// src/lib.rs
mod front_of_house;  // Loads from src/front_of_house.rs

// src/front_of_house.rs
pub mod hosting;  // Loads from src/front_of_house/hosting.rs

// src/front_of_house/hosting.rs
pub fn add_to_waitlist() {}
```

## Cargo

### Project Structure

```
my_project/
├── Cargo.toml
├── Cargo.lock
├── src/
│   ├── main.rs       # Binary crate
│   ├── lib.rs        # Library crate
│   └── bin/          # Additional binaries
│       └── tool.rs
├── tests/            # Integration tests
│   └── integration_test.rs
├── benches/          # Benchmarks
│   └── benchmark.rs
└── examples/         # Examples
    └── example.rs
```

### Cargo Commands

```bash
# Create new project
cargo new my_project
cargo new --lib my_library

# Build
cargo build             # Debug build
cargo build --release   # Optimized build

# Run
cargo run
cargo run --release

# Check (faster than build)
cargo check

# Test
cargo test
cargo test test_name
cargo test -- --nocapture  # Show println output

# Documentation
cargo doc
cargo doc --open

# Format
cargo fmt

# Lint
cargo clippy

# Update dependencies
cargo update

# Clean build artifacts
cargo clean

# Install binary
cargo install ripgrep

# Publish to crates.io
cargo publish
```

### Cargo.toml

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"
authors = ["Your Name <you@example.com>"]

[dependencies]
serde = "1.0"
serde_json = "1.0"
tokio = { version = "1.0", features = ["full"] }

[dev-dependencies]
criterion = "0.5"

[profile.release]
opt-level = 3
lto = true

[[bin]]
name = "my-tool"
path = "src/bin/tool.rs"
```

## Common Patterns

### Iterator Pattern

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    // Map
    let v2: Vec<_> = v.iter().map(|x| x * 2).collect();

    // Filter
    let v3: Vec<_> = v.iter().filter(|&&x| x > 2).collect();

    // Fold (reduce)
    let sum: i32 = v.iter().fold(0, |acc, &x| acc + x);

    // Chain
    let v4: Vec<_> = v
        .iter()
        .filter(|&&x| x % 2 == 0)
        .map(|x| x * 2)
        .collect();

    // For_each
    v.iter().for_each(|x| println!("{}", x));

    // Any/All
    let has_even = v.iter().any(|&x| x % 2 == 0);
    let all_positive = v.iter().all(|&x| x > 0);

    // Find
    let first_even = v.iter().find(|&&x| x % 2 == 0);

    // Enumerate
    for (i, &val) in v.iter().enumerate() {
        println!("{}: {}", i, val);
    }
}
```

### Smart Pointers

```rust
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    // Box<T> - heap allocation
    let b = Box::new(5);
    println!("{}", b);

    // Rc<T> - reference counting
    let a = Rc::new(5);
    let b = Rc::clone(&a);
    println!("Count: {}", Rc::strong_count(&a));

    // RefCell<T> - interior mutability
    let value = RefCell::new(5);
    *value.borrow_mut() += 10;
    println!("{}", value.borrow());

    // Combining Rc and RefCell
    let shared = Rc::new(RefCell::new(5));
    let a = Rc::clone(&shared);
    *a.borrow_mut() += 1;
}
```

### Concurrency

```rust
use std::thread;
use std::sync::{Arc, Mutex};

fn main() {
    // Spawn thread
    let handle = thread::spawn(|| {
        println!("Hello from thread!");
    });
    handle.join().unwrap();

    // Move closure
    let v = vec![1, 2, 3];
    let handle = thread::spawn(move || {
        println!("{:?}", v);
    });
    handle.join().unwrap();

    // Shared state with Arc and Mutex
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

## Additional Resources

- [The Rust Programming Language (Book)](https://doc.rust-lang.org/book/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [Rust Standard Library Documentation](https://doc.rust-lang.org/std/)
- [Rust Playground](https://play.rust-lang.org/)
- [Crates.io](https://crates.io/) - Package registry

---

*Last updated: 2025-11-16*
