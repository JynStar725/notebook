# CentOS 7 下FastDFS + Nginx + fastdfs-nginx-module 安装配置技术手册
<br>

## 安装环境及相关软件

    系统：CentOS 7
    软件版本：FastDFS 5.11    fastdfs-nginx-module    nginx-1.14.0
    服务器IP：31.8.119.101
    代理服务器：31.8.130.162
<br>
<br>
1.关闭filewall防火墙，启用iptable防火墙

```shell{.line-numbers}
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl mask firewalld.service
```
<br>
2.安装iptables防火墙

```shell{.line-numbers}
yum install iptables-services -y
```

<br>
3.启动设置防火墙

```shell{.line-numbers}
systemctl enable iptables
systemctl start iptables
```

<br>
4.查看防火墙状态

```shell{.line-numbers}
systemctl status iptables
```


<br>
5.编辑防火墙，增加开放端口

```shell{.line-numbers}
vi /etc/sysconfig/iptables #编辑防火墙配置文件
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
:wq! #保存退出
```
<br>
6.重启配置，重启系统

```shell{.line-numbers}
systemctl restart iptables.service #重启防火墙使配置生效
systemctl enable iptables.service #设置防火墙开机启动
```
<br>
7.查看端口开放情况

```shell{.line-numbers}
iptables -L -n --line-number 
```
<br>
8.安装相关环境和工具

```shell{.line-numbers}
yum install -y wget vim git gcc gcc-c++
```
<br>
9.设置有关代理

```shell{.line-numbers}
	==============================================================
	全局代理proxy
		vim /etc/profile
	在最后添加
		http_proxy=http://31.8.130.162:3128
		https_proxy=https://31.8.130.162:3128
		export http_proxy
		export https_proxy

	设置环境变量
		source /etc/profile
	==============================================================
	yum proxy(已经存在)
		vim /etc/yum.conf
		
	添加
		proxy=http://31.8.130.162:3128
		
	环境变量
		source /etc/profile
	==============================================================
	wget proxy
	vim /etc/wgetrc

	添加：
	http_proxy=http://31.8.130.162:3128
	https_proxy=http://31.8.130.162:3128

	环境变量：
	source /etc/profile
	==============================================================
```
<br/>

开始安装FastDFS
===============
**所有安装需要的`源代码`，`tar文件`以及`tar文件解压后的文件夹`都放在 /opt 目录下**

<br>

## 1.FastDFS需要libfastcommon依赖，所以先要安装libfastcommon，并进入libfastcommon目录

```shell{.line-numbers}
root@server:/opt#  git clone https://github.com/happyfish100/libfastcommon.git
root@server:/opt#  cd libfastcommon
```

<br/>

## 2.&nbsp;`./make.sh`进行编译，安装
```shell{.line-numbers}
root@server:/opt/libfastcommon# ./make.sh && ./make.sh install
```

<br/>

## 3.创建符号链接，这个时候我们就要建立一个软链接了，实际上也相当于windows上的快捷方式。
```shell{.line-numbers}
root@server:/opt/libfastcommon# ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so
root@server:/opt/libfastcommon# ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so
root@server:/opt/libfastcommon# ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so
root@server:/opt/libfastcommon# ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so
```

<br/>

## 4.回到`/opt`目录下下载fastDFS 5.11版本
```shell{.line-numbers}
root@server:/opt# wget https://github.com/happyfish100/fastdfs/archive/V5.11.tar.gz
```

<br/>

## 5.解压tar文件
```shell{.line-numbers}
root@server:/opt# tar -zxvf V5.11.tar.gz
```

<br/>

## 6.进入解压后的fastdfs-5.11文件夹，编译，安装
```shell{.line-numbers}
root@server:/opt/fastdfs-5.11# ./make.sh && ./make.sh install
```

<br/>

