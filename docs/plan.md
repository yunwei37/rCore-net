# 在rCore和zCore上面尝试添加网络功能 - 方案设计文档

这是 `操作系统专题训练2020：在rCore和zCore上面尝试添加网络功能` 小组的方案设计文档：

成员：

- 卢睿博：dingiso
- 郑昱笙：yunwei37

## 实现网络功能的已有工作和参考

- rCore [https://github.com/rcore-os/rCore](https://github.com/rcore-os/rCore)
- rCore 使用的tcp/ip协议栈 smoltcp：[https://github.com/smoltcp-rs/smoltcp]([smoltcp](https://github.com/smoltcp-rs/smoltcp))，其中的部分例子以及其原理是很重要的参考资料；

## 实验方案设计思路

1. 阅读 rCore 网络部分相关的源代码，并寻找其他参考资料，了解实现网络功能的原理； (基本完成)

   - 目前这部分我们主要参考的是 rCore 的源代码，以及 smoltcp 的相关资料；通过这些可以基本了解到网络协议栈的实现原理；
   - 当前已经对 rcore 网络功能的部分代码添加了一下注释，并增加了一下测试用例；对 smoltcp 和 rcore 网络相关部分撰写了分析文档。

2. 将 rCore-Net 运行起来，以备后面运行时的参考；

   ###                                                            															async

   * rCore 的 读写相关的模块async 进行重新适配过，二者不兼容，因此暂时不可用  
     * 需要回滚到先前未更改过async的版本
   * 使用锁进行调度，面临阻塞问题的处理
   * 只有用户态主动访问网络，网络功能才进行使用

3. 根据旧有已经能运行的源代码，尝试修复rcore上面的网络功能，并使用async进行重新适配； 

   - 我们可以先尝试使用 `async` 对 `kernel/src/net/struct.rs` 中的 `trait Socket` 以及相关实现进行改造，以适配新的 `async read/write/poll` 等操作；这部分应该可以参考之前在 zcore 上移植和改造相关系统调用的实现；
   - 面临网络功能阻塞的时候放一个`waker`，网卡有更新的时候放一个唤醒一下，利用`waker`判断是否可以读
   - 增加一个线程去poll网络栈`smoltcp` ， 以响应 `ping`

4. 接下来的两个方向可以进行尝试：

   - 尝试在 `zcore` 上面实现网络功能；
   - 更改一下 `rcore` 的网络协议栈：
     - `smoltcp` 协议栈是一个适用于嵌入式设备的单线程协议栈
     - 相对独立于操作系统的存在，没有与操作系统很好的耦合

## 分工

对于修复网络功能的部分，分工还不是很明确；

对于接下来的方向：

​	郑昱笙 ： zCore 网络功能实现

​	卢睿博 ： 更改 网络协议栈



### 更改的阶段性目标

以 单核 为目标 

需要增加更多的调研工作 - 包括其他更精简的协议栈 和 rust 协议栈