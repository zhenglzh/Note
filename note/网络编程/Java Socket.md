### 常见问题

1. java.net.SocketException: Software caused connection abort: socket write error

   客户端关闭，服务端提示

2. java.net.SocketTimeoutException

   连接超时一个是在connect ，一个是读写时候设置了soTimeOut

3. java.net.ConnectException: Connection refused: connect

   出现原因:该异常发生在客户端进行`new Socket(Ip,Port)`或`socket.connect(address,timeout)`操作时，原因**就是指定的ip地址不能被找到，或者说`ip`地址存在，但是找不到对应的端口进行监听，或者主机连接队列已满**。

   解决方案:首先检查客户端的`ip`和`port`是否写错了，假如正确可以测试客户端和服务器端时候可以`ping`通，如果可以`ping`通，则在服务端重新找一个可以用的端口；如果`ping`不通，则需要另外想办法了。

4. java.net.SocketException: Socket is closed

   出现原因:该异常在客户端和服务器端均可能发生，原因就是，**客户端或者服务器端主动关闭了链接**，Socket的close方法，随后再次对网络链接进行一系列操作。
   解决方案:首先我们要弄清楚主动关闭链接的原因，杜绝以后再次被关闭的可能性；然后我们重启客户端和Server端，重新建立通讯即可。

5. java.net.SocketException:Connection reset 或者 Connect reset by peer:Socket write error出现原因

   - 如果一端的`Socket`被关闭(主动或者异常引起的关闭)后，另一方还在继续放松数据，**发送端**发送的数据包会引发异常`Connect reset by peer`；
   - 另一个是端退出，但退出时为关闭链接，另一端从**连接中读取数据**则抛出异常`Connection reset`.总结一下便是，因为由链接断开后的读和写操作引起的





### Java GC时清理socket资源

Java的socket最终关联到AbstractPlainSocketImpl,且其重写了object的finalize方法

```JAVA
abstract class AbstractPlainSocketImpl extends SocketImpl 
{
	......
    /**
     * Cleans up if the user forgets to close it.
     */
    protected void finalize() throws IOException {
        close()
    }
    ......
}
```

所以Java会在GC时刻会关闭没有被引用的socket,但是切记不要寄希望于Java的GC,因为GC时刻并不是以未引用的socket数量来判断的，所以有可能泄露了一堆socket,但仍旧没有触发GC。