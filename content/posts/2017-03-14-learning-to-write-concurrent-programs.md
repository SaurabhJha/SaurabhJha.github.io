+++
title = 'Learning to write concurrent programs'
date = 2017-03-14T00:00:00+00:00
+++

I taught myself some concurrent programming lately. I did it as part of my
operating systems studies. In this essay, I will illustrate it using an
example problem.

I will be illustrating concurrent programming using operating system primitives.
[1] There are several different ways to do concurrent programming that does
not involve interacting with the operating system kernel. Even if we restrict
ourselves to using kernel primitives, there are many variants. I will
mainly be concerned with concurrency using mutexes and condition variables. In
literature, it is sometimes called *concurrent programming using shared objects*.

## Platform details
I developed this program on a Vagrant VM using Virtualbox that ran FreeBSD 10.2.
Even though I have a lot more experience with Linux [2], I feel I am more
comfortable using FreeBSD, even though I don't have a lot of experience with it.
There are a couple of reasons.
* The documentation and man pages of FreeBSD are really good.
* The kernel code of FreeBSD is more readable to me than that of Linux.

## Problem definition
I am choosing a contrived problem so that we can focus on concurrency,
without getting distracted by incidental details. The problem is this.

We have a fixed number of integers. Our objective is to calculate the Fibonacci
number of all these integers. [3] The constraint is, for each integer, we will
calculate the Fibonacci number exactly once.

## General ideas about the solution
We can solve this problem in many different ways. A straightforward
way is to put all the integers in an array, loop over the array, and calculate
all Fibonacci numbers. This is fine but we won't be taking advantage of multiple
cores. [4] If we can make our program concurrent using threads, it might
increase its throughput, the number of jobs processed per second.

Concurrent programming is hard because the data structures are shared between
multiple threads. We need to ensure that the threads access the shared data
structures safely without causing any inconsistency or corruption.

Since the shared data structure is the central problem here, we will tackle that
first. What data structure should we choose for this problem? Remember the
problem is we have got integers and each integer has to be processed exactly
once.

For this problem, we can use a simple circular queue. It will look like this.
We can have producer threads enqueuing the queue with integers and we can have
consumer threads dequeuing integers from queue and process it.

## Sequential solution
The struct for a sequential queue looks like this. [5][6]

{{< highlight c >}}
struct queue {
    /* State variables */
    int *storage;
    int front; /* Front of the queue. Should have element
                  when queue has at least one. */
    int rear; /* Next free slot. Should not have element
                 unless the queue is full. */
    int length;
}
{{< / highlight >}}

Of course, we will have problems with this `struct` if multiple threads access
it simultaneously. We will address this later.

The code that inserts and deletes elements from this queue, as well as mutates its
state variables, would look like this.

{{< highlight c >}}
struct queue *initialize_queue(int queue_length)
{
    /* Allocate memory for queue_object and queue_object->storage. */
    return queue_object;
}

void destruct_queue(struct queue *queue_object)
{
    /* Free memory of queue_object */
}

bool enqueue(struct queue *queue_object, int element)
{
    /* Try to insert element into queue_object. Return true on success and
       return false on failure. */
    return true;
}

int dequeue(struct queue *queue_object)
{
    /* Remove the front element from the queue and return it. */
    return element;
}

bool is_queue_full(struct queue *queue_object)
{
    /* Returns a boolean indicating if the queue is filled up. */
    return false;
}

bool is_queue_empty(struct queue *queue_object)
{
    /* Returns a boolean indiciating if the queue is empty. */
    return false;
}
{{< / highlight >}}

Now it is very important that all state changes happen inside these functions
and nowhere else. Its the responsibility of these functions to do insertion/deletion
of elements and do state changes (for example, changing front and rear properties) safely.
For simplicity, I will call these functions "public functions" from now on. [7]

Also, note that we have got two simple predicate functions, `is_queue_full` and
`is_queue_empty`. They will come in use in a bit.

## Requirements of concurrency
The requirements of concurrency are as follows. We will refine these as we go
along.
* Only one thread can execute a public function at a time.
* If a currently running thread can't make progress with a public function, it
  must allow other threads to make progress with that function.
