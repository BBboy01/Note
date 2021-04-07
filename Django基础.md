# Django基本操作

```python
# 命令行操作
	# 1.创建Django项目
  	django-admin startproject 项目名(mysite)
    				
    		mysite文件夹
        	manage.py
          mysite文件夹
          	settings.py
            urls.py
            wsgi.py
  # 2.启动Django项目
    """
      一定要先切换到项目目录下
      cd mysite
    """
    python mange.py runserver [127.0.0.1:9090]
    # 默认 http://127.0.0.1:8000/
 # 3.创建应用
		"""
			Next, start your first app by running python manage.py startapp 						[app_label].
		"""
  python manage.py startapp 应用名字(app01)
	 
# pycharm操作
	# 1.new project 选择左侧第二个django即可
  
  # 2.启动
  		1.还是命令行启动
    	2.直接点击绿色小箭头即可
  
  # 3.创建应用
  	1.pycharm提供的终端直接输入完整命令
    2.pycharm
    		tools
      		run manage.py task提示
  
  # 4.修改端口号
```

# 主要文件介绍

```python
-mysite项目文件夹
	--mysite文件夹
  	---settings.py	配置文件
    ---urls.py			路由与视图函数对应关系(路由层)
    ---wsgi.py			wsgi模块(不考虑)
  --manage.py				Django的入口文件
  --db.sqlite3			Django自带的sqlite3数据库(小型数据库，功能不是很多还有bug)
  --app01文件夹
  	---admin.py			Django后台管理
    ---apps.py			注册使用
    ---migrations文件夹		数据库迁移记录
    ---models.py		数据库相关的模型类(orm)
    ---tests.py			测试文件
    ---views.py			视图函数(视图层)
```

# 用命令行与pycharm创建的区别

```python
# 1.命令行创建不会自动有template文件夹 需要自己手动创建，而pycharm会帮你自动创建并且还会自动在配置文件中配置对应的路径
# pycharm 创建
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')]
    },
]

# 命令行创建
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [ ]
    },
]
"""
也就意味着用命令行创建Django项目的时候需要配置路径
"""
```

# Django小白必会三板斧

```python
"""
from django.shortcuts import render, HttpResponse, redirect

HttpResponse
	返回字符串类型的数据

render
	返回html文件

redirect
	重定向
"""
```



# 静态文件配置

```python
# 登录功能

"""
html文件默认都放在template文件夹下
网站所用的静态文件默认都放在static文件夹下

静态文件夹
	前段以及写好的能直接调用的文件
		网站写好的js文件
		网站写好的css文件
		网站用到的图片文件
		第三方前段框架
		...
"""
# Django默认是不会自动创建static文件夹 需要自己手动创建


"""
在浏览器中输入url能看到对应的资源
是因为后端提前开设了该资源的接口

如果访问不到资源
说明后端没有开放该资源的接口
"""

# 手动配置静态文件


"""
**************************************************************
当在写Django项目的时候 可能会出现后端修改了代码但是前段页面没有变化的情况
	1.在同一个端口开了好几个Django项目
	
	2.浏览器缓存问题
		settings
			network
				disable cache 勾上
**************************************************************
"""

# 静态文件配置
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static')
]

# 静态文件动态解析
	{% load static %}
    <link rel="stylesheet" href="{% static 'bootstrap-3.3.7-dist/css/bootstrap.min.css' %}">
    <script src="{% static 'bootstrap-3.3.7-dist/js/bootstrap.min.js' %}"></script>
    
# form 表单默认是 get 请求
"""
form 表单action参数
	1.不写 默认朝当前所在的url提交数据
	2.全写 指名道姓
	3.只写后缀 /login/
"""


# 在Django中使用post请求时 需要在配置文件中注释一行代码
# 'django.middleware.csrf.CsrfViewMiddleware',
```

# 开放后端文件夹

```python
from django.views.static import serve
from django.conf import settings
urlpatterns = [
    path('admin/', admin.site.urls),
    # 开放media文件夹
    re_path('media/(?P<path>.*)', serve, {'document_root': settings.MEDIA_ROOT})

]
```



# request对象方法初始

```python
request.method  # 返回请求方式 并且全是大写的字符串形式
request.POST  # 获取用户post请求提交的普通数据不包含文件
	request.POST.get()  # 只获取列表最后一个元素
  request.POST.getlist()  # 直接将列表取出
request.GET  # 同 request.POST 类似
```

# Django链接数据库（MySQL）

```python
# 默认用的是 sqlite3
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

# Django链接MySQL
	1.配置文件中的settings
  	DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'day60',
        'USER': 'root',
        'PASSWORD': 'baba94shuai',
        'HOST': '127.0.0.1',
        'PORT': 3306,
        'CHARSET': 'utf8'
    }
}
  
  2.代码声明
  	Django默认用的是mysqldb模块链接MySQL
    但是该模块的兼容性不好 需要手动改为用pymysql链接
    
    # 在项目名下的 __init__ 或者任意的应用名下的 __init__ 文件中书写以下代码
    	import pymysql
			pymysql.install_as_MySQLdb()
```

# Django ORM

```python
"""
ORM 对象关系映射
类										表
对象								 记录
对象属性							记录某个字段对应的值

应用下面的models.py文件
"""

# 1.先去models.py中写一个类
	class User(models.Model):
    # 当不定义主键字段的时候 ORM会自动创建一个名为id的主键字段
    # id int primary key auto increment
    id = models.AutoField(primary_key=True)
    # username varchar(32)
    username = models.CharField(max_length=32, verbose_name='用户名')
    """
    CharField必须要指定max_length参数
    verbose_name是所有字段都有的 用来对字段的解释
    """
    # password int
    password = models.IntegerField()
   
*******************2.数据库迁移命令*********************
python manage.py makemigrations  # 将操作记录记录到migrations文件夹
python manage.py migrate  			 # 将操作真正的同步到数据库中
# 只要修改了models.py中跟数据库相关的代码 就必须重新执行上述的两条命令



```

# 字段的增删改

```python
# 字段的增加
	1.可以在终端内直接给出默认值
  2.该字段可以为空
  	# 该字段可以为空
    info = models.CharField(max_length=32, null=True, verbose_name='个人简介')
    # 默认为 study
    hobby = models.CharField(max_length=32, verbose_name='爱好',default='study')
    
# 字段的修改
	直接修改代码 然后执行数据库迁移的两条命令
  
# 字段的删
	直接注释对应字段 然后执行数据库迁移命令
  执行完毕后字段对应的数据就没有了
 
```

# 数据的增删改查

