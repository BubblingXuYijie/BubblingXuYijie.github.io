---
title: Springboot 配置使用 RabbitMQ 并实现延时队列
date: 2022-10-08 15:01:28
categories: Java
tags:
    - SpringBoot
    - RabbitMQ
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/RabbitMQ.jpg
---
# 前言

> `RabbitMQ作用`：举几个例子，1、系统解耦，A系统无需关心B系统是否执行成功，无需等待B系统响应，直接把操作扔给mq就可以干其他事情了。2、系统使用高峰期，每秒产生10000条消息需要存储，一次性存入数据库恐怕不太行，所以先把数据发送到 RabbitMQ ，然后设置延时队列，每秒从队列取出1000条存入数据库，这样可以减少数据库压力。3、购买商品下订单以后，发送到延时队列，如果20分钟后没有付款，则从队列删除订单，也就是自动取消订单，如果支付了，则取出存入数据库，下单成功。

---


# 一、安装 RabbitMQ
## Windows安装
> 太简单，自己bing一下

## Linux安装
> rabbitmq需要erlang语言环境
> 更新 apt 库，安装 erlang 环境，然后执行 `erl` 查看是否安装成功
```bash
apt update
apt install erlang
erl
```
> 安装 rabbitmq

```bash
apt install rabbitmq-server
```

> 查看 rabbitmq 运行状态

```bash
systemctl status rabbitmq-server
```

> 开启图形化管理界面，然后就可以访问 ip:15672，默认账号密码是 guest
```bash
rabbitmq-plugins enable rabbitmq_management
```
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SpringbootRabbitMQ0.png)
> 默认的guest用户是只能通过本机访问的，所以远程管理后台界面登录需要配置个用户，才能通过外网浏览器访问

```bash
#账号root,密码root
rabbitmqctl add_user root root
# 设置为管理员账户
rabbitmqctl set_user_tags root administrator
# 分配所有权限
rabbitmqctl set_permissions -p / root “.*” “.*” “.*”
```

> 开放防火墙 5672 和 15672 端口

```bash
# Debian/Ubuntu ufw
ufw allow 5672
ufw allow 15672
ufw reload
# Debian/Ubuntu iptables（这个叼毛防火墙好麻烦，我没用过，不知道是不是这样）
iptables -A INPUT -p tcp --dport 5672 -j ACCEPT
iptables -A INPUT -p tcp --dport 15672 -j ACCEPT
iptables-restore
# CentOS
firewall-cmd --zone=public --add-port=5672/tcp --permanent
firewall-cmd --zone=public --add-port=15672/tcp --permanent
firewall-cmd --reload
```



---

# 二、新建项目
> 新建一个 provider 一个 consumer，两个 springboot 项目，都需要引入下面的依赖，或者新建的时候勾选自动添加 rabbitmq 的依赖
> ![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SpringbootRabbitMQ1.png)

## 1、引入依赖

```xml
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit-test</artifactId>
    <scope>test</scope>
</dependency>
```

## 2、配置 yml

> provider 和 consumer 都这样配置，端口改成不一样就行了

```yaml
server:
  port: 8081
spring:
  rabbitmq:
    host: 192.168.0.105
    port: 5672
    username: root
    password: root
    virtualHost: /
    # 确认机制
    publisher-confirm-type: correlated
    # 发布确认，如果不配置确认机制，发布确认也不用配置
    publisher-returns: true

```

## 3、启动类开启 Rabbitmq 注解
> consumer 和 provider 都需要这个注解

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SpringbootRabbitMQ2.png)


## 4、配置 provider 的 RabbitmqConfig

