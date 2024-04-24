[Source](https://www.baeldung.com/cs/monitor#monitor-features)
A *monitor* is a synchronization mechanism that allows threads to have:
- _mutual exclusion_ – only one thread can execute the method at a certain point in time, using _locks_
- _cooperation_ – the ability to make threads wait for certain conditions to be met, using _wait-set_

Why is this feature called “monitor”? Because **it monitors how threads access some resources.**
# Features
Monitors provide three main features to the concurrent programming:
- only one thread at a time has mutually exclusive access to a critical code section
- threads running in a monitor could be blocked while they’re waiting for certain conditions to be met
- one thread can notify other threads when conditions they’re waiting on are met