```python
# 查
res = models.User.objects.filter(username=username)
"""
返回值为 列表套数据对象的格式
它支持索引取值 切片操作 但不支持负数索引
它不推荐使用索引的方式取值 推荐.first（）
res = models.User.objects.filter(username=username).first()
"""
filter 括号内可以携带多个参数 参数与参数之间默认是and关系
可以吧filter联想成where

		# 年龄大于32岁的
    res = models.User.objects.filter(age__gt=32)
    # 年龄小于32岁的
    res = models.User.objects.filter(age__lt=32)
    # 年龄大于等于32岁的
    res = models.User.objects.filter(age__gte=32)
    # 年龄小于等于32岁的
    res = models.User.objects.filter(age__lte=32)

    # 年龄是18，32或者40
    res = models.User.objects.filter(age__in=[18, 32, 40])
    # 年龄在18到40岁之间的
    res = models.User.objects.filter(age__range=[18, 40])

    # 查询出名字里面含有n的数据 默认区分大小写
    res = models.User.objects.filter(name__contains='n')
    # 查询出名字里面含有n的数据 忽略区分大小写
    res = models.User.objects.filter(name__icontains='n')

    # 查询注册时间是 2020年1月
    res = models.User.objects.filter(register_time__month='1')


# 增
	from app01 import models
	# 第一种增加
	# 返回值就是当前被创建的对象本身
	# res = models.User.objects.create(username=uname, password=upwd)
        
	# 第二种增加
	user_obj = models.User(username=uname, password=upwd)
	user_obj.save()  # 保存数据


# 查
	user_queryset = models.User.objects.all()
  
  
# 改
	# 点击编辑按钮朝后端发送编辑数据请求
  """
  将编辑按钮所在那一行数据的主键值发送给后端
  利用url问号后面携带参数的方式
  
  {% for user_obj in user_queryset %}
                        <tr>
                            <td>{{ user_obj.id }}</td>
                            <td>{{ user_obj.username }}</td>
                            <td>{{ user_obj.password }}</td>
                            <td>
                                <a href="/edit_user/?user_id={{ user_obj.id }}" class="btn btn-primary btn-xs">编辑</a>
                                <a href="" class="btn btn-danger btn-xs">删除</a>
                            </td>
                        </tr>
                    {% endfor %}
  """
  
  # 修改方式1
        # models.User.objects.filter(id=edit_id).update(username=uname, password=upwd)
        """
        批量更新
        只修改被修改的字段
        """
        # 修改方式2
        user_obj.username = uname
        user_obj.password = upwd
        user_obj.save()
        """
        当字段特别多的时候效率非常低
        从头到尾将数据的所有字段全部再写一遍
        """
        
  # 后端查询出用户想要编辑的数据对象 展示到前端供用户查看和编辑
  
  
# 删
	# 获取用户想要删除的对象
    # res = models.User.objects.filter(pk=2).delete()
    """
    pk 会自动查找到当前表的主键字段 指代的就是当前表的主键字段 这样就不需要知道主键叫什么了
    """
    # user_obj = models.User.objects.filter(pk=1).first()
    # user_obj.delete()
```

# 必知必会13种ORM数据操作

```python
# 1.all()   查询所有数据
    # 2.filter()    带有过滤的查询
    # 3.get()   直接拿数据对象 但条件不存在会报错
    # 4.first() 拿queryset里面第一个元素
    # 5.last()  拿queryset里面最后一个元素
    # 6.values()    可以指定获取的数据字段 return 列表套字典
    # 7.values_list()   可以指定获取的数据字段 return 列表套元组
    """
    查看内部封装的sql语句
    只能用于queryset对象
    只有queryset对象才能通过 .query 查看内部的sql语句
    """
    # 8.distinct()  去重 一定要是一模一样的数据
    # 9.order_by()  排序
    # 10.reverse()  反转  先排序才能反转
    # 11.count()    统计当前数据个数
    # 12.exclude()  排除在外
    # 13.exists()   判断某个东西是否存在  return bool
```



# ORM 创建外键关系的表

```python
from django.db import models

# Create your models here.


# 创建表关系 先将基表创建出来 然后再填加外键字段
class Book(models.Model):
    title = models.CharField(max_length=32)
    # 小数总共八位 小数点后只有两位
    price = models.DecimalField(max_digits=8, decimal_places=2)
    """
    图书和出版社是多对一的关系
    图书是多的一方
    所以外键放在图书
    """
    publisher = models.ForeignKey(to='Publisher')  # 默认是与出版社表的主键字段做外键关联
    """
    如果对应的字段是 ForeignKey 那么ORM会自动在字段名字后面加_id
    如果自己加了_id ORM还是会在字段名字后面加_id
    """
    """
    图书和作者是多对多的关系
    外键字段在任意一方均可
    但是推荐放在查询频率较高的一方
    """
    # author 是一个虚拟字段 主要是告诉ORM图书和作者是多对多的关系 ORM会自动创建第三章关系表
    author = models.ManyToManyField(to='Author')
    
    register_time = models.DateField()  # 年月日
    """
    DateField
    DateTimeField
        两个重要参数
        auto_now:每次操作数据的时候 该字段会自动将当前时间更新
        auto_now_add：在创建数据的时候会自动将当前创建时间记录 之后只要不人为修改就一直不变
    """


class Publisher(models.Model):
    name = models.CharField(max_length=32)
    addr = models.CharField(max_length=32)


class Author(models.Model):
    name = models.CharField(max_length=32)
    age = models.IntegerField()
    """
    作者与作者详情是一对一的关系
    外键字段放在任意一方均可
    但是推荐放在查询频率较高的表
    """
    author_detail = models.OneToOneField(to='AuthorDetail')
    """
    OneToOneField 也会自动在字段名后面加 _id
    """


class AuthorDetail(models.Model):
    phone = models.BigIntegerField()
    addr = models.CharField(max_length=32)
 
"""
ORM中如何定义三种关系
	    publisher = models.ForeignKey(to='Publisher')  # 默认是与出版社表的主键字段做外键关联
			author = models.ManyToManyField(to='Author')
      author_detail = models.OneToOneField(to='AuthorDetail')

					ForeignKey OneToOneField
					会自动在字段后面加_id后缀
"""
# 在Django1.x版本中外键默认都是级联更新删除的
```

# 路由层

## 路由匹配

```python
# 路由匹配
    url(r'test', views.test),
    url(r'testadd', views.testadd)
"""
url方法第一个参数是正则表达式
	只要第一个参数正则表达式能匹配到内容 就直接停止往下匹配并执行视图函数
"""
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    # 首页
    url(r'^$', views.home),
    # 尾页
    url(r'', views.error),
    # 路由匹配
    url(r'^test/', views.test),
    url(r'^testadd/', views.testadd),
]
```



## 无名分组

```python
url(r'^test/(\d+)/(\d+)/', views.test)

def test(request, xx):
    print(xx)
    return HttpResponse('test')
  
# 无名分组就是讲括号内正则匹配到的内容当做参数传递给对应的视图函数
```



## 有名分组

```python
url(r'^testadd/(?P<year>\d+/)', views.testadd)

def testadd(request, year):
    print(year)
    return HttpResponse('testadd')
  
# 有名分组就是将括号内正则表达式匹配到的内容当做关键字参数传递给对应的视图函数
```

# 反向解析

```python
# 通过一些方法得到一个结果 该结果可以直接访问对应的url出发视图函数

# 先给路由与视图函数起一个别名
	url(r'^func/', views.func, name='ooo')
# 反向解析
	# 前段反向解析
  	<a href="{% url 'ooo' %}">111</a>
  # 后端反向解析
  	from django.shortcuts import reverse
  	reverse('ooo')
```

## 无名分组反向解析

```python
# 无名分组反向解析
    url(r'^index/(\d+)/', views.index, name='xxx')
 
# 前段
		{% url 'xxx' 123 %}
# 后端
		reverse('xxx', args=(1, ))
"""
数字一般情况下放的是数据的主键值 数据的编辑和删除
url(r'^index/(\d+)/', views.index, name='xxx')

def edit(request, edit_id):
	reserve('xxx', args=(edit_id, ))

{%for user_obj in user_queryset%}
	<a href="{% url 'xxx' user_obj.id %}">编辑</a>
{%endfor%}
"""
```

## 有名分组反向解析

```python
# 有名分组反向解析
    url(r'^func/(?P<year>\d+/)', views.func, name='ooo')
# 前段
	<a href="{% url 'ooo' year=123 %}"></a>  # 了解
	<a href="{% url 'ooo' 123 %}"></a>
# 后端
	# 有名分组反向解析 写法1 了解
    print(reverse('ooo', kwargs={'year': 123}))
    # 写法2
    print(reverse('ooo', args=(111, )))
```

# 路由分发

```python
"""
Django的每一个应用都可以有自己的template文件夹 urls.py static文件夹
正是基于上述特点 Django能够非常好的做到分组开发（每个人只写自己的app）
作为组长 只需要将手下书写的app全部拷贝到一个新的Django项目中 然后再在配置文件中注册
所有的app再利用路由分发的特点 将所有的app整合起来

利用路由分发后 总路由不再干路由与视图函数的直接对应关系 而是做一个分发处理
	识别当前url是属于哪个应用下的 直接分发给对应的应用去处理
"""

# 总路由
	# from app01 import urls as app01_urls
	# from app02 import urls as app02_urls

	urlpatterns = [
    url(r'^admin/', admin.site.urls),
    # 路由分发
    # url(r'^app01/', include(app01_urls)),  # 只要url前缀是aoo01开头 全部交给app01处理
    # url(r'^app02/', include(app02_urls)),  # 只要url前缀是aoo02开头 全部交给app02处理
    # 终极写法
    url(r'^app01/', include('app01.urls')),
    url(r'^app02/', include('app02.urls')),
    # 注意：总路由里面的url不能加$结尾 否则无法向下匹配
	]

# 子路由
	from django.conf.urls import url
	from app01 import views

	urlpatterns = [
    url(r'^reg/', views.reg)
	]
```