> 大家可以根据 15672 那个图形化管理界面看看下面的一些概念
>  *  Broker:它提供一种传输服务,它的角色就是维护一条从生产者到消费者的路线，保证数据能按照指定的方式进行传输
> *  Exchange：消息交换机,它指定消息按什么规则,路由到哪个队列。
>  *  Queue:消息的载体,每个消息都会被投到一个或多个队列。
> *  Binding:绑定，它的作用就是把exchange和queue按照路由规则绑定起来.
> *  Routing Key:路由关键字,exchange根据这个关键字进行消息投递。
> *  vhost:虚拟主机,一个broker里可以有多个vhost，用作不同用户的权限分离。
> *  Producer:消息生产者,就是投递消息的程序.
> *  Consumer:消息消费者,就是接受消息的程序.
> *  Channel:消息通道,在客户端的每个连接里,可建立多个channel.

```java
package icu.xuyijie.provider.config;

import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Scope;
import org.springframework.web.filter.CharacterEncodingFilter;

import java.util.HashMap;
import java.util.Map;

/**
 * @author 徐一杰
 * @date 2022/9/30 9:38
 * @description
 */
@SpringBootConfiguration
public class RabbitmqConfig {

    public static final String QUEUE_MESSAGE = "queue_message";
    public static final String QUEUE_ORDER = "queue_order";
    public static final String EXCHANGE_A = "exchange_A";
    /**
     * # 是通配符，可以匹配任意，如下面的可以匹配到 aa.bb.message.cc.dd
     * 还有 * 是匹配一个.里面的字符，如 .*.message 只能匹配 .aa.message，不能匹配 .aa.bb.message
     */
    public static final String ROUTING_KEY_MESSAGE = "#.message.#";
    public static final String ROUTING_KEY_ORDER = "#.order.#";

    /**
     * 这个可以不写，写了的话，以这个为准，会覆盖掉 yml 的配置
     * @return
     */
    @Bean
    public ConnectionFactory connectionFactory() {
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory("192.168.0.105", 5672);
        connectionFactory.setUsername("root");
        connectionFactory.setPassword("root");
        connectionFactory.setVirtualHost("/");
        //确认机制
        connectionFactory.setPublisherConfirmType(CachingConnectionFactory.ConfirmType.CORRELATED);
        //发布确认，如果不配置确认机制，发布确认也不用配置
        connectionFactory.setPublisherReturns(true);
        return connectionFactory;
    }

    /**
     * 使用 yml 连接的话这个可以不写，这个是配合 connectionFactory 的
     * RabbitMQ的使用入口。scope必须是prototype类型
     * @return
     */
    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public RabbitTemplate rabbitTemplate() {
        RabbitTemplate template = new RabbitTemplate(this.connectionFactory());
        template.setMessageConverter(this.jsonMessageConverter());
        template.setMandatory(true);
        return template;
    }

    /**
     * 序列化json
     * @return
     */
    @Bean
    public MessageConverter jsonMessageConverter() {
        return new Jackson2JsonMessageConverter();
    }

    @Bean
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new CharacterEncodingFilter();
        filter.setEncoding("UTF-8");
        filter.setForceEncoding(true);
        return filter;
    }

    /**
     * 声明交换机
     *  针对消费者配置
     *  设置交换机类型
     *  将队列绑定到交换机
     *    FanoutExchange: 将消息分发到所有的绑定队列，无routing key的概念
     *    HeadersExchange：通过添加属性key-value匹配
     *    DirectExchange: 按照routing key分发到指定队列
     *    TopicExchange: 多关键字匹配
     * @return
     */
    @Bean
    public Exchange exchangeA(){
        //durable(true) 持久化，mq重启之后交换机还在
        return ExchangeBuilder.topicExchange(EXCHANGE_A).durable(true).build();
    }

    /**
     * 声明QUEUE_MESSAGE队列
     * durable:是否持久化,默认是false,持久化队列：会被存储在磁盘上，当消息代理重启时仍然存在，暂存队列：当前连接有效
     * exclusive:默认也是false，只能被当前创建的连接使用，而且当连接关闭后队列即被删除。此参考优先级高于durable
     * autoDelete:是否自动删除，当没有生产者或者消费者使用此队列，该队列会自动删除。
     * 一般设置一下队列的持久化就好,其余两个就是默认false
     * @return
     */
    @Bean
    public Queue queueMessage(){
        return new Queue(QUEUE_MESSAGE, true, false, false);
    }

    /**
     * 声明QUEUE_ORDER队列
     * @return
     */
    @Bean
    public Queue queueOrder(){
        return new Queue(QUEUE_ORDER);
    }

    /**
     * 队列绑定交换机，指定routingKey
     * @return
     */
    @Bean
    public Binding bindingQueueMessage(){
        return BindingBuilder.bind(queueMessage()).to(exchangeA()).with(ROUTING_KEY_MESSAGE).noargs();
    }

    /**
     * 队列绑定交换机，指定routingKey
     * @return
     */
    @Bean
    public Binding bindingQueueOrder(){
        return BindingBuilder.bind(queueOrder()).to(exchangeA()).with(ROUTING_KEY_ORDER).noargs();
    }

}

```

