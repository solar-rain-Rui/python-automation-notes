
#### 字符串与字节串

^01

- 字节串(bytes)：是计算机存储和传输的**原始字节**。比如：`b'\xe4\xbd\xa0\xe5\xa5\xbd'`，`b'Hello'`。
- 字符串：**(`str`)**：是人类可读的**文本**。比如："你好"，"Hello"。

字符串和字节串之间的桥梁是**字符编码**（如 UTF-8, ASCII, GBK）。

- **编码 (Encode)**：将 **字符串 (`str`)** 转换为 **字节串 (`bytes`)** 的过程。
    
    - **为什么？** 因为计算机只能存储和处理字节。你要把文本存到文件里或者通过网络发送，就必须把它变成字节。
        
    - **方法：** `str.encode(encoding='utf-8')`
        
- **解码 (Decode)**：将 **字节串 (`bytes`)** 转换回 **字符串 (`str`)** 的过程。
    
    - **为什么？** 当你从文件或网络收到一串字节时，你需要知道它原本是用什么编码的文本，才能正确地把它解释回人类可读的文字。
        
    - **方法：** `bytes.decode(encoding='utf-8')`
        
- 从 `str` 到 `bytes` 叫 **编码** (`encode`)。
    
- 从 `bytes` 到 `str` 叫 **解码** (`decode`)。
**这个过程是绝对不能出错的！** 用错误的编码方式去解码字节串，就会产生乱码

#### 处理Json数据

^e60c47

##### json.dumps()

**`json.dumps()`** 就像 **打包**。你把一个Python对象（比如字典、列表）**打包（序列化）** 成一个JSON格式的**字符串**，方便传输或存储。

`dumps` 的全称是 "dump string"

**作用**：将 Python 对象（如 `dict`, `list`, `tuple`, `str`, `int`, `float`, `bool`, `None`）编码（序列化）为一个 **JSON 格式的字符串**。
###### 常用参数：

- `ensure_ascii=False`: 这是处理**中文**的关键参数。默认是 `True`，会将所有非ASCII字符（如中文）转义为 `\uXXXX` 的格式。设置为 `False` 后，输出才会是正常的中文。
    
- `indent=4`: 让生成的JSON字符串变得格式化、美化，有缩进和换行，非常便于阅读。数字表示缩进的空格数。
    
- `sort_keys=True`: 将输出的字典按键（key）的字母顺序排序。

**主要用途**：在发送HTTP请求（例如使用 `requests` 库）时，你需要将请求体（通常是字典）转换成JSON字符串。

```python
import requests

payload = {"key": "value"}
headers = {'Content-Type': 'application/json'}

# 必须用 dumps 将字典转为字符串才能发送
response = requests.post('https://api.example.com/data', data=json.dumps(payload), headers=headers)

# 更简单的写法（requests库会自动帮你做dumps并设置header）:
# response = requests.post('https://api.example.com/data', json=payload)
```
##### json.loads()

**`json.loads()`** 就像 **拆包**。你收到一个JSON格式的**字符串**，把它**拆包（反序列化）** 回一个Python对象，方便在代码里操作。
`loads` 的全称是 "load string"。

**作用**：将一个 **JSON 格式的字符串** 解码（反序列化）为一个 **Python 对象**（通常是字典或列表）。

**主要用途**：处理从网络API接收到的响应（HTTP Response），这些响应通常是JSON字符串，你需要把它们转换成Python字典来提取数据

```python
import requests

response = requests.get('https://api.example.com/data')
# response.text 是字符串形式的响应内容
json_response = response.text 
print(type(json_response)) # <class 'str'>

# 将字符串响应解析为Python字典
data_dict = json.loads(json_response)
# 更简单的写法（requests库提供了直接解析的方法）:
# data_dict = response.json()

# 现在可以方便地提取数据了
print(data_dict['some_key'])
```

#### Session

##### 为什么需要Session机制？—— HTTP协议的无状态性

HTTP协议本身是“无状态”的。这意味着服务器处理完一个请求后，不会记得是谁发来的请求。

##### Session的工作流程（核心）

这个过程完美诠释了Cookie和Session如何协同工作：

![session工作流程|400](session.png)

1. **Session数据存储在服务器端**，更安全，可以存储大量数据。
    
2. **客户端只保存一个Session ID**，通常通过Cookie传递。
    
3. 每次请求，浏览器都会**自动**将Cookie（包含Session ID）发送给服务器，无需手动处理。

##### cookie和session对比

|特性|Cookie|Session|
|---|---|---|
|**存储位置**|**客户端**（浏览器）|**服务器端**|
|**安全性**|较低，用户可见可修改|较高，仅存在于服务器|
|**数据类型**|只能存字符串|可存任意数据类型（对象、列表等）|
|**生命周期**|可设置长期有效|通常用户关闭浏览器或超时即失效|
|**容量限制**|有大小和数量限制|理论上无限制（受服务器资源影响）|


##### 在自动化测试中的应用(requests库)

在自动化测试中，我们经常需要模拟一个登录后的用户进行一系列操作。使用Session对象可以让你轻松管理会话状态

```python
import requests

# 创建一个Session对象（相当于打开一个浏览器Tab页）
s = requests.Session()

# 1. 使用这个Session进行登录
login_url = 'https://example.com/login'
login_data = {'username': 'test', 'password': '123456'}
s.post(login_url, data=login_data)  # 这个Session对象s会自动保存服务器返回的Cookie（Session ID）

# 2. 使用同一个Session发送后续请求
# Session对象s会自动在请求中带上登录后得到的Cookie
user_info_response = s.get('https://example.com/userinfo')
print(user_info_response.text) # 成功返回用户信息

order_list_response = s.get('https://example.com/orders')
# 可以继续用同一个s做各种操作，服务器会一直认为你是登录状态

# 3. 最后可以关闭Session (虽然不是必须的)
s.close()
```



