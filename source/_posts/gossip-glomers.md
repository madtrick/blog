---
title: Gossip glomers
date: 2023-04-17 08:14:56
tags:
---



![image-20230417081551548](image-glomers.png)

The folks at [fly.io](https://fly.io/) have open to the public some of their [exercises](https://fly.io/dist-sys/) they use to test the system design skills and distributed sytems knowledge of their candidates. At my current employer ([www.contentful.com](www.contentful.com)) we also also have candidates do a take home exercise and for senior positions also a system design exercise, so it's very interesting seeing how others do this.

But this post isn't about hiring exercises but about distributed systems. As Fly mentions on the [blog post](https://fly.io/blog/gossip-glomers/) where the announced this exercises, knowing the theory is good but having experience building such systems is even better.

> Reading a stack of MIT PhD dissertations may be a good Friday night, but it won't equip you for debugging a multi-service outage at 2am. That requires real-world experience.

I won't repeat here what Fly.io already wrote on their [announcement blog post](https://fly.io/blog/gossip-glomers/) about the exercises. Instead I will encourage those who are interested in distributed systems to give [Gossip glomers](https://fly.io/dist-sys/) (that's the name for the exercises) a try. These exercises are the perfect sandbox to get one's hands dirty without all the overhead that running such distributed systems would require. You will get a chance to learn or brush up your skills about distributed systems. Some of the topics that are covered in the exercises are:

- Consistency models. There are a bunch of different [https://jepsen.io/consistency](https://jepsen.io/consistency) that one can consider when building data storage and distributed systems. In the exercises you will have to use sequential and linearizable stores and also reason about the different anomalies that can pop up in a single node database.
- Partial failures. How will your system behave when parts of the system are down or partitioned?
- Latencies. How efficient is your implementation when the network is slow?



So far I have only explored what's requested in the gossip glomers exercises but there's much more to learn by using Maestrom with different configuration values to test different scenarios (e.g. different consistency modes, availability expectations, latencies, etc.). I'll follow this post with a series of other posts where I'll talk about the different exercises and how I solved them.

## What's a glomer?

When I first read the announcement blog post I could connect the "gossip" part to distributed systems but couldn't make any sense of the "glomers" part. One google search later I found out that a glomer was a magical creature in a 80s tv show called Punky Brewster. These creature looks similar to the drawings in the gossip glomers exercises so that is clear.

![Figure](glomer.png)

Figure. The Glomer.

What I still don't know is what sort of pun the folks at Fly.io are referring to when they write

> **What the f$#\* is a Glomer?**
> It's an elaborate pun about the CAP theorem.

If anyone at Fly.io can explain that to me it would be great 😄
