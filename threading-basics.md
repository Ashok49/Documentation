# C++ Threading – Section 1: Basics of Concurrency

This section covers the foundational concepts of concurrency in C++ using the standard thread support library. Understanding these basics is essential before diving into synchronization, atomics, or advanced patterns.

---

## 1. Process vs Thread
- **Process**: A process is an independent program in execution. It has its own memory space, system resources, and execution environment. Processes are isolated from one another, so communication between them typically requires IPC (Inter-Process Communication) mechanisms.
- **Thread**: A thread is the smallest execution unit within a process. All threads in the same process share the same address space and resources, which makes data sharing easy but requires careful synchronization to avoid race conditions.

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
