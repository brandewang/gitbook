# command

## cpu
- 同一个进程同一时间段只能在一个cpu中运行，如果进程数小于cpu数，那么未使用的cpu将会空闲。
- 同一个线程同一时间段只能在一个cpu内核中运行，如果线程数小于cpu内核数，那么将有多余的内核空闲。

## shell
```bash
#登录shell
1:/etc/profile
2:/etc/profile.d/*.sh
3:$HOME/.bash_profile 会加载$HOME/.bashrc和/etc/bashrc
4:$HOME/.bash_login
5:$HOME/.profile

#交互式非登录shell
1.$HOME/.bashrc
2./etc/bashrc  会加载/etc/profile.d/*.sh
```

## bash
```bash
#打印调试信息
set -x
#遇错退出
set -e
```

## sed
```bash
#\1代表第一个括号匹配
echo 'abcabcabc' | sed 's/\(ab\)c/\1/'
```
## xargs
```bash
#同时使用一行的多个参数
echo arg1 arg2|xargs -l bash -c 'echo $0 \|$1'
```

## nfs

```bash
开机自动挂载
vi /etc/fstab
10.55.5.12:/volume1/nfs-bak01 /opt/bak  nfs     rw,soft,intr,timeo=30,retry=3,_netdev   0 0
```

## 持久化systemd-journal日志
```bash
mkdir /var/log/journal
chown root:systemd-journal /var/log/journal
chmod 2755 /var/log/jounal
systemctl restart systemd-journald
```

## 生成随机密码
```bash
openssl rand -base64 10
openssl rand -base64 10|tr A-Z a-z
openssl rand -base64 32|tr A-Z a-z|cut -c 1-10
```

## 时间同步
```bash
chronyc -a makestep
ntpdate 10.28.50.10
```

