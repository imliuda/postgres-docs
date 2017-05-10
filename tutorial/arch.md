# 1.2 基本结构

在继续进行之前，你要明白PostgreSQL的基本系统结构。理解PostgreSQL个部分之间如何交互能让你对本章内容由一个更清晰的理解。

PostgreSQL采用了客户端/服务器模型，一个PostgreSQL会话由下列协同进程组成（程序）：
 - 服务端程序，该程序负责管理数据库文件，接受来自客户端的连接，代表客户端执行数据库的操作。该服务端程序叫postgres。
 - 想要对数据库进行操作的客户端程序，客户端程序可以具有多种不同的形态，可以是一个文本工具，图形工具，web服务器或者是特殊的数据库维护工具。一些客户端程序是PostgreSQL发型时自带的，但大多数是由用户开发的。
 
作为一个典型的客户端/服务器模型应用，客户端和服务端可以位于不同的主机上。这种情况下，他们之间通过TCP/IP网络进行通信。你要记住这一点，You should keep this in mind, because the files that can be accessed on a client machine might not be accessible (or might only be accessible using a different file name) on the database server machine.

PostgreSQL服务器可同时处理来自客户端的多个连接。对于每一个连接，postgres会调用fork创建一个新的进程。因此，客户端和新创建的服务进程之间的交互不会影响到原来的postgres进程。因此，服务器的主进程总是运行着的，等待客户端连接请求，waiting for client connections, whereas client and associated server processes come and go. (All of this is of course invisible to the user. We only mention it here for completeness.)
