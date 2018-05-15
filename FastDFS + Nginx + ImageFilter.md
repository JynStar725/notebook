# FastDFS + Nginx + fastdfs-nginx-module 安装配置技术手册
<br>

## 安装环境及相关软件

    系统：Ubuntu 16.04
    软件版本：FastDFS 5.11    fastdfs-nginx-module    nginx-1.14.0
    本机IP地址：31.8.130.197
    代理服务器：31.8.130.162
<br>
<br>

安装前的准备工作
================

    试验性的安装都是在虚拟机上进行，系统环境为 Ubuntu 16.04 Desktop版本。这与实际生产的系统环境不同，实际生产环境的系统是 Ubuntu16.04 server版，而且不能直连网络，必须要通过`代理`才能联网。因为是英文原版的server版本，因此自带依赖和部分配置也和虚拟机上的环境有所不同。

<b><p style="font-size: 20px; color: #d64170;">
全新的Ubuntu Server，防火墙是默认关闭的。为了防止在开启了防火墙后服务器重启或者关机，ssh连接被拒绝的情况发生，所以先要允许防火墙对外开放ssh连接端口</p></b>
```shell
在root权限下

ufw allow 22/tcp                        // 允许22端口对外开放
ufw allow from 31.8.130.200             // 允许来自31.8.130.200的所有访问
```



针对apt工具的代理设置：需配置/etc/apt/apt.conf文件
-----------------
```
vi /etc/apt/apt.conf
Acquire::http::proxy "http://31.8.130.162:3128/";
Acquire::https::proxy "https://31.8.130.162:3128/";
```

设置代理的环境变量，编辑/etc/profile，新加入以下配置，可以在`root`权限下保留代理设置
----------------
```
vi /etc/profile
export http_proxy="http://31.8.130.162:3128"
export https_proxy="http://31.8.130.162:3128"
```


设置代理（设置全局代理，暂时的，重启后失效）
----------------
```
export http_proxy=http://31.8.130.162:3128      // http协议
export https_proxy=http://31.8.130.162:3128      // https协议
```

更换apt-get源，默认是hk或者us或者其他的，要更换为cn的。如已经是cn的可以忽略
----------------
**将所有的hk换为cn**
```
root@server:~# cd /etc/apt
root@server:/etc/apt# cp source.list source.list.bak    // 先将原文件备份 
root@server:/etc/apt# vi source.list
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

开始安装FastDFS
===============
**所有安装需要的`源代码`，`tar文件`以及`tar文件解压后的文件夹`都放在 /opt 目录下**

<br>

## 1.  FastDFS需要libfastcommon依赖，所以先要安装libfastcommon，并进入libfastcommon目录

```
root@server:/opt#  git clone https://github.com/happyfish100/libfastcommon.git
root@server:/opt#  cd libfastcommon
```

<br/>

## 2.&nbsp;`./make.sh`进行编译，安装
```
root@server:/opt/libfastcommon# ./make.sh               //  编译
root@server:/opt/libfastcommon# ./make.sh install       //  安装
```

<br/>

## 3.创建符号链接
```
root@server:/opt/libfastcommon# ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so
```

<br/>

## 4.回到`/opt`目录下下载fastDFS 5.11版本
```
root@server:/opt# wget https://github.com/happyfish100/fastdfs/archive/V5.11.tar.gz
```

<br/>

## 5.解压tar文件
```
root@server:/opt# tar -zxvf V5.11.tar.gz
```

<br/>

## 6.进入解压后的fastdfs-5.11文件夹，编译，安装
```
root@server:/opt/fastdfs-5.11# ./make.sh
root@server:/opt/fastdfs-5.11# ./make.sh install
```

<br/>

## 7.创建三个配置文件`tracker.conf`、`storage.conf`、`client.conf`
```
root@server:/opt/fastdfs-5.11# cp /etc/fdfs/tracker.conf.sample  /etc/fdfs/tracker.conf
root@server:/opt/fastdfs-5.11# cp /etc/fdfs/storage.conf.sample  /etc/fdfs/storage.conf
root@server:/opt/fastdfs-5.11# cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf
```

<br/>

## 8.修改tracker.conf里的`base_path`参数，保存退出
```
root@server:/opt/fastdfs-5.11# vi /etc/fdfs/tracker.conf

