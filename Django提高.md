# DRF(Django REST FULL)简单使用

```python
# CBV
1.写一个序列化的类，继承Serializer
2.在类中写要序列化的字段，想序列化哪个字段，就在类中写哪个字段
3.在视图类中使用，导入-->实例化得到序列化类的对象，把要序列化的对象传入
4.序列化类的对象.data	type为str
5.把字典返回，如果不使用rest_framework提供的Response，就得使用JsonResponse

# 序列化类的字段类型
有很多 但常用的为：CharField, IntegerField, DateField...


# 序列化组件修改数据
1.写一个序列化的类，继承Serializer
2.在类中书写要反序列化的字段，想反序列化哪个字段，就在类中写哪个字段
		字段的属性：
  			max_length				最大长度
    		min_length				最小长度
      	allow_blank				是否允许为空
        trim_whitespace		是否截断空白字符
        max_value					最小值
        min_value					最大值
3.在视图类中使用，导入-->实例化得到序列化类的对象，把要修改的对象传入，修改的数据传入
		boo_ser = BookSerializer(book_obj, request.data)
  	boo_ser = BookSerializer(instance=book_obj, data=request.data)
4.数据校验	if boo_ser.is_valid()
5.如果校验通过，就保存数据
		boo_ser.save()  # 注意不是book_obj.save()
6.如果字段的校验规则不够，可以写钩子函数(局部和全局)
		# 局部钩子
  	def validate_price(self, data):  # validate_字段名 接收一个参数
      	# 如果价格小于10 就校验不通过
        if float(data) > 10:
          return data
        else:
          # 校验失败 抛异常
          raise ValidationError("价格太低")
     
    # 全局钩子
    def validate(self, validate_data):
      author = valodate_data.get('author')
      public = valodate_data.get('public')
      if author == public:
        raise ValidationError('作者名字和出版社一样')
      else:
        return validate_data
      
8.可以使用字段的author=serializer.CharField(validators=[check_author]) 来校验
		def check_author(data):
    		if data.startswith('sb'):
       			raise ValidationError('作者名字不能以sb开头')
        else:
          	return data
9.read_only和wirte_only
	read_only		表明该字段仅用于序列化输出，默认为False 如果设置成True，查询时可以看到，修改时不需要传该字段
  wirte_only	表明该字段仅用于反序列化输入，默认为False 如果设置成True，查询时看不到该字段。但修改时需要传递该字段
```

# 查询所有

```python
# views.py
class BooksView(APIView):
  def get(self, request):
    response_msg = {'status': 100, 'msg': '成功'}
    books = Book.objects.all()
    book_ser = BookSerializer(books, many=True)  # 序列化多条
    reponse_msg['data'] = book_ser.data
    return Response(response_msg)
  
# urls.py
path('books/', views.BooksView.as_view())
```

# 新增数据

```python
# views.py
class BooksView(APIView):
  # 新增
  def post(self, request):
    response_msg = {'status': 100, 'msg': '成功'}
    # 修改才有instance 新增只有data
    book_ser = BooksSerializer(data=request.data)
    # 校验字段
    if book_ser.is_valid():
        book_ser.save()
        response_msg['data'] = book_ser.data
    else:
      	response_msg['status'] = 202
        response_msg['msg'] = '数据校验失败'
        response_msg['data'] = book_ser.errors
        
# ser.py 序列化类重写creat方法
def creat(self, validate_data):
  instance = Book.objects.creat(**validate_data)
  return instance

# urls.py
path('books/', views.BooksView.as_veiw())
```

# 删除一个数据

```python
# views.py
class BooksView(APIView):
  	def delete(self, request, pk):
      	Book.objects.filter(pk=pk).delete()
        return Response({'status': 100, 'msg': '删除成功'})
      
# urls.py
re_path('books/(?P<pk>\d+)', views.BooksView.as_view())
```

# 模型类序列化器

```python
class BookModelSerializer(serializer.ModelSerializer):
  	class Meta:
      	model = Book  # 对应上model.py中的模型
        fields = '__all__'
        # fields = ('name', 'price', 'id', 'author')  只序列化指定的字段
        # exclude = ('name',)  写谁就表示不要谁
        # read_only_fields = ('price',)
        extra_kwargs = {
          	'price': {'wirte_only': True},
        }
# 其他使用不变
# 但不需要重写creat和update方法
```

