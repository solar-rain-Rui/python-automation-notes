
**pytest** 是一个用于编写和运行测试的 Python 框架。

- pytest能够支持简单的单元测试和复杂的功能测试
- pytest可以结合Requests实现接口测试；结合Selenium、Appium实现自动化功能测试
- 使用pytest结合ALlure集成到Jenkins中可以实现持续集成
- pytest支持315种以上的插件，兼容unittest，定制化插件开发


#### 编写测试用例

##### 格式要求

- 测试文件命名以 `test_` 开头（如 `test_math.py`），或者以`_test`结尾。 （test_开头更常见）
- 类名以 `Test_` 开头。
- 测试函数/方法以 `test_` 开头
- 使用 `assert` 进行断言。

注：测试类中不可添加`__init__`构造函数

##### 用例结构

三部分构成：
- 用例名称
- 用例步骤
- 用例断言

```python
#类级别的用例实例
class TestXXX:
	def setup(self):
	#资源准备
	pass
	
	def teardown(self):
	#资源销毁
	pass
	
	def test_XXX(self):
	#测试步骤1
	#测试步骤2
	#断言 实际结果对比预期结果
	assert ActualResult == ExpectedResult
```

##### 测试用例断言

断言(assert)：为了验证预期的结果
当程序执行到断言位置时，对应的断言如果不为真，程序会终止执行局，并给出错误信息

断言写法：
- assert<表达式>
- assert<表达式>,<描述>


测试项：
-  test为前缀 的 **函数**
- Test为前缀的**类**里面的 test为前缀的**方法**

##### pytest 如何知道你哪些代码是自动化的测试用例？

pytest 默认会递归地查找**当前目录及其所有子目录**下符合以下命名规则的文件：

- `test_*.py` （以 `test_` 开头的文件）
- `*_test.py` （以 `_test` 结尾的文件）

在符合命名规则的文件里，pytest 会进一步查找符合以下规则的函数和类：

- **测试函数**：所有以 `test_` 开头的函数。
- **测试类**：所有以 `Test` 开头的类。
		并且，在这个类中，**所有以 `test_` 开头的方法**都会被识别为测试用例。不以 `test_` 开头的方法被视为普通方法，不会被自动执行。

**查看**它发现了哪些测试用例，而不真正运行它们的命令：
`pytest --collect-only`
##### 测试

1. 测试安装
`pip install pytest`
`pip install pytest-html`      安装第三方插件用来产生测试报表
2. 命令行终端进入项目的根目录，执行命令`pytest`
3. 由于pytest会截获print打印的内容，如果要显示加上命令行参数`pytest -s`,如果要更详细的信息，包括每个测试类、测试函数名字，可以加参数 -v`pytest -sv`
4. 产生测试报告
```bash
pytest cases --html=report.html --self-contained-html
#产生名为rport.html的测试报告文件，可以在浏览器中打开
```

#### 测试框架结构

全局模块级：setup_module/teardown_module

类级，只在类中前后运行一次：setup_class/teardown_class

函数级，在类外：setup_function/teardown_function

方法级，类中的每个方法执行前后：setup_method/teardown_method（兼容unnitest的setup和teardown)

在类中，运行中调用方法的前后(重点)：setup/teardown


##### 模块级别

模块级别的初始化、清除 分别 在整个模块的测试用例 执行**前后执行**，并且 **只会执行1次**

模块级别的初始化、清除 在 整个模块所有用例 执行前后 分别 执行1次

它主要是用来为该 模块 中 所有的测试用例做 **公共的**初始化 和 清除

##### 类级别

类级别的初始化、清除 分别 在整个类的测试用例 执行前后执行，并且 只会执行1次

级别的初始化、清除 在 整个类 所有用例 执行前后 **分别 执行1次** 。

它主要是用来为该 类中的所有测试用例做 公共的 初始化 和 清除

##### 方法级别

方法级别 的初始化、清除 分别 在类的 **每个测试方法 执行前后执行**，并且 **每个用例分别执行1次**

#### 参数化

- 通过参数的方式传递数据，从而实现**数据和脚本分离**
- 并且可以实现用例的重复生成与执行 

**实现方法**：

装饰器：`@pytest.mark.parametrize`

- **测试数据源**（列表/字典/函数）
    
- 通过 **`@pytest.mark.parametrize` 装饰器** 传递给测试函数
    
- **测试函数** 被多次执行（每组参数执行一次）
    
- 生成 **多个测试用例**
    
- 最后在 **测试报告** 中分别展示结果

##### 单参数情况

单参数，可以把数据放在列表中

每一条测试数据都会生成一条测试用例

```python
#语法
@pytest.mark.parametrize("参数名", [参数值1, 参数值2, ...])
def test_函数名(参数名):
    # 测试逻辑
    assert 参数名 条件


#示例
import pytest  
  
search_list=["appium","selenium","pytest"]  
#参数化实现测试用例的动态生成  
#单参数情况  
@pytest.mark.parametrize("search_key", ["appium","selenium","pytest"," ","requests","abc"])  #参数和对应的值
def test_search_param(search_key):  
    assert search_key in search_list
```

#####  多参数情况

- 将数据放在列表嵌套元组中
- 将数据放在列表嵌套列表中

```python
#多参数情况示例  
@pytest.mark.parametrize("username,password",[["rightusername","rightpassword"],["wrongusername","wrongpassword"],
[" ","password"]])  
def test_login(username,password):  
    print(f"登录用户名：{username},密码：{password}")
```

##### 用例重命名-添加ids参数

通过ids参数，将别名放在列表中

ids列表参数的个数要与参数值的个数一致

