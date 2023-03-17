---
title: Springboot 配置 Retrofit 并使用各种请求方式
date: 2022-03-04 11:18:41
categories: Java
tags:
    - SpringBoot
    - Retrofit
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/RetrofitLogo.jpg
---
# 前言

<font color=#999AAA >Retrofit 这个东西是简化我们在 java 代码里面书写 http 请求的工具，支持 restful 风格的请求，我们通常发送请求，要用到 hutools 和 httpUtil 这些东西，要写好多行，现在用 Retrofit 只需要两三行。</font>

源码先给你们：[retrofit-demo](https://download.csdn.net/download/qq_48922459/83321037)

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 一、引入依赖
##  MAVEN

```xml
<dependency>
    <groupId>com.github.lianjiatech</groupId>
    <artifactId>retrofit-spring-boot-starter</artifactId>
    <version>2.3.8</version>
</dependency>
```

##  Gradle

```gradle
compile group: 'com.github.lianjiatech', name: 'retrofit-spring-boot-starter', version: '2.3.8'
```

<font color=#999AAA >这个工具尚不成熟，最好紧跟版本，使用最新版



# 二、使用步骤
## 1.在启动类上添加注解
这个路径是用书写 retrofit 请求方法的文件的包路径，下面我有介绍
```java
@RetrofitScan("com.example.retrofitdemo.retrofitinterface")
```

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Retrofit0.png)


## 2.项目解析
###  结构
<font color=#999AAA >entity、controller 和 service 都是普通的 entity、controller和service，`retrofit` 要创建的东西只有 `interceptor` 和 `retrofitInterceptor`

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Retrofit1.png)

###  各种请求方法的书写方式
<font color=#999AAA >retrofit 的请求方法有很多，除了普通请求，还支持 Restful 风格，我在 RetrofitTest 里面几乎把所有请求方式的demo都写了，并且有详细的注释


```java
package com.example.retrofitdemo.retrofitinterface;

import com.example.retrofitdemo.entity.TestEntity;
import com.example.retrofitdemo.interceptor.TokenInterceptor;
import com.github.lianjiatech.retrofit.spring.boot.annotation.Intercept;
import com.github.lianjiatech.retrofit.spring.boot.annotation.RetrofitClient;
import okhttp3.MultipartBody;
import okhttp3.Response;
import org.springframework.stereotype.Component;
import retrofit2.http.*;

import java.util.List;
import java.util.Map;

@Component
//路径前缀，必须以 / 结尾
@RetrofitClient(baseUrl = "http://localhost:8081/")
//include里面是要拦截的路径，还可配置exclude = ""不包含路径
@Intercept(handler = TokenInterceptor.class, include = "test/**")
public interface RetrofitTest {

    /**
     *方法注解：@GET、@POST、@PUT、@DELETE、@PATH、@HEAD、@OPTIONS、@HTTP。
     * 标记注解：@FormUrlEncoded、@Multipart、@Streaming。
     * 参数注解：@Query，@QueryMap；@Body；@Field、@FieldMap；@Part，@PartMap。
     * 其他注解：@Path；@Header,@Headers；@Url
     *
     * 下面是一些使用示例
     */

    //发送 Get 请求时，不可传值实体类，会报错
    //传送实体类的时候必须加 @Body
    @POST("test/postList")
    List<TestEntity> postList(@Body TestEntity testEntity);

    @GET("test/getList")
    List<TestEntity> getList(@Query("pageNum") Integer pageNum, @Query("pageSize") Integer pageSize);

    @GET("test/{id}")
    List<TestEntity> getList2(@Path("id") Long id);

    //!!!!!
    //注意
    //注意，没有返回值，用void会启动报错，要用大写的 Void 才不会报错
    @GET("test/delete/{id}")
    Void delete(@Path("id") Long id);

    @POST("test/add")
    Void add(@Body TestEntity testEntity);

    @POST("test/update/{id}")
    Void update(@Path("id") Long id, @Body TestEntity testEntity);

    //@FormUrlEncoded 代表发送表单数据
    //@Field会拼接name和age为 name1=age1&name2=age2 这种格式
    @FormUrlEncoded
    @POST("test")
    Void postFrom(@Field("name") String name,@Field("occupation") String age);

    //还有一个@FieldMap
    @FormUrlEncoded
    @POST("test")
    Void postMap(@FieldMap Map<String, String> fields);

    //@Header注解:作用于方法的参数,用于添加请求头，使用该注解定义的请求头可以为空,当为空时,会自动忽略,当传入一个List或array时,为拼接每个非空的item的值到请求头中
    //可作用在参数上
    @GET("test")
    Void header(@Header("Accept-Language") String head);

    //还有一个@HeaderMap：以map的方式添加多个请求头,map中的key为请求头的名称,value为请求头的值,且value使用String.valueOf()统一转换为String类型，
    //map中每一项的键和值都不能为空,否则抛出IllegalArgumentException异常
    @GET("test")
    Void headers(@HeaderMap Map<String, String> headers);

    //也可作用于方法上,具有相同名称的请求头不会相互覆盖,而是会照样添加到请求头中
    @Headers("Accept: text/plain")
    @GET("/")
    Void headers2();

    //HTTP注解：作用于方法,用于发送一个自定义的HTTP请求
    //发送一个DELETE请求,发送实体类要加hasBody
    @HTTP(method = "DELETE", path = "test/delete", hasBody = true)
    Void delete2(@Body TestEntity testEntity);

    //也可以自定义HTTP请求的标准样式
    @HTTP(method = "CUSTOM", path = "test/")
    Void custom();

    //@HTTP(method = "DELETE"）等价于 @DELETE("/")，@HTTP(method = "PUT"）等价于 @PUT("/")

    //Multipart注解：使用该注解,表示请求体是多部分的。 每一部分作为一个参数,且用Part注解声明

    //使用Part注解定义的参数类型有以下3种方式可选:
    //1, 如果类型是okhttp3.MultipartBody.Part，内容将被直接使用。 省略part中的名称,即 @Part MultipartBody.Part part
    //2, 如果类型是RequestBody，那么该值将直接与其内容类型一起使用。 在注释中提供part名称（例如，@Part（“foo”）RequestBody foo）。
    //3, 其他对象类型将通过使用转换器转换为适当的格式。 在注释中提供part名称（例如，@Part（“foo”）Image photo）
    @Multipart@POST("/")
    Void multipartExample(@Part("description") String description, @Part(value = "image", encoding = "8-bit") TestEntity image, @PartMap Map<String, String> params);

    //下面演示一下上传文件和下载文件，serviceImpl 的写法有些不一样
    @POST("test/upload")
    @Multipart
    Void upload(@Part MultipartBody.Part file);

    @GET("test/{fileKey}")
    Response download(@Path("fileKey") String fileKey);

}

```