# Serializer高级用法

```python
# source的使用(类似于 表名.source的值)
  1.可以改字段名字 xxx = serializers.CharField(source='title')
  2.可以 .跨表  publish = serializers.CharField(source='publish.email')
  3.可以执行方法  pub_date = serializers.CharField(source='test') test是Book表模型中的方法
# SerializerMethodField() 的使用
	1.它需要有个配套方法，方法名叫 get_字段名 返回值就是要显示的数据
  authors = serializers.SerializerMethodField()
  def get_authors(self, instance):
    	# book对象
      authors_obj = instance.authors.all()
      ll = list()
      for author in authors_obj:
        	ll.append({'name': author.name, 'age': author.age})
      return ll
```

# 路由

```python
# 1.在urls.py中配置
	path('books4/', views.Book4View.as_view()),
    re_pate('books4/(?P<pk>\d+)', views.Book4View.as_view()),
# 2.一旦视图类继承了ViewSetMixin
	# 查询所有
	path('book5/', views.Book5View.as_view(actions={'get': 'list', 'post': 'creat'}))  # 当路径匹配到又是get请求时会执行Book5View的list方法
    # 操作单个
    re_path('book5/(?P<pk>/d+)', views.Book4View.as_view(actions={'get': 'retrive', 'put': 'update', 'delete': 'destroy'}))
# 3.继承自视图类 ModelViewSet的路由写法(自动生成路由)
	-urls.py
        # 第一步：导入routers模块
        from rest_framework import routers
        # 第二步：有两个类 实例化对象得到
        # routers.DefaultRouter 生成的路由更多
        # routers.SimpleRouter
        router = routers.SimpleRouter()
        # 第三步：注册
        # router.register('book', views.BookViewSet) 地址不用加/
        # 第四步
        # router.urls 自动生成的路由 加入到原路由中
        urlpatterns += router.urls
    -views.py
    	from rest_framework.viewsets import ModelViewSet
        from app01.models import Book
        from app01.ser import BookSerializer
        class BookViewSet(ModelViewSet):
            queryset = Book.objects
            serializer = BookSerializer
```

# action的使用

```python
# action是为了给继承了ModelViewSet的视图类中定义的函数也添加路由
# 使用
class BookViewSet(ModelViewSet):
    authentication_classes = [MyAuthentication]
    queryset = models.Book.objects.all()
    serializer_class = BookSerializer
    # methods第一个参数 传一个列表 列表中放请求方式
    # ^books/get_1/$ [name='book-get-1']
    @action(methods=['GET', 'POST'], detail=True)
    def get_1(self, request, pk):
        book = self.get_queryset()[:2]  # 从0开始取1条
        ser = self.get_serializer(book.many = True)
        return Response(ser.data)
# 装饰器 放在被装饰的函数上方 methods：请求方式 detail：是否带pk
```

# 认证

```python
# 认证的实现
	1.写一个类，继承BaseAuthentication,重写authenticate，认证的逻辑写在里面
    2.全局使用，局部使用
    
# 可以有多个认证 从左到右依次执行
# 局部禁用 视图类中写 authentication_classes = []
# 全局配置
# settings.py
REST_FRAMEWORK={
    "DEFAULT_AUTHENTICATION_CLASSES": ["app01.app_auth.MyAuthentication",]
}

# 局部配置 在视图类里写
# views.py
authentication_classes = [MyAuthentication]
    
# 生成token views.py
class LoginView(APIView):
    authentication_classes = []  # 避免全局配置后未登录就要认证的bug
    def post(self, request):
        username = request.data.get('username')
        password = request.data.get('password')
        user_obj = models.User.objects.filter(username=username, password=password).first()
        if user_obj:
            # 登录成功,生成一个随机字符串，便于浏览器带着查询
            token = uuid.uuid4()
            # 存到usertoken表中
            # models.UserInfo.objects.create(token=token, user=user_obj)  # 用它每次登录都会记录一条 lj
            models.UserInfo.objects.update_or_create(defaults={'token': token}, user=user_obj)  # 有就更新 没有就创建 根据kwargs的参数查询决定创建default还是更新
            return Response({'status': 100, 'msg': '登录成功', 'token': token})
        else:
            return Response({'status': 101, 'msg': '用户名或密码错误'})
        
# app_auth.py(自己创建的)
from rest_framework.authentication import BaseAuthentication
from rest_framework.exceptions import AuthenticationFailed
from app01.models import UserInfo
class MyAuthentication(BaseAuthentication):
    def authenticate(self, request):
        # 认证逻辑，如果认证通过，返回两个值
        # 如果认证失败，抛出AuthenticationFailed异常
        token = request.GET.get('token')
        if token:
            user_token = UserInfo.objects.filter(token=token).first()
            if user_token:
                # 认证通过
                return user_token.user, token
            else:
                raise AuthenticationFailed('认证失败')
        else:
            raise AuthenticationFailed('请求地址中需要携带token')
            
```

