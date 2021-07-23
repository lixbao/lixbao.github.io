# Linux入侵排查思路

![image-20210702134811730](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210712233520.png)

## 1.检测系统账号安全

### 1.1 用户文件信息/etc/passwd

```shell 
root:x:0:0:root:/root:/bin/bash
account:password:UID:GID:GECOS:directory:shell
#用户名：密码：用户ID：组ID：用户说明：家目录：登陆之后shell
```

> 注意：无密码只允许本机登陆，远程不允许登陆

### 1.2 影子文件

```shell
t:$6$oGs1PqhL2p3ZetrE$X7o7bzoouHQVSEmSgsYN5UD4.kMHx6qgbTqwNVC5oOAouXvcjQSt.Ft7ql1WpkopY0UV9ajBwUt1DpYxTCVvI/:16809:0:99999:7:::
#用户名：加密密码：密码最后一次修改日期：两次密码的修改时间间隔：密码有效期：密码修改到期到的警告天数：密码过期之后的宽限天数：账号失效时间：保留
```

> who	查看当前登录用户（tty本地登陆 pts远程登录）
>
> w		查看系统信息，想知道某一时刻用户的行为
>
> uptime	查看登陆多久、多少用户，负载

### 1.3 入侵排查

+ **查询特权用户（uid为0）**

```shell 
awk -F: '$3==0{print $1}' /etc/passwd
```

+ **查询可以远程登录的帐号信息**

```shell
awk '/\$1|\$6/{print $1}' /etc/shadow
```

+ **除root帐号外，其他帐号是否存在sudo权限。**如非管理需要，普通帐号应删除sudo权限

```shell
more /etc/sudoers | grep -v "^#\|^$" | grep "ALL=(ALL) "
```

+ **禁用或删除多余及可疑的帐号**

```shell
usermod -L user	#禁用帐号，帐号无法录，/etc/shadow第二栏为!开头
userdel user	#删除user用户
userdel -r user	#将删除user用户，并且将/home目录下的user目录一并删除
```

## 2.历史命令查看

### 2.1 root 的历史命令

``` shell
history
```

### 2.2 打开/home各帐号目录下的.bash_history，查看普通帐号的历史命令

+ 如果用户的默认shell 是bash，那么bash会记录用户的默认历史记录。

+ ~/.bash_history记录的是上一次登陆系统所执行过的命令，而当前登陆这一次则保存在内存缓存中，当系统关机/重启后会更新到~/.bash_history文件中。

### 2.3 入侵排查

+ 进入用户目录下

+ cat .bash_history >> history.txt

### 2.4 历史操作命令清除

```shell
history –c 
#该命令并不会清楚保存在文件中的记录，需要手动删除~/.bash_history文件中的记录。
```

### 2.5 历史命令优化查看

+ 增加历史命令查看条数、增加显示登陆的IP和执行命令的时间信息

+ 保存1万条命令配置：

```shell
sed -i 's/^HISTSIZE=1000/HISTSIZE=10000/g' /etc/profile
```

+ 在**/etc/profile**的文件尾部添加如下行数配置信息：

```shell
######jiagu history xianshi#########
USER_IP=`who -u am i 2>/dev/null | awk '{print $NF}' | sed -e 's/[()]//g'`
if [ "$USER_IP" = "" ]
then
USER_IP=`hostname`
fi
export HISTTIMEFORMAT="%F %T $USER_IP `whoami` "
shopt -s histappend
export PROMPT_COMMAND="history -a"
######### jiagu history xianshi ##########
```

+ **使配置生效：source  /etc/profile**

## 3.检查端口、进程

检查异常端口、进程

+ 使用netstat 网络连接命令，分析可疑端口、IP、PID

+ netstat -antlp|more

+ 查看下pid所对应的进程文件路径，运行ls -l /proc/$PID/exe或file /proc/$PID/exe（$PID 为对应的pid 号），查看启动命令ps –ef |grep $PID（效果同 ps aux |grep $PID）

## 4.启动项

> 有时木马或者病毒等脚本会隐藏在开机启动中，而Linux的启动项是取决于当前用户的运行级别的。

### 4.1 Linux系统运行级别

| 运行级别 | 含义                                                      |
| -------- | --------------------------------------------------------- |
| 0        | 关机                                                      |
| 1        | 单用户模式，可以想象为windows的安全模式，主要用于系统修复 |
| 2        | 不完全的命令行模式，不含NFS服务                           |
| 3        | 完全的命令行模式，就是标准字符界面                        |
| 4        | 系统保留                                                  |
| 5        | 图形模式                                                  |
| 6        | 重启动                                                    |

+ 运行级别命令 runlevel

### 4.2开机启动配置文件

