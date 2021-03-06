---
title: DJango验证和授权系统概述
date: 2017-06-28 17:40:53
tags: [DJango]
---



### 验证和授权概述：

django有一个内置授权系统，它用来处理用户、分组、权限以及基于cookie的会话系统.

django的授权系统包括验证和授权两个部分。

验证是验证这个用户是否是他声称的人呢（比如用户名和密码验证，角色验证），授权是给与他响应的权限。

Django内置的权限系统包括以下方面：

1. 用户。
2. 权限。
3. 分组。
4. 一个可以配置的密码哈希系统。
5. 一个可以插拔的后台管理系统。
   - 比较灵活，想用就用不想用可以不适用。

### 使用授权系统：

默认中创建完一个django项目后，其实就是已经集成了授权系统。

哪哪部分是跟授权系统相关的配置呢。

下面做一个简单的列表：

**INSTALLED_APPS:**

1. `django.contrib.auth`    包含一个核心授权框架，以及大部分的模型定义。
2. `django.contrib.contenttypes`   : `Content Tpye` 系统，可以用来关联模型和权限。

### 中间件：

1. `SessionMiddleware`  ：用来管理 `session`.
2. `AuthenticationMiddleware`：用来处理当前  `session` 相关联的用户。



```python
INSTALLED_APPS = [
    'django.contrib.admin',
    #包含一个核心授权框架，以及大部分的模型定义。
    'django.contrib.auth',
    #content type 系统，可以用来关联模型和权限
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',  #用来管理session
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',   #用来处理和当前session相关联的用户。
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

```



