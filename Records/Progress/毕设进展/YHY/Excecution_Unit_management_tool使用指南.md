# EU 使用指南

## 前言

根据 [官方 EU 文档](https://docs.nvidia.com/doca/sdk/nvidia+doca+dpa+execution+unit+management+tool/index.html) 记录一些 EU 命令的使用方法及功能

以下全都在 arm 上操作

```bash
ssh ubuntu@192.168.100.2

pwd: 123456
```

dpaeumgmt 工具在目录 `/opt/mellanox/doca/tools`，有些功能需要在这个目录下才能使用。

```bash
cd /opt/mellanox/doca/tools/
```

dpaeumgmt  使用户能够创建、销毁和查询 EU 对象。

可以使用 `dpaeumgmt help` 查看该工具的一些使用方法

## 常规命令

* `dpaeumgmt -h` 打印工具的基本使用信息
* `dpaeumgmt help` 打印工具命令的详细帮助菜单
* `dpaeumgmt version` 打印版本信息

## EU 组命令

以下小节中列出的命令用于配置 EU 组。

### Info EU 组

打印与 EU 组相关的 DPA 资源信息

```bash
ubuntu@localhost:/opt/mellanox/doca/tools$ sudo ./dpaeumgmt eu_group info -d mlx5_0
Max number of DPA EU groups: 15
Max number of DPA EUs in one DPA EU group: 190
Max DPA EU number available to use: 190
Max EU group name length is 15 chars
```

### 创建 EU 组

```bash
ubuntu@localhost:/opt/mellanox/doca/tools$ sudo ./dpaeumgmt eu_group create -d mlx5_0 -n "HG hello world1" -r "6-8,16,55,70"
Group created successfully-
EU group ID: 1
EU group name: HG hello world1
Member EUs are: 6,7,8,16,55,70
```

### 销毁 EU 组

```bash
ubuntu@localhost:/opt/mellanox/doca/tools$ sudo ./dpaeumgmt eu_group destroy -d mlx5_0 -g 1
Group with group id: 1, was destroyed successfully
```

### 查询 EU 组

```bash
ubuntu@localhost:/opt/mellanox/doca/tools$ sudo ./dpaeumgmt eu_group query -d mlx5_0
In total there are 0 EU groups configured.
ubuntu@localhost:/opt/mellanox/doca/tools$ sudo ./dpaeumgmt eu_group create -d mlx5_0 -n "HG hello world1" -r "6-8,16,55,70"
Group created successfully-
EU group ID: 1
EU group name: HG hello world1
Member EUs are: 6,7,8,16,55,70
ubuntu@localhost:/opt/mellanox/doca/tools$ sudo ./dpaeumgmt eu_group query -d mlx5_0
1) EU group ID: 1
EU group name: HG hello world1
Member EUs are: 6,7,8,16,55,70

In total there are 1 EU groups configured.
```

更多选项：

```bash
ubuntu@localhost:/opt/mellanox/doca/tools$ sudo ./dpaeumgmt eu_group query -d mlx5_0 -n "HG hello world1"
EU group ID: 1
EU group name: HG hello world1
Member EUs are: 6,7,8,16,55,70
ubuntu@localhost:/opt/mellanox/doca/tools$ sudo ./dpaeumgmt eu_group query -d mlx5_0 -g 1
EU group ID: 1
EU group name: HG hello world1
Member EUs are: 6,7,8,16,55,70
```

### 申请 EU 组

```bash
dpaeumgmt eu_group apply --dpa_device <device> --file_groups <file>
```

文件格式示例：

```

{
        "eu_groups": [
                { "name": "hg1", "range": "178-180"},
                { "name": "hg2", "range": "2-10"}
        ]
}
```

> 警告：
>
> 该命令将删除 DPA 设备所属的 EU 分区上定义的所有先前的 EU 组，并应用文件中的 EU 组。

```bash
$ sudo ./dpaeumgmt eu_group apply -d mlx5_0 --file_groups example.json
1) EU group ID: 1
EU group name: hg1
Member EUs are: 178,179,180
 
1) EU group ID: 2
EU group name: hg2
Member EUs are: 2,3,4,5,6,7,8,9,10
 
In total there are 2 EU groups configured.
```

## EU 分区命令

以下小节中列出的命令用于配置 EU 分区。

### Info EU 分区

```bash
ubuntu@localhost:/opt/mellanox/doca/tools$ sudo ./dpaeumgmt partition info -d mlx5_0
Max number of DPA EU partitions: 15
Max number of VHCAs associated with a single partition: 32
Max number of DPA EU groups: 15
Note- an allocation of a partition consumes from the number of DPA EU *groups* available to create
Max number of DPA EUs available to use: 190
```

### 创建 EU 分区

前面创建一个 Group 没销毁 这里要先销毁

```bash
ubuntu@localhost:/opt/mellanox/doca/tools$ sudo ./dpaeumgmt partition create -d mlx5_0 -v 1 -r 10-20 -m 1
Partition created successfully-
EU Partition ID: 1
Maximal number of groups: 1
The partition has a total of 1 associated VHCA IDs, namely: 1
Partition's member EUs are: 10,11,12,13,14,15,16,17,18,19,20
```

### 破坏 EU 分区

```bash
ubuntu@localhost:/opt/mellanox/doca/tools$ sudo ./dpaeumgmt partition destroy -d mlx5_0 -p 1
Partition with partition id: 1, was destroyed successfully
```

### 查询 EU 分区

```bash
ubuntu@localhost:/opt/mellanox/doca/tools$ sudo ./dpaeumgmt partition create -d mlx5_0 -v 1 -r 10-20 -m 1
Partition created successfully-
EU Partition ID: 1
Maximal number of groups: 1
The partition has a total of 1 associated VHCA IDs, namely: 1
Partition's member EUs are: 10,11,12,13,14,15,16,17,18,19,20
ubuntu@localhost:/opt/mellanox/doca/tools$ sudo ./dpaeumgmt partition query -d mlx5_0 -p 1
EU Partition ID: 1
Maximal number of groups: 1
The partition has a total of 1 associated VHCA IDs, namely: 1
Partition's member EUs are: 10,11,12,13,14,15,16,17,18,19,20
```

## 实战

运行 `~/DPU_test/doca/samples/doca_dpa/dpa_wait_kernel_launch/build/doca_dpa_wait_kernel_launch` 之前，需要创建 EU 分区。

需要注意，网卡核心在不同的设备上抽象出来的设备 id 是不一样的，比如，在 arm 上，抽象出来的是 mlx5_0，在 host 上，抽象出来的是 mlx5_1

![image-20231216144738097](https://cdn.jsdelivr.net/gh/i-rong/BlogImages@main/img/202312161447173.png)

![image-20231216144820646](https://cdn.jsdelivr.net/gh/i-rong/BlogImages@main/img/202312161448729.png)

但是这样运行，发现

```bash
qs@powerleader:~/DPU_test/doca/samples/doca_dpa/dpa_wait_kernel_launch/build$ sudo ./doca_dpa_wait_kernel_launch -d mlx5_1
[06:49:08:951617][DOCA][INF][WAIT_KERNEL_LAUNCH::MAIN:52]: Starting the sample
flexio_create_prm_thread 395 - Failed to create PRM thread object. Error number is 121.
create_thread 168 - Failed to create thread
process_call 385 - Failed to create thread
internal_msg_stream_create 695 - Failed to call msg stream config dev handler

[06:49:09:127208][DOCA][ERR][DPA_COMMON:271]: Failed to create DOCA DPA context: DOCA Driver call failure
[06:49:09:127563][DOCA][ERR][WAIT_KERNEL_LAUNCH::MAIN:76]: Failed to Allocate DPA Resources: DOCA Driver call failure
[06:49:09:127572][DOCA][INF][WAIT_KERNEL_LAUNCH::MAIN:102]: Sample finished with errors
```

说明 mlx5_1 不对

试试 mlx5_2

```bash
qs@powerleader:~/DPU_test/doca/samples/doca_dpa/dpa_wait_kernel_launch/build$ sudo ./doca_dpa_wait_kernel_launch -d mlx5_2
[06:49:10:671245][DOCA][INF][WAIT_KERNEL_LAUNCH::MAIN:52]: Starting the sample
[06:49:10:863783][DOCA][INF][WAIT_KERNEL_LAUNCH::SAMPLE:87]: All DPA resources have been created
/  7/Hello from kernel
[06:49:18:034557][DOCA][INF][WAIT_KERNEL_LAUNCH::MAIN:100]: Sample finished successfully
```

居然成功了，说明它不是按照这个对应的，在创建的时候是一个一个枚举的。

这里先不管怎么对应上了，反正也没几个设备，一个一个试就完事了，之后遇到问题再解决吧。





