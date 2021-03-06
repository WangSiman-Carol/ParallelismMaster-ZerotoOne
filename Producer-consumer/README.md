# Producer-consumer
This application defines a producer that generates work tasks and consumers that consume the work tasks. Producer and consumer communication via a queue that has a maximum capacity of 100 tasks. The consumer executes the tasks by operating on a set.

*********
### Solutions

I implemented multi-threaded version of producer-consumer program under four optimizations:
1. One thread runs as a producer, while one thread runs as a consumer
    - Use a global lock on the task queue, and a global lock on the element set
2. One thread runs as a producer, while N threads run as N consumers, where N is configurable N={1, 2, 4, 8, 16, 32}
    - N consumer threads execute tasks in N queues respectively. Use N locks to provide mutual exclusion on N queues.
3. To reduce contention on the task queue, the producer generates 100 tasks at a time and enqueues them in one queue while holding the queue lock. 
    - In this case, the producer thread needs to sleep for a sutible time in between producing batches of tasks to allow consumers to consume tasks.
    - N consumers work as those in Optimization 2
4. Partition the set into a number of smaller sets where one consumer operates on one set.
    - N consumer threads use N locks to provide mutual exclusion on N sets.
    - The producer thread works the same as in Optimization 3

********
### Analysis of a critical code block
The following code block appears in Solution 1. The producer is trying to acquire the global lock on the queue.
```C++
    q_mtx.lock();
    while (q.size() > 100) {
        q_mtx.unlock();
        q_mtx.lock();
    }
```
Compiler optimized ```while (qm[qid].size() > 100)``` by storing the size of queue in a local cache. ```q.size()```is not an atomic operation, it can have a data race if any other thread is writing to the queue when it's called. Thus, a mutex is needed to protect queue's size.

```q_mtx.unlock()``` within the while loop will wake up sleeping consumers.
********

### Instructions for Build and Run
BUILD
```
make
```
RUN
```
./pc <number of threads>
```
