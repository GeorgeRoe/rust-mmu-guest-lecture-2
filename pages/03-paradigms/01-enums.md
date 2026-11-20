---
level: 2
layout: center
---

# Enums & Pattern Matching

<!--
- The first feature, and probably the one that changes how you write code the most day to day.
- Everything in this section builds up to one idea: make invalid states unrepresentable.
-->

---
level: 3
---

## Enums Are Just Names

```java
enum Status { READY, RUNNING, FAILED }

class Simulation {
  Status status;
  long step;
  String error;
}

// Creating a simulation with an impossible state:
Simulation sim = new Simulation();
sim.status = Status.READY;
sim.error = "out of memory";
```

<!--
- This is the classic Java enum most of you will have used. Each variant is just a name.
- As soon as a state needs some data - a step number, an error message - it goes in a field on the class.
- And now you can build nonsense: a READY simulation with an error, a FAILED simulation with no error, a RUNNING simulation whose step is left over from last time.
- The comments are the only thing describing the rules. The compiler has no idea.
- So you get defensive null checks everywhere.
- Fair to Java: since Java 21 you can model this properly with a sealed interface and a record per variant, plus pattern matching in switch. But it's four type declarations instead of one, it's opt-in, and almost no existing code or library does it.
-->

---
level: 3
---

## Enums Are Algebraic Data Types

```rust
enum Simulation {
    Ready,
    Running(u64),
    Failed(String),
}

let sim = Simulation::Failed("out of memory".into());
```

<!--
- One enum, three variants, each carrying exactly the data that makes sense for it.
- No separate status field and no class around it - the enum *is* the simulation and its state.
- A step number only exists while it's Running, an error message only exists once it's Failed.
- There's no field to forget to clear, and a Ready simulation with an error can't even be written down.
- This is the idea the rest of this section builds on - and the whole Rust standard library is built on it, as the next few slides show.
-->

---
level: 3
---

## Exhaustive Matching

````md magic-move {lines: true}
```rust
enum Simulation { Ready, Running(u64), Failed(String) }

fn describe(sim: &Simulation) -> String {
    match sim {
        Simulation::Ready => "ready".to_string(),
        Simulation::Running(n) => format!("step {n}"),
        Simulation::Failed(msg) => format!("failed: {msg}"),
    }
}
```

```rust
enum Simulation { Ready, Running(u64), Failed(String), Cancelled }

fn describe(sim: &Simulation) -> String {
    match sim {
        Simulation::Ready => "ready".to_string(),
        Simulation::Running(n) => format!("step {n}"),
        Simulation::Failed(msg) => format!("failed: {msg}"),
    }
}
```

```rust
enum Simulation { Ready, Running(u64), Failed(String), Cancelled }

fn describe(sim: &Simulation) -> String {
    match sim {
//        ^^^ error[E0004]: non-exhaustive patterns: `&Simulation::Cancelled` not covered
        Simulation::Ready => "ready".to_string(),
        Simulation::Running(n) => format!("step {n}"),
        Simulation::Failed(msg) => format!("failed: {msg}"),
    }
}
```

```rust
enum Simulation { Ready, Running(u64), Failed(String), Cancelled }

fn describe(sim: &Simulation) -> String {
    match sim {
        Simulation::Ready => "ready".to_string(),
        Simulation::Running(n) => format!("step {n}"),
        Simulation::Failed(msg) => format!("failed: {msg}"),
        Simulation::Cancelled => "cancelled".to_string(),
    }
}
```
````

<!--
- Start: a match that handles all three variants. Compiles.
- [click] Someone adds a Cancelled variant to the enum. In a big codebase, that enum is matched on in dozens of places.
- [click] The compiler refuses to build until every one of them handles it. This is the real error - it even suggests adding `&Simulation::Cancelled => todo!()`.
  - Java: switch expressions and pattern-matching switches are checked, but the classic `switch (x) { case A: ... }` statement isn't - which is what most existing code uses.
- [click] Handle it, and you *know* you've covered every case. Refactoring stops being scary.
- Escape hatch exists: a `_ =>` wildcard arm. Use it sparingly, because it switches this check off.
-->

---
level: 3
---

## Pattern Matching

```rust
fn describe(sim: &Simulation) -> String {
    match sim {
        Simulation::Ready => "waiting".into(),
        Simulation::Running(0) => "just started".into(),
        Simulation::Running(n) if n % 100 == 0 => format!("checkpoint at step {n}"),
        Simulation::Running(n) => format!("step {n}"),
        Simulation::Failed(msg) if msg.contains("memory") => "out of memory!".into(),
        Simulation::Failed(msg) => format!("failed: {msg}"),
    }
}
```

