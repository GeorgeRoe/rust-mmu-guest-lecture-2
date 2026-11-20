---
layout: center
---

# Web Development with Leptos

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
