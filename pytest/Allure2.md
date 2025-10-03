
- **Allure** 是一个开源的测试报告工具，能把自动化测试结果生成可视化、交互式的报告。
- **Allure2** 是 Allure 的第二代版本，也是目前主流版本（老版本 Allure1 已经过时）。
- 它本身不是测试框架，而是 **测试框架的报告生成工具**。
- 支持多语言、多平台

#### 生成测试报告流程

-  运行测试用例，生成测试结果(中间结果 JSON/XML 文件)
-  allure serve：如`allure serve ./reports/raw`
	会先自动生成 HTML 报告，再开启一个临时的本地服务（通常是 `http://127.0.0.1:port`）。
    
	- 报告会直接在浏览器中打开。
    
	- 生成的报告是 **临时的**，不会保存在你的项目目录下，关掉服务就没了。
适合场景：  
👉 本地调试时，快速查看一次性测试报告。

-   allure generate：如`allure generate ./reports/raw -o ./reports/html --clean`
	-  会把报告 **生成到指定目录**（这里是 `./reports/html`）。
    
	- 生成的报告是 **持久的**，可以提交到 Git、放到服务器上，或者后续随时用 `allure open` 打开。
适合场景：  
👉 项目里要保留报告（比如交付、存档、CI/CD 集成时）

##### 生成报告

```bash
pytest --alluredir=./reports/raw
# 后面加--clean-alluredir，可以把上次结果清理掉
```
- 运行测试时，pytest 会把执行结果（不是最终的 HTML，而是中间结果 JSON/XML 文件）存到 `./reports/raw` 目录。
    
- 这些结果文件就是后续 `allure serve` 或 `allure generate` 的数据来源

**临时查看报告**(生成在线测试报告)：
```bash
allure serve ./reports/raw
```
启动本地服务，从默认浏览器打开报告。

**生成持久报告**：
```bash
allure generate ./reports
#通过命令打开
allure open allure-report

#-c/--clean清理上一次结果（如果报告路径重复）
# -o/--output 输出报告的路径
allure generate ./reports -o
```
把最终报告生成到 `./reports/html`(可以随时随地打开报告)

打开报告：

```bash
#打开报告命令
allure open
	-h/--host 主机IP地址   #此主机将用于启动报表的web服务器
	-p/--port 主机端口     #此端口讲用于启动报表的web服务器
```


#### 添加用例标题

- 通过装饰器`@allure.title`可以为测试用例自定义标题

1. 直接使用`@allure.title`为测试用例自定义标题
2. `@allure.title`在参数化测试里，用例标题可以通过格式化字符串的方式，自动把测试函数的参数值带入标题里，可以实现测试用例标题参数化
3. `allure.dynamic.title`动态更新测试用例标题

#### 添加用例步骤

##### 方式一：`@allure.step` 装饰器

- 用来标记一个函数是「步骤」，调用时会在报告里显示。
```python
import pytest
import allure

@allure.step("输入用户名：{username}")
def input_username(username):
    print(f"输入 {username}")

@allure.step("输入密码：{password}")
def input_password(password):
    print(f"输入 {password}")

@allure.step("点击登录按钮")
def click_login():
    print("点击登录")

def test_login():
    input_username("admin")
    input_password("123456")
    click_login()

```

在报告里会显示：
```txt
输入用户名：admin
输入密码：123456
点击登录按钮
```

**特点**：
函数本身就是一个「步骤」，**每次调用它都会在报告里自动记录**
- **定义在函数上**，通用性强；
    
- 适合那些经常会复用的操作（登录、输入用户名、数据库查询等）；
    
- 调用一次，它就在报告里生成一次步骤记录

##### 方法二：使用with allure.step()

有些操作不想单独封装成函数，只在 **某个测试函数里用一下**，这时可以用 `with allure.step` 作为「上下文管理器」

```python
import pytest
import allure

def test_search():
    with allure.step("打开首页"):
        print("首页已打开")

    with allure.step("输入搜索关键字"):
        print("输入：pytest allure")

    with allure.step("点击搜索按钮"):
        print("点击按钮")

    with allure.step("检查搜索结果"):
        assert "pytest" in "pytest allure 教程"

```

在报告里会显示：

```txt
打开首页
输入搜索关键字
点击搜索按钮
检查搜索结果
```

**特点**：
- 不需要定义额外的函数，临时写就行；
    
- 适合那些 **只在单个用例里有意义的步骤**；
    
- 步骤范围由 `with` 块控制

#### 添加用例链接

