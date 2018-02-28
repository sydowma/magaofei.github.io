---
title: macOS 下常用软件安装和配置
date: 2018-2-28 12:28
tags:
- macOS
---

#  macOS 下常用软件安装和配置



适用于刚刚接触macOS 系统的同学。



macOS 和Windows不同，所以不要把Windows的使用习惯带到 macOS 上 😁



#### Homebrew

Homebrew是一个包管理器，就相当于 `yum` 和 `apt` ，通过 Homebrew 我们可以在macOS上安装软件。它有几个优点，最重要的就是不需要超级管理员权限，比如默认的 `pip` 安装一些库的时候，每次都需要 `sudo` 



#### 安装Homebrew

```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
# 可能会提示你安装Xcode Command Line 耐心等待即可
```



#### 使用Homebrew安装

```shell
# python2
brew install python
# python3
brew install python3

# nodejs
brew install node

# jmeter 安装完毕后在 终端 直接输入 jmeter 即可
brew install jmeter

brew install redis

brew install ruby

brew install mysql

brew install maven

brew install git
# ...
```



#### 使用Homebrew安装软件

现在 Homebrew 支持安装一些常用的软件，使用 `brew cask install` 即可



##### 示例

```shell
# install java8
brew cask install java8

# 推荐使用 iterm2 替代 系统自带的terminal 
brew cask install iterm2

# chrome
brew cask install google-chrome

# sublime
brew cask install sublime
```



#### ZSH

系统默认自带的是 `bash` ，我个人比较喜欢 `zsh` ，也是 shell 的一种，但更灵活，更强大。[相关配置](https://github.com/robbyrussell/oh-my-zsh)

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```





#### 配置 shadowsocks

如果你使用 `shadowsocks` 在下载完客户端之后就可以使用了，这里只提醒两点，Git 和 终端 需要额外设置。



```shell
# terminal
export http_proxy='http://127.0.0.1:1080'    
export https_proxy='http://127.0.0.1:1080'

# git
git config --global https.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080

# or sock
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
```

