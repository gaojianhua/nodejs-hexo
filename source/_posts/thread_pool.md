
title: JAVA线程池基础
date: 2017-06-20 21:19:23
tags: java
categories: java
---

## Java线程池
Java线程池多种多样，本质主要是基于Java原生线程池ThreadPoolExecutor的各种封装，比如常用的newCachedThreadPool、newFixedThreadPool、newScheduledThreadPool、newSingleThreadExecutor，ScheduledThreadPoolExecutor，以及Spring封装的ThreadPoolTaskExecutor、ScheduledThreadPoolExecutor等。
<!-- more -->
这里讲ThreadPoolExecutor的几个主要参数：

- corePoolSize（核心线程数）
核心线程数，核心线程会一直存活，即使没有任务需要处理。
- maxPoolSize
最大线程数
- queueCapacity
任务队列容量，实际是一个BlockingQueue，用于存放暂时未来得及消费执行的任务
- keepAliveTime
线程空闲时间，当线程空闲时间达到keepAliveTime，该线程会退出，直到线程数量等于corePoolSize。
- allowCoreThreadTimeout
是否允许核心线程空闲退出，默认值为false。当置为true时，核心线程的空闲时间小于keepAliveTime时，也会退出。

## 线程池线程数变化过程
- 一个任务提交，当线程数小于核心线程数时，创建线程执行任务；
- 当线程数大于等于核心线程数，且任务队列未满时，将任务放入任务队列；
- 当线程数大于等于核心线程数，且任务队列已满时，若线程数小于最大线程数，创建线程执行，此时执行的是最新提交的这个任务，所以，如果说队列满了以后，任务还是源源不断的提交过来，队列中任务可能得不到执行；
- 当线程数大于等于核心线程数，且任务队列已满时，若线程数等于最大线程数，抛出异常，拒绝任务，如果用户设置了RejectedExecutionHandler，会将任务甩给Handler处理



## 提交任务被拒绝怎么办？
- 首先，把被拒绝当做一种生活中的正常状态。
- 其次，提交被拒绝时如果没有定义RejectedExecutionHandler，会抛出RejectedExecution，如果定义了RejectedExecutionHandler，任务会被移交给RejectedExecutionHandler处理。java提供了四种默认的处理Handler：
- ThreadPoolExecutor.AbortPolicy()抛出java.util.concurrent.RejectedExecutionException异常 终止策略是默认的饱和策略；
- ThreadPoolExecutor.CallerRunsPolicy()当抛出RejectedExecutionException异常时，会调rejectedExecution方法 调用者运行策略实现了一种调节机制，该策略既不会抛弃任务也不会爆出异常，而是将任务退回给调用者，实际会在调用者的线程中把任务执行掉，从而降低新任务的流量
- ThreadPoolExecutor.DiscardOldestPolicy()抛弃旧的任务；当新提交的任务无法保存到队列中等待执行时将抛弃最旧的任务，然后尝试提交新任务。如果等待队列是一个优先级队列，抛弃最旧的策略将导致抛弃优先级最高的任务，因此AbortPolicy最好不要和优先级队列一起使用。
ThreadPoolExecutor.DiscardPolicy()抛弃当前的任务
- 注意(敲黑板)
设置 maxPool、corePool、RejectedExecutionHandler等值的操作，都必须在调用initialize()方法之前，否则不会生效。
最后的最后，补一张 Spring的定时框架@Scheduled里面三个参数的差别(图来源于图片原文)：

![三个参数的差别](https://upload-images.jianshu.io/upload_images/5935687-f933ad295f314775.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
