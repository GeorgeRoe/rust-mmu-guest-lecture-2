---
level: 2
layout: center
---

# Traits

<!--
- Rust has no classes and no inheritance. At all.
- That sounds like a huge loss if you've spent three years learning OOP in Java. The next few slides show what it uses instead, and why you don't miss it.
-->

---
level: 3
---

## Inheritance In Java

```java
Stack<String> stack = new Stack<>();
stack.push("first");
stack.push("second");

// all inherited from Vector...
stack.add(0, "sneaky");
stack.remove(1);
stack.get(0);
```

<!--
- java.util.Stack has extended Vector since Java 1.0. Its own Javadoc now tells you to use Deque instead.
- Inheritance mixes up two different things: reusing code, and being a kind of something. You can't take one without the other.
- They wanted the first and got both.
- And you can't un-inherit a method without breaking everyone who uses it, so it's stuck like that forever.
- Other classic problems: fragile base class (changing the parent breaks children you've never seen), Penguin extends Bird but can't fly(), deep hierarchies like Swing where you have to read five classes to find where a method actually lives.
-->

---
level: 3
---

## Has-a, Not Is-a

Rust has no inheritance. **Structs** hold data, **traits** describe behaviour, and you reuse code by putting one struct inside another.

```rust
struct Stack<T> {
    items: Vec<T>, // has-a, not is-a
}

impl<T> Stack<T> {
    fn push(&mut self, item: T) {
        self.items.push(item);
    }

    fn pop(&mut self) -> Option<T> {
        self.items.pop()
    }
}
```

A `Stack` *contains* a `Vec`, and only exposes the methods it chooses to.

<!--
- The Stack owns a Vec, but isn't one. Nobody outside can call insert or remove on it.
- Notice pop returns an Option - no exception on an empty stack, from the last section.
- "Favour composition over inheritance" is advice in Java - in Rust it's the only option.
- But if there's no inheritance, how do you write code that works with lots of different types? That's what traits are for.
-->

---
level: 3
---

## Interfaces Are Closed

```java
interface Describe {
  String describe();
}

class Job implements Describe {
  public String describe() { return "a job"; }
}

// String is final, so it can never implement Describe.
// The best you can do is a "utils" class:
class StringUtils {
  static String describe(String s) { return "the text " + s; }
}
```

<!--
- Java interfaces are great - but the list of interfaces is part of the class definition.
- If you didn't write the class, you can't add to that list.
- String is final, so you can't even subclass it.
- Every Java codebase has a StringUtils, DateUtils, CollectionUtils... and you can't pass a String to a method that expects a Describe.
-->

---
level: 3
---

## Traits

A trait is a set of methods a type promises to provide, like an interface. But the `impl` is separate from the type, so you can implement one for **any** type.

````md magic-move {lines: true}
```rust
trait Describe {
    fn name(&self) -> String;
    // default method, can be overridden
    fn describe(&self) -> String {
        format!("This is {}", self.name())
    }
}

struct Job { id: u32 }

impl Describe for Job {
    fn name(&self) -> String { format!("job #{}", self.id) }
}

Job { id: 7 }.describe() // "This is job #7"
```

```rust
trait Describe {
    fn name(&self) -> String;
    // default method, can be overridden
    fn describe(&self) -> String {
        format!("This is {}", self.name())
    }
}

// a type from the standard library!
impl Describe for str {
    fn name(&self) -> String { format!("the text {self:?}") }
}

"ferris".describe() // "This is the text \"ferris\""
```
````

<div v-if="$clicks < 1">

Looks just like a Java interface, default methods included. But the `impl` block sits **next to** the struct, not inside it.

</div>
<div v-else>

So you can add your trait to `str`, `i32`, `Vec`... anything. No more `StringUtils`.

</div>

<!--
- Trait definition looks just like a Java interface, including default methods (Java has had those since Java 8).
- `impl Describe for Job` - the implementation is a separate block. The struct definition doesn't list its traits.
- [click] Because implementations are separate, they can live anywhere - including on types you don't own. Here we've taught the built-in str type a new method.
- The one rule (the "orphan rule"): either the trait or the type must be yours. You can't implement someone else's trait for someone else's type - otherwise two libraries could clash.
-->

---
level: 3
---

## Generics and Trait Bounds

Traits say what a generic type must be able to do. And you get to choose how the method is called.

````md magic-move {lines: true}
```rust
// static dispatch: compiled once per type
fn announce(item: &impl Describe) {
    println!("{}", item.describe());
}

// the same thing, written as a generic
fn announce<T: Describe>(item: &T) {
    println!("{}", item.describe());
}
```

```rust
// static dispatch: compiled once per type
fn announce(item: &impl Describe) {
    println!("{}", item.describe());
}

announce(&3.14);
//       ^^^^^ error[E0277]: the trait bound `{float}: Describe` is not satisfied
```

```rust
// dynamic dispatch: one function, method looked up at runtime
fn announce(item: &dyn Describe) {
    println!("{}", item.describe());
}

// different types in one list
let things: Vec<Box<dyn Describe>> = vec![Box::new(Job { id: 1 }), Box::new(Cat)];
```
````

<div v-if="$clicks < 1">

`T: Describe` means *"any type, as long as it implements `Describe`"*.

</div>
<div v-else-if="$clicks === 1">

Pass something that doesn't, and it fails at **compile time**.

</div>
<div v-else>

`dyn Describe` works like a Java interface. Java *always* looks methods up at runtime. Rust lets you choose, and the default is static.

</div>

<!--
- `impl Describe` and `<T: Describe>` mean the same thing - the first is shorthand.
- [click] The compiler checks the bound when you call it. This is the real error.
- [click] If you need to mix different types in one collection, you use `dyn Describe`. That's a "trait object" - a pointer to the data plus a pointer to a table of methods (a vtable). This is exactly how Java interface calls work.
- Java's JIT can sometimes optimise virtual calls away at runtime. Rust does it at compile time by default - static dispatch means the compiler generates a separate copy of announce for each type and can inline it. That's how Rust gets abstractions with no runtime cost.
-->

---
level: 3
---

## Boilerplate

```java
class Point {
  final int x, y;
  Point(int x, int y) { this.x = x; this.y = y; }

  @Override public boolean equals(Object o) {
    if (!(o instanceof Point p)) return false;
    return x == p.x && y == p.y;
  }
  @Override public int hashCode() { return Objects.hash(x, y); }
  @Override public String toString() { return "Point[x=" + x + ", y=" + y + "]"; }
}
```

<!--
- 12 lines, and the only interesting part is "a point has an x and a y".
- Or your IDE generates it - and then nobody keeps it up to date.
- If equals and hashCode disagree, HashMap and HashSet quietly stop working.
- Credit where it's due, records fixed most of this. But records are always immutable and final, and you can't pick which methods get generated.
-->

---
level: 3
---

## Derive

The compiler can write common trait implementations for you.

```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
struct Point {
    x: i32,
    y: i32,
}

let a = Point { x: 1, y: 2 };
let b = a.clone();

assert_eq!(a, b);
println!("{a:?}"); // Point { x: 1, y: 2 }
```

<v-clicks>

- Works on any struct or enum, and updates automatically when you add a field
- Pick exactly the traits you want, nothing more
- Libraries can provide their own derives, e.g. `Serialize` from **serde** and `Error` from **thiserror**

</v-clicks>

<!--
- derive generates the impl at compile time from the struct definition.
- [clicks]
  - Not deriving Clone is a design decision: it means "copying this is expensive or doesn't make sense".
  - Custom derives are huge in the ecosystem. #[derive(Serialize, Deserialize)] gives you JSON (and lots of other formats) for free. You'll see #[derive(Debug, Error)] in GUILT in a few minutes.
-->

---
level: 3
---

## Everything Is a Trait

Most of Rust's syntax is just calls to trait methods, so your own types can join in.

<div class="grid grid-cols-[1fr_1.4fr] gap-8 items-start">
<div class="trait-table">

| You write | Trait it calls |
|---|---|
| `a + b` | `Add` |
| `a == b` | `PartialEq` |
| `println!("{a}")` | `Display` |
| `for x in a` | `IntoIterator` |
| `a?` | `From` |
| `}` (end of scope) | `Drop` |
| `thread::spawn(...)` | `Send` |

</div>
<div>

````md magic-move {lines: true}
```rust
impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

let c = a + b; // Point { x: 2, y: 4 }
```

```rust
enum ConfigError {
    BadNumber(ParseIntError),
}

impl From<ParseIntError> for ConfigError {
    fn from(e: ParseIntError) -> Self {
        ConfigError::BadNumber(e)
    }
}

fn read_port(s: &str) -> Result<u16, ConfigError> {
    let port = s.parse()?; // converts the error
    Ok(port)
}
```
````

</div>
</div>

<style>
.trait-table table {
  font-size: 0.85rem;
}
.trait-table th,
.trait-table td {
  padding-top: 0.3rem;
  padding-bottom: 0.3rem;
}
</style>

<!--
- Left: a lot of the language is "sugar" over traits from the standard library.
- Right: implement Add and your type works with +. Java doesn't allow operator overloading at all - BigDecimal is a.add(b).
- [click] The important one for real code: `?` calls From to convert the error type. Implement From once, and ? turns a ParseIntError into your own error type automatically. In GUILT, thiserror's #[from] writes these From impls for me.
- The last two rows are a teaser for the next section: Drop is how Rust frees memory, Send is how it stops data races between threads.
-->
