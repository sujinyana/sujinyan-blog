---
title: FastdFS特性以及工作原理
date: 2017-05-18 10:45:12
tags: [FastDFS]
---



# FastDFS简介 

### 1.1、FastDFS简介

FastdFS是一个轻量级的开源分布式文件系统，主要解决了大容量的文件存储和高并发访问问题，文件存取时实现了负载均衡。纯C语言实现，支持Linux、FreeBSD、AIX等UNIX系统。它只能通过专有API对文件进行存取访问，不支持POSIX接口方式，不能mount使用。

### 1.2、FastDFS的特性 

（1）分组存储、灵活简洁、对等结构、不存在单点 
（2）文件ID由FastDFS生成，作为文件访问凭证，FastDFS不需要传统的name server 
（3）和流行的web server无缝衔接，FastDFS已提供Apache和nginx扩展模块 
（4）大、中、小文件均可以很好支持，支持海量小文件存储 
（5）支持多块磁盘，支持单盘数据恢复 
（6）支持相同文件内容只保存一份，节省存储空间 
（7）存储服务器上可以保存文件附加属性 
（8）下载文件支持多线程方式，支持断点续传 

### 1.3、FastDFS功能简介 

系统简洁:只有两个角色tracker和storage 
系统性能高：没有使用数据库，文件同步直接点对点，不经过tracker中转 
系统稳定性高：C语言开发，可以支持高并发和高负载 
RAID方式：分组（组内冗余），灵活性较大 
通信协议：专有协议，下载文件支持HTTP 
支持：文件附加属性、相同内容文件只保存一份、下载文件时支持文件偏移量 

# 二、FastDFS工作原理 

### 2.1、FastDFS架构解读： 

只有两个角色，tracker server和storage server，不需要存储文件索引信息所有服务器都是对等的，不存在Master-Slave关系 
存储服务器采用分组方式，同组内存储服务器上的文件完全相同（RAID1）不同组的storage server之间不会相互通信 
由storage server定时主动向tracker server报告状态信息，tracker server之间通常不会相互通信。 

### 2.2、文件传输机制 

FastDFS系统中客户端上传文件流程： 
（1）client询问tracker上传到的storage 
（2）tracker返回一台可用的storage 
（3）client直接和storage通信完成文件上传，storage返回文件ID给客户端。

FastDFS系统中客户端下载文件流程： 
（1）client询问tracker下载文件的storage，参数为文件组名和文件名 
（2）tracker去storage查询访问的数据资源位置，将存储该资源的storage信息及文件位置信息返回给client 
（3）client直接和storage通信完成文件下载 

### 2.3、FastDFS的同步机制: 

采用binlog文件记录更新操作，根据binlog进行文件同步，同一组内的storage server之间是对等的，文件上传、删除等操作可以在任意一台storage server上进行，更改的数据由发生更改的storage server（源服务器）采用push方式，向同组内的其他storage server（目标服务器）进行同步 
源数据（发生更改的数据）才需要同步，备份数据（原有的同步数据）不需要再次同步，否则就构成了环路了。 
对于新增加的一台storage server时，由已有的一台storage server将已有的所有的数据（包括新增的数据和备份数据）同步到新增的服务器。 

### 2.4、FastDFS的两大核心组件 

Tracker：调度器 
负责维持集群的信息，例如各group及其内部的storage node，这些信息也是storage node报告所生成的，每个storage node会周期性向tracker发心跳信息，让tracker知道自己还正常运行

Storage server： 
以group为单位进行组织，任何一个storage server都应该属于某个group，一个group应该包含多个storage server，在同一个group内，各storage server的数据相互冗余。在同一组内storage节点之间的数据是相同的。 

### 2.5、FastDFS查询存储机制 

当客户端下载数据的时候，如何在组中挑选storage server： 
（1）rr #轮循 
（2）以ip为次序，找第一个，即IP地址较小者 
（3）以优先级为序，找第一个 
如何选择磁盘（存储路径） 
（1）rr #轮循 
（2）剩余可用空间大者优先 

### 2.6、FastDFS常用命令介绍 

FastDFS常用命令： 
（1）查看存储节点状态 
fdfs_monitor /etc/fdfs/client.conf 
（2）上传测试 
fdfs_test upload [FILE | BUFF | CALLBACK] 
（3）文件上传 
fdfs_upload_file /etc/fdfs/client.conf /root/solo-2.2.0.war 
（4）文件查看 
fdfs_file_info /etc/fdfs/client.conf group1/M00/00/00/rBH7vFoax3KANb_FAUlr7-L-yRM9.0.war 
（5）文件下载 
fdfs_download_file /etc/fdfs/client.conf group1/M00/00/00/rBH7vFoax3KANb_FAUlr7-L-yRM9.0.war 