## 7.创建三个配置文件`tracker.conf`、`storage.conf`、`client.conf`
```shell{.line-numbers}
root@server:/opt/fastdfs-5.11# cp /etc/fdfs/tracker.conf.sample  /etc/fdfs/tracker.conf
root@server:/opt/fastdfs-5.11# cp /etc/fdfs/storage.conf.sample  /etc/fdfs/storage.conf
root@server:/opt/fastdfs-5.11# cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf
```

<br/>

**安装tracker**

先创建tracker的工作目录，目录可以自定义，用来保存tracker的data和log
```shell{.line-numbers}
[root@localhost usr]# mkdir -p fastdfs_storaged
```
**配置tracker**
```shell{.line-numbers}
cd /etc/fdfs
vim tracker.conf
```
## 8.修改tracker.conf里的`base_path`参数，保存退出
```shell{.line-numbers}
base_path=/usr/fastdfs_storaged
```
<br>

**启动tracker**
```shell{.line-numbers}
保存配置后启动tracker，命令如下：
service fdfs_trackerd start

如果不能启动，或提示用systemctl可改用命令：
systemctl start fdfs_trackerd

成功后应该可以看到：
[root@localhost fdfs]# service fdfs_trackerd start
Starting fdfs_trackerd (via systemctl):                    [  OK  ]

进行刚刚创建的tracker目录，发现目录中多了data和log两个目录
```
<br>

**最后我们需要给tracker加入开机启动**
```shell{.line-numbers}
ll /etc/rc.d/rc.local

如果没有权限
chmod +x /etc/rc.d/rc.local

修改rc.local
vim /etc/rc.d/rc.local
```
```shell{.line-numbers}
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.

touch /var/lock/subsys/local
service fdfs_trackerd start
```
```
查看端口监听情况：netstat -unltp|grep fdfs
tcp        0      0 0.0.0.0:22122           0.0.0.0:*               LISTEN      2231/fdfs_trackerd
```
端口22122成功监听
<br/>

**安装Storage**
## 9.修改storage.conf里的`base_path`, `store_path0`, `tracker_server`参数，保存退出
```shell{.line-numbers}
root@server:/opt/fastdfs-5.11# vi /etc/fdfs/storage.conf

base_path=/usr/fastdfs_storaged
store_path0=/usr/fastdfs_storaged
tracker_server=31.8.130.197:22122
```

```shell{.line-numbers}
启动storage
service fdfs_storaged start

如果不能启动，或提示用systemctl可改用命令：
systemctl start fdfs_storaged

成功后应该可以看到：
[root@localhost fdfs]# service fdfs_stroaged start
Starting fdfs_storaged (via systemctl):                    [  OK  ]

同样的，设置开机启动： 
修改rc.local
vim /etc/rc.d/rc.local
```
```shell{.line-numbers}
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.

touch /var/lock/subsys/local
service fdfs_trackerd start
service fdfs_storaged start
```

```
查看一下服务是否启动：
[root@localhost fastdfs]# netstat -unltp | grep fdfs
tcp        0      0 0.0.0.0:22122           0.0.0.0:*               LISTEN      2231/fdfs_trackerd
tcp        0      0 0.0.0.0:23000           0.0.0.0:*               LISTEN      2323/fdfs_storaged
```
服务已正常启动
<br/>

**校验整合**

到这里，fastdfs的东西都已安装完成，最后我们还要确定一下，storage是否注册到了tracker中去。
查看命令：
/usr/bin/fdfs_monitor /etc/fdfs/storage.conf

成功后可以看到： 
ip_addr = 31.8.119.101  ACTIVE (localhost.localdomain)

