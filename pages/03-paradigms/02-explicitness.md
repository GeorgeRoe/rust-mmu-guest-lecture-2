---
level: 2
layout: center
---

# Explicitness & Correctness by Default

---
level: 3
layout: comparison
---

## Mutability

In Rust, values are immutable by default. You opt in to mutation, which helps prevent accidental state changes and enables stronger compile-time guarantees.

::left::

### Java - `final` Isn't Immutability

```java
int x = 5;
x = 10; // mutable by default

final int y = 5;
y = 10; // Error! Final cannot be overwritten

final List<Integer> values = new ArrayList<>();
values.add(1); // no error...
values = new ArrayList<>(); // Error!
```

`final` only stops the variable being reassigned; the object it points to can still change. Immutable types exist (e.g., `String`, records), but the compiler can't stop a method changing an object you pass to it.

::right::

### Rust – Immutable by Default

```rust
struct Counter { count: i32 }

impl Counter {
  pub fn peek(&self) -> i32 {
    self.count
  }

  pub fn increment(&mut self) {
    self.count += 1;
  }
}

let counter = Counter { count: 0 };
println!("count: {}", counter.peek());
counter.increment(); // Error!
```

Rust forces you to declare mutability using `mut`, and `&self` vs `&mut self`.

---
level: 3
layout: comparison
---

## Nullability

Safe Rust has no `null`: if a type says a value is there, it is, and if it might be missing, the type says that too.

::left::

### Java - Null Pointer Exceptions

```java
public class Example {
  public static void main(String[] args) {
    String result = fetchData();
    System.out.println(result.toUpperCase());
    // Runtime: NullPointerException
  }

  static String fetchData() {
    return null; // query failure / missing data
  }
}
```

Java doesn't prevent nulls from appearing where values are expected. `Optional` exists, but nothing forces you to use it, so you find out at runtime.

::right::

### Rust - Option\<T\>

```rust
// This is built in
enum Option<T> {
  None,
  Some(T),
}

let something: Option<i32> = Some(1);
let nothing: Option<i32> = None;
```

If you expect that something might not have a value, you explicitly state that using the Option enum.

---
level: 3
layout: comparison
---

## Results over Exceptions

There are no exceptions in Rust. Instead, functions that can fail return a `Result`, which is either `Ok(value)` or `Err(error)`, where the error can be any type. (Panics exist, but they're for bugs, not expected errors.)

::left::

### Java - Exceptions

```java
int parseInt(String s) {
    return Integer.parseInt(s); // throws NumberFormatException
}

try {
    int n = parseInt("abc");
} catch (NumberFormatException e) {
    System.out.println("Error!");
}
```

Checked exceptions must be declared and caught, but many (like `NumberFormatException`) are unchecked, so nothing in the signature warns you and the compiler doesn't make you handle them.

::right::

### Rust - Result

```rust
fn parse_int(s: &str) -> Result<i32, ParseIntError> {
    s.parse()
}

let n = match parse_int("abc") {
    Ok(n) => n,
    Err(_) => {
        println!("Error!");
        0
    }
};
```

By adding errors to the return type it is more obvious what can and cannot fail. Just make sure to not be like Cloudflare and use `unwrap` in prod (it panics on an `Err`, crashing much like an uncaught exception).
