
### 接口测试

- 依据接口规范，写出测试用例
- 使用软件工具，直接通过消息接口对被测系统进行消息收发
- 验证被测系统行为是否正确

网页、图片、css 这些资源，都是 **静态资源** ， 就是一个个文件存储在服务器上的，获取这些信息，服务端直接读取文件，返回给客户端即可，无需特别的数据处理。

而 API接口请求消息，通常都需要 服务端程序进行 一番处理，比如：对请求的权限检查，从数据库中读出数据，进行信息过滤和 格式转换，最后在HTTP响应中返回给客户端。

接口测试需要 `工具` 和 被测系统之间进行消息（通常是HTTP消息）的收发， 这个工具 可以是 别人开发 好的， 也可以自己开发。

基于 HTTP 的接口测试工具， 常见的 有 Postman、Jmeter等

这些工具，核心功能都是类似的：都是用来 构建HTTP请求消息，并且解析收到的HTTP响应消息， 用户来判断是否符合预期

**主要工作**
- 获取接口文档，评审文档，了解接口的实现细节
    
- 根据接口文档，写出测试用例
    
- 等产品发布后，根据测试用例，使用软件工具，直接通过消息接口 对 被测系统 进行消息收发，验证被测系统行为是否正确 

**编写测试用例**：通常可以采用 条件组合、边界值、错误猜测 等方法。


#### Requests库

Requests库不是Python标准库，而是第三方开发的
Requests 库 是用来发送HTTP请求，接收HTTP响应的一个Python库。

Requests库经常被用来 爬取 网站信息。 用它发起HTTP请求到网站，从HTTP响应消息中提取信息。

Requests库也经常被用来做 网络服务系统的Web API 接口测试。因为Web API 接口的消息基本上都是通过HTTP协议传输的

```python
import requests  
  
# 尝试访问一个测试网站  
response = requests.get('https://mirrors.sohu.com')  
#返回response类的实例对象  
print(response.text)        # 打印响应的文本
```
这些 get、post、put、delete 函数被用，发送http消息给 服务端后，都会返回一个Requests库里面定义的一个 Response 类的实例对象。 这个对象代表着响应消息。

上面的例子中 `response.text` 使用了这个对象的 text 属性。就返回了http响应消息体中的文本内容


##### requests程序抓包

要让requests 发送请求使用代理，只需要如下参数进行设定即可

```python
import requests

proxies = {
  'http': 'http://127.0.0.1:8888',
  'https': 'http://127.0.0.1:8888',
}

response = requests.get('http://mirrors.sohu.com/', proxies=proxies)
print(response.text)
```

##### 构建请求URL参数

**url参数**

比如：
`https://www.baidu.com/s?wd=iphone&rsv_spt=1`

问号后面的部分 `wd=iphone&rsv_spt=1` 就是 url 参数，

每个参数之间是用 `&` 隔开的。

上面的例子中 有两个参数 wd 和 rsv_spt， 他们的值分别为 iphone 和 1 。

url参数的格式，有个术语叫 `urlencoded` 格式。

使用Requests发送HTTP请求，url里面的参数，通常可以直接写在url里面，比如：
`response = requests.get('https://www.baidu.com/s?wd=iphone&rsv_spt=1')`

但是有的时候，我们的url参数里面有些特殊字符，比如 参数的值就包含了 & 这个符号。

那么我们可以把这些参数放到一个字典里面，然后把字典对象传递给 Requests请求方法的 params 参数，如下
```python
urlpara = {
    'wd':'iphone&ipad',
    'rsv_spt':'1'
}

response = requests.get('https://www.baidu.com/s',params=urlpara)
```

##### 构建请求消息头

有时候，我们需要自定义一些http的消息头

每个消息头也就是一种 键值对的格式存放数据，如下所示
```python
user-agent: my-app/0.0.1
auth-type: jwt-token
```

Requests发送这样的数据，只需要将这些键值对的数据填入一个字典。

然后使用post方法的时候，指定参数 headers 的值为这个字典就可以了，如下:

```python
headers = {
    'user-agent': 'my-app/0.0.1', 
    'auth-type': 'jwt-token'
}

r = requests.post("http://httpbin.org/post", headers=headers)
print(r.text)
```

##### 构建请求消息体

进行API 接口测试的时候， 根据接口规范，构建的http请求，通常需要构建消息体。

http 的 消息体就是一串字节，里面包含了一些信息。这些信息可能是文本，比如html网页作为消息体，也可能是视频、音频等信息。

消息体可能很短 ，只有一个字节， 比如字符 a。 也可能很长，有几百兆个字节，比如一个视频文件。

最常见的消息体格式当然是 表示网页内容的 HTML。

Web API接口中，消息体基本都是文本，文本的格式主要是这3种： urlencoded ，json ， XML。

**注意**：消息体采用什么格式，是由 开发人员设计的决定的

###### XML格式消息体

消息体就是存放信息的地方，信息的格式完全取决设计者的需要。

如果设计者决定用 XML 格式传输一段信息，用Requests库，只需要这样：

```python
payload = '''
<?xml version="1.0" encoding="UTF-8"?>
<WorkReport>
    <Overall>良好</Overall>
    <Progress>30%</Progress>
    <Problems>暂无</Problems>
</WorkReport>
'''

r = requests.post("http://httpbin.org/post",
                  data=payload.encode('utf8'))
                  #如果传入的是字符串类型，Requests 会使用缺省编码 latin-1 编码为字节串放到http消息体中，发送出去;但是此例子中有中文，不能用缺省 latin-1编码
                  #所以用 utf8 编码为字节串对象Bytes 传入给data参数
print(r.text)
```