# 虚拟环境

```python
"""
在正常开发中 我们会给每一个项目配备一个该项目独有的解释器环境
该环境内只有该项目用到的模块 用不到的一概不装

虚拟环境
	每创建一个虚拟环境就类似于重新下载了一个纯净的python解释器
	但是虚拟环境不要过多创建 会消耗大量磁盘空间
	
	开发中我们会给每一个项目配备一个requirements.txt文件
	里面书写了该项目所有用到的模块和版本
"""
```

# Django版本区别

```python
"""
Django1.x路由层使用的是url方法
而在Django2.x和3.x版本中路由层使用的是path方法
	url()第一个参数支持正则
	path()第一个参数不支持正则 写什么就匹配什么
"""
```

# JsonResponse对象

```python
"""
Json格式数据
	前后端数据交互需要使用 实现跨语言传输数据
	
前段序列化
	JSON.stringfy()				json.dumps()
	JSON.parse()					json.loads()
"""
import json
from django.http import JsonResponse
def ab_json(request):
    user_dic = {'name': 'HB', 'age': 20, 'hobby': 'coding', 'log': '强无敌'}
    
    l = [11, 22, 33, 44]
    # json_str = json.dumps(user_dic, ensure_ascii=False)
    # return HttpResponse(json_str)
    # return JsonResponse(user_dic, json_dumps_params={'ensure_ascii': False})
    return JsonResponse(l, safe=False)  # 默认只能序列化字典 序列化其他需要加safe参数
```

# FORM表单上传文件及后端如何获取

```python
"""
form表单上传文件类型的数据
	1.method必须指定为POST
	2.enctype必须换成formdata
"""
def ab_file(request):
    if request.method == 'POST':
        # request.POST  # 只能获取到普通键值对数据 文件不行
        print(request.FILES)  # 获取文件数据
        file_obj = request.FILES.get('file')  # 文件对象
        print(file_obj.name)
        with open(file_obj.name, 'wb') as f:
            for line in file_obj.chunks():  # 推荐加上chunks方法 不加也一样
                f.write(line)
    return request(request, 'form.html')
```



# request对象方法

```python
"""
request.method
request.POST
request.GET
request.FILES
request.body  原生的浏览器发来的二进制数据
request.path
request.path_info
request.get_full_path  能够获取完整的url及get的参数
"""
		print(request.path)  # /app01/ab_file/
    print(request.path_info)  # /app01/ab_file/
    print(request.get_full_path())  # /app01/ab_file/?username=jason
```

# FBV与CBV

```python
# 视图函数可以是函数也可以是类

# CBV
	# CBV路由
    url(r'^login/', views.MyLogin.as_view())
    
  from django.views import View


class MyLogin(View):
    def get(self, request):
        return HttpResponse('get way')

    def post(self, request):
        return HttpResponse('post way')
      
"""
CBV特点
	能够直接根据请求方式的不同直接匹配到对应的方法执行
"""
```

# 模板语法传值

## {{}}:变量相关

## {%%}:逻辑相关

```python
# Django模板语法取值 是固定的格式 只能采用"句点式" .
<p>{{ d.username }}</p>
<p>{{ d.hobby.3.info }}</p>
# 即可以点键 也可以点索引 还可以两者混用
```

## 过滤器

```python
# 过滤器类似于模板语法 内置方法

# 基本语法
{{ 数据|过滤器:参数 }}

<p>统计长度：{{ s|length }}</p>
<p>默认值：{{ s|default:'nothing in it' }}</p>
<p>文件大小：{{ file_size|filesizeformat }}</p>
<p>日期格式化：{{ current_time|date:'Y-m-d H:i:s' }}</p>
<p>切片操作：{{ l|slice:'0:4:2' }}</p>
<p>切取字符（包含...）：{{ info|truncatechars:9 }}</p>
<p>切取单词（不包含...）：{{ egl|truncatewords:9 }}</p>
<p>移除特定的字符：{{ msg|cut:' ' }}</p>
<p>拼接：{{ l|join:'$'}}</p>
<p>拼接（加法）：{{ n|add:10 }}</p>
<p>拼接（加法）：{{ n|add:msg }}</p>
# 转义
 # 前段
  |safe
 # 后端
	from django.utils.safestring import mark_safe
  res = mark_safe('<h1>haha</h1>')
```

## 标签

```python
# for 循环
	{% for foo in l %}
		<p>{{ forloop }}</p>
	{% endfor %}

# if 判断
	{% if b %}
  	<p>baby</p>
  {% elif c %}
  	<p>666</p>
  {% else %}
  	<p>haha</p>
  {% endif %}

# for if 混用
	{% for foo in l %}
		{% if forloop.first %}
  		<p>这是第一次</p>
  	{% elif forloop.last %}
  		<p>这是最后一次</p>
  	{% else %}
  		<p>{{ foo }}</p>
  	{% endif %}
	{% endfor %}
 
# with 起别名
	{% with d.hobby.3.info as nb %}
  	<p>{{ nb }}</p>
    <p>{{ d.hobby.3.info }}</p>
  {% endwith %}
```

# 自定义过滤器、标签、inclusion_tag

```python
"""
1.在对应的应用下创建一个名字必须叫templatetags文件夹
2.该文件夹内创建一个任意名称的py文件
3.在该py文件内需要先书写两句固定的代码
	from django import template
	register = template.Library()
"""
# 自定义过滤器
@register.filter(name='过滤器的名字')
def index(v1, v2):
  return v1 + v2

# 自定义标签
@register.simple_tag(name='标签的名字')
def func(*args):
  pass

# 自定义inclusion_tag
@register.inclusion_tag('html文件名')
def bar(n):
  return locals()

# 使用上述三者
{% load mytag %}
{{ data|过滤器的名字:'参数' }}
{% 标签的名字 参数1 参数2 参数3 %}
{% bar %}
```



# 模板的继承

```python
"""
网站页面整体都大差不差 只是某一些局部在做变化
"""
# 模板的继承 自己先选好想要继承的模板页面
{% extends 'home.html' %}

# 继承了之后子页面跟模板页面长的一模一样 需要在模板页面上提前划定可以被修改的区域
{% block content %}
	模板内容(666)
{% endblock %}

# 子页面就可以声明想要修改哪块划定了的区域
{% block content %}
	子页面内容
  
  # 子页面除了自己写的之外 还可以继续使用模板的内容
  {{ block.super}}  # 666
{% endblock %}

# 一般情况下 模板页面上应该至少有三块可以被修改的区域
	1.css区域
  2.html区域
  3.js区域
  {% block css %}
  
  {% endblock %}
  
  {% block content %}
  
  {% endblock %}
  
  {% block js %}
  
  {% endblock %}
  
  每一个子页面就都可以有自己独有的css、js、html代码
```

# 模板的导入

```python
"""
将页面的某一个局部当成模块的形式
哪个地方需要就可以直接导入使用
"""
<p>模板的导入</p>
{% include 'wasai.html' %}
```

# 测试脚本

```python
"""
当你只想测试Django中的某一个py文件内容 可以不用写前后端交互形式 直接写一个测试脚本即可
"""
# 测试环境的准备 去manage.py中拷贝前四行代码 然后自己写两行
import os

if __name__ == "__main__":
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "day63.settings")
    import django
    django.setup()
    # 所有代码必须等环境准备完后才能书写
		from app01 import models
```

# 查看内部SQL语句方式

