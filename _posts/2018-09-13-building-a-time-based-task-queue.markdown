---
layout: post
title: Building a time based task queue
date: 2018-09-13 10:00:00 +0300
description: Building a highly concurrent time based queue
img: workflow.jpg
tags: [Programming, Java, Concurrency, Fibers, Channels]
---
<!-- The content of this blog  -->
In the world of asynchronous task execution, we have to do execute tasks in the background. Some of this tasks need to be executed at specific times and no sooner. Traditionally we use rabbitmq as our queue, but seemingly ttl specification for rabbitmq does not seem to actually fit  this scenario as the queue recipe we want to achieve is as below:<br/>
**Queue recipe<br/>**:

> 1. At least once delivery
> 2. Acknowledge delivery of messages
> 3. Jobs are picked when only when their time of execution has arrived
> 4. Highly concurrent(millions of jobs need to be executed)

So I set out to build [Milau](https://github.com/kigsmtua/milau) a task queue that implements  the above recipe. Milau ships a client and a worker module that does all the work for you.<br/>

**How do you build a time based queue<br/>**

To build a time based queue you need to know when your jobs are due. After much brainstorming, I figured we need to store the elements in a data structure sorted by time, meaning on fetch you check for the elements that are between 0 and the current time , fetching only the jobs that are ready for execution at a specific time.

Awesome my first idea was to use rabbitmq and store my elements in a sorted set in the application, on restart of the application all you have to do is rebuild the sorted set on restarts such that you remain with your sorted jobs in memory and you can pick them up again. Simple enough , fast enough not much effort required apart from rebuilding the index

This could look like below:<br/>

```java
 import java.util.SortedSet;
 import java.util.TreeSet;
 import com.rabbitmq.client.Connection;
 import com.rabbitmq.client.Channel;

 class JobStore {
   SortedSet<String> jobs = new TreeSet<>();
   public void initialize() {
     Job job = rabbitmq.getJobs();
     jobs.add(job.getDelay)
   }
 }
```

This in itself presents a problem, how big can does this queue have to grow in memory before it becomes to expensive, considering we are doing a lot of transactions per seconds and jobs could be scheduled to happen up to 30 days from now

This hence proves not to be a too optimal selection at scale.<br/>

**Choosing redis<br/>**

Redis architecture lends nicely to queue design as it provides a set of data structures that can be used to schedule messages without coupling the application to this process as with the above example<br/>

**Impelementing the queue recipe<br/>**

Milau consists of two parts a client module, this module adds items to the Queue based on the time it needs to be executed<br/>

*how the client module is build<br/>**

For each task 3 queues are created
> 1. A sorted set containing elements stored by score (the score is what we call taskId) - jobs-queue
> 2. A Hash set that contains the the message payload, the key is the the taskId generated above job-payloads-queue
> 3. A sorted set containing messages consumed by the worker yet to be Acknowledged , a jpb runs periodically requing the jobs to the job queue if they are not acked after 5 minutes -jobs-ack-queue

A job in milau is a class that implements Runnable such a below<br/>

```java
@Task(
   queueName = "my-task-queue"
)
Class MyJob implements Runnable {

   private String name;

   public String getName() {
     return this.name;
   }
   public void setName(String name) {
     this.name = name;
   }
   public void run () {
       try {
            /// Sleep for some time to simulate execution
            Thread.sleep(1);
        } catch (InterruptedException e) {
      }
   }
}
```
The @Task annotation declares where the job is to be queued. We then instantiate a client which has a connection to redis and tell it to schedule the job for execution at a particular time<br/>

```java
Config config = new Config.ConfigBuilder("127.0.0.1", 6379).build();

Map<String , Object> jopProperties = new HashMap<>();
jopProperties.put("name", "johnDoe");

Client client = new Client(config);
client.enqueue(null,  MyJob.class, jopProperties, 0);
```

The client enqueue does the following

> 1. The last parameter is the milliseconds from now that the job is to be executed  
> 2. Adds the message to the sorted set with the value from step one
> 3. Add the message payload(properties and job class to be excuted) into a redis hashed set the key is the delay time gotten from step one.<br/>

**how the worker module is build is build<br/>**

The worker  polls for the queue by getting the messages ready for execution. The messages ready for execution are gotten as below.
```java
/**
 * Get the messages that are ready to execute. Messages only leave the queue
 * once they are ready for execution so we check for the message between
 * zero and current time
 *
 * @return
 */
protected Set getReadyTasks() {
    long currentTime = System.currentTimeMillis();
    return jedis.zrangeByScore(this.queue, 0, Double.valueOf(currentTime));
}

```
Meaning jobs are only picked when ready for execution
On getting the ready jobs, the worker follows the below steps
> 1. Add the taskID to unack set and remove from the sorted set for the queue.
> 2. If the previous step succeeds, retrieve the message payload from the Redis set based on taskId
> 3. Spins up a number of threds the threads spinned are gotten from the below function connections = ((core_count * 2) + effective_spindle_count).If we have fewer jobs than the result of the function, We use the the number of jobs gotten as the number of threads to spin.Otherwise we use the result of the function as as shown below

```java
int processorCount = Runtime.getRuntime().availableProcessors();

int optimalPoolSize  = (processorCount * 2) + 2;

ExecutorService executor;

if (tasks.size() <= optimalPoolSize) {
    executor = Executors.newFixedThreadPool(tasks.size());
} else {
    executor = Executors.newFixedThreadPool(optimalPoolSize);
}
```

**What next**<br/>

I intent to use the [Quarsar](https://github.com/puniverse/quasar) library to use fibers instead of traditional threads  This will have jobs defined as Fibers such as below

```java
final Fiber<Integer> fiber = new Fiber<Integer>(){
     @Override
     protected Integer run() throws SuspendExecution {
         Fiber.park(100, TimeUnit.MILLISECONDS);
         return 123;
     }
 };
```
Will this improve perfomance ?(Follow me as I find out!)