* If a thread is waiting for some condition to become true, we must be able to
  notify that sleeping thread when that condition holds true.

## Thread basics
First, we will look at how to create and manipulate threads. The standard
function to create a thread in C is `pthread_create`. It is used like this.

{{< highlight c >}}
pthread_create(&thread_id, &thread_attr, thread_routine, thread_arg);
{{< / highlight >}}

Here, `thread_id` is a variable of type `pthread_t`, `thread_attr` is a
variable of type `pthread_attr_t`, `thread_routine` is a function that
takes an argument of type `void *` and returns `void *` and `thread_arg`
is a variable of type `void *` that is passed to `thread_routine` function.

Let's take an example. Suppose we want to execute a function `fib` in a
separate thread. The declaration of `fib` looks like this.

{{< highlight c >}}
int fib(int n);
{{< / highlight >}}

Suppose we want to calculate `fib(3)`. We will call it in a separate thread
like this.

{{< highlight c >}}
pthread_t some_thread_id;

pthread_create(&some_thread_id, NULL, fib, 3);
{{< / highlight >}}

Notice that the second argument, which was supposed to be a `thread_attr`, is
NULL. This is what we do when we want default attributes. Also, notice that the
`pthread_create` spec said that we should use a function that takes a `void *` and
returns a `void *` and I am instead passing a function that takes an `int` and
returns an `int`. I want to keep things simple and I am relying on the compiler
to do the type coercions for me. In my experience, these particular coercions
are safe and do not lead to any corruption or information loss.
The same holds for the fourth argument `thread_arg`.

Suppose in our `main` function, we create a thread that executes `fib`.
It will look like this.

{{< highlight c >}}
int main()
{
    pthread_t random_thread_id;

    pthread_t(&random_thread_id, NULL, fib, 3);
    return 0;
}
{{< / highlight >}}

The problem is `main` can return before the thread finishes execution. To
*wait* for the current thread to finish, we use the function `thread_join`.

{{< highlight c >}}
int main()
{
    pthread_t random_thread_id;

    pthread_t(&random_thread_id, NULL, fib, 3);
    pthread_join(random_thread_id);
}
{{< / highlight >}}

`pthread_join` returns immediately if the thread has finished its execution.
Otherwise, it will block till the thread finishes, and then it will return. By
induction, if you create n separate threads, you can wait for all n of them to
complete by doing `pthread_join` on all the thread_ids. This is what I did for
all producer and consumer threads.

There is no guarantee that a thread created by `pthread_create` will start
immediately. Whenever we call `pthread_create`, the operating system puts the
thread into something called the **ready list** of the schedular. The kernel
schedular then picks up a thread to run from the ready list based on some
scheduling policy. So, `pthread_create` creates a thread and puts it into the
ready list to be run later.

## Making queue concurrent
Now let's get back to our `queue` struct. We want to make do some changes so that
this data structure can be used by multiple threads safely. Let's revisit our
criteria for concurrency.
1. Mutual exclusion: Only one thread can change the state of the queue at a
   time.
2. Wait if we can't progress: If a thread can't progress, it must allow other
   threads to make progress.
3. Notify waiting threads: If something happens that allows waiting threads
   to make progress, we should be able to notify them.

Now let's discuss these requirements in more detail and implement them.

### Mutual exclusion
The primitive that provides mutual exclusion is called the **mutex**. Its a variable
of type `pthread_mutex_t`. We will consider two functions that operate on mutexes.
These are `pthread_mutex_lock` and `pthread_mutex_unlock`. They are used like this.

{{< highlight c >}}
pthread_mutex_t my_mutex;

pthread_mutex_lock(&my_mutex);
/* Some code */
pthread_mutex_unlock(&my_mutex);
{{< / highlight >}}

Now, `/* Some code */` can only be executed by that one thread who has called
`pthread_mutex_lock` and hasn't called `pthread_mutex_unlock` yet. If a thread
calls `pthread_mutex_lock` while another thread is still using it, the thread
is stopped and put into a waitlist associated with `my_mutex`. Once `my_mutex`
becomes free, one thread is popped from the waitlist of `my_mutex` and made
available to run by putting it back into the ready list.

