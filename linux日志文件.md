2.2.1 linux日志的基本介绍
日志的基本介绍：

日志文件是重要的系统信息文件，其中记录了很多重要的系统事件，包括用户的登录信息、系统的启动信息、系统的安全信息、邮件相关信息、各种服务相关信息等。

日志对于安全来说也很重要，它记录了系统每天发生的各种事情，通过日志来检查错误发生的原因，或者受到攻击时攻击者留下的痕迹。

一句话理解日志：日志是用来记录重大事件的工具。

那么linux系统中的日志信息保存在哪里呢?绝大多数的日志文件保存在/var/log目录中。

下面我们看看系统中的常用日志：

下面我们就来看看重要的日志文件：
我们使用lastlog来查看/var/log/lastlog。
[root@xq100  log]# lastlog
[root@xq100  log]# who
root     pts/0        2022-09-09 15:52 (desktop-eki0p48)

应用案例：使用root用户通过xshell7登录，第一次使用错误的密码，第二次使用正确的密码登录成功，查看日志文件/var/log/secure里面有没有记录相关信息。
[root@xq100 log]# cat secure
[root@xq100 log]# echo '' > secure   # 情况secure文件中的内容
[root@xq100 log]# cat secure  # 发现没有任何文件
    
[root@xq100 log]#
接下来我们连续两次输入错误密码，然后第三次输入正确的密码。
2.2.2 linux日志管理服务
centos7日志服务是rsyslogd,centos6日志服务是syslogd。rsyslogd日志服务功能更加强大。rsyslogd的使用、日志文件的格式和syslogd是兼容的。
现在，我们思考一个问题，就是在linux系统中是谁帮助我们将日志信息记录在不同的日志文件里面去的?我们通过一幅图就能理解linux进行日志管理的原理。

所以linux日志服务帮助我们进行日志管理，是借助了/etc/rsyslog.conf配置文件来实现的。
我们可以去查看这个日志文件
[root@xq100 log]# more /etc/rsyslog.conf
查询linux的rsyslogd服务是否启动 ps aux | grep “rsyslog” | grep -v “grep”
[root@xq100 log]#  ps aux | grep "rsyslog" -- 查看rsyslogd服务是否启动
root       1336  0.0  0.3 222760  5816 ?        Ssl  15:29   0:00 /usr/sbin/rsyslogd -n
root       4002  0.0  0.0 112716   960 pts/0    R+   16:28   0:00 grep --color=auto rsyslog
[root@xq100 log]#  ps aux | grep "rsyslog" | grep -v "grep" -- 反向过滤，将包含grep的指令去掉
root       1336  0.0  0.3 222760  5808 ?        Ssl  15:29   0:00 /usr/sbin/rsyslogd -n

查询rsyslogd服务的启动状态 systemctl list-unit-files | grep rsyslog
[root@xq100 log]# systemctl  list-unit-files | grep rsyslog
rsyslog.service 

2.2.3 日志服务管理的配置文件
管理日志的配置文件/etc/rsyslog.conf。那么我们应该怎么去理解这个配置文件里面的内容？
日志文件的格式是*.* 存放的日志文件

那么这两个*是什么意思呢？
第一个*: 代表日志类型

日志类型可以分为：

日志类型	日志描述
auth	pam产生的日志
authpriv	ssh ftp等登陆信息的验证信息
corn	时间任务相关的信息
kern	内核相关
lpr	打印相关的信息
mail	邮件相关的信息
mark(syslog)-rsyslog	服务内部信息
news	新闻组
user	用户程序产生的相关信息
local 1-7	自定义日志设备

第二个*:代表日志级别

日志级别	日志级别的描述信息
debug	有调试信息的，记录的日志信息最多
info	一般日志信息，最常用
notice	提醒信息，需要检查一下程序了，不理会可能会出现错误。
warning	警告信息,当出现警告时，你的程序可能已经出现了问题,但不影响程序正常运行,尽快进行处理，以免导致服务宕掉。
err	错误信息，出现这一项时，已经挑明服务出现了问题,服务都无法确认是否能正常运行。
crit	严重级别，阻止整个系统或程序不能正常工作的信息
alert	需要立即修改的信息
emerg	记录内核崩溃等信息
none	什么都不记录
注意：从上到下，日志级别从低到高，记录的信息也越来越少。
由日志服务rsyslogd记录的日志文件，日志文件的格式包含以下4列：
事件产生的时间
产生事件的服务器(主机名)
产生事件的服务名和程序名
事件的具体信息
日志查看实例：查看一下/var/log/secure日志，这个日志记录的是用户验证和授权方面的信息，来分析如何查看：

