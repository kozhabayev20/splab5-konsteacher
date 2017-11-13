# Lab5 - race condition and critical section

This lab is the homework of [Chapter 26](http://www.cs.wisc.edu/~remzi/OSTEP/threads-intro.pdf)

Format your answers neatly and submit.
### Q1

```
./x86.py -p loop.s -t 1 -i 100 -R dx
...
   dx          Thread 0         
    0   
   -1   1000 sub  $1,%dx
   -1   1001 test $0,%dx
   -1   1002 jgte .top
   -1   1003 halt
```

---
1. __Can you figure out what the value of `%dx` will be during the run?__  
_`%dx` will be `-1`._

### Q2
```
./x86.py -p loop.s -t 2 -i 100 -a dx=3,dx=3 -R dx -c
...
   dx          Thread 0                Thread 1         
    3   
    2   1000 sub  $1,%dx
    2   1001 test $0,%dx
    2   1002 jgte .top
    1   1000 sub  $1,%dx
    1   1001 test $0,%dx
    1   1002 jgte .top
    0   1000 sub  $1,%dx
    0   1001 test $0,%dx
    0   1002 jgte .top
   -1   1000 sub  $1,%dx
   -1   1001 test $0,%dx
   -1   1002 jgte .top
   -1   1003 halt
    3   ----- Halt;Switch -----  ----- Halt;Switch -----  
    2                            1000 sub  $1,%dx
    2                            1001 test $0,%dx
    2                            1002 jgte .top
    1                            1000 sub  $1,%dx
    1                            1001 test $0,%dx
    1                            1002 jgte .top
    0                            1000 sub  $1,%dx
    0                            1001 test $0,%dx
    0                            1002 jgte .top
   -1                            1000 sub  $1,%dx
   -1                            1001 test $0,%dx
   -1                            1002 jgte .top
   -1                            1003 halt
```
---
1. __What values will `%dx` see? Run with the `-c` flag to see the answers.__  

_`%dx` decrements by 1 after each iteration, until it get equal `-1`._

2. __Does the presence of multiple threads affect anything about your calculations? Is there a race condition in this code?__ 


_Multiple threads don't affect our calculations because there is no race condtion. The threads are executed without interleavings because the threads complete before an interrupt occurs._

### Q3

```
./x86.py -p loop.s -t 2 -i 3 -r -a dx=3,dx=3 -R dx
...
   dx          Thread 0                Thread 1         
    3   
    2   1000 sub  $1,%dx
    2   1001 test $0,%dx
    2   1002 jgte .top
    3   ------ Interrupt ------  ------ Interrupt ------  
    2                            1000 sub  $1,%dx
    2                            1001 test $0,%dx
    2                            1002 jgte .top
    2   ------ Interrupt ------  ------ Interrupt ------  
    1   1000 sub  $1,%dx
    1   1001 test $0,%dx
    1   1002 jgte .top
    2   ------ Interrupt ------  ------ Interrupt ------  
    1                            1000 sub  $1,%dx
    1                            1001 test $0,%dx
    1                            1002 jgte .top
    1   ------ Interrupt ------  ------ Interrupt ------  
    0   1000 sub  $1,%dx
    0   1001 test $0,%dx
    0   1002 jgte .top
    1   ------ Interrupt ------  ------ Interrupt ------  
    0                            1000 sub  $1,%dx
    0                            1001 test $0,%dx
    0                            1002 jgte .top
    0   ------ Interrupt ------  ------ Interrupt ------  
   -1   1000 sub  $1,%dx
   -1   1001 test $0,%dx
   -1   1002 jgte .top
    0   ------ Interrupt ------  ------ Interrupt ------  
   -1                            1000 sub  $1,%dx
   -1                            1001 test $0,%dx
   -1                            1002 jgte .top
   -1   ------ Interrupt ------  ------ Interrupt ------  
   -1   1003 halt
   -1   ----- Halt;Switch -----  ----- Halt;Switch -----  
   -1                            1003 halt
```

This makes the interrupt interval quite small and random; use different seeds with `-s` to see different interleavings.

---
1. __Does the frequency of interruption change anything about this program?__   
_Frequency affect the interleavings, i.e. the frequency of interleavings, but not the result, because the two threads don't have_  ___critical sections.___

### Q4
```
./x86.py -p looping-race-nolock.s -t 1 -M 2000
ARG seed None
ARG numthreads 1
ARG program looping-race-nolock.s
ARG interrupt frequency 50
ARG interrupt randomness False
ARG argv 
ARG load address 1000
ARG memsize 128
ARG memtrace 2000
ARG regtrace 
ARG cctrace False
ARG printstats False
ARG verbose False

 2000          Thread 0         
    0   
    0   1000 mov 2000, %ax
    0   1001 add $1, %ax
    1   1002 mov %ax, 2000
    1   1003 sub  $1, %bx
    1   1004 test $0, %bx
    1   1005 jgt .top
    1   1006 halt

```
__1. What value is found in x (i.e., at memory address 2000) throughout the run?__

_After running the source code of looping-race-nolock.s, the value was 1_

### Q5
```
./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 0
ARG seed None
ARG numthreads 2
ARG program looping-race-nolock.s
ARG interrupt frequency 50
ARG interrupt randomness False
ARG argv bx=3
ARG load address 1000
ARG memsize 128
ARG memtrace 2000
ARG regtrace 
ARG cctrace False
ARG printstats False
ARG verbose False

 2000          Thread 0                Thread 1         
    0   
    0   1000 mov 2000, %ax
    0   1001 add $1, %ax
    1   1002 mov %ax, 2000
    1   1003 sub  $1, %bx
    1   1004 test $0, %bx
    1   1005 jgt .top
    1   1000 mov 2000, %ax
    1   1001 add $1, %ax
    2   1002 mov %ax, 2000
    2   1003 sub  $1, %bx
    2   1004 test $0, %bx
    2   1005 jgt .top
    2   1000 mov 2000, %ax
    2   1001 add $1, %ax
    3   1002 mov %ax, 2000
    3   1003 sub  $1, %bx
    3   1004 test $0, %bx
    3   1005 jgt .top
    3   1006 halt
    3   ----- Halt;Switch -----  ----- Halt;Switch -----  
    3                            1000 mov 2000, %ax
    3                            1001 add $1, %ax
    4                            1002 mov %ax, 2000
    4                            1003 sub  $1, %bx
    4                            1004 test $0, %bx
    4                            1005 jgt .top
    4                            1000 mov 2000, %ax
    4                            1001 add $1, %ax
    5                            1002 mov %ax, 2000
    5                            1003 sub  $1, %bx
    5                            1004 test $0, %bx
    5                            1005 jgt .top
    5                            1000 mov 2000, %ax
    5                            1001 add $1, %ax
    6                            1002 mov %ax, 2000
    6                            1003 sub  $1, %bx
    6                            1004 test $0, %bx
    6                            1005 jgt .top
    6                            1006 halt
```
__1. Do you understand why the code in each thread loops three times? What will the final value of x be?__
_Two the threads loop 3 times and each iteration increments the value at address 2000 by 1, therefore the value is 6_

### Q6
```
./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 0
ARG seed 0
ARG numthreads 2
ARG program looping-race-nolock.s
ARG interrupt frequency 4
ARG interrupt randomness True
ARG argv 
ARG load address 1000
ARG memsize 128
ARG memtrace 2000
ARG regtrace 
ARG cctrace False
ARG printstats False
ARG verbose False

 2000          Thread 0                Thread 1         
    0   
    0   1000 mov 2000, %ax
    0   1001 add $1, %ax
    1   1002 mov %ax, 2000
    1   1003 sub  $1, %bx
    1   ------ Interrupt ------  ------ Interrupt ------  
    1                            1000 mov 2000, %ax
    1                            1001 add $1, %ax
    2                            1002 mov %ax, 2000
    2                            1003 sub  $1, %bx
    2   ------ Interrupt ------  ------ Interrupt ------  
    2   1004 test $0, %bx
    2   1005 jgt .top
    2   ------ Interrupt ------  ------ Interrupt ------  
    2                            1004 test $0, %bx
    2                            1005 jgt .top
    2   ------ Interrupt ------  ------ Interrupt ------  
    2   1006 halt
    2   ----- Halt;Switch -----  ----- Halt;Switch -----  
    2                            1006 halt

./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 0
ARG seed 5
ARG numthreads 2
ARG program looping-race-nolock.s
ARG interrupt frequency 4
ARG interrupt randomness True
ARG argv 
ARG load address 1000
ARG memsize 128
ARG memtrace 2000
ARG regtrace 
ARG cctrace False
ARG printstats False
ARG verbose False

 2000          Thread 0                Thread 1         
    0   
    0   1000 mov 2000, %ax
    0   1001 add $1, %ax
    1   1002 mov %ax, 2000
    1   ------ Interrupt ------  ------ Interrupt ------  
    1                            1000 mov 2000, %ax
    1                            1001 add $1, %ax
    2                            1002 mov %ax, 2000
    2   ------ Interrupt ------  ------ Interrupt ------  
    2   1003 sub  $1, %bx
    2   1004 test $0, %bx
    2   1005 jgt .top
    2   1006 halt
    2   ----- Halt;Switch -----  ----- Halt;Switch -----  
    2   ------ Interrupt ------  ------ Interrupt ------  
    2                            1003 sub  $1, %bx
    2                            1004 test $0, %bx
    2                            1005 jgt .t
```
__1. Can you tell, just by looking at the thread interleaving, what the final value of x will be?__
__2. Where can it safely occur?__
__3. Where does an interrupt cause trouble?__
__4. In other words, where is the critical section exactly?__
Answer
_Yes, if you step through the interleaving the value was 2. The critical section that must not be interrupted is between sub $1, %ax and test $0, %bx. This is because the sub $1, %ax statement will decrement the ax register and if the following test $0, %bx is interrupted then the other thread could also run sub $1, %bx, and that means that whole iteration will be skipped (bx went from 2 to 0, for instance, decremented twice, without being compared)._
### Q7
```
./x86.py -p looping-race-nolock.s -a bx=1 -t 2 -M 2000 -i 1
ARG seed None
ARG numthreads 2
ARG program looping-race-nolock.s
ARG interrupt frequency 1
ARG interrupt randomness False
ARG argv bx=1
ARG load address 1000
ARG memsize 128
ARG memtrace 2000
ARG regtrace 
ARG cctrace False
ARG printstats False
ARG verbose False

 2000          Thread 0                Thread 1         
    0   
    0   1000 mov 2000, %ax
    0   ------ Interrupt ------  ------ Interrupt ------  
    0                            1000 mov 2000, %ax
    0   ------ Interrupt ------  ------ Interrupt ------  
    0   1001 add $1, %ax
    0   ------ Interrupt ------  ------ Interrupt ------  
    0                            1001 add $1, %ax
    0   ------ Interrupt ------  ------ Interrupt ------  
    1   1002 mov %ax, 2000
    1   ------ Interrupt ------  ------ Interrupt ------  
    1                            1002 mov %ax, 2000
    1   ------ Interrupt ------  ------ Interrupt ------  
    1   1003 sub  $1, %bx
    1   ------ Interrupt ------  ------ Interrupt ------  
    1                            1003 sub  $1, %bx
    1   ------ Interrupt ------  ------ Interrupt ------  
    1   1004 test $0, %bx
    1   ------ Interrupt ------  ------ Interrupt ------  
    1                            1004 test $0, %bx
    1   ------ Interrupt ------  ------ Interrupt ------  
    1   1005 jgt .top
    1   ------ Interrupt ------  ------ Interrupt ------  
    1                            1005 jgt .top
    1   ------ Interrupt ------  ------ Interrupt ------  
    1   1006 halt
    1   ----- Halt;Switch -----  ----- Halt;Switch -----  
    1   ------ Interrupt ------  ------ Interrupt ------  
    1                            1006 halt

./x86.py -p looping-race-nolock.s -a bx=1 -t 2 -M 2000 -i 3
ARG seed None
ARG numthreads 2
ARG program looping-race-nolock.s
ARG interrupt frequency 3
ARG interrupt randomness False
ARG argv bx=1
ARG load address 1000
ARG memsize 128
ARG memtrace 2000
ARG regtrace 
ARG cctrace False
ARG printstats False
ARG verbose False

 2000          Thread 0                Thread 1         
    0   
    0   1000 mov 2000, %ax
    0   1001 add $1, %ax
    1   1002 mov %ax, 2000
    1   ------ Interrupt ------  ------ Interrupt ------  
    1                            1000 mov 2000, %ax
    1                            1001 add $1, %ax
    2                            1002 mov %ax, 2000
    2   ------ Interrupt ------  ------ Interrupt ------  
    2   1003 sub  $1, %bx
    2   1004 test $0, %bx
    2   1005 jgt .top
    2   ------ Interrupt ------  ------ Interrupt ------  
    2                            1003 sub  $1, %bx
    2                            1004 test $0, %bx
    2                            1005 jgt .top
    2   ------ Interrupt ------  ------ Interrupt ------  
    2   1006 halt
    2   ----- Halt;Switch -----  ----- Halt;Switch -----  
    2                            1006 halt

```
__See if you can guess what the final value of the shared variable x will be. What about when you change -i 2, -i 3, etc.? For which interrupt intervals does the program give the “correct” final answer?__
_The final value of 2000 is 1 and if greater than or equal to 3 given the correct answer of 2_

### Q8
```
./x86.py -p looping-race-nolock.s -a bx=100 -t 2 -M 2000

  163                            1003 sub  $1, %bx
  163                            1004 test $0, %bx
  163                            1005 jgt .top
  163                            1000 mov 2000, %ax
  163                            1001 add $1, %ax
  164                            1002 mov %ax, 2000
  164                            1003 sub  $1, %bx
  164                            1004 test $0, %bx
  164                            1005 jgt .top
  164   ------ Interrupt ------  ------ Interrupt ------  
  164   1006 halt
  164   ----- Halt;Switch -----  ----- Halt;Switch -----  
  164                            1006 halt

./x86.py -p looping-race-nolock.s -a bx=100 -t 2 -M 2000 -i 12
  199                            1003 sub  $1, %bx
  199                            1004 test $0, %bx
  199                            1005 jgt .top
  199                            1000 mov 2000, %ax
  199                            1001 add $1, %ax
  200                            1002 mov %ax, 2000
  200                            1003 sub  $1, %bx
  200                            1004 test $0, %bx
  200                            1005 jgt .top
  200   ------ Interrupt ------  ------ Interrupt ------  
  200   1006 halt
  200   ----- Halt;Switch -----  ----- Halt;Switch -----  
  200                            1006 halt

```
__Now run the same code for more loops (e.g.,set -a bx=100). What interrupt intervals, set with the -i flag, lead to a “correct” outcome? Which intervals lead to surprising results?__
_3, 6, 9, 12, etc all give the correct answer of 200.All others give some variation from 100 to less than 200_

### Q9
```
./x86.py -p wait-for-me.s -a ax=1,ax=0 -R ax -M 2000
ARG seed None
ARG numthreads 2
ARG program wait-for-me.s
ARG interrupt frequency 50
ARG interrupt randomness False
ARG argv ax=1,ax=0
ARG load address 1000
ARG memsize 128
ARG memtrace 2000
ARG regtrace ax
ARG cctrace False
ARG printstats False
ARG verbose False

 2000      ax          Thread 0                Thread 1         
    0       1   
    0       1   1000 test $1, %ax
    0       1   1001 je .signaller
    1       1   1006 mov  $1, 2000
    1       1   1007 halt
    1       0   ----- Halt;Switch -----  ----- Halt;Switch -----  
    1       0                            1000 test $1, %ax
    1       0                            1001 je .signaller
    1       0                            1002 mov  2000, %cx
    1       0                            1003 test $1, %cx
    1       0                            1004 jne .waiter
    1       0                            1005 halt
```
__How should the code behave?__ 
__How is the value at location 2000 being used by the threads? What will its final value be?__
_ax should be 1 (signaller) or 0 (waiter). The value at location 2000 is being used as a flag to determine if the signaller or waiter should execute.The first run with ax=1,ax=0 tells the first thread to be a signaller and the second thread to be a waiter. The first thread executes and sets the value at location 2000 to 1, which means that the waiter can run. When the second thread runs, it checks the value at location 2000 to see if it can run: it can, because the signaller has already said it can._

### Q10

```
./x86.py -p wait-for-me.s -a ax=0,ax=1 -R ax -M 2000 -c
ARG seed None
ARG numthreads 2
ARG program wait-for-me.s
ARG interrupt frequency 50
ARG interrupt randomness False
ARG argv ax=0,ax=1
ARG load address 1000
ARG memsize 128
ARG memtrace 2000
ARG regtrace ax
ARG cctrace False
ARG printstats False
ARG verbose False

 2000      ax          Thread 0                Thread 1         
    0       0   
    0       0   1000 test $1, %ax
    0       0   1001 je .signaller
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       0   1002 mov  2000, %cx
    0       0   1003 test $1, %cx
    0       0   1004 jne .waiter
    0       1   ------ Interrupt ------  ------ Interrupt ------  
    0       1                            1000 test $1, %ax
    0       1                            1001 je .signaller
    1       1                            1006 mov  $1, 2000
    1       1                            1007 halt
    1       0   ----- Halt;Switch -----  ----- Halt;Switch -----  
    1       0   1002 mov  2000, %cx
    1       0   1003 test $1, %cx
    1       0   1004 jne .waiter
    1       0   1005 halt

```
---
__How do the threads behave? What is thread 0 doing?__ 
__How would changing the interrupt interval (e.g., -i 1000, or perhaps to use random intervals) change the trace outcome?__
__Is the program efficiently using the CPU?__

_The second run with ax=0,ax=1 tells the first thread to be the waiter and the second thread to be the waiter. However, the interrupt frequency is defaulting to 50 and therefore the waiter loops through many times before the second thread, representin the signaller, is executed. When the second, signaller, thread finally executes and halt, the first, waiter, thread can now continue and halt._