```python
# 方式1
res = models.User.object.value_list('name', 'age')  # <QuerySet>
# 只有queryset对象才能通过 .query 查看内部的sql语句

# 方式2 所有类型的对象都适用
# 去配置文件中配置一下即可
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console':{
            'level':'DEBUG',
            'class':'logging.StreamHandler',
        },
    },
    'loggers': {
        'django.db.backends': {
            'handlers': ['console'],
            'propagate': True,
            'level':'DEBUG',
        },
    }
}　
```

# 外键表关系的增删改查

```python
# 一对多外键增删改查

    # 增
    # 1.直接写实际字段 id
    # models.Book.objects.create(title='三国', price=123, publish_id=1)
    # 2.虚拟字段 对象
    # publish_obj = models.Publisher.objects.filter(pk=2).first()
    # models.Book.objects.create(title='四国', price=234, publish=publish_obj)

    # 删
    # models.Publisher.objects.filter(pk=2).delete()  # 级联删除

    # 改
    # models.Book.objects.filter(pk=1).update(publish_id=2)

    # publish_obj = models.Publisher.objects.filter(pk=2).first()
    # models.Book.objects.filter(pk=2).update(publish=publish_obj)

    # 多对多外键增删改查 就是在操作第三张表
    # 增
    book_obj = models.Book.objects.filter(pk=1).first()
    # book_obj.author  # 类似于已经到了第三章关系表
    book_obj.author.add(1)  # 书籍id为1的书籍绑定一个主键为1的作者
    book_obj.author.add(1, 3)  # 书籍id为1的书籍绑定一个主键为1、3的作者

    author_obj = models.Author.objects.filter(pk=1)
    author_obj1 = models.Author.objects.filter(pk=2)
    author_obj2 = models.Author.objects.filter(pk=3)
    book_obj.author.add(author_obj, author_obj1, author_obj2)
    """
    add给第三张表添加数据
        括号内即可以是数字也可以是对象 并且支持多个
    """

    # 删
    author_obj = models.Author.objects.filter(pk=1)
    author_obj1 = models.Author.objects.filter(pk=2)
    book_obj.author.remove(author_obj, author_obj1)
    """
    remove
        括号内即可以是数字也可以是对象 并且支持多个
    """

    # 改
    book_obj.author.set([1, 2])  # 括号内参数必须为可迭代对象
    book_obj.author.set([author_obj, author_obj1])
    """
    set
        括号内必须传一个可迭代对象 该对象内即可以是数字也可以是对象 并且支持多个
    """

    # 清空
    # 在第三张关系表中清空某个作者与书籍的绑定关系
    book_obj.author.clear()
    """
    clear
        括号内不要加任何参数
    """
```

# 基于对象的跨表查询

```python
# 1.查询书籍主键为1的出版社
    # book_obj = models.Book.objects.filter(pk=1).first()
    # # 书查出版社 正向
    # res = book_obj.publish
    # res.name
    # res.addr

    # 2.查询书籍主键为2的作者
    book_obj = models.Book.objects.filter(pk=2).first()
    # 书籍查作者 正向
    # res = book_obj.author  # app01.Author.None
    res = book_obj.author.all()

    # 3.查询作者json的电话号码
    author_obj = models.Author.objects.filter(name='json').first()
    res = author_obj.author_detail
    res.phone
    res.addr

    """
    当你的结果可能与多个的时候需要 .all()
    """
    # 4.查询出版社是东方出版社出版的书
    publish_obj = models.Publisher.objects.filter(name='东方出版社').first()
    # 出版社查书 反向
    res = publish_obj.book_set.all()

    # 5.查询作者是json写的书
    author_obj = models.Author.objects.filter(name='json').first()
    # 作者查书 反向
    res = author_obj.book_set.all()

    # 6.查询手机号是110的作者姓名
    author_detail_obj = models.AuthorDetail.objects.filter(phone=110).first()
    # 手机号查作者 反向
    res = author_detail_obj.author
    """
    基于对象
        反向查询
            当查询结果可能与多个的时候 必须加 _set.all()
            当查询结果只有一个的时候 不需要加 _set.all()
    """
```

# 基于双下划线的跨表查询

```python
# 1.查询json的手机号和作者姓名
    res = models.Author.objects.filter(name='json').values('author_detail__phone', 'name')
    res = models.AuthorDetail.objects.filter(author__name='json').values('author__name', 'phone')

    # 2.查询书籍主键为1的出版社名称和书的名称
    res = models.Book.objects.filter(pk=1).values('title', 'publish__name')
    res = models.Publisher.objects.filter(book__id=1).values('name', 'book__title')

    # 3.查询书籍主键为1的作者姓名
    res = models.Book.objects.filter(pk=1).values('author__name')
    res = models.Author.objects.filter(book_id=1).values('name')

    # 查询书籍主键为1的作者的手机号
    res = models.Book.objects.filter(pk=1).values('author__author_detail__phone')
```

# 聚合查询

```python
# 聚合查询  aggregate
    """
    聚合查询通常情况下都是配合分组一起使用的
    只要是跟数据库相关的模块 基本上都在django.db.models里面
    如果没有 那么就在django.db里面
    """
    from django.db.models import Sum, Max, Min, Count, Avg
    # 1.书的平均价格
    res = models.Book.objects.aggregate(Avg('price'))
    # 2.上述方法一次性使用
    res = models.Book.objects.aggregate(Max('price'), Min('price'), Count('price'), Sum('price'))
```

# 分组查询

```python
# 分组查询 annotate
    """
    MySQL分组查询特点
        分组之后默认只能获取到分组的依据 组内其他字段都无法直接获取了
            严格模式
                ONLY_FULL_GROUP_BY
    """
    from django.db.models import Sum, Max, Min, Count, Avg
    # 1.统计每一本书的作者个数
    res = models.Book.objects.annotate()  # models后面点什么 就是按什么分组
    res = models.Book.objects.annotate(author_num=Count('author')).values('title', 'author_num')  # models后面点什么 就是按什么分组
    """
    author_num是我们自己定义的字段 用来存储统计出来的每本书对应的作者个数
    """

    # 2.统计每个出版社卖的最便宜的书的价格
    res = models.Publisher.objects.annotate(min_price=Min('book__price')).values('name', 'min_price')

    # 3.统计不止一个作者的图书
        # 1.先按照书来分组
        # 2.再过滤出不止一个作者的书
    res = models.Book.objects.annotate(author_num=Count('author')).filter(author_num__gt=1).values('title', 'author_num')
    """
    只要ORM语句得出的结果还是一个queryset对象
    那么就可以继续点queryset对象封装的方法
    """

    # 4.查询每个作者出的书的总价格
    res = models.Author.objects.annotate(sum_price=Sum('book__price')).values('name', 'sum_price')
```

# F与Q查询

```python
# F与Q查询
    # 1.查询卖出数大于库存数的书籍
    # F查询
    from django.db.models import F
    """
    能够直接获取到表中某个字段对应的数据
    """
    res = models.Book.objects.filter(maichu__gt=F('kucun'))

    # 2.将所有书的价格提升50
    models.Book.objects.update(price=F('price')+50)

    # 3.将所有书的名称后面加上爆款两字
    """
    在操作字符类型的数据的时候 F不能直接进行字符串的拼接
    """

    # Q查询
    # 1.卖出数大于100或者价格小于600的书
    res = models.Book.objects.filter(maichu__gt=100, price__lt=600)  # and的关系
    from django.db.models import Q
    res = models.Book.objects.filter(Q(maichu__gt=100), Q(price__lt=600))  # , and关系
    res = models.Book.objects.filter(Q(maichu__gt=100) | Q(price__lt=600))  # | or关系
    res = models.Book.objects.filter(~Q(maichu__gt=100) | Q(price__lt=600))  # ~ not关系
```

# Django中开启事务

```python
"""
事务
	ACID
		原子性
			不可分割的最小点位
		一致性
			跟原子性相辅相成
		隔离性
			事务之间互不干扰
		持久性
			事务一旦曲儿永久生效
			
	事务的回滚
		rollback
	事务的确认
		commit
"""

# 事务
    from django.db import transaction
    with transaction.atomic():
        # sql1
        # sql2
        # 在with代码块内书写的所有ORM操作都属于同一个事务
    print('执行其他操作')
```

