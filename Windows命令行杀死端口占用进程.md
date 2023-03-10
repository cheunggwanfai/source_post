---
title: Windows命令行杀死端口占用进程
date: 2023-02-06 15:33:32
categories: Technique
tags: [Windows, CMD, 笔记]
---

### 查看特定端口的使用情况 ###

```
以9999端口为例，查询端口被哪个进程占用

命令行：

	netstat -ano | findstr "9999"

		-a  显示所有连接和侦听端口
		-n  以数字形式显示地址和端口号
		-o  显示拥有的与每个连接关联的进程 ID

结果：

	TCP    0.0.0.0:9999    0.0.0.0:0    LISTENING     10020
```

可以看到占用9999端口对应的进程pid为10020


### 根据pid查找对应的程序 ###

```
命令行：

	tasklist | findstr "10020"

结果:

	WXWork.exe      10020   Console      1    236,276 K

上述结果代表的是：进程名，pid，会话名，会话，内存使用
```

### 杀死进程 ###

```
命令行：

	taskkill /f /t /im 10020
		/f  指定强制终止进程
		/t  终止指定的进程和由它启用的子进程
		/im 指定要终止的进程的图像(进程)名。通配符 '*' 可用来指定所有图像名

结果：
	
	成功：已终止 PID 10020
```


