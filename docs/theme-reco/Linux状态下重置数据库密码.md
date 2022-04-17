---
title: Linux状态下重置数据库密码
date: 2018-07-15 15:20:52
tags: [Linux]
---



# Linux-Mysql 数据库重置密码步骤

第一步：cat /var/log/mysqld.log |grep "password"

第二步：set password = password("123456");

第三步：set global validate_password_policy=0;

第四步：set global validate_password_length=1;

第五步：alter user 'root'@'localhost' password expire never;

第六步：flush privileges;

第七步：exit;

 

拓展：

第一步：pip install flask-sqlalchemy

第二步：yum -y install mysql-devel gcc gcc-devel python-devel

第三步：pip install mysqlclient