# ORM中常用字段及参数

```python
AutoField
	主键字段 primary_key=True
  
CharField				varchar
	verbose_name	字段的注释
  max_length		长度
  
IntegerField		int
BigIntegerFIeld	bigint

DecimalField
	max_digits=8
  decimal_places=2
  
EmailField		varchar(254)

DateField			date
DateTimeField	datetime
	auto_now		每次修改数据的时候都会更新当前时间
  auto_now_add 只在创建数据的时候回记录后续只要不人为更改就不会变
  
BooleanField	布尔类型
	该字段传布尔值(False/True)	数据库里面存0/1
  
TextField			文本类型
	该字段可以用来存大段内容(文章、博客...)	没有字数限制
  
FileField
	upload_to = '/data'	给该字段传一个对象 会自动将文件保存到/data目录下然后文件保存路径保存在数据库中/data/a.txt
  

# 外键字段及参数
unique = True
	ForeignKey(unique=True)  ===  OneToOneField()
  
db_index
	如果db_index=True 则代表着为此字段设置索引
  
to_field
	设置需要关联的表的字段 默认不写关联的就是另外一张的主键字段
  
on_delete
	当删除关联表中数据的时候 当前表与其关联的行为
  """
  django2.x版本及以上 需要自己指定外键字段的级联更新级联删除
  """
```

# 数据库查询优化

```python
only与defer
select_related与prefetch_related
"""
orm语句特点：
	惰性查询
		如果仅仅只是书写了orm语句 但后面没有用到该语句查询出来的结果
		那么orm会自动识别 直接不执行
"""

# only与defer
# 获取书籍表中所有书的名字
    res = models.Book.objects.values('title')  # 字典列表
    for d in res:
        print(d.get('title'))
    # only 查询
    """
        对象只有title属性 其他都没有
        查询其他属性要重新走数据库
    """
    res = models.Book.objects.only('title')  # 对象列表
    for i in res:
        print(i.title)  # 查询only内的字段不会走数据库
        print(i.price)  # 查询only外的字段要走数据库
    # defer 查询
    """
    对象除了没有title属性之外 其他都有
    查询title要重新走数据库 其他则不用
    """
    res = models.Book.objects.defer('title')  # 对象列表
    for i in res:
        print(i.title)
        
# select_related与prefetch_related 跟跨表操作有关
    res = models.Book.objects.all()
    for i in res:
        print(i.publish.name)  # 每循环一次就要走一次数据库查询

    res = models.Book.objects.select_related('publish')  # inner join
    """
    select_related内部直接先将book与publish连起来 然后一次性将大表里面的所有数据全部封装给查询出来的对象
    这时候无论是查询book还是publish中的数据都不需要再走数据库了
    select_related括号内只能放外键字段    一对多 一对一 （多对多不行）
    """
    res = models.Book.objects.prefetch_related('publish')  # 子查询
    """
    prefetch_related内部是子查询
        将子查询查询出来的所有结果封装到查询出来的对象中
    """
```

# choices参数

```python
"""
只要某个字段的可能性是可以列举完全的 那么一般情况下都会采用choices参数
"""
class User(models.Model):
  username = models.CharField(max_length=32)
  # 性别
  gender_choices = (
  		(1, '男'),
    	(2, '女'),
    	(3, '其他')
  )
  gender = IntegerField(choices=gender_choices)
  
  # 只要是choices参数的字段 如果想要获取对应的信息 固定写法 get_字段名_display
  user_obj = models.User.objects.filter(pk=1).first()
  user_obj.get_gender_display()
```

# MTV与MVC模型

```python
# MTV
M:models
T:templates
V:views
# MVC
M:models
V:views
C:controller
```

# 多对多三种查询方式

```python
# 全自动：利用orm自动帮我们创建第三张表
	class Book(models.Model):
    name = models.CharField(max_length=32)
    author = models.ManyToMany(to='Author')
    
  class Author(models.Model):
    name = models.CharField(max_length=32)
  """
  优点：代码不需要自己写 非常方便 支持orm踢动操作第三张表关系方法
  缺点：第三张表的扩展性差（没有办法额外添加字段）
  """
  
# 纯手动
	class Book(models.Model):
    name = models.CharField(max_length=32)
    
  class Author(models.Model):
    name = models.CharField(max_length=32)
    
  class Book2Author(models.Model):
    book_id = models.ForeignKey(to='Book')
    author_id = models.ForeignKey(to='Author')
  """
  优点：第三张表完全取决于自己进行额外的扩展
  缺点：需要些的代码多 不能够再使用orm提供的简单方法
  """
  
# 半自动
	class Book(models.Model):
    name = models.CharField(max_length=32)
    author = models.ManyToMany(to='Author',
                               through='Book2Author',
                               through_fields=('book', 'author')
                              )
    
  class Author(models.Model):
    name = models.CharField(max_length=32)
    # book = models.ManyToMany(to='Book',
    #                           through='Book2Author',
    #                           through_fields=('author', 'book')
    #                          )
    
  class Book2Author(models.Model):
    book = models.ForeignKey(to='Book')
    author = models.ForeignKey(to='Author')
    
    """
    througn_fields字段先后顺序
    	判断本质：
    		通过第三张表查询对应的表 需要用到哪个字段就把哪个字段放前面
    	简化判断：
    		当前表是谁 就把对应的关联字段放前面
    
    优点：可以使用orm的正反向查询
    缺点：没法使用add、set、remove和clear这四个方法
    """
# 为了拓展性更高 一般会使用半自动
```

# Ajax

```python
"""
异步请求
局部刷新

朝后端发送请求的方式
	1.浏览器地址栏直接输入url		 GET请求
	2.a标签href属性						 GET请求
	3.form表单							   POST/GET请求
	4.ajax									  异步POST/GET请求
"""
$('#btn').click(function () {
        $.ajax({
            url:'',  // 不写就是朝当前地址提交
            type:'get',  // 不指定默认是get
            data:{'i1':$('#d1'), 'i2':$('#d2')},
            // 当后端返回结果的时候回自动调用
            success:function (args) {
                {#alert(args)#}
                {#$('#d3').val(args)#}
                {#JSON.parse(args)#}
                console.log(typeof args)
            }
        })
    })
              
"""
针对后端如果是用HttpResponse返回数据 回调函数不会自动帮你反序列化
如果后端直接使用JsonResponse返回数据 回调函数会自动帮你反序列化

HttpResponse解决办法
	1.自己在前段使用JSON.parse()
	2.在ajax配置一个参数 dataType:'JSON'
"""
```

# 前后端传输数据的编码格式(contentType)

```python
"""
get请求数据就是直接放在url后面的
"""

# 可以朝后端发送post请求的方式
	"""
	1.form表单
	2.ajax请求
	"""

"""
前后端传输数据的编码格式
	urlencoded
	formdata
	json
"""
  
# form表单
	默认数据编码格式：urlencoded
  数据格式：username=jason&password=123
  django后端针对urlencoded编码格式的数据会自动解析封装到request.POST中
  
  如果把编码格式改成formdata 那么针对普通的键值对还是解析到request.POST中
  而将文件解析到request.FILES中
  
  form 表单无法发送json格式的数据
  
# ajax请求
	默认数据编码格式：urlencoded
  数据格式：username=jason&password=123
  django后端针对urlencoded编码格式的数据会自动解析封装到request.POST中
```

# ajax发送json格式数据

```python
"""
前后端传输数据的时候一定要确保编码格式跟数据真正的格式是一致的

{"username":"json","age":20}
在request.POST中找不到

django不对json格式的数据做处理 接收到的还是二进制数据

request.is_ajax()  返回布尔值 判断请求是否是ajax请求

json.loads()  会自动将二进制格式数据先解码再反序列化
"""

$('#d1').click(function () {
        $.ajax({
            url:'',
            type:'post',
            data:JSON.stringfy({'username':'json', 'pwd':123}),  // 转化为json
            contentType:'application/json',   // 指定编码格式
            success:function (args) {

            }
        })
    })

json_bytes = request.body
json_dict = json.loads(jons_bytes)
```

