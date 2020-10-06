# 在rCore和zCore上面尝试添加网络功能 - 方案设计文档

这是 `操作系统专题训练2020：在rCore和zCore上面尝试添加网络功能` 小组的方案设计文档：

成员：

- 卢睿博：dingiso
- 郑昱笙：yunwei37

## 实现网络功能的已有工作和参考

- rCore [https://github.com/rcore-os/rCore](https://github.com/rcore-os/rCore)
- rCore 使用的tcp/ip协议栈 smoltcp：[https://github.com/smoltcp-rs/smoltcp]([smoltcp](https://github.com/smoltcp-rs/smoltcp))，其中的部分例子以及其原理是很重要的参考资料；

## 实验方案设计思路

1. 阅读 rCore 网络部分相关的源代码，并寻找其他参考资料，了解实现网络功能的原理；
   
    - 目前这部分我们主要参考的是 rCore 的源代码，以及 smoltcp 的相关资料；通过这些可以基本了解到网络协议栈的实现原理；
    - 当前已经对 rcore 网络功能的部分代码添加了一下注释，并增加了一下测试用例；对 smoltcp 和 rcore 网络相关部分撰写了一点简单的分析文档。

2. 根据旧有已经能运行的源代码，尝试修复rcore上面的网络功能，并使用async进行重新适配；
    - 目前 rCore 的源代码在网络部分还基本没有进行修改（在不可读的时候还是使用锁进行调度），但由于读写相关的模块已经使用 async 进行重新适配过，二者不兼容，因此暂时不可用；
    - 我们可以先尝试使用 async 对 kernel/src/net/struct.rs 中的 trait Socket 以及相关实现进行改造，以适配新的 async read/write/poll 等操作；这部分应该可以参考之前在 zcore 上移植和改造相关系统调用的实现；

3. 如果还有多余的时间的话，有两个方向可以进行尝试：
    
    - 尝试在 zcore 上面实现网络功能；
    - 更改一下 rcore 的网络协议栈：目前的 smoltcp 协议栈是一个适用于嵌入式设备的单线程协议栈，并不是很适合于 rcore 这样的操作系统；学长提议所可以重新实现一个。

## 分工

暂时还不明确；