Our queue struct will look like this now. We add a new field in `struct` called
`mutex`.

{{< highlight c >}}
struct queue {
    /* State variables */
    int *storage;
    int front; /* Front of the queue. Should have element
                  when queue has at least one. */
    int rear; /* Next free slot. Should not have element
                 unless the queue is full. */
    int length;

    /* Synchronization variables */
    pthread_mutex_t *mutex;
}
{{< / highlight >}}

We have got functions that modify the state variables of the `queue`. They are
the `enqueue` and the `dequeue`. There is no state change outside of these functions.

So **we acquire the lock at the beginning of these functions and release that at the
end**. We modify our `enqueue` and `dequeue` functions like this.

{{< highlight c >}}
bool enqueue(struct queue *queue_object, int element)
{
    /* Try to insert element into queue_object. Return true on success and
       return false on failure. */
    pthread_mutex_lock(queue_object->mutex);
    /* Insertion code */
    pthread_mutex_unlock(queue_object->mutex);
    return true;
}

int dequeue(struct queue *queue_object)
{
    /* Remove the front element from the queue and return it. */
    pthread_mutex_lock(queue_object->mutex);
    /* Removal code */
    pthread_mutex_unlock(queue_object->mutex);
    return element;
}
{{< / highlight >}}

This is where restricting state changes to these functions start paying off.
We only need to worry about mutual exclusion in these functions. Now it is
very important to note that **we hold the lock at the beginning and release
the lock at the end of these functions**. That is, don't acquire and release
the locks anywhere else in the code.

You might think that this is inefficient and coarse-grained. Or you might think
that holding locks like this will increase lock contention ("It changes state
only in those three lines so let's just wrap those inside lock"). The problem
with that approach is it makes it very hard to understand and modify existing
code safely. Furthermore, it is not inefficient and won't lead to lock
contention as we will see right now.

### Wait if we can't make progress
Suppose a thread is executing `enqueue` and finds that the queue is full. What
should we do here?

We should give up the mutex we are holding and go to the waiting state. The
object that helps us achieve this is called a **condition variable**.

Condition variables give us the mechanism to **atomically release the lock
and go to the waiting state**. A condition variable is a variable of type
`pthread_cond_t` Each condition variable has associated with it
a waitlist of threads.

We have a function, `pthread_cond_wait`, that takes a mutex and a condition variable
as arguments and releases the mutex and puts the running thread to the waitlist of the condition variable.
The greatest value provided by `pthread_cond_wait` is it does these two operations *atomically*.
So if we have code like this.

{{< highlight c >}}
pthread_cond_t *code_cond_var;
pthread_mutex_t *code_mutex;

pthread_cond_wait(code_cond_var, code_mutex);
{{< / highlight >}}

The thread running this snippet will give up `code_mutex` and go to the
waitlist of `code_cond_var`.

When would we want to call `pthread_cond_wait`? We want to call it
whenever we can't make progress. Whenever we design a shared object and
deciding on condition variables, we ask this question: When can a thread
wait? For our problem, the answers will be:

1. Whenever we are running `enqueue` and the queue is full.
2. Whenever we are running `dequeue` and the queue is empty.

Based on that, we would need two condition variables `queue_has_space`
and `queue_has_element` for 1) and 2) respectively.

Our queue struct looks like this now.

{{< highlight c >}}
struct queue {
    /* State variables */
    int *storage;
    int front; /* Front of the queue. Should have element
                  when queue has at least one. */
    int rear; /* Next free slot. Should not have element
                 unless the queue is full. */
    int length;

    /* Synchronization variables */
    pthread_mutex_t *mutex;
    pthread_cond_t *queue_has_space;
    pthread_cond_t *queue_has_element;
}
{{< / highlight >}}

We added two new fields of type `pthread_cond_t`: `queue_has_space` and
`queue_has_element`. Our state-changing functions look like this now.

