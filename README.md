# Pushing Rust Outside of Its Comfort Zone

The thing I miss most about university, by some margin, is my programming language theory classes. Within the confines of seemingly restrictive languages, some neat, weird, and wonderfully wacky solutions can emerge. Creativity born of constraint feels like a universal phenomenon, but this was recently brought into sharp focus when I came across a compile-time Sudoku checker written solely using [Rust’s type system](https://www.reddit.com/r/rust/comments/1kvpdxz/sudoku_checker_in_rust_type_system/). Naturally, the comments pointed me towards something even more unhinged, an implementation of Doom written entirely with [TypeScript types](https://www.reddit.com/r/programming/comments/1iyqeu7/typescript_types_can_run_doom/).

Despite having written my bachelor’s thesis on Rust and worked with it professionally, I am still regularly impressed and fascinated by the language’s design. Its affine type system and the concept of ownership force you to solve problems in ways that go against the grain of garbage-collected programming languages. Its roots in functional programming, including ad-hoc polymorphism, algebraic data types, and first-class functions, provide an expressive foundation. That is all before we even get to declarative and procedural macros, a system so powerful you can write your own DSLs or automate boilerplate with surgical precision.

For those who haven’t written Rust before, one of its core premises is its **affine type system**.

Consider the following C# code:

```csharp
var x = new SomeType();
var y = process(x);
Console.WriteLine(x);
```

This works fine because `process` receives a **reference** to `x`, leaving `x` itself untouched and accessible afterwards.

In Rust, however, the equivalent code would fail to compile:

```rust
let x = String::from("Hello, world!");
let y = process(x);
println!("{}", x); // error: use of moved value `x`
```

This happens because Rust’s `process` function takes **ownership** of the data previously owned by `x`. When `process` finishes, Rust automatically runs the destructor on that data. The original binding `x` is no longer valid because ownership has been transferred and can only be held by one place at a time. This does not apply to types that opt into "copy semantics", but we will ignore those here.

This is what we mean by affine types, you can use a piece of data **at most once**. Contrast this with languages like C#, where you can use a value many times through references, which can blur the question of who “owns” the data.

Affine types model real-world ownership more naturally. If I have a coin and give it to you, I no longer possess that coin. In C# style semantics, it is as if we both have a claim to the same coin at the same time, which can lead to confusion and bugs like double spending or data races.

Rust’s ownership system enforces this constraint at compile time, preventing many classes of runtime errors related to memory and concurrency.

You get around this limitation by letting scopes borrow data, either immutably or mutably. You can give out as many immutable references as you want, provided no mutable references exist. On the other hand, you can only provide one mutable reference at a time.

Since data races are caused by two things, aliasing and mutability, this type system effectively prevents data races from ever occurring in your program at compile time. This is neat and allows you to avoid some very common, nasty, and hard-to-debug bugs.

The point I am trying to make, though, is that this level of abstraction and altered way of thinking makes traditional software tasks, especially GUI programming, quite tiresome. It can be hard to get the compiler to do what you want without it constantly flagging errors and complaining.

This is why a project like mine has been so rewarding. Rust for the web is not new, but pushing it to achieve tangible and effective results is what makes it exciting.

Yes, Rust offers impressive guarantees over memory safety and “fearless concurrency”, but development with it is not without trade-offs. Criticism about the maturity of its ecosystem, such as “Rust is not ready for front-end”, “There is no GUI story”, and so on, often rings true. But I see those not as limitations, but as challenges, the sort I would have relished as a student and miss now.

I am by no means a front-end artist, so do not judge, but it turns out you can now write some pretty fast WebAssembly applications. The demo I have been working on takes UDP packets from the F1 series of video games, parses them, maps them over a WebSocket, and updates the DOM more than 60 times per second. Throw in a bit of matrix maths to map car coordinates onto a 2D track map, and you have a web-based live telemetry dashboard rendered entirely in the browser. It is all vanilla HTML and CSS on the front-end. Everything else is written in Rust. I built it after running into persistent latency issues with JavaScript and React-based solutions.

Because Rust is a systems language, I have also been able to reuse some of the underlying logic in embedded targets, for example, driving custom rev lights or speedometers on racing wheels.

<iframe width="560" height="315" src="https://www.youtube.com/embed/0DpGJS9nseI" title="Rust WASM F1 Telemetry Dashboard" frameborder="0" allowfullscreen></iframe>

> 💡 P.S. The model library used for parsing game data is [open-source](https://github.com/11bthornton/f1-game-library-models) and also available on [crates.io](https://crates.io/crates/f1-game-library-models).

If you want to learn more, Jon Gjenset is a great educator on YouTube.
