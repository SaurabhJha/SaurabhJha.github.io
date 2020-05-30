---
layout: post
title: Review&#58; The Structure of the "THE" Multiprogramming System
categories: [operating-systems, paper-review]
---
# {{page.title}}
This paper was written by the great Edsger Dijkstra. Before reviewing it, I
should point out a couple of quick facts:
1. It was published in SOSP 1967.
2. It predates UNIX and C.

This paper describes a multiprogramming system named "THE". It's main
contributions are:
1. A demonstration of the concept of software layering and how it leads to better verification
   and testing of a system.
2. The idea of software page-based virtual memory.
3. The idea of semaphore for use in concurrency.

This paper is about fifty years old. I will try to describe it in
today's terminology rather than using terms like "core memory", "drum memory"
and so on.

The objectives for building this system were:
* Improvement of throughput for short-duration programs.
* Economic use of I/O devices.
* Virtual memory and new techniques for CPU scheduling.
* Allowing the use of the machine by apps that need functionality rather than the speed of
  the computers. In other words, multiprogramming.

The salient features of this system were:
1. All activities were divided into sequential processes.
2. All processes were organized in a hierarchy.
3. Each hierarchy level implemented one or more abstractions.
This hierarchical design was crucial for the testing and verification of this
system. I will expand on this point in more detail after I describe all levels
in this system.

Prof. Dijkstra was concerned about debugging interrupt handlers. This led to
the group's main contribution to the art of system design: division of systems into
different levels. This enabled the group to demonstrate the correctness of their
system rather easily.

First, the paper describes the virtual memory mechanism, which is termed as
"automatic control of secondary storage". The idea is to have a strict
distinction between "pages" (in RAM or disk) and information units termed
"segments". Pages are a division of RAM and disk in appropriate units. In
modern systems, a page is 4KB. Segments are the containers of the information and
have the same size as a page. Segments have a separate identification system
that allows them to exceed the combined number of RAM and disk pages. This
independence allowed for a segment to be allocated to a disk page in the most
optimal way. It also allowed for a program to occupy non-consecutive pages.
In modern terms, pages would be called physical pages and segments would
be virtual pages.

Next, the paper describes a multiprogramming environment. There are multiple
sequential processes, each running at an undetermined speed. Each user program,
the input, and the output device is represented by a sequential process. The
coordination between processes is done using semaphores.

Next, the paper describes system levels:
* Level 0: Schedular in modern terms. A timer interrupt is used to
  multiprogram between multiple sequential processes. Above this level,
  the number of processors on a machine is irrelevant.
* Level 1: Virtual memory subsystem. This layer is responsible for bookkeeping
  operations of virtual pages and their allocation in physical pages, either
  in RAM or on disk.
* Level 2: Message interpreter in the original paper. A part of I/O subsystem in
  modern terms. This layer allocates a console keyboard. When a key is pressed,
  the character is sent to the machine along with a device interrupt. Above this
  level, each process thinks they have their private conversational console.
  This sharing is achieved using semaphores. This level was implemented above
  level 1 because conversational load may be huge and may need a backing store.
* Level 3: Sequential processes associated with the buffering of the input and the output
  streams. In modern terms, I/O subsystem. This abstracts actual devices as
  "logical communication units". Again, devices are shared using semaphores.
* Level 4: User programs
* Level 5: System operator (not sure if this was intended as a joke).

This separation of levels was crucial for testing of the system. For example,
this enabled them to ignore timer interrupts on level 1 and above, and
differences between disk speed and processor speed on level 2 and above.
Layering helped them keep the number of test cases at each level manageable.

I normally ignore appendices. But this paper's appendix introduced the very
important idea of semaphores. It is similar to modern semaphores:
* "P" operation decreases the value of semaphore by 1. If the resulting value
  is non-negative, the process can continue executing the next statement. Otherwise,
  it is put into the waitlist associated with that semaphore
* "Q" increments a semaphore value to 1. If the resulting value of the semaphore is
  positive, there is no further effect. If it's nonpositive, one process is
  removed from the waitlist of that semaphore. If there is more than one
  process in the waitlist, any process can be removed from the waitlist;
  there is no ordering between processes in the waitlist.

So the origin of semaphores was in operating systems. It was first used for concurrency
in operating systems.

Amazingly, this fifty-year-old paper is still so highly relevant. If I
were a researcher, I would have aspired to write a paper like this. This paper
is a major achievement not only in the ideas it introduced but also in the prose.
