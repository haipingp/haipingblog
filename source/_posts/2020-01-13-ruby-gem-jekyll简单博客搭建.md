---
title: ruby-gem-jekyll简单博客搭建
date: 2020-01-13 23:30:09
categories:
- 博客搭建
---


## 一、服务器环境
* 我使用的服务器环境是：Ubuntu 18.04 Server版 

<!-- more -->

## 二、搭建步骤
### 2.1. 安装ruby
* 由于之前有一次安装的ruby版本比较低，可能影响了后面的rubygems的安装，所以这一次我采用的目前最新的稳定版ruby 2.7.0
* 之前有一次因为ruby版本低，卸载重新安装的痛苦经历，卸载之后有一些其他软件不能用了（具体忘记是啥软件了）。这一次安装ruby，看了网上很多安装教程推荐使用Rbenv这个轻量级的ruby版本管理器来安装，找到了一个靠谱的安装教程和一个参考链接很重要：
https://linuxize.com/post/how-to-install-ruby-on-ubuntu-18-04/。
https://gorails.com/setup/ubuntu/18.04
有了它，后面只需要解决一些边边角角的问题了
* 由于默认Rbenv是没有版本控制功能的，需要安装`ruby-build`来完成，以下是按照上面链接中的安装步骤:

1. 首先需要安装ruby-build用来安装ruby的准备工具，从源码安装：
``` bash
sudo apt update
sudo apt install git curl libssl-dev libreadline-dev zlib1g-dev autoconf bison build-essential libyaml-dev libreadline-dev libncurses5-dev libffi-dev libgdbm-dev
``` 

2. 运行curl命令安装rbenv和ruby-build ：
``` bash
curl -sL https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-installer | bash -
```
`curl -s`表示使用静默模式跳转链接，`curl -L`表示允许重定向到一个新的地址，不像简单的get一个链接
后面的`bash -`命令通过前面的管道标识符`|`执行前面下载的脚本
上述脚本会从将rbenv和ruby-build仓库从GitHub拷贝到`~/.rbenv`目录下。同时，这个安装脚本会验证此次安装，输出如下：

3. 将`$HOME/.rbenv/bin`添加到环境变量中： 
``` bash
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
source ~/.bashrc
```

4. 通过rbenv安装ruby:
``` bash
rbenv install 2.7.0
```
该命令在线安装超级慢：会一直卡在一个下载链接的进度上，于是我复制了下载链接中的包名，并到ruby官网的镜像地址中下载了该包，并放在`.rbenv/cache`目录下；
再执行`rbenv install 2.7.0`命令，显示进度为“安装ruby 2.7.0”而不是下载包的进度了

5. 为了防止安装了多个ruby版本，通过下面的命令指定默认使用的ruby版本:
``` bash
rbenv global 2.7.0
```

6. 下载RubyGems(rubygems-3.1.2.zip)，并在目录下执行`ruby setup.rb`:

### 2.2 安装jekyll
``` bash
gem install jekyll
```

### 2.3 使用jekyll
* 需要按照jekyll的格式建立一定格式的文件夹
* 其中`_posts`文件夹下放的就是博客文章
* 一些博客模板：
https://github.com/poole/poole
http://jekyllthemes.org/

* 使用`jekyll serve`报错时，尝试使用`bundle exec jekyll serve`启动，起来了之后发现无法在本地通过ip地址访问的话，采用`bundle exec jekyll serve --host 0.0.0.0`启动，通过`http://192.168.56.101:8008/`，端口可以在`_config.yml`中配置

## 三、遇到的问题
* 一个依赖未找到：

`in 'block in materialize': Could not find public_suffix-3.0.0 in any of the sources (Bundler::GemNotFound)`

解决方法：按照提示通过gem安装：`gem install public_suffix --version 3.0.0`

* 还有东西没找到：

`spec_set.rb:87:in 'block in materialize': Could not find addressable-2.5.2 in any of the sources (Bundler::GemNotFound)`

解决方法：`bundle update`或`bundle install`，这个过程可能有点慢