```shell
/etc/rc.local/
etc/rc.d/rc[0~6].d  #0~6 代表运行级别
```

> 当需要开机启动脚本时，做法是将可执行脚本丢在/etc/init.d目录下，接着在/etc/rc.d/rc*.d中建立软链接即可。

### 4.3 入侵排查

+ 启动项文件

```shell
ls –l /etc/rc.d/rc[0~6].d 
```

## 5.检查系统定时任务

> 入侵系统的黑客一般会使用crontab创建定期任务，如定期下载木马，定期下载挖矿病毒，定期进行内网扫描等等。

```shell
crontab -l #列出某个用户cron服务的详细内容
crontab -e #进入crontab任务编辑
crontab -r #删除所有计划任务
#不同用户的crontab 任务保存在/var/spool/cron/crontabs/用户名 文件路径下
```

> Linux 系统创建定时任务除了crontab之外，还有一个工具是**anacron**。anacron是一个**异步定时任务调度工具**，用来弥补crontab的不足。

使用案例：

+ 每天运行/home/backup.sh脚本：

```shell
vi /etc/anacrontab #加入如下内容
@daily 10 example.daily /bin/bash /home/backup.sh
#当机器在backup.sh 期望被运行时是关机的，anacron会在机器开机10分钟之后运行他，而无需在等待下一个周期。
```

入侵排查：重点关注一下目录是否存在恶意脚本

```shell
/var/spool/cron/* 
/etc/crontab
/etc/cron.d/*
/etc/cron.daily/* 
/etc/cron.hourly/* 
/etc/cron.monthly/*
/etc/cron.weekly/
/etc/anacrontab
/var/spool/anacron/*
```

## 6.检查系统服务

服务自启动使用chkconfig

```shell
chkconfig [--add][--del][--list][系统服务]
--add 增加服务
--del 删除服务
chkconfig --level httpd 2345 on 设置httpd在运行级别2、3、4、5的情况下是on(默认2345)
```

服务自启动 修改 /etc/re.d/rc.local 文件 

``` 
加入 /etc/init.d/httpd start
```

排查

```shell
chkconfig  --list  	#查看服务自启动状态
ps aux | grep crond #查看当前服务
系统在3与5级别下的启动项 
中文环境:chkconfig --list | grep "3:启用\|5:启用"
英文环境:chkconfig --list | grep "3:on\|5:on"
查看服务安装位置 ，一般是在/user/local/
service httpd start
搜索/etc/rc.d/init.d/  查看是否存在
```

## 7.检查异常文件

### 7.1 查看敏感目录

+ 如/tmp目录下的文件，同时注意隐藏文件夹，以“..”为名的文件夹具有隐藏属性

### 7.2 得到发现WEBSHELL、远控木马的创建时间，如何找出同一时间范围内创建的文件？

+ 可以使用find命令来查找，如 find /opt -iname "*" -atime 1 -type f 找出 /opt 下一天前访问过的文件

### 7.3 针对可疑文件可以使用stat进行创建修改时间

![image-20210719161637365](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210719161637.png)

## 8.日志分析

>日志默认存放位置： /var/log
>
>查看日志配置情况： more /etc/rsyslog.conf

![image-20210719161910258](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210719161910.png)

### 8.1 定位有多少IP在爆破主机的root帐号： 

```shell
grep "Failed password for root" /var/log/secure | awk '{print $11}' | sort | uniq -c | sort -nr | more
```

### 8.2 攻击者爆破哪些用户名？

```shell
grep "Failed password" /var/log/secure|perl -e 'while($_=<>){ /for(.*?) from/; print "$1\n";}'|uniq -c|sort -nr
```

### 8.3 登录成功的IP有哪些？

```shell
grep "Accepted " /var/log/secure | awk '{print $11}' | sort | uniq -c | sort -nr | more

#登录成功的日期、用户名、IP：
grep "Accepted " /var/log/secure | awk '{print $1,$2,$3,$9,$11}' 
```

### 8.4 系统添加、删除用户的记录和时间

```shell
grep "useradd" /var/log/secure 

grep "userdel" /var/log/secure
```

# Linux入侵排查工具介绍

Rootkit查杀

+ Chkrootkit：网址：http://www.chkrootkit.org

+ Rkhunter：网址：http://rkhunter.sourceforge.net

病毒查杀

+ Clamav

+ ClamAV的官方下载地址为：http://www.clamav.net/download.html

webshell查杀linux版：

+ 河马webshell查杀：http://www.shellpub.com

+ 深信服Webshell网站后门检测工具：http://edr.sangfor.com.cn/backdoor_detection.html

linux安全检查脚本

+ https://github.com/grayddq/GScan

+ https://github.com/ppabc/security_check

+ https://github.com/T0xst/linux

