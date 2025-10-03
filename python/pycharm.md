
### 环境问题

**电脑上存在多个Python环境，而PyCharm和cmd使用的不是同一个**

**系统全局环境 (System Global Environment)**：

- 当你直接从Python官网安装Python时，会创建一个全局环境。在cmd中直接运行 `python` 或 `pip` 命令，默认操作的就是这个环境

**虚拟环境 (Virtual Environment)**：

- **PyCharm默认会为每一个新项目创建一个独立的虚拟环境**。这是一个非常好的习惯，它能让每个项目的依赖库互不干扰。这个环境的路径通常在你的项目目录下，比如 `venv`、`.venv` 或 `my_project_venv` 这样的文件夹里