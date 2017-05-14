# 1.3 创建数据库

第一个测试是要看你是否能够访问数据库服务器，我们会尝试创建一个新的数据库。PostgreSQL服务器可以管理很多数据库。通常，每个项目或每个用户单独使用一个数据库。

你的站点管理员可能事先为你创建好了数据库供你使用。如果是这样的话，你可以略过此步，前往下一节。

要创建一个名称为mydb的新数据库，使用下面的命令：

```shell
$ createdb mydb
```
如果这个命令没有任何输出，则说明成功创建了数据库。你可以跳过后面的内容。

如果你看见了类似与下面的信息：

```
createdb: command not found
```
那么则是PostgreSQL没有被正确的安装。可能是根本就没有安装，或者是你的shell搜索路径中没有包含它。试着用绝对路径来调用该命令：

```
$ /usr/local/pgsql/bin/createdb mydb
```
这个路径在你的站点上可能会不一样。联系你的站点管理员或者查看安装指引来进行更正。

另外可能出现的信息是：

```
createdb: could not connect to database postgres: could not connect to server: No such file or directory
          Is the server running locally and accepting connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
```

这说明数据库服务没有启动，或者没有按照createdb所期望的方式启动。同样，检查安装指引或联系你的站点管理员。

其他还可能出现的信息可能是：

```
createdb: could not connect to database postgres: FATAL:  role "joe" does not exist
```

上面的登录名是你自己的。如果站点管理员还没有创建该用户的话，会发生这种情况（PostgreSQL的账户不同于系统中的账户）。如果你就是管理员，请参考[第21章](../user/manag.md)查看如何创建账户。你需要切换到PostgreSQL的系统账户下（通常是postgres）来创建用户账户，可能你为PostgreSQL指定了一个不同与你系统账户的用户名。这种情况下，你需要使用-U选项或者设置PGUSER环境变量来指定PostgreSQL的用户名。

如果你拥有一个账户，但是提示没有权限创建数据库的话，你会看到下面的信息：

```
createdb: database creation failed: ERROR:  permission denied to create database
```
不是每个用户都可以创建新的数据库。如果PostgreSQL拒绝为你创建新用户，你需要站点管理员提升你的权限来创建数据库。如果是自己安装的PostgreSQL，你应该使用启动数据库服务的用户的身份进行登录。<a id="AEN469" href="FTN.AEN469">[1]</a>

你也可以用其他用户名创建数据库。在一个站点上，PostgreSQL允许你创建任意数量的数据库。数据库名称必须以字母开头，别切长度不能超过63个字节。一个方便的选择是创建和你登录用户名相同的数据库。许多工具会假定你的登录用户名就是数据库名，所以，你可以减少一些输入。如果要创建这样的数据库，只需简单的数据：

```
$ createdb
```

如果你不在再使用某个数据库，你可以删除它。例如，如果你是mydb数据库的所有者，你可以用下面这个命令删除它：

```
$ dropdb mydb
```

（这种情况下，数据不是默认的用户账户名）这这动作将会物理上删除与所有该数据库相关的文件，并且不能撤销，所以请在充分考虑之后再做决定。

更多有关于createdb和dropdb，请分别参考[createdb](../app/createdb.md)和[dropdb](../app/dropdb.md)。

**注释**

 - <a id="FTN.AEN469" href="#AEN469">[1]</a> PostgreSQL的用户名和系统账户中的用户账户是分开的。当你连接数据库时，你可以选择用那个用户名进行连接。如果没有指定用户名。将会默认采用你当前登录到操作系统的用户名。As it happens, there will always be a PostgreSQL user account that has the same name as the operating system user that started the server, and it also happens that that user always has permission to create databases. Instead of logging in as that user you can also specify the -U option everywhere to select a PostgreSQL user name to connect as.
