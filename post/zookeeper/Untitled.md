客户端连接服务端必须要进行初始化

ConnectRequest

![image-20210201211409556](https://tva1.sinaimg.cn/large/008eGmZEly1gn8cevse2xj30nm0aejuq.jpg)

连接完成之后才能发送create等一系列命令



### Request

![image-20210201220209498](https://tva1.sinaimg.cn/large/008eGmZEly1gn8dst6dgnj318e0i50z9.jpg)





socket连接，如果客户端kill调 服务端是怎么感知 并抛出ioexception。自己试试看



leader选举的过程时，是不能接收客户端的请求