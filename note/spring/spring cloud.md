# Hytrix源码解析

## 特性整理

- 断路器机制
  - 计算线路健康度
- Fallback ( 失败回退 )
- 资源隔离
  - 方式
    - 线程池
    - 信号量
  - 依赖隔离
- 执行模型
  - 同步执行
  - 异步执行
  - Reactive模式执行
    - observe
    - toObservable
- 运维平台
  - 基础 Dashboard
  - 整合 Turbine
- 缓存
- 请求合并( HystrixCollapser )

## RXJAVA的了解

### 角色

在RxJava中主要有4个角色：

- Observable
- Subject
- Observer
- Subscriber

Observable和Subject是两个“生产”实体，Observer和Subscriber是两个“消费”实体。说直白点Observable对应于观察者模式中的**被观察者**，而Observer和Subscriber对应于观察者模式中的**观察者**。Subscriber其实是一个实现了Observer的抽象类，后面我们分析源码的时候也会介绍到。Subject比较复杂，以后再分析。

上一篇文章中我们说到RxJava中有个关键概念：**事件**。观察者Observer和被观察者Observable通过subscribe()方法实现订阅关系。从而Observable 可以在需要的时候发出**事件**来通知Observer。

## **Hystrix 执行命令方法**