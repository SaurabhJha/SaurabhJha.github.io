---
layout: post
title: Review&#58; End-to-end Arguments in System Design
categories: [operating-systems, paper-review]
---
# {{page.title}}
These are my thoughts on the classic paper "End-to-end Arguments in System
Design" written by Jerome Saltzer, David Reed, and David Clarke. It's a very old
paper; the version I read was published in 1984. The paper argues against
implementing functionality at lower levels of the system. It appeals to
application requirements and argues for moving a function upwards in a layered
system.

I believe these ideas started gaining more ground after the advent of networked
computers. The concept is motivated using examples from data communication
networks and transactional systems.

The basic thesis is this: in a layered system, the correctness can only be
achieved at the end-to-end level. The lower levels don't have all the information
needed to do it perfectly.

The first illustrative example in this paper is the file transfer over the network.
One way to solve reliability issues is to reinforce each step and have retry
policies on every network hop. Another approach to ensure reliability is
*end-to-end check and retry*. End-to-end check and retry policies work
wonderfully well if network failures are rare. Also, it results in a simpler
implementation.

I can rephrase this argument in modern terms as follows. Suppose you are
communicating files using TCP. TCP can ensure that your files have been
transferred correctly. However, only the application can judge whether that file has
been processed correctly. The communication system reliability is a performance
enhancement but in no way it is sufficient.

Since not all of us play at the network level, here's one more realistic example.
Suppose you have a set of worker processes. You can communicate jobs to be done
to workers using a reliable queue. The queue can ensure that jobs are delivered
to workers. However, only workers can confirm that they have executed the tasks
communicated to them. The queue itself cannot ensure it.

The conclusion is correctness can only be achieved at the end-to-end level. The
lower levels can only help with performance. This changes the fundamental
question. Instead of asking: what do we need to do to make our network highly
reliable, we should ask: what changes should we do to make applications built on
top of the network highly performant. Looking at it this way, this has become a
performance problem. However, it is different from the majority of performance
problems we encounter in our day-to-day work which look more like empirical
science.

1. You measure the system
2. You identify the bottlenecks
3. You fix those bottlenecks.

In contrast, the original network performance problem described above is more of
an art problem. How can network help with application performance? Often, it
differs from application-to-application. I am repeating the excellent example
given in the paper.

Suppose your application is doing real-time voice communication. In this case,
having reliability at the network layer would be disastrous. The nature of this
problem demands that voice packets are fed constantly to the receiver. The lost
packets should be replaced with silence. However, if your application is
voice-recording, having a reliable network would make the application faster. That's
because voice-recording application have no real-time constraints. The network
can take its time to ensure that all packets have been transmitted reliably.
Because of a reliable network, it's likely that the final end-to-end check will
work on the first try.

In the conclusion section of the paper, a pitfall is pointed out by the authors:
Specifying the communication subsystem before the specification of applications that use that
subsystem. This leads to a burden on the communication subsystem designer who attempts to
do more to "help" app developers.

I enjoyed reading this paper. You can find it [here](http://research.cs.wisc.edu/areas/os/Qual/papers/end-to-end.pdf).
