---
layout: post
title:  "Determining non-deterministic code"
post_image: '<img src="\assets\img\2022-03-20-non-determinism\preview.png"  style="width:100%; display: block; margin-left: auto; margin-right: auto;" >'
---

I have spent the last two days at work trying to understand why some test was behaving so weirdly. It should have been passing all the time, but it would randomly fail without a good reason. For a bit we went on by lowering some thresholds to allow some "light failures", but there was no reasons for this to happen, nor any guarantee that it would not fail regardless. When I finally decided to address this I took advantage of the occasion to do a bit of research on the topic, which I am sharing here.

# What is unintentional non-determinism?

A piece of code is deterministic if it produces the same output when given the same input. There are instances where non-determinism is enforced by design - think of a program that just returns the result of a coin flip -, but even when this is the case, the non-deterministic behavior should be well enclosed and handled to make sure that it would not leak in places where it is not supposed to be.

From now on we focus on all the other cases, which is when non-determinism in code is unintentional. It is surprisingly common, usually underestimated and extremely undesirable.

# Why is non-deterministic code bad?

Writing deterministic code is extremely important for the following reasons:
 - the resulting software is more predictable in succeeding and in failing, and both things are good,
 - the software can be tested more reliably, as tests on non-deterministic code produce false positives/negatives,
 - non-determinism spreads easily, it's hard to debug, and it gets harder to find as the project grows, so should be found immediately even if it does not seem to be a priority,
 - identical non-deterministic code might compile into different binaries with different checksums, which leads to various safety and performance issues,
 - non-deterministic code derives from poor coding practices, addressing them will improve future development.

# Possible causes of non-deterministic code

I came across several possible causes for non-determinism, here is a possibly incomplete list, roughly ordered from the most obvious to the least obvious.

#### Time dependence

This is self-explanatory, as the use of any kind of time dependence is inherently non deterministic. Time dependent variables should exist only for their intended usage and be properly disposed at the end of their life cycle.

Using random numbers generators falls also in this category. Note that used random number generators are actually deterministic, meaning that given the same seed they will produce the same numbers. What makes them truly random, when needed, is to use a timestamp as seed. Note that some applications require a non-deterministic behavior, still this should be avoided in tests, or - even better -made deterministic with a constant seed.

#### Uninitialized data

This happens when trying to use a variable whose storage space has been allocated in memory, but a value has not been written neither explicitly nor implicitly through a default. At this point, reading the value will result into reading whatever value was previously stored in memory, which cause the non-determinism. This is generally referred to as UMR (uninitialized memory read), but might also happen in form of ABR (array bound read), i.e. dereferencing an array outside of its bounds or FMR (free memory read), i.e. accessing memory after it has been freed).

```cpp
    int *p = (int*) malloc(sizeof(int));
    printf("*p = %d\n",*p); // uninitialized memory read
```

```cpp
    int a[5];
    ...
    printf("a[5] = %d\n",a[5]);  // array bound read
```

```cpp
    int *p = (int*) malloc(sizeof(int));
    free(p);
    ...
    printf("p = 0x%h\n",p); // free memory read
```

#### Race conditions

This has been, in my experience, the most common cause for non-deterministic code. It happens when concurrent threads access and try to change shared data, as the order in which the thread will perform such operation cannot be guaranteed. This can be avoided with a good multithreading practices, most commonly by avoiding shared resources for different threads whenever possible, and use locks to avoid simultaneous non-atomic operations in all the other cases. Lock will force concurrent threads to line up and wait for their turn when shared resources need to be used.

#### Memory address dependence and unordered iterations

Memory address dependence happens when the address of variables play an active role in the code. As for time dependence, dependence on memory addresses causes non-deterministic behaviors as memory management is not guaranteed to happen consistently across multiple runs of the same executable. A common instance of this kind of non-determinism is caused by sorting pointers by value, for example by storing them into an unordered set.

Non-determinism can also be caused by performing non commutative operations while iterating over a container where total order is not guaranteed. A common example is the addition of floats value in such a hash set or an unordered set (remind that [addition in floating arithmetic is not mathematically associative](/2021/04/25/floating-arithmetic.html)).

