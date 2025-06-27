---
title: 起点聊天室详细介绍以及开发日志
date: 2024-3-23
tags: [C++,Qt,项目]

---

#一个大型项目

<!--more-->

## 起点聊天室的详细介绍

> 本项目仍在持续更新中。

### 介绍
起点聊天室项目包含了**局域网**和**全局网络**两大模块。

### 全局网模块

*   **目的**：以 QQ 为参照，熟悉大型即时通讯软件的开发流程。
*   **服务端技术栈**：`Linux`, `C++`, `libevent` 网络库, 线程池, 数据库连接池, 文件服务器, `MySQL`, `Log4j` 风格的日志框架, 单例模式, 观察者模式, `gdb` 调试工具。
*   **客户端技术栈**：`Qt`, 信号与槽, `RAII`, 文件 IO, 网络编程, 多线程, `Json` 解析。

#### 主要功能

1.  **登录模块**
    *   通过文件 IO 保存登录过的用户头像、账号，并可选择性保存密码。
    *   通过选择历史账号，自动填充头像和密码。
    *   支持无账号密码直接登录局域网聊天。
    *   实现自动登录、记住密码、找回密码、注册账号等基础功能。
2.  **个人信息模块**
    *   支持个人信息的修改与展示。
3.  **好友系统**
    *   支持添加好友。
4.  **聊天功能**
    *   实现一对一单人聊天。
    *   实现多人在线群聊。

### 局域网模块

*   **目的**：开发一个在公司内部使用的轻量化聊天与文件传输工具，防止信息泄露，**无需中心服务器**。
*   **技术栈**：`Qt Widget`, `C++`, 网络编程, 文件 IO, `UDP` 广播, `TCP` 文件传输。

#### 主要功能
1.  实现局域网内所有用户的自动发现、群聊和私聊。
2.  支持选择性地加入或创建群聊。
3.  基于 `TCP` 的可靠文件传输。

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------



## 开发日志

> 本日志仍在持续更新中。

### 登录模块

##### 前端展示
![](/images/qidianliaotian/login.gif)

#### 2024年3月23日

开发了 `login_dat` 接口，用于通过字节流高效地存储和读取用户的头像、账号及密码。

**接口设计**

*   **技术**：利用 `RAII` 思想管理 `user.dat` 文件的资源，在构造时获取资源并初始化，在析构时自动关闭和保存。
*   `login_dat()`
    *   构造函数：自动识别并创建 `./user.dat` 文件。如果文件已存在，则自动读取数据到内存中的 `user_info_list`。
*   `~login_dat()`
    *   析构函数：将内存中 `user_info_list` 的最新状态写回 `user.dat` 文件。
*   `write_user_info(const QPixmap &avater, const QString &name, const QString &password="")`
    *   将新的用户信息写入 `user_info_list`。如果用户已存在，则不进行任何操作。
*   `get_user_info_list() ---> QVector<user_info>*`
    *   返回 `user_info_list` 的指针。
*   `is_has(const QString& name) ---> bool`
    *   检查指定账号是否存在。

**`user.dat` 字节流格式**
```
[qint64: 用户列表数量]
    [byteArray: 用户1头像] -> [QString: 用户1账号] -> [qint64: 密码是否存在] -> [QString: 用户1密码]
    [byteArray: 用户2头像] -> [QString: 用户2账号] -> [qint64: 密码是否存在] -> [QString: 用户2密码]
    ...
```

**开发难点**

1.  **QPixmap 与 QByteArray 之间的转化**
    *   从 `QByteArray` 加载 `QPixmap`：`QPixmap.loadFromData(QByteArray)`
    *   将 `QPixmap` 保存到 `QByteArray`：需要一个 `QBuffer` 作为中介。
    ```cpp
    QByteArray bytes;
    QBuffer buffer(&bytes);
    buffer.open(QIODevice::WriteOnly);
    pixmap.save(&buffer, "PNG"); // format
    buffer.close();
    ```
    > **为什么需要 QBuffer？**
    > 因为 `QPixmap::save()` 方法需要一个 `QIODevice` 设备作为写入目标，而 `QByteArray` 本身不是 `QIODevice`。`QBuffer` 可以将 `QByteArray` 包装成一个内存中的 `QIODevice` 设备，从而让 `save()` 方法可以将图片数据写入到这个字节数组中。

### Log4j 日志框架

##### 框架分析
![](/images/qidianliaotian/微信图片_20240326213819.jpg)

#### 2024年3月26日

断断续续分析了两天，大致框架终于搞懂了。日志系统主要分为四大模块：**日志器 (Logger)**、**日志事件 (LogEvent)**、**格式化器 (Formatter)** 和 **输出地 (Appender)**。

**代码亮点：**

