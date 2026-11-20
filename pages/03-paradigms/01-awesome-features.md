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

Inheritance (not subtyping) is a horrible anti pattern that forces developers into corners that never needed to exist. Rust gets rid of inheritance entirely, relying on traits and implementations instead.

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

::right::

### Rust - Traits

```rust
trait Greet {
  fn greet(&self);
}

struct Person {
  name: String
}

impl Greet for Person {
  fn greet(&self) {
    println!("Hello, {}!", self.name);
  }
}
```

Rust uses traits to create behavioural contracts that any type (structs, enums, even `i32`) can implement. Dynamic dispatch is possible using `&dyn Greet` or `Box<dyn Greet>`.

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
