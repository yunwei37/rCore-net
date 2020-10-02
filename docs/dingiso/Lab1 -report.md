# Lab1 -report

在实验一中，我对 jiegec 学长的代码进行了分析，写了注释，并更改了原来的测试程序，使其能充分测试网络部分的相关功能，

### 已提交的文件

注释后的网络部分代码 ： [struct.rs](https://github.com/yunwei37/rCore-net/blob/master/docs/dingiso/structs.rs)

更改后的测试代码： [test.rs](https://github.com/yunwei37/rCore-net/blob/master/docs/dingiso/test.rs)

我对与 rCore -Net 和 smoltcp 的分析 ： [analysis.md](https://github.com/yunwei37/rCore-net/blob/master/docs/dingiso/rCore-Net.md)

### 现存的问题：

* 在 rCore 重新实现运行的方法上还存在问题，暂时还没办法重新运行net

* 还需要和学长沟通，复现当时能成功运行时的环境
* Smoltcp 更多的是针对于嵌入式开发的网络栈，且只能实现单线程，我们先设法运行，最终可能需要改动这个库