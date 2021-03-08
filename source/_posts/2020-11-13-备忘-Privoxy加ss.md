---
title: 备忘-Privoxy加ss
date: 2020-11-13 22:39:37
categories:
- 科学上网
---

## 一、安装shadowsocks
* 通过网址下载源码，并编译安装:
`https://github.com/shadowsocksr-backup/shadowsocksr-libev.git`

* 通过如下命令运行ss客户端，其中路径为服务器配置的路径：
`ss-local -c /home/php/socketpro/client/servers/PRU1.json`

* 通过以下命令测试，是否运行成功了:
`curl -I -x socks5://127.0.0.1:1080 https://www.google.com`
成功了的话，显示如下:
<!-- more -->

![使用ss访问google](proxygoogle.png)


## 二、设置git代理
* git操作时，设置一下代理，比较方便：
![git设置代理](gitconfig.png)

## 三、Privoxy的使用
* 开始我使用hexo init的时候，只是卡在了`git clone`这一步，也就是说，我完成了`二`就可以了。
* 但是我想到linux上很多地方都卡在墙上了，就想把http/https的一些请求，也可以通过`127.0.0.1:1080`这个端口通过ss，后来发现搞了半天没搞好，不过把目前做的内容记录一下，下次说不定找到问题了。

### 3.1 安装
* 安装autoconf，用于生成linux下的编译环境:
`yum  -y intall autoconf`

* 下载Privoxy的源码：
`https://www.privoxy.org/sf-download-mirror/`

* 安装(之前在root和另外一个用户下安装，都出错，还没明白这里为什么一定要这样新建一个用户):
    - `#cd privoxy-3.0.8-stable`
    - `#autoheader`
    - `#autoconf`
    - `#./configure --prefix=/usr/local/privoxy`
    - `#make`
    - `#useradd  privoxy  -r  -s /usr/sbin/nologin`
    - `make install`

* 编译后的文件：
![privoxy编译后文件](privoxyconfig.png)

### 3.2 配置
* 编辑`/usr/local/privoxy/etc/config`，增加：
```
listen-address 127.0.0.1:8118
forward-socks5 /   127.0.0.1:1080   .
```

* 编辑`/etc/profile`，增加：
```
export https_proxy=http://127.0.0.1:8118
export http_proxy=http://127.0.0.1:8118
export ftp_proxy=http://127.0.0.1:8118
```

### 3.3 运行
* 可以将config作为参数，直接运行
* 不过为了方便后续运行，配置成一项服务。编写privoxy的unit文件：
```
######################################################

[Unit]

Description=Privoxy Web Proxy With Advanced Filtering Capabilities

Wants=network-online.target

After=network-online.target

[Service]

Type=forking

PIDFile=/run/privoxy.pid

ExecStart=/usr/local/privoxy/sbin/privoxy --pidfile /run/privoxy.pid --user privoxy /usr/local/privoxy/etc/config

[Install]

WantedBy=multi-user.target

##########################################################
```

* 启用服务
```
systemctl daemon-reload   # 加载unit文件
systemctl enable privoxy  # 开机启动
systemctl start privoxy   # 启动服务
systemctl status privoxy  # 查看服务状态
```

* 服务状态
![Privoxy服务开始后的状态](privoxy_service.png)

* 查看是否正常运行：
  - `ps aux | grep privoxy`
  - `ss -tan | grep 8118`

* 在运行服务时，可能会遇到一些问题。其中一些问题出现在运行Privoxy之前的话，直接搜一下相关问题就行；如果出现在运行Privoxy的过程中的话，可以看Privoxy的日志:`/var/log/privoxy/logfile`

* 遇到的一个问题，日志中说，找不到一些action配置文件。发现Privoxy运行时，找配置的目录在`/usr/local/privoxy/etc`下，而这些action文件不知为啥，在`/usr/local/etc/privoxy`下，所以拷贝了一下。

* 有个博客介绍privoxy比较好：
`https://www.cnblogs.com/hongdada/p/10787924.html`

## 四、遗留问题和解决
* 运行了ss和privoxy之后，发现用`curl https://www.google.com`一直报503错误，尚未解决
* 后来发现，更新yum源为阿里源，可以直接通过`yum install privoxy`进行安装。原来安装的privoxy版本也比较低了



