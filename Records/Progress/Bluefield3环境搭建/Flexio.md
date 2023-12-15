# DOCA FlexIO SDK Programming Guide
数据路径加速器(DPA)处理器是一种辅助处理器，用于加速数据包处理和其他数据路径操作。FlexIO SDK公开了一个用于管理设备和在其上执行本机代码的API。\
本文想搞清楚的问题：1、Flexio API提供的能力。2、编程方式，和DOCA_DPA提供的接口怎么一起使用。

## Flexio是什么
Flexio写的程序在主机与DPU上都能运行\
Flexio库分成如下几个层次：\
1.libflexio,主要用于DPU侧的操作，用于资源管理\
2.libflexio_dev, 用于DPA侧数据路径的实现与操作\
3.ibflexio_os,用于ARM/host侧DPA的访问
4.libflexio_libc,DPA侧通用的libc库

## Flexio怎么用
DPA上运行的程序并不能创建资源，因此由主机/ARM创建FlexIO进程、window等等资源并传递至DPU侧。
```
flexio_buf_dev_alloc //. Allocate a buffer on the DPA side
```
## RDMA报文解析可能需要用到的接口
Buffers/Rings For DPA Queues\
FlexIO在DPU或者DPA上分配Queues，并且DPA可以可以将其地址开放给DPA，供其访问