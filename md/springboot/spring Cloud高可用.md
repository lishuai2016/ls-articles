---
title: spring Cloud高可用
categories: 
- Markdown
tags:
---

#spring Cloud高可用

主要的技术手段
- 资源隔离
- 接口调用超时设置和线程池设置
- 服务降级



目录
一、概述
二、业务场景介绍
三、线上经验 _ 如何设置Hystrix线程池大小
四、线上经验 _ 如何设置请求超时时间
五、服务降级
六、总结



## 一、概述

微服务架构本身最最核心的保障高可用的措施，就是两点：
- 一个是基于Hystrix做资源隔离以及熔断；
- 另一个是做备用降级方案。
如果资源隔离和降级都做的很完善，那么在双11这种高并发场景下，也许可能会出现个别的服务故障，但是绝不会蔓延到整个系统全部宕机。

## 二、业务场景介绍

<div align="center"> <img src="/images/spring Cloud高可用1.png"/> </div><br>

如上图，核心服务A调用了核心服务B和C，在核心服务B响应过慢时，会导致核心服务A的某个线程池全部卡死。
但是此时因为你用了hystrix做了资源隔离，所以核心服务A是可以正常调用服务C的，
那么就可以保证用户起码是可以使用APP的部分功能的，只不过跟服务B关联的页面刷不出来，功能无法使用罢了。
当然这种情况在生产系统中，是绝对不被允许的，所以大家不要让上述情况发生。
系统优化成了下图这样：
<div align="center"> <img src="/images/spring Cloud高可用2.png"/> </div><br>

要保证一个hystrix线程池可以轻松处理每秒钟的请求
同时还有合理的超时时间设置，避免请求太慢卡死线程。


## 三、线上经验—如何设置Hystrix线程池大小

好，现在问题来了，在生产环境中，我们到底应该如何设置服务中每个hystrix线程池的大小？
下面是我们在线上经过了大量系统优化后的生产经验总结：
假设你的服务A，每秒钟会接收30个请求，同时会向服务B发起30个请求，然后每个请求的响应时长经验值大概在200ms，
那么你的hystrix线程池需要多少个线程呢？
计算公式是：30（每秒请求数量） * 0.2（每个请求的处理秒数） + 4（给点缓冲buffer） = 10（线程数量）。
如果对上述公式存在疑问，不妨反过来推算一下，为什么10个线程可以轻松抗住每秒30个请求？
一个线程200毫秒可以执行完一个请求，那么一个线程1秒可以执行5个请求，理论上，只要6个线程，每秒就可以执行30个请求。
也就是说，线程里的10个线程中，就6个线程足以抗住每秒30个请求了。剩下4个线程都在玩儿，空闲着。
那为啥要多搞4个线程呢？很简单，因为你要留一点buffer空间。
万一在系统高峰期，系统性能略有下降，此时不少请求都耗费了300多毫秒才执行完，那么一个线程每秒只能处理3个请求了，
10个线程刚刚好勉强可以hold住每秒30个请求。所以你必须多考虑留几个线程。
老规矩，给大家来一张图，直观的感受一下整个过程。
<div align="center"> <img src="/images/spring Cloud高可用3.png"/> </div><br>


## 四、线上经验—如何设置请求超时时间

线程数量OK了，那么请求的超时时间设置为多少？答案是300毫秒。
为啥呢？很简单啊，如果你的超时时间设置成了500毫秒，想想可能会有什么后果？
考虑极端情况，如果服务B响应变慢，要500毫秒才响应，你一个线程每秒最多只能处理2个请求了，10个线程只能处理20个请求。
而每秒是30个请求过来，结局会如何？
咱们回看一下第一张图就知道了，大量的线程会全部卡死，来不及处理那么多请求，最后用户会刷不出来页面。
还是有点不理解？再给你一张图，让你感受一下这个不合理的超时时间导致的问题！
<div align="center"> <img src="/images/spring Cloud高可用4.png"/> </div><br>

如果你的线程池大小和超时时间没有配合着设置好，很可能会导致服务B短暂的性能波动，瞬间导致服务A的线程池卡死，
里面的线程要卡顿一段时间才能继续执行下一个请求。
哪怕一段时间后，服务B的接口性能恢复到200毫秒以内了，服务A的线程池里卡死的状况也要好一会儿才能恢复过来。
你的超时时间设置的越不合理，比如设置的越长，设置到了1秒、2秒，那么这种卡死的情况就需要越长的时间来恢复。
所以说，此时你的超时时间得设置成300毫秒，保证一个请求300毫秒内执行不完，立马超时返回。
这样线程池里的线程不会长时间卡死，可以有条不紊的处理多出来的请求，大不了就是300毫秒内处理不完立即超时返回，
但是线程始终保持可以运行的状态。
这样当服务B的接口性能恢复到200毫秒以内后，服务A的线程池里的线程很快就可以恢复。
这就是生产系统上的hystrix参数设置优化经验，你需要考虑到各种参数应该如何设置。
否则的话，很可能会出现上文那样的情况，用了高大上的Spring Cloud架构，结果跟黑盒子一样，莫名其妙系统故障，
各种卡死，宕机什么的。

好了，我们继续。如果现在这套系统每秒有6000请求，然后核心服务A一共部署了60台机器，每台机器就是每秒会收到100个请求，
那么此时你的线程池需要多少个线程？
很简单，10个线程抗30个请求，30个线程抗100请求，差不多了吧。
这个时候，你应该知道服务A的线程池调用服务B的线程池分配多少线程了吧？超时时间如何设置应该也知道了！
其实这个东西不是固定死的，但是你要知道他的计算方法。
根据服务的响应时间、系统高峰QPS、有多少台机器，来计算出来，线程池的大小以及超时时间！


## 五、服务降级

设置完这些后，就应该要考虑服务降级的事了。
如果你的某个服务挂了，那么你的hystrix会走熔断器，然后就会降级，你需要考虑到各个服务的降级逻辑。

举一些常见的例子：
如果查询数据的服务挂了，你可以查本地的缓存
如果写入数据的服务挂了，你可以先把这个写入操作记录日志到比如mysql里，或者写入MQ里，后面再慢慢恢复
如果redis挂了，你可以查mysql
如果mysql挂了，你可以把操作日志记录到es里去，后面再慢慢恢复数据。
具体用什么降级策略，要根据业务来定，不是一成不变的。

## 六、总结

最后总结一下，排除那些基础设施的故障，你要玩儿微服务架构的话，需要保证两点：
首先你的hystrix资源隔离以及超时这块，必须设置合理的参数，避免高峰期，频繁的hystrix线程卡死
其次，针对个别的服务故障，要设置合理的降级策略，保证各个服务挂了，可以合理的降级，系统整体可用！