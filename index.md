class: center
name: title
count: false

# Next steps for the Rust type system

.p60[![Ferris](./images/ferris.svg)]

.me[.grey[*by* **Nicholas Matsakis**]]
.left[.citation[View slides at `https://nikomatsakis.github.io/ppl-2023/`]]

---

# Who is this guy

.text300[👋🏽 Hi!]

---

# Me

.p20[![Me](./images/me.png)]

- Senior Principal Engineer at AWS
- Been working on Rust since 2011
- Co-lead of the Rust language design team
- Likes to blog ([babysteps])

[babysteps]: https://smallcultfollowing.com/babysteps/

---

# Rust sprouting up all over

![Foundation sponsors](images/sponsors.jpg)

... and those are just the foundation platinum sponsors.

---

# What are people doing with Rust?

All kinds of things...

- Networking
- Embedded development
- Kernels, kernel modules
- Blockchain
- CLI apps (ripgrep, just, tokei, ...)
- ...and much more

---

# Why work on Rust?

???

Why work on Rust for so long?

Do I just hate garbage collectors?

--

![Garbage collection](./images/gc.gif)

???

No, though I do think they have a tendency to make a mess.

---

# Why work on Rust?

???

I love Rust because I like to see people cool stuff, and Rust is great at that.

--

![Rust web page](./images/rust-web-page.png)

???

It's right there on our page: Rust is a tool for helping everyone to build
reliable and efficient software. And what could be more rewarding than that?

There's a lot packed into this To start with, the phrase _empowering everyone_
refers to the fact that Rust aims to broaden the pool of people doing systems
programming. We want to get past the idea of _systems programming wizards_ and build
an accessible tool that can be used by anybody who needs to build a fast, reliable
program. Interestingly, while a lot of Rust's users have a background in C++, there are
also a number of people who jump to Rust from higher-level languages like Python, JavaScript,
or Go, and we're proud of that.

---

# What's Rust's secret sauce?

???

So, if Rust is a tool for empowerment, it's natural to wonder-- how does it do that?

--

A strict and unforgiving type system!

???

The answer is our **type system**.

--

![Huh.](./images/huh.gif)

???

This can be a bit surprising, I know.

Type systems don't always have a reputation as empowering.

---

# Rust's type system == spinach

![Spinach](./images/spinach.jpg)

