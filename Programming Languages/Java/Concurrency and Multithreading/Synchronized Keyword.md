[Source](https://www.baeldung.com/java-synchronized)
Simply put, in a multi-threaded environment, a [[Race Condition|race condition]] occurs when two or more threads attempt to update mutable shared data at the same time. Java offers a mechanism to avoid race conditions by synchronizing thread access to shared data.

A piece of logic marked with `synchronized` becomes a synchronized block, **allowing only one thread to execute at any given time**.
# Why Synchronization?
Let’s consider a typical race condition where we calculate the sum, and multiple threads execute the `calculate()` method:
```java
public class SynchronizedMethods {

    private int sum = 0;

    public void calculate() {
        setSum(getSum() + 1);
    }

    // standard setters and getters
}
```

Then let’s write a simple test:
```java
@Test
public void givenMultiThread_whenNonSyncMethod() {
    final var service = Executors.newFixedThreadPool(3);
    final var summation = new SynchronizedMethods();

    IntStream.range(0, 1000)
      .forEach(count -> service.submit(summation::calculate));
    service.awaitTermination(1000, TimeUnit.MILLISECONDS);

    assertEquals(1000, summation.getSum());
}
```

We’re using an `ExecutorService` with a 3-threads pool to execute the `calculate()` 1000 times.

If we executed this serially, the expected output would be 1000, but **our multi-threaded execution fails almost every time** with an inconsistent actual output:

```java
java.lang.AssertionError: expected:<1000> but was:<965>
at org.junit.Assert.fail(Assert.java:88)
at org.junit.Assert.failNotEquals(Assert.java:834)
...
```

Of course, we don’t find this result unexpected.

A simple way to avoid the race condition is to make the operation thread-safe using the `synchronized` keyword.
# The `synchronized` keyword
We can use the `synchronized` keyword on different levels:
- Instance methods
- Static methods
- Code blocks
 
 When we use a `synchronized` block, Java internally uses a [[Computer Science/Concurrency and Multithreding/Monitor|monitor]], also known as a monitor lock or intrinsic lock, to provide synchronization. These monitors are bound to an object; therefore, all synchronized blocks of the same object can have only one thread executing them at the same time.
## `synchronized` instance methods
We can add the `synchronized` keyword in the method declaration to make the method synchronized:
```java
public synchronized void synchronisedCalculate() {
    setSum(getSum() + 1);
}
```

Notice that once we synchronize the method, the test case passes with the actual output as 1000:
```java
@Test
public void givenMultiThread_whenMethodSync() {
    final var service = Executors.newFixedThreadPool(3);
    final var method = new SynchronizedMethods();

    IntStream.range(0, 1000)
      .forEach(count -> service.submit(method::synchronisedCalculate));
    service.awaitTermination(1000, TimeUnit.MILLISECONDS);

    assertEquals(1000, method.getSum());
}
```

Instance methods are `synchronized` over the instance of the class owning the method, which means only one thread per instance of the class can execute this method.
## `synchronized` static methods
Static methods are `synchronized` just like instance methods:
```java
 public static synchronized void syncStaticCalculate() {
     staticSum = staticSum + 1;
 }
```

These methods are `synchronized` on the `Class` object associated with the class. Since only one `Class` object exists per JVM per class, only one thread can execute inside a `static synchronized` method per class, irrespective of the number of instances it has.

Let’s test it:
```java
@Test
public void givenMultiThread_whenStaticSyncMethod() {
    final var service = Executors.newCachedThreadPool();

    IntStream.range(0, 1000)
      .forEach(count -> 
        service.submit(SynchronizedMethods::syncStaticCalculate));
    service.awaitTermination(100, TimeUnit.MILLISECONDS);

    assertEquals(1000, SynchronizedMethods.staticSum);
}
```
## `synchronized` blocks within methods
Sometimes we don’t want to synchronize the entire method, only some instructions within it. We can achieve this by _applying_ synchronized to a block:

```java
public void performSynchronisedTask() {
    synchronized (this) {
        setCount(getCount()+1);
    }
}
```

Then we can test the change:
```java
@Test
public void givenMultiThread_whenBlockSync() {
    final var service = Executors.newFixedThreadPool(3);
    final var synchronizedBlocks = new SynchronizedBlocks();

    IntStream.range(0, 1000)
      .forEach(count -> 
        service.submit(synchronizedBlocks::performSynchronisedTask));
    service.awaitTermination(100, TimeUnit.MILLISECONDS);

    assertEquals(1000, synchronizedBlocks.getCount());
}
```

Notice that we passed a parameter `this` to the `synchronized` block. This is the [[Programming Languages/Java/Concurrency and Multithreading/Monitor|monitor]] object. The code inside the block gets synchronized on the monitor object. Simply put, only one thread per monitor object can execute inside that code block.

If the method was `static`, we would pass the class name in place of the object reference, and the class would be a monitor for synchronization of the block:
```java
public static void performStaticSyncTask(){
    synchronized (SynchronisedBlocks.class) {
        setStaticCount(getStaticCount() + 1);
    }
}
```

Let’s test the block inside the `static` method:
```java
@Test
public void givenMultiThread_whenStaticSyncBlock() {
    final var service = Executors.newCachedThreadPool();

    IntStream.range(0, 1000)
      .forEach(count -> 
        service.submit(SynchronizedBlocks::performStaticSyncTask));
    service.awaitTermination(100, TimeUnit.MILLISECONDS);

    assertEquals(1000, SynchronizedBlocks.getStaticCount());
}
```
## Reentrancy
**The lock behind the `synchronized` methods and blocks is a reentrant.** This means the current thread can acquire the same `synchronized` lock over and over again while holding it:
```java
final var lock = new Object();

synchronized (lock) {
    System.out.println("First time acquiring it");

    synchronized (lock) {
        System.out.println("Entering again");

         synchronized (lock) {
             System.out.println("And again");
         }
    }
}
```

As shown above, while in a `synchronized` block, we can repeatedly acquire the same monitor lock.