---
layout: post
title: Writing a command shell
categories: [operating-systems, project]
---
# {{page.title}}
I got interested in systems programming due to an assignment at work that
involved sockets. I got curious about what could be happening behind such an
interface.

I started studying the python interface to sockets. It was very high level and
I did not learn much. So, I started studying the UNIX interface. With UNIX, I got overwhelmed
and painfully realized that I won't be able to understand it unless I know how operating
systems work. I began my long quest towards learning how low-level things work.

The first program I wrote in this domain was a command shell. It was the
first significant program I wrote in C.

## Why C?
Apart from frivolous reasons like "I wanted to program in C", I have two good reasons.

1. C introduces tiny overhead at runtime compared to other languages. Its
   runtime is so simple that you can guess the assembly instructions of a code
   snippet. This is important because as a job executor, a shell should
   introduce minimum overhead.
2. In C, you get to manage memory yourself. Many people consider it old
   fashioned. But for long-running processes like shell or server, you need very
   tight control over memory. Memory leaks can turn your program into a memory hog.
   Garbage collection subsystems like that of Python do not give any encouragement either.

## Design overview
The purpose of the command shell is to take command strings as input, parse
them, execute them, and repeat the cycle. Seen this way, you need a part to
read inputs, a part to parse inputs, and a part to execute inputs. Let
me describe the road I took to write this whole system.

### Prototype
To get my feet wet, I wrote something that echoes back my inputs.

{% highlight c %}
while (1) {
    input_string = get_new_input();
    print(input_string);
}
{% endhighlight %}

### Parsing the input
The next thing I did was this. Given a string obeying certain rules, translate it into a
format suitable as input for an exec frontend like `execv`.

{% highlight c %}
while (1) {
    input_string = get_new_input();
    input_object = parse(input_string);
}
{% endhighlight %}

### Executing input
Finally, we can execute the input object we constructed.

{% highlight c %}
while (1) {
    input_string = get_new_input();
    input_object = parse(input_string);
    exec(input_object);
}
{% endhighlight %}

### Continuing input processing
The above snippet will replace the shell process with the program image specified in the
`input_object`. We would like to continue processing new input. One solution is to
make a copy of the shell process and then replace that copied process with the
program specified in the `input_object`. This copying is done using `fork` system call.

{% highlight c %}
while (1) {
    input_string = get_new_input();
    input_object = parse(input_string);
    if (fork()) {
        exec(input_object);
    } else {
        wait();
    }
}
{% endhighlight %}

### Setting up a terminal interface
Finally, we want to move our shell to foreground of our terminal
device. Broadly, I linked the shell file descriptor with the current foreground process group
associated with terminal device.

{% highlight c %}
while (1) {
    /* Kill all process groups not associated with shell */
    tcsetpgrp(shell_fd, foreground_pgrp);
    input_string = get_new_input();
    input_object = parse(input_string);
    if (fork()) {
        exec(input_object);
    } else {
        wait();
    }
}
{% endhighlight %}

## Going further
The source code of the above description can be found [here](https://github.com/saurabhjha/bogus-shell).
After some initial excitement, I got bored. There is no single program that can
solve your problems. To solve interesting problems, you need multiple programs
communicating with each other. And the best way for communications is the sockets interface.
The word "sockets" draw a picture of reliable communication over
ethernet or wifi interfaces. TCP over IPv4 is only one part of the
operating system's socket interface. For address spaces, you have UNIX sockets
as well as raw sockets. For communication semantics, you have TCP, UDP, RAW, and
SEQPACKET.

But the above stupid program does illustrate some things you need to worry
about in distributed programs

### Application-level protocol
The input needs to be in a specified format for the program to be able to process
it. Most of the programmers know about HTTP protocol but I don't think the
overhead of parsing HTTP requests is worth it for internal communication in
network programs. For that, you need a custom protocol.

### Handling multiple clients
The shell executes inside a computer and has only one client. But it is not hard
to see the approach of fork/exec working for multiple clients. Forking a process
for every client is the classic UNIX approach used since the 1980s. Of course, there
are other low-overhead approaches like threads or continuations. But many
[programs](http://www.postgresql.org/docs/current/static/connect-estab.html)
still use this idea.

### Efficient memory allocation
If you notice [here](https://github.com/SaurabhJha/bogus-shell/blob/master/io.c),
I attempt to minimize memory consumption by reallocating space for command
string array and freeing the originally allocated array. Memory allocations are
expensive and there are many projects like [jemalloc](http://www.canonware.com/jemalloc/)
that aim to solve this problem.

This shell was a good starting point for writing the kind of programs that solve
more intresting problems.