使用宏定义简化字符串到枚举的转换，代码更简洁，但牺牲了一定的可读性。
```cpp
logLevel::Level logLevel::FromString(std::string str)
{
#define XX(xlevel, v) \
    if(#xlevel == str || #v == str) \
        return logLevel::xlevel;
    XX(DEBUG, debug);
    XX(INFO, info);
    XX(WARN, warn);
    XX(ERROR, erron);
    XX(FATAL, fatal);
#undef XX
    return logLevel::UNKNOW;
}
```

复习 C/C++ 的可变参数列表：
```cpp
void logEvent::format(const char *fmt, ...)
{
    va_list al;
    va_start(al, fmt);
    char *buf = nullptr;
    // vasprintf会动态分配内存，需要手动free
    int len = vasprintf(&buf, fmt, al);
    if(len != -1){
        m_ss << std::string(buf, len);
        free(buf);
    }
    va_end(al);
}
```
*   `va_list`: 定义一个指向可变参数列表的指针。
*   `va_start`: 初始化 `va_list` 指针，使其指向第一个可变参数。
*   `va_arg`: 读取当前参数，并使指针后移。
*   `va_end`: 清理可变参数列表。

#### 2024年3月29日

通过 `gdb` 调试，终于把日志系统的核心业务逻辑彻底搞懂了。这是一个典型的责任链与多态结合的设计模式。

**核心流程：**
1.  **日志管理器 (LoggerManager)**：采用单例模式管理所有的日志器。默认会创建一个名为 `root` 的根日志器。
2.  **日志器 (Logger)**：`root` 日志器在构造时，会默认创建一个标准输出 `StdOutAppender` 和一个默认格式 `DefaultFormatter`。
3.  **事件 (LogEvent)**：当代码中产生一条日志时，会创建一个 `LogEvent` 对象。该对象必须关联一个 `Logger`。
4.  **事件包装器 (LogEventWrap)**：`LogEvent` 通常被 `LogEventWrap` 包装。在 `LogEventWrap` 的析构函数中，它会调用 `Logger` 的写入函数，将 `LogEvent` 自身作为参数传入。
5.  **写入过程**：
    *   `Logger` 收到 `LogEvent` 后，会遍历其下所有可用的 `Appender` 列表。
    *   通过多态调用每个 `Appender` 的写入函数。
    *   在 `Appender` 内部，再通过多态调用其关联的 `Formatter` 的解析函数。
    *   `Formatter` 将日志格式字符串（如 `%d{%Y-%m-%d %H:%M:%S}%T%t%T%N%T%F%T[%p]%T[%c]%T%f:%l%T%m%n`）解析成一个个具体的 `FormatItem` 子类。
    *   最后，每个 `FormatItem` 子类从 `LogEvent` 中提取自己需要的信息（如时间、行号、线程ID等），写入最终的输出流。

**设计优点：**
1.  **高度解耦，控制力强**：可以自由组合 `Logger`, `Appender`, `Formatter`。例如，可以给不同的 `Appender` 设置不同的 `Formatter`，再将这些 `Appender` 绑定到同一个 `Logger`，实现一次日志调用，多格式、多目的地输出。
2.  **为多线程优化，性能强大**：在多线程环境中，可以为不同线程或模块设置不同的 `Logger` 实例。只有在通过 `LoggerManager` 获取 `Logger` 时需要加锁，后续的日志写入过程几乎是无锁的，大大减少了线程等待时间。

#### 2024年3月30日
<img src="/images/qidianliaotian/微信图片_20240330214310.jpg" style="zoom: 33%;" />

进行了客户端下一个页面的 UI 设计：
![](/images/qidianliaotian/1711806368133.png)

好友列表计划使用 `QListWidget` 结合自定义控件来实现。

**如何实现页面跳转？**
通过 `hide()` 和 `show()` 方法，并利用信号槽机制处理返回逻辑。
```cpp
void login::on_local_btn_released()
{
    if(local_net == nullptr)
    {
    	local_net = new Widget();
	    // 当子页面发出 noReturn 信号时，调用本页面的 show() 方法
	    connect(local_net, &Widget::noReturn, this, &login::show);
    }
    local_net->show();
    this->hide();
}
```

**`qq_client` 类的作用？**

此类作为客户端的核心，负责网络连接和数据管理。
*   **业务逻辑**：使用 `TCP` 连接服务器。登录成功后，服务器返回一个唯一 `ID`。客户端根据此 `ID` 检查本地是否存在缓存文件（如 `id_{XXX}.dat`），以减少不必要的网络请求。如果缓存不存在，则通过 `TCP` 请求个人信息、好友列表等。同时，通过心跳包机制定时检查好友的在线状态。
*   **异步处理**：QT 中强大的信号槽机制是实现 `TCP` 异步编程的核心。当网络套接字上有数据可读时，会自动发出 `readyRead()` 信号，我们只需连接此信号到对应的处理槽函数即可，无需手动创建和管理线程，使代码更简洁、健壮。