采用 xml、json 这样的标准格式，可以很好的表示复杂的信息数据
而且 编程语言都有现成的库处理解析 XML、json 这样的数据格式，我们直接拿来使用，非常方便；
而自己定义的格式就难以表达这样复制的数据格式，而且还要自己写代码在发送前进行格式转化，接收后进行格式解析，非常麻烦。

###### urlencoded 格式消息体

这种格式的消息体就是一种 键值对的格式存放数据，如下所示
`key1=value1&key2=value2`

将这些键值对的数据填入一个字典。

然后使用post方法的时候，指定参数 data 的值为这个字典就可以了，如下

```python
payload = {'key1': 'value1', 'key2': 'value2'}

r = requests.post("http://httpbin.org/post", data=payload)
print(r.text)
```


###### json 格式消息体

json 格式 当前被 Web API 接口广泛采用。

json 是一种表示数据的语法格式。 它和Python 表示数据的语法非常像。

比如要表示上面的报告信息，可以这样
```python
{
    "Overall":"良好",
    "Progress":"30%",
    "Problems":[
        {
            "No" : 1,
            "desc": "问题1...."
        },
        {
            "No" : 2,
            "desc": "问题2...."
        }
    ]
}

```

优点是：比xml更加简洁、清晰， 所以程序处理起来效率也更高

可以使用json库的dumps方法转换成json格式的字符串，如下
```python
import requests,json

payload = {
    "Overall":"良好",
    "Progress":"30%",
    "Problems":[
        {
            "No" : 1,
            "desc": "问题1...."
        },
        {
            "No" : 2,
            "desc": "问题2...."
        },
    ]
}

r = requests.post("http://httpbin.org/post", data=json.dumps(payload))
#如果想不让dumps把中文变成unicode编码方式
r = requests.post("http://httpbin.org/post", data=json.dumps(payload，ensure_adcii=False).encode())
```

也可以将 数据对象 直接 传递给post方法的 json参数，如下
`r = requests.post("http://httpbin.org/post", json=payload)`


#### 检查http响应

##### 响应状态码

检查 HTTP 响应 的状态码，直接 通过 reponse对象的 `status_code` 属性获取

```python
import requests

response = requests.get('http://mirrors.sohu.com/')
print(response.status_code)
```

##### 响应消息头

直接 通过 reponse对象的 `headers` 属性获取

response.headers 对象的类型 是 继承自 Dict 字典 类型的一个 类。

我们也可以像操作字典一样操作它

##### 响应消息头

获取响应的消息体的文本内容，直接通过response对象 的 `text` 属性即可获取
[[QA#^02|response的text与content]]
requests是 以什么编码格式 把HTTP响应消息体中的 字节串 解码 为 字符串的呢？
requests 会根据响应消息头（比如 Content-Type）对编码格式做推测；
但是有时候，服务端并不一定会在消息头中指定编码格式，这时， requests的推测可能有误，需要我们指定编码格式

通过这样方式指定
```python
import requests
response = requests.get('http://mirrors.sohu.com/')
response.encoding='utf8'
print(response.text)
```

如果我们要直接获取消息体中的字节串内容，可以使用 `content` 属性，
比如
```python
import requests

response = requests.get('http://mirrors.sohu.com/')
print(response.content)
```

###### 处理响应中的json格式数据

为了 `方便处理 响应消息中json 格式的数据` ，我们通常应该把 json 格式的字符串 转化为 python 中的数据对象

转化成字典，方便处理取出数据

参考[[note1#处理Json数据|处理json数据]]


#### sesssion机制

session详细了解看这个[[note1#Session的工作流程（核心）| session工作流程]]

用户使用客户端登录，服务端进行验证（比如验证用户名、密码）。

验证通过后，服务端系统就会为这次登录创建一个session。

session就是一个数据结构，保存该客户这次登录操作相关信息。通常保存在数据库中。

同时创建一个唯一的**sessionid**（就是一个字符串），标志这个session。

然后，服务端通过HTTP响应，（通过消息头Set-Cookie)把sessionid告诉客户端。

客户端在后续的HTTP请求消息头中，都要包含这个sessionid。这样服务端就会知道，这个请求对应哪个session，从而知道对应哪个客户。

##### Cookie

客户端的后续请求，是通过 HTTP的请求头 `Cookie` 告诉服务端它所持有的sessionid的。

cookie 英文就是小甜饼的意思，这里表示一小段数据。

HTTP 协议规定了， 网站服务端放HTTP响应中 消息头 `Set-Cookie` 里面的数据， 叫做 cookie 数据， 浏览器客户端 必须保存下来。

而且后续访问该网站，必须在 HTTP的请求头 `Cookie` 中携带保存的所有cookie数据

#### requests处理session-cookie

requests库给我们提供一个 `Session` 类 。

通过这个类，无需我们操心， requests库自动帮我们保存服务端返回的 cookie数据， HTTP请求自动 在消息头中放入 cookie 数据。

```python
import requests

# 打印HTTP响应消息的函数
def printResponse(response):
    print('\n\n-------- HTTP response * begin -------')
    print(response.status_code)

    for k, v in response.headers.items():
        print(f'{k}: {v}')

    print('')

    print(response.content.decode('utf8'))
    print('-------- HTTP response * end -------\n\n')


# 创建 Session 对象
s = requests.Session()

# 通过 Session 对象 发送请求
response = s.post("http://127.0.0.1/api/mgr/signin",
       data={
           'username': 'byhy',
           'password': '88888888'
       })

printResponse(response)

# 通过 Session 对象 发送请求
response = s.get("http://127.0.0.1/api/mgr/customers",
      params={
          'action'    :  'list_customer',
          'pagesize'  :  10,
          'pagenum'   :  1,
          'keywords'  :  '',
      })

printResponse(response)
```