# 权限

```python
# 写一个类 继承BasePermission 重写has_permisssion 如果权限通过就返回True 不通过返回False
from rest_framework.permissions import BasePermission
class UserPermission(BasePermission):
    def has_permission(self, request, view):
        # 不是超级用户不能访问
        user = request.user  # 当前登录用户
        # 如果该字段用了choice 通过get_字段名_display()就能取出choice里的中文
        if user.user_type == 1:
            return True
        else:
            return False
```

# 频率

```python
# 内置频率的限制
# 全局使用 限制未登录的用户1分钟访问5次
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': (
        'rest_framework.throttling.AnonRateThrottle',
    ),
    'DEFAULT_THROTTLE_RATES': {
        'anon': '5/m',
    }
}

class TestView3(APIView):
    authentication_classes = []
    permission_classes = []
    def get(self, request):
        return Response("这是测试数据")
    
# 局部使用 限制未登录用户1分钟访问5次
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_RATES': {
        'anon': '5/m',
    }
}

from rest_framework.throttling import AnonRateThrottle
class TestView4(APIView):
    authentication_classes = []
    permission_classes = []
    throttle_classes = [AnonRateThrottle, ]
    def get(self, request):
        return Response("这是测试数据")
    
    
    
    
# 自定义
from rest_framework.throttling import SimpleRateThrottle
class MyThrottle(SimpleRateThrottle):
    scope = 'aaa'

    def get_cache_key(self, request, view):
        return request.META.get('REMOT_ADDR')  # 根据ip来限制
    
class BookAPIView(APIView):
    throttle_classes = [MyThrottle, ]
```

# 过滤

```python
# 1.安装 pip install django-filter
# 2.注册 settings.py 中 INSTALLED_APPS 中注册
# 3.全局配或局部配
# 全局
REST_FRAMEWORK = {'DEFAULT_FILTER_BACKENDS': {'django_filter.rest_framework.DjangoFilterBackend',}}
# 4.视图类(查所有)
class BookView(ListAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    filter_fields = ('name', 'price')  # 配置可以按照哪个字段来过滤
```

# 排序

```python
# 局部使用和全局使用
# 局部
from rest_framework.generics import ListAPIView
from rest_framework.filters import OrderingFilter
from app01.models import Book
from app01.ser import BookSerializer
class Book2View(ListAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    filter_backends = [OrderingFilter]
    ordering_fields = ('id', 'price')
    
# 使用
http://127.0.0.1:8000/books2/?ordering=price
http://127.0.0.1:8000/books2/?ordering=-price  # 降序
```

# 异常处理

```python
# 目的：统一接口的返回

# 自定义异常方法，替换全局
# 写一个方法
# 自定义异常处理方法
from rest_framework import views
from rest_framework.response import Response
from rest_framework import status
def my_exception_handler(exc, context):
    response = views.exception_handler(exc, context)
    if not response:
        return Response(data={'status': 999, 'msg': str(exc), 'status': status.HTTP_400_BAD_REQUEST})
    else:
        return Response(data={'status': 888, 'msg': response.data.get('detail'), 'status': status.HTTP_400_BAD_REQUEST})

REST_FRAMEWORK = {'EXCEPTION_HANDLER': 'app01.app_auth.my_exception_handler'}
```

# 封装Response对象

```python
class APIResponse(Response):
    def __init__(self, code=100, msg='成功', data=None, status=None, headers=None, **kwargs):
        dic = {'code': code, 'mgs': msg}
        if not data:
            dic = {'code': code, 'mgs': msg, 'data': data}
        dic.update(kwargs)
        super().__init__(data=dic, status=status, headers=headers)
        
# 使用
# views.py
return APIResponse(data='cnkdnao', token='dnkncondosa')

# settings.py
REST_FRAMEWORK = {
    'DEFAULT_SCHEMA_CLASS': 'rest_framework.schemas.coreapi.AutoSchema'
}
```

