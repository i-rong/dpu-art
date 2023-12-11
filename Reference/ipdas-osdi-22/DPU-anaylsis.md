# 论文阅读记录
## 作者/机构
IPDAS OSDI
## 开源内容
### Docs
[USENIX](https://www.usenix.org/conference/osdi23/presentation/wei-smartnic)

[OSDI '23 Slides](https://www.usenix.org/system/files/osdi23_slides_wei_smartnic.pdf)
### Code
[Github](https://github.com/smartnickit-project/smartnic-bench/tree/main)
## Background and Motivation
DPU 网卡目前只是简单的做功能卸载，没有对其内部数据通路进行充分分析。

## Desigen

DPU网卡可以有很多种通路：RMDA clent->pcie>host client->pcie->pcie switch>host soc->pice switch->nic->switch DMA soc->host

将这些路径进行充分复用以及路选可以极高的提升DPU资源利用率

## Experiment
### 关键指标分析

### 实验结果

