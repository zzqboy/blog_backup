---
title: web的原理
date: 2019-03-23 22:42:10
tags: http
categories: 技术
---
# 介绍
http协议可以说是最常用的一种网络协议，但是也很容易忽略了他的原理。还记得“在浏览器上打开一个网页请求，请你说出这背后的原理”这样的面试题吗？这一切在读《HTTP权威指南》后就可以了如指掌了。
本文在简述原理后给出一个文中的栗子...
<!-- more -->

# 原理
当我们发出一个get请求的时候，http server大概发生了下面几个步骤： 
1. 接受客户端请求
2. 接受请求报文
3. 处理请求
4. 对资源的映射和访问
5. 构建响应
6. 发送响应
7. 记录日志

这一切看来非常像游戏中一条协议的处理，下面详细讲这几个

## 请求
http的请求的本质是tcp，自然会先建立tcp连接，那么server会在接受连接后，等待报文的到达。

## 报文
解析报文的过程：
![http.png](/img/http.png)
由图可知，当数据没有接受完是不能进行解析报文的。 每行的CRLF和Content-length就是用来判断是否接受完

## 处理
在解析报文后，就知道浏览器想让server干嘛了，就可以对应做出各种响应。
(GET、POST、PUT、Delete、HEAD、Options)

## 响应报文
通常包括:(主要还是看协议)
1. Content-Type描述了是哪个MIME类型
2. Content-Length
3. 主体内容

MIME的类型
![http2.png](/img/http2.png)

不过也有可能是重定向的响应，响应码为3XX，不是最终的响应

## 发送
server在发送完报文后，会关闭非持久的连接，对于持久连接是不会关闭的。

# echo server
文中有个perl的echo server，这里用python重现

```python
#coding:utf-8
import socket
import traceback


def star_server():
	s_socket = socket.socket()
	s_socket.bind(("127.0.0.1", 80))
	s_socket.listen(5)
	print "echo server at 80"

	conn, address = None, None
	while not conn:
		conn, address = s_socket.accept()

		recv_chunk = conn.recv(1024)
		recv_data = recv_chunk
		print "------------------------------"
		print recv_data
		print "from", address
		print "------------------------------"

		path = recv_data.split()[1]
		if path == "/":  
			echo_content = raw_input("echo:\n")
			try:
				response =  "HTTP/1.1 200 OK\r\nConnection:close\r\nContent-type:text-plain\r\n\r\n"
				response = response + echo_content
				conn.sendall(bytes(response))
			except Exception, e:
				traceback.print_exc()
		conn.close()
		conn, address = None, None

star_server()
```