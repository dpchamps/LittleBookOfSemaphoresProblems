# Intro

Synchronization -- refers to the relationships among events. 

*Serialization* - Event A must happen before Event B

*Mutual Exclusion* - Event A and B must not happen at the same time.

### Execution Model

In the simplest model, a processor executes operations one after another in sequence. 

Compilcations: 

1. The computer has multiple processors running in parallel
    1. In this case it's not easy to know if a statement on one processor is executed before a statement on another
2. The computer has a single processor running multiple threads. 
    1. The programmer has no control over when a context switch will occur, therefore cannot know generally when statements in different threads will be executed

From a synchronization standpoint, there's no difference between parallel in multithreaded model.

#### Puzzle

Real world example:

You and Bob live in different cities. How can you tell how ate dinner first? 

Assuming that Bob is willing to follow simple instructions, is there any way you can _guatantee_ that tomorrow you will eat lunch before Bob?

*answer*

Ask Bob to wait for a call from me telling him I've eaten dinner.

### Serialization with Messages

The idea behind the solution to the puzzle above is known as message passing.

Within a thread of execution, you can always determine the order for which things happen. There is generally no way to compose events from different threads.

| a                                             | b                                            |
|-----------------------------------------------|----------------------------------------------|
| 1 Eat breakfast 2 Work 3 Eat lunch 4 Call Bob | 1Eat breakfast 2 Wait for a call 3 Eat lunch |

Considering in these separate threads of execution we have

```none
a1 < a2 < a3 < a4
b1 < b2 < b3
```

There's no way to determine whether or not `a1` or `b1` occured first. But because `b2` depends on `a4`, we're able to reason that the sequence of events is

`b3 > b2 > a4 > a3`

between threads.

For the above case, 

* Bob ate lunch sequentially, because when know the order of events
* Me and Bob ate breakfast concurrently, because there's no way to know.

*Concurrency* - Two events are concurrent if we cannot tell by looking at the program which will happen first.

### Non-determinism

*non-deterministic* - A program is non-deterministic if there's no way to determine what will happen without running it.

Concurrent programs are often non-deterministic.

Concurrency bugs and race conditions are nearly impossible to discover through testing.

### Shared state

Usually there is some state that's shared between threads, which is one way threads can interact with one another.

If threads are not synchronized, it's impossible to tell when either thread will interact with the shared state. e.g. there's no way to determine whether the reader will read the value the writer wrote, an uninitialized variable, or an old value.

One way to solve this is by enforcing a constraint that the reader cannot read until after the writer writes. 

Other ways threads may interact on shared state:

* concurrent writes
    * two or or more writers
* concurrent updates
    * two or more threads performing a read followed by a write

Note: concurrent reads does not generally present a synchonization problem.

#### Concurrent writes

Consider the following:

| Thread A          | Thread B |
|-------------------|----------|
| 1 x = 5  | 1 x = 7  |

| 2 print x | |

What gets printed to standard out depends on the order for which statements are executed.

Puzzle: What path yields output 5 and final value 5?

*answer* 

`a1 < a2 < b1`

Puzzle: What path yields output 7 and final value 7

*answer* 

`a1 < b1 < a2`

Puzzle: Is there a path that yields output 7 and final value 5

*answer*

No, within the execution of Thread A the order `a1 < a2` is guaranteed. Therefore, the possibilities for Thread B to occur must be at some discreet step:

`b1 < a1 < a2` - Value 5, Output 5

`a1 < b1 < a2` - Value 7, Output 7

`a1 < a2 < b1` - Value 7, Output 5


#### Concurrent updates

An update is an operation that reads the value of a variable, computes a new value based on the old value and wrotes the new value.

Consider the following:

| Thread A             | Thread B |
|----------------------|----------|
| 1 count = count + 1  | 1 count = count + 1  |

The problem is masked because the language syntax makes two operations look like one. Really, what we have is something like this:

| Thread A             | Thread B |
|----------------------|----------|
| 1 temp = count   | 1 temp = count  |
| 2 count = temp + 1 | 2 count = temp + 1

Consider then the execution path

`a1 < b1 < b2 < a2`

Assuming that the initial value of count is 0, the final value will be 1. Because `a1` and `b1` reference `temp` when it's in the initial state of `0`. `b2` increments temp to 1 and reassigns it to count. But it is already too late in the local execution of Thread A to get that update, `temp` still has the value `0`, performs an increment and then updates count.

Most modern computers provide an increment instruction that cannot be interrupted. 

*atomic* - An instruction that cannot be interrupted.

The most common solution to synchronization problems is to make a conservative assumption that all updates and writes are not atomic and use synchronization constraints to controll concurrent access to shared variables.

One of the more common constraints is mutual exclusion (a mutex). A mutex guarantees that only one thread accesses a shared variable at a time.

Puzzle:

Suppose that 100 threads run the following program concurrently:

```
for i in range(100):
    temp = count
    count = temp+1
```

What is the largest possible value of `count` after all threads have completed?

Let a thread be denoted Tn, where `n` is the thread number, n > 0 and n < 101

Let `Ri` denote the read operation `temp = count`, where `i` is the current loop execution

Let `Wi` denote the write opertation `count = temp+1`

A full event will be denoted `TnRi, TnWi`, e.g. `T1R2`, which states "Thread 1 read operation in the second iteration of the for loop"

The largest possible value is if each thread ran in series, s.t.

`T1R1 < T1W1 < T1R2 < T1W2 < ... < T1R100 < T1W100 < ... < T100R1 < T100W1 < ... < T100R100 < T100W100`

In which case the final value would be: 100 incrementes 100 times, or 10,000

What is the smallest possible value?

Consider an execution of two threads, where

Let eval(Tn, start, end) = the execution of some thread where start = iteration start and end = iteration end, s.t.

`eval(T1, 0, 3) = T1R1 < T1W1 < T2R2 < T2W2 < T2R3 < T2W3`

Let concurrent(threadNumStart, threadNumEnd, expression) = The concurrent execution of threads starting at threadNumStart and ending at threadNumEnd, s.t.

`concurrent(0, 10, (t) => eval(t, 0, 10)) = the concurrent execution of Threads 1-10, performing 0-10 iterations`

A thread could run in such a way that it reads the lowest possible value of a single increment on it's last iteration, the rest of the program could evaluate and the last operation would be the write of the lowest increment, or 2:

`T1R1 < eval(T2, 1, 99) < T1W1 < T2R2 < concurrent(3, 100) < eval(T1, 2, 100) < T2W2`

In this scenario count = 2


Puzzle: 

You and Bob operate a nuclear reactor that you monitor from remote stations. Both of you are allowed to take lunch breaks, but you must not take lunch at the same time. It doesn't matter who eats lunch first.

What is the minimum number of messages required? 

Before eating lunch, the person who wants to go to lunch must issue a lunch request and wait for a response that says ok. The response acknowledges that the other person is at thier post and not at lunch.

Therefore, the minimum amount of messages required is two: a request and and ack.