# the base path to store data and log files （注释）
base_path=/usr/fastdfs-storaged
```

<br/>

## 9.修改storage.conf里的`base_path`, `store_path0`, `tracker_server`参数，保存退出
```
root@server:/opt/fastdfs-5.11# vi /etc/fdfs/storage.conf

base_path=/usr/fastdfs-storaged
store_path0=/usr/fastdfs-storaged
tracker_server=31.8.130.197:22122
```

<br/>

## 10.修改client.conf里的`base_path`，`tracker-server`参数，并将最后一行的<b><em>'##include http.conf'</em></b>改为<b><em>'#include http.conf'</em></b>
```
root@server:/opt/fastdfs-5.11#  vi /etc/fdfs/client.conf
base_path=/usr/fastdfs-storaged
tracker_server=31.8.130.197:22122
#include http.conf
```

<br/>

## 11.将fastdfs-5.11下conf目录下的`http.conf`，`mime.types`拷贝到/etc/fdfs下
```
root@server:/opt/fastdfs-5.11# cp /opt/fastdfs-5.11/conf/http.conf /etc/fdfs/
root@server:/opt/fastdfs-5.11# cp /opt/fastdfs-5.11/conf/mime.types /etc/fdfs/
```

<br/>

## 12.启动tracker进程，会显示`Starting FastDFS tracker server:`
```
root@server:/opt/fastdfs-5.11# /etc/init.d/fdfs_trackerd start
```

<br/>

## 13启动storage进程，会显示`Starting FastDFS storage server:`
```
root@server:/opt/fastdfs-5.11# /etc/init.d/fdfs_storaged start
```

<br/>

## 14.启动完成后，可以查看监听
```
root@server:/opt/fastdfs-5.11# netstat -unltp | grep fdfs
```

<br/>

**启动Storage前要确保Tracker是启动的，初次启动成功，会在/usr/fastdfs/目录下创建data,logs两个目录**<br>
**如果看到23000端口正常被监听后，这时候说明Storage服务启动成功了**
<br/>
```
root@server:/opt/fastdfs-5.11# netstat -unltp | grep fdfs
tcp        0      0 0.0.0.0:22122           0.0.0.0:*               LISTEN      4454/fdfs_trackerd
tcp        0      0 0.0.0.0:23000           0.0.0.0:*               LISTEN      4448/fdfs_storaged
```
<br/>
## 15.在服务器本地测试图片上传，使用自带的fdfs_test来测试。

```
root@server:/opt/fastdfs-5.11# fdfs_test /etc/fdfs/client.conf upload /home/suser/timg.jpg
```
<br/>
<br/>

开始安装nginx并且配置fastdfs-nginx-module 
===============
**在安装nginx之前，先要安装Nginx需要依赖的四个库文件**

<br/>

## 1.安装依赖ssl（全局安装）

```
apt-get install openssl libssl-dev
```

<br/>

## 2.安装依赖pcre（全局安装）

```
apt-get install libpcre3 libpcre3-dev
```

<br/>

## 3.安装依赖zlib1g（全局安装）

```
apt-get install zlib1g-dev
```

<br/>

## 4.安装gd库（全局安装）

```
apt-get install libgd2-dev
```

<br/>

## 5.进入`/opt`目录下，下载nginx

```
root@server:/opt# wget http://nginx.org/download/nginx-1.14.0.tar.gz
```

<br/>

## 6.解压tar文件

```
root@server:/opt# tar -zxvf nginx-1.14.0.tar.gz
```

<br/>

## 7.下载`fastdfs-nginx-module`的源代码

```
root@server:/opt# git clone https://github.com/happyfish100/fastdfs-nginx-module.git
```

<br/>

## 8.将`fastdfs-nginx-module/src`里面的`mod_fastdfs.conf`拷贝到`/etc/fdfs`下

```
root@server:/opt# cp /opt/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs
```

<br/>

## 9.编辑`mod_fastdfs.conf`文件，修改`tracker_server`，`store_path0`，`url_have_group_name`的值，保存退出

```
root@server:/opt# vi /etc/fdfs/mod_fastdfs.conf

