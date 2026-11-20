---
level: 2
layout: center
---

# Ownership & Borrowing

<!--
- This is the feature that makes Rust *Rust*. It's also the one people find hardest at first - so we'll go slowly.
- It answers a question every language has to answer: who is responsible for a piece of data, and when can it be thrown away?
-->

---
level: 3
---

## Shared, Mutable Data

````md magic-move {lines: true}
```java
class IntBox {
  public int value;
  IntBox(int value) { this.value = value; }
}

IntBox a = new IntBox(1);
IntBox b = a; // two references, one object

a.value = 2;

System.out.println(b.value); // 2
```

```java
List<String> names = new ArrayList<>(List.of("a", "b", "c"));

for (String name : names) {
  if (name.equals("a")) {
    names.remove(name);
  }
}
// Runtime: ConcurrentModificationException
```
````

<div v-if="$clicks < 1">

Any number of variables can point to the same object, and any of them can change it. In a big codebase, you can't tell who else is holding a reference.

</div>
<div v-else>

Change a list while looping over it, and Java throws, but only *on a best-effort basis*. Remove `"b"` instead and there's no exception: it just silently skips `"c"`.

</div>

<!--
- First: aliasing. Two variables, one object. Fine in a small example, a nightmare when the "other reference" is in a cache, another class, or another thread.
- [click] Second: the classic. Modify a list while iterating it. Java's ArrayList keeps a modification counter and throws when it notices.
- The ArrayList Javadoc literally says "fail-fast behavior cannot be guaranteed" and is "best-effort".
- Try it: remove "b" instead of "a" and it doesn't throw - the loop just ends early and "c" is never visited. No error, wrong answer.
- In C++ the same code is undefined behaviour - it might crash, might corrupt memory, might seem to work.
- Root cause of both: shared AND mutable at the same time. Rust's whole ownership system is built around never allowing both at once.
-->

---
level: 3
---

## Rule 1: Every Value Has One Owner

Assigning a value or passing it to a function **moves** it. The old variable can't be used any more.

````md magic-move {lines: true}
```rust
let a = String::from("ferris");
let b = a; // ownership moves to b
```

```rust
let a = String::from("ferris");
let b = a; // ownership moves to b

println!("{a}");
//         ^ error[E0382]: borrow of moved value: `a`
```

```rust
fn consume(s: String) { /* ... */ }

let name = String::from("ferris");
consume(name); // moved into the function

println!("{name}");
//         ^^^^ error[E0382]: borrow of moved value: `name`
```

```rust
let a = String::from("ferris");
let b = a.clone(); // an explicit, deep copy
println!("{a} {b}"); // fine

let x = 5;
let y = x; // small types like i32 are just copied
println!("{x} {y}"); // fine
```
````

<div v-if="$clicks < 1">

In Java, `b = a` copies a reference. In Rust, it moves the `String`: `b` owns it now.

</div>
<div v-else-if="$clicks === 1">

Use `a` after the move, and it **won't compile**.

</div>
<div v-else-if="$clicks === 2">

Functions take ownership too, unless you *lend* the value to them (more on that soon).

</div>
<div v-else>

Want two copies? Ask for one with `.clone()`, so expensive copies are visible in the code.

</div>

<!--
- In Java, `b = a` copies a reference. In Rust, `b = a` moves the String - b now owns it, a is empty.
- [click] Use a afterwards and you get this. The real error also says "consider cloning the value if the performance cost is acceptable".
- [click] Same thing with functions - consume now owns the string, and will free it when it returns.
- [click] Two ways out:
  - .clone() - explicit copy. You can grep a codebase for .clone() to find expensive copies. In Java, you can't easily see where objects are shared vs copied.
  - Small, simple types like integers implement the Copy trait (from the last section), so they're just copied, like Java primitives.
-->

---
level: 3
---

## Cleaning Up Is Your Job

```java
Object data = new Object();
// freed whenever the garbage collector decides

try (var file = new FileWriter("out.txt")) {
  file.write("hello");
} // closed here - if you remembered `try`

lock.lock();
try {
  // ...
} finally {
  lock.unlock(); // if you remembered `finally`
}
```

