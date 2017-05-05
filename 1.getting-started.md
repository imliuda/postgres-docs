# 1. 开始

## 1.1 安装
要使用PostgreSQL，你需要安装它。当然，它也可能在你的站点上是已经安装了的，或者操作系统自带，或者是由系统管理员进行安装的。如果已经安装好，你需要通过系统文档或者联系系统管理员获得访问数据库的信息。

如果你不确定PostgreSQL是否已经安装，以及是否可用。你可以自己进行安装，这并不困难，并且是一个好的练习。PostgreSQL可由任何非特权用进行安装；不需要超级用户权限。

如果你要自己进行安装，请参考[第16章]()中的指令步骤，安装好后再回到本向导。请确认严格遵循指导，设置了正确的环境变量。

如果你的站点管理员没有采用默认的设置，你可能需要做一些额外的工作。例如，如果数据库服务器是一个远程机器的话，你需要设置PGHOST环境变量为该数据库服务器的主机名。同样，还需要设置PGPORT环境变量。总而言之，当你启动一个应用程序时，如果报了不能连接到数据库的信息，你应该咨询你的站点管理员，或者如果你就是管理员的话，你应该参考文档来确定是否设置了正确的环境变量。如果你不明白前面这段话的话，请接着阅读下一节。

## 1.2 基础结构
在继续进行之前，你要明白PostgreSQL的基本系统结构。理解PostgreSQL个部分之间如何交互能让你对本章内容由一个更清晰的理解。

PostgreSQL采用了客户端/服务器模型，一个PostgreSQL会话由下列协同进程组成（程序）：
 - 服务端程序，该程序负责管理数据库文件，接受来自客户端的连接，代表客户端执行数据库的操作。该服务端程序叫postgres。
 - 想要对数据库进行操作的客户端程序，客户端程序可以具有多种不同的形态，可以是一个文本工具，图形工具，web服务器或者是特殊的数据库维护工具。一些客户端程序是PostgreSQL发型时自带的，但大多数是由用户开发的。
 
作为一个典型的客户端/服务器模型应用，客户端和服务端可以位于不同的主机上。这种情况下，他们之间通过TCP/IP网络进行通信。你要记住这一点，You should keep this in mind, because the files that can be accessed on a client machine might not be accessible (or might only be accessible using a different file name) on the database server machine.

PostgreSQL服务器可同时处理来自客户端的多个连接。对于每一个连接，postgres会调用fork创建一个新的进程。因此，客户端和新创建的服务进程之间的交互不会影响到原来的postgres进程。因此，服务器的主进程总是运行着的，等待客户端连接请求，waiting for client connections, whereas client and associated server processes come and go. (All of this is of course invisible to the user. We only mention it here for completeness.)
