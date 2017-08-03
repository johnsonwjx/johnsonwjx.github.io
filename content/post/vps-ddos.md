+++
categories = ["vps"]
date = "2017-08-02T10:33:04+08:00"
tags = ["vps","ddos"]
title = "(转载)DDoS deflate - Linux下防御/减轻DDOS攻击"

+++
原文 <https://www.vpser.net/security/ddos-deflate.html>
前言

互联网如同现实社会一样充满钩心斗角，网站被DDOS也成为站长最头疼的事。在没有硬防的情况下，寻找软件代替是最直接的方法，比如用iptables，但是iptables不能在自动屏蔽，只能手动屏蔽。今天要说的就是一款能够自动屏蔽DDOS攻击者IP的软件：DDoS deflate。

DDoS deflate介绍

DDoS deflate是一款免费的用来防御和减轻DDoS攻击的脚本。它通过netstat监测跟踪创建大量网络连接的IP地址，在检测到某个结点超过预设的限 制时，该程序会通过APF或IPTABLES禁止或阻挡这些IP.

DDoS deflate官方网站：http://deflate.medialayer.com/

如何确认是否受到DDOS攻击？

执行：

netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -n
执行后，将会显示服务器上所有的每个IP多少个连接数。

以下是我自己用VPS测试的结果：

li88-99:~# netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -n
1 114.226.9.132
1 174.129.237.157
1 58.60.118.142
1 Address
1 servers)
2 118.26.131.78
3 123.125.1.202
3 220.248.43.119
4 117.36.231.253
4 119.162.46.124
6 219.140.232.128
8 220.181.61.31    VPS侦探 https://www.vpser.net/
2311 67.215.242.196
每个IP几个、十几个或几十个连接数都还算比较正常，如果像上面成百上千肯定就不正常了。

1、安装DDoS deflate

wget http://www.inetbase.com/scripts/ddos/install.sh   //下载DDoS  deflate
chmod 0700 install.sh    //添加权限
./install.sh             //执行
2、配置DDoS deflate

下面是DDoS deflate的默认配置位于/usr/local/ddos/ddos.conf ，内容如下：

##### Paths of the script and other files
PROGDIR="/usr/local/ddos"
PROG="/usr/local/ddos/ddos.sh"
IGNORE_IP_LIST="/usr/local/ddos/ignore.ip.list"  //IP地址白名单
CRON="/etc/cron.d/ddos.cron"    //定时执行程序
APF="/etc/apf/apf"
IPT="/sbin/iptables"

##### frequency in minutes for running the script
##### Caution: Every time this setting is changed, run the script with --cron
#####          option so that the new frequency takes effect
FREQ=1   //检查时间间隔，默认1分钟

##### How many connections define a bad IP? Indicate that below.
NO_OF_CONNECTIONS=150     //最大连接数，超过这个数IP就会被屏蔽，一般默认即可

##### APF_BAN=1 (Make sure your APF version is atleast 0.96)
##### APF_BAN=0 (Uses iptables for banning ips instead of APF)
APF_BAN=1        //使用APF还是iptables。推荐使用iptables,将APF_BAN的值改为0即可。

##### KILL=0 (Bad IPs are'nt banned, good for interactive execution of script)
##### KILL=1 (Recommended setting)
KILL=1   //是否屏蔽IP，默认即可

##### An email is sent to the following address when an IP is banned.
##### Blank would suppress sending of mails
EMAIL_TO="root"   //当IP被屏蔽时给指定邮箱发送邮件，推荐使用，换成自己的邮箱即可

##### Number of seconds the banned ip should remain in blacklist.
BAN_PERIOD=600    //禁用IP时间，默认600秒，可根据情况调整
用户可根据给默认配置文件加上的注释提示内容，修改配置文件。

查看/usr/local/ddos/ddos.sh文件的第117行

netstat -ntu | awk ‘{print $5}’ | cut -d: -f1 | sort | uniq -c | sort -nr > $BAD_IP_LIST

修改为以下代码即可！

netstat -ntu | awk ‘{print $5}’ | cut -d: -f1 | sed -n ‘/[0-9]/p’ | sort | uniq -c | sort -nr > $BAD_IP_LIST

喜欢折腾的可以用Web压力测试软件测试<https://www.vpser.net/opt/webserver-test.html>一下效果，相信DDoS deflate还是能给你的VPS或服务器抵御一部分DDOS攻击，给你的网站更多的保护。