<!--
- The GC handles memory, but at an unpredictable time and with a background thread doing the work. Fine for most apps, a problem for games, trading systems, anything with tight latency.
- For files, sockets and locks you need try-with-resources or finally.
- Forgetting is a resource leak - and the compiler won't tell you.
-->

---
level: 3
---

## Rule 2: When the Owner Goes, the Value Goes

When the owner goes out of scope, the value is **dropped**: its memory is freed and any resources it holds are released.

````md magic-move {lines: true}
```rust
{
    let data = vec![1, 2, 3];
    let mut file = File::create("out.txt")?;
    file.write_all(b"hello")?;
} // file closed, then data freed - always

let guard = mutex.lock().unwrap();
// ...
// unlocked when `guard` goes out of scope
```

```rust
struct Noisy(&'static str);

impl Drop for Noisy {
    fn drop(&mut self) {
        println!("dropping {}", self.0);
    }
}

{
    let _first = Noisy("first");
    let _second = Noisy("second");
} // prints "dropping second", then "dropping first"
```
````

<div v-if="$clicks < 1">

There's exactly one owner, so the compiler knows exactly where to clean up. No garbage collector, no `free()`, and nothing to forget.

</div>
<div v-else>

It's the `Drop` trait from the last section, so you can implement it for your own types.

</div>

<!--
- Because there's exactly one owner, the compiler knows exactly where every value stops being used. It inserts the cleanup there. No GC, and no free() like in C.
- Files close, mutexes unlock, network connections shut - all automatically, even if you return early or hit a ? on an error.
- C++ people call this RAII. Rust just does it for everything.
- [click] Implement Drop for your own types. Values are dropped in reverse order of creation.
-->

---
level: 3
---

## Borrowing

You don't have to give a value away to use it. You can **lend** it with a reference, and everything is immutable unless you say `mut`.

````md magic-move {lines: true}
```rust
fn length(s: &String) -> usize {
    s.len() // borrow to read
}

fn shout(s: &mut String) {
    s.push_str("!"); // borrow to change
}

let mut name = String::from("ferris");
let n = length(&name);
shout(&mut name);

println!("{name} {n}"); // still ours: "ferris! 6"
```

```rust
fn length(s: &String) -> usize {
    s.len() // borrow to read
}

fn shout(s: &mut String) {
    s.push_str("!"); // borrow to change
}

let name = String::from("ferris");
shout(&mut name);
//    ^^^^^^^^^ error[E0596]: cannot borrow `name` as mutable, as it is not declared as mutable
```
````

<div v-if="$clicks < 1">

`&T` lends it to read, `&mut T` lends it to change. A function's signature tells you whether it will change your data.

</div>
<div v-else>

Forget `mut`, and the compiler stops you. Compare that to Java's `final`, which doesn't stop you adding to a `final List`.

</div>

<!--
- `&name` lends name to length. length can read it, and when it returns, the loan is over and we still own name.
- `&mut name` lends it mutably - shout can change it.
- The big win: the signature is documentation the compiler enforces. `fn length(s: &String)` *cannot* modify your string. In Java, any method you pass an object to might change it.
- Same for methods: `&self` vs `&mut self` - you can see which methods change an object just from their signatures.
- [click] Forget `mut` on the variable and the compiler stops you. Immutable is the default.
- Java's final only stops you reassigning the variable - the object it points to can still change.
-->

---
level: 3
---

## Rule 3: Share or Change, Never Both

At any moment, a value can have **either** any number of `&` references **or** exactly one `&mut` reference.

````md magic-move {lines: true}
```rust
let mut names = vec!["a", "b", "c"];

for (i, name) in names.iter().enumerate() {
    if *name == "a" {
        names.remove(i);
    }
}
```

```rust
let mut names = vec!["a", "b", "c"];

for (i, name) in names.iter().enumerate() {
//               ------------ immutable borrow occurs here
    if *name == "a" {
        names.remove(i);
//      ^^^^^^^^^^^^^^^ error[E0502]: cannot borrow `names` as mutable
//                      because it is also borrowed as immutable
    }
}
```

```rust
let mut names = vec!["a", "b", "c"];

names.retain(|name| *name != "a");

println!("{names:?}"); // ["b", "c"]
```
````