# ajax发送文件

```python
<script>
    $('#d4').on('click', function () {
        // 1.需要先利用FormData内置对象
        let formDataObj = new FormData
        // 2.添加普通的键值对
        formDataObj.append('username', $('#d1').val())
        formDataObj.append('password', $('#d2').val())
        // 3.同样支持添加文件
        formDataObj.append('myfile', $('#d3')[0].files[0])
        // 4.将对象基于ajax发送给后端
        $.ajax({
            url: '',
            type: 'post',
            data: formDataObj,  // 直接将对象放在data后面即可
            
            // ajax发送文件必须要指定两个参数
            contentType: false,  // 不需要任何编码 django后端能够自动识别formdata对象
            processData: false,  // 告诉浏览器不要对你的数据做任何处理
            
            success: function (args) {
                
            }
        })
    })
    
</script>



def ab_file(request):
    if request.is_ajax():
        if request.method == 'POST':
            print(request.POST)
            print(request.FILES)
    return render(request, 'ab_file.html')
  
  
  
"""
总结：
1.需要利用内置对象FormData
		// 2.添加普通的键值对
    formDataObj.append('username', $('#d1').val())
    formDataObj.append('password', $('#d2').val())
    // 3.同样支持添加文件
    formDataObj.append('myfile', $('#d3')[0].files[0])
 2.需要指定两个关键性参数
 		contentType: false,  // 不需要任何编码 django后端能够自动识别formdata对象
    processData: false,  // 告诉浏览器不要对你的数据做任何处理
 3.django后端能够直接识别到formdata对象 并且能够将内部的普通键值自动解析并封装到request.POST中 文件数据自动解析并封装到request.FILES中
"""
```

# 批量插入数据

```python
book_list = list()
for i in range(100000):
  book_obj = models.Book(title='第%s本书'%i)
  book_list.append(book_obj)
models.Book.objects.bulk_create(book_list)
```

# Django内置的序列化组件

```python
"""
工作中 大部分项目都是前后端分离的
也就意味着我们可能无法直接用django提供的模板语法来实现前后端数据交互

所以这个时候需要将数据处理成大家公共都能处理的格式（json）
一般情况下都是处理成列表套字典的形式[{},{}...]

由于针对数据的封装会非常的繁琐 所以用直接的模块直接完成
	1.django内置的模块
	2.第三方模块：django restframework（drf）
"""
from django.core import serializers
user_querryset = models.User.objects.all()
res = serializers.serialize('json', user_querryset)  # json字符串
return HttpResponse(res)
"""
[
	{
		"models":"app01.User",
		"pk":1,
		"fields":{
			"username":"jason",
					...
		}
	},
	{},{},...
]
"""
```

# 自定义分页器的拷贝及使用

```python
"""
当我们需要使用到django内置的第三方功能或者组件代码的时候
一般情况下会创建一个名为utils文件夹 在该文件夹内对模块进行功能划分
	utils可以在每个应用下创建 具体结合实际情况
"""
# 后端
book_queryset = models.Book.objects.all()
current_page = request.GET.get('page', 1)  # 1为默认值
all_count = book_queryset.count()

# 1.传值生成对象
page_obj = Pagination(current_page=current_page,all_count=all_count)
# 2.直接对总数据进行切片操作
page_queryset = book_queryset[page_obj.start: page_obj.end]
# 3.将page_queryset传递到页面 替换之前的book_queryset
return render(request, 'ab_pl.html', locals())


# 前端
{% for book_obj in page_queyset %}
	<p>{{ book_obj,title }}</p>
  <nav aria-label="Page navigation"></nav>
{% endfor %}
// 利用自定义分类器直接显示分页器样式
{{ page_obj.page_html|safe}}
```

# FORM组件

## 基本使用

```python
class MyForm(forms.Form):
    # username字符串类型 最小3位最大8位
    username = forms.CharField(min_length=3, max_length=8, label='用户名',
                               error_messages={
                                   'min_length': '用户名最少3位',
                                   'max_length': '用户名最多8位',
                                   'required': '用户名不能为空'
                               },
                              widget=forms.widgets.TextInput(attrs={
                                							'class': 'form-contorl c1 c2',
                                							'username': 'jason'
                              								})
                              )
    # password字符串类型 最小3位最大8位
    password = forms.CharField(min_length=3, max_length=8,label='密码',
                               error_messages={
                                   'min_length': '密码最少3位',
                                   'max_length': '密码最多8位',
                                   'required': '密码不能为空'
                               },
                              widget=forms.widgets.PasswordInput()
                              )
    # email字段必须符合邮箱格式 xx@xx.com
    email = forms.EmailField(label='邮箱',
                             error_messages={
                                 'required': '邮箱不能为空',
                                 'invalid': '邮箱格式不正确'
                             })
```

## 校验数据

```python
"""
1.测试环境可以自己拷贝代码准备
2.pycharm中有准备好的
	python console
"""
from app01 import views
# 1.将待校验的数据组织成字典的形式传入即可
form_data = views.MyForm({'username':'jason','password':'123','email':'122'})
# 2.判断数据是否合法	该方法只有在所有数据全部合法的情况下才会返回True
form_data.is_valid()	False
# 3.查看所有校验通过的数据
form_data.cleaned_data()	{'username':'jason','password':'123'}
# 4.查看所有不合法的数据及原因
form_data.errors	{'email':[ 'Enter a valid email address.' ]}
# 5.校验数据只校验类中出现的字段 多传不影响 多传的字段直接忽略
form_data = views.MyForm({'username':'jason','password':'123','email':'112@gg.com'， 'hobby':'read'})
form_data.is_valid()	True
# 6.校验数据 默认情况下 类里面所有字段都必须传值
form_data = views.MyForm({'username':'jason','password':'123'})
form_data.is_valid()	False
```

## 渲染标签

```python
"""
form组件只会自动渲染获取用户输入的标签(input select radio checkbox)
不会渲染提交按钮
"""
def index(request):
    # 1.先生成一个空对象
    form_obj = MyForm()
    # 2.直接将该空对象传递给html页面
    return render(request, 'index.html', locals())

# 前段利用空对象操作  
<form action="" method="post">
{#    <p>第一种渲染方式:代码书写少,封装程度太高 不便于后续的扩展</p>#}
{#    {{ form_obj.as_p }}#}
{#    {{ form_obj.as_ul }}#}
{#    {{ form_obj.as_table }}#}
{#    <p>第二种渲染方式:可扩展性强 但是需要书写的代码太多</p>#}
{#    <p>{{ form_obj.username.label }}:{{ form_obj.username }}</p>#}
{#    <p>{{ form_obj.password.label }}:{{ form_obj.password }}</p>#}
{#    <p>{{ form_obj.email.label }}:{{ form_obj.email }}</p>#}
    <p>第三种渲染方式(推荐使用):代码书写简单 并且扩展性高</p>
    {% for form in form_obj %}
        <p>{{ form.label }}:{{ form }}</p>
    {% endfor %}
    <input type="submit" class="btn btn-primary">
</form>
"""
label属性默认展示的是类中定义的字段首字母大写的形式
也可以自己修改 直接给字段对象设置label属性即可
username = forms.CharField(min_length=3, max_length=8, label='用户名')
"""
```

## 展示提示信息

```python
"""
浏览器会自动帮你校验数据 但是前段的校验弱不禁风
如何让浏览器不校验
	<form action="" method="post" novalidate>
"""
def index(request):
    # 1.先生成一个空对象
    form_obj = MyForm()
    if request.method == 'POST':
        # 获取用户数据并且校验
        """
        1.数据获取繁琐
        2.校验数据需要构造成字典的格式传入才行
        tips:request。POST可以看成是一个字典
        """
        # 两次对象的名字要相同
        # 3.校验数据
        form_obj = MyForm(request.POST)
        # 4.判断数据是否合法
        if form_obj.is_valid():
            # 5.如果合法 操作数据库存储数据
            return HttpResponse('OK')
        # 5.如果不合法 将错误信息展示到前段页面上

    # 2.直接将该空对象传递给html页面
    return render(request, 'index.html', locals())
  
  
{% for form in form_obj %}
        <p>
            {{ form.label }}:{{ form }}
            <span style="color: brown">{{ form.errors.0 }}</span>
        </p>
{% endfor %}
"""
1.必备条件 get请求和post传给html页面对象变量名必须一样
2.forms组件当你数据不合法的情况下 会保存上次的数据 让你基于之前的结果进行修改
错误提示也可以自己定制
"""
```