###  在 ServiceImpl 中调用
在 `service` 中注入上面的类，然后调用里面的方法，就可以发送请求了

<font color=#999AAA >使用 retrofit 发送上传和下载请求时，serviceImpl 要做一些处理

```java
package com.example.retrofitdemo.service.impl;

import com.example.retrofitdemo.entity.TestEntity;
import com.example.retrofitdemo.retrofitinterface.RetrofitTest;
import com.example.retrofitdemo.service.TestService;
import okhttp3.MediaType;
import okhttp3.MultipartBody;
import okhttp3.Response;
import okhttp3.ResponseBody;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.net.URLEncoder;
import java.util.Objects;
import java.util.UUID;

@Service
public class TestServiceImpl implements TestService {
    //注入 retrofitTest
    @Autowired
    private RetrofitTest retrofitTest;

    @Override
    public void Test() {
        TestEntity testEntity = new TestEntity();
        testEntity.setPageNum(1);
        testEntity.setPageQuantity(5);
        System.out.println(retrofitTest.postList(testEntity));
    }

    //这是上传文件的代码，需对文件名使用URLEncoder进行编码
    public void upload(MultipartFile file) throws IOException {
        String fileName = URLEncoder.encode(Objects.requireNonNull(file.getOriginalFilename()),"utf-8");
        okhttp3.RequestBody requestBody=okhttp3.RequestBody.create(MediaType.parse("multipart/form-data"),file.getBytes());
        MultipartBody.Part part = MultipartBody.Part.createFormData("file",fileName,requestBody);
        //处理完成，调用请求
        retrofitTest.upload(part);
    }

    //这是下载文件的代码
    public void download() throws Exception {
        String fileKey = "6302d742-ebc8-4649-95cf-62ccf57a1add";
        //调用请求
        Response response = retrofitTest.download(fileKey);
        ResponseBody responseBody = response.body();
        // 二进制流
        InputStream is = responseBody.byteStream();

        // 具体如何处理二进制流，由业务自行控制。这里以写入文件为例
        File tempDirectory = new File("temp");
        if (!tempDirectory.exists()) {
            tempDirectory.mkdir();
        }
        File file = new File(tempDirectory, UUID.randomUUID().toString());
        if (!file.exists()) {
            file.createNewFile();
        }
        FileOutputStream fos = new FileOutputStream(file);
        byte[] b = new byte[1024];
        int length;
        while ((length = is.read(b)) > 0) {
            fos.write(b, 0, length);
        }
        is.close();
        fos.close();
    }

}

```