.citation[Image credit: [Clyde Robinson](https://flickr.com/photos/crobj/3184283013/)]

???

To many people, a type system feels a bit like spinach. A vegetable that you eat because you know it's healthy, but you don't actually enjoy. I actually agree with the first part: type systems _are_ like spinach....

(True fact: I actually really like spinach, if properly prepared.)

---

# Rust's type system == POPEYE spinach

.p80[![Popeye](./images/popeye.jpg)]

.citation[Image credit: [Mike Mozart](https://flickr.com/photos/jeepersmedia/17331456031/)]

???

...but it's popeye spinach. A good type system gives you a scaffolding and structure that lets you build
programs you can rely on. It lets you go from "this ought to be easy" to code that works with confidence.

---

# Example: Mozilla and Stylo

![Bugzilla 631527, opened 12 years ago, closed 4 years ago](images/bugzilla-631527-intro.png)

???

Let me give you an example, one that comes from Mozilla. Mozilla is the company where Rust was created, of course, and I worked there for a long time. (I'm at AWS now.) Mozilla makes Firefox, the famous browser. A key part of a browser, of course, is the CSS styling system. If you're familiar with CSS, you know that the idea is to traverse the HTML and to determine how big each piece of text should be, whether it should be bold, where it should appear, etc.

--

![Comment 1: should be easy](images/bugzilla-631527-comment-1.png)

.opened[![Arrow](./images/Arrow.png)]

???

So about 12 years ago, it was recognized that this could be done in parallel. In fact, it's an "embarassingly parallel" problem, which means that there is no coordination needed between the threads. Should be easy, though bz. If you know bz, you'll know the guy is a genius. He knows Firefox inside and out. If you read the thread, though, you'll see that not one but two distinct attempts were made in C++ over the years, and neither was successful. Each of them fell prey to various problems: some of them were bugs in the parallel logic, some of them were small variations between windows, mac, and linux that made the code not work, etc. At the end of the day, people felt that the benefits of the patch were not worth the maintenance burden of landing it.

--

![Final comment: done with stylo](images/bugzilla-631527-comment-n.png)

.closed[![Arrow](./images/Arrow.png)]

???

The 3rd and final attempt used Rust, and had the codename stylo. This version landed -- though it too was a non-trivial effort, don't get me wrong! Using Rust helped to give the team confidence that they could not only make the code work, but they could maintain it over time. For one thing, the Rust type system helped them to find bugs and logic errors at compilation time, instead of having to test the heck out of the thing. So this was a clear case where Rust enabled the team, a group of hardened C++ experts, to do something they had not been able to achieve before.

---

# What's Rust's secret sauce? (take 2)

"Rust: making type theory accessible since 2015."<sup>1</sup>

.footnote[
<sup>1</sup> Or trying, anyway.
]

---

# Lesson #1

### "Simple language != Simple to use"

---

name: bcs

# Borrow checker story

```rust
fn test(v: &mut Vec<String>) {
    v.push("Hello".to_string());
    let s: &String = &v[v.len() - 1];
    v.push("World".to_string());
    print(s);
}
```

---

template: bcs

```

+-----+    +------------+      +-----------+
|  v  +--> |data        +----> | "...."    |
|     |    |size      1 |      |           |
+-----+    |capacity  2 |      +-----------+
           +------------+
```

.line1[![Arrow](./images/Arrow.png)]

---

template: bcs

.line2[![Arrow](./images/Arrow.png)]

```

+-----+    +------------+      +-----------+
|  v  +--> |data        +----> | "...."    |
|     |    |size      2 |      | "Hello"   |
+-----+    |capacity  2 |      +-----------+
           +------------+
```

---

template: bcs

.line3[![Arrow](./images/Arrow.png)]

```

+-----+    +------------+      +-----------+
|  v  +--> |data        +----> | "...."    |
|  s  +--+ |size      2 |  +-> | "Hello"   |
+-----+  | |capacity  2 |  |   +-----------+
         | +------------+  |
         |                 |
         +-----------------+
```

---

template: bcs

.line4[![Arrow](./images/Arrow.png)]

```
                           +-----------------+
+-----+    +------------+  |   +-----------+ | +-------------+
|  v  +--> |data        +--+   | XXXXXXX   | +>+ "..."       |
|  s  +--+ |size      3 |  +-> | XXXXXXX   |   | "Hello"     |
+-----+  | |capacity  4 |  |   +-----------+   | "World"     |
         | +------------+  |                   |             |
         |                 |                   +-------------+
         +-----------------+
```

---

template: bcs

.line5[![Arrow](./images/Arrow.png)]

```
                           +-----------------+
+-----+    +------------+  |   +-----------+ | +-------------+
|  v  +--> |data        +--+   | XXXXXXX   | +>+ "..."       |
|  s  +--+ |size      3 |  +-> | XXXXXXX   |   | "Hello"     |
+-----+  | |capacity  4 |  |   +-----------+   | "World"     |
         | +------------+  |                   |             |
         |                 |                   +-------------+
         +-----------------+
```

--

.line5boom[💥]


---

# What happens in Rust?

[Let's try it](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=d24963f6ba269ccb2a5e4a9e595c774f)

---

name: borrows

# Rust rules: _borrows_

```rust
fn test(v: &mut Vec<String>) {
    v.push("Hello".to_string());
    let s: &String = &v[v.len() - 1];
    v.push("World".to_string());
    print(s);
}
```

---

template: borrows

.line3[![Arrow](./images/Arrow.png)]

---

template: borrows

.borrowbracket_left[{]

.borrowbracket_right[}]

`v` is considered *borrowed* for this region of code.

`v` cannot be modified while it is borrowed.

---

template: borrows

.borrowbracket_left[{]

.borrowbracket_right[}]

.errorline4[❌]

```
error[E0502]: cannot borrow `*v` as mutable because it is also borrowed as immutable
 --> src/lib.rs:4:5
  |
3 |     let s: &String = &v[v.len() - 1];
  |                       - immutable borrow occurs here
4 |     v.push("World".to_string());
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^ mutable borrow occurs here
5 |     print(s);
  |           - immutable borrow later used here
```

---

# What happens if we swap the last two lines?

```rust
fn test(v: &mut Vec<String>) {
    v.push("Hello".to_string());
    let s: &String = &v[v.len() - 1];
    print(s);
    v.push("World".to_string());
}
```

.line5[![Arrow](./images/Arrow.png)]

Error or not?

---

# Answer: it depends.

---

# Rust in 2015: Lexical

```rust
fn test(v: &mut Vec<String>) {
    v.push("Hello".to_string());
    let s: &String = &v[v.len() - 1];
    print(s);
    v.push("World".to_string());
}
```

.borrowbracket_left[{]

.borrowbracket_right[}]

.errorline5[❌]

In Rust 2015, you got an error.

Borrows extended until end of the enclosing block.

---

# Rust in 2015: Lexical

```rust
fn test(v: &mut Vec<String>) {
    v.push("Hello".to_string());
    {
        let s: &String = &v[v.len() - 1];
        print(s);
    }
    v.push("World".to_string()); ✅
}
```

.line3[![Arrow](./images/Arrow.png)]

.line6[![Arrow](./images/Arrow.png)]

---

# Why lexical rules?

* Formalism was simpler:
    * Lexical structure was a tree

---

# Rust in 2018: Non-lexical

```rust
fn test(v: &mut Vec<String>) {
    v.push("Hello".to_string());
    let s: &String = &v[v.len() - 1];
    print(s);
    v.push("World".to_string());
}
```

.borrow2018_left[{]

.borrow2018_right[}]

.errorline5[✅]

In 2018, we changed to stretch from first to last use.

---

# All done?

---

name: a-struct

# All done?

```rust
struct Data {
    keys: Vec<Key>,
    stream: Vec<Element>,
}

fn process_data(data: &mut Data) {
    let k = &data.keys[0];
    data.stream.push(Element::new());
    read(k);
}
```

---

template: a-struct

.line2[![Arrow](./images/Arrow.png)]

---

template: a-struct

.line7[![Arrow](./images/Arrow.png)]

---

template: a-struct

.line7[![Arrow](./images/Arrow.png)]

.borrowc_right[}]

`data.keys` is borrowed for this region

---

template: a-struct

.borrowc_right[}]

.line8[![Arrow](./images/Arrow.png)]

`data.keys` is borrowed for this region

Question: is it ok to mutate `data.stream`?


---

template: a-struct

.borrowc_right[}]

.line8[![Arrow](./images/Arrow.png)]

Answer: Yes! Disjoint access allowed:

- borrowing `data.keys`
- writing `data.stream`

---

# Push two elements

```rust
struct Data {
    keys: Vec<Key>,
    stream: Vec<Element>,
}

fn process_data(data: &mut Data) {
    let k = &data.keys[0];
    data.stream.push(Element::new());
    data.stream.push(Element::new());
    read(k);
}
```

.line9[![Arrow](./images/Arrow.png)]

What if we wanted to push two elements? 

Still ok, of course.

---

# Make a closure

```rust
struct Data {
    keys: Vec<Key>,
    stream: Vec<Element>,
}

fn process_data(data: &mut Data) {
    let k = &data.keys[0];
    let c = || data.stream.push(Element::new());
    c();
    c();
    read(k);
}
```

.line8[![Arrow](./images/Arrow.png)]

Does this compile?

---

# It depends on your Rust edition

---

# Closures desugared

a closure expression...

```rust
let c = || data.stream.push(Element::new());
```

---

# Closures desugared

a closure expression becomes a struct borrowing each free variable...

```rust
// let c = || data.stream.push(Element::new());
let c = ClosureStruct { data: &mut data };

struct ClosureStruct<'d> {
    data: &'d mut Data,
}
```

---

# Closures desugared

a closure expression becomes a struct borrowing each free variable...

```rust
// let c = || data.stream.push(Element::new());
let c = ClosureStruct { data: &mut data };

struct ClosureStruct<'d> {
    data: &'d mut Data,
}

impl FnMut<()> for ClosureStruct<'_> {
    fn call_mut(&mut self, args: ()) {
        self.data.stream.push(Element::new())
    }
}
```

...and implementing `FnMut`, which makes it callable

---

# Make a closure

```rust
struct Data {
    keys: Vec<Key>,
    stream: Vec<Element>,
}

fn process_data(data: &mut Data) {
    let k = &data.keys[0];
    let c = || data.stream.push(Element::new());
    c();
    c();
    read(k);
}
```

.line8[![Arrow](./images/Arrow.png)]

---

name: make-a-closure

# Desugared closure example

```rust
struct Data {
    keys: Vec<Key>,
    stream: Vec<Element>,
}

fn process_data(data: &mut Data) {
    let k = &data.keys[0];
    // let c = || data.stream.push(Element::new());
    let c = ClosureStruct { data: &mut data };
    c();
    c();
    read(k);
}
```

---

template: make-a-closure

.line9[![Arrow](./images/Arrow.png)]

---

template: make-a-closure

.line9[![Arrow](./images/Arrow.png)]

Under original desugaring, closure mutates *entire variable* `data`


---

template: make-a-closure

.line7[![Arrow](./images/Arrow.png)]

Under original desugaring, closure mutates *entire variable* `data`

But we have borrowed `data.keys` -- error!

---

# Workaround you used to need

```rust
struct Data {
    keys: Vec<Key>,
    stream: Vec<Element>,
}

fn process_data(data: &mut Data) {
    let k = &data.keys[0];
    let s = &mut data.stream;
    let c = || s.push(Element::new());
    c();
    c();
    read(k);
}
```

.line8[![Arrow](./images/Arrow.png)]

Now accesses are disjoint again.

---

# Closures in Rust 2021

a closure expression...

```rust
let c = || data.stream.push(Element::new());
```

---

# Closures in Rust 2021

a closure expression becomes a struct borrowing each **free path**...

```rust
// let c = || data.stream.push(Element::new());
let c = ClosureStruct { stream: &mut data.stream };
```

---

# Closures in Rust 2021

a closure expression becomes a struct borrowing each **free path**...

```rust
// let c = || data.stream.push(Element::new());
let c = ClosureStruct { stream: &mut data.stream };

struct ClosureStruct<'d> {
    stream: &'d mut Stream,
}

impl FnMut<()> for ClosureStruct<'_> {
    fn call_mut(&mut self, args: ()) {
        self.stream.push(Element::new())
    }
}
```

...and implementing `FnMut`, which makes it callable

---

# No workaround needed in Rust 2021

```rust
struct Data {
    keys: Vec<Key>,
    stream: Vec<Element>,
}

fn process_data(data: &mut Data) {
    let k = &data.keys[0];
    // let c = || data.stream.push(Element::new());
    let c = ClosureStruct { stream: &mut data.stream };
    c();
    c();
    read(k);
}
```

.line9[![Arrow](./images/Arrow.png)]

- borrowing `data.keys`
- writing `data.stream` -- disjoint ✅

---

# "Simple language != Simple to use"

Short history of the Rust borrow checker...

- Rust 2015: Rust 1.0 released
- Rust 2018: Non-lexical borrows
- Rust 2021: Closures capture paths, not variables

--

Each step made the underlying formalism more complex.

--

Each step made the _language_ easier to understand.

--

Rust 2024...?

---

# Rust 2024: Polonius

> Neither a borrower nor a lender be;
> For loan oft loses both itself and friend.
> -- Polonius to his son Laertes, Hamlet

---

name: polonius-code

# Polonius

```rust
fn get_or_insert<K, V>(map: &mut Map<K, V>, key: K) -> &mut V {
    if let Some(value) = get_mut(&mut map, &key) {
        return value;
    }

    insert(&mut map, &key, default_value());
    return &mut map[&key];
}
```

---

template: polonius-code

.line2[![Arrow](./images/Arrow.png)]

`get_mut` returns reference into the map for the given key

```rust
fn get_mut<'m ,K, V>(m: &'m mut Map<K, V>), k: &K -> Option<&'m mut V>
//                       --                                  --
//                      "Return a reference into the map `m`"
```

---

template: polonius-code

.line3[![Arrow](./images/Arrow.png)]

if there is a value for the key, return it to the caller

---

template: polonius-code

.line6[![Arrow](./images/Arrow.png)]

otherwise, add a new value

---

template: polonius-code

.line7[![Arrow](./images/Arrow.png)]

and return that

---

template: polonius-code

.line2[![Arrow](./images/Arrow.png)]

Borrow starts at `get_mut`, and it may _seem_ confined...

.borrowp_right[}]

---

template: polonius-code

.line3[![Arrow](./images/Arrow.png)]

...but the compiler sees that the reference `value` is ultimately going to be returned...

---

template: polonius-code

.line2[![Arrow](./images/Arrow.png)]
.line3[![Arrow](./images/Arrow.png)]

...and `value` is derived from the call to `get_mut`...

---

template: polonius-code

.line2[![Arrow](./images/Arrow.png)]

...so it decides that the borrow from `get_mut` must flow until the end of the function, and hence `insert` is in error.

.borrowp1_right[}]

.errorline6[❌]

---

# Polonius

Remake the analysis to be based on "where reference came from" rather than "where is it used". Avoids this problem.

See more: https://nikomatsakis.github.io/rust-belt-rust-2019/

---

# After Rust 2024? View types?

```rust
struct MyType {
    counter: u32,
    map: Map<K, V>
}

impl MyType {
    fn increment_counter(&mut self) {
        self.counter += 1;
    }
}
```

.line8[![Arrow](./images/Arrow.png)]

`increment_counter` borrows _the whole struct_...

...but it only modifies _one field_ (`counter`)

---

# After Rust 2024? View types?

```rust
struct MyType {
    counter: u32,
    map: Map<K, V>
}

impl MyType {
    fn increment_counter(&mut {counter} self) {
        self.counter += 1;
    }
}
```

.line7[![Arrow](./images/Arrow.png)]

Can we declare that?

.footnote[
Read more at https://smallcultfollowing.com/babysteps//blog/2021/11/05/view-types/
]

---

# After Rust 2024? Interior references?

```rust
struct Pair {
    data: Map<K, V>,
    element: &'self.data mut V, // borrowed from `data` field
}
```

Rust borrows are always confined to a stack frame right now.

Can we store them in a struct?

---

# Lesson #2

### Play four-dimensional chess

Have to think about what happens people edit their programs.

Flexibility, preventing errors is in tension with program evolution.

---

# Rust traits

```rust
trait Datum {
    fn is_significant(&self) -> bool;
}
```

---

# Implementing a trait

```rust
impl Datum for u32 {
    fn is_significant(&self) -> bool {
        self > 10
    }
}
```

---

# Invariant: at most one impl for a given type

Within one library, can check this readily enough.

--

...but across libraries?

---

# Libraries

- Crate A: declares `Datum` trait
- Crate B: implements `Datum` for `u32` and checks `self > 10`
- Crate C: implements `Datum` for `u32` and checks `self != 0`
- Crate D: depends on both B and C

Which definition of `Datum` for `u32` do we use?

---

# Orphan rules (simplified)

Borrow a rule from Haskell...

- Can only implement a trait if...
  - you created the trait _or_
  - you created the type

--

Under this rule:

- Crate A: declares `Datum` trait ✅
- Crate B: implements `Datum` for `u32` ❌ _did not declare `Datum` or `u32`_
- Crate C: implements `Datum` for `u32` ❌ _did not declare `Datum` or `u32`_

---

# Libraries

- Crate A: declares `Datum` trait ✅
- Crate B: declares `DataB`, implements `Datum` for `DataB` ✅
- Crate C: declares `DataC`, implements `Datum` for `DataC` ✅
- Crate D: depends on both A, B, and C ✅

No problem.

---

# Evolution over time

- Crate A v1.0: declares `Datum` trait ✅
- Crate B v1.0: declares `DataA` (but doesn't implement `Datum`)
- Crate D v1.0: depends on A, B ✅

---

# Evolution over time

- Crate A v1.0: declares `Datum` trait ✅
- ~~Crate B v1.0~~: declares `DataB` (but doesn't implement `Datum`)
- Crate B v1.1: declares `DataB` and implements `DataB` for `Datum`
- Crate D v1.0: depends on both A and B ✅ (or is it?)

---

# But wait...

```rust
// Crate D
trait Process { ... }

impl<D: Datum> Process for D { ... }

// In v1.0, DataB did not implement Datum
impl Process for DataB { ... }
```

- Crate D can declare its own trait, say `Process` and give two impls...
  - one for any `Datum`
  - one for `DataB` (which is not a `Datum`)

---

# Wait for it...

- Crate A v1.0: declares `Datum` trait ✅
- Crate B v1.0: declares `DataA` (but doesn't implement `Datum`)
- Crate D v1.0: declares and implements `Process` for:
  - `D: Datum`
  - `DataA` ✅

---

# ...there it is

- Crate A v1.0: declares `Datum` trait ✅
- ~~Crate B v1.0~~: declares `DataB` (but doesn't implement `Datum`)
- Crate B v1.1: declares `DataB` and implements `DataB` for `Datum`
- Crate D v1.0: declares and implements `Process` for:
  - `D: Datum`
  - `DataA` ❌ -- overlaps with the other impl

Implication:

- Adding a new impl would not be a forwards compatible change.

---

# "Rebalancing coherence"

- Crate A v1.0: declares `Datum` trait ✅
- Crate B v1.0: declares `DataA` (but doesn't implement `Datum`)
- Crate D v1.0: declares and implements `Process` for:
  - `D: Datum`
  - `DataA` ❌ -- cannot assume that `DataA` will not implement in the future

Fixed with a kind of "modal logic", shortly before 1.0:

- To prove impls are disjoint, must consider impls that may be added backwards compatibly.

---

# Problem solved...?

Not quite.

---

# Coherence flaw: no way to say no

```rust
trait MyTrait { }

impl<T> MyTrait for T where T: Display { }

impl<E> MyTrait for Vec<E> { } ❌
```

- Vec will _never_ implement `Display`
- But type system doesn't know that!

---

# Coherence flaw: holds back library interop

Consider:

- serde: widely used crate that defines serialization traits
- petgraph: widely used crate to represent graphs

--

Suppose I want serializable graphs?

- either serde must implement its traits for petgraph
- or petgraph must implement serde traits for its types

...but maybe neither crate wants to?

---

# Lifting the coherence rules (active work)

--

<u>Marker traits</u> ([implemented, pending stabilization][mt])

[mt]: https://github.com/rust-lang/rust/issues/29864

```rust
trait MyMarkerTrait { /* nothing here */ }
```

--

<u>Explicit negative impls</u> ([active initiative][ni])

[ni]: https://github.com/rust-lang/negative-impls-initiative

```rust
impl<E> !Display for Vec<E> { }
```

---

# Lifting the coherence rules (speculative)

--

<u>Permit impls in leaf crates</u> ([blog post][clwc])

[clwc]: https://smallcultfollowing.com/babysteps/blog/2022/04/17/coherence-and-crate-level-where-clauses/

```rust
impl serde::Serialize for petgraph::Graph { }
```

--

<u>Permit identical duplicates</u>

```rust
#[derive(serde::Serialize)]
struct petgraph::Graph;
```

---

# Common pattern: extension methods

```rust
// Itertools crate
trait IteratorExt {
    fn cartesian_product(...);
}

impl<I> IteratorExt for I
where
    I: Iterator,
{...}
```

---

# Common pattern: extension methods

```rust
// client crate
use itertools::IteratorExt;

some_iterator
    .cartesian_product(another_iterator)
    .foreach(|(a, b)| ...);
```

Compiler resolves `cartesian_product` to a method from `IteratorExt`

---

# Standard library

```rust
// client crate
use itertools::IteratorExt;

some_iterator
    .cartesian_product(another_iterator)
    .foreach(|(a, b)| ...);
```

What if we want to bring `cartesian_product` into the stdlib for all iterators?

.errorline5[❌]

Ambiguity: which `cartesian_product` method did you want?

---

# Rust 2024?

![RFC 3240](./images/RFC3240.png)

Idea:

- modify method dispatch to consider when method was defined
- allows us to add new methods without affecting existing code

---

# Lesson #3

### Everyone cares about portability...

---

# Lesson #3

### Everyone wants portability...unless they don't

---

Rust distinguishes

- Fixed-size integers (`u32`, `u64`, etc)
- Platform-dependent integers (`usize`)

---

# Strict == good

```rust
fn main() {
    let x: usize = 22;
    let y: u32 = x; ❌ // cannot convert `usize` to `u32`
}
```

.line3[![Arrow](./images/Arrow.png)]

---

name: from-trait

# From trait permits safe interconversions

```rust
trait From<T> {
    fn from(value: T) -> Self;
}

...

let x = TargetType::from(y);
```

---

template: from-trait

.line2[![Arrow](./images/Arrow.png)]

---

template: from-trait

.line7[![Arrow](./images/Arrow.png)]

works if there is a `From` impl, errors if not

---

name: from-impls

# From impls for cases that don't lose data

```rust
impl From<u32> for u64 { }

fn main() {
    let x: u32 = 22;
    let y: u64 = u64::from(x); ✅ // ok
}
```

---

template: from-impls

.line1[![Arrow](./images/Arrow.png)]

---

template: from-impls

.line5[![Arrow](./images/Arrow.png)]

---

# From trait won't let you lose data

```rust
fn main() {
    let x: u64 = 22;
    let y: u32 = u32::from(x); ❌ // no such impl
}
```

.line3[![Arrow](./images/Arrow.png)]

---

# For fallible conversions, there is TryFrom

```rust
trait TryFrom<T> {
    fn try_from(value: T) -> Option<Self>;
}
```

.footnote[NB. Slightly simplified from the actual trait.]

---

# Using TryFrom

```rust
fn main() {
    let x: u64 = 22;
    match u32::try_from(x) {
        Some(y) => /* ok */,
        None => /* overflow */,
    }
}
```

---

# What if I am targeting only 32 bit?

```rust
fn store_results(v: u32) { }
```

You might think, ok, I just use `u32` everywhere

---

# What if I am targeting only 32 bit?

```rust
fn store_results(v: u32) { }

fn process_data(data: &[u32]) {
    let l: usize = data.len();
    store_results(l); ❌ // cannot convert `usize` to `u32`
}
```

...but `usize` leak in!

---

# unwrap is plausible, but risky

```rust
fn store_results(v: u32) { }

fn process_data(data: &[u32]) {
    let l: usize = data.len();
    store_results(u32::try_from(l).unwrap()); ✅
}
```

.line5[![Arrow](./images/Arrow.png)]

Forfeiting compiler checks, back in C land.

---

# Could use conditional compilation

```rust
#[cfg(platform_is_32_bit)]
impl From<usize> for u32
{...}
```

but:

- bad for docs
- bad for CI/CD
- bad for IDEs

---

# What if we leveraged where clauses?

```rust
impl<E> Debug for Vec<E>
where
    E: Debug,
{...}
```

.line3[![Arrow](./images/Arrow.png)]

---

# Where clauses for platform details

```rust
impl From<usize> for u32
where
    Platform: Has32Bits,
{...}
```

.line3[![Arrow](./images/Arrow.png)]

--

Here:

- `Platform` is some global _resource type_ whose precise value is hidden during compilation
- `Has32Bits` is a trait defined in stdlib

---

# "Resource types"

Proposal around this is under active discussion. Uses:

- Alternative to `#[cfg]` for writing portable code
- Portability across global allocators
- Portability across panic handling modes
- Portability across async runtimes
- ...

---

# Summary

- "Simple language != Simple to use"
  - ✅ Non-lexical Lifetimes (Rust 2018)
  - ✅ Closures (Rust 2021)
  - ⚙️ Polonius (Rust 2024)
  - ❓ View types, internal references
- "Play four-dimensional chess"
  - ✅ Limited negative reasoning
  - ⚙️ Marker traits (Rust 2024)
  - ⚙️ Negative impls (Rust 2024)
  - ⚙️ Edition-dependent method dispatch (Rust 2024)
  - ❓ Better coherence rules
- "Everyone Everyone wants portability...unless they don't"
  - ✅ strict usize/u32, `From`/`TryFrom`, etc
  - ⚙️ Where-clause based portability rules (unclear)
