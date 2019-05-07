# sqlserver运行超大sql文件

# 场景描述

在往sqlserver数据库运行sql文件导入数据时，对于小的sql文件，直接在SQL Server Management Studio里打开执行就行了，但有几个表的数据量非常大，运行sql文件时提示内存不足。

# 参考解决方法

使用自带sqlcmd命令工具进行执行导入。

1、如我使用的是sqlserver2008，是安装在d盘的，打开命令行，进入Binn目录：

```
cd D:\Program Files\Microsoft SQL Server\100\Tools\Binn
```

2、输入以下命令

```
sqlcmd -S localhost -U sa -P sasa -d chrysler -i e:\sql\user_folder.sql
```
说明：

1. `-S`：数据库服务器地址，我这里是本机直接用localhost
2. `-U`：用户名
3. `-P`：密码
4. `-d`：数据库名
5. `-i`：sql文件
