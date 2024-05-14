---
title: Springboot 配置线程池创建线程和配置 @Async 异步操作线程池
date: 2022-09-20 15:34:14
categories:
  - Linux
tags:
  - SpringBoot
  - 异步
  - 多线程
  - 线程池
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/springbootLogo.jpeg
---
# 前言
> 众所周知，创建显示线程和直接使用未配置的线程池创建线程，都会被阿里的大佬给diss，所以我们要规范的创建线程。
>  <br>
> ![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/ThreadPool0.png)
> ![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/ThreadPool1.png)
> ![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/ThreadPool2.png)



<br>

> 至于 @Async 异步任务的用处是不想等待方法执行完就返回结果，提高软件前台响应速度，一个程序中会用到很多异步方法，所以需要使用线程池管理，防止影响性能。



---

# 一、创建一个Springboot Web项目
`需要一个Springboot项目`
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/ThreadPool3.png)


# 二、新建ThreadPoolConfig
> /**
* 可以直接return一个内置线程池
* Executors类创建线程池的方法归根结底都是调用ThreadPoolExecutor类，只不过对每个方法赋值不同的参数去构造ThreadPoolExecutor对象。
* newCachedThreadPool:创建一个可缓存的线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
* newFixedThreadPool: 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待
* newScheduledThreadPool: 创建一个定长线程池，支持定时及周期性任务执行。
* newSingleThreadExecutor: 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。
*
* 也可以自己new一个ThreadPoolExecutor自定义参数
* 参数说明：
* corePoolSize: 常驻核心线程数，如果大于0，即使本地任务执行完也不会被销毁
* maximumPoolSize： 线程池能够容纳可同时执行的最大线程数
* keepAliveTime: 线程池中线程空闲的时间，当空闲时间达到该值时，线程会被销毁， 只剩下 corePoolSize 个线程数量。
* unit： 空闲时间的单位。一般以TimeUnit类定义时分秒。
* workQueue: 当请求的线程数大于 corePoolSize 时，线程进入该阻塞队列。
* threadFactory: 线程工厂，用来生产一组相同任务的线程，同时也可以通过它增加前缀名，虚拟机栈分析时更清晰
* handler: 执行拒绝策略，当 workQueue 已满，且超过maximumPoolSize 最大值，就要通过这个来处理，比如拒绝，丢弃等，这是一种限流的保护措施。
*
* 阻塞队列的实现类
* LinkedBlockingQueue 无界队列，当不指定队列大小时，将会默认为Integer.MAX_VALUE大小的队列，因此大量的任务将会堆积在队列中，最终可能触发OOM。
* ArrayBlockingQueue 有界队列，基于数组的先进先出队列，此队列创建时必须指定大小。
* PriorityBlockingQueue 有界队列，基于优先级任务的，它是通过Comparator决定的。
* SynchronousQueue 这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务
*
* 处理策略Handler：
* AbortPolicy 默认的拒绝策略，抛RejectedExecutionException异常
* DiscardPolicy 相当大胆的策略，直接丢弃任务，没有任何异常抛出
* DiscardOldestPolicy 丢弃最老的任务，其实就是把最早进入工作队列的任务丢弃，然后把新任务加入到工作队列
* CallerRunsPolicy 提交任务的线程自己去执行该任务
*
* 线程池的关闭
* shutdown() : 不会立刻终止线程，等所有缓存队列中的任务都执行完毕后才会终止。
* shutdownNow() : 立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务
*/

```java
package com.xuyijie.threadpooldemo.config;

import org.springframework.boot.SpringBootConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.*;

/**
 * @author 徐一杰
 * @date 2022/9/20 13:05
 * @description 配置线程池
 */
@SpringBootConfiguration
@EnableAsync
public class ThreadPoolConfig {

    @Bean
    public ExecutorService getThreadPool(){
        ExecutorService threadPool = new ThreadPoolExecutor(2,5,
                1L, TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy());
        return threadPool;
//        return Executors.newCachedThreadPool();
    }

    /**
     * 下面的配置是配置Springboot的@Async注解所用的线程池
     */
    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        //获取到cpu内核数
        int i = Runtime.getRuntime().availableProcessors();
        // 设置线程池核心容量
        executor.setCorePoolSize(i);
        // 设置线程池最大容量
        executor.setMaxPoolSize(i * 2);
        // 设置任务队列长度
        executor.setQueueCapacity(200);
        // 设置线程超时时间
        executor.setKeepAliveSeconds(60);
        // 设置线程名称前缀
        executor.setThreadNamePrefix("xyjAsyncPool-");
        // 设置任务丢弃后的处理策略,当poolSize已达到maxPoolSize，如何处理新任务（是拒绝还是交由其它线程处理）,CallerRunsPolicy：不在新线程中执行任务，而是由调用者所在的线程来执
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }

}

```
>/**
* 使用此线程池时直接注入ExecutorService，然后
* executorService.execute(() -> {
*             try {
*                 String message = redisUtil.listRightPop("queue:queueData", 5, TimeUnit.SECONDS);
*                 System.out.println("接收到了消息message" + message);
*             } catch (Exception ex) {
*                 date.set(simpleDateFormat.format(new Date()));
*                 System.out.println("队列阻塞超时-" + date + ex.getMessage());
*             } finally {
*                 date.set(simpleDateFormat.format(new Date()));
*                 System.out.println("线程销毁-" + date);
*                 executorService.shutdown();
*             }
*         });
*/

---

# 三、新建controller测试

```java
package com.xuyijie.threadpooldemo.controller;

import com.xuyijie.threadpooldemo.async.AsyncMethod;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.ExecutorService;

/**
 * @author 徐一杰
 * @date 2022/9/20 10:30
 * @description
 */
@RestController
@RequestMapping("/test")
public class TestController {

    @Autowired
    private ExecutorService executorService;
    @Autowired
    private AsyncMethod asyncMethod;
    
    @GetMapping("/helloThread")
    public void helloThread(){
        executorService.execute(() -> {
            for (int i = 0;i < 100;i++){
                System.out.println("111");
            }
        });
        executorService.execute(() -> {
            for (int i = 0;i < 100;i++){
                System.out.println("222");
            }
        });
    }
	
	@GetMapping("/helloAsync")
    public String helloAsync(){
    	// 这个方法是异步的
        asyncMethod.print();
        System.out.println("print方法还在循环，但我已经可以执行了");
        return "print方法还在循环，但我已经可以执行了";
    }

}

```
`AsyncMethod.java`

```java
package com.xuyijie.threadpooldemo.async;

import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Component;

/**
 * @author 徐一杰
 * @date 2022/9/20 15:10
 * @description 异步方法
 */
@Component
public class AsyncMethod {

	/**
     * 异步方法，如果@Async加在类的上面，则整个类中的方法都是异步的
     */
    @Async
    public void print(){
        for (int i = 0;i < 100;i++){
            System.out.println(i);
        }
    }
}

```



# 四、演示结果
> 首先演示 helloThread 这个接口，创建了2个线程，发现他们并发执行，成功
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/ThreadPool4.png)
<br>
> 再演示 helloAsync 这个接口，发现 `System.out.println("print方法还在循环，但我已经可以执行了");`这行代码无需等待上面AsyncMethod中的 print 方法执行完毕，就可以开始执行，说明 print 方法是异步的，而且我输出的日志注意看，`[xyjAsyncPool - ]`，我设置的线程池前缀，已经生效了，成功
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/ThreadPool5.png)



# 总结

