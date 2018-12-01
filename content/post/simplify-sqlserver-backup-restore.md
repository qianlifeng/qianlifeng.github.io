---
title: 利用python简化sql server数据导入导出
date: 2012-02-08 10:28:00
tags: [数据库, python]
---

前面我们介绍了如何使用 sqlserver 的 BCP 命令进行数据的导出和导入。使用这个命令一个不足的地方的是不能对整个数据库进行一次性操作。如果表很多的时候就要一个个命令操作，不可谓不麻烦！
正好最近在学习 python，于是打算用 python 实现了一个简化 bcp 操作的程序。点击程序后就会自动将指定的数据库所有的表的备份数据放到一个特定的目录下。当然，程序也提供对应的自动化导入功能

因为要获得一个数据库中所有的表，所以我使用了 pymssql 来进行数据库操作并进行了简单的封装，形成了 MSSQLHelper 类。具体的 pymssql 的下载和用法大家可以到网上进行搜索。思路很简单：

- 导入的时候查看是否已经存在特定的导出目录，如果存在则删除该目录下所有的文件
- 利用 pymssql 读取指定数据库中所有的表名字，对每个表进行 bcp 命令
- 导入的功能则是省去了删除已存在文件那个操作，其他都差不多。

```
#!/usr/bin/env python
# -*- coding: UTF-8 -*-
#-------------------------------------------------------------------------------
# Name:        导出数据库数据.py
# Purpose:
#
# Author:      SQ1000
#
# Created:     08-02-2012
#-------------------------------------------------------------------------------

import os
import pymssql
import sys

class MSSQLHelper:
    """
    对pymssql的简单封装
    pymssql库，该库到这里下载：http://www.lfd.uci.edu/~gohlke/pythonlibs/#pymssql
    使用该库时，需要在Sql Server Configuration Manager里面将TCP/IP协议开启

    用法：

    """

    def __init__(self,host,user,pwd,db):
        self.host = host
        self.user = user
        self.pwd = pwd
        self.db = db

    def __GetConnect(self):
        """
        得到连接信息
        返回: conn.cursor()
        """
        if not self.db:
            raise(NameError,"没有设置数据库信息")
        self.conn = pymssql.connect(host=self.host,user=self.user,password=self.pwd,database=self.db,charset="utf8")
        cur = self.conn.cursor()
        if not cur:
            raise(NameError,"连接数据库失败")
        else:
            return cur

    def ExecQuery(self,sql):
        """
        执行查询语句
        返回的是一个包含tuple的list，list的元素是记录行，tuple的元素是每行记录的字段

        调用示例：
                ms = MSSQLHelper(host="localhost",user="sa",pwd="123456",db="PythonWeiboStatistics")
                resList = ms.ExecQuery("SELECT id,NickName FROM WeiBoUser")
                for (id,NickName) in resList:
                    print str(id),NickName
        """
        cur = self.__GetConnect()
        cur.execute(sql.encode("utf8"))
        resList =  cur.fetchall()

        #查询完毕后必须关闭连接
        self.conn.close()
        return resList

    def ExecNonQuery(self,sql):
        """
        执行非查询语句

        调用示例：
            cur = self.__GetConnect()
            cur.execute(sql)
            self.conn.commit()
            self.conn.close()
        """
        cur = self.__GetConnect()
        cur.execute(sql.encode("utf8"))
        self.conn.commit()
        self.conn.close()


def RemoveDirectory (top):
    while 1:
        if os.path.exists(top):
            if len(os.listdir(top)) == 0:
                os.rmdir (top)
                break
            else:
                for root, dirs, files in os.walk(top, topdown=False):
                    for name in files:
                        os.remove(os.path.join(root, name))
                    for name in dirs:
                        os.rmdir(os.path.join(root, name))
        else:
            break

def export(db,user,password):
    """导出数据库"""
    #得到当前脚本的执行目录
    currentDirectory = os.getcwd()

    #查看是否已经存在备份目录，如果有则删除，没有则新建目录
    backUpDirectory = "%s\\%s" %( currentDirectory,db+"Backup")
    if os.path.exists(backUpDirectory):
        RemoveDirectory(backUpDirectory)
        os.mkdir(backUpDirectory)
    else:
        os.mkdir(backUpDirectory)

    #得到要到处的数据库的所有表
    ms = MSSQLHelper(host="localhost",user=user,pwd=password,db=db)
    for (name,) in ms.ExecQuery("select name from sysobjects where xtype='U'"):
        currentTablePath = "%s\\%s.txt"%(backUpDirectory,name)
        r = os.popen('BCP %s..%s out %s -c -U"%s" -P"%s"' % (db,name,currentTablePath,user,password))
        print r.read()
        r.close()

def inport(db,user,password):
    """导入数据库"""
    #得到当前脚本的执行目录
    currentDirectory = os.getcwd()

    #查看是否已经存在备份目录，如果有则删除，没有则新建目录
    backUpDirectory = "%s\\%s" %( currentDirectory,db+"Backup")
    if os.path.exists(backUpDirectory):
        #得到要到处的数据库的所有表
        ms = MSSQLHelper(host="localhost",user=user,pwd=password,db=db)
        for (name,) in ms.ExecQuery("select name from sysobjects where xtype='U'"):
            currentTablePath = "%s\\%s.txt"%(backUpDirectory,name)
            r = os.popen('BCP %s..%s in %s -c -U"%s" -P"%s"' % (db,name,currentTablePath,user,password))
            print r.read()
            r.close()


def main():

    db = "PythonWeiboStatistics"
    user = "sa"
    password = "123456"

    #这边可以根据不同的参数选择不同的操作
    #我是使用了两个文件，一个是导入一个导出
    export(db,user,password)

if __name__ == '__main__':
    main()

    print u"\n导出完成...\n回车键退出"
    raw_input()
```