```shell{.line-numbers}
[2018-05-31 09:11:01] DEBUG - base_path=/usr/fastdfs_storaged, connect_timeout=30, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=0, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0

server_count=1, server_index=0

tracker server is 31.8.119.101:22122

group count: 1

Group 1:
group name = group1
disk total space = 51175 MB
disk free space = 46326 MB
trunk free space = 0 MB
storage server count = 1
active server count = 1
storage server port = 23000
storage HTTP port = 8888
store path count = 1
subdir count per path = 256
current write server index = 0
current trunk file id = 0

        Storage 1:
                id = 31.8.119.101
                ip_addr = 31.8.119.101  ACTIVE
                http domain = 
                version = 5.11
                join time = 2018-05-31 05:58:33
                up time = 2018-05-31 05:58:33
                total storage = 51175 MB
                free storage = 46326 MB
                upload priority = 10
                store_path_count = 1
                subdir_count_per_path = 256
                storage_port = 23000
                storage_http_port = 8888
                current_write_path = 0
                source storage id = 
                if_trunk_server = 0
                connection.alloc_count = 256
                connection.current_count = 0
                connection.max_count = 2
                total_upload_count = 2
                success_upload_count = 2
                total_append_count = 0
                success_append_count = 0
                total_modify_count = 0
                success_modify_count = 0
                total_truncate_count = 0
                success_truncate_count = 0
                total_set_meta_count = 2
                success_set_meta_count = 2
                total_delete_count = 0
                success_delete_count = 0
                total_download_count = 0
                success_download_count = 0
                total_get_meta_count = 0
                success_get_meta_count = 0
                total_create_link_count = 0
                success_create_link_count = 0
                total_delete_link_count = 0
                success_delete_link_count = 0
                total_upload_bytes = 248628
                success_upload_bytes = 248628
                total_append_bytes = 0
                success_append_bytes = 0
                total_modify_bytes = 0
                success_modify_bytes = 0
                stotal_download_bytes = 0
                success_download_bytes = 0
                total_sync_in_bytes = 0
                success_sync_in_bytes = 0
                total_sync_out_bytes = 0
                success_sync_out_bytes = 0
                total_file_open_count = 2
                success_file_open_count = 2
                total_file_read_count = 0
                success_file_read_count = 0
                total_file_write_count = 2
                success_file_write_count = 2
                last_heart_beat_time = 2018-05-31 09:10:34
                last_source_update = 2018-05-31 06:10:06
                last_sync_update = 1969-12-31 19:00:00
                last_synced_timestamp = 1969-12-31 19:00:00 
```
<br>

**配置客户端**

同样的，需要修改客户端的配置文件：
```
vim /etc/fdfs/client.conf
```

## 10.修改client.conf里的`base_path`，`tracker-server`参数，并将最后一行的<b><em>'##include http.conf'</em></b>改为<b><em>'#include http.conf'</em></b>
```shell{.line-numbers}
base_path=/usr/fastdfs_storaged
tracker_server=31.8.119.101:22122
#include http.conf
```

<br/>

## 11.将fastdfs-5.11下conf目录下的`http.conf`，`mime.types`拷贝到/etc/fdfs下
```shell{.line-numbers}
root@server:/opt/fastdfs-5.11# cp /opt/fastdfs-5.11/conf/http.conf /etc/fdfs/
root@server:/opt/fastdfs-5.11# cp /opt/fastdfs-5.11/conf/mime.types /etc/fdfs/
```

<br/>

## 12.在服务器本地测试图片上传，使用自带的fdfs_test来测试。

```shell{.line-numbers}
root@server:/opt/fastdfs-5.11# fdfs_test /etc/fdfs/client.conf upload /home/suser/timg.jpg
```
<br/>
## 13.如果需要清空storage，做法如下，保持tracked开启，关闭storaged
```shell{.line-numbers}
/etc/init.d/fdfs_storaged stop
```
<br/>
## 14.使用fdfs_monitor删除group关联
```shell{.line-numbers}
fdfs_monitor /etc/fdfs/storage.conf delete group1 31.8.130.197
```
<br/>
## 15.删除原来fastdfs下的data和logs
```shell{.line-numbers}
root@ubuntu-197:/usr/fastdfs# rm -r data logs
```
<br/>
## 16.重启storaged，会自动再生成group
/etc/init.d/fdfs_storaged start
<br/>
<br/>

开始安装nginx并且配置fastdfs-nginx-module 
===============
**在安装nginx之前，先要安装Nginx需要依赖的四个库文件**

