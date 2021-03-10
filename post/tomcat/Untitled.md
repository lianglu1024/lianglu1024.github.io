### 容器

![image-20210310110926501](https://tva1.sinaimg.cn/large/008eGmZEly1goemuzjkunj312u06aq8x.jpg)

Tomcat的核心组件：Connector和Container

Connector将Socket输入转为Request对象，交由Catalina容器进行处理，处理请求完成后，Catalina通过Connector提供的Response对象将结果写入输出流

![image-20210309154507661](https://tva1.sinaimg.cn/large/008eGmZEly1godp7jlg7nj30eb08swfp.jpg)

