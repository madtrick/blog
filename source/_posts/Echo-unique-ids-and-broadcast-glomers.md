---
title: Echo, unique ids and broadcast glomers
date: 2023-04-21 16:43:20
tags:
---

This is the first post of a series I will be writing about the Gossip Glomers. What are the Gossip Glomers you might be wondering? These are some exercises to test/grow your distributed systems skills. I wrote {% post_link gossip-glomers "this other post"%} where I explain what they are about in more detail.

Since the first 3 glomers (echo server, unique id generation and broadcasting) are quite straightforward I decided to put them all together in on post. The other 3 glomers will have one post each.


# Echo server

The echo server (https://fly.io/dist-sys/1/) is the first glomer. It's meant to help you familiarize with maelstrom and the communication protocol. You have to build an echo server that replies with the value that is sent to it.

```typescript
node.on(MessageType.Echo, (node, _state, message) => {

  node.send(message.src, {
    type: MessageType.EchoOk,
    in_reply_to: message.body.msg_id,
    echo: message.body.echo,
  })
})
```

The code for this glomer can be found [here](https://github.com/madtrick/gossip-glomers/blob/master/workloads/echo.ts)

# Unique id generation

The unique id generation (https://fly.io/dist-sys/2/) is the second glomer. This time you have to build a totally available unique generation service. This exercise is also simple if we consider that the ids can be anything: strings, booleans, integers, .... It's easy to get a unique id by concatenating the node id with a per-node monotonically increasing counter. By doing this, we have a leaderless archicture where any node can reply to a request for an id.

```typescript
node.on(MessageType.Generate, (node, state, message) => {
  node.send(message.src, {
    type: MessageType.GenerateOk,
    in_reply_to: (message.body as MessageBodyGenerate).msg_id,
    id: `${node.id}-${state.counter}`,
  })

  state.counter += 1
})

```

The code for this glomer can be found [here](https://github.com/madtrick/gossip-glomers/blob/master/workloads/unique-ids.ts).

In a real life scenario an unique id generation service wouldn't have been so straight forward to build. We would have to consider crashes of nodes and the durability of the in-memory counter. We would also have to guarantee that no two nodes have the same id. It could be that a node is considered dead and a replacement is launched with the same id but the other node was just partitioned. And we would also have to take into account functional requirements like uniqueness across data centers, semantics of the ids (e.g. should they be orderable).

# Broadcast

The broadcast exercise (https://fly.io/dist-sys/3a/) is the third exercise. This exercise is split into several parts. You start with one node and extend it to an efficient multi node distributed system.

This exercise can be broken down into two parts: topology selection and broadcasting. Two satisfy the message exchange and latency requirements you can opt for picking a different topology than that chosen by Maelstrom.

```typescript
node.on(MessageType.Topology, (node, _state, message) => {
  const { body } = message as Message<MessageBodyTopology>
  const nodeIds = new Set()

  for (const key in body.topology) {
    nodeIds.add(key)
    body.topology[key].forEach(nodeIds.add.bind(nodeIds))
  }

  const idNumber = Number(node.id.substring(1))
  const neighbours = []

  if (idNumber % 6 === 0) {
    log(`[topology] node id ${node.id} is head`)

    for (let i = 1; i < 6 && idNumber + i < nodeIds.size; i++) {
      neighbours.push(`n${idNumber + i}`)
    }

    if (idNumber + 6 < nodeIds.size) {
      neighbours.push(`n${idNumber + 6}`)
    }

    if (idNumber - 6 >= 0) {
      neighbours.push(`n${idNumber - 6}`)
    }
  } else {
    const offset = idNumber % 6
    neighbours.push(`n${idNumber - offset}`)
  }

  log(`[topology] node id ${node.id} neighbours ${neighbours}`)

  node.neighbours = neighbours

  node.send(message.src, {
    type: MessageType.TopologyOk,
    in_reply_to: body.msg_id,
  })
})
```

In my solution I picked clustering nodes and allowing for bidirectional communication among the nodes of the cluster. There's also bidirectional communication between adjacent clusters.

![](broadcast-clusters.png)

With this setup the maximum number of steps to broadcast a message to all the nodes is the number of clusters + 1. The code snippet that I include above configures a topology with 6 clusters.

The broadcast exercise is the first one to simulate network partitions. To be fault tolerant the system has to be able to reach a stable state even in the presence of network failures or process crashes. Since in the glomers the process won't crash (unless you have a bug in the code) you only have to care about network failures. In the exercise we rely on a combination of a timeout and delivery ack to guarantee that a message is broadcasted. In my solutions the combination of timeout and retrials is given for free by my `Node` implementation.


```typescript
this.pendingRPCs[msgId] = {
	callback,
	timeout: setTimeout(() => {
	  log('[rpc] timeout expired')
	  if (this.pendingRPCs[msgId] === undefined) {
	    log('[rpc] reply received - not retrying')
	    return
	  }
	
	  log('[rpc] reply not received - retrying')
	  send()
	}, 1000),
}
```

```typescript
/**
* When a message is received and that message is a reply
* to a message sent from this node, we clear the timeout
* so we don't retry the delivery again
*/
if (this.pendingRPCs[data.body.in_reply_to]) {
    const callback = this.pendingRPCs[data.body.in_reply_to].callback
    callback && callback(this, this.state, data)

    clearTimeout(
      this.pendingRPCs[(data.body as unknown as ReplyMessage).in_reply_to]
        .timeout
    )

    delete this.pendingRPCs[data.body.in_reply_to]
}
```

With that we have most of the exercise solved. The other relevant pieces is making sure that we don't create and endless loop of broadcasting by ping-ponging messages between nodes.

You can find the code for the `Node` [here](https://github.com/madtrick/gossip-glomers/blob/master/node.ts) and for the broadcast workload [here](https://github.com/madtrick/gossip-glomers/blob/master/workloads/broadcast.ts).