## 钩子函数(HOOK)

```python
"""
在特定的节点自动触发完成相应的操作

钩子函数在forms组件中类似于第二道关卡 能够让我们自定义校验规则

在forms组件中有两类钩子
	1.局部钩子
		当你需要给单个字段增加校验规则的时候可以使用
	2.全局钩子
		当你需要给多个字段增加校验规则的时候可以使用
"""
# 案例

# 1.校验用户名中不能含有666				只是校验username字段 局部钩子

# 2.校验密码和确认密码是否一致			password confirm两个字段 全局钩子


# 钩子函数 在类里面书写方法即可
		# 局部钩子
    def clean_username(self):
        # 获取到用户名
        username = self.cleaned_data.get('username')
        if '666' in username:
            # 提示前端展示错误信息
            self.add_error('username', '有不合法')
        # 将猴子函数钩出来的数据再放回去
        return username
    
    # 全局钩子
    def clean(self):
        password = self.cleaned_data.get('password')
        confirm_password = self.cleaned_data.get('confirm_password')
        if confirm_password != password:
            self.add_error('confirm_password', '两次密码不一致')
        # 返回所有数据
        return self.cleaned_data
```

## forms组件其他参数及补充

```python
label						字段名
error_message		自定义报错信息
initial					默认值
required				是否必填
widget					自定义样式
validators			自定义校验方式(支持正则)

"""
1.字段没有样式
2.针对不同类型的input如何修改
	text
	password
	date
	radio
	checkbox
"""
# 多个属性值 直接空格隔开即可
widget=forms.widgets.TextInput(attrs={'class': 'form-contorl c1 c2',
                                			'username': 'jason'
                              				})
# 第一道关卡里 还支持正则校验
validators=[RegexValidator(r'^[0-9]+$', '请输入数字'),
            RegexValidator(r'^159[0-9]+$', '数字必须以159开头')
]
```

# cookie与session

```python
cookie
	服务端保存在客户端浏览器上的信息都可以称之为cookie
  它的表现形式一般都是k:v键值对(可以有多个)
session
	数据是保存在服务端的 并且它的表现形式一般都是k:v键值对(可以有多个)
token
	session虽然将数据保存在服务端 但禁不住量大
  服务端不再保存数据 登录成功后 将一段信息进行加密处理
  将加密之后的结果拼在信息后面 整体返回给浏览器保存
  浏览器下次访问的时候带着该信息 服务端自动切去前面一段信息再次使用自己的加密算法跟浏览器尾部的密文进行比对
总结：
	1.cookie是保存在客户端浏览器上的信息
  2.session是保存在服务端上的信息
  3.session是基于cookie工作的
```

## cookie操作

```python
# 视图函数返回值
return HttpResponse()
return render()
return redirect()

obj1 = HttpResponse()
# 操作cookie
return obj1

obj2 = render()
# 操作cookie
return obj2

obj3 = redirect()
# 操作cookie
return obj3

"""
设置cookie
	obj.set_cookie(key, value)
获取cookie
	request.COOKIES.get(key)
在设置cookie的时候可以设置过期时间
obj.set_cookie('username', 'jason666', max_age=3, expires=3)  # 3秒过期
	max_age
	expires
		两者都是设置超时时间的 并且都是以秒为单位
		需要注意的是 针对IE浏览器需要使用expires
主动删除cookie
	@login_auth
	def logout(request):
    obj = redirect('/login/')
    obj.delete_cookie('username')
    return obj
"""
def login_auth(func):
    def call(request, *args, **kwargs):
        target_url = request.get_full_path()
        if request.COOKIES.get('username'):
            return func(request, *args, **kwargs)
        return redirect('/login/?next=%s' % target_url)
    return call


def login(request):
    if request.method == 'POST':
        username = request.POST.get('uname')
        password = request.POST.get('pwd')
        if username == 'jason' and password == '123':
            # 获取用户上一次想要访问的url
            target_url = request.GET.get('next')
            if target_url:
                obj = redirect(target_url)
            else:
                # 保存用户登录状态
                obj = redirect('/home/')
            # 让浏览器记录cookie数据
            obj.set_cookie('username', 'jason666', max_age=3, expires=3)  # 3秒过期
            """
            浏览器不单单帮你存cookie
            而且后面每次访问的时候还会带着它
            """
            # 跳转到一个需要用户登录之后才能看到的页面
            return obj
    return render(request, 'login.html')


@login_auth
def home(request):
    # # 获取cookie信息 判断是否登录
    # if request.COOKIES.get('username') == 'jason666':
    #     return HttpResponse('我是home页面')
    # # 没有登录跳转到登录页面
    # return redirect('/login/')
    return HttpResponse('this is home page')
```

## session操作

```python
"""
session数据是保存在服务端的 给客户端返回的是一个随机字符串
	sessionid:字符串
	
在默认情况下操作session的时候需要django默认的一张django_session表
	数据库迁移命令
		django会自动创建很多表 django_session就是其中一张

设置session
request.session['key'] = value

获取session
request.session.get('key')
		
django默认session的过期时间是14天
设置过期时间
	request.session.set_expiry()
		括号内可以放四种类型的参数
			1.整数			多少秒后失效
			2.日期对象	 到指定日期就失效
			3.0					一旦当前浏览器窗口关闭立刻失效
			4.不写			失效时间取决于django内部全局session默认的失效时间

清除session
	request.session.delete()  # 只删服务端的 客户端的不删
	request.session.flush()  # 浏览器和服务端都清空(推荐使用)
	
session是保存在服务端的 但是session的保存位置可以有多种选择
	1.MySQL
	2.文件
	3.redis
	4.memcache
	...

django_session表中的数据条数是取决于浏览器的
同一个计算机(IP地址)上同一个浏览器上只会与一条数据生效
"""
# 设置session
def set_session(request):
    request.session['hobby'] = 'girl'
    """
    内部发生的事
        1.django内部自动帮你生成一个随机字符串
        2.django内部自动将随机字符串和对应的数据存储到django_session表中
            2.1先在内存中产生操作数据的缓存
            2.2在响应结果经过django中间件的时候才真正操作数据库
        3.将产生的随机字符串返回给客户端浏览器保存
    """
    return HttpResponse('hhh')


# 获取session
def get_session(request):
    data = request.session.get('hobby')
    """
    内部发生的事
        1.自动从浏览器请求中获取sessionid对应的随机字符串
        2.拿着该随机字符串去django_session表中查找对应的数据
        3.如果比对上了 则将对应的数据取出来并以字典的形式封装到request.session中
          否则request,session返回的是None
    """
    return HttpResponse(data)
```

# CBV加装饰器

```python
from django.views import View
from django.utils.decorators import method_decorator
"""
CBV中 django不建议直接给类的方法加装饰器
无论该装饰器能否正常工作 都不建议直接加
"""
# @method_decorator(login_auth, name='get')  # 方式2(可以添加多个针对不同的方法加不同的装饰器)
# @method_decorator(login_auth, name='post')
class MyLogin(View):
    @method_decorator(login_auth)  # 方法3(直接作用于当前类里面的所有方法)
    def dispatch(self, request, *args, **kwargs):
        pass
    
    # @method_decorator(login_auth)  # 方式1
    def get(self, request):
        return HttpResponse('get request')

    def post(self, request):
        return HttpResponse('post request')
```

# django中间件

