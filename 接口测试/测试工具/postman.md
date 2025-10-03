API平台

可以创建API、测试API、管理API以及对API进行协作

Authentication=who you are 身份验证

Authorization = what you can do 被允许做什么（授权）

scripts:
- Pre-request 预请求脚本     在实际请求之前执行
- Post-response 收到响应后执行

#### 页面

- History  接口请求历史记录
- Collections：可根据不同的模块或项目，自定义保存接口请求集合
- APIs：帮助用户直接在Postman设计和开发其API来改进API开发
- Params：请求参数设置
- Authorizatin：权限
- Headers：头信息，有隐藏的头信息(postman自动加的，可以自己勾掉重新添加来修改)
- Body：传数据（post)
- Pre-request Script：前置脚本
- Tests：断言 (现版本整合到scripts下的Post-response中)


**添加api请求**
1. 选择http
2. 填入api地址(url)
3. 添加标头、授权等等
4. 对于Post、Put请求需要加正文消息体(body)
5. 检查响应

#### variables(变量)

键值对
##### postman中变量的种类和作用域

- Data：在测试集中上传的数据
- Environment：环境范围
- Collection：集合范围
- Global：全局范围
- Local：在脚本中设置的变量

变量的使用：
- 在请求url、Params参数或Body表格或JSON/XML文本中通过`{{变量名}}`使用
- 在Pre-request Script和Tests脚本中使用封装好的语句获取或设置对应变量

##### 全局变量(Globals)

在**整个 Postman 应用内全局生效**，所有集合、请求、脚本都能访问

现版本界面变动(2025.09):
- Value” 输入框，就是原来版本的 “Initial Value”；这里输入的值，就是变量的初始值
- **“Current Value”** 这个界面元素**已经被移除**。变量的当前值由脚本动态控制
	 在新版设计中，不再有独立的 “Current Value” 输入框;变量的 “当前值” 实际上就是您在脚本中 通过 `pm.globals.set` 方法所设置的最新值
- "Type"选项移除，用mark as Sensitive
- “Persist All” 按钮-：在新版Postman中，**变量的“当前值”会自动持久化**。
	当您在脚本中使用 `pm.collectionVariables.set(...)` 或 `pm.globals.set(...)` 修改了一个变量的值，并且成功发送了请求，这个新值就会被自动保存下来，并在您下次打开Postman时保持不变。
    这意味着“Persist All”的功能已经变成了默认的自动化行为，不再需要手动点击
- “Reset All” 按钮也被移除，现在您需要**逐个变量**进行重置
	在面板中修改成自己想要的值
	由于“当前值”是自动持久化的，直接修改“Value”并保存，就相当于同时修改了它的“初始值”和“当前值”。


在脚本中设置、获取全局变量
```javascript
pm.globals.set("status","sold");

var s=pm.globals.get("status");//获取全局变量

console.log(s) //打印变量

```

##### 环境变量

**environment**

区分不同的服务器环境，比如开发环境、测试环境、生产环境
Postman 中可以创建多个环境，每个环境是一个独立的变量集合
只需切换 “环境”，就能让同一套测试用例在不同环境中运行，避免重复配置。

作用范围：只在**当前激活的环境中**生效

**Local**

脚本中写的局部变量

**优先级**

在不同级别或范围创建了具有相同名称的变量，优先级从高到低如下：
- Data
- Environment
- Collection
- Global
- Local

##### 通过脚本获取or设置变量

```javascript
pm.globals.set("name", "Rutherford");

pm.collectionVariables.set("name", "Ramanujan");

//pm.environment.set("name", "Einstein");

pm.variables.set("name1", "Newton");

var s=pm.globals.get("name");//获取变量

console.log("Hello,World!!!!");

let val=pm.globals.get("name");
//打印变量
console.log(val);
console.log(pm.collectionVariables.get("name")) ;
console.log(pm.environment.get("name"));
console.log(pm.variables.get("name"));
```

#### 断言

- 验证响应状态码
- 验证响应体中是否包含某个字符串
- 验证Json中的某个值是否等于预期的值
- 验证响应体是否与某个字符串完全相同
- 验证响应头信息中的Content-Type是否存在
- 验证响应时间是否小于某个值

```javascript
//Status Code:Code is 200
//验证响应状态码
pm.test("响应预期状态码为200",function(){pm.response.to.have.status(200);
});

//Response Body:contains string
//验证响应体中是否包含某个字符串
pm.test("响应体中包含预期的字符串",function(){
	pm.expect(pm.response.text()).to.include("doggie");
});

//Response Body:JSON value check
//验证JSON中的某个值是否等于预期的值
pm.test("宠物名称为 doggie",function(){
	var jsonData=pm.response.json();
	pm.expect(jsonData[0].name).to.eql("doggie");
});

//Response Body:Is equal to a string
//验证响应体是否与某个字符串完全相同
pm.test("响应体正确",function(){
	pm.response.to.have.body("response_body_string");
});

//Response Body:Content-Type header check
//验证响应头信息中的Content-Type是否存在
pm.test("Content-Type is present",function(){
	pm.response.to.have.header("Content-Type");
});

//Response time is less than 200ms
//验证响应时间是否小于某个值
pm.test("Response time is less than 200ms",function(){
	pm.expect(pm.response.responseTime).to.be.below(200);
});

```

#### 运行测试集

点击Run

- 选择运行接口，可以调整上下顺序
- iterations:设置迭代次数
- Delay:设置延迟时间
- Keep variable values:保持变量值
- Save cookies after coolection run:执行后保存cookies