# 自动生成接口文档

```python
# 1.安装 pip install coreapi

# 2.在路由中配置
from rest_framework.documentation import include_docs_urls
urlpatterns = {
    ...
    path('docs/', include_docs_urls(title='站点页面标题'))
}

# 3.视图类：自动接口文档能生成的是继承自APIView及其子类的视图
直接加文档字符串
class BookView(ListAPIView):
    '''
    返回图书所有信息
    '''
    def get(self, request):
        '''
        返回信息
        '''
```

# 一个视图多种功能

```python
# ser.py
from rest_framework import serializers
from rest_framework.exceptions import ValidationError
from api import models

class UserModelSerializer(serializers.ModelSerializer):
    re_password = serializers.CharField(max_length=16, min_length=4, required=True, write_only=True)  # 因为表中没有re_password这个字段 所有要在这定义

    class Meta:
        model = models.User
        fields = ('username', 'password', 'models', 're_password')
        extra_kwargs = {
            'username': {'max_length': 16},
            'password': {'write_only': True}
        }

    def validate_models(self, data):
        if not len(data) == 11:
            raise ValidationError('手机号不合法')
        return data

    def validate(self, attrs):
        if not (attrs.get('password') == attrs.get('re_password')):
            raise ValidationError('两次密码不一致')
        attrs.pop('re_password')
        return attrs

    def create(self, validated_data):
        # models.User.objects.create(**validated_data)  # 这个密码不会加密
        user = models.User.objects.create_user(**validated_data)
        return user


class UserReadOnlyModelSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.User
        fields = ('username', 'models')


class UserImageModelSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.User
        fields = ('icon', )

        
        
# views.py
from django.shortcuts import render
from rest_framework.viewsets import GenericViewSet
from rest_framework.mixins import CreateModelMixin, RetrieveModelMixin, UpdateModelMixin
from api import models
from api.ser import UserModelSerializer
from api import ser

# Create your views here.
# ViewSetMixin重写了as_view(),路由配置变样了 generics.GenericAPIView只需要配置queryset和serializer_class
class RegisterView(GenericViewSet, CreateModelMixin, RetrieveModelMixin, UpdateModelMixin):
    queryset = models.User.objects.all()
    # serializer_class = UserModelSerializer

    # 假设get请求和post请求 用的序列化类不一样
    # 重写get_serializer_class 返回什么 用的序列化类就是什么
    # 注册用的序列化类是UserModelSerializer 查询用的是UserReadOnlyModelSerializer
    def get_serializer_class(self):
        if self.action == 'create':
            return ser.UserModelSerializer
        elif self.action == 'retrieve':
            return ser.UserReadOnlyModelSerializer
        elif self.action == 'update':
            return ser.UserImageModelSerializer

        

# urls.py
from django.urls import path, include
from api import views
from rest_framework.routers import SimpleRouter

router = SimpleRouter()
router.register('register', views.RegisterView)

urlpatterns = [
    # path('register/', views.RegisterView.as_view(action={'post': 'create'})),
    path('', include(router.urls)),  # 第二种方式
]

urlpatterns += router.urls



# settings.py
MEDIA_URL = '/media/'

MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

AUTH_USER_MODEL = 'api.user'  # app名字.表名
```

# jwt控制登录用户访问

```python
from django.shortcuts import render
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework_jwt.authentication import JSONWebTokenAuthentication
# 内置权限类
from rest_framework.permissions import IsAuthenticated


# Create your views here.
# 使用jwt提供的认证类 局部使用
# 可以通过认证类JSONWebTokenAuthentication 和权限类IsAuthenticated 来控制用户登录以后才能访问某些接口
# 如果用户不登录 但是想访问某些接口 只需要把权限类IsAuthenticated 去掉就好了
class Order(APIView):  # 登录才能访问
    authentication_classes = [JSONWebTokenAuthentication, ]
    # 权限控制
    permission_classes = [IsAuthenticated, ]

    def get(self, request):
        return Response('这是订单信息')


class UserInfoAPIView(APIView):  # 不登录就可以访问
    authentication_classes = [JSONWebTokenAuthentication, ]
    # 权限控制
    # permission_classes = [IsAuthenticated, ]

    def get(self, request):
        return Response('UserInfoAPIView')
```

