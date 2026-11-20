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

Rust frees memory deterministically at compile time using ownership rules. This means there are no GC pauses and also no manual memory management (e.g., `free()`).

::left::

### Java — Garbage Collection

```java
Object data = new Object();
// Freed when GC decides
```

Memory is reclaimed unpredictably and may pause execution.

::right::

### Rust — Compiled

```rust
{
  let data = vec![1, 2, 3];
} // freed here immediately
```

Memory is released when values go out of scope leading to zero runtime cost.

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
b = a; // Two references to the same object

a.value = 2;

System.out.println(b.value); // 2
```

It is not only possible, but very easy for two variables to point to the same instance. This can lead to data races.

::right::

### Rust - Strict Ownership

```rust
struct IntBox {
  pub value: i32
}

let mut a = IntBox { value: 1 };
let b = &mut a; // cannot have two mutable references

let c = &a; // both types of reference can't coexist

c.value = 2; // cannot mutate an immutable reference

let d = a;
println!("{}", a.value); // a's data has been moved
```

- At most one mutable reference can exist at a time
- Multiple immutable references are allowed
- Mutable and immutable references cannot coexist
