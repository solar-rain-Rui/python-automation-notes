
在 pytest 里，它是一种 **测试前置/后置的机制**，用来在测试运行前准备环境，在测试结束后清理环境

用 `@pytest.fixture` 装饰一个函数，这个函数就变成了一个 fixture，可以在测试用例中 **直接当作参数注入**

特点：
- 写法命名更灵活
- 数据共享：在`conftest.py`配置里写方法可以实现数据共享，不需要import导入，可以跨文件共享
- fixture的作用域scope可控
- 支持参数化，适合数据驱动

#### 基本用法

```python
import pytest

# 定义 fixture
@pytest.fixture
def db_conn():
    print("连接数据库")
    yield "db_connection"
    print("关闭数据库")

# 使用 fixture
def test_case1(db_conn):
    print("测试用例1，用到:", db_conn)
    assert db_conn == "db_connection"

def test_case2(db_conn):
    print("测试用例2，用到:", db_conn)
    assert isinstance(db_conn, str)

```

执行结果为：
```makefile
连接数据库
测试用例1，用到: db_connection
关闭数据库
连接数据库
测试用例2，用到: db_connection
关闭数据库
```

#### 作用域

默认是函数(function)级别

|scope 值|说明|执行次数|
|---|---|---|
|`"function"`|默认值，每个测试函数都会执行一次|每个用例执行一次|
|`"class"`|每个测试类执行一次，类里多个方法共享|每个类执行一次|
|`"module"`|每个测试模块执行一次，模块里所有用例共享|每个文件执行一次|
|`"session"`|整个测试会话执行一次，所有用例共享|整个 pytest 运行过程只执行一次|
- `session_fixture` → 全程 1 次
    
- `module_fixture` → 每个 py 文件 1 次
    
- `class_fixture` → 每个测试类 1 次
    
- `func_fixture` → 每条用例 1 次

#### yield关键字

普通函数用 `return` 返回值后，函数执行就结束了；  
而 **带 `yield` 的函数** → 会变成一个 **生成器函数**（generator function）

👉 **区别**：
- `return`：一次性返回结果，函数结束。
    
- `yield`：返回一个值，但函数不会结束，而是 **暂停在这里**，等待下一次调用继续往下执行。

在 **pytest fixture** 里，`yield` 用来 **分隔“前置”和“后置”操作**。

- `yield` 前面的部分 = setup（前置）
    
- `yield` 返回的值 = fixture 提供给测试用例的数据/对象
    
- `yield` 后面的部分 = teardown（后置清理）

```python
import pytest

@pytest.fixture
def resource():
    print("\n前置：打开数据库连接")
    yield "db_connection"   # 这里返回给测试用例
    #fixture 执行到这里会暂停，把"db_connection"交给用例
    print("后置：关闭数据库连接") #用例执行完成后，pytest 再回来继续执行yield后面的清理代码

def test_case(resource):
    print("测试用例执行，使用 ->", resource)

```

执行结果：
```txt
前置：打开数据库连接
测试用例执行，使用 -> db_connection
后置：关闭数据库连接
```

`test_case(resource)` 里的参数 `resource` 能等于 `"db_connection"`，是因为 pytest 在执行时会自动调用同名的 fixture，把 `yield` 的值作为参数传进去



`yield` 在 fixture 里的作用：

- 保证无论测试是否通过，`yield` 后面的清理动作都会执行。
    
- 相当于把 `setup` 和 `teardown` 合并到了一个函数里。

`yield`的好处：
- **节省内存**：不需要一次性生成全部数据，按需返回。
    
- **提高灵活性**：可以在测试中优雅地拆分前置和后置。
    
- **避免遗漏清理**：pytest 会自动确保 `yield` 后的清理逻辑执行

#### 数据共享

使用`conftest.py`这个文件进行数据共享

- pytest 会**自动识别并加载**当前目录及其子目录下的 `conftest.py` 文件。
    
- 它里面定义的 **fixture、hook 函数** 不需要 `import`，就能在用例里直接用。
    
- 用来做 **数据共享、公共配置、全局钩子** 最合适

#### 自动应用

使用fixture中参数`autouse=True`实现

在方法上面加`@pytest.fixture(autouse=True)`

#### Fixture参数化

**让同一个测试用例，用不同的输入数据运行多次**，而且测试用例本身不用改

```python
import pytest

# 定义一个参数化的 fixture
@pytest.fixture(params=[1, 2, 3])
def number(request):#要通过request拿到值
    return request.param   # 每次调用 fixture 时，取一个参数值

def test_number(number):
    print(f"\n运行测试，number = {number}")
    assert number > 0

```

 **执行结果**

- `test_number` 会运行 3 次：分别拿到 `1`、`2`、`3`。
    
- `request.param` 就是 fixture 参数化时传进来的值。

##### 与普通参数化对比

|特点|普通参数化|fixture 参数化|
|---|---|---|
|写法位置|**函数装饰器**|**fixture 装饰器**|
|参数来源|`@pytest.mark.parametrize` 里的列表|`fixture(params=...)` 里的列表|
|数据传递方式|直接作为函数参数|通过 `request.param` 传递|
|影响范围|仅当前函数|所有依赖该 fixture 的函数|
|典型场景|单个函数不同输入|环境准备、外部数据驱动、复用性高的场景|
