### 容器

![image-20210310110926501](https://tva1.sinaimg.cn/large/008eGmZEly1goemuzjkunj312u06aq8x.jpg)

Tomcat的核心组件：Connector和Container

Connector将Socket输入转为Request对象，交由Catalina容器进行处理，处理请求完成后，Catalina通过Connector提供的Response对象将结果写入输出流

![image-20210309154507661](https://tva1.sinaimg.cn/large/008eGmZEly1godp7jlg7nj30eb08swfp.jpg)





![image-20210310213246820](https://tva1.sinaimg.cn/large/008eGmZEly1gof4vl78pyj30vh0jqdhy.jpg)



![image-20210310215135859](https://tva1.sinaimg.cn/large/008eGmZEly1gof5f4u7rvj30px0c50tp.jpg)





Http11Proceseeor

-getAdapter().service(request, response);

CoyoteAdapter->

```
// Calling the container
connector.getService().getContainer().getPipeline().getFirst().invoke(
request, response);
```

Engine

每个容器内部都有一个Pipeline属性









Host

```
host.getPipeline().getFirst().invoke(request, response);
```



![image-20210312002617200](https://tva1.sinaimg.cn/large/008eGmZEly1gogfifbqq2j30x20bzteh.jpg)