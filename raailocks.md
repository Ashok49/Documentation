# RAII Lock Wrappers in C++ — GitHub Notes

## TL;DR
**RAII (Resource Acquisition Is Initialization)** ties a lock’s lifetime to an object’s scope: **lock in constructor → unlock in destructor**. This makes critical sections exception-safe and prevents forgotten unlocks.

---

## Core RAII Wrappers (At a Glance)

| Wrapper | Locks When | Unlocks When | Re-lock Later | Timed Locking | Multi-Mutex | CondVar Compatible | Typical Use |
|---|---|---:|---:|---:|---:|---:|---|
| `std::lock_guard<Mutex>` | On construction | On destruction | ✖ | ✖ | ✖ | ✖ | Small, straight critical sections |
| `std::unique_lock<Mutex>` | Configurable (immediate or deferred) | On destruction (if owns) | ✔ | ✔ (`try_lock_for/ until`) | ✖ | ✔ | Complex regions; condition variables; timed attempts |
| `std::scoped_lock<Mutex...>` *(C++17)* | On construction (deadlock-safe for multiple) | On destruction | ✖ | ✖ | ✔ | ✖ | Locking multiple mutexes atomically |
| `std::shared_lock<std::shared_mutex>` *(C++14/17)* | On construction | On destruction | ✖ | ✔ | ✖ | ✖ | Read-mostly workloads (multiple readers) |

> Notes  
> • “Re-lock later” means the wrapper lets you unlock and re-acquire within the same scope (only `unique_lock`).  
> • “CondVar Compatible” refers to `std::condition_variable` which **requires** `std::unique_lock<std::mutex>` (not `lock_guard`).  
> • For read/write, pair `std::shared_lock` (readers) with `std::unique_lock<std::shared_mutex>` (writer).

---

## Locking Tags & Helpers

- **`std::defer_lock`** — Construct a `unique_lock` *without* locking yet; call `.lock()` later or pass to `std::lock`.
- **`std::try_to_lock`** — Try to acquire without blocking (check `.owns_lock()` after).
- **`std::adopt_lock`** — Tell the wrapper “the mutex is already locked by this thread”; wrapper will unlock on destruction. Use only if certain.
- **`std::lock(m1, m2, ...)`** — Deadlock-free acquire of multiple mutexes; often followed by `adopt_lock`.

---

## When to Use What

- **Single mutex, short scope:** `std::lock_guard`
- **Need `condition_variable`:** `std::unique_lock`
- **Need timed/try lock or unlock/relock:** `std::unique_lock`
- **Lock several mutexes:** `std::scoped_lock` or `std::lock` + `adopt_lock`
- **Read-mostly workload:** `std::shared_lock` (readers), `std::unique_lock` (writer)

---

## Best Practices

- Keep critical sections tiny.
- Always prefer RAII over manual lock/unlock.
- Document what each mutex protects.
- Avoid holding locks during I/O or long waits.
- For multiple locks, fix a lock order or use `std::scoped_lock`.
- Prefer `shared_mutex` for read-heavy cases.

---

## Common Pitfalls

- Using `lock_guard` with `condition_variable` — doesn’t work; use `unique_lock`.
- Forgetting `.owns_lock()` after try/timed locks.
- Misusing `adopt_lock` — only adopt if already locked by same thread.
- Copying lock objects — locks are non-copyable; move only if needed.

---

## Glossary

- **Mutex** — Only one thread can hold it at a time.
- **Condition Variable** — Allows threads to wait until notified.
- **Shared Mutex** — Multiple readers, single writer.
- **Deadlock** — Two threads wait on each other forever.

---

## Checklist

- [ ] Scope is minimal.
- [ ] Correct wrapper type chosen.
- [ ] Waiting? → `unique_lock`.
- [ ] Multi-mutex? → `scoped_lock` or ordering.
- [ ] No long ops while locked.
- [ ] Mutex purpose documented.