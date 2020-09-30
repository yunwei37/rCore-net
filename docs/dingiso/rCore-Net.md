# rCore-Net 分析

rCore-Net 主要借用 smoltcp 的crate ，来完成socket，tcp等功能



普遍函数调用过程 ，获取 SOCKETS 的互斥锁， 并获取他的句柄，转类型为 TcpSocket 或 UcpSocket

进行对 socket 的操作，然后及时将锁和句柄drop 掉

**SOCKETS** ：smoltcp是单线程的网络协议栈，定义一个全局的互斥锁，以保证操作的互斥性







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

