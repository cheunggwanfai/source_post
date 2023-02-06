---
title: HttpServletRequest获取URL,URI区别
date: 2023-02-01 12:30:16
categories: Technique
tags: [Java, Http, 笔记]
---

HttpServletRequest可以分别获取完整的URL，URL路径以及URL后面的参数等。

```
request.getRequestURL(); // 返回的是完整的url，包括Http协议，端口号，servlet名字和映射路径，但它不包含请求参数。

request.getRequestURI()    // 得到的是URL的前部分值

request.getServletPath()    // 返回调用servlet的部分url

request.getQueryString()    // 返回url路径后面的查询字符串
```

例子：

```
请求url：http://localhost:8000/test/test?a=1&b=2&c=3

    System.out.println(request.getRequestURI());
    System.out.println(request.getRequestURL());
    System.out.println(request.getServletPath());
    System.out.println(request.getQueryString());
```

得到的结果是：

```
/test/test
http://localhost:8000/test/test
/test/test
a=1&b=2&c=3
```
