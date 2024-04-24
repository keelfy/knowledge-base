[Source](https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/How-the-Actor-Model-works-by-example)
The Actor Model is a style of software architecture in which the basic computational unit is called an actor.
# Actor vs. classes
What differentiates an actor from a class are the following set of principles:
- Actors run concurrently.
- Actors send messages asynchronously.
- Actors do not share state.
- The location of an actor is transparent.
### Scalability through concurrency
The Actor Model is intended for very large systems that support a great many users simultaneously. As such, actors are independent and asynchronous. A typical use case well-suited for the Actor Model is a shopping cart in an e-commerce application.

At scale, such an application quite conceivably can have thousands or millions of users, each with their own shopping cart. In the Actor Model, each shopping cart is represented as a distinct actor among many that run simultaneously.

In order to work efficiently the shopping carts must run concurrently and not create blockage or race conditions. Thus, each actor is an asynchronous actor that does work when it can, as it can with no direct dependency on the state or activities of other actors.
### Loose coupling through asynchronous messaging
Unlike class-to-class interactions — in which one class can create an instance of another class and then call public methods and properties on that other class directly — under the Actor model all communication between actors is conducted asynchronously via messages.

There is no other means of communication.

Typically, some type of message broker technology manages messaging activity between actors. Some frameworks such as Akka abstract the message broker via configuration thus making it opaque to underlying actors.
### State isolation
Under the Actor Model, the state of each actor is separate from all other actors in the system. This makes sense; actors are asynchronous by nature.

It’s quite conceivable to have a system in which many continuously changing actors operate concurrently, all the time. Thus, it’s difficult if not impossible for one actor to share the state of another actor at any given moment. If things change too quickly, [[Race Condition|race conditions]] can occur.

A better approach is to make each actor responsible for its own state, and also to communicate information that might be relevant to other actors within the payload of an asynchronous message.
### Location transparency
In the Actor Model, actors can be distributed among any number of host computers within a given geography.

For example, Actor A can be hosted on a computer running in the western US while Actor B runs on a computer in northern India.

# Understanding messaging
As mentioned above, all communication between actors is conducted via [[Communication Patterns#Asynchronous Microservice Communication|asynchronous messaging]]. There are three modes of message interaction:
1. Fire and forget it.
2. Request and response mode.
3. Message forwarding.

Fire and forget occurs when Actor A sends a message to Actor B, then Actor A moves forward with its execution expecting no response from Actor B.

The response mode is when Actor A sends a message to Actor B but does expect a reply from Actor.

Message forwarding is when Actor A sends a message to Actor B along with instructions to forward a response message to Actor C.

In terms of asynchronous communication, at the conceptual level, message order is not guaranteed. Also, the speed of message delivery between actors is arbitrary.
