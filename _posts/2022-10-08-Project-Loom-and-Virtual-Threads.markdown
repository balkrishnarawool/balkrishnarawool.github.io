---
layout: post
title: "Project Loom and Virtual Threads"
date: 2020-10-08
description: Project Loom and Virtual Threads
img: project-loom-virtual-threads.jpg
tags: [java, project-loom, virtual-threads, concurrency, threads, loom, scalability, performance]
---

[Project loom](https://openjdk.org/projects/loom/) is one of the most important projects at OpenJDK. The purpose of this project is to provide easy-to-use, high-throughput lightweight concurrency and new programming models for it. There are 3 main aspects of project loom: Virtual threads, Structured concurrency and Scoped values.

[Virtual threads](https://openjdk.org/jeps/444) are lightweight threads that dramatically reduce the effort of writing, maintaining, and observing high-throughput concurrent applications. [Structured concurrency](https://openjdk.org/jeps/453) treats groups of related tasks running in different threads as a single unit of work, thereby streamlining error handling and cancellation, improving reliability, and enhancing observability. [Scoped values](https://openjdk.org/jeps/446) enable sharing of immutable data within and across threads.

In this article, we will only discuss virtual threads. But first, let’s start with platform threads.

## Platform threads
The traditional threads which have been available since Java 1.1 are now called platform threads.
<p align="center">
<img src="../assets/img/platform-threads.png" height="200" />
</p>
Platform threads are an abstraction over Operating System (or kernel) threads. When an application requires a new platform thread, JVM requests OS to create a new OS thread and only when this OS thread is created, a new platform thread is instantiated inside JVM.
They are typically resource intensive. A relatively large amount of memory is needed to create an OS thread. That means only so many OS threads can be created. And thereby only so many platform threads can be created. If an application uses request-per-thread model (which means a new thread is created every time a new request is received) then, with platform threads, a limited number of requests can be handled by the application concurrently.
This makes platform threads a rare resource and therefore they are pooled. JVM maintains thread pools so that when a new platform thread is needed, it doesn’t have to go through the whole process of asking OS to create a new OS thread. But this comes with an added responsibility of managing these thread pools.

## Virtual threads
Virtual threads are lightweight user threads.
<p align="center">
<img src="../assets/img/virtual-threads.png" height="350" />
</p>
They are lightweight because they require much less memory than platform threads. Because of this we can create them in abundance. Perhaps even thousands of times more than platform threads. They do not have direct dependency on OS threads. So while creating a new virtual thread, JVM does not have to request OS to create a new OS thread nor does it have to look into thread-pool for an available thread. This makes creating virtual threads a quick process. As quick as creating a bunch of objects! And because virtual threads are so cheap to create, we don’t have to pool them. No overhead of managing thread-pools.

## Scheduler and Mounting/unmounting of virtual threads
Virtual threads always need a platform thread to execute a piece of code. This process is called mounting and the platform thread is called the carrier thread of the virtual thread. So when a new virtual thread is created, it is mounted on a platform thread. If, at any point in time in future, the virtual thread is blocked (e.g. if it has to wait for a resource to be available), the virtual thread is then unmounted from the platform thread. At this point, the platform thread is free to have another virtual thread mounted. That way, even if the virtual threads are blocked, the platform threads are not blocked. This means we don’t have to worry about calling blocking code from inside virtual thread.
This process of mounting and unmoving is carried by a scheduler which uses a dedicated fork-join pool in FIFO mode.

## Pinning
When a virtual thread executes a blocking call from inside a synchronized method or block of code, JVM cannot unmount the virtual thread from the carrier thread. This is called pinning. In specific cases where the blocking call is lengthy, it can lead to scalability issues. Such occurrences can be identified with JFR (JDK Flight Recorder) and the can be replaced with reentrant lock to avoid pinning.

## Summary
In this article we looked at Project Loom and, specifically, virtual threads. Virtual threads are lightweight and they provide much higher scalability than platform threads (in thread-per-request model). We should not pool virtual threads. There is a special scheduler to perform mounting/unmounting of virtual threads. Pinning can occur when synchronized code is executed by virtual thread which can be avoided by using reentrant lock.
Virtual threads are cheap to create and It’s ok to have blocking calls from within virtual thread. These two characteristics make a new concurrency model possible in Java: structured concurrency. We will learn more about that in the next article.
