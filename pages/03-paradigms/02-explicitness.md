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

### Java - True Immutability Doesn't Exist

```java
int x = 5;
x = 10; // mutable by default

final int y = 5;
y = 10; // Error! Final cannot be overwritten

final List<Integer> values = new ArrayList<>();
values.add(new Integer(1)); // no error...
values = new ArrayList<>(); // Error!
```

Java only has `final`, which isn't even true immutability, it only refers to the variable not being able to be overwritten.

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

Rust forces you to declare mutability using `mut`.

---
level: 3
layout: comparison
---

## Nullability

In Rust `null` doesn't exist: if a value should be somewhere, it is, and it's impossible for it to not be.

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

Java doesn't prevent nulls from appearing where values are expected; it just throws errors at runtime.

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

There are no exceptions in Rust, instead functions return Results, which can be either `Ok` or any type to represent an error.

::left::

### Java - Exceptions

```java
int parseInt(String s) {
    return Integer.parseInt(s); // throws Exception
}

try {
    int n = parseInt("abc");
} catch (NumberFormatException e) {
    System.out.println("Error!");
}
```

Exceptions can be checked to ensure that they are caught, however it is completely unique syntax and feels like a "bolt on".

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

By adding errors to the return type it is more obvious what can and cannot fail. Just make sure to not be like Cloudflare and use `unwrap` in prod (Rust equivalent of unchecked exceptions).