## 5、provider 发送消息

> 我们因为配置了确认机制，所以我们配置了回调方法，这里使用构造器注入 `rabbitTemplate`，如果不配置回调方法，则可以使用 `@Autowired` 注入，并且类无需实现 `RabbitTemplate.ConfirmCallback`，sendExchange 方法没有使用回调方法，使用回调方法的话需要像 sendCallback 方法一样多传一个值 correlationId

```java
package icu.xuyijie.provider.controller;

import icu.xuyijie.provider.config.RabbitmqConfig;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;
import java.util.concurrent.ExecutionException;

/**
 * @author 徐一杰
 * @date 2022/9/30 10:08
 * @description
 */
@RestController
@RequestMapping("/provider")
public class ProviderController implements RabbitTemplate.ConfirmCallback {

	/**
	 * 我们因为配置了确认机制，所以我们配置了回调方法，这里使用构造器注入 rabbitTemplate，如果不配置
	 * 回调方法，则可以使用 @Autowired 注入，并且类无需实现 RabbitTemplate.ConfirmCallback
	 */
    private final RabbitTemplate rabbitTemplate;

    public ProviderController(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
        //设置回调为当前类对象
        this.rabbitTemplate.setConfirmCallback(this);
    }

    @RequestMapping("/sendExchange")
    public void sendExchange(){
        //使用rabbitTemplate发送消息
        String message = "这是一条发送到exchangeA的消息";
        /**
         * 参数：
         * 1、交换机名称
         * 2、routingKey
         * 3、消息内容
         */
        rabbitTemplate.convertAndSend(RabbitmqConfig.EXCHANGE_A, "a.message", message);
    }
    
    /**
     * 如果使用回调方法，则需要多传一个参数 correlationId
     */
    @RequestMapping("/sendCallback")
    public void sendCallback(){
        String message = "这是一条发送到exchangeA的消息";
        //构建回调id为uuid
        String callBackId = UUID.randomUUID().toString();
        CorrelationData correlationId = new CorrelationData(callBackId);
        //发送消息到消息队列
        rabbitTemplate.convertAndSend(RabbitmqConfig.EXCHANGE_A, "a.message", message, correlationId);
        System.out.println("发送回调id: " + callBackId);
    }

    /**
     * 消息回调确认方法
     * @param correlationData 请求数据对象
     * @param ack 是否发送成功
     * @param s
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String s) {
        assert correlationData != null;
        System.out.println("这是回调方法打印的：回调id: " + correlationData.getId());
        try {
            System.out.println("这是回调方法打印的：回调message: " + correlationData.getFuture().get());
        } catch (InterruptedException | ExecutionException e) {
            throw new RuntimeException(e);
        }
        if (ack) {
            System.out.println("这是回调方法打印的：消息发送成功");
        } else {
            System.out.println("消息发送失败" + s);
        }
    }

}

```
## 6、consumer 接收消息

