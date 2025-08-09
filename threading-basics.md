# C++ Threading – Section 1: Basics of Concurrency

This section covers the foundational concepts of concurrency in C++ using the standard thread support library. Understanding these basics is essential before diving into synchronization, atomics, or advanced patterns.

---

## 1) Process, Core, and Thread

Understanding the distinction between **process**, **core**, and **thread** is fundamental to working with concurrency in C++.

- **Process**  
  A process is a running instance of a program with its own memory space, resources, and execution context. Processes are isolated from each other by the operating system. Inter-process communication (IPC) is required for processes to exchange data, and this typically has more overhead than communication between threads.

- **Core**  
  A CPU core is a physical processing unit capable of executing instructions. Modern CPUs often have multiple cores, enabling true parallel execution of multiple threads. Some CPUs use **Simultaneous Multithreading (SMT)**, like Intel’s Hyper-Threading, allowing each core to execute multiple hardware threads.

- **Thread**  
  A thread is the smallest schedulable unit of execution. In software terms, a thread runs inside a process and shares the process's memory and resources. Threads are scheduled to run on available CPU cores. Multiple threads can execute concurrently on multiple cores, or be time-sliced on a single core.

**Relationship**  
- A process can have **one or many threads**.  
- Threads run on **cores**, which execute instructions.  
- More cores allow more threads to run truly in parallel (subject to hardware limits), while the OS scheduler time-slices threads if there are more runnable threads than available cores.

**Example**  
On a quad-core CPU with Hyper-Threading (8 hardware threads), the OS can run up to 8 threads simultaneously without time-slicing, but if your program spawns 16 threads, they will share the cores through scheduling.


---

## 2. C++ Thread Support Library
- C++ provides the `<thread>` header for creating and managing threads in a portable and type-safe way.
- Threads are created by passing a callable (function, functor, or lambda) to the `std::thread` constructor.  
- Example:
  ```cpp
  #include <thread>
  #include <iostream>

  void task() {
      std::cout << "Hello from thread!\n";
  }

  int main() {
      std::thread t(task); // Create and start thread
      t.join();            // Wait for thread to finish
  }

### Joining vs Detaching

- **join()**  
  Blocks the calling thread until the target thread finishes execution.  
  This ensures that resources are cleaned up in an orderly fashion and that the thread's results (if any) are fully computed before continuing.  
  **Use Case:** When the calling thread depends on the completion of the spawned thread.

- **detach()**  
  Separates the thread from the `std::thread` object, allowing it to run independently in the background.  
  Once detached, you cannot join the thread later, and you lose the ability to synchronize with its completion.  
  **Use Case:** For background tasks that do not interact with data that might be destroyed before the task completes.

**Important:**  
If a `std::thread` object is destroyed while still joinable (i.e., neither `join()` nor `detach()` has been called), the program will call `std::terminate()` and abort. Always make sure a thread is properly joined or detached before it goes out of scope.

---

### Using Lambdas and Functors

Threads can execute any callable object, including lambdas and functors, not just plain functions.  
This makes it easy to pass state into a thread without relying on global variables.

Example:
```cpp
#include <thread>
#include <iostream>

int main() {
    std::thread t([] {
        std::cout << "Hello from lambda thread!\n";
    });
    t.join();
}


## 3) Thread Lifecycle

A thread in C++ typically moves through the following states during its existence. Understanding these stages helps with debugging, resource management, and performance tuning.

1. **New**  
   The thread object is created and associated with a callable, but execution has not started yet. This state exists only briefly after construction.

2. **Runnable**  
   The thread has been started (via `std::thread` constructor) and is ready to run but is waiting for CPU scheduling. Multiple runnable threads may exist, and the OS decides which gets CPU time.

3. **Running**  
   The thread is actively executing instructions on a CPU core. This state can change rapidly as the scheduler may preempt it to run another thread.

4. **Blocked / Waiting**  
   The thread is temporarily paused, waiting for a resource or event (e.g., I/O completion, acquiring a mutex, receiving a condition variable signal). While blocked, it does not consume CPU time.

5. **Terminated**  
   The thread’s task has completed, and execution has ended. Before destruction of the `std::thread` object, it must be either joined (`join()`) or detached (`detach()`); otherwise, destruction will call `std::terminate()`.

**Key Notes**  
- Threads can move between **Running** and **Blocked/Waiting** multiple times during their lifetime.  
- Proper cleanup (joining or detaching) is critical to prevent resource leaks or abrupt program termination.  
- The exact state tracking is managed by the OS scheduler; C++ does not expose direct state query APIs.

