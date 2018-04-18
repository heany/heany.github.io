---
title: python3.x与MySQL的安装与链接
date: 2017-10-15 14:02:45
categories:
- python
tags:
- python学习笔记
comments: true
---
<!-->
 需要用到python来进行mysql数据库处理，遇到了一些问题，比如python如何连接MySQL进行数据库相关操作等，接下来我将一些python3中一些MySQL处理的一些操作写在下面。
<!-- more -->

> python2.x版本连接MySQL数据库需要安装MySQL-python库，打开终端输入命令：

    pip install MySQL-python
    或者
    pip install MySQLdb

> python3.x版本链接MySQL数据库需要安装PyMySQL库，打开终端输入命令：

    pip install PyMySQL
还可以通过Pycharm软件里面的库管理来添加pymysql
```python
import pymysql  
      
# 连接数据库  
connect = pymysql.Connect(  
    host='localhost',  
    port=3306,  
    user='root',  
    passwd='1234',  
    db='save',  
    charset='utf8'  
 )  
      
# 获取游标  
cursor = connect.cursor()  
  
# 插入数据  
sql = "INSERT INTO money (name, account, saving) VALUES ( '%s', '%s', %.2f )"  
data = ('雷军', '13512345678', 10000)  
cursor.execute(sql % data)  
connect.commit()  
print('成功插入', cursor.rowcount, '条数据')  
  
# 修改数据  
sql = "UPDATE money SET saving = %.2f WHERE account = '%s' "  
data = (8888, '13512345678')  
cursor.execute(sql % data)  
connect.commit()  
print('成功修改', cursor.rowcount, '条数据')  
  
# 查询数据  
sql = "SELECT name,saving FROM money WHERE account = '%s' "  
data = ('13512345678',)  
cursor.execute(sql % data)  
for row in cursor.fetchall():  
    print("Name:%s\tSaving:%.2f" % row)  
print('共查找出', cursor.rowcount, '条数据')  
  
# 删除数据  
sql = "DELETE FROM money  WHERE account = '%s' LIMIT %d"  
data = ('13512345678', 1)  
cursor.execute(sql % data)  
connect.commit()  
print('成功删除', cursor.rowcount, '条数据')  
  
# 事务处理  
sql_1 = "UPDATE money SET saving = saving + 1000 WHERE account = '18012345678' "  
sql_2 = "UPDATE money SET expend = expend + 1000 WHERE account = '18012345678' "  
sql_3 = "UPDATE money SET income = income + 2000 WHERE account = '18012345678' "  
  
try:  
    cursor.execute(sql_1)  # 储蓄增加1000  
    cursor.execute(sql_2)  # 支出增加1000  
    cursor.execute(sql_3)  # 收入增加2000  
except Exception as e:  
    connect.rollback()  # 事务回滚  
    print('事务处理失败', e)  
else:  
    connect.commit()  # 事务提交  
    print('事务处理成功', cursor.rowcount)  
  
# 关闭连接  
cursor.close()  
connect.close()  
```
