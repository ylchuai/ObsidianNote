# Flask 基础

## Flask环境搭建

### 1. Python 虚拟环境

作用：类似node_module 搭建开发用环境
创建一个虚拟环境

```bash
python3 -m venv {路径}
```

激活虚拟环境

在 Microsoft Windows 上，为了启用 `Activate.ps1` 脚本，可能需要修改用户的执行策略。可以运行以下 PowerShell 命令来执行此操作：

```bash
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

```bash
# <windows>cmd -> bat powershell -> psl
<venv-path>\Scripts\activate.*
# <linux>
<venv-path>/bin/activate.*
```

### 2. Flask安装

```bash
# 激活虚拟环境后
pip install flask
```

### 3. Flask相关依赖

```bash
# 激活虚拟环境后
pip list

Package      Version
------------ -------
# 创建命令行相关
click        8.0.3
# 可以跨多终端，显示字体不同的颜色和背景
colorama     0.4.4
# 本体
Flask        2.0.2
# 数据加密签名
itsdangerous 2.0.1
# 模板处理
Jinja2       3.0.3
# 防止注入攻击
MarkupSafe   2.0.1
# ??So, why are you here?
pip          21.2.3
# 打包工具
setuptools   57.4.0
# WSGI工具包 Flask 构建基础
Werkzeug     2.0.2
```

### 4. 如何在linux后台启动Flask

bash.sh 启动命令
xxx.log 记录日志

```bash
nohup <bash.sh> > xxx.log 2>&1 &
```

## 开发环境

### 1. 目录结构

```text
----main
|-templates
|  |-- xxx.html
|
|-static
|  |--js
|  |--css
|  |--img
|
|- app.py
```

### 2. 开发环境运行

-p 监听端口
--reload  开启文件监听
--debugger 开启debug模式

```bash
flask run -p 19090 --reload --debugger
```

### 3. 获取项目依赖

```shell
pip install pipreqs

pipreqs .

# INFO: Successfully saved requirements file in .\requirements.txt
```

requirements.txt

```text
Flask==2.0.2
```

### 4.离线包的下载与使用

下载

```shell
pip download -d ./require_pack -r requirements.txt
```

使用

```shell
pip install --no-index --find-links=./require_pack -r requirements.txt
```
