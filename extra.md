# Example: enums (algebraic data types)

```rust
enum Result<R, E> {
    Ok(R),
    Err(E),
}
```

---

# Matching

```rust
fn load_data() -> Result<Data, Error> { ... }

match load_data() {
    Ok(v) => /* success */,
    Err(e) => /* error */,
}
```

.line3[![Arrow](./images/Arrow.png)]

---

# Rust requires exhaustive matching

```rust
fn load_data() -> Result<Data, Error> { ... }

match load_data() {
    Ok(v) => /* success */,
}
```

.errorline4[❌]

Need an `Err` variant to compile.

Or you could write `_`

---

# Enumerating error cases

```rust
enum Error {
    NoFile,
    BadFormat,
}

fn load_data() -> Result<Data, Error> { ... }
```

---

# Enumerating error cases

```rust
fn process_data() -> Result<Process, &str> {
    match load_data() {
        Ok(v) => Ok(process(v)),
        Err(Error::NoFile) => Err("no file"),
        Err(Error::BadFormat) => Err("bad format"),
    }
}
```

---

# Adding a new case

```rust
enum Error {
    NoFile,
    BadFormat,
    TemporaryError,
}
```

---

# Enumerating error cases

```rust
fn process_data() -> Result<Process, &str> {
    match load_data() {
        Ok(v) => Ok(process(v)),
        Err(Error::NoFile) => Err("no file"),
        Err(Error::BadFormat) => Err("bad format"),
    }
}
```

---

# Enumerating error cases

```rust
fn process_data() -> Result<Process, &str> {
    match load_data() {
        Ok(v) => Ok(process(v)),
        Err(Error::NoFile) => Err("no file"),
        Err(Error::BadFormat) => Err("bad format"),
    }
}
```

---
