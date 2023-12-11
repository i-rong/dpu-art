# 手动环境搭建trouble-shooting
按照NVIDIA DOCA Installation Guide for Linux 在ubuntu20.04上安装host端驱动可能会遇到的问题
## issue 1
### 安装doca-cx-Connect相关包的适合会遇到

dpkg-deb: error: paste subprocess was killed by signal (Broken pipe)
且会卡在 sudo apt-get -f install

### solution：

sudo dpkg -i --force-overwrite /opt/mello....

## issue 2
### meson build smaples 会遇到少包的问题 dependency doca found: NO (tried cmake) 包括缺少dpdk
### solution：
[ref](https://docs.nvidia.com/doca/sdk/troubleshooting/index.html#doca-sdk-compilation)

export PKG_CONFIG_PATH=${PKG_CONFIG_PATH}:/opt/mellanox/doca/lib/x86_64-linux-gnu/pkgconfig

export PKG_CONFIG_PATH=${PKG_CONFIG_PATH}:/opt/mellanox/dpdk/lib/x86_64-linux-gnu/pkgconfig

export PKG_CONFIG_PATH=${PKG_CONFIG_PATH}:/opt/mellanox/doca/lib/aarch64-linux-gnu/pkgconfig
export PKG_CONFIG_PATH=${PKG_CONFIG_PATH}:/opt/mellanox/dpdk/lib/aarch64-linux-gnu/pkgconfig
export PKG_CONFIG_PATH=${PKG_CONFIG_PATH}:/opt/mellanox/flexio/lib/pkgconfig/
## issue 3
### 编译sample会遇到少库的情况，这是因为meson.build 文件写的有问题，找一下缺少的函数在哪个库里加上就行
### solution：
/opt/mellanox/doca/samples/doca_dpa/dpa_kernel_launch/build/../../dpa_common.c:290: undefined reference to `doca_dpa_destroy'

sample_dependencies += declare_dependency(link_args : ['-L/opt/mellanox/doca/lib/x86_64-linux-gnu/', '-ldoca_dpa'])

sample_dependencies += declare_dependency(link_args : ['-L/opt/mellanox/doca/lib/aarch64-linux-gnu/', '-ldoca_dma'])


## issue 4
主机上创建DPA线程时会有flexio创建线程的情况，这是主机上没有EU可用，偶然的解决方案：在DPU上用dpaeumgmt申请，然后主机端reset。\
flexio_create_prm_thread 395 - Failed to create PRM thread object. Error number is 121.

主机运行DPA程序之前，要在DPU上创建的EU的partition资源。-v 0，1指定了 vHCA即mlx的编号。-r制定了DPA的可用范围,-m指定了逐渐上可见的EU group
 ./dpaeumgmt  partition create -d mlx5_0 -v 0,1 -r 1-100 -m 2 