```python
# 控制登录接口返回的数据格式
    -用内置方法控制登录接口返回的数据格式
    	-jwt的配置信息中有个属性
        	'JWT_RESPONSE_PAYLOAD_HANDLER':
    		'rest_framework_jwt.utils.jwt_response_payload_handler'
        -重写jwt_response_payload_handler 配置成我们自己的
        # utils.py
        def my_jwt_response_payload_handler(token, user=None, request=None):  # 返回什么 前段就能看到什么
    return {
        'token': token,
        'msg': '成功',
        'status': 100,
        'username': user.username
    }

		# settings.py
    	# jwt的配置
        JWT_AUTH = {
            'JWT_RESPONSE_PAYLOAD_HANDLER': 				   'app02.utils.my_jwt_response_payload_handler'
        }
        
        # response
        {
            "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjo1LCJ1c2VybmFtZSI6ImVnb24iLCJleHAiOjE2MDI5MzA4MzYsImVtYWlsIjoiIn0.Hsizdwmn7SKS1KLUoY3R-HS8FMxbTCtEUfFa3Ig7C5E",
            "msg": "成功",
            "status": 100,
            "username": "egon"
        }
```

```python
# 实现jwt验证
# utils.py
from rest_framework.authentication import BaseAuthentication
from rest_framework_jwt.authentication import BaseJSONWebTokenAuthentication
from rest_framework_jwt.utils import jwt_decode_handler
from rest_framework.exceptions import AuthenticationFailed
from api import models
import jwt
# 第一种 继承BaseAuthentication 需要自己写
class MyJWTAuthentication(BaseAuthentication):
    def authenticate(self, request):
        jwt_value = request.META.get('HTTP_AUTHORIZATION')
        if jwt_value:
            # jwt 提供了通过三段token取出payload的方法 并且有校验功能
            try:
                payload = jwt_decode_handler(jwt_value)
            except jwt.ExpiredSignature as e:
                raise AuthenticationFailed("签名过期")
            except jwt.InvalidIssuer as e:
                raise AuthenticationFailed("非法用户信息")
            except Exception as e:
                raise AuthenticationFailed(str(e))
            # payload就是用户信息的字典
            # 需要返回user对象
            # 第一种 自己去数据库查 效率低
            # user = models.User.objects.get(pk=payload.get('user_id'))
            # 第二种 自己转
            user = models.User(id=payload.get('user_id'), username=payload.get('username'))
            return user, jwt_value
        raise AuthenticationFailed('未携带认证信息')


# 第二种 继承BaseJSONWebTokenAuthentication 有内置方法
class MyJWTAuthentication2(BaseJSONWebTokenAuthentication):
    def authenticate(self, request):
        jwt_value = request.META.get('HTTP_AUTHORIZATION')
        if jwt_value:
            # jwt 提供了通过三段token取出payload的方法 并且有校验功能
            try:
                payload = jwt_decode_handler(jwt_value)
            except jwt.ExpiredSignature as e:
                raise AuthenticationFailed("签名过期")
            except jwt.InvalidIssuer as e:
                raise AuthenticationFailed("非法用户信息")
            except Exception as e:
                raise AuthenticationFailed(str(e))
            user = self.authenticate_credentials(payload)
            return user, jwt_value
        raise AuthenticationFailed('未携带认证信息')
        
        
        
# views.py
from app02.utils import MyJWTAuthentication
class MerchandiseInfoAPIView(APIView):
    authentication_classes = [MyJWTAuthentication, ]
    # 权限控制
    # permission_classes = [IsAuthenticated, ]

    def get(self, request, *args, **kwargs):
        print(request.user)
        return Response('商品信息')
```

# 手动签发token(多方式登录)

