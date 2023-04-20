---
title: andriod andserver 全局转发代理（部分接口在本地）的实现思路
date: 2022-05-11 11:36:15
tags: Andserver
categories: andorid
thumbnail: "/images/jp_fengmian.jpg"
---

# 前言
AndServer 是 Android 平台的 Web Server 和 Web Framework。 它基于编译时注解提供了类似 SpringMVC 的注解和功能，如果您熟悉 SpringMVC，则可以非常快速地掌握它。----yanzhenjie
## 参考资料
 链接:源码地址 [https://github.com/yanzhenjie/AndServer/](https://github.com/yanzhenjie/AndServer/).
  链接:文档地址 [https://yanzhenjie.github.io/AndServer/](https://yanzhenjie.github.io/AndServer/).
   链接:旧版文档 [https://yanzhenjie.github.io/AndServer/1.x/](https://yanzhenjie.github.io/AndServer/1.x/).

# 实现思路
### 业务需求

项目采用了andserver的架构来实现一个安卓端的网络框架，由一个webview套一个网页发请求到andserver，但是很多接口是需要中台处理的。

安卓端代码大概是这样的：
  项目前期在andserver的接口接收到请求后，使用retrofit转发给中台，接收到中台的请求后，再由andserver返回给网页；甚至有的接口需要在安卓端做一些处理然后再转发。    
```javascript
    @RequestMapping(method = {RequestMethod.OPTIONS, RequestMethod.GET}, path = "/test")
    public AndroidResponse test(@RequestParam("organizationId") String store, @RequestParam("customerId") String customer) throws IOException {
        AndroidResponse response = new AndroidResponse();
        //这里可以自己做处理(请求前)
        Call<AndroidResponse> call = api.test(store, customer);
        Response<AndroidResponse> responseResponse = call.execute();
        if (!responseResponse.isSuccessful() || responseResponse.errorBody() != null) {
            Logger.err("Unable to sysLookupItem request:isSuccessful" + responseResponse.isSuccessful() + ",errorBody:" + responseResponse.errorBody());
//                throw new RuntimeException("Unable to reqeust");
            List<LookupitemModel> lookupitemModels = offlineLookupitem();
            if (lookupitemModels != null) {
                response .setData(lookupitemModels);
            }
            return response ;
        }
        response = responseResponse.body();
        //这里可以自己做处理(请求后)
        return response ;
    }
```

### 瓶颈
前面写的挺顺畅的，看到这儿大家可能都觉得没什么问题，这确实是一个能解决问题的方案，但是有一天前后台的接口因为业务需求进行了一些改动，比如加了个入参之类的，这个时候安卓端不仅需要改动andserver的api，还需要改动retrofit的api，改一个接口没有问题，改两个接口也没有问题，但是当改动的接口涉及到整个项目，且项目比较赶的时候，这个问题就很大了。

架构师说：“我们必须要有一个全局的转发，然后还不能全转发，一些调硬件的接口得你自己处理，一些登录的初始化你的拦截做了处理然后发中台......”

我的内心：“/-+！@#￥%&**”

我的回答：“好的，我下去看看”
![在这里插入图片描述](/images/e245f3d9f01e4ea6af2f7f35e8e8c9ff.jpg)
### 思考
然后就是查资料哇，面向百度编程嘛，开始我就想，既然andserver提供了类似 SpringMVC 的注解和功能，我是不是可以通过Interceptor拦截器来实现然后把文档墙放好，一头撞了上去

andserver的文档是这么解释的：
   链接:  HandlerInterceptor [https://yanzhenjie.com/AndServer/class/HandlerInterceptor.html](https://yanzhenjie.com/AndServer/class/HandlerInterceptor.html).
   
```javascript
import com.aw.ccpos.client.logger.Logger;
import com.yanzhenjie.andserver.annotation.Interceptor;
import com.yanzhenjie.andserver.framework.HandlerInterceptor;
import com.yanzhenjie.andserver.framework.handler.RequestHandler;
import com.yanzhenjie.andserver.http.HttpRequest;
import com.yanzhenjie.andserver.http.HttpResponse;

import androidx.annotation.NonNull;

@Interceptor
public class SysInterceptor implements HandlerInterceptor {

    /**
     * Intercept the execution of a handler.
     *
     * @param request  current request.
     * @param response current response.
     * @param handler  the corresponding handler of the current request.
     * @return true if the interceptor has processed the request and responded.
     */
    @Override
    public boolean onIntercept(@NonNull HttpRequest request, @NonNull HttpResponse response, @NonNull RequestHandler handler) throws Exception {
        RequestModel requestModel = new RequestModel();
        String app = request.getHeader("app");
        requestModel.setApp(app);
        String token = request.getHeader("x-token");
        requestModel.setToken(token);
        String storeId = request.getHeader("storeId");
        requestModel.setStoreId(storeId);
        String terminalId = request.getHeader("terminalId");
        requestModel.setTerminalId(terminalId);
        String terminalNo = request.getHeader("terminalNo");
        requestModel.setTerminalNo(terminalNo);

        String organizationId = request.getHeader("organizationId");
        requestModel.setOrganizationId(organizationId);

        String userid = request.getHeader("userid");
        requestModel.setUserid(userid);

        String username = request.getHeader("username");
        requestModel.setUsername(username);

        String displayName = request.getHeader("displayname");
        requestModel.setDisplayName(displayName);

        String customerId = request.getHeader("terminalCustomerId");
        requestModel.setCustomerId(customerId);

        String storeCode = request.getHeader("storeCode");
        requestModel.setStoreCode(storeCode);

        String sName = request.getHeader("storeName");
        requestModel.setStoreNameOriginal(sName);

        SysUtil.create(requestModel);
        Logger.d("SysInterceptor check : requestModel " + requestModel);

        //外部链接拒绝访问
//        String ip = request.getHeader("Host");
//        if (!ip.equals("0.0.0.0:8080")){
//            Logger.w("External address access, blocked");
//            return true;
//        }
        return false;
    }
}
```

示例中写的功能和我们需要转发的接口功能差不多，看到过后我就开撸代码，撸完一跑，确实能拦截到，可以转发，就是所有的接口都被拦截了，我一个接口都别想处理，有的同学就说了，你每一个要处理的url判断一下放过去不就行了？
![在这里插入图片描述](/images/e245f3d9f01e4ea6af2f7f35e8e8c9ff.jpg)
确实可以，没有问题，但是这个方法和第一种有什么区别呢？所以还是另寻它法吧

### 最终解决
ExeceptionResolver
   链接:  ExeceptionResolver[https://yanzhenjie.com/AndServer/class/ExceptionResolver.html](https://yanzhenjie.com/AndServer/class/ExceptionResolver.html).
andserver的文档是这么解释的：
ExeceptionResolver用来处理所有请求Http Api时发生的异常，默认情况下会输出异常的Message到客户端。

这个时候又有同学要说了，看起来和我们需求没关系啊
![在这里插入图片描述](/images/e245f3d9f01e4ea6af2f7f35e8e8c9ff.jpg)
别着急，我给你讲讲我清晰的脑回路，当我们本地有接口的时候，andserver正常走接口，我们正常处理然后返回，需要在接口中请求中台就使用retrofit，接口有改动安卓端必须跟着改，这个是无法避免的，但是如果是直接转发走的接口，我们自定义的ExeceptionResolver就会捕获到异常然后从新转发出去就好了
代码如下：
```javascript
import com.aw.ccpos.client.Client;
import com.aw.ccpos.client.logger.Logger;
import com.yanzhenjie.andserver.annotation.Resolver;
import com.yanzhenjie.andserver.error.MethodNotSupportException;
import com.yanzhenjie.andserver.error.NotFoundException;
import com.yanzhenjie.andserver.framework.ExceptionResolver;
import com.yanzhenjie.andserver.framework.body.JsonBody;
import com.yanzhenjie.andserver.http.HttpMethod;
import com.yanzhenjie.andserver.http.HttpRequest;
import com.yanzhenjie.andserver.http.HttpResponse;
import com.yanzhenjie.andserver.http.RequestBody;

import java.util.HashMap;
import java.util.Map;

import androidx.annotation.NonNull;

/**
 * @ProjectName : 
 * @Author : yifeng_zeng
 * @Time : 2021/2/5 10:09
 * @Description : Andserver GlobalExceptionSolver
 */
@Resolver
public class GlobalExceptionSolver implements ExceptionResolver {
    @Override
    public void onResolve(@NonNull HttpRequest request, @NonNull HttpResponse response, @NonNull Throwable e) {
        HttpMethod method = request.getMethod();
	//GlobalException后面再说
        if (!(e instanceof NotFoundException) && !(e instanceof MethodNotSupportException)
                && !(e instanceof GlobalException)) {
            Logger.err("handle request failed", e);
            return;
        }
        String uri = request.getURI();
        uri = uri.replace("scheme:", "");
//        uri = uri.replace("/spin/user","/user");
        if (uri.startsWith("/")) {
            uri = uri.substring(1);
        }
        Logger.i("redirect with URI " + uri);
        String url = Client.i().getServerUrl() + uri;

       	Logger.i("redirect to url " + url);
        String terminalId = request.getHeader("terminalId");
        SysUtil.requestModelLocal.setTerminalId(terminalId);

        String token = request.getHeader("x-token");
        SysUtil.requestModelLocal.setToken(token);

        String terminalNo = request.getHeader("terminalNo");
         SysUtil.requestModelLocal.setTerminalNo(terminalNo);

        String organizationId = request.getHeader("organizationId");
         SysUtil.requestModelLocal.setOrganizationId(organizationId);

        switch (method) {
            case POST:
                try {
                    RequestBody body = request.getBody();
                    String string = body.string();
                    if (e instanceof GlobalException){
                        string = e.getMessage();
                    }
                    Map<String, String> headers = new HashMap<>();
                    for (String header : request.getHeaderNames()) {
                        headers.put(header, request.getHeader(header));
                    }
//                    List<Cookie> cookies= request.getCookies();
                    String result = HttpUtils.postSync(url, string, headers);
                    response.setBody(new JsonBody(result));

                } catch (Exception ioException) {
                    Logger.err("Exception redirect post", ioException);
                }
                break;
            case GET:
                try {
                    Map<String, String> parameters = new HashMap<>();
//                    for (String parameter : request.getParameterNames()) {
//                        parameters.put(parameter, request.getParameter(parameter));
//                    }

                    Map<String, String> headers = new HashMap<>();
                    for (String header : request.getHeaderNames()) {
                        headers.put(header, request.getHeader(header));
                    }

                    String result = HttpUtils.getSync(url, headers, parameters);
                    response.setBody(new JsonBody(result));

                } catch (Exception ioException) {
                    Logger.err("Exception redirect get", ioException);
                }

                break;
            case DELETE:
                try {
                    Map<String, String> parameters = new HashMap<>();
                    for (String parameter : request.getParameterNames()) {
                        parameters.put(parameter, request.getParameter(parameter));
                    }
                    Map<String, String> headers = new HashMap<>();
                    for (String header : request.getHeaderNames()) {
                        headers.put(header, request.getHeader(header));
                    }
                    if (parameters.size() == 0) {
                        RequestBody body = request.getBody();
                        String string = body.string();
                        if (e instanceof GlobalException){
                            string = e.getMessage();
                        }
                        String result = HttpUtils.deleteSync(url, string, headers);
                        response.setBody(new JsonBody(result));
                    } else {
                        String result = HttpUtils.deleteSync(url, headers, parameters);
                        response.setBody(new JsonBody(result));
                    }


                } catch (Exception ioException) {
                    Logger.err("Exception redirect DELETE", ioException);
                }
                break;

            case PUT:
                try {
                    RequestBody body = request.getBody();
                    String string = body.string();
                    if (e instanceof GlobalException){
                        string = e.getMessage();
                    }
                    Map<String, String> headers = new HashMap<>();
                    for (String header : request.getHeaderNames()) {
                        headers.put(header, request.getHeader(header));
                    }
                    String result = HttpUtils.putSync(url, string, headers);
                    response.setBody(new JsonBody(result));

                } catch (Exception ioException) {
                    Logger.err("Exception redirect PUT", ioException);
                }

        }
    }

}

```

有伸手党就要说了，我没有你转发这个HttpUtils，我又难得找，你想不想要赞啊，emmm
![在这里插入图片描述](/images/e245f3d9f01e4ea6af2f7f35e8e8c9ff.jpg)
```javascript
import java.io.IOException;
import java.net.ConnectException;
import java.net.SocketTimeoutException;
import java.net.UnknownHostException;
import java.util.Map;
import java.util.concurrent.TimeUnit;

import javax.net.ssl.HostnameVerifier;
import javax.net.ssl.SSLSession;
import java.util.concurrent.TimeUnit;

import okhttp3.Call;
import okhttp3.Callback;
import okhttp3.HttpUrl;
import okhttp3.Interceptor;
import okhttp3.MediaType;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.RequestBody;
import okhttp3.Response;

public class HttpUtils {

    public static final MediaType JSON = MediaType.parse("application/json; charset=utf-8");
    public static OkHttpClient client = new OkHttpClient.Builder().
    connectTimeout(300, TimeUnit.SECONDS).
    readTimeout(300, TimeUnit.SECONDS)
    .sslSocketFactory(HttpsUtils.getSslSocketFactory(null,null,null))
    .hostnameVerifier(new HostnameVerifier() {
        @Override
        public boolean verify(String hostname, SSLSession session) {
            return true;
        }
    }).writeTimeout(300, TimeUnit.SECONDS).addInterceptor(new Interceptor() {
        @Override
        public Response intercept(Chain chain) throws IOException {
            Request.Builder builder = chain.request()
                    .newBuilder();
            if (SysUtil.get().getToken() != null) {
                builder.addHeader("x-token", SysUtil.get().getToken());
            }
            if (SysUtil.get().getOrganizationId() != null) {
                builder.addHeader("organizationId", SysUtil.get().getOrganizationId());
            }
            if (SysUtil.get().getTerminalId() != null) {
                builder.addHeader("terminalId", SysUtil.get().getTerminalId());
            }
            if (SysUtil.get().getTerminalNo() != null) {
                builder.addHeader("terminalNo", SysUtil.get().getTerminalNo());
            }
            Request request = builder.build();
            return chain.proceed(request);
        }
    }).build();
    /*
     *�첽http post����
     */
    public static void postAsync(String url, String body) throws Exception {

        RequestBody params = RequestBody.create(JSON, body);
        Request request = new Request.Builder().addHeader("Content-Type", "application/json").url(url).post(params).build();
        try {
            Call call = client.newCall(request);
            call.enqueue(new Callback() {
                @Override
                public void onFailure(Call call, IOException e) {

                }

                @Override
                public void onResponse(Call call, Response response) throws IOException {
                    if (response.isSuccessful()) {
                        response.body().string();
                    }
                }
            });
        } catch (Exception e) {
            throw e;
        }
    }

    /*
     * �첽http get����
     */
    public static void getAsync(String url, String body) throws Exception {

        Request request = new Request.Builder().addHeader("Content-Type", "application/json").url(url).get().build();
        try {
            Call call = client.newCall(request);
            call.enqueue(new Callback() {
                @Override
                public void onFailure(Call call, IOException e) {

                }

                @Override
                public void onResponse(Call call, Response response) throws IOException {
                    if (response.isSuccessful()) {
                        response.body().string();
                    }
                }
            });
        } catch (Exception e) {
            throw e;
        }
    }

    /*
     * ͬ��http post����
     */
    public static String postSync(String url, String body, Map<String, String> headers) throws Exception {

        String returnMessage = "";
        RequestBody params = RequestBody.create(JSON, body);
        Request.Builder builder = new Request.Builder();
        builder.addHeader("Content-Type", "application/json");
        for (Map.Entry<String, String> header : headers.entrySet()) {
            builder.addHeader(header.getKey(), header.getValue());
        }
        Request request = builder.url(url).post(params).build();

        try {
            Response response = client.newCall(request).execute();
            if (response.isSuccessful()) {

                returnMessage = response.body().string();
            } else {
                Exception e = new Exception("�������쳣��Ϣ��" + response.code() + ":" + response.message());
                throw e;
            }
        } catch (UnknownHostException e) {

            throw e;
        } catch (ConnectException e) {

            throw e;
        } catch (SocketTimeoutException e) {

            throw e;
        } catch (Exception e) {

            throw e;
        }
        return returnMessage;
    }

    /*
     * ͬ��http get����
     */
    public static String getSync(String url,Map<String, String> headers,Map<String, String> params) throws Exception {

        String returnMessage = "";

        HttpUrl.Builder httpBuilder = HttpUrl.parse(url).newBuilder();

        if (params != null) {
            for (Map.Entry<String, String> param : params.entrySet()) {
                httpBuilder.addQueryParameter(param.getKey(), param.getValue());
            }
        }

        Request.Builder builder = new Request.Builder();
        builder.addHeader("Content-Type", "application/json");

        for (Map.Entry<String, String> header : headers.entrySet()) {
            builder.addHeader(header.getKey(), header.getValue());
        }
        Request request = builder.url(httpBuilder.build())
                .build();
        try {
            Response response = client.newCall(request).execute();
            if (response.isSuccessful()) {

                returnMessage = response.body().string();
            } else {
                Exception e = new Exception("" + response.code() + ":" + response.message());
                throw e;
            }
        } catch (UnknownHostException e) {

            throw e;
        } catch (ConnectException e) {

            throw e;
        } catch (SocketTimeoutException e) {

            throw e;
        } catch (Exception e) {

            throw e;
        }
        return returnMessage;
    }

    /*
    * 同步put请求*
    /

     */

    public static String putSync(String url, String body,Map<String, String> headers) throws Exception {
        String returnMessage = "";
        RequestBody params = RequestBody.create(JSON, body);
        Request.Builder builder = new Request.Builder();
        builder.addHeader("Content-Type", "application/json");
        for (Map.Entry<String, String> header : headers.entrySet()) {
            builder.addHeader(header.getKey(), header.getValue());
        }
        Request request = builder.url(url).put(params).build();
        try {
            Response response = client.newCall(request).execute();
            if (response.isSuccessful()) {

                returnMessage = response.body().string();
            } else {
                Exception e = new Exception("�������쳣��Ϣ��" + response.code() + ":" + response.message());
                throw e;
            }
        } catch (UnknownHostException e) {

            throw e;
        } catch (ConnectException e) {

            throw e;
        } catch (SocketTimeoutException e) {

            throw e;
        } catch (Exception e) {

            throw e;
        }
        return returnMessage;
    }

    /*
     * 同步delete请求,body
     * */
    public static String deleteSync(String url, String body,Map<String, String> headers) throws Exception {
        String returnMessage = "";
        RequestBody params = RequestBody.create(JSON, body);
        Request.Builder builder = new Request.Builder();
        builder.addHeader("Content-Type", "application/json");
        for (Map.Entry<String, String> header : headers.entrySet()) {
            builder.addHeader(header.getKey(), header.getValue());
        }
        Request request = builder.url(url).delete(params).build();
        try {
            Response response = client.newCall(request).execute();
            if (response.isSuccessful()) {

                returnMessage = response.body().string();
            } else {
                Exception e = new Exception("�������쳣��Ϣ��" + response.code() + ":" + response.message());
                throw e;
            }
        } catch (UnknownHostException e) {

            throw e;
        } catch (ConnectException e) {

            throw e;
        } catch (SocketTimeoutException e) {

            throw e;
        } catch (Exception e) {

            throw e;
        }
        return returnMessage;
    }

    /*
     * 同步delete请求,body
     * */
    public static String deleteSync(String url,Map<String, String> headers,Map<String, String> params) throws Exception {
        String returnMessage = "";
        HttpUrl.Builder httpBuilder = HttpUrl.parse(url).newBuilder();


        if (params != null) {
            for (Map.Entry<String, String> param : params.entrySet()) {
                httpBuilder.addQueryParameter(param.getKey(), param.getValue());
            }
        }
        Request.Builder builder = new Request.Builder();
        builder.addHeader("Content-Type", "application/json");
        for (Map.Entry<String, String> header : headers.entrySet()) {
            builder.addHeader(header.getKey(), header.getValue());
        }

        Request request = builder.addHeader("Content-Type", "application/json")
                .url(httpBuilder.build())
                .delete()
                .build();

        try {
            Response response = client.newCall(request).execute();
            if (response.isSuccessful()) {

                returnMessage = response.body().string();
            } else {
                Exception e = new Exception("�������쳣��Ϣ��" + response.code() + ":" + response.message());
                throw e;
            }
        } catch (UnknownHostException e) {

            throw e;
        } catch (ConnectException e) {

            throw e;
        } catch (SocketTimeoutException e) {

            throw e;
        } catch (Exception e) {

            throw e;
        }
        return returnMessage;
    }
}





import java.io.IOException;
import java.io.InputStream;
import java.lang.reflect.Field;
import java.security.KeyManagementException;
import java.security.KeyStore;
import java.security.KeyStoreException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.security.cert.CertificateException;
import java.security.cert.CertificateFactory;
import java.security.cert.X509Certificate;

import javax.net.ssl.HostnameVerifier;
import javax.net.ssl.KeyManager;
import javax.net.ssl.KeyManagerFactory;
import javax.net.ssl.SSLContext;
import javax.net.ssl.SSLSession;
import javax.net.ssl.SSLSocketFactory;
import javax.net.ssl.TrustManager;
import javax.net.ssl.TrustManagerFactory;
import javax.net.ssl.X509TrustManager;

import okhttp3.OkHttpClient;

public class HttpsUtils {

    public static SSLSocketFactory getSslSocketFactory(InputStream[] certificates, InputStream bksFile, String password) {
        try {
            TrustManager[] trustManagers = prepareTrustManager(certificates);
            KeyManager[] keyManagers = prepareKeyManager(bksFile, password);
            SSLContext sslContext = SSLContext.getInstance("TLS");
            TrustManager trustManager = null;
            if (trustManagers != null) {
                trustManager = new MyTrustManager(chooseTrustManager(trustManagers));
            } else {
                trustManager = new UnSafeTrustManager();
            }
            sslContext.init(keyManagers, new TrustManager[]{trustManager}, new SecureRandom());
            return sslContext.getSocketFactory();
        } catch (NoSuchAlgorithmException e) {
            throw new AssertionError(e);
        } catch (KeyManagementException e) {
            throw new AssertionError(e);
        } catch (KeyStoreException e) {
            throw new AssertionError(e);
        }
    }

    private static TrustManager[] prepareTrustManager(InputStream... certificates) {
        if (certificates == null || certificates.length <= 0) return null;
        try {
            CertificateFactory certificateFactory = CertificateFactory.getInstance("X.509");
            KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
            keyStore.load(null);
            int index = 0;
            for (InputStream certificate : certificates) {
                String certificateAlias = Integer.toString(index++);
                keyStore.setCertificateEntry(certificateAlias, certificateFactory.generateCertificate(certificate));
                try {
                    if (certificate != null)
                        certificate.close();
                } catch (IOException e) {
                    Logger.err("error to close certificate", e);
                }
            }
            TrustManagerFactory trustManagerFactory = null;
            trustManagerFactory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
            trustManagerFactory.init(keyStore);
            TrustManager[] trustManagers = trustManagerFactory.getTrustManagers();
            return trustManagers;
        } catch (Exception e) {
            Logger.err("fail to prepareTrustManager", e);
        }
        return null;

    }

    private static KeyManager[] prepareKeyManager(InputStream bksFile, String password) {
        try {
            if (bksFile == null || password == null) return null;
            KeyStore clientKeyStore = KeyStore.getInstance("BKS");
            clientKeyStore.load(bksFile, password.toCharArray());
            KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
            keyManagerFactory.init(clientKeyStore, password.toCharArray());
            return keyManagerFactory.getKeyManagers();
        } catch (Exception e) {
            Logger.err("fail to prepareKeyManager", e);
        }
        return null;
    }

    private static X509TrustManager chooseTrustManager(TrustManager[] trustManagers) {
        for (TrustManager trustManager : trustManagers) {
            if (trustManager instanceof X509TrustManager) {
                return (X509TrustManager) trustManager;
            }
        }
        return null;
    }

    public static void ignoreSSLCheck(OkHttpClient sClient) {
        SSLContext sc = null;
        try {
            sc = SSLContext.getInstance("SSL");
            sc.init(null, new TrustManager[]{new X509TrustManager() {
                @Override
                public void checkClientTrusted(java.security.cert.X509Certificate[] chain, String authType) throws java.security.cert.CertificateException {

                }

                @Override
                public void checkServerTrusted(java.security.cert.X509Certificate[] chain, String authType) throws java.security.cert.CertificateException {

                }

                @Override
                public java.security.cert.X509Certificate[] getAcceptedIssuers() {
                    return null;
                }
            }}, new SecureRandom());
        } catch (Exception e) {
            Logger.err("fail to ignoreSSLCheck", e);
        }

        HostnameVerifier hv1 = new HostnameVerifier() {
            @Override
            public boolean verify(String hostname, SSLSession session) {
                return true;
            }
        };

        String workerClassName = "okhttp3.OkHttpClient";
        try {
            Class workerClass = Class.forName(workerClassName);
            Field hostnameVerifier = workerClass.getDeclaredField("hostnameVerifier");
            hostnameVerifier.setAccessible(true);
            hostnameVerifier.set(sClient, hv1);

            Field sslSocketFactory = workerClass.getDeclaredField("sslSocketFactory");
            sslSocketFactory.setAccessible(true);
            sslSocketFactory.set(sClient, sc.getSocketFactory());
        } catch (Exception e) {
            Logger.err("fail to hostnameVerifier", e);
        }
    }

    private static class UnSafeTrustManager implements X509TrustManager {
        @Override
        public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {
        }

        @Override
        public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
        }

        @Override
        public X509Certificate[] getAcceptedIssuers() {
            return new X509Certificate[]{};
        }
    }

    private static class MyTrustManager implements X509TrustManager {
        private X509TrustManager defaultTrustManager;
        private X509TrustManager localTrustManager;

        public MyTrustManager(X509TrustManager localTrustManager) throws NoSuchAlgorithmException, KeyStoreException {
            TrustManagerFactory var4 = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
            var4.init((KeyStore) null);
            defaultTrustManager = chooseTrustManager(var4.getTrustManagers());
            this.localTrustManager = localTrustManager;
        }

        @Override
        public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {
        }

        @Override
        public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
            try {
                defaultTrustManager.checkServerTrusted(chain, authType);
            } catch (CertificateException ce) {
                localTrustManager.checkServerTrusted(chain, authType);
            }
        }

        @Override
        public X509Certificate[] getAcceptedIssuers() {
            return new X509Certificate[0];
        }
    }

    private class UnSafeHostnameVerifier implements HostnameVerifier {
        @Override
        public boolean verify(String hostname, SSLSession session) {
            return true;
        }
    }

}

```

### 升级版本
有细心的同学发现我们还有一个GlobalException的异常，这个东西是我们在andserver接收到请求，对请求处理过后不需要处理返回的信息的时候使用的自定义Exception；可以根据你们自己的业务需求取用
```javascript
/**
 * 
 * @Author : yifeng_zeng
 * @Time : 2021/11/5 14:22
 * @Description : 全局转发异常，为了把body传给GlobalExceptionSolver
 */
public class GlobalException extends BasicException {
    private static int statusCode = 100;

    public GlobalException(DataRequest request) {
        super(statusCode,JSON.toJSONString(request));
    }

    public GlobalException(String request) {
        super(statusCode,request);
    }
}
```

使用
```javascript
@Override
	@RequestMapping(method = {RequestMethod.OPTIONS, RequestMethod.POST}, path = "/finishOrderSale")
	public Response finishOrderSale(@RequestBody Request requestModel)throws IOException{
		if (requestModel.getData()!=null){
		//这里可以对requestModel进行处理
			throw new GlobalException(requestModel);
		}
		Response  response = new  Response();
		return response;
	}
```

这个时候，坐在屏幕前的你是不是应该说一句：“秒啊！”
![在这里插入图片描述](/images/e245f3d9f01e4ea6af2f7f35e8e8c9ff.jpg)