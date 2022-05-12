---
layout: post
title: Four Lessons on multi-threaded programming
---




Writing a good multi-threaded program can be challenging. But living in an age where Moore's Law no longer brings us 2X speed up every year and computer manufacturers are sticking more cores in a computer, it is an essential technique to learn to write good multi-threaded programs. Before diving into examples of multi-threaded programs, we must first understand what a thread is and why we need it.

### Concurrency and threads
Modern computers are typically equipped with more than one CPU. Aside from using each CPU on a different task, another way to utilize all CPUs is to use all of them to do one job faster. To fulfill the new need to run a process on multiple CPUs, we need to further divide a process into threads. Each thread is very much similar to a separate process, except they share the same address space and thus can access the same data (so that they can concurrently work on the same task). Moreover, Threads can also run concurrently on a single CPU to avoid being blocked by I/O. Imagine a program that has to alternate between I/O and computation (for example, a video game). Without threads, the CPU has to wait for I/O to finish before it can do the computation. But, if we can divide the task into multiple threads, while one thread in your program waits for I/O, the CPU scheduler can switch to other threads, which are ready to run and do something useful.[1]

### data race

However, all these benefits from using multi-threaded programming come with a cost. As the number of threads increases, more CPU time is spent on communication between threads (to keep the data synchronized). Other than that, sharing data between threads can lead to inconsistent results if it's not carefully designed (we will explore this in more detail later). This is what we refer to as a data race. Or, more formally, a data race occurs when[2]: 1) two or more threads in a single process are accessing the same memory location concurrently, 2) at least one thread is writing this memory, 3)  threads are not using any locks to control their accesses to that memory. 

The problem here is that scheduler might decide to switch from one thread (t1) to another (t2) while t1 has loaded the memory but has not done any work yet. t2 then loads the same memory, does its work, and stores the value back to the memory. Then when context switching back to the p1, it won't reload the memory; instead, it will do its work and store the value back to the memory. Now, anything done by t2 will be erased. Furthermore, due to the complex nature of the scheduler, this might lead to different results every time we run the program.

### How to fix a data race

Now that we have a rough idea of what is data race and what is causing data race, we should be thinking about a fix.

### Data Race Examples


Example 1:

First, let's look at this simple program

```
static int sum = 0;

threadfunc(void *){
    for (int i=0; i<100000000; i++)
        sum += 1;
}

main(){
    pthread_t p1, p2;
    pthread_create(&p1, NULL, threadfunc, NULL);
    pthread_create(&p2, NULL, threadfunc, NULL);

    pthread_join(p1, NULL); // wait for thread p1 to finish
    pthread_join(p2, NULL); // wait for thread p2 to finish

    printf("sum = %d\n", sum);
}

```

Intuitively, this program is running two threads, each adding 100000000 to the gloabal variable `sum`. So the result of this program would be 200000000. But, if you actually run this program, you would only get 200000000. The result will most likely falls in the range 100000000 to 200000000.

So what is going on here? The problem is that although `sum += 1;` takes only one line, it is atomic. CPU is actually doing three instructions for this one line of code, loading sum, do addition and store result. And as mentioned above, schedueler could cut in between any of these instructions and context switch to another thread, thereby causing the problem.

<b>Lesson 1: One liner does not guarantee atomicity</b>.

Example 2:

Now, let's look a failed attempt to fix the above code

```
static int sum = 0;

threadfunc(void *){
    pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

    for (int i=0; i<100000000; i++){
        Pthread_mutex_lock(&lock);
        sum += 1;
        Pthread_mutex_unlock(&lock);
        }
}

main(){
    pthread_t p1, p2;
    pthread_create(&p1, NULL, threadfunc, NULL);
    pthread_create(&p2, NULL, threadfunc, NULL);

    pthread_join(p1, NULL); // wait for thread p1 to finish
    pthread_join(p2, NULL); // wait for thread p2 to finish

    printf("sum = %d\n", sum);
}

```

To avoid data race, we now add a mutex to keep multiple threads from accessing `sum` at the same time. So the outcome should be 200000000 now, right?

No, the above code does not prevent data race. Since mutex is declared and initialised in function `threadfunc()`, each thread will get its own lock and lock its own lock while accessing `sum`. Therefore, p1 locking its lock won't prevent p2 from accessing `sum` and data race still exist in the code.

<b>Lesson 2: Only locking the same lock can prevent multiple threads from accessing same piece of data</b>.


Example 3 & 4:

Now, let's give another attempt at fixing the above code.

```
static int sum = 0;
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

threadfunc(void *){

    pthread_mutex_lock(&lock);
    for (int i=0; i<100000000; i++){
        sum += 1;
        }
    pthread_mutex_unlock(&lock);
}

main(){
    pthread_t p1, p2;
    pthread_create(&p1, NULL, threadfunc, NULL);
    pthread_create(&p2, NULL, threadfunc, NULL);

    pthread_join(p1, NULL); // wait for thread p1 to finish
    pthread_join(p2, NULL); // wait for thread p2 to finish

    printf("sum = %d\n", sum);
}

```

Fourtunately this time, the above code is finally producing the correct output (200000000). But does this code really runs concurrenly? Probably not.

Since the whole thread is included in the critical area, one thread has to finish running before another thread can do any work.

How about we use the following code to replace the `threadfunc()`

```
threadfunc(void *){

    for (int i=0; i<100000000; i++){
        pthread_mutex_lock(&lock);
        sum += 1;
        pthread_mutex_unlock(&lock);
        }
}

```
This code indeed solve the problem of not actually doing the work concurrently, but it introduces yet another problem. Here, each thread needs to acquire the lock and release the lock 100000000 times each. Doing so many lock and unlock would cost a drop in performance.

<b>Lesson 3: Including the whole thread in a critical section is not very efficient</b>.

<b>Lesson 4: Locking and unlocking a mutex for every iteration of a loop is also not efficient</b>.

Finally, let's see an example that do the job quickly and correctly (and also concurently).

```
static int sum = 0;
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

threadfunc(void *){

    int partial_sum = 0;
    for (int i=0; i<100000000; i++){
        partial_sum += 1;
        }

    pthread_mutex_lock(&lock);
    sum += partial_sum;
    pthread_mutex_unlock(&lock);
}

main(){
    pthread_t p1, p2;
    pthread_create(&p1, NULL, threadfunc, NULL);
    pthread_create(&p2, NULL, threadfunc, NULL);

    pthread_join(p1, NULL); // wait for thread p1 to finish
    pthread_join(p2, NULL); // wait for thread p2 to finish

    printf("sum = %d\n", sum);
}
```
### conclusion
From above examples, we can see that it's hard to make your program run concurrently. What's even harder is getting a good performance while ensuring a correct result.


### references
- \[1\]: Remzi H. Arpaci-Dusseau and Andrea C. Arpaci-Dusseau  "[Operating System: Three easy pieces](https://pages.cs.wisc.edu/~remzi/OSTEP/threads-intro.pdf)"

- \[2\]:"[What is a Data Race?](https://docs.oracle.com/cd/E19205-01/820-0619/geojs/index.html)"

