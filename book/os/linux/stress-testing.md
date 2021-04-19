# stress-testing

## stress
```bash
#内存压测
stress --vm 1 --vm-bytes 190M --vm-hang 0
stress --vm 2 --vm-bytes 300M --vm-keep #产生两个子进程 每个分配300M内存
#CPU压测
stress -c 2 # 消耗2个cpu资源
```

## fio
```bash
#Linux系统查看默认队列深度
lsscsi -l
#测试场景：
#100%随机，100%读， 4K
fio -filename=/dev/emcpowerb -direct=1 -iodepth 1 -thread -rw=randread -ioengine=psync -bs=4k -size=1000G -numjobs=50 -runtime=180 -group_reporting -name=rand_100read_4k

#100%随机，100%写， 4K
fio -filename=/dev/emcpowerb -direct=1 -iodepth 1 -thread -rw=randwrite -ioengine=psync -bs=4k -size=1000G -numjobs=50 -runtime=180 -group_reporting -name=rand_100write_4k

#100%顺序，100%读 ，4K
fio -filename=/dev/emcpowerb -direct=1 -iodepth 1 -thread -rw=read -ioengine=psync -bs=4k -size=1000G -numjobs=50 -runtime=180 -group_reporting -name=sqe_100read_4k

#100%顺序，100%写 ，4K
fio -filename=/dev/emcpowerb -direct=1 -iodepth 1 -thread -rw=write -ioengine=psync -bs=4k -size=1000G -numjobs=50 -runtime=180 -group_reporting -name=sqe_100write_4k

#100%随机，70%读，30%写 4K
fio -filename=/dev/emcpowerb -direct=1 -iodepth 1 -thread -rw=randrw -rwmixread=70 -ioengine=psync -bs=4k -size=1000G -numjobs=50 -runtime=180 -group_reporting -name=randrw_70read_4k
```

## iperf
```bash
#10.55.3.52 启动监听程序
iperf -s
#10.55.3.51 运行测试命令 间隔1秒反馈 连续检测5秒
iperf -c 10.55.3.52 -i 1 -t 5
111
```