
### 测试数据的数据驱动

数据驱动是一种 **测试设计思想**，核心是：

👉 **用同一份测试逻辑，驱动多组不同的数据，自动生成和执行多条测试用例**

**核心思想**：**分离测试逻辑和测试数据**，让逻辑可以复用，数据可以不断扩展

👉 数据驱动实现方式用：

- **参数化**
	
- **外部数据文件**（CSV、JSON、YAML、Excel）
    
- **数据库数据**
    
- **接口返回值**

除了测试数据的数据驱动，数据驱动还应用在：测试步骤的数据驱动、配置的数据驱动
#### yaml文件

是一种 **轻量级的数据序列化格式**

- 对象：键值对的集合
- 列表：一组按次序排列的值，前面加“-”
- 纯量：最基本的单值，不可再分
		常见纯量：字符串、数字、布尔值、空值、时间、日期

##### 文件使用

- 查看yaml文件
	文本编辑器
	pycharm
- 读取yaml文件
	- 安装库：`pip install pyyaml`
	- 读取方法(yaml->python对象)：`yaml.safe_load(f)`
			如果yaml文件中有中文，必须要加上`encoding="utf-8"`
	- 写回方法(python对象->yaml)：`yaml.safe_dump(f)`

```python
#可以封装成工具函数，用时直接调用
def load_yaml(path):
    with open(path, "r", encoding="utf-8") as f:
        return yaml.safe_load(f)
```

#### Excel文件

##### 常用库

- **openpyxl**  最常用
- **xlrd**：老库，只支持 `.xls`（Excel 2003 格式），新版不支持 `.xlsx`
- **pandas**

##### openpyxl

安装：`pip install openpyxl`

```python
import openpyxl
#获取工作薄
book=ipenpyxl.load_workbook(路径)

#读取工作表
sheet=book.actiove

#读取单个单元格
cell_a1=sheet["A1"]
cell_a3=sheet.cell(column=1,row=3)

#读取多个连续单元格
cells=sheet["A1":"C3"]

#获取单元格的值
cell_a1.value
```

#### csv文件

- **CSV (Comma-Separated Values)**：逗号分隔值文件，本质是一个**纯文本表格文件**。
    
- 每一行代表一条记录，每列之间用逗号（或制表符、分号等）分隔。
    
- 常用来存储和交换表格数据，几乎所有编程语言都支持。

示例（`login_data.csv`）：很像 Excel，但更轻量、更通用
```csv
username,password,expected
admin,123456,success
guest,guest123,success
test,wrong,fail
```

##### 读取数据

- 内置函数：`open()`
- 内置模块：`csv`
- 方法：
	- `csv.reader()` **返回的是列表（list）**，每一行就是一个列表，不会带[[note2#^6edbd7|字段名]](不会自动识别字段名，它把第一行也当成普通数据)
	- `csv.DictReader()` - **返回的是字典（dict）**，会自动把第一行当作字段名。
     每一行就是一个字典，更直观

```python
# reader()
import csv

with open("login_data.csv", encoding="utf-8") as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)

#DictReader()
import csv

with open("login_data.csv", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row)

```

#### json文件

##### 读取文件

- 内置函数`open()`
- 内置库json
- 方法：
	- `json.loads()`  
	- `json.dumps()`  把 Python 对象转成 JSON 格式的字符串（注意是字符串，不是文件）



```python
import json

with open("login_data.json", encoding="utf-8") as f:
    data = json.load(f)

print(data)
#输出（Python 会自动转成列表+字典）
```