<div v-if="$clicks < 1">

The same bug as the Java `ConcurrentModificationException`...

</div>
<div v-else-if="$clicks === 1">

...caught at **compile time**, every time. The loop holds a `&` to `names`, so `remove` can't get a `&mut`.

</div>
<div v-else>

It's all checked while compiling, so it costs nothing when the program runs.

</div>

<!--
- Same bug as the Java slide. The for loop is reading names (a shared borrow) for its whole body.
- [click] remove needs to change it (a mutable borrow). Both at once breaks the rule, so it won't compile. Real error - it even says "this for loop borrows `names` immutably, preventing mutation within its body".
- Unlike Java's check, this isn't best-effort - it can't be missed.
- [click] The idiomatic fix: retain keeps only items matching a condition.
- Why the rule matters: the Vec might reallocate when you change it, which would leave the loop reading freed memory. In C++ that's a crash or a security hole. In Java it's a runtime exception (sometimes). In Rust it's a compile error.
- This checker is the famous "borrow checker".
-->

---
level: 3
---

## Data Races

```java
List<Integer> results = new ArrayList<>();
Runnable work = () -> {
  for (int i = 0; i < 10_000; i++) results.add(i);
};

Thread a = new Thread(work), b = new Thread(work);
a.start(); b.start();
a.join(); b.join();

System.out.println(results.size()); // 20000?
```

<!--
- Two threads each add 10,000 numbers to the same list. You'd expect 20,000.
- ArrayList isn't thread-safe. Two adds at the same time can write to the same slot, or both try to grow the array.
- So you lose items, or crash. And it's non-deterministic - it might pass every test on your laptop and fail in production.
- The fix in Java is to remember to use Collections.synchronizedList, or a lock, or a concurrent collection. Remember being the key word.
-->

---
level: 3
---

## Fearless Concurrency

The same rules that caught the list bug also catch **data races** between threads.

````md magic-move {lines: true}
```rust
let mut results = Vec::new();

let handle = thread::spawn(|| {
    results.push(1);
});

results.push(2);
//  ^^^^^^^ error[E0499]: cannot borrow `results` as mutable more than once at a time
```

```rust
let results = Arc::new(Mutex::new(Vec::new()));

let handle = thread::spawn({
    let results = Arc::clone(&results);
    move || results.lock().unwrap().push(1)
});

results.lock().unwrap().push(2);
```
````

<div v-if="$clicks < 1">

Both threads want a `&mut` to `results` at the same time. That breaks rule 3, so it won't compile.

</div>
<div v-else>

`Arc` shares ownership between threads, and `Mutex` lets only one thread change the data at a time. The `Mutex` *owns* the `Vec`, so you can't touch it without locking.

</div>

<!--
- Same bug as the Java slide, smaller. The spawned thread wants a &mut to results, and so does main. Two &mut at once - same rule as before. Real error.
- [click] The fix: Arc (atomic reference counting) gives shared ownership across threads, Mutex gives exclusive access.
- In Java, a lock and the data it protects are separate - nothing stops you forgetting to lock. In Rust the data is inside the Mutex, so the only way to reach it is lock().
- The Send and Sync traits (from the "everything is a trait" slide) are how the compiler knows what's safe to share between threads.
-->

---
level: 3
---

## Why It's Worth It

<v-clicks>

- No garbage collector, but no manual `free()` either
- No use-after-free, no double free, no dangling pointers
- No data races between threads
- Microsoft and Google have both found that around **70%** of their serious security bugs are memory safety issues
- As Android moved new code to memory-safe languages, memory safety bugs fell from **76%** of its vulnerabilities in 2019 to **24%** in 2024
- The catch: a steep learning curve. Expect to fight the borrow checker for your first few weeks

</v-clicks>

<!--
- [clicks] Summing up the section.
- The 70% figure: Microsoft (2019) and the Chromium project have both published it.
- Android figure: from Google's security blog in September 2024. Memory-safe there means mainly Rust and Kotlin.
- Be honest about the cost: lifetimes ('a annotations) show up eventually, and some designs - graphs, doubly-linked lists, lots of objects pointing at each other - are awkward. But the borrow checker is usually pointing at a real problem.
-->
