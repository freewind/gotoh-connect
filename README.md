gotoh-connect
=============

Original repo: https://bitbucket.org/gotoh/connect

使用这里的connect.c可以对git的ssh协议设置代理，解决了我的大麻烦：
git push的时间由原来的100s降低到10s

设置内容参考下面的文章：

https://gist.github.com/supersheep/7079499

注意：里面提到的将connect.c的1765行注释掉，在这里的connect.c里已经做好了。

以下内容属于原作者，放在这里只是为了查看方便：

---

# Git ssh config

> 嘛，最近 github 间歇式被墙，尼玛一到晚上加班的时候就给我抽风啊，完全没法干活了。

> 所以稍微整理了一下 ssh 设置。

首先，你还是需要一台国外的可以 ssh 的机器，或者 ssh 帐号的，否则，下面就不用看了 =。=

## 步骤

### 1. ssh

平时工作的时候，还是需要让 ssh 连接保持的。

```sh
# ssh -D <port> <username>@<server>
ssh -D 8989 user@11.1.11.1
```

### 2. 修改 ~/.ssh/config

增加类似设置：

```
Host github.com
     HostName           github.com
     User               Kael
     IdentityFile       ~/.ssh/id_rsa
     IdentitiesOnly     yes
     ProxyCommand       ~/.socks5-proxywrapper %h %p
```

其中，`User` 及 `IdentityFile` 根据你的实际情况填写.

### 3. ~/.socks5-proxywrapper

创建该文件，并且内容为：

```sh
#!/bin/bash

connect -S 127.0.0.1:8989 "$@"
```

其中，端口为你 `ssh -D` 的端口

### 4. 安装 connect

下载 [connect](https://bitbucket.org/gotoh/connect/src/ee1fbc21da4ba0c733ed9b8bb7e1eeece57fc75a/connect.c?at=default) 的代码，注释掉 1765 行，编译

```sh
gcc connect.c -o connect

# 根据实际情况，cp 到你的 $PATH 目录
sudo cp connect /usr/local/bin
```

## HTTP 协议

在 ~/.gitconfig 中添加

```
[http]
    proxy = socks5://127.0.0.1:8989
```

## 说明

1. 仅支持 ssh 类型的 git remote 地址（~/.gitconfig 的 http proxy 配置起来很疼，不想去折腾了）
2. 不影响连接内网的 git 服务器（根据实际情况增加新的 Host）
3. 平时可以白天都挂着 ssh
4. 如果远程经常断开，可以在 ~/.ssh/config 设置 `ServerAliveInterval` 参数
5. 本来想写个一键安装脚本的，嘛，自己都折腾好了，懒得写了。
6. [参考](http://chunyemen.org/archives/813)
