---
title: why you shouldn't build a distributed system
date: 2023-08-26 20:47:07
tags:
---

!!! info ""

    I [originally posted](https://www.contentful.com/blog/explaining-distributed-systems/) this article on www.contentful.com

At Contentful, we recently launched new (and) cool content orchestration features. These features and others that will be released in the future require that data from different sources is pulled together, processed and delivered to the customer in a timely and highly available manner.

To build content orchestration we need different systems to cooperate and identify what content has to be composed, from where it must be fetched, and which restrictions have to be enforced. Because of this, I’ve been thinking a lot recently about distributed systems and I came to the conclusion that we shouldn’t be building them. Or should we...?

If you are also an engineer responsible for designing and building distributed systems — or are just interested in the topic — you might find this post useful.

## What is a distributed system?

Before getting into the core of the topic, I want to take a moment to get everybody on the same page with a commonly understood definition of a distributed system. Sometimes, when people think about distributed systems, the first that comes to mind are distributed storage systems like AWS DynamoDB, AWS Aurora, Google Spanner, etc.

A system doesn’t have to use consensus algorithms or replication to be considered a distributed system. Any system that is composed of different units that need to communicate over the network to function is, in fact, a distributed system. It doesn’t make a difference if those units are executing the same process or a different one; in any case, it’s a distributed system.

## Why you shouldn’t build distributed systems

You might disbelieve such a claim but I hope that the case I’ll make in this post convinces you that building a distributed system is a bad idea. Instead of just outlining a bunch of pros and cons of distributed systems, I’m going to use a simple example to drive my point home.

Imagine that you are tasked with writing the software to manage a shop. The software has to manage orders and the stock, and have some basic customer management capabilities. For the purpose of this post, let’s say that you could build it in one of these two ways:

A standalone app that runs on a tablet. All the functionality is self-contained and it doesn’t access the network. This is a non-distributed system.

A tiered application with a frontend application which uses a REST API to interact with the backend. The backend is designed as a collection of microservices. This is a distributed system.

The first thing we want to do is look closely at both applications since we will be referring to their architecture throughout the post.

The standalone app is a monolith, with three modules for each of the required functionalities. It runs exclusively on the tablet and makes no network calls.

![overview monolith](ink__41_.avif)

The tiered application has a hybrid execution mode. Part of the application executes locally in the client and part of it executes in the cloud. The frontend communicates with the backend using a REST API. The backend is composed of several microservices which collaborate to satisfy the customer request.

Part of the application executes locally in the client and part of it executes in the cloud. The frontend communicates with the backend using a REST API. The backend is composed of several microservices which collaborate to satisfy the customer request.

![overview distributed](ink__42_.avif)

For simplicity, I’ll leave out the frontend app in most of the following diagrams of the distributed system.

## Six reasons why you shouldn’t build a distributed system

### Reason one: Partial failures

Non-distributed system: If the tablet where the application runs is overloaded, doesn’t have more disk space, doesn’t turn on, etc., then the functionality of our application will be affected equally: either it works (albeit possibly slowly) or it doesn’t. While this means that the system has a single point of failure, it also means that it’s easier to debug and reason about its state.

![partial failures](ink__43_.avif)

**Distributed system**: The distributed app is composed of different systems that run on different machines. Any of these machines can have any of the failures that we mentioned for the non-distributed system: it’s overloaded, doesn’t have more disk space, it doesn’t turn on, etc.

Since these failures can happen to only one or some of the units in the system, the system can have partial failures. Some functionality of the application might be working, while some might not, with unpredictable consequences for the system as a whole. We say that, in this case, the different functionalities offered by the system don’t [share a fate](https://en.wikipedia.org/wiki/Fate-sharing), since one of them having issues doesn’t have to have a direct impact on the rest.

![not fate sharing](ink__44_.avif)

### Reason two: Harder to code and to test

**Non-distributed system**: This code could be what the system needs to create an order, update the stock, and track the customer action.

```typescript
function processOrder(input: { type: number; customerId: number }): void {
  const order = Orders.create(type, customerId);

  Stock.decrementLemonStock(order.lemonQty);
  Stock.decrementSugarStock(order.sugarQty);

  CRM.track(order);
}
```

Testing this code would also be easy. It could be enough to validate that the code responded correctly to the input params:

```typescript
it('processes the order with the right params', () => {...})
it('rejects the order with the wrong params', () => {...})
```

**Distributed system**: In our distributed system the creation of an order is split across the coordination service and the orders service:

```typescript
// In the coordinator service
function processOrder(input: {
  type: number;
  customerId: number;
}): Promise<void> {
  const order = await ordersClient.create(type, customerId);
  // more code would follow
}

// in the Orders service
function createOrder(input: {
  type: number;
  customerId: number;
}): Promise<void> {
  const order = Orders.create(type, customerId);

  await stockClient.decrementLemonStock();
  await stockClient.decrementSugarStock();

  await crmClient.track(order);

  // more code would follow
}
```

This is already a bit more complicated than the standalone app, but nothing surprising here. We use clients to interact with the different services to hide the details of making requests over the network. The logic which was originally in one place is now spread across multiple services, making it more difficult to comprehend how requests are executed.

Testing is also noticeably more complex. Besides testing for valid or invalid input params we have to also test for the behavior of the system when there are issues with the communication between services. And this has to be repeated on all services communicating with other services.

```typescript
// input params are valid
it(‘creates an order (handle ConnectionError)’, () => {...})
it(‘creates an order (handle SerializationError)’, () => {...})
it(‘creates an order (handle RateLimitError)’, () => {...})
it(‘creates an order (handle TimeoutError)’, () => {...})
```

### Reason three: Unknown unknowns

**Non-distributed system**: The system works entirely or fails entirely (all the functionalities are fate-sharing). This helps to reason about the state of the system and simplifies the design.

**Distributed system**: Distributed systems have partial failures which translates into situations where the state of the system is unknown. “What happens if the Stock service crashes while processing a request? Was the order created? Was the stock updated?” Since the end result is unknown, the system doesn’t have a straightforward solution:

- “Can we retry the order? Or would we be tipping an overloaded system over the failure mode?”
- “Can we retry the order? Or would that result in charging the customer twice?”
- “Can we retry the order?

### Reason four: Everything has its own clock

**Non-distributed system**: All the different functionalities share the same clock. You could produce a reliable total ordering of the events that happened on the system.

![clocks](ink__45_.avif)

**Distributed system**: In a distributed system, each component has its own view of time. Even when using solutions like NTP, the time across the system will be bound to drift. Therefore, [we can’t rely on wall clocks to order the events](https://queue.acm.org/detail.cfm?id=2745385) that happen on a distributed system.

![clocks distributed](ink__46_.avif)

### Reason five: Consistency

**Non-distributed system**: In the non-distributed system, you always have the guarantee that you are seeing the latest data and that it’s consistent with the application’s business rules. We can save an order and all the updates to the stock and CRM automatically.

**Distributed system**: In the distributed system, the data for Orders, Stock, and CRM are kept on separate storage systems, each with their own persistence mechanisms. Two requests to get and order could get contradictory results if, for example, the Stock storage eventually uses a consistent database.

![consistency](ink__47_.avif)

## Why are distributed systems complex?

You might have already seen what’s the commonality to the six reasons I outlined. The reason why distributed systems are complex is what makes them distributed in the first place: the network.

While the network is reliable most of the time (especially within one data center) it sometimes fails and when it doesn’t fail, we don’t have any guarantees about how long it’s going to take to deliver and process a message. Without these guarantees, we can’t know for certain if a request was:

- delivered and processed,
- delivered but not processed, or
- not delivered at all.

Our systems have to be able to operate within this uncertain environment and tolerate failures in the different components that make the system.

Because the network can fail or add too much latency to the request-response cycle we have to decide how the system behaves in those cases: do we favor consistency of the data, or do we favor responding to customer requests at all times (high availability) and fixing consistency later? In other words, do we favor consistency or fast responses?

## So, no more distributed systems, right?

At this point, I might have convinced you that the non-distributed application used in the example is a better choice and that we should throw away our microservices and client-server applications.

After all, I believe I’ve made some very good points as to why non-distributed applications are simpler to build, maintain, and understand — and why distributed systems are complex to build, complex to understand, and can fail in complex ways.

However, we wouldn’t get very far if we built only standalone applications. Yes, they would be better in some aspects, but they would also be less powerful and we would be limited in how feature-rich they could be.

For example, without distributed systems, there wouldn’t be Contentful, or Contentful itself wouldn’t be able to scale as it does to sustain our growth using a [ microservices architecture ](https://www.contentful.com/r/knowledgebase/microservice-architecture/).

## Why we should learn how to build a distributed system

Now it’s time to make a confession. I picked a provocative title for this post to get your attention and I apologize for it. The real title should have been, “Why we should learn how to build distributed systems.”

We are so used to distributed systems (mainly client-server) that we are oblivious to the problems outlined here. However, these problems are not going away or going to get any easier to solve in the future, so instead, we should understand them well enough to make the right trade-offs and pick the right solutions when building them.

## Conclusion

Distributed systems are a necessary tool but they are more complex than what they might initially seem. Knowing and understanding where the complexity originates is the first step to being able to make the right trade-offs and design choices. If you liked this topic and would like to know more about building reliable distributed systems, stay tuned, because I’ll be writing more about it, starting with how to build reliable distributed systems.

## Further reading

If you want to know more about why distributed systems are complex, I recommend that you learn more about the following:

**Network models**: These models represent abstractions that designers use when building distributed systems and algorithms. There are three models: [`asynchronous`, `synchronous` and `partially asynchronous`](https://decentralizedthoughts.github.io/2019-06-01-2019-5-31-models/).

**Failure models**: Similar to the network models, these represent abstractions about how systems [can fail](https://alvaro-videla.com/2013/12/failure-modes-in-distributed-systems.html).

**Message delivery guarantees**: You may have come across terms like `at-most-once`, `at-least-once` and `exactly-once`. They refer to the guarantees a system offers with regard to message delivery over the network. There’s controversy whether `exactly-once` is [possible](https://medium.com/@jaykreps/exactly-once-support-in-apache-kafka-55e1fdd0a35f) or [not](https://www.the-paper-trail.org/post/2017-07-28-exactly-not-atomic-broadcast-still-impossible-kafka/) (other sources for [not](https://lobste.rs/s/ecjfcm/why_is_exactly_once_messaging_not#c_e5qcdf) and [not](https://bravenewgeek.com/you-cannot-have-exactly-once-delivery/)).

**FLP impossibility**: This [research](https://groups.csail.mit.edu/tds/papers/Lynch/jacm85.pdf) has been one of most relevant and influential in the distributed systems space. It basically says that in an asynchronous network, we can’t have a deterministic consensus algorithm. This doesn’t mean that we can’t build such algorithms but it help designers understand [which limitations exist and which tradeoffs are available](https://levelup.gitconnected.com/practical-understanding-of-flp-impossibility-for-distributed-consensus-8886e73cdfe5).

**CAP theorem**: Most likely you have heard about the [CAP theorem](https://www.the-paper-trail.org/page/cap-faq/). However, you not know that its [author wrote in 2012](https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed/) that designers should not be limited by the [constraints it imposes](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html). Almost at the same time the [PACELC theorem](http://cs-www.cs.yale.edu/homes/dna/papers/abadi-pacelc.pdf) was proposed as an extension to the CAP theorem, going further into what trade-offs exists when building distributed systems.
