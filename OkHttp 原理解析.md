* **同步请求**

![2c57eb707bef6f1b23e8afba7c81e3c6.png](evernotecid://563ADC6F-3107-4161-9EFF-F60436E68B79/appyinxiangcom/1531278/ENResource/p10)

* **Dispatcher**

![07ca95a5b8ae4b476aa0b330191641f5.png](evernotecid://563ADC6F-3107-4161-9EFF-F60436E68B79/appyinxiangcom/1531278/ENResource/p8)

![53a60d17f6c7e9233530902bad1a9463.png](evernotecid://563ADC6F-3107-4161-9EFF-F60436E68B79/appyinxiangcom/1531278/ENResource/p9)

* **整体流程**：

![e67029972070a7dd84206023b179dbd1.png](evernotecid://563ADC6F-3107-4161-9EFF-F60436E68B79/appyinxiangcom/1531278/ENResource/p11)

1. 建立请求：
    1. OkHttpClient.newCall --- new RealCall 同步请求
    2. OkHttpClient.enqueue --- new AsyncCall 异步请求，放入到 dispatcher 任务队里中去
2. 发起请求：
    1. execute() 实现逻辑：
        1. getResponseWithInterceptorChain 获取服务器放回
        2. 通知任务分发器 dispatcher 该任务结束

    2. getResponseWithInterceptorChain
        1. 创建一系列拦截器，放到数组中，包括用户自定义拦截器和系统框架内部拦截器
        2. 创建一个拦截器链 RealInterceptorChain, 并执行 proceed() 方法
        3. RealInterceptorChain.proceed() 方法
            1. 创建下一个拦截链。传入index + 1使得下一个拦截器链只能从下一个拦截器开始访问
            2. 执行索引为index的intercept方法，并将下一个拦截器链传入该方法
    3. 拦截器 intercept() 方法
        1. 在发起请求前对 request 进行处理
        2. 调用下一个拦截器，获取 response
        3. 对 response 进行处理，返回给上一个拦截器

如何忽略证书：
mHttpClient = new OkHttpClient().newBuilder()
                    ...
                **.sslSocketFactory(SSLSocketClient.getSSLSocketFactory())//配置
                    .hostnameVerifier(SSLSocketClient.getHostnameVerifier())//配置**
                    .build();
                    
```
public class SSLSocketClient {
 
    //获取这个SSLSocketFactory
    public static SSLSocketFactory getSSLSocketFactory() {
        try {
            SSLContext sslContext = SSLContext.getInstance("SSL");
            sslContext.init(null, getTrustManager(), new SecureRandom());
            return sslContext.getSocketFactory();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
 
    //获取TrustManager
    private static TrustManager[] getTrustManager() {
        TrustManager[] trustAllCerts = new TrustManager[]{
                new X509TrustManager() {
                    @Override
                    public void checkClientTrusted(X509Certificate[] chain, String authType) {
                    }
 
                    @Override
                    public void checkServerTrusted(X509Certificate[] chain, String authType) {
                    }
 
                    @Override
                    public X509Certificate[] getAcceptedIssuers() {
                        return new X509Certificate[]{};
                    }
                }
        };
        return trustAllCerts;
    }
 
    //获取HostnameVerifier
    public static HostnameVerifier getHostnameVerifier() {
        HostnameVerifier hostnameVerifier = new HostnameVerifier() {
            @Override
            public boolean verify(String s, SSLSession sslSession) {
                return true;
            }
        };
        return hostnameVerifier;
    }
}
```
