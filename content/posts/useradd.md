+++
categories = ["liux"]
date = "2017-08-03T13:31:13+08:00"
tags = ["linux"]
title = "useradd命令行tab键路径补全失效"
+++

新增用户 `useradd -m $username` 没指定 *shell* 路径  默认 为*sh*,应该指定 */bin/bash*

`sudo useradd username -s /bin/bash`
如果出现这种情况，修改方法也很简单，修改 /etc/passwd 文件，进入这个文件以后，你会看到你的帐号信息，大概应该是这样的：

```
root:x:0:0:root:/root:/bin/bash
work:x:1000:1000::/home/work
```

你会发现work的root有区别，我们只需要简单的调整为：

`work:x:1000:1000::/home/work:/bin/bash`

然后保存并退出此文件，然后重新连接远程主机，你在试试，是不是已经恢复了？
