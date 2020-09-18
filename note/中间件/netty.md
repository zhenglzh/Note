## 整体流程：

 

### 启动流程分析 ：

####







### 源码分析

#### NioEventLoopGroup
##### 继承体系：

![image-20200917123447738](asserts/image-20200917123447738.png!![image-20200917125630315](asserts/image-20200917125630315.png![image-20200917130039561](asserts/image-20200917130039561.png)(asserts/image-20200917123646503.png)

##### New:

1. 输入线程数否则默认是cpu核心数，创建EventExecutor数组保存NioEventLoop实例数据，默认创建了ThreadPerTaskExecutor继承Executor
2. 

##### 主要方法：

#### NioEventLoop

属性：

```
addTaskWakesUp
```

##### 从NioEventLoopGroup上创建：

 	1. parent 属性指向NioEventLoopGroup创建的实例
 	2. 在SingleThreadEventExecutor创建了addTaskWakesUp、maxPendingTasks，RejectedExecutionHandler、taskQueue(LinkedBlockingQueue)
 	3. 在SingleThreadEventExecutor上创建了Executor，从NioEventLoopGroup带下来的ThreadPerTaskExecutor进行封装，在每个任务执行的时候设置当前NioEventLoop数据。
 	4. 在SingleThreadEventLoop创建了tailTasks(LinkedBlockingQueue) 什么用呢
 	5. 保存provider来源NioEventLoopGroup， selector、unwrappedSelector、 selectStrategy作用io.netty.channel.nio.NioEventLoop#NioEventLoop？
 	6. 



ThreadPerTaskExecutor

![image-20200917132347404](asserts/image-20200917132347404.png)

疑问怎么没有用线程池去管线程

ThreadFactory是来自于DefaultThreadFactory，

##### 主要方法：

#### ServerBootstrap
##### channel
把NioServerSocketChannel.class赋值进去
##### option:
##### handler
##### childOption
##### childHandler
##### attr
##### childAttr
##### bind
1. initAndRegister 初始化；初始化NioServerSocketChannel，把option和attr设置到初始化NioServerSocketChannel.channelConfig中
##### initAndRegister
1. 新建NioServerSocketChannel

#### NioServerSocketChannel
##### 初始化：
1. SelectorProvider打开ServerSocketChannel，新增NioServerSocketChannelConfig

#### NioServerSocketChannelConfig
关联了ServerSocketChannel和socket






Tuple 元组