<!--
- One match, seven arms, every one of them doing something a Java switch can't easily do (at least before Java 21):
  - `Running(n)` / `Failed(msg)` pull the data out of the variant
  - `Running(0)` matches an exact value inside a variant
  - `n @ 1..=9` matches a range and binds the value to n
  - `if n % 100 == 0` and `if msg.contains("memory")` are guards - extra conditions on the arm
  - First matching arm wins, so order matters - and the compiler warns you if an arm can never be reached.
- [click] Two shorthands you'll use constantly. find_user returns an Option - which is coming up next.
  - if let: "if this is a Some, give me the inside"
  - let else: "this must be a Some, otherwise bail out" - keeps the happy path unindented.
-->

---
level: 3
---

## `null` Is Everywhere

```java
String findUser(int id) {
  return id == 1 ? "Duke" : null;
}

String name = findUser(7);
System.out.println(name.toUpperCase());
// Runtime: NullPointerException
```

<div class="mt-6 opacity-80">

> I call it my billion-dollar mistake.
>
> *Tony Hoare, who invented null references in 1965*

</div>

<!--
- The signature is lying. "Returns a String" really means "returns a String or nothing".
- Every reference in Java is secretly "maybe nothing". Annotations like @Nullable help, but the compiler doesn't enforce them.
- And the NPE happens where you *use* the value, which might be three classes away from where it was set to null.
- Tony Hoare said this in 2009, about adding null references to ALGOL W in 1965.
- Java does have Optional - but it's a class like any other, it can itself be null, and nothing forces the rest of the codebase to use it.
-->

---
level: 3
---

## No `null`, Just `Option<T>`

````md magic-move {lines: true}
```rust
// built into the standard library
enum Option<T> {
    None,
    Some(T),
}
```

```rust
fn find_user(id: u32) -> Option<String> {
    if id == 1 { Some("ferris".into()) } else { None }
}

let name = find_user(7);
println!("{}", name.to_uppercase());
//                  ^^^^^^^^^^^^ error[E0599]: no method named `to_uppercase` found for enum `Option<T>`
```

```rust
fn find_user(id: u32) -> Option<String> {
    if id == 1 { Some("ferris".into()) } else { None }
}

match find_user(7) {
    Some(name) => println!("{}", name.to_uppercase()),
    None => println!("no such user"),
}
```
````

<!--
- [start] Option is just an enum like the ones we've been looking at - nothing magic.
- [click] The same code as the Java slide. An Option<String> is a different type to String, so you can't call String methods on it. This is the real compiler error.
  - The compiler forces you to decide what happens when the value is missing - at the point you use it, not three files away.
- [click] So you match on it, or use helpers: unwrap_or for a default, map to transform the inside if it's there.
-->

---
level: 3
---

## Exceptions You Can't See

```java
int doubled(String s) {
  return Integer.parseInt(s) * 2;
}

int n = doubled("abc");
// Runtime: NumberFormatException
```

<!--
- Same story as null: the signature says "returns an int". It doesn't mention that it can blow up.
- parseInt throws NumberFormatException, which extends RuntimeException - unchecked.
- So you only find out by reading the docs, or by a user hitting it.
- Checked exceptions were Java's attempt at this. In practice, most modern libraries and frameworks (Spring, etc.) use unchecked ones, partly because checked exceptions can't cross a lambda in a stream without wrapping.
-->

---
level: 3
---

## No Exceptions, Just `Result<T, E>`

````md magic-move {lines: true}
```rust
// also built into the standard library
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

```rust
fn doubled(s: &str) -> Result<i32, ParseIntError> {
    let n = match s.parse::<i32>() {
        Ok(n) => n,
        Err(e) => return Err(e),
    };
    Ok(n * 2)
}
```

```rust
fn doubled(s: &str) -> Result<i32, ParseIntError> {
    let n: i32 = s.parse()?;
    Ok(n * 2)
}
```
````

<!--
- [start] Result is just another enum. Same idea as Option, but the error case has data.
- [click] Handling it by hand: match, and return early on error. Works, but it's a lot of typing.
- [click] The ? operator does exactly that in one character. Errors are passed up just like exceptions bubble up - but you can see every place it can happen, and it's in the return type.
- Result is marked #[must_use], so if you call doubled and throw away the result, you get a warning.
- You can still crash on purpose: .unwrap() means "I'm sure this is Ok, panic if not". Cloudflare took a big chunk of the internet down in Nov 2025 with an unwrap - so it's greppable, and you should treat it like a code smell in production.
- You'll see this for real in GUILT: every way a command can fail is a variant of an error enum.
-->
