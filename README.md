
<p align="center">
  <img src="https://github.com/ayogun/42-project-badges/raw/main/covers/cover-philosophers.png" alt="Philosophers cover" width="100%">
</p>

<h1 align="center">
  <br>
  <a href="https://github.com/SEU_USUARIO/philosophers">
    <img src="https://github.com/ayogun/42-project-badges/raw/main/badges/philosopherse.png" alt="Philosophers badge" width="200">
  </a>
  <br>
  Philosophers
  <br>
</h1>

<h4 align="center">
  A concurrency problem solved using threads and mutexes in <a href="https://www.c-language.org/" target="_blank">C</a>.
</h4>

<p align="center">
  <img src="https://img.shields.io/badge/Final%20Score-100%2F100-00C853?style=for-the-badge" alt="Final Score 100/100">
  <img src="https://img.shields.io/badge/Language-C-00599C?style=for-the-badge&logo=c" alt="Language C">
  <img src="https://img.shields.io/badge/Concept-Multithreading-blueviolet?style=for-the-badge" alt="Multithreading">
  <img src="https://img.shields.io/badge/Synchronization-Mutex-orange?style=for-the-badge" alt="Mutex">
</p>

<p align="center">
  <a href="#about-the-project">About</a> •
  <a href="#core-concepts">Core Concepts</a> •
  <a href="#the-problem">The Problem</a> •
  <a href="#how-to-solve">How To Solve</a> •
  <a href="#verification">Verification</a> •
  <a href="#execution">Execution</a> •
  <a href="#team">Team</a>
</p>

---

<!-- You can add a GIF later -->
<!-- ![demo](./imgs/philosophers_demo.gif) -->

## About the Project

The **Philosophers** project is based on a classic problem discussed since **1965**:  
the **Dining Philosophers Problem**.

We have:

- **X philosophers**
- **X forks**

Each philosopher needs **two forks** to eat.

If a philosopher cannot get both forks, he cannot eat.

This creates a system where:

- philosophers compete for shared resources
- incorrect coordination may lead to **death or deadlock**

There are also special edge cases:

- with **1 philosopher**, death is inevitable (only one fork exists)

---

## Core Concepts

Each philosopher is represented as a **thread**.

A thread can be seen as:

> a worker executing tasks independently

Instead of a single execution flow, we now have multiple workers acting simultaneously.

This introduces new challenges:

- shared memory access
- synchronization
- race conditions

### Data Race Example

Imagine two guards:

- one sets `threat = 0`
- another sets `threat = 1`

If they act at the same time, the final result may overwrite critical information.

This is called a **data race**.

---

## The Problem

Each philosopher follows a loop:

- thinking
- taking forks
- eating
- releasing forks
- sleeping

Then repeats.

However, a major issue appears:

### Deadlock

If all philosophers try to pick forks in the same order:

> everyone grabs one fork → no one gets two → everyone dies

This is called:

```text
deadlock
````

---

## How To Solve

To solve concurrency problems, we introduce **mutexes**.

A mutex acts like a key:

* only one thread can access a resource at a time
* others must wait until it is released

### Types of Protection

**1. Global mutex**

Used for shared states like:

* printing
* death flag

If one philosopher dies:

* a global flag is set
* all threads stop safely

---

**2. Individual mutex**

Each philosopher protects:

* its own state
* last meal time
* internal data

This avoids unnecessary blocking of other threads.

---

### Avoiding Deadlock

To prevent deadlock:

* philosophers alternate fork order
* or use even/odd strategy

Example:

* even philosophers pick left first
* odd philosophers pick right first

This breaks the circular wait condition.

---

## Verification

To validate concurrency correctness, we use:

* `valgrind`
* `fsanitize`
* **helgrind**

### Why Helgrind?

Unlike normal valgrind, **helgrind detects thread issues**:

* race conditions
* improper synchronization
* unsafe memory access between threads

It was the main tool used to verify correctness in this project.

---

## Execution

First, compile:

```bash id="compile_philo"
make
```

Then run:

```bash id="run_philo"
./philo 2 300 200 100
```

### Arguments

```text
[number_of_philosophers] [time_to_die] [time_to_eat] [time_to_sleep] [meals(optional)]
```

---

### Example Output

```bash id="output1"
0 Philo 1 is thinking
0 Philo 1 has taken a fork
0 Philo 1 has taken a fork
0 Philo 1 is eating
200 Philo 1 is sleeping
300 Philo 1 died
```

---

### Edge Case — 1 Philosopher

```bash id="output2"
./philo 1 800 200 200

0 Philo 1 is thinking
0 Philo 1 has taken a fork
800 Philosopher 1 died
```

---

### Timing Insight

A philosopher’s life is reset when eating starts.

If:

* eat = 200ms
* sleep = 100ms
* life = 300ms

Then:

* 100ms (sleep) + 200ms (other eating) = 300ms → death

This demonstrates how **timing precision is critical**.

---

## Team

**Philosophers** is an individual project at **École 42**.

It was fully developed by me.

---

## Final Note

This project is not just about philosophers.

It is about understanding:

* concurrency
* synchronization
* resource sharing
* timing
* system behavior under pressure

It transforms a theoretical problem into a real system where:

* one mistake can cause total failure
* one correct decision can ensure stability

For me, **Philosophers** was the moment where programming stopped being sequential…
and became truly **parallel**.
