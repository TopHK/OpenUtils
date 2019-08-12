# 异步httpclient(httpasyncclient)的使用与总结
URL: https://blog.csdn.net/ouyang111222/article/details/78884634

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

## example

public class TestHttpClient {
    public static void main(String[] args){

        RequestConfig requestConfig = RequestConfig.custom()
                .setConnectTimeout(50000)
                .setSocketTimeout(50000)
                .setConnectionRequestTimeout(10)//设置为10ms
                .build();

        //配置io线程
        IOReactorConfig ioReactorConfig = IOReactorConfig.custom().
                setIoThreadCount(Runtime.getRuntime().availableProcessors())
                .setSoKeepAlive(true)
                .build();
        //设置连接池大小
        ConnectingIOReactor ioReactor=null;
        try {
            ioReactor = new DefaultConnectingIOReactor(ioReactorConfig);
        } catch (IOReactorException e) {
            e.printStackTrace();
        }
        PoolingNHttpClientConnectionManager connManager = new PoolingNHttpClientConnectionManager(ioReactor);
        connManager.setMaxTotal(1);//最大连接数设置1
        connManager.setDefaultMaxPerRoute(1);//per route最大连接数设置1


        final CloseableHttpAsyncClient client = HttpAsyncClients.custom().
                setConnectionManager(connManager)
                .setDefaultRequestConfig(requestConfig)
                .build();


        //构造请求
        String url = "http://127.0.0.1:9200/_bulk";
        List<HttpPost> list = new ArrayList<HttpPost>();
        for(int i=0;i<2;i++){
            HttpPost httpPost = new HttpPost(url);
            StringEntity entity = null;
            try {
                String a = "{ \"index\": { \"_index\": \"test\", \"_type\": \"test\"} }\n" +
                        "{\"name\": \"上海\",\"age\":33}\n";
                entity = new StringEntity(a);
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }
            httpPost.setEntity(entity);
            list.add(httpPost);
        }

        client.start();

        for(int i=0;i<2;i++){
            client.execute(list.get(i), new Back());
        }

        while(true){
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    static class Back implements FutureCallback<HttpResponse>{

        private long start = System.currentTimeMillis();
        Back(){
        }

        public void completed(HttpResponse httpResponse) {
            try {
                System.out.println("cost is:"+(System.currentTimeMillis()-start)+":"+EntityUtils.toString(httpResponse.getEntity()));
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        public void failed(Exception e) {
            e.printStackTrace();
            System.err.println(" cost is:"+(System.currentTimeMillis()-start)+":"+e);
        }

        public void cancelled() {

        }
    }
}
