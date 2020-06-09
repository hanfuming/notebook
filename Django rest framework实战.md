### Django rest framework实战

#### 内容回顾

1. 开发模式

   - 普通开发方式（前后端放在一起写）
   - 前后端分离

2. 后端开发

       - 为前端提供URL（API/接口的开发）
       - 永远返回HttpResponse

3. Django  FBV  CBV

   - FBV  方法视图

     ```python
     def users(request):
         user_list = ['a','b']
         return HttpResponse(json.dumps((user_list)))
     ```

     

   - CBV 类视图

     基于反射实现根据请求方式不同，执行不同的方法。

     原理: url -> view方法 -> dispatch方法（GET/POST/PUT/DELETE）

     ```python
     #路由
         path('students', views.students.as_view(), name='students'),
     #视图
     from django.views import View
     class StudentsView(View):
         def get(self,request,*args,**kwargs):
             return HttpResponse('GET')
         def post(self,request,*args,**kwargs):
             return HttpResponse('POST')
         def put(self,request,*args,**kwargs):
             return HttpResponse('PUT')
     	def delete(self,request,*args,**kwargs):
             return HttpResponse('DELETE')
     ```

4. 列表生成式

    ```python
   class Foo:
       pass
   class Bar:
       pass
   v = []
   for i in [Foo,bar]:
       obj = i()
       v.append(obj)
    v = [item() for item in [Foo, Bar]]
   #v对象列表
    ```

5. 面向对象

   -  封装

        - 对同一类方法进行封装

          ```python
          class File():
              # 文件增删改查
          ```

        - 将数据封装到对象中

          ```python
          class File:
              def __init__(self,a1,a2)
              self.a1 = a1
              self.a2 = a2
          obj = File(123,456)   
          ```

6. 补充

   - django中间件里面的方法

         -  process_request
         -  process_view
         -  process_response
         -  process_exception
         -  process_render_template

   - 使用中间件做过什么？

        - 权限

        - 用户登录验证

        - django的csrf是如何实现的？

          ```python
          from django.views.decrators.csrf import csrf_exempt,csrf_protect
          from django.utils.decorators import method_decorator
          #CBV
          @csrf_exempt #该函数无需认证
          @csrf_protect #该函数需要认证
          #FBV
          @method_decorator(csrf_exempt,name='despatch')
          ```

7. 总结：

   - 本质：基于反射来实现
   - 流程：路由，view，dispatch(反射)
   - 取消csrf认证（装饰器要加到dispatch方法上，且用method_decorator装饰）

#### 内容概要

##### restful 规范 

1. 根据method不同做不同的操作，（get/post/put/delete)

2. API与用户的通信协议，总是使用HTTPS协议

3. 域名规范：

      - 子域名方式（会存在跨域问题）

        ```
        www.baidu.com
        api.baidu.com
        ```

   - URL方式：

     ```
     www.baidu.com
     www.baidu.com/api/
     ```

4. 版本

   -  URL,如：https://api.example.com/v1/
   - 请求头添加版本

5. 路径，视网络上任何东西都是资源，均使用名词表示

6. method (GET,POST,PUT,PATCH,DELETE)

7. 过滤，通过在url上传参数的形式传递搜索条件

   https://api.example.com/v1/zoos?limit=10

8. 状态码

9. 错误处理

10. 返回结果

#### django rest framework框架

pip3 install djangorestframework

##### 认证

```python
class MyAuthentication(object):
    def autherticate(self,request):
        token = request._request.GET.get('token')
        if token:
            return
        else:
            raise NotAuthenticated("没有登入")

class myView(APIView):
    authentication_classes = [MyAuthentication]
```

```python
class UserInfo(models.Model):
    user_type_choices = ((1,'普通用户'),(2,'VIP'),(3,'SVIP'))
    user_type = models.IntegerField(choices=user_type_choices)
    username = models.CharFiled(max_length=32)
    password = models.CharFiled(max_length=64)
 
class UserToken(models.Model):
    user = models.OneToOneField(to='UserInfo')
    token = models.CharFiled(max_length=64)
```

