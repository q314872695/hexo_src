---
title: 使用python连接mysql数据库——pymysql模块的使用
tags: python
abbrlink: '62279014'
date: 2019-09-01 15:24:00
---

# 安装pymysql

```python
pip install pymysql
```
# 使用pymysql
## 使用数据查询语句
- **查询一条数据fetchone()**
```python
from pymysql import *

conn = connect(
    host='127.0.0.1',
    port=3306, user='root',
    password='123456',
    database='itcast',
    charset='utf8')

# 创建游标
c = conn.cursor()
# 执行sql语句
c.execute("select * from student")
# 查询一行数据
result = c.fetchone()
print(result)
# 关闭游标
c.close()
# 关闭数据库连接
conn.close()
"""
(1, '张三', 18, b'\x01')
"""
```
- **查询多条数据fetchall()**
```python
from pymysql import *

conn = connect(
    host='127.0.0.1',
    port=3306, user='root',
    password='123456',
    database='itcast',
    charset='utf8')

# 创建游标
c = conn.cursor()
# 执行sql语句
c.execute("select * from student")
# 查询多行数据
result = c.fetchall()
for item in result:
    print(item)
# 关闭游标
c.close()
# 关闭数据库连接
conn.close()
"""
(1, '张三', 18, b'\x01')
(2, '李四', 19, b'\x00')
(3, '王五', 20, b'\x01')
"""
```
- **更改游标的默认设置，返回值为字典**
```python
from pymysql import *

conn = connect(
    host='127.0.0.1',
    port=3306, user='root',
    password='123456',
    database='itcast',
    charset='utf8')

# 创建游标，操作设置为字典类型
c = conn.cursor(cursors.DictCursor)
# 执行sql语句
c.execute("select * from student")
# 查询多行数据
result = c.fetchall()
for item in result:
    print(item)
# 关闭游标
c.close()
# 关闭数据库连接
conn.close()
"""
{'id': 1, 'name': '张三', 'age': 18, 'sex': b'\x01'}
{'id': 2, 'name': '李四', 'age': 19, 'sex': b'\x00'}
{'id': 3, 'name': '王五', 'age': 20, 'sex': b'\x01'}
"""
```
返回一条数据时也是一样的。返回字典或者时元组看个人需要。

## 使用数据操作语句
执行增加、删除、更新语句的操作其实是一样的。只写一个作为示范。
```python
from pymysql import *

conn = connect(
    host='127.0.0.1',
    port=3306, user='root',
    password='123456',
    database='itcast',
    charset='utf8')

# 创建游标
c = conn.cursor()
# 执行sql语句
c.execute("insert into student(name,age,sex) values (%s,%s,%s)",("小二",28,1))
# 提交事务
conn.commit()
# 关闭游标
c.close()
# 关闭数据库连接
conn.close()
```
和查询语句不同的是必须使用commit()提交事务，否则操作就是无效的。

# 编写数据库连接类
- **普通版**

MysqlHelper.py
```python
from pymysql import connect,cursors

class MysqlHelper:
    def __init__(self,
                 host="127.0.0.1",
                 user="root",
                 password="123456",
                 database="itcast",
                 charset='utf8',
                 port=3306):
        self.host = host
        self.port = port
        self.user = user
        self.password = password
        self.database = database
        self.charset = charset
        self._conn = None
        self._cursor = None

    def _open(self):
        # print("连接已打开")
        self._conn = connect(host=self.host,
                             port=self.port,
                             user=self.user,
                             password=self.password,
                             database=self.database,
                             charset=self.charset)
        self._cursor = self._conn.cursor(cursors.DictCursor)

    def _close(self):
        # print("连接已关闭")
        self._cursor.close()
        self._conn.close()

    def one(self, sql, params=None):
        result: tuple = None
        try:
            self._open()
            self._cursor.execute(sql, params)
            result = self._cursor.fetchone()
        except Exception as e:
            print(e)
        finally:
            self._close()
        return result

    def all(self, sql, params=None):
        result: tuple = None
        try:
            self._open()
            self._cursor.execute(sql, params)
            result = self._cursor.fetchall()
        except Exception as e:
            print(e)
        finally:
            self._close()
        return result

    def exe(self, sql, params=None):
        try:
            self._open()
            self._cursor.execute(sql, params)
            self._conn.commit()
        except Exception as e:
            print(e)
        finally:
            self._close()

```
该类封装了fetchone、fetchall、execute，省去了数据库连接的打开和关闭和游标的打开和关闭。
下面的代码是调用该类的小示例：
```python
from MysqlHelper import *

mysqlhelper = MysqlHelper()
ret = mysqlhelper.all("select * from student")
for item in ret:
    print(item)
"""
{'id': 1, 'name': '张三', 'age': 18, 'sex': b'\x01'}
{'id': 2, 'name': '李四', 'age': 19, 'sex': b'\x00'}
{'id': 3, 'name': '王五', 'age': 20, 'sex': b'\x01'}
{'id': 5, 'name': '小二', 'age': 28, 'sex': b'\x01'}
{'id': 6, 'name': '娃哈哈', 'age': 28, 'sex': b'\x01'}
{'id': 7, 'name': '娃哈哈', 'age': 28, 'sex': b'\x01'}
"""
```

- **上下文管理器版**

mysql_with.py
```python
from pymysql import connect, cursors

class DB:
    def __init__(self,
                 host='localhost',
                 port=3306,
                 db='itcast',
                 user='root',
                 passwd='123456',
                 charset='utf8'):
        # 建立连接
        self.conn = connect(
            host=host,
            port=port,
            db=db,
            user=user,
            passwd=passwd,
            charset=charset)
        # 创建游标，操作设置为字典类型
        self.cur = self.conn.cursor(cursor=cursors.DictCursor)

    def __enter__(self):
        # 返回游标
        return self.cur

    def __exit__(self, exc_type, exc_val, exc_tb):
        # 提交数据库并执行
        self.conn.commit()
        # 关闭游标
        self.cur.close()
        # 关闭数据库连接
        self.conn.close()
```
**如何使用：**
```python
from mysql_with import DB

with DB() as db:
    db.execute("select * from student")
    ret = db.fetchone()
    print(ret)

"""
{'id': 1, 'name': '张三', 'age': 18, 'sex': b'\x01'}
"""
```
