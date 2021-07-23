# Windows入侵排查思路

![image-20210702184920873](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210712225523.png)



## 1.检查系统账号安全

 检查系统是否存在弱口令、可疑账号、新增账号

+ 询问管理员

+ 查看本地用户和组（cmd=>lusrmgr.msc），禁用或者删除新增或者可疑账号

+ 查看隐藏账号和克隆账号
  + 查看注册表 HKEY_LOCAL_MACHINE\SAM\SA
  + D盾查杀 M\Domains\Account\Users

![image-20210721112514868](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721112514.png)



## 2.检测异常端口、进程

### 2.1 检查端口连接情况，是否有远程连接、可疑连接。

+ netstat -ano 查看目前的网络连接，定位可疑的ESTABLISHED

+ 根据netstat 定位出的pid，再通过tasklist命令进行进程定位 tasklist | findstr “PID”

  

### 2.2 检查进程

+ 开始--运行--输入msinfo32，依次点击“软件环境→正在运行任务”就可以查看到进程的详细信息，比如进程路径、进程ID、文件创建日期、启动时间等。

![image-20210721112822125](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721112822.png)



+ 打开D盾_web查杀，进程查看关注没有签名信息的进程

![image-20210721112925529](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721112925.png)



+ 通过微软官方提供的process explorer等工具进行排查

![image-20210721113737749](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721113737.png)



### 2.3 **查看可疑的进程及其子进程。可以通过观察以下内容：**   

+ 没有签名验证信息的进程   

+ 没有描述信息的进程   

+ 进程的属主   

+ 进程的路径是否合法   

+ CPU或内存资源占用长时间过高的进



## 3.检查启动任务、计划任务、服务

### 3.1 检查启动项

检查服务器是否有异常的启动项

1. 登陆服务器，【单击开始】=》【所有程序】=》【启动】，默认情况下此目录是一个空目录，确认是否有非业务程序在该目录下

   ![image-20210721114936396](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721114936.png)



2. win+r输入msconfig或打开任务管理器，查看是否存在命名异常的启动项目，是则取消勾选并找到改文件并删除

   ![image-20210721115142918](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721115142.png)



3. win+r输入regedit打开注册表，查看开机启动项是否正常，特别注意如下3个注册表

   ◆HKEY_CURRENT_USER\software\micorsoft\windows\currentversion\run

   ◆HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run

   ◆HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Runonce

> 检查右侧是否有启动异常的项目，如有请删除并进行病毒查杀，清楚残留病毒或木马

![image-20210721115531771](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721115531.png)



4. 利用安全软件查看启动项、开机时间管理等



5. 组策略，运行gpedit.msc

   ![image-20210721115658948](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721115659.png)



### 3.2 计划任务

+ 单击【开始】=》【设置】=》【控制面板】=》【任务计划】，查看计划任务属性，便可以发现木马文件路径

  ![image-20210721115820079](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721115820.png)



+ 在cmd中输入at，检查计算机与网络上的其他计算机间的会话或计划任务，如有则确认是否为正常连接

> AT 命令已弃用。请改用 schtasks.exe







## 4.检查系统相关信息

### 4.1 服务自启动

win+r输入services.msc，注意服务状态和启动类型，检查是否有异常服务

![image-20210721120123272](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721120123.png)



### 4.2 查看系统信息

在cmd中输入systeminfo查看系统版本以及补丁信息

![image-20210721135500384](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721135500.png)



### 4.3 查找可疑目录及文件

+ 查看用户目录，新建账户会在这个目录生成一个用户目录，查看是否创建用户目录

  ![image-20210721135931075](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721135931.png)

 

+ win+r输入%UserProfile%\Recent，分析最近打开的文件

  ![image-20210721140130667](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721140130.png)



+ 在服务器各个目录，可根据文件夹列表时间进行排序查找可疑文件

+ 回收站、浏览器下载目录、浏览器历史记录

+ 修改时间在创建时间之前的为可疑文件

  ![image-20210721140749904](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721140749.png)



## 5.自动化查杀

### 5.1 病毒查杀

检查方法：下载安全软件，更新最新病毒库，进行全盘扫描

+ 360
+ 卡巴斯基
+ 火绒安全
+ ......



