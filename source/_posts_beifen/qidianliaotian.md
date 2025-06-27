---
title: 起点聊天室详细介绍以及开发日志
date: 2024-3-23
tags: [C++,Qt,项目]

---

#一个大型项目

<!--more-->

## 起点聊天室的详细介绍（暂未完结持续更新中）

##### 介绍：包含了局域网，全局网络两个大模块。

#####   全局网

###### 		目的：以QQ为参照，熟悉软件开发。

###### 		阿里云服务器服务器：Linux，C++，libevent，线程池，连接池，文件服务器，MySQL，Log4j日志框架，单例模式，观察者模式，gdb工具；

###### 		客户端：Qt，信号与槽，RAII，文件IO，网络编程，多线程，Json；

##### 功能

###### 			1.登录模块

​					A，通过文件IO，保存登录过的用户的登录头像，账号，选择性保存密码，通过选择账号，自动更新头像密码。

​					B，无需账号密码登录局域网聊天系统；

​					C，实现基本的自动登录，记住密码，找回密码，注册账号，安全登录；

###### 2.个人信息模块

​					A，个人信息修改，个人信息展示。

###### 3.添加好友模块

###### 4.个人聊天功能

###### 5.群聊功能

##### 	局域网

###### 		目的：在公司内部使用的聊天工具，目的为了防止信息泄露，**无需服务器**实现了轻量化的聊天，文件传输工具。

###### 		技术点：Qt Widget、网络编程、C++、文件IO、UDP服务器、TCP文件传输

##### 		功能

###### 1.实现了局域网内部的所有人群聊，私聊；

###### 2.选择性的添加群聊；

###### 3.TCP的文件传输；

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------



## 开发日志（暂未完结持续更新中）



#### 登录模块：

##### 		前端展示：![](qidianliaotian/login.gif)

##### 3月23号

​	开发了login_dat接口，接口文档:

​	接口介绍：此接口主要用于字节流存储读取头像，账号，密码，使用非常简洁。

​	技术：RAII思想获取资源及初始化，用RAII管理user.dat文件的打开与关闭。	

​	login_dat()

​		构造过程自动识别并创建当前目录下"./user.dat"文件，如果有user.dat文件，自动读取文件中数据放到user_info_list中。

​	~login_dat()

​		析构函数，主要用于将内存中的user_info_list按照一定规律存储到user.dat文件。

​	write_user_info(const QPixmap &avater,const QString &name,const QString &password="");

​		如果user_info_list中没有则将参数写入user_info_list中，有就什么都不做,最后的password默认参数是指两种写入格式，一种是无密码写入，一种是有密码写入。

​	 get_user_info_list()--->QVector<user_info>*

​		返回user_info_list。

​	is_has(const QString& name) --->bool;

​		查看列表中是否有这个账号

​	user.dat字节流格式：

​			qint64（用户列表数量）---  byteArray（图片字节流）---   QString（账号）---  qint64(用户的密码是否存在？）---   QString（密码）-------------------------------------------------------------------------------

###### 难点：

​	1.QPixmap与QByteArray之间的转化

​		QPixmap.loadFromData(QByteArray)

​		QBuffer(QByteArray)              QPixmap.save(buf)          buf.close()
​	为什么需要buf？

`因为QImage` 的 `save()` 方法需要一个设备接口来写入数据，而 `QByteArray` 本身并不提供这样的接口。通过 `QBuffer`，我们可以将 `QByteArray` 包装为一个 `QIODevice`，从而使得 `QImage` 能够将其数据写入到字节数组中。

#### 

#### Log4j日志框架

##### 分析：![](qidianliaotian/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20240326213819.jpg)

###### 3月26号

断断续续分析了两天了，大致框架终于搞懂了。

首先日志分为了4大模块：

日志入口模块：日志器，日志器管理器，

代码亮点：

```C++
logLevel::Level logLevel::FromString(std::string str)
{
#define XX(xlevel,v) \
        if(#xlevel == str||#v == str)\
            return logLevel::xlevel;
    XX(DEBUG,debug);
    XX(INFO,info);
    XX(WARN,warn);
    XX(ERROR,erron);
    XX(FATAL,fatal);
#undef XX
    return logLevel::UNKNOW;
}
```

用define宏定义使代码简洁，逻辑变简单，但可读性降低了。

复习到了C语言的可变参数列表：

```C++
void logEvent::format(const char *fmt, ...)
{
    va_list al;
    va_start(al,fmt);
    char *buf = nullptr;
    int len = vasprintf(&buf,fmt,al);
    if(len != -1){
        m_ss << std::string(buf,len);
        free(buf);
    }
    va_end(al);
}
```

va_list是定义可变参数列表，va_start代表的是绑定fmt后的参数，va_end是销毁al可变参数列表；补充  va_arg 是将al变量指向下一个参数，并且将变量当成va_arg 函数的第二个参数（第二个参数是type）

###### 3月29号

​	今天通过gdb的调试终于把日志的业务逻辑搞懂了，主要有四个板块，日志器，事件，解析器，日志输出口。

有日志管理器（单例模式）是管理日志器的，默认情况下有一个名字为root的日志器，在root的日志器的构造中默认有一个std::out的日志输出口，一个默认的解析器，并且**日志输出口的解析器指针默认指向root的日志器**。事件就是存放了日志的内容（时间，目前代码执行行数，在那个文件中等等），最重要的是**事件的构造必须要有一个日志器**，用来管理此事件，事件包装器用于包装事件，在事件包装器调用析构函数时，事件用日志器的指针调用日志器的写入函数，将自己作为参数传入，在日志器中的写入函数，遍历可以写入的日志输出口列表，通过多态调用日志输出口的写入函数，在输出口里的写入函数通过遍历调用解析器的写入函数，解析器也是多态的思想实现，将log4j格式解析成item的不同子类，用多态调用这些子类的写入方法，最后在这些子类的写入方法中用事件的内容传入日志输出器，完成了一次写入。

整个过程有点麻烦，但是这样写有很大的优点。

1、控制能力强，比如说可以给不同的日志输出口new不同的解析器，在单线程中用一个日志器绑定这些输出口，等等使用方法限制小，可操作性高。

2、性能强大，在多线程中，可以设置不同的日志器，不同的日志器都绑定成不同的解析器，输出口。在不同的线程中只有日志器管理器分配日志器时需要加锁，其他过程都无需同步，等待时间少，性能强大。



###### 3月30号

<img src="qidianliaotian/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20240330214310.jpg" style="zoom: 33%;" />

进行了客户端下一个页面的设计：

![](qidianliaotian/1711806368133.png)

在此页面的好友列表用List Widget+自定义控件实现。

如何实现页面跳转？

```C++
void login::on_local_btn_released()
{
    if(local_net == nullptr)
    {
    	local_net = new Widget( );
	    connect(local_net,&Widget::noReturn,this,&login::show);
    }
    local_net->show();
    this->hide();
}
```

connect中的noRrturn是返回信号，按返回按键就可触发信号；

qq_client类的作用？

此类用于网络连接，存放数据。业务逻辑：进行TCP连接，首先登录成功后返回一个id，根据id查看id{XXX}.dat本地文件是否有个人信息文件（减少网络连接，提高服务器性能），没有此文件则通过TCP请求返回个人信息，获取好友列表信息，隔一段时间检查好友是否在线。

此类的网络请求怎么异步处理，QT中Tcp异步编程的核心就是`Qt的信号槽机制`，无需创建线程，利用QT特性使代码更简便。
