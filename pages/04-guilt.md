---
layout: center
---

# GUILT: From Python to Rust

<!--
This is where the last section pays off - a real project, a real rewrite.
Look out for: errors as enums and Result (section 1), and derive and From (section 2).
-->

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

<!--
- Works with SLURM (the job scheduler most HPCs use): wraps sbatch, reads sacct, looks up grid carbon intensity, works out emissions per job.
- Point: users can change their own behaviour (e.g. run when the grid is greener) instead of waiting for admins.
- ARCHER2 = UK national supercomputer, Isambard = Bristol, SCARF = STFC's own cluster.
-->

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

<!--
- Joke lands with this audience: look familiar? interfaces/, repositories/, an injector... this is a Spring Boot app wearing a Python costume.
- It worked, May to August 2025. But:
- Python version floor: HPC system Pythons are old, so no newer language features.
- Install: needs the right Python, pip, git, network access from the login node, ideally a venv. Every user hits a different problem.
- Runtime errors: I wrote helpers like json_reader.expect_dict / ensure_get_list to check types by hand. If I got it wrong, the user found out, not me.
-->

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

<!--
- Real code from both versions of GUILT, trimmed to fit.
- Python version: the signature says it returns a list of jobs. That's it. Nothing about failing.
- [click] But it can fail in at least five ways. I only knew about one of them - the one I wrote - and even that's a bare Exception, so callers can't tell it apart from anything else.
  - FileNotFoundError: run it on a machine without SLURM.
  - UnicodeDecodeError: text=True decodes stdout, which can fail.
  - JSONDecodeError / ValueError: sacct output changes between SLURM versions.
  None of these are in the signature, no tool warns you, you find out when a user opens an issue.
- [click] And the flip side: when I did catch, I caught everything and threw the information away. Very Java "catch (Exception e) {}" energy.
- [click] Rust version: the failure is in the return type. You cannot call this and get a Vec without dealing with the Result. The ? passes errors up.
- [click] And every way it can fail is listed in an enum. The "sacct isn't installed" case Python hid from me is just SafeCommandError::Io - when I run it without SLURM I get:
  "Failed to execute Slurm accounting command: I/O error: No such file or directory (os error 2)"
- thiserror generates the Display impl and the From conversions that make ? work - the same From trait from the traits section, and the same "every way it can fail is a variant" idea from the enums section.
-->

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

<!--
- 18 days from first Rust commit to v1.0.0, with the Python version as a spec.
- aarch64 matters: Isambard 3 Grace and Isambard-AI are ARM machines.
- Install went from "debug your Python environment" to "download a file".
- Be honest about "I just wanted to use Rust" - it's a valid reason, and it paid off.
- TODO: check the "What it cost" list is true to your experience - these were guessed.
- Costs: code is longer partly because it does more now (presets, PDF reports, scripting), not just because Rust is verbose. Don't overstate either way.
-->
