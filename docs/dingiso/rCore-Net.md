# rCore-Net 分析

rCore-Net 主要借用 smoltcp 的crate ，主要完成了，TCP，UDP，RAW，PACKET（作为以太网出口），Netlink 等结构和相关功能。

## 总体结构：

**SOCKETS** ：smoltcp是单线程的网络协议栈，定义一个全局的互斥锁，以保证操作的互斥性

### 四种 SocketState 

TCP：利用TCP协议的方式，

UDP：利用UDp协议的方式

Raw：可用于接受其他协议的数据

Packet：没有状态，只用于以太网出口

Netlink：基于socket的通信机制实现的内核空间和用户空间的销量数据的及时交互

### SocketState 的 imp 函数们

read ： 从 buffer 中取出尽可能多的数据，放到一个 u8数组，（data） 中

write ： 

* Tcp 发送到连接的远程 endpoint  
* Udp 发送到参数指定的远程 endpoint  
* Raw 如果头部包含在考虑范围则可以直接发送，否则自己组一个 ip packet包，发送到指定远程 endpoint(通过将远程地址填入头部 destination处)

poll ： 检查socket状态 ， 返回（是否正常，是否能接收，是否能发送）

connect ： 与远程 endpoint 连接 ，通过设置remote_endpoint, Tcp需要分配一个临时的 port 。

bind ： 将 socket 绑定一个本地的 ip

listen ： 如果没有listen，就开启listen，否则不操作

shutdown : 关闭socket

accept ： Tcp 从连接事件队列中取出一个时间，建立新的socket 并替换当前的socket

local_endpoint  : 返回本地ip

remote_endpoint : 返回远程 ip

box_clone :  clone一个自身对象

**ioctl ： TODO**

## 函数的普遍过程

普遍函数调用过程 ，获取 SOCKETS 的互斥锁， 并获取他的句柄，转类型为 TcpSocket 或 UcpSocket

进行对 socket 的操作，然后及时将锁和句柄drop 掉



**TODO**： ioctl 和 ArpSeq 的部分 可能还需要再研究一下

# Smoltcp 分析

## TCP

Tcp State :   [enum] 由 rfc 793 规定的 11 种状态

<img src="https://github.com/yunwei37/rCore-net/blob/master/docs/dingiso/imgs/TCP%E7%8A%B6%E6%80%81%E8%BD%AC%E6%8D%A2.png" alt="TCP状态转换图" />

### Timer : 

* [enum]  由timer 触发的重传，快速重传等行为
* set_for_idle()   设定活跃时间为 当前时间戳 timestamp + 设定的时间间隔 interval
* should_keep_alive()   判断当前时间戳是否 >= 活跃时间 
* should_retransmit() 
  * 时间戳大于重传期限则重传，返回 超时时间+延迟  
  * 快速重传 返回 0
* set_keep_alive() 初始化 活跃时间为 0
* should_close() 时间戳大于关闭期限 返回 true
* poll_at 返回 socket 下一次被 poll 的时间



问题：

* TcpSocketState的 write函数， 第二个参数  `_send_to_endpoint` 并没有使用， 提了 issue [[Bug Report] wrong of fn write in the TcpSocketState](https://github.com/rcore-os/rCore/issues/69)  

好吧，我傻了，此参数只是用来填充以保证 trait 的实现的。（菜）

* smoltcp -  storage/ring_buffer.rs  255行 dequeue_slice 函数为什么计算两次size