{{< highlight c >}}
bool enqueue(struct queue *queue_object, int element)
{
    /* Try to insert element into queue_object. Return true on success and
       return false on failure. */
    pthread_mutex_lock(queue_object->mutex);

    while (is_queue_full(queue_object)) {
        pthread_cond_wait(queue_object->queue_has_space, queue_object->mutex);
    }
    /* Insertion code */
    pthread_mutex_unlock(queue_object->mutex);
    return true;
}

int dequeue(struct queue *queue_object)
{
    /* Remove the front element from the queue and return it. */
    pthread_mutex_lock(queue_object->mutex);

    while (is_queue_empty(queue_object)) {
        pthread_cond_wait(queue_object->queue_has_element, queue_object->mutex);
    }
    /* Removal code */
    pthread_mutex_unlock(queue_object->mutex);
    return element;
}

{{< / highlight >}}

Let's look at `enqueue`. `pthread_cond_wait` atomically releases `queue_object->mutex` and go to the
waitlist of `queue_object->queue_has_space`. It won't return until we chose
to wake up the threads in the waiting list of `queue_object->queue_has_space`.
We will discuss how to wake up threads that are waiting in the next section.

Notice that we wrapped `pthread_cond_wait` calls inside a while loop. This is
important. **We should always wait on a condition variable inside a while
loop**. This is because if later, we return from pthread_cond_wait, there is no
guarantee that the current condition will still allow us to progress. Let's
consider an example.

Suppose we only have space for one element in our queue and we have got an
enqueue thread E1 and two dequeue threads D1 and D2. D1 runs first, sees that
there is no element to dequeue, releases the lock, and go to the waitlist.
Now, E1 acquires the lock and fills up the queue with one element. Suppose after that,
instead of D1, D2 runs and dequeues the thread and we switch to D1. Now, if
we have our `pthread_cond_wait` inside a `while` loop, we will check the
condition again and go to the waitlist. If instead, we had our
`pthread_cond_wait` call inside an `if` statement, we would have progressed
further and might have caused data corruption.

### Notify waiting threads
Finally, we want to notify the threads in the waitlist of condition variables
if they can make progress. For example, after `dequeue`, there is a
chance that some `enqueue` thread was waiting for it. For this, we use a
function `pthread_cond_signal`. It wakes up **one** thread from the waitlist
of a condition variable and puts it into the ready list. [8] Again, as above,
there is no guarantee that the signaled thread will run immediately. It is put
into the ready list and its the scheduler's job to run it according to
scheduling policy. Its called like this.

{{< highlight c >}}
pthread_cond_t *code_condvar;

pthread_cond_signal(code_condvar);
{{< / highlight >}}

Our `enqueue` and `dequeue` functions will look like this now.

{{< highlight c >}}
bool enqueue(struct queue *queue_object, int element)
{
    /* Try to insert element into queue_object. Return true on success and
       return false on failure. */
    pthread_mutex_lock(queue_object->mutex);

    while (is_queue_full(queue_object)) {
        pthread_cond_wait(queue_object->queue_has_space, queue_object->mutex);
    }
    /* Insertion code */
    pthread_cond_signal(queue_object->queue_has_element);
    pthread_mutex_unlock(queue_object->mutex);
    return true;
}

