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

### Basic Mutex Usage
Mutexes provide `lock()` and `unlock()` methods for manual control, but manual locking is error-prone and not recommended for most use cases:

```cpp
#include <mutex>

std::mutex mtx;
int shared_data = 0;

void unsafe_manual_locking() {
    mtx.lock();
    // Critical section - but what if an exception is thrown?
    shared_data++;
    mtx.unlock(); // May never be called if exception occurs!
}
```

**Important:** Always prefer RAII lock wrappers (covered in the next section) over manual `lock()`/`unlock()` calls to ensure proper cleanup and exception safety.

---

## 3) RAII Lock Wrappers

**RAII (Resource Acquisition Is Initialization)** lock wrappers automatically manage mutex locking and unlocking, following the principle that resource management should be tied to object lifetime. They acquire the mutex lock during construction and release it during destruction, ensuring exception safety and preventing common locking errors.

### Why Use RAII Lock Wrappers?
- **Exception Safety**: Locks are automatically released even if exceptions are thrown
- **Correctness**: Eliminates common bugs like forgetting to unlock or double-unlocking
- **Clarity**: Makes the intent clear and reduces boilerplate code
- **Scope-based**: Lock lifetime is tied to the wrapper object's scope

### std::lock_guard
The simplest and most commonly used RAII wrapper. It locks the mutex on construction and unlocks on destruction with no additional features.

```cpp
#include <mutex>
#include <thread>
#include <iostream>

std::mutex mtx;
int counter = 0;

void increment() {
    std::lock_guard<std::mutex> lock(mtx); // Locks on creation, unlocks on destruction
    ++counter;
    // Lock automatically released when 'lock' goes out of scope
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);
    t1.join();
    t2.join();
    std::cout << "Counter: " << counter << "\n"; // Output: Counter: 2
}
```

**Use `std::lock_guard` when:**
- You need simple, scope-based locking
- You don't need to unlock early or conditionally
- You want the most lightweight option

### std::unique_lock
A more flexible RAII wrapper that supports advanced features like deferred locking, manual unlocking, and transferring ownership.

```cpp
#include <mutex>
#include <thread>
#include <iostream>

std::mutex mtx;
int shared_resource = 0;

void flexible_locking() {
    std::unique_lock<std::mutex> lock(mtx, std::defer_lock); // Don't lock immediately
    
    // Do some work that doesn't need the lock
    
    lock.lock(); // Lock when needed
    shared_resource++;
    lock.unlock(); // Unlock early if desired
    
    // Do more work without holding the lock
    
    lock.lock(); // Can lock again
    shared_resource *= 2;
    // Lock automatically released at end of scope
}

void conditional_locking() {
    std::unique_lock<std::mutex> lock(mtx, std::try_to_lock);
    if (lock.owns_lock()) {
        // Successfully acquired the lock
        shared_resource++;
    } else {
        // Could not acquire lock, do alternative work
        std::cout << "Could not acquire lock\n";
    }
}
```

**Use `std::unique_lock` when:**
- You need to unlock before the end of scope
- You need deferred or conditional locking
- You're working with condition variables (which require `std::unique_lock`)
- You need to transfer lock ownership

### std::scoped_lock (C++17)
Can lock multiple mutexes simultaneously, preventing deadlock when multiple locks are needed.

```cpp
#include <mutex>

std::mutex mtx1, mtx2;
int resource1 = 0, resource2 = 0;

void safe_multi_lock() {
    std::scoped_lock lock(mtx1, mtx2); // Locks both mutexes atomically
    resource1++;
    resource2++;
    // Both locks released automatically
}
```

### Best Practices
1. **Always prefer RAII wrappers** over manual `lock()`/`unlock()`
2. **Use `std::lock_guard`** for simple cases - it's the most efficient
3. **Use `std::unique_lock`** when you need flexibility or working with condition variables
4. **Use `std::scoped_lock`** when locking multiple mutexes
5. **Keep critical sections short** to minimize contention
6. **Consider lock-free alternatives** for high-performance scenarios

