# 🕸️ rustloom

A multithreaded HTTP web server built from scratch in Rust, following Chapter 21 of [The Rust Programming Language](https://doc.rust-lang.org/book/ch21-00-final-project-a-web-server.html). Demonstrates core systems programming concepts — thread pools, message passing, and graceful shutdown — using nothing but the standard library.

---

## Features 

- **Thread pool** — spawns a fixed number of worker threads to handle concurrent connections
- **Channel-based job dispatch** — uses `mpsc` channels to send closures from the main thread to workers
- **Graceful shutdown** — `Drop` implementation on `ThreadPool` joins all threads cleanly on exit
- **Zero external dependencies** — pure Rust standard library, no crates

---

## Project Structure

```
rustloom/
├── src/
│   ├── main.rs          # TCP listener, routing, request handling
│   └── lib.rs           # ThreadPool, Worker, and Job definitions
├── hello.html           # Served on GET /
├── 404.html             # Served for unrecognised routes
└── Cargo.toml
```

---

## Getting Started

### Prerequisites

- [Rust](https://www.rust-lang.org/tools/install) (stable, 1.70+)

### Run the server

```bash
git clone https://github.com/killuazoldyck680/rustloom.git
cd rustloom
cargo run
```

The server listens on **127.0.0.1:7878** by default. Open your browser and visit:

```
http://127.0.0.1:7878
```

To test the slow-response route (used to verify concurrency):

```
http://127.0.0.1:7878/sleep
```

---

## How It Works

### Thread Pool

`ThreadPool::new(size)` spawns `size` worker threads, each blocking on the receiving end of a shared `mpsc` channel:

```rust
pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: Option<mpsc::Sender<Job>>,
}
```

### Job Dispatch

Incoming TCP connections are wrapped in a `Job` (a `Box<dyn FnOnce() + Send + 'static>`) and sent down the channel. The first idle worker picks it up and executes it.

### Graceful Shutdown

When `ThreadPool` is dropped, it closes the sender channel and calls `thread.join()` on every worker — ensuring all in-flight requests finish before the process exits.

---

## What I Learned

- Structuring a Rust library crate (`lib.rs`) vs. a binary (`main.rs`)
- Sharing state between threads using `Arc<Mutex<T>>`
- Trait objects and the `Send` + `'static` bounds for thread-safe closures
- Implementing `Drop` to encode cleanup logic into the type system
- Why Rust's ownership model makes concurrent code safer by construction

---

## Reference

Built by following **Chapter 21 — Final Project: Building a Multithreaded Web Server** from *The Rust Programming Language* by Steve Klabnik and Carol Nichols.

- 📖 [Read the chapter online](https://doc.rust-lang.org/book/ch21-00-final-project-a-web-server.html)
- 🦀 [The Rust Book](https://doc.rust-lang.org/book/)

---

## License

[MIT](LICENSE)