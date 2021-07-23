# Linux系统加固思路

![image-20210702162703685](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210712225158.png)

## 1.系统账号安全

### 1.1 禁用或删除无用账号

目的：减少系统无用账号，降低安全风险

检查方法：

+ cat /etc/passwd
+ cat /etc/shadow

> 查看账户、口令文件，与系统管理员确认不必要的账号。对于一些保留的系统伪账户如：bin、sys、adm、uucp、lp、nuucp、hpdb、www、daemon等可根据需要进行锁定登陆

操作步骤：

+ 使用命令 userdel <用户名> 删除不必要的账户
+ 使用命令 passwd -l <用户名> 锁定不必要的账户
+ 使用命令passwd -u <用户名> 解锁必要的用户

### 1.2 检查特殊账号

目的：检查是否存在空口令或和root权限的账号

操作步骤：

（1）查看空口令和root权限账号，确认是否存在异常账号：

+ 使用命令 awk -F: '($2=="")' /etc/shadow 查看空口令账号
+ 使用命令 awk -F: '($3==0)' /etc/passwd 查看UID为0的账号

（2）加固口令账号：

+ 使用命令 passwd <用户名> 为空口令账号设置密码
+ 确认UID为0的账号只有root

### 1.3 添加口令策略

目的：加强口令额复杂度等，降低被猜解的可能性

检测方法：查看密码策略设置

+ ```shell
  cat /etc/login.defs|grep PASS
  ```

操作步骤：

（1）使用命令 vi /etc/login.defs 修改配置文件

+ ``` shell
  PASS_MAX_DAYS 90 #新建用户的密码最长使用天数
  PASS_MIN_DAYS 0 #新建用户的密码最短使用天数
  PASS_WARN_AGE 7 #新建用户的密码到期前提醒天数
  ```

（2）使用chage命令修改用户设置

+ ``` shell
  chage -m 0 -M 30 -E 2022-01-01 -W 7 <user> #将user的密码最长使用天数设为30，最短使用天数设为0，密码于2022-01-01过期，过期前7天警告用户

（3）设置密码复杂度

+ ``` shell
  vim /etc/pam.d/system-auth #做如下修改
  ```

+ 在password requisite pam_cracklib.so后添加try_fist_pass retry=3 dcredit=-1 lcredit=-1 ucredit=-1 ocredit=-1 minlen=8

+ 上述修改意为：至少包含一个数字、一个小写字母、一个大写字母、一个特殊字符且密码长度大于等于8

（4）设置连续输错3次密码改账号锁定5分钟

+ ``` shell
  vi /etc/pam.d/common-auth #添加如下内容
  ```

+ ``` shell
  auth required pam_tally.so onerr=fail deny=3 unlock_time=300
  ```

### 1.4 限制用户su

目的：限制能su到root的用户

操作步骤：

+ ```shell
  vi /etc/pam.d/su #添加如下行
  ```

+ ```shell
  auth required pam_wheel.so group=test
  #仅允许test组用户su到root
  ```

### 1.5 禁止root用户直接登陆



### 1.6 多次登陆失败锁定



## 2.文件系统安全



