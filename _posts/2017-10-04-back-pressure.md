---
layout: post
keywords: reactive, programming, rxjava, backpressure
description:  响应式编程中生产者消费者速度不一致的应对方式
title: "响应式编程中生产者消费者速度不一致的应对方式"
categories: [reactive-programming]
tags: [reactive-programming, rxjava]
group: archive
icon: file-alt
---
{% include codepiano/setup %}

参考：
1. [Backpressure](https://github.com/ReactiveX/RxJava/wiki/Backpressure)

在使用 rxjava 的过程中不可避免的会遇到上游的 Observable 生产数据的速度大于下游 Observer 消费的速度，这个时候需要选择一些策略来解决这个问题，按照 rxjava 官方 wiki 的说明，应对方式有三种，分别为丢弃或缓存、阻塞、back-pressure。

### hot Observable 和 cold Observable

在了解这三种应对方式之前需要先了解 hot Observable 和 cold Observable，因为有些应对方式适用于不同类型的 Observable。

hot Observable: Observable 在创建完成时就可以发出数据，Observer 收到的数据序列是从订阅关系建立这一刻起的子序列，并且 Observable 按照自己的速度发出数据，Observer 无法控制发出数据的速度，必须自己解决消费速度问题，比如用户鼠标和键盘事件、系统事件或股票价格。

cold Observable: Observable 发出的数据序列是固定的，可以按照 Observer 的需要的时间和速度发出数据，比如数据库查询出的结果集、文件检索或网络请求。但是需要注意的是当一个 cold Observable 是 multicast (转换为一个 ConnectableObservable 并且 connect() 方法被调用过) 的，那么它应该被看作 hot Observable。

对于 cold Observable 的生产者速度过快问题，可以使用 back-pressure 策略，而 hot Observable 需要使用丢弃或者缓存。阻塞的情况比较特殊，下面再讲。

#### 1. 丢弃或者缓存

通过 Operator 来缓存或者丢弃 Observable 发出的数据，使其速度小于消费者消费的速度，这样就可以不使用 backpressure 策略来让 Observable 降速。

*丢弃:*

使用 sample() 或者类似功能的 Operator throttleLast()、throttleFirst() 等，来过滤生产者发出的数据，把最终到达 Observer 的数据控制在一个合适的范围之内，被过滤掉的数据会被丢弃。比如某些不重要的日志，如果生产者速率过快，可以采取这种策略，丢掉部分日志。也可以作为某些不重要服务的降级策略。

*缓存:*

使用 buffer() 或者 window()  Operator 缓存 Observable 发出的数据，稍后把这些数据批量发出，再由消费者决定怎么消费这些数据。

生产者发出的数据是均匀分布的：

假如生产者在 10 秒钟内均匀发出了需要保存到数据库的 10000 条用户数据，由于发出数据的速度过快，超过下游消费者保存到数据库的速度，那么可以用 buffer()  Operator 将这 10000 条数据按照 100 毫秒为时间段收集为 100 个 集合，每个集合包含该时间段内的全部数据。然后再把这些集合发送给消费者，消费者可以选择把这些数据批量保存到数据库或者进行其他处理，从而达到降低速度的目的。

生产者发出的数据不是均匀分布的：

官方文档中介绍了 buffer() 配合 debounce() 来应对突发型数据的方法，假如生产者发出的数据不是均匀的，而是爆发式的，那么需要用 debounce()  Operator 来监控 Observable ，在发现过了一个固定时间后 Observable 没有发出任何数据，就把当前 buffer() 内的数据作为一个集合发送给消费者。具体可以看官方的图，很直观。

#### 2. 阻塞

通过调用栈阻塞的方式，阻塞 Observable，使其无法发出数据。这种方式有一个缺点是违背了『响应式』的初衷和非阻塞模型，但是假如被阻塞的线程不重要，可以被安全的阻塞（不影响其他部分），那么这也是一个可选的方案，不过现在的 rxjava 提供的 Operator 不会利用这种方式。

如果一个 Observable，所有操作它的 Operator ，所有订阅它的 Observer 都是在同一个线程上，那么实际上是通过调用栈阻塞形成了一种阻塞 back-pressure。不过要注意很多 Operator 默认情况下是在不同的线程，文档中有说明。

#### 3. 背压(back-pressure)

消费者通过 "reactive pull" 来把生产数据速度过快的问题上移到生产者那里，让生产者去解决问题。

这个词在网络上有很多解释，这里只贴一下 ReactiveManifesto 术语表中的解释。

```
When one component is struggling to keep-up, the system as a whole needs to respond in a sensible way. It is unacceptable for the component under stress to fail catastrophically or to drop messages in an uncontrolled fashion. Since it can’t cope and it can’t fail it should communicate the fact that it is under stress to upstream components and so get them to reduce the load. This back-pressure is an important feedback mechanism that allows systems to gracefully respond to load rather than collapse under it. The back-pressure may cascade all the way up to the user, at which point responsiveness may degrade, but this mechanism will ensure that the system is resilient under load, and will provide information that may allow the system itself to apply other resources to help distribute the load, see Elasticity.
```

当系统中某些组件消费速度跟不上生产者生产的速度时，如果不想让组件崩溃或者以一种无法控制的方式丢弃消息，那么需要一种机制来把消费者组件的压力向上传递给上游的生产者组件，让生产者来减轻消费者的负荷。back-pressure 就是这样的一个让系统可以优雅的对负荷进行响应而不是被压垮的重要机制，把压力一直通知到用户那里，由系统维护者制定解决方案。

具体到 Operator 上就是利用 `Subscriber.request(n)` 的方式来向 Observable 请求数据，把获得数据从 push 模式转换为 pull 模式，从而控制生产者发送数据的速度。而 onBackpressureBuffer()、onBackpressureDrop() 等是为了应对没有实现 reactive pull 模式的 Observable 实现的辅助 Operator 。
