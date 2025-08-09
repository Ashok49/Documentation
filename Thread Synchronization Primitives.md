# C++ Threading — Section 2: Thread Synchronization Primitives

When multiple threads access shared data concurrently, **synchronization** ensures correctness and prevents undefined behavior. Without proper synchronization, race conditions, data corruption, and subtle bugs can occur. This section covers the main synchronization tools in the C++ Standard Library.

---

## 1) Why Synchronization is Needed
- **Race Conditions**  
  Occur when two or more threads access shared data and try to modify it simultaneously without coordination. The program’s outcome depends on the unpredictable timing of thread execution.
- **Data Corruption**  
  Without synchronization, partial updates from one thread can overwrite or conflict with updates from another.
- **Undefined Behavior**  
  C++ does not guarantee safety for concurrent writes or read–write access to shared data unless synchronization mechanisms are used.

---

## 2) Mutexes (Mutual Exclusion Locks)
A **mutex** ensures that only one thread at a time can access a critical section (a block of code that manipulates shared data).

### Types of Mutexes
- **`std::mutex`**  
  Basic mutual exclusion lock, non-recursive.
- **`std::recursive_mutex`**  
  Allows the same thread to acquire the lock multiple times. Useful in certain recursive algorithms but can hide design issues.
- **`std::timed_mutex` / `std::recursive_timed_mutex`**  
  Like `std::mutex`, but supports `try_lock_for` and `try_lock_until` for timeout-based locking.
- **`std::shared_mutex` (C++17)**  
  Supports multiple readers or one writer at a time.

Example:
```cpp
#include <mutex>
#include <thread>
#include <iostream>

std::mutex mtx;
int counter = 0;

void increment() {
    std::lock_guard<std::mutex> lock(mtx); // Locks on creation, unlocks on destruction
    ++counter;
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);
    t1.join();
    t2.join();
    std::cout << "Counter: " << counter << "\n";
} ```

---

## 3) RAII Lock Wrappers

RAII locks release the mutex automatically when they go out of scope, preventing leaks and hard-to-debug hangs on exception paths. Prefer them over manual `lock()`/`unlock()` for correctness and clarity.

- **`std::lock_guard<std::mutex>`**  
  Lightweight scope guard: locks on construction, unlocks on destruction. No early unlock or timed features—use when you just need a simple critical section.
  ```cpp
  std::mutex m;
  void f() {
      std::lock_guard<std::mutex> g(m);  // lock
      // critical section
  }                                       // auto-unlock here