在实际项目里，经常需要把 **测试用例和外部系统**（如 Jira、禅道、需求文档、接口文档）关联起来，这样在报告里就能直接跳转

##### @allure.link

最通用的方式，添加一个普通链接
```python
import allure

@allure.link("https://docs.pytest.org", name="pytest文档")
def test_with_link():
    assert True
#报告里会显示：一个超链接，标题是 pytest文档，点击就能跳转
```




##### @allure.issue

用来关联 **缺陷管理系统** 的 issue（常见 Jira、禅道等）

在展示时会带bug图标的链接

```python
import allure

@allure.issue("BUG-123", name="登录按钮失效")
def test_with_issue():
    assert False
#报告里会显示：登录按钮失效 (BUG-123)，点击就能跳转到缺陷系统
```

###### 模板机制

可以在运行时通过参数`--allure-link-pattern`指定一个模板链接，在运行时自动把 issue ID 替换进去
```bash
pytest --allure-link-pattern=issue:https://jira.company.com/browse/{} --alluredir=./results
```
这里定义了一个模板：
- 类型是 `issue`
    
- 模板链接是 `https://jira.company.com/browse/{}`

代码里只写 **ID** 就可以，不需要写完整 URL

除了命令行，还可以写到 **pytest.ini**;这样每次跑 pytest 都生效，不用手动传 `--allure-link-pattern`

##### @allure.testcase

用来关联 **测试用例管理系统** 里的用例

```python
import allure

@allure.testcase("https://my-tcms.com/case/456", name="登录功能测试用例")
def test_with_testcase():
    assert True
#报告里会显示：登录功能测试用例，点击跳转到你的用例系统
```

#### allure按需求维度分类

##### @allure.epic("需求/项目级别")

- 用来表示 **一个大的需求或项目**
    
- 例如：电商平台、后台管理系统、支付系统

```python
@allure.epic("电商平台")
def test_case():
    pass

```

##### @allure.feature## ("功能模块")

- 用来表示 **功能模块**（一级功能）
    
- 例如：用户管理、订单管理、购物车
```python
@allure.story("取消订单")
def test_case():
    pass
```

##### @allure.story("子功能/业务场景")

- 用来表示 **子功能或业务场景**（二级功能）
    
- 例如：提交订单、取消订单、支付订单
```python
@allure.story("取消订单")
def test_case():
    pass

```

##### 结合使用

```python
import allure

@allure.epic("电商平台")
@allure.feature("订单模块")
@allure.story("取消订单")
def test_cancel_order():
    assert True

```

**在 Allure 报告里的层级展示**
电商平台
 └── 订单模块
      └── 取消订单
          └── test_cancel_order

##### 总结

- **Epic** → 项目/需求大类（最高层级）
    
- **Feature** → 功能模块（一级功能）
    
- **Story** → 子功能/业务场景（二级功能）
    
- 三者结合，可以让报告和产品/需求文档保持一致，方便管理和追踪

#### 添加用例描述

##### 方式一：添加装饰器@allure.description()

```python
import pytest
import allure

@allure.description("这是一个简单的用例描述")
def test_case_1():
    assert 1 == 1

```
在报告里，这个描述会显示在「Description」区域

支持写多行字符串
```python'
@allure.description("""
前置条件：用户已登录
步骤：
  1. 进入首页
  2. 点击搜索
期望结果：显示搜索结果
""") #用"""   """来描述

```

##### 方式二：添加装饰器@allure.description_html()

用 HTML 格式描述

```python
#如果想要报告里的描述更美观（加颜色、加表格等），可以用
@allure.description_html("""
<b>前置条件：</b> 用户已登录<br>
<b>步骤：</b><br>
1. 打开首页<br>
2. 点击搜索<br>
<b>期望结果：</b> 显示搜索结果
""")
def test_case_2():
    assert 2 == 2

```



##### 方式三：直接在代码中添加文档注释

```python
def test_description_docstring():
	"""
	直接在测试用例方法中
	通过编写文档注释
	添加描述
	:return:
	"""
	#文档信息放在方法里面，会按照给定的格式展示
	assert true
```

##### 方式三：动态修改描述

有时候希望在测试函数运行时，动态添加说明，可以用：
```python
def test_case_3():
    allure.dynamic.description("这是运行时动态添加的描述")

```

#### 添加用例优先级

在 **Allure 2** 里，给用例添加「优先级」主要是通过 **严重级别（Severity）** 来体现的。

#####  @allure.severity 设置优先级

```python
import pytest
import allure

@allure.severity(allure.severity_level.CRITICAL)
def test_high_priority():
    assert 1 == 1

```