# 三、 Nginx + FastDFS分布式存储实现过程 

### 3.1、实验环境准备： 

关闭防火墙、关闭selinux 
准备三台centos7 
准本相关安装包： 
下载地址： <https://pan.baidu.com/s/1c1L19iK> 密码：v5nd 
fastdfs相关的包： 
fastdfs-5.0.11-1.el7.centos.x86_64.rpm 
fastdfs-debuginfo-5.0.11-1.el7.centos.x86_64.rpm 
fastdfs-server-5.0.11-1.el7.centos.x86_64.rpm 
fastdfs-tool-5.0.11-1.el7.centos.x86_64.rpm 
安装所需要的依赖库包： 
libfastcommon-1.0.36-1.el7.centos.x86_64.rpm 
libfastcommon-devel-1.0.36-1.el7.centos.x86_64.rpm 
libfdfsclient-5.0.11-1.el7.centos.x86_64.rpm 
libfdfsclient-devel-5.0.11-1.el7.centos.x86_64.rpm

以下是所需要的nginx和nginx模块：（nginx可以换其他版本，同时对应的模块也换对应的版本） 
nginx-1.10.2-1.el7.centos.x86_64.rpm 
nginx-all-modules-1.10.2-1.el7.centos.noarch.rpm 
nginx-filesystem-1.10.2-1.el7.centos.noarch.rpm 
nginx-mod-http-geoip-1.10.2-1.el7.centos.x86_64.rpm 
nginx-mod-http-image-filter-1.10.2-1.el7.centos.x86_64.rpm 
nginx-mod-http-perl-1.10.2-1.el7.centos.x86_64.rpm 
nginx-mod-http-xslt-filter-1.10.2-1.el7.centos.x86_64.rpm 
nginx-mod-mail-1.10.2-1.el7.centos.x86_64.rpm 
nginx-mod-stream-1.10.2-1.el7.centos.x86_64.rpm 
（注意：！这些都是对应的centos7版本的安装包，不适用于centos6，所以想在centos6上尝试的可以去找对应centos6的包。）

### 3.2、划分主机角色 

示例：

```
 tracker节点：node1 192.168.11.28
storage-1节点：node3 192.168.11.107
storage-2节点：jiake-node1 192.168.11.17
123
```

安装服务： 
fastdfs-server包中包含了tracker节点的服务和storage节点的服务，两个节点的服务集成到了一个包中，所以两个节点上都要安装这个包。

### 3.3、tracker节点配置 

下载上述所列出的安装包，进入安装包目录，执行安装：

```
yum localinstall fastdfs* libf* nginx* -y ```
1
```

（如果安装过nginx，且nginx中已经带有ngx_fastdfs_module模块，就不需要安装上述的nginx包了）

由于我们也同时安装了客户端的管理工具，所以可以在tracker节点是上直接管理各个节点 
创建tracker节点的工作目录：

mkdir -p /data/fastdfs/tracker“` 
(如果运行服务的用户不是root，需要改该目录的所属人) 
配置fastdfs配置文件： 
安装好后默认在/etc/fdfs/目录下有模板配置文件， 
cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf 
cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf

配置tracker.conf 
vim /etc/fdfs/tracker.conf 
disabled=false #（默认为false，表示是否有效) 
bind_addr= #端口绑定的监听地址，不写默认监听所有 
port=22122 #监听的端口 
base_path=/data/fastdfs/tracker #tracker节点的工作目录，存储数据和日志文件 
配置client.conf 
vim /etc/fdfs/client.conf 
base_path=/data/fastdfs/tracker #指定目录存放日志文件 
tracker_server=192.168.11.28:22122 #指定tracker节点地址和端口，其他的配置都默认即可。 
配置nginx.conf文件 
vim /etc/nginx/nginx.conf 
在Server中添加： 
location /group1/M00 { 
root /data/fastdfs/storage; #storage节点数据存储路径 
ngx_fastdfs_module; #nginx中fastdfs的模块 
} 
配置mod_fastdfs.conf 
vim /etc/fdfs/mod_fastdfs.conf 
tracker_server=192.168.11.28:22122 #指定tracker节点的地址和端口 
storage_server_port=23000 #指明storage节点的连接端口 
url_have_group_name = true #开启，访问路径中带有组名 
store_path0=/data/fastdfs/storage #storage节点上数据存储路径 
\3. 4、storage节点配置 
两个节点相同，所以配置相同

