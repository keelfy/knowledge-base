[Source](https://www.baeldung.com/java-wait-notify)
In a multithreaded environment, multiple threads might try to modify the same resource. Not managing threads properly will of course lead to consistency issues.
# Guarded Blocks
One tool we can use to coordinate actions of multiple threads in Java is guarded blocks. Such blocks keep a check for a particular condition before resuming the execution.

With that in mind, we’ll make use of the following:
- _[Object.wait()](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html#wait())_ to suspend a thread
- _[Object.notify()](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html#notify())_ to wake a thread up

We can better understand this from the following diagram depicting the life cycle of a `Thread`:
![[Pasted image 20240424154806.png]]

Please note that there are many ways of controlling this life cycle. However, in this article, we’re going to focus only on `wait()` and `notify()`.
# The `wait()` Method
Simply put, calling `wait()` forces the current thread to wait until some other thread invokes `notify()` or `notifyAll()` on the same object.

For this, the current thread must own the object’s [[Computer Science/Concurrency and Multithreding/Monitor|monitor]]. According to [Javadocs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html#notify()), this can happen in the following ways:
- when we’ve executed `synchronized` instance method for the given object
- when we’ve executed the body of a `synchronized` block on the given object
- by executing `synchronized static` methods for objects of type `Class`

**Note that only one active thread can own an object’s monitor at a time.**

This `wait()` method comes with three overloaded signatures. Let’s have a look at these.
## `wait()`
The `wait()` method causes the current thread to wait indefinitely until another thread either invokes `notify()` for this object or `notifyAll()`.
## `wait(long timeout)`
Using this method, we can specify a timeout after which a thread will be woken up automatically. A thread can be woken up before reaching the timeout using `notify()` or `notifyAll()`.

Note that calling `wait(0)` is the same as calling `wait()`.
## `wait(long timeout, int nanos)`
This is yet another signature providing the same functionality. The only difference here is that we can provide higher precision.

The total timeout period (in nanoseconds) is calculated as $1.000.000*timeout + nanos$.
# `notify()` and `notifyAll()`
We use the `notify()` method for waking up threads that are waiting for access to this object’s monitor.
There are two ways of notifying waiting threads.
## `notify()`
For all threads waiting on this object’s monitor (by using any one of the `wait()` methods), the method `notify()` notifies any one of them to wake up arbitrarily. The choice of exactly which thread to wake is nondeterministic and depends upon the implementation.

Since `notify()` wakes up a single random thread, we can use it to implement mutually exclusive locking where threads are doing similar tasks. But in most cases, it would be more viable to implement `notifyAll()`.
## `notifyAll()`
This method simply wakes all threads that are waiting on this object’s monitor.

The awakened threads will compete in the usual manner, like any other thread that is trying to synchronize on this object.

But before we allow their execution to continue, always **define a quick check for the condition required to proceed with the thread.** This is because there may be some situations where the thread got woken up without receiving a notification (this scenario is discussed later in an example).
# Sender-Receiver Synchronization Problem
Now that we understand the basics, let’s go through a simple _Sender–Receiver_ application that will make use of the `wait()` and `notify()` methods to set up synchronization between them:

- The `Sender` is supposed to send a data packet to the `Receiver`.
- The `Receiver` cannot process the data packet until the `Sender` finishes sending it.
- Similarly, the `Sender` shouldn’t attempt to send another packet unless the `Receiver` has already processed the previous packet.

Let’s first create a `Data` class that consists of the data `packet` that will be sent from `Sender` to `Receiver`. We’ll use `wait()` and `notifyAll()` to set up synchronization between them:
```java
public class Data {

    private String packet;
    
    // True if receiver should wait
    // False if sender should wait
    private boolean transfer = true;
 
    public synchronized String receive() {
        while (transfer) {
            try {
                wait();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt(); 
                System.err.println("Thread Interrupted");
            }
        }
        transfer = true;
        
        String returnPacket = packet;
        notifyAll();
        return returnPacket;
    }
 
    public synchronized void send(String packet) {
        while (!transfer) {
            try { 
                wait();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt(); 
                System.err.println("Thread Interrupted");
            }
        }
        transfer = false;
        
        this.packet = packet;
        notifyAll();
    }
}
```

Let’s break down what’s going on here:
- The `packet` variable denotes the data that is being transferred over the network.
- We have a boolean variable `transfer`, which the `Sender` and `Receiver` will use for synchronization:
    - If this variable is `true`, the `Receiver` should wait for `Sender` to send the message.
    - If it’s `false`, `Sender` should wait for `Receiver` to receive the message.
- The `Sender` uses the `send()` method to send data to the `Receiver`:
    - If `transfer` is `false`, we’ll wait by calling `wait()` on this thread.
    - But when it is `true`, we toggle the status, set our message, and call `notifyAll()` to wake up other threads to specify that a significant event has occurred and they can check if they can continue execution.
- Similarly, the `Receiver` will use the `receive()` method:
    - If the `transfer` was set to `false` by `Sender`, only then will it proceed, otherwise we’ll call `wait()` on this thread.
    - When the condition is met, we toggle the status, notify all waiting threads to wake up, and return the data packet that was received.

## Why Enclose `wait()` in a `while` Loop?
Since `notify()` and `notifyAll()` randomly wake up threads that are waiting on this object’s monitor, it’s not always important that the condition is met. Sometimes the thread is woken up, but the condition isn’t actually satisfied yet.

We can also define a check to save us from spurious wakeups — where a thread can wake up from waiting without ever having received a notification.
## Why Do We Need to Synchronize `send()` and `receive()` Methods?
We placed these methods inside `synchronized` methods to provide intrinsic locks. If a thread calling `wait()` method does not own the inherent lock, an error will be thrown.

We’ll now create `Sender` and `Receiver` and implement the `Runnable` interface on both so that their instances can be executed by a thread.

First, we’ll see how `Sender` will work:
```java
public class Sender implements Runnable {

    private Data data;
 
    // standard constructors
 
    public void run() {
        final var packets[] = {
          "First packet",
          "Second packet",
          "Third packet",
          "Fourth packet",
          "End"
        };
 
        for (final var packet : packets) {
            data.send(packet);

            // Thread.sleep() to mimic heavy server-side processing
            try {
                Thread.sleep(ThreadLocalRandom.current().nextInt(1000, 5000));
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt(); 
                System.err.println("Thread Interrupted"); 
            }
        }
    }
}
```

Let’s take a closer look at this `Sender`:

- We’re creating some random data packets that will be sent across the network in `packets[]` array.
- For each packet, we’re merely calling `send()`.
- Then we’re calling `Thread.sleep()` with random interval to mimic heavy server-side processing.

Finally, let’s implement our `Receiver`:
```java
public class Receiver implements Runnable {

    private Data load;
 
    // standard constructors
 
    public void run() {
        for(final var receivedMessage = load.receive();
          !"End".equals(receivedMessage);
          receivedMessage = load.receive()) {
            
            System.out.println(receivedMessage);

            //Thread.sleep() to mimic heavy server-side processing
            try {
                Thread.sleep(ThreadLocalRandom.current().nextInt(1000, 5000));
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt(); 
                System.err.println("Thread Interrupted"); 
            }
        }
    }
}
```

Here, we’re simply calling `load.receive()` in the loop until we get the last `“End”` data packet.

Let’s now see this application in action:
```java
public static void main(String[] args) {
    final var data = new Data();
    final var sender = new Thread(new Sender(data));
    final var receiver = new Thread(new Receiver(data));
    
    sender.start();
    receiver.start();
}
```

We’ll receive the following output:
```
First packet
Second packet
Third packet
Fourth packet
```

And here we are. **We’ve received all data packets in the right, sequential order** and successfully established the correct communication between our sender and receiver.

# Conclusion
It’s worth mentioning that all these low-level APIs, such as `wait()`, `notify()` and `notifyAll()`, are traditional methods that work well, but higher-level mechanisms are often simpler and better — such as Java’s native `Lock` and `Condition` interfaces (available in `java.util.concurrent.locks` package).

