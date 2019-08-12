# 异步httpclient(httpasyncclient)的使用与总结
## 前言
应用层的网络模型有同步与异步。同步意味当前线程是阻塞的，只有本次请求完成后才能进行下一次请求;异步意味着所有的请求可以同时塞入缓冲区,不阻塞当前的线程;
httpclient在4.x之后开始提供基于nio的异步版本httpasyncclient,httpasyncclient借助了Java并发库和nio进行封装(虽说NIO是同步非阻塞IO,但是HttpAsyncClient提供了回调的机制,与netty类似,所以可以模拟类似于AIO的效果),其调用方式非常便捷,但是其中也有许多需要注意的地方。

## pom

        <dependency>  
            <groupId>org.apache.httpcomponents</groupId>  
            <artifactId>httpclient</artifactId>  
            <version>4.5.2</version>  
        </dependency>  

        <dependency>  
            <groupId>org.apache.httpcomponents</groupId>  
            <artifactId>httpcore</artifactId>  
            <version>4.4.5</version>  
        </dependency>  

        <dependency>  
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpcore-nio</artifactId>
            <version>4.4.5</version>
        </dependency>  

        <dependency>  
            <groupId>org.apache.httpcomponents</groupId>  
            <artifactId>httpasyncclient</artifactId>  
            <version>4.1.2</version>  
        </dependency>