tracker_server=31.8.130.197:22122
store_path0=/usr/fastdfs
url_have_group_name = true
```

<br/>

## 10.在`/usr`目录下创建名为'nginx'的文件夹

```
root@server:/opt# cd /usr
root@server:/usr# mkdir -p nginx
```

<br/>

## 11.回到之前解压出来的nginx源代码目录下，配置nginx

```
root@server:/usr# cd /opt/nginx-1.14.0
root@server:/opt/nginx-1.14.0# 
```

<br/>

## 12.配置nginx，--perfix代表安装的路径，就是刚才新建`/usr/nginx`文件夹，另外，把`http_image_filter_module`模块加进去

```
root@server:/opt/nginx-1.14.0# ./configure --prefix=/usr/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_realip_module --add-module=/opt/fastdfs-nginx-module/src --with-http_image_filter_module
```

<br/>

## 13.编译，安装nginx

```
root@server:/opt/nginx-1.14.0# make
root@server:/opt/nginx-1.14.0# make install
```

<br/>

## 14.运行nginx，启动成功后会打印出nginx使用的两个进程号

```
root@server:/opt/nginx-1.14.0# /usr/nginx/sbin/nginx
```

## 15.修改`/usr/nginx/conf`下的`nginx.conf`

```
root@server:/opt/nginx-1.14.0# vi /usr/nginx/conf/nginx.conf
```

<br/>

## 16.修改的`nginx.conf`内容

```
server {
        listen       80;
        server_name  31.8.130.197;

        charset utf-8;

        #access_log  logs/host.access.log  main;

        location ~/group1/M00/(.*)_([0-9]+)x([0-9]+)\.(jpg|gif|png|jpeg)$ {
            root   /usr/fastdfs/data;
            ngx_fastdfs_module;

            set $w $2;
            set $h $3;

            if ($h != "0") {
                #      rewrite /group1/M00/(.+)_(\d+)x(\d+)\.(jpg|gif|png|jpeg)$ /group1/M00/$1.$4 last;
                rewrite group1/M00(.+)_(\d+)x(\d+)\.(jpg|gif|png|jpeg)$ group1/M00$1.$4 break;
            }
            if ($w != "0") {
                rewrite /group1/M00/(.+)_(\d+)x(\d+)\.(jpg|gif|png|jpeg)$ /group1/M00/$1.$4 break;
            }
            image_filter resize $w $h;
            image_filter_buffer 5M;
        }

        location ~/group1/M00/(.+)\.?(.+){
            alias /usr/fastdfs/data;
            ngx_fastdfs_module;
        }
```

<br/>

## 17.重启nginx

```
root@server:/opt/nginx-1.14.0# pgrep nginx
root@server:/opt/nginx-1.14.0# 23245
root@server:/opt/nginx-1.14.0# 23246
root@server:/opt/nginx-1.14.0# kill -9 23245
root@server:/opt/nginx-1.14.0# kill -9 23246
root@server:/opt/nginx-1.14.0# /usr/nginx/sbin/nginx
```

<br/>

## 18.都配置成功之后，原来访问图片的的路径是

```
Sample：
http://31.8.130.197/group1/M00/00/00/wKhtglrYMNmAJ29VAAC8eeqc1ZA863.jpg
```
##  访问50*50大小的缩略图的访问路径为

```
Sample:
http://31.8.130.197/group1/M00/00/00/wKhtglrYMNmAJ29VAAC8eeqc1ZA863_50x50.jpg
```