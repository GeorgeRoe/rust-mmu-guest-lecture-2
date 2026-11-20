---
level: 2
layout: center
---

# Memory Safety & Resource Management

---
level: 3
layout: comparison
---

## Memory Deallocation

Rust uses ownership rules to work out at compile time exactly where memory should be freed, so it's freed deterministically at runtime. This means there are no GC pauses and also no manual memory management (e.g., `free()`).

::left::

### Java — Garbage Collection

```java
Object data = new Object();
// Freed when GC decides
```

Memory is reclaimed unpredictably and may pause execution (modern collectors keep pauses short, but there is still a collector running).

::right::

### Rust — Freed at End of Scope

```rust
{
  let data = vec![1, 2, 3];
} // freed here immediately
```

Memory is released as soon as values go out of scope, with no garbage collector running in the background.

---
level: 3
layout: comparison
---

## Ownership

Rust has strict ownership rules to ensure code is safe.

::left::

### Java - Anyone can own anything

```java
class IntBox {
  public int value;

  public IntBox(int value) {
    this.value = value;
  }
}

IntBox a = new IntBox(1);
IntBox b = a; // Two references to the same object

a.value = 2;

System.out.println(b.value); // 2
```

It is not only possible, but very easy for two variables to point to the same instance. Any code holding a reference can change it, and if that code is on another thread, you get data races.

::right::

### Rust - Strict Ownership

```rust
struct IntBox {
  pub value: i32
}

let mut a = IntBox { value: 1 };

let b = &mut a;
let c = &a; // Error! can't borrow a while b is in use
b.value = 2;

let d = &a;
d.value = 3; // Error! d is an immutable reference

let e = a; // ownership moves to e
println!("{}", a.value); // Error! a has been moved
```

- At most one mutable reference can exist at a time
- Multiple immutable references are allowed
- Mutable and immutable references cannot coexist (a reference lasts until its last use)
