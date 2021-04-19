# optimization

## 内核参数优化
```
cat >>/etc/sysctl.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1  #将桥接流量传递到iptables的链
net.bridge.bridge-nf-call-iptables = 1 #将桥接流量传递到iptables的链
net.ipv4.ip_forward = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
kernel.sysrq = 0
kernel.core_uses_pid = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
#kernel.shmmax = 4294967285   # 单位字节，可用最大共享内存段，4G 大小一般为物理内存-1byte
#kernel.shmmni = 4096     # 最大共享内存段数量，为默认值不必修改
#kernel.shmall = 1048575  # 单位页, 可用最大共享内存分配页, 不小于 kernel.shmmax / getconf PAGE_SIZE
net.core.somaxconn = 10240
net.core.netdev_max_backlog = 250000
#net.core.rmem_default = 8388608  被net.ipv4.tcp_rmem[1] 覆盖
#net.core.wmem_default = 8388608  被net.ipv4.tcp_wmem[1] 覆盖
#net.core.rmem_max = 16777216     被net.ipv4.tcp_rmem[2] 覆盖
#net.core.wmem_max = 16777216     被net.ipv4.tcp_wmem[2] 覆盖
net.ipv4.tcp_rmem = 16384 1048576 12582912
net.ipv4.tcp_wmem = 16384 1048576 12582912
net.ipv4.tcp_mem = 1541646 2055528 3083292
net.ipv4.tcp_max_syn_backlog = 10240
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_syncookies = 1
#net.ipv4.tcp_tw_recycle = 0     内核4.12后已弃用，用nat访问的客户端，容易引起服务端包混乱
net.ipv4.tcp_tw_resue = 1
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_moderate_rcvbuf = 1
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_retries1 = 2
net.ipv4.tcp_retries2 = 3
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_intvl = 2
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.ip_local_port_range = 1024 65000
net.ipv4.tcp_sack = 1
net.ipv4.tcp_window_scaling = 1
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
vm.swappiness = 0
fs.file-max = 204800
EOF
sysctl -p
```

## IO调度优化
- Anticipatory I/O scheduler 适用于大多数环境,特别是写入较多的环境(比如文件服务器)Web,App等应用,但不太合适数据库应用，
- Deadline I/O scheduler 通常与Anticipatory相当,但更简洁小巧,更适合于数据库应用
- CFQ I/O scheduler 为所有进程分配等量的带宽,适合于桌面多任务及多媒体应用，适用于有大量进程的多用户系统，默认IO调度器
- NOOP I/O scheduler NOOP对于闪存设备,RAM,嵌入式系统是最好的选择.其应用环境主要有以下两种：一是物理设备包含自己的I/O调度程序，比如SCSI的TCQ；二是寻道时间可以忽略不计的设备，比如SSD等。

```
#临时设置
cat /sys/block/sda/queue/scheduler
echo deadline > /sys/block/sda/queue/scheduler

#永久设置
/etc/default/grub
GRUB_CMDLINE_LINUX="elevator=deadline"

grub-mkconfig -o /boot/grub/grub.cfg
```