```python

@pytest.mark.parametrize("username,password",[["rightusername","rightpassword"], ["wrongusername","wrongpassword"],  [" ","password"]],  
ids=["right username and password","wrong username and password","username is Null"])  
def test_login(username,password):  
    print(f"登录用户名：{username},密码：{password}")
```

###### ids中文

pytest 默认会把中文转成 `\u4e2d\u6587` 这样的 Unicode 转义

在项目下创建一个conftest.py文件，加入如下代码
```python
def pytest_collection_modifyitems(config, items):
#pytest 提供的一个 hook，在 收集完所有测试用例之后执行;
#items就是 pytest 收集到的测试用例对象列表
    for i in items:
        i.name=i.name.encode("utf-8").decode("unicode-escape")
        #encode把原始字符串转成字节流;decode把\uXXXX这样的转义序列重新解析为真正的中文
        i._nodeid=i.nodeid.encode("utf-8").decode("unicode-escape")
        
#i.name → 单个用例的名字    
#i.nodeid → 测试用例的唯一标识（包括参数化 id、文件路径等），pytest 用它来区分不同测试用例
```

##### 笛卡尔积(嵌套参数化)

在 `pytest.mark.parametrize` 里，可以对不同的参数 **分别定义参数化集合**，pytest 会自动生成所有可能的 **笛卡尔积组合**

pytest **从下往上** 组合参数

#### 标记测试用例

标记就是 **给测试函数贴标签**，方便在执行时进行选择

在 pytest 里用装饰器 `@pytest.mark.xxx` 来实现

执行：`-m`执行自定义标记的相关用例

```python
import pytest

@pytest.mark.smoke
def test_login():
    assert 1 == 1

@pytest.mark.regression
def test_register():
    assert 2 == 2
#这里就有两个标记：smoke（冒烟测试）、regression（回归测试）
```

##### 自定义标记

需要在 `pytest.ini` 里注册自定义的标记

```ini
# pytest.ini
[pytest]
markers =
    smoke: 冒烟测试
    regression: 回归测试
    slow: 慢速测试

```

这样执行时就不会警告

##### 内置标签

可以处理一些特殊的测试用例，不能成功的测试用例

- `skip` 始终跳过该测试用例，参数可以加原因
		代码中也可以添加跳过代码`pytest.skip(reason)`
- `skipif` 遇到特定情况跳过该测试用例
- `xfail` 遇到特定情况，产生一个“期望失败”输出；通过XPASS

#### 运行多条用例

- 执行包下所有用例：`pytest/py.test   [包名]`
- 执行单独一个pytest模块：pytest  文件名.py
- 运行某个模块里面某个类：pytest 文件名.py：：类名
- 运行某个模块里面某个类里面的方法：pytest 文件名.py::类名：：方法名

#### 常用命令行参数

- `-x` 用例一旦失败(出现第一个失败)，就立刻停止执行 
- `--maxfail=num` 最大失败数达到某个值时，立刻停止执行
- `-m` 标记用例
- `-k "关键字"` 执行包含某个关键字的用例
-  `--lf(--last-failed)` 只运行上一次执行失败的用例
- `--ff(--failed-first)` 先运行上一次失败的用例，之后再运行其他用例
- `-v` 详细模式，显示完整用例路径(一般-vs一起使用)
- `-s` 显示 print打印输出 和日志输出（不捕获）
- `--collect-only` 只收集（collect）测试用例，不执行

#### python代码执行pytest

##### 使用main函数

`if __name__ == "__main__":`参考[[模块、python包#^d4df5e|__name__变量解释]]

 ```python
 import pytest

if __name__ == "__main__":
    # 运行当前目录下所有测试
    pytest.main()

#也可以传递参数

import pytest

if __name__ == "__main__":
    # 等价于 pytest -v test_demo.py
    pytest.main(["-v", "test_demo.py"]) #参数作为字符串放到列表里

#在命令行代码执行
python test_*.py
 ```

##### 使用`python -m pytest`调用pytest

理由：
1.  有时直接用 `pytest` 会因为环境变量或 PATH 配置不对而找不到命令。
用 `python -m pytest` 就能确保调用的是当前 Python 环境里的 pytest

2. 多版本 Python 并存

- 如果你同时安装了 Python 3.9 和 Python 3.11，可能 `pytest` 默认走错环境。
- 这时用：
```bash
python3.9 -m pytest
python3.11 -m pytest
```

就能明确指定用哪个 Python 来跑测试

#### 异常处理方法

##### try...except

```python
try:
    x = 1 / 0
except ZeroDivisionError as e:#只捕获除零错误,把异常对象存到变量e
    print("异常类型:", type(e))
    print("异常信息:", e)

```

- `except` 后面必须写 **异常类型（类）**，不能随便写普通变量。
    
- 常见写法：
    
    - `except ValueError:`
        
    - `except (ValueError, TypeError):`
        
    - `except Exception as e:`（兜底）
        
- `as e` 👉 给异常对象起名字，方便使用

##### pytest.raises

这是 pytest 提供的一个 **断言异常的工具**，用于验证某段代码是否会抛出指定异常

```python
import pytest

def test_zero_division():
    with pytest.raises(ZeroDivisionError):
        1 / 0
```

- 如果 `1/0` 没有抛出 `ZeroDivisionError`，这个用例就会 **失败**。
    
- 如果确实抛出了 `ZeroDivisionError`，这个用例就会 **通过**

可以用 `.value` 取到异常对象：
```python
def test_value_error():
    with pytest.raises(ValueError) as excinfo:
        int("abc")

    assert "invalid literal" in str(excinfo.value)  # 校验异常信息

```

`excinfo.value` 就是异常对象（等价于 `except ValueError as e` 里的 `e`)