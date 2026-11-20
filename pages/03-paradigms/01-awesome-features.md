---
level: 2
layout: center
---

# Awesome Features

---
level: 3
layout: comparison
---

## Algebraic Enums

Rust's enums are *algebraic data types*, meaning each variant can store data and represent different shapes of a type.

::left::

### Java - Enums are Value Sets

```java
enum Status {
  READY,
  RUNNING,
  FAILED;
}

switch (status) {
  case READY   -> System.out.println("Ready!");
  case RUNNING -> System.out.println("Running!");
  case FAILED  -> System.out.println("Failed!");
}
```

Java enums are a fixed set of constants. They can have fields and methods, but every constant has the same shape. For real ADTs you need sealed interfaces and records (Java 17+).

::right::

### Rust - Enums are ADTs

```rust
enum Status {
  Ready,
  Running(u64),
  Failed(String),
}

match status {
  Status::Ready => println!("Ready!"),
  Status::Running(n) => println!("Running! step: {n}"),
  Status::Failed(msg) => eprintln!("Failed! msg: {msg}"),
}
```

In Rust, you can make **invalid states unrepresentable**.

---
level: 3
layout: comparison
---

## No Inheritance

Inheritance (not subtyping) is a horrible feature that is never better than an alternative solution.

::left::

### Java - Inheritance

```java
// java.util.Stack has extended Vector since Java 1.0
Stack<String> stack = new Stack<>();
stack.push("first");
stack.push("second");

// ...so it inherits every Vector method
stack.add(0, "sneaky"); // insert at the bottom
stack.remove(1);        // remove from the middle
```

Java only allows single class inheritance, but deep hierarchies are still littered across the standard library (e.g., [AWT/Swing](https://www.cs.vu.nl/~eliens/afstuderen/media/@archive/online/archive/documents/java/swing/swingdoc-static/tool_set.html)).

https://stfc-early-careers-code-club.github.io/talks/inheritance-is-evil

::right::

### Rust - Composition

```rust
pub struct Stack<T> {
  items: Vec<T>, // private
}

impl<T> Stack<T> {
  pub fn new() -> Self { Self { items: Vec::new() } }
  pub fn push(&mut self, item: T) { self.items.push(item) }
  pub fn pop(&mut self) -> Option<T> { self.items.pop() }
  pub fn peek(&self) -> Option<&T> { self.items.last() }
}

let mut stack = Stack::new();
stack.push("first");
stack.insert(0, "sneaky"); // error: no method named `insert`
```

With no inheritance, composition is the default. The `Vec` is an implementation detail, and `Stack` only exposes the methods a stack should have.

---
level: 3
layout: comparison
---

## Traits

Traits are behavioural contracts, and unlike Java interfaces, they can be implemented after the type itself.

::left::

### Java - Interfaces

```java
interface Describable { String describe(); }

// your interface on someone else's class: impossible
// (String is final, and you can't edit java.lang)
class DescribableString implements Describable {
  private final String inner;
  DescribableString(String s) { inner = s; }
  public String describe() { return "text: " + inner; }
}

record Person(String name) implements Comparable<Person> {
  public int compareTo(Person o) {
    return name.compareTo(o.name);
  }
}
```

Adding behaviour to a type you don't own means wrapping it in a new class.

::right::

### Rust - Traits

```rust
trait Describe { fn describe(&self) -> String; }

// your trait on someone else's type
impl Describe for i32 {
  fn describe(&self) -> String {
    format!("the number {self}")
  }
}

// someone else's trait on your type
struct Person { name: String }
impl fmt::Display for Person {
  fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
    write!(f, "{}", self.name)
  }
}

42.describe();
println!("{person}");
```

---
level: 3
layout: comparison
---

## Tooling

Rust has some of the best, most cohesive tooling of any modern programming language.

::left::

### Java - Maven

```sh
sudo pacman -S jdk-openjdk
sudo pacman -S maven

mvn archetype:generate \
  -DgroupId=com.example \
  -DartifactId=my-app \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DinteractiveMode=false

cd my-app

# manually edit
nvim pom.xml

mvn test
mvn exec:java -Dexec.mainClass="com.example.App"
```

Maven is often used for enterprise projects, however other projects use gradle for package management.

::right::

### Rust - cargo

```sh
curl --proto '=https' --tlsv1.2 \
  -sSf https://sh.rustup.rs | sh

cargo new my-app

cd my-app

# manually edit
nvim Cargo.toml

# or add a dependency from the command line
cargo add some-package

cargo test
cargo run
```

Almost every Rust project uses plain cargo. Only a few huge codebases (e.g., Chromium, the Linux kernel) use their own build systems.
