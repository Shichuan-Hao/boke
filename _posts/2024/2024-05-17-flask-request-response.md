---
title: Flask 的一些设理念
date: 2024-05-17 10:57:57 +0800
categories: [博客]
tags: [python, flask]
---


Flask 从客户端收到请求时，要让视图函数能访问⼀些对象，这样才能处理请求。**请求对象**就
是⼀个很好的例⼦，它封装了客户端发送的HTTP请求。

要想让视图函数能够访问请求对象，⼀种直截了当的⽅式是将其作为参数传⼊视图函数，不过这会
导致应⽤中的每个视图函数都多出⼀个参数。除了访问请求对象，如果视图函数在处理请求时还要
访问其他对象，情况会变得更糟。

为避免⼤量可有可⽆的参数把视图函数弄得⼀团糟，Flask使⽤**上下⽂**临时把某些对象变为全
局可访问。有了上下⽂，便可以像下⾯这样编写视图函数：

```python
@app.route('/')
def index():
    user_agent = request.headers.get('Use-Agent')
    return '<p>Your browser is {}</p>'.format(user_agent)
```

> 注意，在这个视图函数中我们把request当作全局变量使⽤。事实上，request不可能是全局变
量。试想，在多线程服务器中，多个线程同时处理不同客户端发送的不同请求时，每个线程看到的
request对象必然不同。Flask 使⽤上下⽂让特定的变量在⼀个线程中全局可访问，与此同时却不
会⼲扰其他线程。
{: .prompt-info }

在 Flask 中有两种上下文：**应用上下文**和**请求上下文**。

表：Flask上下文全局变量

| 变量名      | 上下文     | 说明 |
|:------------|:----------|--------:|
| `current_app`| 应用上下文 | 当前应用的应用实例 |
| `g`          | 应用上下文 | 处理请求时⽤作临时存储的对象，每次请求都会重设这个变量 |
| `request` | 请求上下⽂ | 请求对象，封装了客户端发出的 HTTP 请求中的内容 |
| `session` | 请求上下⽂ | ⽤户会话，值为⼀个字典，存储请求之间需要“记住”的值 |

Flask在分派请求之前先激活（或推送 ）应⽤和请求上下⽂，请求处理完成后再将其删除。
- 应⽤上下⽂被推送后，就可以在当前线程中使⽤`current_app`和`g`变量。
- 类似地，请求上下⽂被推送后，就可以使⽤`request`和`session`变量。
如果使⽤这些变量时没有激活应⽤上下⽂或请求上下⽂，就会导致错误。

## 请求分派

应⽤收到客户端发来的请求时，要找到处理该请求的视图函数。为完成这个任务，Flask会在应⽤
的URL映射中查找请求的URL。

URL映射是URL和视图函数之间的对应关系。Flask使⽤`app.route`装饰器或者作⽤相同的
`app.add_url_rule()`⽅法构建映射。

要想查看 Flask 应⽤中的 URL 映射是什么样⼦，可以在 Python shell 中
审查为 hello.py ⽣成的映射。测试之前，请确保你激活了虚拟环境：

```shell
(flasky) PS D:\byteswalk\pyoject\flasky\chap02> 
>>> from hello import app
>>> app.url_map

Map([<Rule '/' (HEAD, OPTIONS, GET) -> index>,
 <Rule '/static/<filename>' (HEAD, OPTIONS, GET) -> static>,
 <Rule '/user/<name>' (HEAD, OPTIONS, GET) -> user>])
```

- `/` 和 `/user/<name>` 路由在应⽤中使⽤ app.route 装饰器定义。
- `/static/<filename>` 路由是 Flask 添加的特殊路由，⽤于访问静态⽂件。

URL映射中的 (HEAD, OPTIONS, GET) 是**请求⽅法**，由路由进⾏处理。HTTP规范中规定，每
个请求都有对应的处理⽅法，这通常表⽰客户端想让服务器执⾏什么样的操作。Flask为每个路由都
指定了请求⽅法，这样即使不同的请求⽅法发送到相同的URL上时，也会使⽤不同的视图函数处理。
HEAD 和 OPTIONS ⽅法由 Flask ⾃动处理，因此可以这么说，在这个应⽤中，URL映射中的3个路
由都使⽤GET⽅法（表⽰客户端想请求信息，例如⼀个⽹⻚）。


## 请求对象

Flask通过上下⽂变量request对外开放请求对象。这个对象⾮常有⽤，包含客户端发送
的HTTP请求的全部信息。