```python
from django.http import HttpResponse, JsonResponse
from rest_framework.views import APIView
from user import models
import time
import base64
import hmac


def get_token(key, expire=3600):
    '''
    :param key: str (用户给定的key，需要用户保存以便之后验证token,每次产生token时的key 都可以是同一个key)
    :param expire: int(最大有效时间，单位为s)
    :return: token
    '''
    ts_str = str(time.time() + expire)
    ts_byte = ts_str.encode("utf-8")
    sha1_tshexstr = hmac.new(key.encode("utf-8"), ts_byte, 'sha1').hexdigest()
    token = ts_str+':'+sha1_tshexstr
    b64_token = base64.urlsafe_b64encode(token.encode("utf-8"))
    return b64_token.decode("utf-8")


def out_token(key, token):
    '''
    :param key: 服务器给的固定key
    :param token: 前端传过来的token
    :return: true,false
    '''

    # token是前端传过来的token字符串
    try:
        token_str = base64.urlsafe_b64decode(token).decode('utf-8')
        token_list = token_str.split(':')
        if len(token_list) != 2:
            return False
        ts_str = token_list[0]
        if float(ts_str) < time.time():
            # token expired
            return False
        known_sha1_tsstr = token_list[1]
        sha1 = hmac.new(key.encode("utf-8"),ts_str.encode('utf-8'),'sha1')
        calc_sha1_tsstr = sha1.hexdigest()
        if calc_sha1_tsstr != known_sha1_tsstr:
            # token certification failed
            return False
        # token certification success
        return True
    except Exception as e:
        print(e)
        
from rest_framework.authentication import BaseAuthentication
from user import models
from rest_framework.exceptions import NotAuthenticated       
class TokenAuth2(BaseAuthentication):
    def authenticate(self,request):
        token=request.GET.get("token")
        name=request.GET.get("name")
        token_obj=out_token(name,token)
        if token_obj:
            return
        else:
            raise NotAuthenticated("你没有登入")

class AuthLogin(APIView):
    '''登录认证，创建token'''
    def post(self, request):
        response = {"code": 20000, "data": {"name": None, "token": None, "msg": None}}
        try:
            name = request.data.get("username")
            pwd = request.data.get("password")
            user = models.UserInfo.objects.filter(username=name, password=pwd).first()
            if user:
                # token = get_random(name)
                # 将name进行加密,3600设定超时时间
                token = get_token(name, 60)
                models.UserToken.objects.update_or_create(user=user, defaults={"token": token})
                response["data"]["msg"] = "登入成功"
                response["data"]["token"] = token
                response["data"]["name"] = user.username
            else:
                response["code"] = 20001
                response["data"]["msg"] = "用户名或密码错误"
        except Exception as e:
                response["code"] = 20002
                response["data"]["msg"] = "请求异常"

        return JsonResponse(response)
```

- 问题1： 有些API需要用户登录成功后才能访问，有些无需登录就能访问；
- 解决：
      - 创建两张表，用户信息表和用户token表
      - 用户登录（返回token并保存到数据库）
- 认证流程原理

##### 权限

   - 使用 

        - 类，必须继承：BasePermission,必须实现：has_permission方法

          ```python
          from rest_framework.permissions import BasePermission
          class MyPermission(BasePermission):
              message = "没有访问权限"
              def has_permission(self, request, view):
                  print(request.user.user_type)
                  if request.user.user_type != 3:
                      return False
                  return True
          ```

     - 返回值：

       - True,有权限访问
       - False,无权限访问

     - 全局使用

       ```python
       REST_FRAMEWORK = {
           "DEFAULT_AUTHENTICATION_CLASSES": ['myModule.authentication_module.FirstAuthtication', 'myModule.authentication_module.TokenAuth2'],
           "UNAUTHENTICATED_USER": None,
           "UNAUTHENTICATED_TOKEN": None,
           "DEFAULT_RENDERER_CLASSES": [
               'rest_framework.renderers.JSONRenderer',
               'rest_framework.renderers.BrowsableAPIRenderer',
           ],
           "DEFAULT_PERMISSION_CLASSES": ["myModule.permission_module.MyPermission",]
       
       }
       ```

##### 节流（用户的访问频率控制）

    - 类， 继承：BaseThrottle, 实现：allow_request、wait
    - 类，继承：SimpleRateThrottle,实现：get_cache_key、scope = "自定义（配置文件中的Key）"

##### 版本处理



##### 解析器

##### 序列化（比较重要）

- 请求数据进行校验

- QuerySet进行序列化

##### 分页（重要）

##### 路由（重要）

##### 视图（重要）

##### 渲染器