int dequeue(struct queue *queue_object)
{
    /* Remove the front element from the queue and return it. */
    pthread_mutex_lock(queue_object->mutex);

    while (is_queue_empty(queue_object)) {
        pthread_cond_wait(queue_object->queue_has_element, queue_object->mutex);
    }
    /* Removal stuff */
    pthread_cond_signal(queue_object->queue_has_space);
    pthread_mutex_unlock(queue_object->mutex);
    return element;
{{< / highlight >}}

Concurrent programming has a reputation that we have to reason about the global
structure of the program. If we wrap our condition variable checks inside a
`while` loop as we did above, we avoid some of that. In our case,
`pthread_cond_signal` can be regarded as a *hint* for other threads to check if
they can run again.

Thus, we can liberally do `pthread_cond_signal` calls whenever we think we changed
something that might allow others to progress. We hope that others will check the condition
again on their end.

## General Remarks
The full implementation can be found [here](https://github.com/saurabhjha/bogus-queue).

Thread programming using locks has got a bad reputation in recent years.
Especially for I/O concurrency, if you propose a solution that involves threads,
others would probably think you have gone nuts. Or maybe fire you while they are
at it.

However, I still think I would use threads even for I/O concurrency. The
reasons are

1. I don't want to worry about thread scheduling. The kernel does it for me. It does it
   better than me because it has got privileges I as a user program don't have.
2. I want my solution to work for any kind of I/O. Network I/O is not the only
   kind of I/O, you know.
3. I want my solution to be portable across UNIX implementations. I don't want
   to change my implementation from `epoll` to `kqueue`. [9]

I think this whole sentiment of *threads sucks* originated from Dan Kegel's
landmark C10K paper. [10] It has been updated from time to time but I think the
kernel implementations have come a long way since then. For example, how much
do you think it costs to acquire an uncontended lock. It can be as fast as
10 instructions. In some implementations, it is as low as 2 instructions. [11]

What about processes? `fork` is definitely expensive right?
After all, you have to copy every open file for the child process. Well, in recent
Copy-On-Write (COW) implementations, [12] you don't even copy those.

It seems to me that despite all the modernity of developers (using the latest
frameworks, latest libraries, etc.), they are still programming using old ideas
about the platforms on which their applications run.

That's for I/O concurrency. For CPU concurrency, I don't think there is any
debate. Thread programming is the way to go.

However, these abstractions of mutexes and condition variables were developed
for operating system purposes. Are they suitable for application programs? I am
not sure. For example, Erlang doesn't use operating system threads or processes
and its model sounds promising. There have been successful
companies/projects built on it. I would love to learn Erlang and
maybe after that I will conclude that all these locks, condition variables, etc.
are not good abstractions for normal programs. Or maybe I will conclude that
there is a place for both kinds of abstractions. I am certainly looking forward
to that.

Incidentally, while I was writing this program, I spent much more time
implementing sequential queue then concurrency aspects. It is pretty clear to me
how much I suck in data structures and algorithms. In fact, until last year, I
used to think of algorithms as a topic that people study to crack job
interviews. Oh well, better late than never.

## Further reading
If you are interested, these are good books to learn more about the topics
covered here.

1. [Operating systems: Principles and Practice](http://ospp.cs.washington.edu)
   by Thomas Anderson and Michael Dahlin. Its a pretty recent book.
2. [Unix network programming](http://www.kohala.com/start/unp.html) by Richard
   Stevens. A classic even though it was written 27 years ago.

## Notes

[1] By kernel primitives, I mean C standard library functions that are
    interfaces to system calls.\
[2] I basically used it in all my previous projects, official or personal.\
[3] Fibonacci functions is useful to get a wide range of computational time.
    For example, there's a noticeable time difference between calculating
    fibonacci of 1 and fibonacci of 32. I got this idea from one of the David
    Beazley's [talk](https://www.youtube.com/watch?v=MCs5OvhV9S4).\
[4] There are number of gotchas with using synchronization variables in multiple
    cores as well. But I will delve into that in some other essay.\
[5] Its often said that you cannot turn a sequential program into a
    concurrent one without changing program design in a major way. I haven't
    faced this problem here.\
[6] I am using C here. I have omitted details about memory allocation to focus
    on the ideas. I will give a link to the full implementation in the end.\
[7] If you using  an object oriented language, you can think of it as a queue
    class having a set of public methods and private state variables.\
[8] There is another function `pthread_cond_broadcast` that wakes up all threads
    in the waiting list at once. We won't be using it for this problem. But
    they are useful in some problems, for example in a Readers-Writers lock
    implementation.\
[9] I really like `kqueue` though and I am looking forward to learning and using
    it.\
[10] http://www.kegel.com/c10k.html \
[11] I am not able to find specific code in FreeBSD that achieve this. However,
    the basic idea is if the lock is uncontested, we don't have to go through
    the schedular, don't have to acquire schedular's spinlock and we don't have
    to disable interrupts in order to acquire a lock. If the lock is busy
    though, we get a scheduling overhead. \
[12] They might be pretty old now.