2.2.4 自定义日志服务
自定义日志服务管理实例：
在/etc/rsyslog.conf中添加一个日志文件/var/log/xq.log，当有事件发生时(比如sshd相关服务的事件)。该文件会接收到信息并保存。比如我们登录 重启linux系统的时候，看看对应的日志信息是否成功保存。
[root@xq100 log]# vim /etc/rsyslog.conf 
我们编辑/etc/rsyslog.conf文件，添加自定义日志信息：
接下来，我们创建xq.log日志文件
[root@xq100 ~]# cd /var/log
[root@xq100 log]# vim xq.log
[root@xq100 log]# cat xq.log 
[root@xq100 log]# reboot 
我们查看日志xq.log文件：
[root@xq100 log]# cat xq.log | grep sshd
Sep  9 17:25:24 xq100 sshd[1292]: Server listening on 0.0.0.0 port 22.
Sep  9 17:25:24 xq100 sshd[1292]: Server listening on :: port 22.
Sep  9 17:25:33 xq100 sshd[1682]: Accepted password for root from 192.168.10.1 port 49929 ssh2
Sep  9 17:25:33 xq100 sshd[1682]: pam_unix(sshd:session): session opened for user root by (uid=0)
我们发现，日志信息成功被记录了。


2.2.5 日志轮替
日志轮替就是按照一定的规则，将一些不需要的旧的文件删掉。
日志轮替，我们使用了/etc/logrotate.conf这个配置文件进行管理的。
[root@xq100 logrotate.d]# cat /etc/logrotate.conf
# see "man logrotate" for details
# rotate log files weekly
weekly
# keep 4 weeks worth of backlogs
rotate 4
# create new (empty) log files after rotating old ones
create
# use date as a suffix of the rotated file
dateext
# uncomment this if you want your log files compressed
#compress
# RPM packages drop log rotation information into this directory
include /etc/logrotate.d
# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {
    monthly
    create 0664 root utmp
	minsize 1M
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}
# system-specific logs may be also be configured here.

我们也可以自定义日志轮替规则。
日志文件地址 {
	参数
}
参数：
daily：轮替周期 每天
weekly：轮替周期 每周
monthly：轮替周期 每月
rotate [num]：保存日志文件的个数
compress：轮替时对旧日志进行压缩
create mode owner group：建立新日志的同时指定权限 所有者 所属组
mail address：日志轮替时输出内容通过邮件发送到指定的邮件地址
missingok：如果日志不存在则忽略日志的警告信息
notifempty：如果日志为空文件则不进行日志轮替
minsize [size]：日志轮替的最小值 即超过该大小才会轮替 否则到达轮替周期也不会轮替
size [size[：日志达到指定大小进行轮替 而不是按照轮替的时间周期
dateext：使用日期作为日志轮替文件的后缀
sharedscripts：在此关键字之后的脚本只执行一次
prerotate/endscripts：在日志轮替之前执行脚本命令
postrotate/endscripts：在日志轮替之后执行脚本命令
我们查看可以自定义的日志轮替文件：
[root@xq100 logrotate.d]# cd /etc/logrotate.d
[root@xq100 logrotate.d]# ll
[root@xq100 logrotate.d]# cat bootlog 
/var/log/boot.log
{
    missingok   # 如果日志不存在则忽略该日志的警告信息
    daily  # 每天轮替一次
    copytruncate  # 先把原始文件拷贝一份重命名，然后把原始文件清空
    rotate 7  # 仅保留7个日志备份
    notifempty  # 如果日志为空文件则不进行日志轮替
}
[root@xq100 logrotate.d]#

应用实例：将我们自定义的日志文件加入日志轮替。
第一种方法：直接在/etc/logrotate.conf配置文件中写入对该日志的轮替策略。

第二种方法：在/etc/logrotate.d目录下新建该日志的轮替文件。在该轮替文件中定义轮替策略，因为该目录中的文件都会被包含到主配置文件logrotate.conf中。

我们推荐使用第二种方法，因为系统中需要轮替的日志文件非常多，为了可读性方便，建立单独定义轮替规则。
[root@xq100 logrotate.d]# cd /etc/logrotate.d
[root@xq100 logrotate.d]# vim xqlog

定义轮替规则：
注意：logrotate.conf只是定义了日志轮替的规则，那么日志轮替(在指定的时间备份日志)的这个动作，依赖于系统定时任务。可以在 /etc/cron.daily/ 中发现一个可执行文件logrotate。
[root@xq100 logrotate.d]# cd /etc/cron.daily/
[root@xq100 cron.daily]# ll

2.2.6 内存日志
在linux中，有一部分日志信息是没有写到日志文件里面去的，而是写在内存中的。这些日志的特点是日志信息都在随时发生变化。比如linux内核的日志信息。内存日志还有一个特点是linux系统在重新启动的时候，内存日志就会被清空。
操作内存日志的常用指令如下：
journalctl查看所有的内存日志
journalctl -n 3查看最新3条
journalctl --since 15:00 --until 15:10查看区间时间内的日志 可加日期
journalctl -p err查看报错日志
journalctl -o verbose日志详细内容
