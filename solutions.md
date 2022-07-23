## Introduction

You and Bob live in different cities. How can you tell how ate dinner first? 

Assuming that Bob is willing to follow simple instructions, is there any way you can _guatantee_ that tomorrow you will eat lunch before Bob?

*answer*

Ask Bob to wait for a call from me telling him I've eaten dinner.

----

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

----

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

---

Puzzle: 

You and Bob operate a nuclear reactor that you monitor from remote stations. Both of you are allowed to take lunch breaks, but you must not take lunch at the same time. It doesn't matter who eats lunch first.

What is the minimum number of messages required? 

Before eating lunch, the person who wants to go to lunch must issue a lunch request and wait for a response that says ok. The response acknowledges that the other person is at thier post and not at lunch.

Therefore, the minimum amount of messages required is two: a request and and ack.