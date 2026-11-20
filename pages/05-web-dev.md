---
layout: center
---

# Web Development with Leptos

<!--
Rust isn't just for CLIs and systems code - it's genuinely nice for web apps too. Leptos is my favourite example.
-->

---
level: 2
layout: comparison
---

# Components Run Once

Leptos uses **signals** instead of a virtual DOM.

::left::

### React

```jsx
function App() {
  const [count, setCount] = useState(0)
  console.log("render") // every click

  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        Click me: {count}
      </button>
      <p>Double count: {count * 2}</p>
    </>
  )
}
```

::right::

### Leptos

```rust
#[component]
fn App() -> impl IntoView {
    let (count, set_count) = signal(0);
    log!("setup"); // once, ever

    view! {
        <button on:click=move |_| *set_count.write() += 1>
            "Click me: " {count}
        </button>
        <p>"Double count: " {move || count.get() * 2}</p>
    }
}
```

<!--
- Most of you will have seen React, so start there.
- React: every setCount re-runs App, builds a new virtual DOM tree, diffs it against the old one, then patches the real DOM.
- Leptos: in the book's words, "the body of the component function is a set-up function that runs once, not a render function that reruns multiple times."
  - signal(0) gives you a getter and a setter.
  - {count} and the move || closure subscribe to the signal. When it changes, just those two text nodes update - no diffing.
- And view! is a macro, so the compiler checks it: misspell `count` or pass the wrong type and it won't build.
- Notice the `move |_|` closures - that's ownership from section 3: the closure takes its own copy of the signal handle so it can outlive the function.
-->

---
level: 2
---

# Your Backend Is a Function Call

```rust {1-11|13-24|all}
#[server]
pub async fn add_todo(title: String) -> Result<(), ServerFnError> {
    let mut conn = db().await?;

    sqlx::query("INSERT INTO todos (title, completed) VALUES ($1, false)")
        .bind(title)
        .execute(&mut conn)
        .await
        .map_err(|e| ServerFnError::ServerError(e.to_string()))?;

    Ok(())
}
#[component]
pub fn AddTodoButton() -> impl IntoView {
    view! {
        <button on:click=move |_| {
            spawn_local(async {
                let _ = add_todo("So much to do!".to_string()).await;
            });
        }>
            "Add Todo"
        </button>
    }
}
```

<!--
- Adapted from the Leptos book's server functions chapter.
- [click] The top half is backend code: a database insert with sqlx.
- [click] The bottom half is frontend code: a button. It calls add_todo like any other async function.
- Leptos compiles your crate twice: once for the server (body included), once to WASM for the browser (body swapped for a fetch to an auto-generated endpoint).
- Caveat worth saying: it's still a public HTTP endpoint - you still need auth etc.
-->

---
level: 2
---

# And It's Fast


| | Leptos | React | Vanilla JS |
|---|---:|---:|---:|
| Swap rows | **12.2 ms** | 86.9 ms | 11.3 ms |
| Select row | **3.2 ms** | 5.5 ms | 2.5 ms |
| Create 10k rows | **262.8 ms** | 344.6 ms | 209.9 ms |
| Memory after 1k rows | **2.7 MB** | 4.4 MB | 1.9 MB |

<p class="text-xs opacity-60 mt-2">

[js-framework-benchmark](https://krausest.github.io/js-framework-benchmark/) - Leptos 0.7 vs React 19.2, same app, Nov 2026

</p>

<!--
- js-framework-benchmark: every framework implements the same table app and is measured the same way. Vanilla JS is the "as fast as it gets" baseline.
- Leptos is much faster than React at some operations (swapping rows ~7x), and uses a lot less memory.
- Bundle size is about the same as React - don't claim WASM is tiny. The Leptos book itself says WASM binaries tend to be larger than the equivalent JS; Leptos just keeps up.
- If asked about Vue/Svelte/Solid: they're in the same league as Leptos on speed, and their bundles are smaller. Leptos' selling point is getting this performance *and* Rust's compiler.
-->
