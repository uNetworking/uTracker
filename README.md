# MasterNode
WIP - uWS pub/sub cluster master node

## Usage
A very simple set-up consists of 3 steps:

1. Deploy the (one) MasterNode executable
2. Mark your uWS instances as workers of this MasterNode (functionality to be added to uWS itself).
3. Use uWSClusterConnector library in the producer to easily publish messages to this uWS cluster.

Both producer and workers need to know the IP of the MasterNode.

## Problem with Redis
Many users of uWS pub/sub use Redis to distribute a message from the producer, to many uWS instances. This works fine for small cases but introduces an extra hop and bottlenecks every single message at the Redis instance, here marked red:

![](With%20Redis.png)

## A distributed solution
Instead of copying every single message via Redis, the idea with MasterNode (name pending!) is to allow an easy-to-setup distributed alternative where no single instance becomes a bottleneck:

![](Without%20Redis.png)

Producers send directly to the uWS instances and use only a simple bookkeeping node, MasterNode, for keeping track of available uWS instances. Compared with Redis, MasterNode does not receive any significant traffic - it merely keeps track of online uWS instances.

## Reducing internal traffic
Even though the source(s) and uWS instaces communicate over internal LAN, one can still reduce unnecessary traffic. Because the uWS instances are directly connected to the source(s), the two parts can exchange subscription lists. By doing so, the source(s) can know which uWS instance(s) it should send a publish to. This is an optimization that really only makes sense for large amounts of uWS instances as it is typically not a problem sending LAN traffic to a handful endpoints.

Whenever a subscription takes place in an uWS instance, and this topic is new for the uWS instance, it sends the new topic name to all connected sources. If the source goes offline, on reconnect the whole subscription list is sent again. Same goes for unsubscription; if the last subscriber to a topic unsubscribes, this removed topic name is sent to all connected sources so that they know not to send to this uWS instance.

The sources then need a tree where it can map outgoing publishes to uWS instances in O(log n). This is implemented in the helper library for Node.js (uWSClusterConnector) and should support wildcard supscriptions.
