# Lab5 - race condtion and critical section

This lab is the homework of [Chapter 26](http://www.cs.wisc.edu/~remzi/OSTEP/threads-intro.pdf)

Format your answers neatly and submit.

## Q1
`./x86.py -p loop.s -t 1 -i 100 -R dx`
---

## Q10

```
./x86.py -p wait-for-me.s -a ax=0,ax=1 -R ax -M 2000

.
.
.

```
---
1. How do the threads behave?
2. What is thread 0 doing?
3. How would changing the interrupt interval (e.g., -i 1000, or perhaps to use random intervals) change the trace outcome?
4. Is the program efficiently using the CPU?