可以通过命令行筛选运行特定优先级的用例：

```bash
pytest --allure-severities=critical,blocker
```

类中没有添加级别的方法，按类上添加的级别来
###### Allure 内置的优先级等级：

- **BLOCKER** → 阻塞缺陷（最高优先级，必须立刻修复）(程序无响应，无法执行下一步)
    
- **CRITICAL** → 严重缺陷（主要功能点无法实现）
    
- **NORMAL** → 一般缺陷（影响部分功能，存在替代方案）(数值计算错误等)
    
- **MINOR** → 次要缺陷（界面ui错误、偶发错误）
    
- **TRIVIAL** →轻微缺陷， 不影响使用的小问题（最低优先级）(必输项无提示或提示不规范等)

##### 动态设置优先级

如果在运行时想改，可以用：
```python
def test_dynamic_priority():
    allure.dynamic.severity(allure.severity_level.BLOCKER)
    assert True

```

#### 添加测试用例标签

`xfail、skip`严格来说属于 **pytest 提供的标记，但 Allure 也能识别并在报告里展示。


```python
#跳过测试用例，不执行
import pytest
import allure

@allure.feature("用户登录")
@pytest.mark.skip(reason="功能未实现")
def test_login_skip():
    assert 1 == 1

```
效果：
- 用例不会执行
    
- Allure 报告里显示为 **跳过（Skipped）**，并显示 `reason`

#### 记录失败重试

失败重试的功能要依赖 pytest 插件 `pytest-rerunfailures`，然后 Allure 会把多次执行的结果展示出来。

重试的结果信息，会展实在详情页面的“Retries"选项卡中

先安装插件`pip install pytest-rerunfailures`

使用装饰器`@pytest.mark.flaky()`
针对单个用例设置重试次数：
```python
import pytest
import allure

@allure.feature("登录功能")
@pytest.mark.flaky(reruns=2, reruns_delay=1)   # 失败后最多重试 2 次，间隔 1 秒
def test_login():
    assert 1 == 2   # 故意失败

```
- `reruns=2` → 失败后重试 2 次
    
- `reruns_delay=1` → 每次重试间隔 1 秒

###### 在 Allure 报告中的表现

- 如果用例 **第一次失败 → 第二次通过**，报告里会显示 **“重试历史”**，能看到之前失败的截图、日志等。
    
- 如果 **多次重试后还是失败** → 报告中最终状态还是 **失败**，但你可以点开看中间的重试记录。

#### 添加附件

##### allure.attach.file()

- 用于附加 **现有文件**（图片、日志文件、报告片段等）
    
- 内部会把文件读入并保存到 Allure 的结果目录

```python
def test_attach_file():
    allure.attach.file("screenshot.png",
                       name="失败截图",
                       attachment_type=allure.attachment_type.PNG)
#传入文件路径
```

##### allure.attach()

- 直接把 **内容（字符串/二进制）** 作为参数传进去
    
- 适合动态生成的数据（如接口响应、日志、HTML 内容）
```python
import allure

def test_attach_text():
    allure.attach("接口响应：success",
                  name="接口返回",
                  attachment_type=allure.attachment_type.TEXT)

def test_attach_json():
    data = '{"code":200, "msg":"ok"}'
    allure.attach(data,
                  name="JSON数据",
                  attachment_type=allure.attachment_type.JSON)

```
这里数据是 **直接传递的**。

#### 添加日志

##### 结合 Python logging

可以在 logging 里写日志，同时附加到 Allure：
```python
import logging
import allure

logger = logging.getLogger(__name__)

def test_logging_in_allure():
    logger.info("开始执行测试用例")
    logger.error("执行失败，错误码 500")
    
    # 在测试结束时把日志写入 Allure
    with open("logs/test.log", "r", encoding="utf-8") as f:
        allure.attach(f.read(), name="测试日志", attachment_type=allure.attachment_type.TEXT)

```

日志展示在Test body标签下，标签下可展示多个子标签代表不同的日志输出渠道：
- log子标签：展示日志信息
- stdout子标签：展示print信息
- stderr子标签：展示终端输出的信息

**禁用日志**，可以使用命令行参数控制：`allure-no-capture`
##### 总结

- **临时字符串日志** → 用 `allure.attach("日志内容", ...)`
    
- **已有日志文件** → 用 `allure.attach.file("path", ...)`
    
- **企业常见做法**：测试框架里统一用 `logging` 写日志文件 → 测试完成后再 attach 到 Allure