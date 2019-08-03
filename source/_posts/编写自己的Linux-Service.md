---
title: 编写自己的Linux Service
top: false
date: 2019-05-20 17:35:16
tags: Linux
categories: 电脑技巧
---
与此相关的两个文件
- /etc/init.d/xxx.service（✘）
- /lib/systemd/system/xxx（✔）

<!-- more -->

Linux操作系统的启动首先从BIOS开始，接下来进入boot loader，并由其载入内核进行初始化。内核初始化的第一步就是启动pid为1的进程，这个进程是系统的第一个进程，它负责产生其他所有用户进程。init以守护进程方式存在，是其它进程的祖先。

以前，绝大多数Linux发行版服务都遵循sysvinit风格。比如Ubuntu、Kali。如今systemd正在逐渐取代sysvinit。它使用unit文件进行服务控制。

- /etc/systemd/system/      -- 供系统管理员和用户使用
- /run/systemd/system/      -- 运行时配置文件
- /usr/lib/systemd/system   -- 安装程序的使用（如rpm包）

我编写了一个简单的脚本，用来启动Beef-XSS daemon

![](/uploads/beef-xss-service.png)
