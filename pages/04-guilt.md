---
layout: center
---

# GUILT: From Python to Rust

---
level: 2
---

# GUILT

**G**reen **U**sage **I**mpact **L**ogging **T**ool

- A CLI that tracks the **carbon emissions of your jobs** on a supercomputer
- HPC users get little to no feedback on their footprint - what tracking exists is usually admin-only
- Kiran Jonathan's idea, developed by me, now run by the **SCD Energy Efficiency Team**
- Presets for **ARCHER2**, **Isambard 3**, **Isambard-AI** and **SCARF**

![GUILT repo](https://opengraph.githubassets.com/1/SCD-Energy-Efficiency-Team/guilt){.w-xs.mt-4}

---
level: 2
layout: comparison
---

# Version 1: Python

I wrote Python like a Java developer.

::left::

```txt
guilt/
├── cli.py
├── commands/
├── dependencies/
│   └── injector.py
├── interfaces/
│   ├── models/
│   ├── repositories/
│   ├── services/
│   └── strategies/
├── mappers/
├── models/
├── parsers/
├── repositories/
├── services/
└── utility/
```

::right::

- **80 files**, 15 of them just interfaces
- A hand-rolled **dependency injector**
- Stuck on **Python 3.9** - the oldest version on the HPCs I targeted
- Installed with `pip install git+https://...` on a login node
- Every JSON field validated by hand - mistakes only found **at runtime**

---
level: 2
---

# Where Did That Error Come From?

````md magic-move {lines: true}
```python
def run(job_ids: list[str]) -> Sequence[LazySlurmAccountingResult]:
  command = ["sacct", "--json", "--jobs", ",".join(job_ids)]

  result = subprocess.run(command, capture_output=True, text=True)

  if result.returncode != 0:
    raise Exception(f"Command failed: {result.stderr.strip()}")

  raw_data = json_reader.expect_dict(json.loads(result.stdout))
  jobs_data = json_reader.ensure_get_list(raw_data, "jobs")

  return [LazySlurmAccountingResult(json_reader.expect_dict(job)) for job in jobs_data]
```

```python
def run(job_ids: list[str]) -> Sequence[LazySlurmAccountingResult]:
  command = ["sacct", "--json", "--jobs", ",".join(job_ids)]

  result = subprocess.run(command, capture_output=True, text=True)
  # FileNotFoundError  - sacct isn't installed
  # UnicodeDecodeError - output isn't valid text

  if result.returncode != 0:
    raise Exception(f"Command failed: {result.stderr.strip()}")
    # the only one I knew about - and it's a bare Exception

  raw_data = json_reader.expect_dict(json.loads(result.stdout))
  # JSONDecodeError    - sacct printed something that isn't JSON
  jobs_data = json_reader.ensure_get_list(raw_data, "jobs")
  # ValueError         - the JSON isn't the shape I expected

  return [LazySlurmAccountingResult(json_reader.expect_dict(job)) for job in jobs_data]
```

```python
# and when I did catch errors...
try:
  with self._path.open("w") as file:
    json.dump(raw_jobs, file, indent=2)
  return True
except Exception:
  return False
  # disk full? no permission? bad data?
  # the caller just gets False
```

```rust
pub fn get_jobs_by_id(
    job_ids: &[String],
) -> Result<Vec<SlurmAccountingResult>, SlurmAccountingCommandError> {
    let output = Command::new("sacct")
        .arg("--jobs")
        .arg(job_ids.join(","))
        .arg("--json")
        .output();

    let stdout = safe_get_stdout(output)?;

    Ok(parse_command_output(&stdout)?)
}
```

```rust
#[derive(Debug, Error)]
pub enum SlurmAccountingCommandError {
    #[error("Failed to execute Slurm accounting command: {0}")]
    SafeCommand(#[from] SafeCommandError),
    #[error("Failed to parse Slurm accounting JSON output: {0}")]
    Json(#[from] StructuredJsonParsingError),
}

#[derive(Debug, Error)]
pub enum SafeCommandError {
    #[error("I/O error: {0}")]
    Io(#[from] std::io::Error),
    #[error("Failed to parse output as UTF-8: {0}")]
    Utf8(#[from] std::str::Utf8Error),
    #[error("Command failed: {0}")]
    CommandFailed(String),
    #[error("Command not found: {0}")]
    CommandNotFound(String),
}
```
````

---
level: 2
layout: comparison
---

# Version 2: Rust

First Rust commit 9th August 2025, **v1.0.0 on 27th August**.

::left::

### What I gained

- **One binary** per architecture - `x86_64` and `aarch64`
- No Python, no pip, no venv:
  ```sh
  curl -sSL .../install-release.sh | sh
  ```
- Bad data is a `Result`, missing data is an `Option`
- Whole classes of bugs caught by the compiler, not by users
- I got to use Rust

::right::

### What it cost

- Fighting the borrow checker for the first week
- Cross-compiling for ARM in CI took some setting up
- More lines of code - Rust is explicit
- Compile times are noticeably slower than "just run it"