```python
# 使用用户名、手机号、邮箱 都可以登录
# 前端传的数据格式
{
	"username": "hb/17609243433/1156678003@qq.com",
    "password": "hb123"
}

# ser.py
from rest_framework import serializers
from rest_framework.exceptions import ValidationError
from rest_framework_jwt.utils import jwt_payload_handler, jwt_encode_handler
from api import models
import re

class LoginModelSerializer(serializers.ModelSerializer):
    username = serializers.CharField()  # 重新覆盖字段 数据库中它是unique 自己有校验
    class Meta:
        model = models.User
        fields = ['username', 'password']

    def validate(self, attrs):
        username = attrs.get('username')  # 用户名有三种方式
        password = attrs.get('password')
        # 通过判断 username数据不同 查询字段不一样
        if re.match('^1[3-9][0-9]{9}$', username):
            user = models.User.objects.filter(models=username).first()
        elif re.match('^.+@.+$', username):
            user = models.User.objects.filter(email=username).first()
        else:
            user = models.User.objects.filter(username=username).first()
        if user:
            if user.check_password(password):
                # 签发token
                payload = jwt_payload_handler(user)
                token = jwt_encode_handler(payload)
                self.context['token'] = token
                self.context['username'] = user.username
                return attrs
            else:
                raise ValidationError('密码错误')
        else:
            raise ValidationError('用户不存在')

            

# views.py
from rest_framework.views import APIView
from rest_framework.viewsets import ViewSetMixin, ViewSet
from app02 import ser
# 手动签发token 完成多方式登录
# class Login2View(ViewSetMixin, APIView):
class Login2View(ViewSet):
    # 这是登录接口
    # def post(self, request):
    #     pass
    def login(self, request, *args, **kwargs):
        # 1.需要有个序列化的类
        # 2.生成序列化类对象
        login_ser = ser.LoginModelSerializer(data=request.data, context={'request': request})
        # 3.调用序列化对象的is_valid
        login_ser.is_valid(raise_exception=True)
        token = login_ser.context.get('token')
        username = login_ser.context.get('username')
        # 4.return
        return Response({'status': 100, 'msg': '登录成功', 'username': username, 'token': token})
    
    
    
    
# urls.py
from rest_framework_jwt.views import obtain_jwt_token

urlpatterns = [
    path('login2/', views.Login2View.as_view({'post': 'login'})),
]
```

# Jwt_token过期时间配置

```python
# settings.py
import datetime
JWT_AUTH = {
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=7),  # 过期时间 手动配置
}
```

# 总结

