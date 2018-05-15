# JDK 安装配置技术手册
<br>

## 安装环境及相关软件

    系统：Ubuntu 16.04
    软件版本：Java version "1.8.0_171"
    本机IP地址：31.8.130.197
    代理服务器：31.8.130.162
<br>
<br>

安装前的准备工作
================

    试验性的安装都是在虚拟机上进行，系统环境为 Ubuntu16.04 Desktop版本。这与实际生产的系统环境不同，实际生产环境的系统是 Ubuntu16.04 server版，而且不能直连网络，必须要通过`代理`才能联网。因为是英文原版的server版本，因此自带依赖和部分配置也和虚拟机上的环境有所不同。

针对apt工具的代理设置：需配置/etc/apt/apt.conf文件
-----------------
```
Acquire::http::proxy "http://31.8.130.162:3128/";
Acquire::https::proxy "https://31.8.130.162:3128/";
```

设置代理的环境变量，编辑/etc/profile，新加入以下配置，可以在`root`权限下保留代理设置
----------------
```
export http_proxy="http://31.8.130.162:3128"
export https_proxy="http://31.8.130.162:3128"
```


设置代理（设置全局代理，暂时的，重启后失效）
----------------
```
export http_proxy=http://31.8.130.162:3128      // http协议
export https_proxy=http://31.8.130.162:3128      // https协议
```

更换apt-get源，默认是hk的，要更换为cn的。如已经是cn的可以忽略
----------------
**将所有的hk换为cn**
```
root@server:~# cd /etc/apt
root@server:/etc/apt# cp source.list source.list.bak    // 先将原文件备份 
root@server:/etc/apt# vi source.list
```

这里开始安装JDK，需要先导出环境变量，在控制台输入(必须)
---------------
```
export http_proxy="http://31.8.130.162:3128"
export https_proxy="http://31.8.130.162:3128"
```

添加PPA的Git依赖
----------------
```
sudo -E add-apt-repository ppa:git-core/ppa
```
会得到一个提示
```
➜〜sudo -E apt-add-repository ppa：git-core / ppa
 Ubuntu的Git的最新稳定版本。

对于发布候选版本，请转到https://launchpad.net/~git-core/+archive/candidate。
 更多信息：https：//launchpad.net/~git-core/+archive/ubuntu/ppa
按[ENTER]继续或ctrl-c取消添加
```
按下`Enter`，就可以从PPA安装JDK了

首先添加PPA
-----------------
```
sudo add-apt-repository ppa:webupd8team/java
```

然后更新下系统，刷新软件源
----------------
```
apt-get update
```

最后开始安装
---------------
```
apt-get install oracle-java8-installer
```

安装完成后，查看JDK版本
---------------
```
java -version
```

升级apt-get
----------------
```
apt-get update
```

在之后的安装过程中发现，gcc可能版本低，可能缺少g++和make指令，所以安装这两个依赖库
----------------
```
apt-get install gcc     // 安装gcc，已有的忽略
apt-get install g++     // 安装g++，已有的忽略
apt-get install make    // 安装make指令，已有的忽略
```

安装ssh
--------------
```
apt-get install openssh-server
```
安装git
--------------
```
apt-get install git
```
安装Vim
--------------
```
apt-get install vim
```
<br/>
以上差不多完成了准备工作。
<br/>
<br/>
<br/>