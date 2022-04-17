---
title: 分享一个QQ第三方登录详解
date: 2018-09-15 20:45:34
tags: [第三方登陆]
---



在web开发过程中，经常会使用到第三方登录的功能，那么今天就详细的分享一下关于QQ的第三方登录。

（本篇后端框架采用的是Tornado框架）



如果阅读理解能力没有问题的情况下，可以详细阅读QQ第三方登录的官方文档：



<https://wiki.connect.qq.com/%E4%BD%BF%E7%94%A8authorization_code%E8%8E%B7%E5%8F%96access_token> 



首先说明一下，使用QQ第三方登录，必须要有审核通过的域名！！！



好了，现在该详细介绍具体步骤如何实现了：





###第一步：

​		再前端页面放一个按钮，按钮的样式官方文档上有，可以随意选择，选好按钮以后给按钮一个点击事	

​		件，向后台发起请求

###第二步：

​		后端接口收到请求后，向 https://graph.qq.com/oauth2.0/authorize 发起requests请求获取

​		Authorization Code，这里要注意一下，在请求时需要附带必要参数，在文档上都有详细介绍

​		

```Python
# 获取code url
class GetAuthorizationCode(BaseHandler):
   async def get(self, *args, **kwargs):
        url = 'https://graph.qq.com/oauth2.0/authorize'
        data = {
            'response_type':'code',
            'client_id':'101487535',
            'redirect_uri':'http://login.yskj599.com/user/login/qq_login',
            'state':'101487535'
        }
        con = requests.get(url=url,params=data)
        urls = con.url
        data = {
            # 前端页面需要跳转的url地址
            'url':urls
        }
        self.write(json.dumps(data))
```



​		请求后我们可以拿到一个url，这个url就是需要我们返回给前端进行跳转的url地址



###第三步：

​		再跳转至登录页面并登陆时，我们需要先获取到access_token和OpenID，在这里我们可以定义两个函	

​		数，qq_get_access_token、qq_get_OpenID，具体代码如下：



```Python
# QQ登录获取access_token
def qq_get_access_token(code,app_id):
    app = {
        'grant_type': 'authorization_code',
        'client_id': app_id,
        'client_secret': '9c383c0a358b3382ed52269ccff3691f',
        'code': code,
        'redirect_uri': 'http://login.yskj599.com/user/login'
    }
    access_url = 'https://graph.qq.com/oauth2.0/token'
    response = requests.get(url=access_url, params=app)
    alist = response.text.split('=')
    access_token = alist[1].split('&')[0]
    refresh_token = alist[3]
    return access_token
```

```Python
# QQ登录获取OpenID
def qq_get_OpenID(access_token):
    OpenID_url = 'https://graph.qq.com/oauth2.0/me'
    response = requests.get(OpenID_url, params={'access_token': access_token})
    OpenID = response.text.split('"')[-2]
    return OpenID
```

​		



​		接下来就非常简单了，我们只需要安装文档的需求，调用我们定义好的函数，并做好业务逻辑的判断就	

​		ok了！



```Python
#QQ登录接口
class QQLogin(BaseHandler):
   async def get(self, *args, **kwargs):
        params = self.request.arguments
        app_id = '101487535'
        try:
            code = params['code'][0].decode()
            access_token = QQ_get_access_token(code,app_id)
            OpenID = QQ_get_OpenID(access_token)
            params2 = {
                'access_token':access_token,
                'oauth_consumer_key':'101487535',
                'openid':OpenID
            }
            # QQ返回登录用户的信息
            get_info_url = 'https://graph.qq.com/user/get_user_info'
            response = requests.get(url = get_info_url,params=params2)
            user_info = response.json()
            nickname = user_info['nickname']
            # 拿到用户的nickname后，我们就可以做一些业务逻辑的判断
            user = sess.query(User).filter(User.nick == nickname).first()
            if user:
                self.redirect('http://127.0.0.1:8080?u_name={}'.format(nickname))
            else:
                qq_user = User(nick = nickname,password = '123456',gender=1,
                               addres =user_info['province'] + user_info['city'],
                               attributes = 2,is_activates = 1)
                try:
                    sess.add(qq_user)
                    sess.commit()
                	self.redirect('http://127.0.0.1:8080?u_name={}'.format(nickname))
                except:
                    pass
        except:
            self.write('登录失败')
```





好了，关于QQ的第三方登录就基本上完成了，是不是很简单呢？