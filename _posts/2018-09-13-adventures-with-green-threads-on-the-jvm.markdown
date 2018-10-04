---
layout: post
title: Adventures with green threads on the jvm
date: 2018-09-13 10:00:00 +0300
description: Building a highly concurrent time based queue
img: workflow.jpg
tags: [Programming, Java, Concurrency, Fibers, Channels]
---
<!-- The content of this blog  -->
I was recently faced with a challenge at work on how to schedule tasks and have them only
be picked up from the queue at a specified time in the future. Together with my team we figured
was we needed to store the tasks in a sorted manner and only pick elements from the top of the sorted set. Some of the constraints involved here were:
> 1. At least once delivery
> 2. Highly concurrent (we are doing millions of requests per day)
> 3. No loss of messages
> 4. Acknowledge delivery of messages

On peeking I discovered redis has sorted sets which will enable us to solve most of the challenges we face here
so the queue recipe goes as follows
>1. A Sorted Set containing queued elements by score (score is time task is to be picked up)
>2. A Hash set that contains the task with the key as the task id
>3. A Sorted Set containing messages consumed by clients yet to be acknowleged

**The birth of milau**
---
[Milau](https://github.com/kigsmtua/milau)
Milau is a distributed task queue that implements the above recipe. It comes shipped with a client and a worker module.

*Design principles*
>1. Software for humans(should be as simple to use as possible)
>2. Tiny footprint , the whole codebase is so tiny it could be understood in one sitting
>3. Highy concurrent, able to plough through millions of tasks in a breeze

Concurrency is achieved by a mixture of using channels and actors and traditional threads.
Parrarel universe have an awesome go like concurrency model for building unlimited number of threads called  fibers you can have channels as well, and actors which is a combination of channels fibers and actors. As well as traditional threds
This is where we define what a task interface actually looks like
Talk about fibers and

```java
  final Fiber<Integer> fiber = new Fiber<Integer>(){
      @Override
      protected Integer run() throws SuspendExecution {
          Fiber.park(100, TimeUnit.MILLISECONDS);
          return 123;
      }
  };
```

How to use Milau

The client can be used in a simple manner such as
```java
Config config = new Config.ConfigBuilder(localhost, 6379).withTimeout(5000)
Client client = new Client(Config)
client.enqueue(Task);
```


The worker can be configured with multiple queues, different concurrencies per queue
It is bundled along with the application as of now but can be used separately

```java
 // Define queues and concurrency for each queue
Map<String, Integer> queues = new HashMap<>();
// Instantiate a worker instance
Worker worker = new Worker(queues, config);
// Have the worker work
worker.work();
```

Fairy tales are true not because they tell us that dragons exits, but because they tell us that dragons can be beaten.
