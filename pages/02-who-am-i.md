---
layout: center
---

# Who Am I?

---
level: 2
---

# George Roe

- Final year **Level 6 Degree Apprentice**
  - **MMU** - Digital and Technology Solutions Professional
  - **STFC** - Software Engineer, Scientific Computing Department
- First year of the apprenticeship at a small web development company
- Not a Rust expert - just someone who uses it at work and for fun

<!--
- Final year now, same course you might be on / thinking about.
- Web dev company first, then moved to STFC - so I've seen both small agency work and big public sector research.
- Set expectations: I'm not on the Rust core team, I'm a working developer who picked it up a couple of years ago.
-->

---
level: 2
---

# How I Found Rust

- Kept hearing about it - "most admired language" in every developer survey
- A colleague told me to try it
- So I did [Rustlings](https://rustlings.rust-lang.org/): small broken programs you fix until the compiler is happy

<v-click>

````md magic-move {lines: true}
```rust
// exercises/01_variables/variables1.rs
fn main() {
    // TODO: Add the missing keyword.
    x = 5;

    println!("x has the value {x}");
}
```

```rust
// exercises/01_variables/variables1.rs
fn main() {
    // TODO: Add the missing keyword.
    x = 5;
//  ^ error[E0425]: cannot find value `x` in this scope
//    help: you might have meant to introduce a new binding: `let x = 5;`
    println!("x has the value {x}");
}
```

```rust
// exercises/01_variables/variables1.rs
fn main() {
    let x = 5; // compiles!

    println!("x has the value {x}");
}
```
````

</v-click>

<!--
- Rustlings is ~90 tiny exercises, run in your terminal, it watches the files and re-checks as you edit.
- [click] First exercise: assign to x.
- [click] The compiler doesn't just say "error" - it tells you what you probably meant.
- [click] Fix it, move on. That feedback loop - compiler telling you exactly what's wrong - is the theme of this whole talk, and it comes back in the AI section.
-->