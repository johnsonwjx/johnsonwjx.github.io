+++
categories = ["vps"]
date = "2017-08-02T08:20:21+08:00"
tags = ["vps","tools"]
title = "(转载)SSH暴力攻击的几种防范方法"

+++

原文 <http://securityer.lofter.com/post/1d0f3ee7_e0240b4>

# Fail2ban

fail2ban是一款实用软件，可以监视系统日志，然后匹配日志的错误信息（正则式匹配）执行相应的屏蔽动作。

功能和特性

1. 支持大量服务。如sshd,apache,qmail,proftpd,sasl等等

2. 支持多种动作。如iptables,tcp-wrapper,shorewall(iptables第三方工具),mail notifications(邮件通知)等等。

3. 在logpath选项中支持通配符

4. 需要Gamin支持(注：Gamin是用于监视文件和目录是否更改的服务工具)

5. 需要安装python,iptables,tcp-wrapper,shorewall,Gamin。如果想要发邮件，那必需安装postfix或sendmail

## 测试环境

```bash
 docker pull superitman/fail2ban
 docker run -d -it \-v /var/log:/var/log \--name fail2ban \--net host \--privileged \superitman/fail2ban:latest
```

# DenyHosts

DenyHosts是用Python语言写的一个程序，它会分析sshd的日志文件（/var/log/secure），当发现重复的攻击时就会记录IP到/etc/hosts.deny文件，从而达到自动屏IP的功能

## 测试环境

```bash
 docker pull custa/docker-denyhosts
 docker run -d --restart=always --name denyhosts \ -v /var/log/secure:/var/log/secure -v /etc/hosts.deny:/etc/hosts.deny \ custa/docker-denyhosts
```

## denyhosts.cfg配置文件说明

```bash
SECURE_LOG = /var/log/secure #ssh 日志文件，它是根据这个文件来判断的，如还有其他的只要更改名字即可，例如将secure改为secure.1等 HOSTS_DENY = /etc/hosts.deny #控制用户登陆的文件，将多次连接失败的IP添加到此文件，达到屏蔽的作用<br>
PURGE_DENY = #过多久后清除已经禁止的，我这里为空表示永远不解禁 BLOCK_SERVICE = sshd #禁止的服务名，如还要添加其他服务，只需添加逗号跟上相应的服务即可 DENY_THRESHOLD_INVALID = 1 #允许无效用户失败的次数<br>
DENY_THRESHOLD_VALID = 2 #允许有效用户登录失败的次数<br>
DENY_THRESHOLD_ROOT = 3 #允许root登录失败的次数<br>
HOSTNAME_LOOKUP=NO # 是否做域名反解，这里表示不做 ADMIN_EMAIL = ... #管理员邮件地址,它会给管理员发邮件 DAEMON_LOG = /var/log/denyhosts #自己的日志文件

其他： AGE_RESET_VALID=5d #（h表示小时，d表示天，m表示月，w表示周，y表示年） AGE_RESET_ROOT=25d AGE_RESET_RESTRICTED=25d AGE_RESET_INVALID=10d #用户的登陆失败计数会在多长时间后重置为0 RESET_ON_SUCCESS = yes #如果一个ip登陆成功后，失败的登陆计数是否重置为0 DAEMON_SLEEP = 30s #当以后台方式运行时，每读一次日志文件的时间间隔。 DAEMON_PURGE = 1h #当以后台方式运行时，清除机制在 HOSTS_DENY 中终止旧条目的时间间隔,这个会影响PURGE_DENY的
```

# SshGuard

sshguard是用**C编写的，所以它在运行时占用的内存和处理器资源比较少**，但仍能收到很好的效果

sshguard在机器上作为一个很小的后台驻留程序来运行，能够接收日志消息（可接收来自诸多途径的消息，比如来自系统日志）。如果它确定了X地址对Y服务造成危害，就会启动机器里面防火墙（注：防火墙须得到支持）中的规则，以阻止X地址。

shguard阻止X地址的状态会保持一段时间，但是之后会自动解禁。

请注意：尽管名称很容易让人误解，但是sshguard在默认情况下能够阻止针对许多服务的攻击，不仅仅是针对SSH的攻击，还能针对几种ftpd、Exim和dovecot等服务的攻击。它能够适用于各大防火墙系统，并且提供对IPv6、白名单、悬挂和消息验证等功能的支持

官方URL

<http://www.sshguard.net>

# Logcheck

Logcheck用来分析庞大的日志文件，过滤出有潜在安全风险或其他不正常情况的日志项目，然后以电子邮件的形式通知指定的用户，可用crontab设置自动运行.此工具只能做到检测但与其他工具如防火墙结合,可以做到自动防范的效果

官方URL

<http://logcheck.org>
