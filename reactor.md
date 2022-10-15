# reactor

## 前置知识

- socket编程
- IO多路复用epoll

## 常见网络编程模型

## server-client连接原理

一共三类socketfd：\
1、connfd：客户端调用connect与服务端建立连接\
2、listenfd：服务端的监听套接字\
3、clientfd：服务端获取的已连接客户端套接字
