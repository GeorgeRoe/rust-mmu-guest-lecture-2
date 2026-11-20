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

Java enums are basically a way to remove "magic numbers/strings". The only way Java can replicate ADTs is with sealed interfaces.

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

In Rust, **invalid states are unrepresentable**.

---
level: 3
layout: comparison
---

## No Inheritance

Inheritance (not subtyping) is a horrible anti pattern that forces developers into corners that never needed to exist. Rust gets rid of inheritance entirely, relying on traits and implementations instead.

::left::

### Java - Inheritance

```java
// B and C inherit from A and both override doSomething

class D extends B, C {
  @Override
  public void doSomething() {
    B.super.doSomething();
    C.super.doSomething();
    // which one do I use?!?!
    // (double diamond problem)
  }
}
```

Although Java now has interfaces, inheritance is littered across the standard library (e.g., [AWT/Swing](https://www.cs.vu.nl/~eliens/afstuderen/media/@archive/online/archive/documents/java/swing/swingdoc-static/tool_set.html)).

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

Rust uses traits to create behavioural contracts on structs. Dynamic dispatch is possible using `Box<dyn Greet>` (heap allocation).

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
nvim cargo.toml

# or install cargo extension
cargo install cargo-edit
cargo add some-package

cargo test
cargo run
```

I have never come across a Rust project that isnt using vanilla cargo.