## 环境变量
```bash
#mysql env
export PATH=/usr/local/mysql/bin:$PATH

#jdk env
export JAVA_HOME=/usr/java/jdk1.8.0_151
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

## kill信号
```bash
#强制关闭
kill -9 $PID
#重载配置
kill -HUP $PID
#重读配置文件
kill -USR1 $PID
#重载程序
kill -USR2 $PID
```
## 日志切割

尽量使用logrotate,避免重复造轮

- compress                                #通过gzip压缩转储以后的日志
- nocompress                              #不压缩
- copytruncate                    #用于还在打开中的日志文件，把当前日志备份并截断
- nocopytruncate                  #备份日志文件但是不截断
- create mode owner group #转储文件，使用指定的文件模式创建新的日志文件
- nocreate                #不建立新的日志文件
- delaycompress           #和 compress 一起使用时，转储的日志文件到下一次转储时才压缩
- nodelaycompress         #覆盖 delaycompress 选项，转储同时压缩。
- errors address          #专储时的错误信息发送到指定的Email 地址
- ifempty                 #即使是空文件也转储，这个是 logrotate 的缺省选项。
- notifempty              #如果是空文件的话，不转储
- mail address            #把转储的日志文件发送到指定的E-mail 地址
- nomail                  #转储时不发送日志文件
- olddir directory        #转储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统
- noolddir                #转储后的日志文件和当前日志文件放在同一个目录下
- prerotate/endscript     #在转储以前需要执行的命令可以放入这个对，这两个关键字必须单独成行
- daily                   #指定转储周期为每天
- weekly                  #指定转储周期为每周
- monthly                 #指定转储周期为每月
- rotate count            #指定日志文件删除之前转储的次数，0 指没有备份，5 指保留5 个备份
- tabooext [+] list               #让logrotate不转储指定扩展名的文件，缺省的扩展名是：.rpm-orig, .rpmsave, v, 和 ~
- size size               #当日志文件到达指定的大小时才转储，bytes(缺省)及KB(sizek)或MB(sizem)
- missingok                               #在日志轮循期间，任何错误将被忽略，例如“文件无法找到”之类的错误。

```bash
#vim /etc/logrotate.d/nginx
/opt/logs/nginx/*.log {
        daily
        rotate 5
        missingok
        notifempty
        sharedscripts
        postrotate
            if [ -f /opt/run/nginx.pid ]; then
                kill -USR1 `cat /opt/run/nginx.pid`
            fi
        endscript
}

#vim /etc/logrotated.d/services
/opt/logs/*/*.log {
    daily
    rotate 5
    missingok
    notifempty
    sharedscripts
    postrotate
                cat /opt/run/*.pid|xargs -I {} kill -USR1 {}
    endscript
}
```

## 常用命令
```bash
#历史操作格式化
cat >> /etc/profile <<EOF
export HISTTIMEFORMAT="[%F %T] [`whoami`] "
EOF

#丢包率
lostpk=`timeout 5  ping -q -s 500 202.96.209.133 -W 1000 -c 100 -A |grep transmitted|awk '{print $6}'`
#总耗时
rrt=`timeout 5  ping -q -s 500 202.96.209.133 -W 1000 -c 100 -A |grep transmitted|awk '{print $10}'`

#bash调试模式并加入时间戳，调试默认通道为stderr,加入IFS可以保留stdin的头尾空格
bash -x  test.sh 2>&1 |while IFS= read -r line; do echo "$(date +'%H:%M:%S') $line"; done

#获取TCP连接情况
netstat -n |awk '/^tcp/ {++S[$NF]} END {for(a in S) print a,S[a]}'
```

## ssh

```bash
#解决dsa类型的key认证
##加入校验类型
/etc/ssh/sshd_config
PubkeyAcceptedKeyTypes=+ssh-dss
##允许所有加密规则
/etc/sysconfig/sshd
CRYPTO_POLICY=
```

## 内核升级

```bash
1.载入公钥
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org

2.安装ELRepo
yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm

3.添加repository 后, 列出可以使用的kernel包版本
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available

4.安装需要的kernel版本，这里安装 kernel-lt
yum --enablerepo=elrepo-kernel install kernel-lt

内核版本介绍：
lt:longterm的缩写：长期维护版；
ml:mainline的缩写：最新稳定版；

5.查看内核的启动顺序
awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg

6.设置启动顺序
默认启动的顺序是从0开始，新内核是从头插入（目前位置在0，而4.4.4的是在1），所以需要选择0。
grub2-set-default 0
#查看内核启动项
grub2-editenv list

7.卸载老版本kernel内核工具
rpm -qa|grep kernel|grep 3.10
rpm -qa|grep kernel|grep 3.10|xargs yum remove -y
备注：有一个正在运行的kernel3.10卸载不了，因为正在运行中，重启之后可卸载。

8.安装新版的工具包
yum --enablerepo=elrepo-kernel install -y kernel-lt-tools
检查：
rpm -qa|grep kernel

9.重启并检查版本
reboot
uname -a
```

## iptables
```bash
#清空规则
iptables -F
iptables -X
iptables -Z
iptables -L -n

iptables -F -t nat
iptables -X -t nat
iptables -Z -t nat
iptables -L -n -t nat

```

## gre tunnel
|vpc|外网地址|内网地址|隧道互联地址|
|--|--|--|--|
|杭州|121.196.208.175|10.55.5.100|192.168.1.1|
|北京|60.205.187.39|10.55.6.100|192.168.1.1|
``` bash
#杭州
modprobe  ip_gre
ip tunnel add tun1-bj mode gre remote 60.205.187.39 local 121.196.208.175
ip link set tun1-bj up
ipaddr add 192.168.1.1 peer 192.168.1.2 dev tun1-bj
route add -net 10.55.6.0/24 tun1-bj

#北京
modprobe  ip_gre
ip tunnel add tun1-hz mode gre remote 121.196.208.175 local 60.205.187.39
ip link set tun1-hz up
ipaddr add 192.168.1.2 peer 192.168.1.1 dev tun1-hz
route add -net 10.55.5.0/24 tun1-hz

```
