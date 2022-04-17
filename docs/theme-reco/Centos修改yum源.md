---
title: Centos修改yum源
date: 2017-08-20 18:20:31
tags: [Centos,yum]
---



## 修改yum源为阿里（耗时网）

1.备份本地yum源
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo_bak 
2.获取阿里yum源配置文件
将https://mirrors.tuna.tsinghua.edu.cn/help/centos/ 中的内容粘贴到CentOS-Base.repo
3.更新cache
yum clean all
yum makecache 
4.查看
yum -y update 
修改yum源为阿里
备份本地yum源
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo_bak 
2.获取阿里yum源配置文件
将https://mirrors.tuna.tsinghua.edu.cn/help/centos/ 中的内容粘贴到CentOS-Base.repo
3.更新cache
yum clean all
yum makecache 
4.查看
yum -y update 