> `@RabbitListener`就是监听的队列，可以监听多个

```java
package icu.xuyijie.consumer.consumer;

import com.rabbitmq.client.Channel;
import icu.xuyijie.consumer.config.RabbitmqConfig;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * @author 徐一杰
 * @date 2022/9/30 10:10
 * @description
 */
@Component
public class ReceiveHandler {

    @RabbitListener(queues = {"queue_message", "queue_order"})
    public void receiveMessage(Message message, Channel channel){
        System.out.println("接收 queue_message 和 queue_order 队列的消息 " + message);
        System.out.println("对应Channel " + channel);
    }

    @RabbitListener(queues = {"queue_order"})
    public void receiveOrder(Message message, Channel channel){
        System.out.println("接收 queue_order  队列的消息" + message);
        System.out.println("对应Channel " + channel);
    }

}
```

## 7、演示
> 我们直接调用 sendCallback 这个接口

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SpringbootRabbitMQ3.png)
> consumer 接收到消息

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SpringbootRabbitMQ4.png)
> provider 触发回调方法

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SpringbootRabbitMQ5.png)

---


# 三、延时队列

## 1、在 provider 的 RabbitmqConfig 中增加配置

> 增加了一个一个交换机、一个队列、一个路由键的配置，注意 delayQueue() 方法，我的注释有解释

```java
	public static final String QUEUE_DELAY = "queue_delay";
    public static final String EXCHANGE_DELAY = "exchange_delay";
    public static final String ROUTER_DELAY_KEY = "router_delay_key";

    /**
     * 延迟交换机
     *
     * @return
     */
    @Bean
    public DirectExchange delayExchange() {
        return new DirectExchange(EXCHANGE_DELAY);
    }

    /**
     * 延迟队列
     * map 的设置意思是接收此队列的延迟消息需要监听 EXCHANGE_RECEIVE 队列，直接监听 QUEUE_DELAY 无法实现延时队列
     *
     * @return
     */
    @Bean
    public Queue delayQueue() {
        Map<String, Object> map = new HashMap<>(16);
        map.put("x-dead-letter-exchange", EXCHANGE_A);
        map.put("x-dead-letter-routing-key", ROUTING_KEY_ORDER);
        return new Queue(QUEUE_DELAY, true, false, false, map);
    }

    /**
     * 给延迟队列绑定交换机
     *
     * @return
     */
    @Bean
    public Binding delayBinding() {
        return BindingBuilder.bind(delayQueue()).to(delayExchange()).with(ROUTER_DELAY_KEY);
    }
```

## 2、在 provider 增加一个接口
> 发送消息到我们刚刚配置的延时交换机

```java
	/**
     * 给延迟队列发送消息
     */
    @RequestMapping("/sendDelay")
    public void sendDelay(){
        rabbitTemplate.convertAndSend(RabbitmqConfig.EXCHANGE_DELAY, RabbitmqConfig.ROUTER_DELAY_KEY, "这是一条延时队列的消息", message -> {
            message.getMessageProperties().setExpiration("3000");
            return message;
        }, new CorrelationData(UUID.randomUUID().toString()));
        System.out.println("延时队列发送成功");
    }
```

## 3、consumer 接收延时消息
> 注意，上面我们把延时消息发送到了延时队列 EXCHANGE_DELAY，但是我们接收，要监听 `RabbitmqConfig`中 delayQueue() 方法 配置的 EXCHANGE_A，路由键为 ROUTING_KEY_ORDER，也就是说无需改动 consumer ，consumer 的两个方法都能收到延时消息，因为他们都监听了 ROUTING_KEY_ORDER 对应的队列

直接调用 sendDelay 方法，2次（因为两个方法都监听 queue_order，所以他们会交替获得消息）

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SpringbootRabbitMQ6.png)
3秒后 consumer 的两个方法都能接收到延时消息

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SpringbootRabbitMQ7.png)



---

# 总结