###  按需配置拦截器
<font color=#999AAA >这个拦截器的功能是请求前在请求中加入 token 的功能，可以按需创建，也可以不创建，拦截器分为局部拦截器和全局拦截器，我在TokenInterceptor 里写了很详细的注释。

```java
package com.example.retrofitdemo.interceptor;

import com.github.lianjiatech.retrofit.spring.boot.interceptor.BasePathMatchInterceptor;
import okhttp3.Request;
import okhttp3.Response;
import org.springframework.stereotype.Component;

import java.io.IOException;

/**
 * 这个拦截器是局部拦截器，可以在请求之前加入一些东西，比如token
 * 调用拦截器，在 RetrofitTest 里面用@Intercept注解配置拦截器和拦截路径
 *
 * 如果要创建全局拦截器，把 extends 后面换成 BaseGlobalInterceptor 就可以了，这样不需要加注解，全局生效
 */
@Component
public class TokenInterceptor extends BasePathMatchInterceptor {
    @Override
    protected Response doIntercept(Chain chain) throws IOException {
        String token = "加入这是一个token";
        Request request = chain.request();
        if (!"".equals(token)) {
            request = request.newBuilder()
                    .header("Authorization", token)
                    .build();
        }
        return chain.proceed(request);
    }
}

```

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

#  进阶配置文件
<font color=#999AAA >retrofit 下面的配置可选，不配置也不影响使用

```yaml
server:
  port: 8080
# 下面配置可选，不配置也不影响使用
retrofit:
  # 连接池配置
  pool:
    # test1连接池配置
    test1:
      # 最大空闲连接数
      max-idle-connections: 3
      # 连接保活时间(秒)
      keep-alive-second: 100

  # 是否禁用void返回值类型
  disable-void-return-type: false

  # 全局转换器工厂
  global-converter-factories:
    - com.github.lianjiatech.retrofit.spring.boot.core.BasicTypeConverterFactory
    - retrofit2.converter.jackson.JacksonConverterFactory
  # 全局调用适配器工厂
  global-call-adapter-factories:
    - com.github.lianjiatech.retrofit.spring.boot.core.BodyCallAdapterFactory
    - com.github.lianjiatech.retrofit.spring.boot.core.ResponseCallAdapterFactory

  # 日志打印配置
  log:
    # 启用日志打印
    enable: true
    # 日志打印拦截器
    logging-interceptor: com.github.lianjiatech.retrofit.spring.boot.interceptor.DefaultLoggingInterceptor
    # 全局日志打印级别
    global-log-level: info
    # 全局日志打印策略
    global-log-strategy: body


  # 重试配置
  retry:
    # 是否启用全局重试
    enable-global-retry: true
    # 全局重试间隔时间
    global-interval-ms: 1
    # 全局最大重试次数
    global-max-retries: 1
    # 全局重试规则
    global-retry-rules:
      - response_status_not_2xx
      - occur_io_exception
    # 重试拦截器
    retry-interceptor: com.github.lianjiatech.retrofit.spring.boot.retry.DefaultRetryInterceptor

  # 熔断降级配置
  degrade:
    # 是否启用熔断降级
    enable: true
    # 熔断降级实现方式
    degrade-type: sentinel
    # 熔断资源名称解析器
    resource-name-parser: com.github.lianjiatech.retrofit.spring.boot.degrade.DefaultResourceNameParser
  # 全局连接超时时间，还有局部的，在@RetrofitClient注解上可以设置超时时间，针对当前接口生效，优先级更高。具体字段有connectTimeoutMs、readTimeoutMs、writeTimeoutMs、callTimeoutMs等
  global-connect-timeout-ms: 5000
  # 全局读取超时时间
  global-read-timeout-ms: 5000
  # 全局写入超时时间
  global-write-timeout-ms: 5000
  # 全局完整调用超时时间
  global-call-timeout-ms: 0
```

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 总结

<font color=#999AAA >还可以请求微服务，不过我不懂，更多详细的用法可以看 gitee 的详解 https://gitee.com/lianjiatech/retrofit-spring-boot-starter

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">