Flask请求对象中最常⽤的属性和⽅法⻅表：
| 属性或方法 | 说明     |
|:------------|:----------|
| `from` | ⼀个字典，存储请求提交的所有表单字段 |
| `args` | ⼀个字典，存储通过 URL 查询字符串传递的所有参数 |
| `values` | ⼀个字典，form 和 args 的合集 |
| `cookies` | ⼀个字典，存储请求的所有 cookie |
| `headers` | ⼀个字典，存储请求的所有 HTTP ⾸部 |
| `files` | ⼀个字典，存储请求上传的所有⽂件 |
| `get_data()` | 返回请求主体缓冲的数据 |
| `get_json()` | 返回⼀个 Python 字典，包含解析请求主体后得到的 JSON |
| `blueprint` | 处理请求的 Flask 蓝本的名称 |
| `endpoint` | 处理请求的 Flask 端点的名称；Flask 把视图函数的名称⽤作路由端
点的名称 |
| `method` | HTTP 请求⽅法，例如 GET 或 POST |
| `scheme` | URL ⽅案（http 或 https ） |
| `is_secure()` | 通过安全的连接（HTTPS）发送请求时返回 True |
| `host` | 请求定义的主机名，如果客户端定义了端⼝号，还包括端⼝号 |
| `path` | URL 的路径部分 |
| `query_string` | URL 的查询字符串部分，返回原始⼆进制值 |
| `full_path` | URL 的路径和查询字符串部分 |
| `url` | 客户端请求的完整 URL |
| `base_url` | 同 url ，但没有查询字符串部分 |
| `remote_addr` | 客户端的 IP 地址 |
| `environ` | 请求的原始 WSGI 环境字典 |


## 请求钩子

有时在处理请求之前或之后执⾏代码会很有⽤。

例如，在请求开始时，我们可能需要创建数据库连接或者验证发起请求的⽤户⾝份。为了避免在
每个视图函数中都重复编写代码，Flask 提供了注册通⽤函数的功能，注册的函数可在请求被分
派到视图函数之前或之后调⽤。请求钩⼦通过装饰器实现。

Flask ⽀持以下 4 种钩⼦：

- `before_request`：注册⼀个函数，在每次请求之前运⾏。
- `before_first_request`：注册⼀个函数，只在处理第⼀个请求之前运⾏。可以通过这个钩
⼦添加服务器初始化任务。
- `after_request`：注册⼀个函数，如果没有未处理的异常抛出，在每次请求之后运
⾏。
- `teardown_request`：注册⼀个函数，即使有未处理的异常抛出，也在每次请求之后运
⾏。

在请求钩⼦函数和视图函数之间共享数据⼀般使⽤上下⽂全局变量 `g`。例如，`before_request`处
理程序可以从数据库中加载已登录⽤户，并将其保存到`g.user`中。随后调⽤视图函数时，便可以
通过`g.user`获取⽤户。

## 响应

Flask调⽤视图函数后，会将其返回值作为响应的内容。多数情况下，响应就是⼀个简单的字符
串，作为HTML⻚⾯回送客户端。

但HTTP协议需要的不仅是作为请求响应的字符串。HTTP 响应中⼀个很重要的部分是状态码，
Flask默认设为 200，表明请求已被成功处理。

如果视图函数返回的响应需要使⽤不同的状态码，可以把数字代码作为第⼆个返回值，添加到响应
⽂本之后。

例如，下述视图函数返回 400状态码，表⽰请求⽆效：
```python
@app.route('/')
def index():
    return '<h1>Bad Request</h1>', 400
```

视图函数返回的响应还可接受第三个参数，这是⼀个由 HTTP 响应⾸部组成的字典。

如果不想返回由 1 个、2 个或 3 个值组成的元组，Flask 视图函数还可以返回⼀个响应对象 。
`make_response()` 函数可接受 1 个、2 个或3 个参数（和视图函数的返回值⼀样），然后返
回⼀个等效的响应对象。有时我们需要在视图函数中⽣成响应对象，然后在响应对象上调⽤各个⽅
法，进⼀步设置响应。下例创建⼀个响应对象，然后设置 cookie：
```python
from flask import make_response
@app.route('/')
def index():
    response = make_response('<h1>This document carries a cookie!</h1>')response.set_cookie('answer', '42')
    return response
```

响应对象最常使⽤的属性和⽅法⻅表：
| 属性或方法 | 说明     |
|:------------|:----------|
| `status_code` | HTTP 数字状态码 |
| `headers` | ⼀个类似字典的对象，包含随响应发送的所有⾸部 |
| `set_cookie()` | 为响应添加⼀个 cookie |
| `delete_cookie()` | 删除⼀个 cookie |
| `content_length` | 响应主体的⻓度 |
| `content_type` | 响应主体的媒体类型 |
| `set_data()` | 使⽤字符串或字节值设定响应 |
| `get_data()` | 获取响应主体 |

响应有个特殊的类型，称为**重定向** 。这种响应没有⻚⾯⽂档，只会告
诉浏览器⼀个新 URL，⽤以加载新⻚⾯。重定向经常在 Web 表单中使
⽤。

重定向的状态码通常是 302，在 Location ⾸部中提供⽬标 URL。重
定向响应可以使⽤ 3 个值形式的返回值⽣成，也可在响应对象中设
定。不过，由于使⽤频繁，Flask 提供了 redirect() 辅助函数，⽤
于⽣成这种响应：
```python
from flask import redirect
@app.route('/')
def index():
    return redirect('http://www.example.com')
```

还有⼀种特殊的响应由 abort() 函数⽣成，⽤于处理错误。在下⾯这
个例⼦中，如果 URL 中动态参数 id 对应的⽤户不存在，就返回状态
码 404：
```python
from flask import abort
@app.route('/user/<id>')
def get_user(id):
    user = load_user(id)
    if not user:
        abort(404)
    return '<h1>Hello, {}</h1>'.format(user.name)
```

注意，`abort()` 不会把控制权交还给调⽤它的函数，⽽是抛出异常。