### 5.2 webshell查杀

检查方法：选择具体站点路径进行webshell查杀，建议使用两款webshell查杀工具同时查杀，可互相补充规则库的不足

![image-20210721141044348](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721141044.png)



## 6.日志分析

### 6.1 系统日志分析方法

+ 前提：win+r输入secpol.msc开启审核策略，若日后系统出现故障、安全事故则可以查看系统的日志文件，排除故障及追查入侵者信息等

  ![image-20210721141601683](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721141601.png)

  

+ win+r输入eventvwr.msc打开【事件查看器】

  ![image-20210721141633441](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721141633.png)

  

+ 导出应用程序日志、安全日志、系统日志并利用log parser进行分析



### 6.2 查看管理员账户是否存在异常

不借助工具：利用上述的事件查看器

对于Windows事件日志分析，不同的EVENT ID代表了不同的意义，每个成功登陆的事件都会标记一个登陆类型，不同登陆类型代表不同的方式

![image-20210721143215305](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721143215.png)



+ 利用eventlog事件来查看系统账号登陆情况

  1. 同意打开事件查看器
  2. 单击“安全”，查看安全日志
  3. 在右侧点击筛选当前日志，输入事件ID进行筛选

  > 输入事件 ID：4625 进行日志筛选，发现事件 ID：4625，事件数 175904，即用户登录失败了 175904次，那么这台服务器管理员账号可能遭遇了暴力猜解。 

  ![image-20210721143440774](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721143440.png)



借助工具：Log Parser、Event Log Explorer

+ Log Parser提取登陆成功的用户名和IP

  ![image-20210721143751168](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721143751.png)

``` shell
LogParser -i:EVT –o:DATAGRID "SELECT EXTRACT_TOKEN(Message,13,' ') as EventType,TimeGenerated as LoginTime,EXTRACT_TOKEN(Strings,5,'|') as Username,EXTRACT_TOKEN(Message,38,' ') as Loginip FROM 1.evtx where EventID=4624"

```

+ Event Log Explorer 是一款非常好用的 Windows 日志分析工具。可用于查看，监视和分析跟事件记录，包括安全，系统，应用程序和其他微软 Windows 的记录被记载的事件，其强大的过滤功能可以快速的过滤出有价值的信息。

  ![image-20210721144122669](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721144122.png)



### 6.3 WEB访问日志分析方法

+ 找到web日志并打包到本地方便分析（Windows推荐使用EmEditor）

  ![image-20210721145118853](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721145118.png)



常见web中间件的默认日志目录

+ Apache

  C:\Program Files (x86)\Apache Software Foundation\Apache2.2\logs\access.log 访问日志

  C:\Program Files (x86)\Apache Software Foundation\Apache2.2\logs\error.log 错误日志

+ IIS

  %SystemDrive%\inetpub\logs\LogFiles

+ Nginx

  /etc/nginx/nginx.conf 在这个配置文件进行设置

+ Tomcat

  TOMCAT_HOME/logs/catalina.out



WEB访问日志分析技巧

+ 第一种：确认入侵的时间范围，以此为线索，查找这个时间范围内可疑的日志进行进一步排查，最终确定攻击者，还原攻击过程

+ 第二种：攻击者在入侵网站后，通常会留下后门维持权限以方便二次访问，我们可以找到该文件，并以此为线索展开分析

  ![image-20210721145620802](https://raw.githubusercontent.com/lixbao/PicGo/main/img/20210721145620.png)



以Apache access.log日志为例

+ 日志1：可以得知：来源IP、时间、行为、访问的域名、状态码

```shell
10.10.10.1 - - [15/Jul/2019:15:06:14 +0800] "POST /m/login.php?op=login HTTP/1.1" 200 243 "http://10.10.10.128/m/login.php" "Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko“
```



+ 日志2：可以得知用户在什么 IP、什么时间、用什么操作系统、什么浏览器的情况下访问了你网站的哪个页面，是否访问成功。

```shell
10.10.10.1 - - [15/Jul/2019:15:06:14 +0800] "POST /m/login.php?op=login HTTP/1.1" 200 243 "http://10.10.10.128/m/login.php" "Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko“
```