```python
# 1.restful规范-10条
# 2.django上符合restful规范的接口
# 3.drf写接口
# 4.APIView-->继承了原生View-->get,post方法
	- （为什么get请求来了，就会执行get方法：原生view的dispatch控制的）
    - 路由配置：视图类.as_view()--->view(闭包函数)的内存地址
    - 请求来了，就会执行view(request,分组分出的字段,默认传的字段)--->self.dispatch()
    - APIView重写了dispatch：包装了request对象，解析器，分页，三大认证，响应器，全局异常，去掉了csrf
# 5.Request对象：request, _request, request.data
	- 前端传过来的数据从哪取？
    	- 地址栏里：request.GET/query_params
        - 请求体重的数据：request.data/POST(json格式解释不了)-->request.body中取
        - 请求头中数据：request.META.get("HTTP_变成大写")
# Response对象-->封装了原生的HttpResponse，Response(data, headers={}, status=1/2/3/4/5开头的)
# 7.自己封装了Response对象
# 8.序列化类：
	- Serializer
    	- 写字段，字段名要跟表名字段对应，想不对应可以使用source修改，有属性 read_only, write_only, max_length...
        - SerializerMethodField必须配套一个 get_字段名 ,返回什么前台就看到什么
    - ModelSerializer
    	- class Meta：
        	表对应
            取出的字段(__all__, 列表)
            排除的字段
            extra_kwargs会给字段的属性
        - 重写某个字段
        	password = serializer.SerializerMethodField()
            def get_password(self, instance):
                return 'xxx'
        - 校验：字段自己的校验，局部钩子，全局钩子
        	- 只要序列化类的对象执行了is_valiad(),这些钩子都会走，可以在钩子里面写逻辑
    - 序列化多条(many=True):本质：ListSerializer内部套了一个个的serializer对象
    - 序列化类(instance, data, many, context={request:request})
    - 视图函数中给序列化对象传递数据，使用context，传回来，放进去直接使用序列化对象.context.get()
# 9.视图
	- 两个视图基类 APIView, GenericAPIView(继承APIView)：涉及到数据库和序列化类的操作，尽量用GenericAPIView
    - 5个视图扩展类(父类都是object)
    	CreateModelMixin：creat
        DestroyModelMixin：destroy
        ListModelMixin：get(many=True)
        RetrieveModelMixin：get
        UpdateModelMixin：update
    - 9个视图子类(GenericAPIView + 上面5个视图扩展类中的一个或多个)
    	DestroyAPIView
        CreateAPIView
        ListAPIView
        ListCreateAPIView
        RetrieveAPIView
        RetrieveUpdateAPIView
        RetrieveDestroyAPIView
        RetrieveUpdateDestroyAPIView
        UpdateAPIView
    - 视图集
    	GenericViewSet：ViewSetMixin + GenericAPIView
        ModelViewSet：5大接口都有了
        ReadOnlyModelViewSet：获取一条和获取多条的接口
        ViewSet：ViewSetMixin + APIView
        ViewSetMixin：重写了 as_view() 
# 10.路由
	- 基本配置：和之前一样
    - 有action的：必须继承ViewSetMixin
    - 自动生成：DefaultRouter和SimpleRouter
    	- 导入 实例化得到对象 注册多个 对象.urls(自动生成的路由)
        - 路由相加 urlpath += router.urls  /  include:path('', include(router.urls))
    - 视图类自定义的方法，如何自动生成路由
    	- 在自己定义的方法上加装饰器（aciton）
        - 两个参数 methods=[GET,POST] 表示这两种请求都能接受
        - 两个参数 detail=True 表示生成带pk的链接
# 11.三大认证
	- 认证组件
    	- 写一个认证类，继承BaseAuthentication，重写authenticate，内部写认证逻辑，认证通过返回两个值，第一个是user对象，认证失败抛出异常
        - 全局使用，局部使用，局部禁用
    - 权限：校验用户是否有权限进行后续操作
    	- 写一个类继承BasePermission，重写has_permission,返回True或False
    - 频率：限制用户访问频次
    	- 写一个类，继承SimpleRateThrottle，重写get_cache_key，返回什么，就以什么做限制  scop=luffy字段：需要跟setting中的key对应 luffy:3/h(一小时访问三次)
        - 全局使用，局部使用，局部禁用
# 12.解析器：前端传的编码格式，能不能解析（默认三种全配了，基本不需要改），可能你写了个上传文件接口，局部配置一下，只能传form_data格式 局部使用：MultiPartParser
# 13.响应器：响应的数据，是json格式还是浏览器的那种（不需要配）
# 14.过滤器：借助第三方django-filter
	- 注册应用
    - setting中配置DjangoFilterBackend或者局部配置
    - filter_field - ('age', 'sex')
# 15.排序
	- 全局或者局部配置rest_framework.filter.OrderingFilter
    - 视图类中配置：order_fields = ['id', 'age']
# 16.分页
	- 使用：
    	继承了APIView的视图类中使用
        	# 创建分页器对象
            page = Mypage()
            # 在数据库中获取分页的数据
            page_list = page.paginate_queryset(queryset对象, request, view=self)
            # 对分页进行序列化
            ser = BookSerializer(instance=page_list, many=True)
            return Response(ser.data)
    	继承了视图子类的属兔中使用
        	pagination_class = PageNumberPagination(配置成自己重写的,可以修改字段)
	- CursorPagination
    - LimitOffsetPagination
    - PageNumberPagination：最常用，需要在setting中配置page_size，四个参数
    	page_sieze 每页数目
        page_query_param 前端发送的页数关键字名 默认为'page'
        page_size_query_param 前端发送的每页数目关键字名 默认为'None'
        max_page_size 前端最多能设置的每页数量
# 17.全局异常
	- 写一个方法
    - 配置文件中配置（以后所有的drf异常都会走到这里）
    	REST_FRAMEWORK = {
            'EXCEPTION_HANDLER': 'my_project.my_app.utils.custom_exception_handler'
        }
# 18.jwt
	- 是什么：json web token 新的认证方式
    - 三段：头，荷载(用户信息)，签名
    - 使用：最简单方式(在路由中配置)
    	- path('login', obtain_jwt_token)
	- 自定制：
    	多方式登录，手动签发token(两个方法)
    - 自定制基于jwt的认证类
    	- 取出token
        - 调用jwt提供的解析出payload的方法(校验是否过期，是否合法，如果合法返回荷载信息)
        - 转成user对象
        - 返回
# 19.RBAC：基于角色的权限控制（django默认的auth就是在做这个事），公司内部权限锅里 对外用三大认证
    - 用户表
    - 用户组表
    - 权限表
    - 用户对用户组中间表
    - 用户组对权限中间表
	- 用户对权限中间表
```