```python
"""
django中间件是django的门户
1.请求进来的时候需要先经过中间件才能到达真正的django后端
2.响应走的时候也要经过中间件才能出去

django自带7个中间件
"""

"""
django支持程序员自定义中间件并且暴露给程序员五个可以自定义的方法
	1.必须掌握
		process_request
			1.请求来的时候需要经过每一个中间件里面的process_request方法
			经过的顺序是按照配置文件中注册中间件从上往下的顺序依次执行
			2.如果中间件里面没有定义该方法 那么久直接跳过去执行下一个中间件
			3.如果该方法返回了HttpResponse对象 那么请求将不再继续往后执行 而是原路返回——process_request方法就是来做全局相关的所有限制功能
			
		process_response
			1.响应走的时候需要经过每一个中间件里面的process_response方法
			该方法有两个额外的参数 request, response
			2.该方法必须返回一个HttpResponse对象
				1.默认返回的就是形参response
				2.也可以返回自己定义的
			3.顺序是按照配置文件中注册了的中间件的顺序从下往上执行
			
		当在一个process_request方法里返回了HttpResponse对象 那么响应会直接走它同级别的process_response返回
		
	2.了解即可
		process_view
			路由匹配成功之后执行实行视图函数之前 会自动执行中间件里面的该方法
			经过的顺序是按照配置文件中注册中间件从上往下的顺序依次执行
			
		process_template_response
			返回的HttpResponse对象与render属性的时候才会触发
			顺序是按照配置文件中注册了的中间件的顺序从下往上执行
			
		porcess_exception
			当视图函数中出现异常的情况下触发
			顺序是按照配置文件中注册了的中间件的顺序从下往上执行
"""
```

## 自定义中间件

```python
"""
1.在项目名或者应用名下创建一个任意名称的文件夹
2.在该文件夹内创建一个任意名称的py文件
3.在该py文件内需要书写类(必须继承MiddlewareMixin)
	然后在这个类里面就可以自定义五个方法了(用几个写几个)
4.需要将类的路径以字符串的形式注册到配置文件中才能生效
"""
```

# csrf跨站请求伪造

```python
"""
内部本质
	我们在钓鱼网站的页面 针对对方账户 只给用户提供一个没有name属性的普通input框
	然后我们在内部隐藏一个已经写得name和value的input框
	
如何规避上述问题
	csrf跨站请求伪造校验
		网站在给用户返回一个具有提交数据功能页面的时候会给这个页面加一个唯一标识
		当这个页面朝后端发送post请求的时候 我们后端会先校验唯一标识 如果唯一标识不对 直接拒绝
"""
```

## 如何符合校验

```python
# form表单如何校验
<form action="" method="post">
    {% csrf_token %}
    <p>username:<input type="text" name="uname"></p>
    <p>password:<input type="password" name="pwd"></p>
    <input type="submit">
</form>

# ajax如何符合校验
            // 第一种 利用标签查找获取页面上的随机字符串
            {#data: {'username': 'jason', 'csrfmiddlewaretoken': $('[name=csrfmiddlewaretoken]').val()},#}
            // 第二种 直接利用模板语法
            {#data: {'username': 'jason', 'csrfmiddlewaretoken': '{{ csrf_token }}'},#}
            // 第三种 通用方式直接拷贝js代码并引用到自己的html页面即可
            data: {'username': 'jason'},
```

## csrf相关装饰器

```python
"""
1.网站整体不校验csrf 就单单几个视图函数需要校验
2.网站整体都校验csrf 就单单几个视图函数不校验
"""
from django.views.decorators.csrf import csrf_protect, csrf_exempt
"""
csrf_protect    需要校验
    符合装饰器的三种玩法
csrf_exempt     忽视校验
    只能给dispatch方法加才有效
"""
# @csrf_exempt
@csrf_protect
def transfer(request):
    if request.method == 'POST':
        usernam = request.POST.get('username')
        password = request.POST.get('password')
        money = request.POST.get('money')
    return render(request, 'transfer.html')


from django.views import View
from django.utils.decorators import method_decorator


# @method_decorator(csrf_protect)  # 第二种方法
# @method_decorator(csrf_exempt)  # 第二种方法 不行
class MyCsrfToken(View):
    # @method_decorator(csrf_protect)  # 第三种方法
    # @method_decorator(csrf_exempt)  # 第三种方法 可以
    def dispatch(self, request, *args, **kwargs):
        return super(MyCsrfToken, self).dispatch(request, *args, **kwargs)
    
    def get(self, request):
        return HttpResponse('get')
    
    # @method_decorator(csrf_protect)  # 第一种方法
    # @method_decorator(csrf_exempt)  # 第一种方法 不行
    def post(self, requets):
        return HttpResponse('post')
```

# 利用字符串是否注释决定是否开启相关功能

```python
import settings
import importlib


def send_all(content):
    for path_str in settings.NOTIFY_LIST:  # 'notify.email.Email'
        model_path, class_name = path_str.rsplit('.', maxsplit=1)  # 'notify.email Email
        # 1.利用字符串导入模块
        model = importlib.import_module(model_path)  # from notify import email
        # 2.利用反射获取类
        cls = getattr(model, class_name)  # <class Email object>
        # 3.生成对象
        obj = cls()
        # 4.直接调用send方法
        obj.send(content)
```

# Auth模块

```python
"""
其实我们在创建好一个django项目之后直接执行数据库迁移命令会自动生成很多表
	django_session
	auth_user
django在启动之后就可以直接访问admin路由 需要输入用户名和密码 数据参考的就是auth_user表 并且还必须是管理员才能进入

创建超级用户(管理员)
	python manage.py createsuperuser
	
依赖于auth_user表完成用户相关的所有功能
"""
```

## 方法总结

```python
from django.contrib import auth
# 1.比对用户名和密码是否正确
user_obj = auth.authenticate(request, username=username, password=password)     # 用户对象 如果数据不符合返回None user_obj.password 为密文
  # 括号内必须同时传入用户名和密码

# 2.保存用户状态
auth.login(request, user=user_obj)  # 类似于request.session[key] = user_obj
	# 只要执行了该方法 就可以在任何地方通过request.user获取到当前登录的用户对象
  
# 3.判断当前用户是否登录
request.user.is_authenticated()

# 4.获取当前登录用户
request.user

# 5.校验用户是否登录(装饰器)
from django.contrib.auth.decorators import login_required
	# 局部配置
  @login_required(login_url='/login/')
  # 全局配置
  LOGIN_URL = '/login/'
  # 优先级：局部>全局

# 6.比对原密码
	# 校验老密码对不对
	is_right = request.user.check_password(old_password)  # 自己加码比对密码

# 7.修改密码
request.user.set_password(confirm_password)
request.user.save()

# 8.注销
auth.logout(request)

# 9.注册
from django.contrib.auth.models import User
# 创建普通用户
# User.objects.create(username=username, password=password)  # 写入数据不能用create 密码是明文未加密
User.objects.create_user(username=username, password=password)
```

## auth模块表字段扩展

```python
from django.contrib.auth.models import User, AbstractUser

# 第一种：一对一关系 不推荐
class UserDetail(models.Model):
    phone = models.BigIntegerField()
    user = models.OneToOneField(to='User')
    

# 第二种：面向对象继承    
class UserInfo(AbstractUser):
    """
    如果继承了AbstractUser
    那么在执行数据库迁移命令的时候auth_user表就不会再创建出来了
    而UserInfo表中会出现auth_user所有的字段外加自己扩展的字段
    
    这样一来就能够直接点击你自己的表更加快速的完成操作及扩展
    
    前提：
        1.在继承之前没有执行过数据库迁移命令
            auth_user没有被创建，如果当前库已经创建了 那么就需要换一个库
        2.继承的类里面不要覆盖AbstractUser里面的字段
            表里面有的字段不要动 只扩展就行
        3.需要在配置文件中告诉django需要用UserInfo替代auth_user
            AUTH_USER_MODEL = 'app01.UserInfo'  # 应用名.表名
    """
    phone = models.BigIntegerField()
    """
    如果自己写表替代了auth_user
    那么auth模块的功能还是照常使用 参考的表由原来的auth_user变成了UserInfo
    """
```

