---
title: How to Measure Memory Usage in C++
categories: [c++]
tags: [c++]
permalink: /memory-usage-cpp/
date: 2024-01-25 00:00:00
updated: 2024-01-25 00:00:00
---

<!-- toc -->

I've been struggling with calculating the memory usage for a week. Here's the case: I got a program that needs to estimate how much memory it may consume during runtime with some predefined inputs, such as the size of images, etc. The problem is that the program is so complicated that nearly no one understands the code fully. Not to mention, there are lots of parallel codes in the program, scaling the memory usage by the dynamic number of threads. <!--more-->

## getrusage
I really need a way to measure the memory usage of each function, so I googled it and found [Jacob's video](https://youtu.be/Os5cK0H8EOA?si=kQ9j6nD6iHn_b7Pq) talking about `getrusage` function. According to the [document](https://man7.org/linux/man-pages/man2/getrusage.2.html), this function returns mulitple resource usage measures for the calling process, the calling thread or all children of the calling process that have terminated and been waited for. The memory measure is defined in the field `ru_maxrss` of struct `rusage`, which denotes the maximum resident set size (nearly the amount of memory used) in kilobytes. Here's the code I learned from the Jacob's video:
```cpp
#include <iostream>
#include <string>
#include <cstring>
#include <sys/resource.h>

long get_mem_usage()
{
    struct rusage usage;
    int ret;
    ret = getrusage(RUSAGE_SELF, &usage);
    return usage.ru_maxrss; // in KB
}

int main()
{
    long currentMem = get_mem_usage();
    for (int i = 0; i < 100; i++)
    {
        char *p = (char *)malloc(1024 * 100);
        std::memset(p, 1, 1024 * 100);
        std::printf("usage: %ld + %ld\n", currentMem, get_mem_usage() - currentMem);
    }
    return 0;
}
```

The output was quite different from what Jacob got in the video. I got memory usage ranges from 6856KB to 7016KB. I did allocate 100 KB for 100 times, 10000KB totally. So, what happened?

Well, Jacob mentioned some compiler optimizations that the compiler may not allocate memory if it's not used. However, it couldn't explain my results here. I found [this answer](https://unix.stackexchange.com/questions/30940/getrusage-system-call-what-is-maximum-resident-set-size) on stackexchange, which says:

>For instance, if a process allocates a chunk of memory (say 100Mb) and uses it actively (reads/writes to it), its resident set size will be about 100Mb (plus overhead, the code segment, etc.). If after the process then stops using (but doesn't release) that memory for a while, the OS could opt to swap chunks of that memory to swap, to make room for other processes (or cache). The resident set size would then decrease by the amount the kernel swapped out. If the process wakes up and starts re-using that memory, the kernel would re-load the data from swap, and the resident set size would go up again.

Sounds reasonable, isn't it? I changed my code and allocated 1MB for 100 times, 100MB totally. This time the memory usage was about 97.33MB. So, just remember that `getrusage` may be affected by many factors, and the values it returns may not correspond precisely to the theoretical values.
