---
title: 使用pymysql实现数据库操作
abbrlink: 7d212608
date: 2020-06-02 21:58:29
tags: 
    - Python
    - pymysql
category:
  - Python
  - Mysql
comments: true
---

# 使用Python的pymysql实现数据库操作

pymysql是Python中用于连接MySQL的一个库，可以实现增删改查等各种操作。

## install

首先我们使用祖传的安装方法将它装好：
```
pip install PyMySQL
```

如果你的系统不支持祖传的安装方法，可以使用`git`命令把项目克隆到本地安装：
 - **step1:**克隆项目
```
git clone https://github.com/PyMySQL/PyMySQL
```
 - **step2:**进入项目目录
```
cd PyMySQL
```
 - **step3:**安装
```
python setup.py install
```

## 连接MySQL

废话不多说，直接上代码：
```python
import pymysql

# 打开数据库连接
db = pymysql.connect('host', 'username', 'password', 'db_name', 'port')

# 创建数据库游标cursor对象
cursor = db.cursor()

# 使用execute()执行sql语句
cursor.execute("SELECT something FROM some_table")

# 使用fetchone()获取单独一条数据
fetch_one = cursor.fetchone()

# 使用fetchall()获取所有数据
fetch_all = cursor.fetchall()

# 使用fetchmany()获取指定数量的数据
fetch_many = cursor.fetchmany()

# 提交修改
db.commit()

# 发生错误时回滚
db.rollback()

# 关闭数据库连接
db.close()
```
是不是很简单，这样我们就能实现各种数据库的操作了。

## 自动化测试中的数据库操作实践

我们在开发自动化时肯定会遇到需要操作数据库来完善用例操作的时候，这时就要在项目中增加一个数据库操作的功能。
经过上篇文章后，我们已经有了一个配置文件`config.yaml`，里面已经有了mysql的配置信息。
接下来我们在项目中的`common`文件夹中创建一个`mysql.py`文件，写入以下代码：
```python
import pymysql

from common.read_config import read_config  # 导入项目公共模块的配置文件读取


def get_data_with_conditions(conditions):
    sql = "SELECT something FROM some_table WHERE title=%s"
    tup_data = execute_sql(sql, conditions)
    try:
        data = tup_data[0][0]
    except IndexError:
        data = '没有查询到数据'
    except BaseException as e:
        data = f'{e},查询数据异常'

    return data


def get_data_without_conditions():
    sql = "SELECT something FROM some_table"
    tup_data = execute_sql(sql)
    try:
        data = tup_data[0][0]
    except IndexError:
        data = '没有查询到数据'
    except BaseException as e:
        data = f'{e},查询数据异常'

    return data


def execute_sql(sql, conditions=None, lib_name='数据库名称'):
    config = read_config('project_name')['mysql']
    db = pymysql.connect(
        config['host'], config['username'],
        config['password'], lib_name, config['port']
    )
    cursor = db.cursor()
    if conditions:
        cursor.execute(sql, (conditions,))
    else:
        cursor.execute(sql)
    data = cursor.fetchall()
    db.close()

    return data
```
到这里我们就基本实现了在用例中查询数据库中的数据并返回的功能，代码中的`execute_sql()`用于执行查询的sql语句，在用例中我们主要使用`get_data()`来获取数据，你也可以根据你自己的需求增加执行sql的函数再增加对应的sql语句处理操作。
