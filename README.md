# Parallel Pi Calculation with Pthreads and Mutexes

Developed a multithreaded C application to parallelize a Monte Carlo Pi estimation, implementing thread-safe mechanisms to prevent race conditions on shared data.

The program uses the POSIX Threads (Pthreads) API to demonstrate a classic concurrency problem and its solution, contrasting the performance and data sharing models of multithreading (Pthreads) and multiprocessing (`fork()`).

![Language](https://img.shields.io/badge/Language-C-A8B9CC.svg?logo=c&logoColor=white)
![Concurrency](https://img.shields.io/badge/API-Pthreads-00758F.svg)
![Sync](https://img.shields.io/badge/Synchronization-Mutex-red.svg)

---

## 🎯 Project Goal

The goal was to accelerate the Monte Carlo Pi estimation by distributing the computational work across multiple threads. This immediately introduces a **race condition** on the shared counter, and the primary objective is to solve this using a **mutex** to ensure a correct, thread-safe calculation.

### The Monte Carlo Method
Pi is estimated by generating millions of random (x, y) points within a 1x1 square. The ratio of points that fall *inside* a unit circle to the *total* points approximates Pi / 4.

`π ≈ 4 * (points_in_circle / total_points)`

This is "embarrassingly parallel," as each point generation is an independent task, making it perfect for multithreading.

---

## 💥 The Concurrency Challenge: Race Condition

Each of the worker threads needs to update a single global variable, `circle_count`, every time one of its random points lands inside the circle.

* **Critical Section:** The line `circle_count++;` is the critical section.
* **The Problem (Before Mutex):** This operation is not atomic. It's a three-step "read-modify-write" process:
    1.  Thread A **reads** `circle_count` (e.g., 500).
    2.  Thread B **reads** `circle_count` (also 500).
    3.  Thread A increments its local value to 501 and **writes** it back.
    4.  Thread B increments its local value to 501 and **writes** it back.
* **Result:** Two points were found, but the counter only increased by one. The final count is wrong, leading to an inaccurate, lower-than-expected value for Pi.

![!Image: Concurrency problem race condition diagram](.media/concurrency_challenge.png)

## Solution: `pthread_mutex_t`

To solve the race condition, a **mutex** (`pthread_mutex_t`) is used to enforce mutual exclusion. This "lock" ensures that only one thread can enter the critical section at a time.

The logic in the worker thread was transformed as follows:

**Before (Incorrect):**
```c
/* --- RACE CONDITION --- */
if (is_in_circle) {
    circle_count++;
}
```

**After (Thread-Safe):**
```c
/* --- THREAD-SAFE --- */
if (is_in_circle) {
    // Acquire the lock before entering the critical section
  V,
    
    // --- Critical Section ---
    circle_count++;
    // --- End Critical Section ---
    
    // Release the lock so other threads can enter
    pthread_mutex_unlock(&my_lock);
}
```
This guarantees that the "read-modify-write" operation is atomic, resulting in a correct final count and an accurate estimation of Pi.

---

## 🛠️ Key Technical Concepts

* **Pthreads API:**
    * `pthread_create()`: To spawn multiple worker threads, each running the Monte Carlo simulation on a subset of the points.
    * `pthread_join()`: To wait for all worker threads to complete before calculating the final result.
* **Mutex Synchronization:**
    * `pthread_mutex_init()`: To initialize the global lock.
    * `pthread_mutex_lock()` / `pthread_mutex_unlock()`: To protect the critical section (`circle_count++`).
* **Concurrency Models:** The project also included a version using `fork()` to create child processes. This was used to contrast the **multi-process** model (heavyweight, separate memory, no race condition by default) with the **multi-thread** model (lightweight, shared memory, requires manual synchronization).

---

## 🚀 How to Compile & Run

### Prerequisites
* A C compiler (e.g., `gcc`)
* A POSIX-compliant system (Linux, macOS)

### Compile
You must link the Pthreads library using the `-lpthread` flag.

```bash
gcc pi_calculator.c -o pi_calculator -lpthread
```

### Run
The program can be run from the command line:

```bash
# ./executable_name [number_of_threads] [number_of_points]
./pi_calculator 4 10000000
```