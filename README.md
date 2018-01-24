# Linux-df--inodes-100-
这个是一个服务器空间被占满的一种错误的产生和解决方法
我们的服务器是阿里云的服务器，发现这个问题是源于今天登录数据库报错Starting MySQL.....................................The serv[FAILED]without updating PID file (/data/mysqldata/aliyun247.pid).而后提交工单，得到的回复说是磁盘已经占满，需要删除垃圾文件，后来使用df指令查看情况如下：
Filesystem 1K-blocks  Used     Available    Use% Mounted on  
/dev/xvda1 20641404   10732088 8860792      55%  / 
tmpfs      8165456    0        8165456      0%   /dev/shm 
/dev/xvdb1 516054864  5569152  484271652    2%   /data
由于本人linux知识的缺乏，以为没有占满，然后求教专业人员，经过指导人家使用的指令是df -i，得到的结果如下：
Filesystem      Inodes  IUsed   IFree      IUse% Mounted on
rootfs         1310720 1310720  0          100%  /
devtmpfs       2038816    625   2038191    1%    /dev
tmpfs          2041364      1   2041363    1%    /dev/shm
发现被占用100%，知识少真实太可怕了，那么找到问题所在该怎么解决问题呢？当然是百度，关键是怎么查找问题。
在输入框输入df -i,然后提示“df -i 100%处理”，就它吧，搜索后，查找出来如下解决方案，链接：http://www.linuxidc.com/Linux/2014-02/96836.htm，遇到的问题和我的一模一样，关键是看人家怎么解决的。
首先通过脚本查找那个目录下面文件最多，脚本如下
for i in /*; do echo $i; find $i | wc -l; done
写这个脚本的不知道是不是牛逼人，至少我是不会写，但是出于编码的通用规则，大概知道这个是遍历目录，然后统计目录下面文件的脚本，wc -l统计数据时用的么。
然后得到如下结果：
。。。省去
。。。省去
/tomcat
1
/usr
138190
/var
1165432
省去不重要部分，发现/var目录下面文件居然多达一百多万个，可能有问题，然后一级级查下去，当查到：
for i in /var/spool/postfix/*; do echo $i; find $i | wc -l; done
发现绝大部分文件在/var/spool/postfix/maildrop目录下面，然后再上网百度这个目录是干什么的，得到如下结果：
由于linux在执行cron时，会将cron执行脚本中的output和warning信息，都会以邮件的形式发送cron所有者， 而我的服务器中关闭了postfix，导致邮件发送不成功，全部小文件堆积在了maildrop目录下面。如果sendmail或者postfix正常运行，则会在/var/mail目录下也会堆积大量的邮件。

     解决方法：

     修改“/etc/crontab”

     将‘MAILTO=root’替换成‘MAILTO=""’修改之后没有成功，需要重启crond服务才可以

     也可从在crontab（crontab -e）中最前面直接加入MAILTO=""
