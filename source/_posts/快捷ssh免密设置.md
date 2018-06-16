---
title: ssh免密设置
date: 2018-06-16 23:05:47
tags: linux
---

常用的 `ssh` 或者 `scp` 命令一般都会选择配置免密码登录，不然每次输入密码会挺痛苦的。

常规是手动将公钥拷贝到目标主机 `~/.ssh/authorized_keys`，再对文件目录访问权限进行配置。
但现在有了更方便的操作方法，一个方便的命令 **`ssh-copy-id`**，特此记录备忘。



### 设置 ssh 免密登录

#### 1. 在本地生成　ssh　公钥和秘钥


```
#　生成公钥密钥对

ssh-keygen -b 4096 -t rsa

```

这里指定的是 `rsa`　加密的类型，长度为4096位。
生成的秘钥就存储在 `~/.ssh/id_rsa` 中，公钥在 `~/.ssh/id_rsa.pub` 中。


#### 2. 将公钥上传到目标主机

这一步就可以使用 `ssh-copy-id` 来完成, 而这个命令包含在**`openssh-client`**软件包中。


```shell
# 先安装`openssh-client`软件包
sudo apt-get install openssh-client

# 确认ssh-copy-id可用
which ssh-copy-id

# 添加秘钥到目标主机
ssh-copy-id username@remote-server
```

其中 username 为目标主机的用户名, remote-server 为目标主机的host或者ip；
本次登录需要输入密码，登录后`ssh-copy-id`命令就自动完成了其他的免密设置。


### 参考


1. {% link ssh-key：两个简单步骤实现ssh无密码登录 https://www.linuxdashen.com/ssh-key%EF%BC%9A%E4%B8%A4%E4%B8%AA%E7%AE%80%E5%8D%95%E6%AD%A5%E9%AA%A4%E5%AE%9E%E7%8E%B0ssh%E6%97%A0%E5%AF%86%E7%A0%81%E7%99%BB%E5%BD%95 %}

2. {% link Ubuntu Manpages ssh-copy-id http://manpages.ubuntu.com/manpages/precise/man1/ssh-copy-id.1.html %}
