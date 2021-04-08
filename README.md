# MasterNode
WIP - uWS pub/sub cluster master node

## Usage
A very simple set-up consists of 3 steps:

1. Deploy the (one) MasterNode executable
2. Mark your uWS instances as workers (functionality to be added to uWS itself).
3. Use uWSClusterConnector library in the producer to easily publish messages to this uWS cluster.

## Problem with Redis
Many users of uWS pub/sub use Redis to distribute a message from the producer, to many uWS instances. This works fine for small cases but introduces an extra hop and bottlenecks every single message at the Redis instance, here marked red:

![](With%20Redis.png)

## A distributed solution
Instead of copying every single message via Redis, the idea with MasterNode (name pending!) is to allow an easy-to-setup distributed alternative where no single instance becomes a bottleneck:

![](Without%20Redis.png)

Producers send directly to the uWS instances and use only a simple bookkeeping node, MasterNode, for keeping track of available uWS instances. Compared with Redis, MasterNode does not receive any significant traffic - it merely keeps track of online uWS instances.