同样下载安装包，进行安装

yum localinstall fastdfs* libf* nginx* -y“` 
创建存放数据的目录

mkdir -p /data/fastdfs/storage“` 
(如果运行服务的用户不是root，需要改该目录的所属人) 
配置storage.conf文件

cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf

vim /etc/fdfs/storage.conf 
disabled=false

group_name=group1 #可以指定该节点的所属组

port=23000 #监听端口 
base_path=/data/fastdfs/storage #服务工作目录，存放服务数据和日志文件 
store_path0=/data/fastdfs/storage #数据存储目录，存放来自客户端的数据 
tracker_server=192.168.11.28:22122 #tracker节点地址和端口 
allow_hosts=* #允许访问的数据，*默认所有，可以指定具体网段或具体地址 
配置nginx.conf 
vim /etc/nginx/nginx.conf 
在Server端： 
location /group1/M00 { 
root /data/fastdfs/storage; 
ngx_fastdfs_module; 
} 
配置mod_fastdfs.conf 
vim /etc/fdfs/mod_fastdfs.conf 
tracker_server=192.168.11.28:22122 #指定tracker节点的地址和端口 
storage_server_port=23000 #指明storage节点的连接端口 
url_have_group_name = true #开启，访问路径中带有组名 
store_path0=/data/fastdfs/storage #storage节点上数据存储路径 
\3. 5、启动服务 
tracker节点：/etc/init.d/fdfs_trackerd start 
systemctl start nginx` ![](http://i2.51cto.com/images/blog/201712/11/d9a9c20e8b15675f7149d4b3f4c23bc5.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=) storage-1节点：/etc/init.d/fdfs_storaged start systemctl start nginx` 
FastDFS实现分布式存储

storage-2节点：/etc/init.d/fdfs_storaged start 
systemctl start nginx“` 
![img](http://i2.51cto.com/images/blog/201712/11/038ccc38d2f6a32ca45cb0b161b0c03b.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

## 3.6、查看节点状态

> 我们把管理工具安装到了tracker节点，所以在tracker节点上进行上传操作，如果管理节点在其他的主机上也可以，不过一定要将/etc/fdfs/client.conf配置文件中的tracker指定对，指定tracker节点的地址和监听端口。 
> fdfs_monitor /etc/fdfs/client.conf“` 
> FastDFS实现分布式存储

### 3.7、上传文件 

fdfs_upload_file /etc/fdfs/client.conf /root/2.jpg 
（格式：命令 + client.conf + file_path） 
会自动返回存储文件的位置信息“` 
![img](http://i2.51cto.com/images/blog/201712/11/728ae635f2f09d204de303052e1502c9.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

### 3.8、查看文件信息

fdfs_file_info /etc/fdfs/client.conf group1/M00/00/00/wKgLEVofqfeAE7PeAAD0h4ofB2Y347.jpg 
（格式：命令 + client.conf + 文件组及信息）“` 
FastDFS实现分布式存储

### 3.9、通过浏览器访问该文件 

地址指向tracker节点 
<http://192.168.11.11/roup1/M00/00/00/wKgLEVofqfeAE7PeAAD0h4ofB2Y347.jpg>

FastDFS实现分布式存储

# 四、 错误总结及注意事项 

### 1、 错误总结： 

启动的时候显示启动失败的几种原因： 
（1） 存储数据的目录没有创建 
（2） 配置文件中tracker_server的地址没有配置，这个地址是必须配置的，尤其注意在tracker.conf、storage.conf、mod_fastdfs.conf这些配置文件中的tracker服务的地址一定要指定对 
（3） 如果之前该进程没有正常关闭，也会导致该进程再次启动其不来，即使端口没有开，可能后台的进程还在运行，所以，当启动的时候显示启动ok，但是端口一直打不开，可以向将进程关闭：/etc/init.d/fdfs_storaged stop 再进行开启：/etc/init.d/fdfs_storaged start 

### 2、 注意事项： 

(1） tracker节点和storage节点上存放数据的目录都要创建，并对应配置文件中指定的目录。 
（2） 注意配置文件中的工作目录和数据存储目录不一样，虽然在写的时候两个目录相同，但是在实际的区分中两个目录的角色是不一样的。 
（3） 如果有什么错误，可以查看数据目录下的logs目录中的access.log日志文件 
（4） 注意不指定运行用户，默认的是root用户，如果要更改运行用户，一定要注意目录的权限，将数据目录的权限更改为运行用户所能够访问。