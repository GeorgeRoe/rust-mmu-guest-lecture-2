---
layout: center
---

# Who Am I?

---
level: 2
---

# George Roe

<v-clicks>

- Final year **Level 6 Degree Apprentice**
  - **MMU** - Digital and Technology Solutions Professional
  - **STFC** - Software Engineer, Scientific Computing Department
- First year of the apprenticeship at a small web development company
- Not a Rust expert - just someone who uses it at work and for fun

</v-clicks>

<!--
- Final year now, same course you might be on / thinking about.
- Web dev company first, then moved to STFC - so I've seen both small agency work and big public sector research.
- Set expectations: I'm not on the Rust core team, I'm a working developer who picked it up a couple of years ago.
-->

---
level: 2
---

# How I Found Rust

<v-clicks>

- Kept hearing about it - "most admired language" in every developer survey
- A colleague told me to try it
- So I did [Rustlings](https://rustlings.rust-lang.org/): small broken programs you fix until the compiler is happy

</v-clicks>

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

---
level: 2
---

# My Rust Timeline

<v-clicks>

- <span class="date">2024 Q3</span> Kept hearing about Rust
- <span class="date">2024 Q4</span> Colleague says "learn Rust" - worked through Rustlings
- <span class="date">2025 Q1</span> Started **GUILT** in Python
- <span class="date">2025 Q2</span> Rewrote GUILT in **Rust**
- <span class="date">2025 Q3</span> Gave this lecture for the first time
- <span class="date">2026 Q2</span> Web development experiments with **Leptos**
- <span class="date">2026 Q4</span> Lecture version 2

</v-clicks>

<style>
ul {
  list-style: none;
  border-left: 2px solid #f74c00;
  padding-left: 1.5rem !important;
  margin-left: 0.5rem;
}
li {
  position: relative;
  margin: 0.4rem 0 !important;
}
li::before {
  content: '';
  position: absolute;
  left: -1.95rem;
  top: 0.55rem;
  width: 0.75rem;
  height: 0.75rem;
  border-radius: 9999px;
  background: #f74c00;
}
.date {
  display: inline-block;
  min-width: 6rem;
  font-family: monospace;
  opacity: 0.7;
}
</style>