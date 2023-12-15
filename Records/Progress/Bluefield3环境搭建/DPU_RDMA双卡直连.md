# 双卡直连
所有 InfiniBand 网络都必须运行子网管理器才能正常工作。即使两台机器没有使用交换机直接进行连接，也是如此。

有可能有一个以上的子网管理器。在这种情况下，一个 master 充当一个主子网管理器，另一个子网管理器充当从属子网管理器，当主子网管理器出现故障时将接管。

大多数 InfiniBand 交换机都包含一个嵌入式子网管理器。然而，如果您需要一个更新的子网管理器，或者您需要更多控制，
https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/8/html-single/configuring_infiniband_and_rdma_networks/index

## rdma bandwitdh test
Node A :ib_send_bw -R -d rocep130s0 -a -F\
Node B : ib_send_bw -R -d mlx5_1 -i 1 192.168.0.204 --report_gbits -n 1000 -a -F\
[参考文档](https://blog.csdn.net/bandaoyu/article/details/115798045)