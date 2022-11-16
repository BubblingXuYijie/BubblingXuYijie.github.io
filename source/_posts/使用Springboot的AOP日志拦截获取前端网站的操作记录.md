---
title: 使用Springboot的AOP日志拦截获取前端网站的操作记录
date: 2021-12-08 21:50:41
categories: Java
tags:
    - SpringBoot
cover: /img/springbootLogo.jpeg
---
# 前言

<font color=#999AAA >随着我们的不断学习，我们的技术不断沉淀，做出来的项目也不断成熟</font>

<font color=#999AAA >所以，我们的网站怎么能没有日志记录呢</font>



<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 1、创建Springboot Web项目并添加依赖


<font color=#999AAA >选择左边那个 Spring Initializr 来创建
![在这里插入图片描述](https://img-blog.csdnimg.cn/b3e3c4b42d5a4ad38915a6cdb760bddb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_19,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >这一步是选择项目要用到的依赖，勾选以后就不用配置 Maven 的 pom 了，当然这里面都是些常用依赖，里面没有的还是要手动添加 pom，Springboot 的Web项目选择Spring Web就行了，根据需要选择

![在这里插入图片描述](https://img-blog.csdnimg.cn/ad4cb1031cac46ccb917cffecae10765.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_19,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >完成后我们向 ==pom.xml== 中添加一条依赖，用于日志的拦截和输出

```xml
	<!--aop-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
```


<font color=#999AAA >先建立项目结构， annotation 和 aop 就是用于日志拦截的文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/655b405402bd4867bc373b926316eb26.png)

---

# 2、新建保存日志实体类（非必须）

<font color=#999AAA >我们先新建一个实体类来存储我们拦截到的日志，如果不想存储就不用新建这个了，直接把日志打印到控制台就行。

```java
import lombok.Data;

import java.util.Date;

/**
 * @author 徐一杰
 * @date 2022/10/17
 * 存储操作日志的实体类
 */
@Data
public class LogSave {
    /**
     * 操作模块
     */
    private String operationModule;
    /**
     * 操作类型
     */
    private String operationType;
    /**
     * 具体操作
     */
    private String operationDescription;
    /**
     * 请求的接口地址
     */
    private String requestUrl;
    /**
     * 请求方法，get/post
     */
    private String requestMethod;
    /**
     * 请求的后台接口
     */
    private String requestInterface;
    /**
     * 请求参数
     */
    private String requestParam;
    /**
     * 返回值
     */
    private String responseResult;
    /**
     * 方法执行时长
     */
    private long operationPeriod;
    /**
     * 操作时间
     */
    private Date operationTime;
}

```

# 3、简简单单写个Controller

<font color=#999AAA >`controller` 里面很简单，就是在网页打印一句话，然后加上我们自定义的 `@Log` 注解，@Log代码在下面

```java
import com.xuyijie.aoplogintercept.annotation.Log;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author 徐一杰
 * @date 2022/10/17
 */
@RestController
public class TestController {
    /**
     * log里自定义的内容，用于存入数据库对这个方法的日志进行描述
     */
    @Log(operationModule = "主题", operationType = "操作类型：查询/删除", operationDescription = "具体操作：删除了文件")
    @RequestMapping("/log")
    public String log(){
        return "这是自定义的返回结果";
    }
}
```
<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 4、自定义注解


<font color=#999AAA >`annotation包` 下面的 `Log`，是自定义注解，字段的解释下面的注释中很清晰哦

```java
import java.lang.annotation.*;

/**
 * @author 徐一杰
 * @date 2021/11/24
 * Aop日志拦截自定义注解
 */
@Documented
@Target(ElementType.METHOD) //注解放置的目标位置即方法级别，也就是注解放在controller里的方法上面
@Retention(RetentionPolicy.RUNTIME) //注解在哪个阶段执行
public @interface Log {
    //写在注解里面，可以用于存到数据库时的对方法的描述，可以写多个
    String operationModule() default ""; // 操作模块

    String operationType() default "";  // 操作类型

    String operationDescription() default "";  // 操作说明
}
//@Aspect 面向切面编程注解，通常应用在类上
//@Pointcut Pointcut是植入Advice的触发条件。每个Pointcut的定义包括2部分，一是表达式，二是方法签名。方法签名必须是 public及void型。可以将Pointcut中的方法看作是一个被Advice引用的助记符，因为表达式不直观，因此我们可以通过方法签名的方式为 此表达式命名。因此Pointcut中的方法只需要方法签名，而不需要在方法体内编写实际代码
//@Around：环绕增强，相当于MethodInterceptor
//@AfterReturning：后置增强，相当于AfterReturningAdvice，方法正常退出时执行
//@Before：标识一个前置增强方法，相当于BeforeAdvice的功能，相似功能的还有
//@AfterThrowing：异常抛出增强，相当于ThrowsAdvice
//@After: final增强，不管是抛出异常或者正常退出都会执行
```

# 5、开始拦截

<font color=#999AAA >`aop包` 下面的` LogAspect `， `@Pointcut` 里面的包名修改成你们自己的

```java
import com.xuyijie.aoplogintercept.annotation.Log;
import entity.LogSave;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.lang.reflect.Method;
import java.util.Arrays;

/**
 * @author 徐一杰
 * @date 2021/11/24
 * Aop日志拦截，并保存日志到数据库，如果不想存储数据库，直接打印到控制台也行
 */
@Aspect
@Component
public class LogAspect {

    private LogSave logSave = new LogSave();

    /**
     * @annotation: 指定 @Log 注解位置，这里 @annotation 里的配置代表只拦截增加了自定义注解 @Log 的方法
     *
     * execution：指定 controller 包下的注解，.*代表controller包下所有文件，*(..)代表所有方法
     * 这里execution是拦截全部controller接口的意思，如果只想拦截增加了自定义注解的方法，使用@annotation
     */
    // @Pointcut("execution(* com.xuyijie.aoplogintercept.controller.*.*(..))")
    @Pointcut("@annotation(com.xuyijie.aoplogintercept.annotation.Log)")
    public void logPointCut() {

    }

    /**
     * 指定当前执行方法在logPointCut之前执行
     */
    @Before("logPointCut()")
    public void doBefore(JoinPoint joinPoint) {
        // 接收到请求，记录请求内容
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        assert attributes != null;
        HttpServletRequest request = attributes.getRequest();
        // 记录下请求内容
        logSave = new LogSave();
        logSave.setRequestUrl(request.getRequestURI());
        logSave.setRequestMethod(request.getMethod());
        logSave.setRequestInterface(joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName());
        logSave.setRequestParam(Arrays.toString(joinPoint.getArgs()));
        // 从切面织入点处通过反射机制获取织入点处的方法
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        //获取切入点所在的方法
        Method method = signature.getMethod();
        //获取操作，也就是前面 OperationLogAnnotation 里定义的
        Log logAnnotation = method.getAnnotation(Log.class);
        logSave.setOperationModule(logAnnotation.operationModule());
        logSave.setOperationType(logAnnotation.operationType());
        logSave.setOperationDescription(logAnnotation.operationDescription());
    }

    /**
     * 指定在方法之后返回
     * returning的值和doAfterReturning的参数名一致
     */
    @AfterReturning(returning = "ret", pointcut = "logPointCut()")
    public void doAfterReturning(Object ret) {
        logSave.setResponseResult(ret.toString());
    }

    @Around("logPointCut()")
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable {
        long startTime = System.currentTimeMillis();
        // ob 为方法的返回值
        Object ob = pjp.proceed();
        logSave.setOperationPeriod(System.currentTimeMillis() - startTime);
        //TODO 存储日志到数据库
        // LogSaveDao.save(log);
        return ob;
    }

}
```

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 6、运行项目
<font color=#999AAA >启动项目


<font color=#999AAA >然后在浏览器的地址栏里输入 localhost:8080/log/?name=啦啦啦 ，回车访问

<font color=#999AAA >我没有配置 application.properties 文件，所以端口默认8080，/log是controller里面方法上面配置的请求路径，?name=啦啦啦 ，name 是 controller 里面方法的传参。


![在这里插入图片描述](https://img-blog.csdnimg.cn/9a8c4fe91c50448dac165f0cd04d4f80.png)

<font color=#999AAA >你们自己在在 `LogAspect` 里面添加一点打印语句，控制台打印出拦截结果，请求地址、请求方法、传参 等等


![在这里插入图片描述](https://img-blog.csdnimg.cn/9317421854874cb987e050aa473bbf4e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

#  7、代码解析

<font color=#999AAA >首先是各注解的意思


```java
@Aspect 面向切面编程注解，通常应用在类上
@Pointcut Pointcut是植入Advice的触发条件。每个Pointcut的定义包括2部分，一是表达式，二是方法签名。方法签名必须是 public及void型。可以将Pointcut中的方法看作是一个被Advice引用的助记符，因为表达式不直观，因此我们可以通过方法签名的方式为 此表达式命名。因此Pointcut中的方法只需要方法签名，而不需要在方法体内编写实际代码
@Around：环绕增强，相当于MethodInterceptor
@AfterReturning：后置增强，相当于AfterReturningAdvice，方法正常退出时执行
@Before：标识一个前置增强方法，相当于BeforeAdvice的功能，相似功能的还有
@AfterThrowing：异常抛出增强，相当于ThrowsAdvice
@After: final增强，不管是抛出异常或者正常退出都会执行

然后是，LogAspect 里面的包路径配置，
@annotation: 指定 @Log 注解位置，这里 @annotation 里的配置代表只拦截增加了自定义注解 @Log 的方法，想要拦截哪个方法的日志，就在哪个方法上面加注解 `@Log()`
execution：指定 controller 包下的注解，.*代表controller包下所有文件，*(..)代表所有方法
这里execution是拦截全部controller接口的意思，如果只想拦截增加了自定义注解的方法，使用@annotation
```

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 总结


<font color=#999AAA >@Arround 是个万能注解，可以代替 @Before 和 @After，所以我在保存数据库的那个文件里只是用了 @Arround，保存到数据库的字段可以自己定制，下面是我另一个项目的日志，给你们参考一下。
![在这里插入图片描述](https://img-blog.csdnimg.cn/77b717e552e94421a00aae464b0a83fe.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