<br/>
## 1.下载安装nginx依赖
```shell{.line-numbers}
yum install -y pcre pcre-devel zlib zlib-devel openssl openssl-devel gd-devel zlib1g-devel
```

## 2.进入`/opt`目录下，下载nginx

```shell{.line-numbers}
root@server:/opt# wget http://nginx.org/download/nginx-1.14.0.tar.gz
```

<br/>

## 3.解压tar文件

```shell{.line-numbers}
root@server:/opt# tar -zxvf nginx-1.14.0.tar.gz
```

<br/>

## 4.下载`fastdfs-nginx-module`的源代码

```shell{.line-numbers}
root@server:/opt# git clone https://github.com/happyfish100/fastdfs-nginx-module.git
```

<br/>

## 5.将`fastdfs-nginx-module/src`里面的`mod_fastdfs.conf`拷贝到`/etc/fdfs`下

```shell{.line-numbers}
root@server:/opt# cp /opt/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs
```

<br/>

## 6.编辑`mod_fastdfs.conf`文件，修改`tracker_server`，`store_path0`，`url_have_group_name`的值，保存退出

```shell{.line-numbers}
root@server:/opt# vi /etc/fdfs/mod_fastdfs.conf

tracker_server=31.8.130.197:22122
store_path0=/usr/fastdfs_storaged
url_have_group_name = true
```

<br/>

## 7.在`/usr`目录下创建名为'nginx'的文件夹

```shell{.line-numbers}
root@server:/opt# cd /usr
root@server:/usr# mkdir -p nginx
```

<br/>

## 8.回到之前解压出来的nginx源代码目录下，配置nginx

```shell{.line-numbers}
root@server:/usr# cd /opt/nginx-1.14.0
root@server:/opt/nginx-1.14.0# 
```

<br/>

## 9.配置nginx，--perfix代表安装的路径，就是刚才新建`/usr/nginx`文件夹，另外，把`http_image_filter_module`模块加进去

```shell{.line-numbers}
root@server:/opt/nginx-1.14.0# ./configure --prefix=/usr/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_realip_module --add-module=/opt/fastdfs-nginx-module/src --with-http_image_filter_module
```

<br/>

## 10.编译，安装nginx

```shell{.line-numbers}
root@server:/opt/nginx-1.14.0# make
root@server:/opt/nginx-1.14.0# make install
```

<br/>

## 11.运行nginx，启动成功后会打印出nginx使用的两个进程号

```shell{.line-numbers}
root@server:/opt/nginx-1.14.0# /usr/nginx/sbin/nginx
```

## 12.修改`/usr/nginx/conf`下的`nginx.conf`

```
root@server:/opt/nginx-1.14.0# vi /usr/nginx/conf/nginx.conf
```

<br/>

## 13.修改的`nginx.conf`内容

```shell{.line-numbers}
server {
        listen       80;
        server_name  31.8.119.101;

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
            image_filter_buffer 20M;
            client_max_body_size 1000m;
        }

        location ~/group1/M00/(.+)\.?(.+){
            alias /usr/fastdfs/data;
            ngx_fastdfs_module;
        }
```

<br/>

## 14.重启nginx

```shell{.line-numbers}
root@server:/opt/nginx-1.14.0# pgrep nginx
root@server:/opt/nginx-1.14.0# 23245
root@server:/opt/nginx-1.14.0# 23246
root@server:/opt/nginx-1.14.0# kill -9 23245
root@server:/opt/nginx-1.14.0# kill -9 23246
root@server:/opt/nginx-1.14.0# /usr/nginx/sbin/nginx
```

<br/>

## 15.都配置成功之后，原来访问图片的的路径是

```shell{.line-numbers}
Sample：
http://31.8.130.197/group1/M00/00/00/wKhtglrYMNmAJ29VAAC8eeqc1ZA863.jpg
```
##  访问50*50大小的缩略图的访问路径为

```shell{.line-numbers}
Sample:
http://31.8.130.197/group1/M00/00/00/wKhtglrYMNmAJ29VAAC8eeqc1ZA863_50x50.jpg
```
