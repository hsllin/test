

```shell
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```
或

```shell
netstat -ant|awk '/^tcp/ {++state[$NF]} END {for(key in state) print (key,state[key])}'

```


它会显示例如下面的信息：

TIME_WAIT 814  
CLOSE_WAIT 1  
FIN_WAIT1 1  
ESTABLISHED 634  
SYN_RECV 2  
LAST_ACK 1

常用的三个状态是：ESTABLISHED 表示正在通信，TIME_WAIT 表示主动关闭，CLOSE_WAIT 表示被动关闭。

---------------------------------

参考
http://hanlinsir.com/network/2020/04/10/%E8%A7%A3%E5%86%B3TIME_WAIT-CLOSE_WAIT%E8%BF%87%E5%A4%9A%E7%9A%84%E9%97%AE%E9%A2%98.html

**服务器大量处于TIME\_WAIT状态，可能是什么原因，会造成什么影响，怎么解决？CLOSE\_WAIT状态呢？**

## 1\. TIME\_WAIT与CLOSE\_WAIT产生原因

TIME\_WAIT与CLOSE\_WAIT 发生于客户端与服务器断开连接时四次挥手协议。

![image-20201124093843977](https://funnylu-1259196254.cos.ap-beijing.myqcloud.com/java/image-20201124093843977.png)

表示 TCP 断开连接的时候,需要客户端和服务端总共发送 4 个包以确认连接的断开；客户端或服务器均可主动发起挥手动作(因为 TCP 是一个全双工协议)，在 socket 编程中，任何一方执行 close() 操作即可产生挥手操作。

上面的一次socket关闭操作，我们可以得出以下几点：

1.  主动关闭连接的一方，也就是主动调用socket的close操作的一方，最终会进入TIME\_WAIT状态
    
2.  被动关闭连接的一方，有一个中间状态，即CLOSE\_WAIT，因为协议层在等待上层的应用程序，主动调close操作后才主动关闭这条连接
    
3.  TIME\_WAIT会默认等待2MSL时间后，才最终进入CLOSED状态。
    
4.  在一个连接没有进入CLOSED状态之前，这个连接是不能被重用的！
    

所以，TIME\_WAIT并不可怕，CLOSE\_WAIT才可怕，因为CLOSE\_WAIT很多，表示说要么是你的应用程序写的有问题，没有合适的关闭socket；要么是说，你的服务器CPU处理不过来（CPU太忙）或者你的应用程序一直睡眠到其它地方(锁，或者文件I/O等等)，你的应用程序获得不到合适的调度时间，造成你的程序没法真正的执行close操作。

