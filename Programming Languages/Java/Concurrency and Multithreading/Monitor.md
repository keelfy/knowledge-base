[Source](https://www.baeldung.com/cs/monitor#monitor-features)
In Java, we use the [[Synchronized Keyword|synchronized keyword]] to mark critical sections. We can use it to mark methods (also called synchronized methods) or even smaller portions of code (synchronized statements).

There are opposed opinions about which approach to favor – method synchronization is usually the [[Synchronized Keyword#_Synchronized_ Instance Methods|recommended simpler approach]], while the synchronized statements could be a better choice from the [security point of view](https://wiki.sei.cmu.edu/confluence/display/java/LCK00-J.+Use+private+final+lock+objects+to+synchronize+classes+that+may+interact+with+untrusted+code?focusedCommentId=88498302#comment-88498302).

In Java, there’s a logical connection between the monitor and every object or class. Hence, they cover instance and also static methods. Mutual exclusion is accomplished with a lock associated with every object and class. This lock is a binary semaphore called a _mutex_.

# Building and Exclusive Room Analogy
Java’s implementation of a [[Programming Languages/Java/Concurrency and Multithreading/Monitor|monitor]] mechanism relies on two concepts – the _entry set_ and the _wait set_. In literature, authors use a building and exclusive room analogy to represent the monitor mechanism. In this analogy, only one person can be present in an exclusive room at a time.

So, in this analogy:
- the monitor is a building that contains two rooms and a hallway
- the synchronized resource is the “exclusive room”
- _wait set_ is a “waiting room”
- _entry set_ is a “hallway”
- threads are people who want to get to the exclusive room
![[Pasted image 20240424154149.png]]

When the person wants to enter the exclusive room, he first goes to the hallway (the _entry set_) where he waits for a scheduler. Therefore, the scheduler will pick the person and send him to the exclusive room.

Schedulers in JVMs use a priority-based scheduling algorithm. In case two threads have the same priority, the JVM uses the FIFO approach.

Hence, when the scheduler picks the person, he enters the exclusive room. It could be that some specific situation is happening in this room, so that person needs to go out and wait for the exclusive room to become available again. Therefore, that person will end up in the waiting room (the _wait set_). Consequently, the scheduler will schedule this person to enter an exclusive room later.

Also, it’s important to mention the steps that threads go through during this process, using the same analogy:
- entering the building – entering the monitor
- entering the exclusive room – acquiring the monitor
- being in the exclusive room – owning the monitor
- leaving the exclusive room – releasing the monitor
- leaving the building – exiting the monitor.

Luckily, Java does most of the work in the background, and we don’t need to write semaphores when we’re dealing with multi-threaded applications. Therefore, the only thing we need to do is wrap our critical section with the [[Synchronized Keyword|synchronized keyword]] and it momentarily becomes a monitor region.
# `wait()` and `notify()`
[[wait() and notify()]] are key methods in Java used in synchronized blocks that enable collaboration between threads.

**`wait()` orders the calling thread to release the monitor and go to sleep until some other thread enters this monitor and calls `notify()`**.  Also, `notify()` wakes up the first thread that called `wait()` on the specific object.