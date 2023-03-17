---
title: Springboot 配置使用 Kafka
date: 2022-11-01 17:44:41
categories: Java
tags:
    - SpringBoot
    - Kafka
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Kafka.jpg
---
# 前言
不多BB讲原理，只教你怎么用，看了全网没有比我更详细的了，yml 配置，Config 工厂代码配置都有，batch-size、acks、offset、auto-commit、trusted-packages、poll-timeout、linger 应有尽有，批量消费、开启事务、定义批量消费数量、延时发送、失败重试、异常处理你还想要什么

> As we all know，当今世界最流行的消息中间件有 RabbitMq、RocketMq、Kafka，其中，应用最广泛的是 `RabbitMq`，`RocketMq` 是阿里巴巴的产品，性能超过 RabbitMq，已经经受了多年的双11考验，但是怕哪天阿里不维护了，用的人不多，`Kafka` 是吞吐量最大的一个，远超前两个，支持事务、可保证消息的不丢失（网上说的事务和消息可靠性不支持是说的旧版，2以后就开始支持了），对比来讲，Kafka相对于前两个，只有一个劣势，不太支持延时队列，其他方面都要优于它们（个人使用体验，勿喷）。

---


# 一、Linux 安装 Kafka
我的另一篇文章：[Debian（Linux通用）安装 Kafka 并配置远程访问](https://blog.csdn.net/qq_48922459/article/details/127633676?spm=1001.2014.3001.5502)

---

# 二、构建项目

> 多模块项目构建，这里不讲，如果你不会，就新建两个普通的web项目 `KafkaConsumer`和 `KafkaProvider`就行

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SpringbootKafka0.png)


---

# 三、引入依赖

> 新建一个标准的spring-web项目，额外依赖真的只需要这一个，网上说的 kafka-client 不是springboot 的东西，那就是个原生的 kafka 客户端， kafka-test也不需要，这个是用代码控制broker的东西

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

---

# 四、配置文件
> 这两种方式的代码会互相覆盖，而且有些配置只能用 `config` 方式配置，建议像我一样，两种都写，`config`里面的配置参数从 `yml` 中获取，就可以不影响使用 Nacos 来在线修改 kafka 的配置了

## 生产者
`配置的意思详解在注释里面都有哦`

### yml 方式

```yaml
server:
  port: 8081
spring:
  kafka:
    producer:
      # Kafka服务器
      bootstrap-servers: 175.24.228.202:9092
      # 开启事务，必须在开启了事务的方法中发送，否则报错
      transaction-id-prefix: kafkaTx-
      # 发生错误后，消息重发的次数，开启事务必须设置大于0。
      retries: 3
      # acks=0 ： 生产者在成功写入消息之前不会等待任何来自服务器的响应。
      # acks=1 ： 只要集群的首领节点收到消息，生产者就会收到一个来自服务器成功响应。
      # acks=all ：只有当所有参与复制的节点全部收到消息时，生产者才会收到一个来自服务器的成功响应。
      # 开启事务时，必须设置为all
      acks: all
      # 当有多个消息需要被发送到同一个分区时，生产者会把它们放在同一个批次里。该参数指定了一个批次可以使用的内存大小，按照字节数计算。
      batch-size: 16384
      # 生产者内存缓冲区的大小。
      buffer-memory: 1024000
      # 键的序列化方式
      key-serializer: org.springframework.kafka.support.serializer.JsonSerializer
      # 值的序列化方式（建议使用Json，这种序列化方式可以无需额外配置传输实体类）
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
```

### Config 方式

```java
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;
import org.springframework.kafka.support.serializer.JsonSerializer;
import org.springframework.kafka.transaction.KafkaTransactionManager;

import java.util.HashMap;
import java.util.Map;

/**
 * @author 徐一杰
 * @date 2022/10/31 18:05
 * kafka配置，也可以写在yml，这个文件会覆盖yml
 */
@SpringBootConfiguration
public class KafkaProviderConfig {

    @Value("${spring.kafka.producer.bootstrap-servers}")
    private String bootstrapServers;
    @Value("${spring.kafka.producer.transaction-id-prefix}")
    private String transactionIdPrefix;
    @Value("${spring.kafka.producer.acks}")
    private String acks;
    @Value("${spring.kafka.producer.retries}")
    private String retries;
    @Value("${spring.kafka.producer.batch-size}")
    private String batchSize;
    @Value("${spring.kafka.producer.buffer-memory}")
    private String bufferMemory;

    @Bean
    public Map<String, Object> producerConfigs() {
        Map<String, Object> props = new HashMap<>(16);
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        //acks=0 ： 生产者在成功写入消息之前不会等待任何来自服务器的响应。
        //acks=1 ： 只要集群的首领节点收到消息，生产者就会收到一个来自服务器成功响应。
        //acks=all ：只有当所有参与复制的节点全部收到消息时，生产者才会收到一个来自服务器的成功响应。
        //开启事务必须设为all
        props.put(ProducerConfig.ACKS_CONFIG, acks);
        //发生错误后，消息重发的次数，开启事务必须大于0
        props.put(ProducerConfig.RETRIES_CONFIG, retries);
        //当多个消息发送到相同分区时,生产者会将消息打包到一起,以减少请求交互. 而不是一条条发送
        //批次的大小可以通过batch.size 参数设置.默认是16KB
        //较小的批次大小有可能降低吞吐量（批次大小为0则完全禁用批处理）。
        //比如说，kafka里的消息5秒钟Batch才凑满了16KB，才能发送出去。那这些消息的延迟就是5秒钟
        //实测batchSize这个参数没有用
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, batchSize);
        //有的时刻消息比较少,过了很久,比如5min也没有凑够16KB,这样延时就很大,所以需要一个参数. 再设置一个时间,到了这个时间,
        //即使数据没达到16KB,也将这个批次发送出去
        props.put(ProducerConfig.LINGER_MS_CONFIG, "5000");
        //生产者内存缓冲区的大小
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, bufferMemory);
        //反序列化，和生产者的序列化方式对应
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        return props;
    }

    @Bean
    public ProducerFactory<Object, Object> producerFactory() {
        DefaultKafkaProducerFactory<Object, Object> factory = new DefaultKafkaProducerFactory<>(producerConfigs());
        //开启事务，会导致 LINGER_MS_CONFIG 配置失效
        factory.setTransactionIdPrefix(transactionIdPrefix);
        return factory;
    }

    @Bean
    public KafkaTransactionManager<Object, Object> kafkaTransactionManager(ProducerFactory<Object, Object> producerFactory) {
        return new KafkaTransactionManager<>(producerFactory);
    }

    @Bean
    public KafkaTemplate<Object, Object> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```


## 消费者

### yml 方式

```yaml
server:
  port: 8082
spring:
  kafka:
    consumer:
      # Kafka服务器
      bootstrap-servers: 175.24.228.202:9092
      group-id: firstGroup
      # 自动提交的时间间隔 在spring boot 2.X 版本中这里采用的是值的类型为Duration 需要符合特定的格式，如1S,1M,2H,5D
      #auto-commit-interval: 2s
      # 该属性指定了消费者在读取一个没有偏移量的分区或者偏移量无效的情况下该作何处理：
      # earliest：当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费分区的记录
      # latest：当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据（在消费者启动之后生成的记录）
      # none：当各分区都存在已提交的offset时，从提交的offset开始消费；只要有一个分区不存在已提交的offset，则抛出异常
      auto-offset-reset: latest
      # 是否自动提交偏移量，默认值是true，为了避免出现重复数据和数据丢失，可以把它设置为false，然后手动提交偏移量
      enable-auto-commit: false
      # 键的反序列化方式
      #key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      key-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      # 值的反序列化方式（建议使用Json，这种序列化方式可以无需额外配置传输实体类）
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      # 配置消费者的 Json 反序列化的可信赖包，反序列化实体类需要
      properties:
        spring:
          json:
            trusted:
              packages: "*"
      # 这个参数定义了poll方法最多可以拉取多少条消息，默认值为500。如果在拉取消息的时候新消息不足500条，那有多少返回多少；如果超过500条，每次只返回500。
      # 这个默认值在有些场景下太大，有些场景很难保证能够在5min内处理完500条消息，
      # 如果消费者无法在5分钟内处理完500条消息的话就会触发reBalance,
      # 然后这批消息会被分配到另一个消费者中，还是会处理不完，这样这批消息就永远也处理不完。
      # 要避免出现上述问题，提前评估好处理一条消息最长需要多少时间，然后覆盖默认的max.poll.records参数
      # 注：需要开启BatchListener批量监听才会生效，如果不开启BatchListener则不会出现reBalance情况
      max-poll-records: 3
    properties:
      # 两次poll之间的最大间隔，默认值为5分钟。如果超过这个间隔会触发reBalance
      max:
        poll:
          interval:
            ms: 600000
      # 当broker多久没有收到consumer的心跳请求后就触发reBalance，默认值是10s
      session:
        timeout:
          ms: 10000
    listener:
      # 在侦听器容器中运行的线程数，一般设置为 机器数*分区数
      concurrency: 4
      # 自动提交关闭，需要设置手动消息确认
      ack-mode: manual_immediate
      # 消费监听接口监听的主题不存在时，默认会报错，所以设置为false忽略错误
      missing-topics-fatal: false
      # 两次poll之间的最大间隔，默认值为5分钟。如果超过这个间隔会触发reBalance
      poll-timeout: 600000
```

### Config 方式

```java
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.listener.ConcurrentMessageListenerContainer;
import org.springframework.kafka.listener.ContainerProperties;
import org.springframework.kafka.support.serializer.JsonDeserializer;

import java.util.HashMap;
import java.util.Map;

/**
 * @author 徐一杰
 * @date 2022/10/31 18:05
 * kafka配置，也可以写在yml，这个文件会覆盖yml
 */
@SpringBootConfiguration
public class KafkaConsumerConfig {

    @Value("${spring.kafka.consumer.bootstrap-servers}")
    private String bootstrapServers;
    @Value("${spring.kafka.consumer.group-id}")
    private String groupId;
    @Value("${spring.kafka.consumer.enable-auto-commit}")
    private boolean enableAutoCommit;
    @Value("${spring.kafka.properties.session.timeout.ms}")
    private String sessionTimeout;
    @Value("${spring.kafka.properties.max.poll.interval.ms}")
    private String maxPollIntervalTime;
    @Value("${spring.kafka.consumer.max-poll-records}")
    private String maxPollRecords;
    @Value("${spring.kafka.consumer.auto-offset-reset}")
    private String autoOffsetReset;
    @Value("${spring.kafka.listener.concurrency}")
    private Integer concurrency;
    @Value("${spring.kafka.listener.missing-topics-fatal}")
    private boolean missingTopicsFatal;
    @Value("${spring.kafka.listener.poll-timeout}")
    private long pollTimeout;

    @Bean
    public Map<String, Object> consumerConfigs() {
        Map<String, Object> propsMap = new HashMap<>(16);
        propsMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        propsMap.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
        //是否自动提交偏移量，默认值是true，为了避免出现重复数据和数据丢失，可以把它设置为false，然后手动提交偏移量
        propsMap.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, enableAutoCommit);
        //自动提交的时间间隔，自动提交开启时生效
        propsMap.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "2000");
        //该属性指定了消费者在读取一个没有偏移量的分区或者偏移量无效的情况下该作何处理：
        //earliest：当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费分区的记录
        //latest：当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据（在消费者启动之后生成的记录）
        //none：当各分区都存在已提交的offset时，从提交的offset开始消费；只要有一个分区不存在已提交的offset，则抛出异常
        propsMap.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, autoOffsetReset);
        //两次poll之间的最大间隔，默认值为5分钟。如果超过这个间隔会触发reBalance
        propsMap.put(ConsumerConfig.MAX_POLL_INTERVAL_MS_CONFIG, maxPollIntervalTime);
        //这个参数定义了poll方法最多可以拉取多少条消息，默认值为500。如果在拉取消息的时候新消息不足500条，那有多少返回多少；如果超过500条，每次只返回500。
        //这个默认值在有些场景下太大，有些场景很难保证能够在5min内处理完500条消息，
        //如果消费者无法在5分钟内处理完500条消息的话就会触发reBalance,
        //然后这批消息会被分配到另一个消费者中，还是会处理不完，这样这批消息就永远也处理不完。
        //要避免出现上述问题，提前评估好处理一条消息最长需要多少时间，然后覆盖默认的max.poll.records参数
        //注：需要开启BatchListener批量监听才会生效，如果不开启BatchListener则不会出现reBalance情况
        propsMap.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, maxPollRecords);
        //当broker多久没有收到consumer的心跳请求后就触发reBalance，默认值是10s
        propsMap.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, sessionTimeout);
        //序列化（建议使用Json，这种序列化方式可以无需额外配置传输实体类）
        propsMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
        propsMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
        return propsMap;
    }

    @Bean
    public ConsumerFactory<Object, Object> consumerFactory() {
        //配置消费者的 Json 反序列化的可信赖包，反序列化实体类需要
        try(JsonDeserializer<Object> deserializer = new JsonDeserializer<>()) {
            deserializer.trustedPackages("*");
            return new DefaultKafkaConsumerFactory<>(consumerConfigs(), new JsonDeserializer<>(), deserializer);
        }
    }

    @Bean
    public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<Object, Object>> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<Object, Object> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        //在侦听器容器中运行的线程数，一般设置为 机器数*分区数
        factory.setConcurrency(concurrency);
        //消费监听接口监听的主题不存在时，默认会报错，所以设置为false忽略错误
        factory.setMissingTopicsFatal(missingTopicsFatal);
        //自动提交关闭，需要设置手动消息确认
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL_IMMEDIATE);
        factory.getContainerProperties().setPollTimeout(pollTimeout);
        //设置为批量监听，需要用List接收
        //factory.setBatchListener(true);
        return factory;
    }
}
```

---


# 五、开始写代码
> 下面我们开始写 Kafka 的消息发送代码

## 生产者
### 发送
`KafkaController`用于发送消息到 Kafka

```java
import icu.xuyijie.provider.entity.User;
import icu.xuyijie.provider.handler.KafkaSendResultHandler;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.springframework.kafka.config.KafkaListenerEndpointRegistry;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.messaging.MessageHeaders;
import org.springframework.messaging.support.GenericMessage;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;
import java.util.Objects;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

/**
 * @author 徐一杰
 * @date 2022/10/31 14:05
 * kafka发送消息
 */
@RestController
@RequestMapping("/provider")
//这个注解代表这个类开启Springboot事务，因为我们在Kafka的配置文件开启了Kafka事务，不然会报错
@Transactional(rollbackFor = RuntimeException.class)
public class KafkaController {

    private final KafkaTemplate<Object, Object> kafkaTemplate;

    public KafkaController(KafkaTemplate<Object, Object> kafkaTemplate, KafkaSendResultHandler kafkaSendResultHandler) {
        this.kafkaTemplate = kafkaTemplate;
        //回调方法、异常处理
        this.kafkaTemplate.setProducerListener(kafkaSendResultHandler);
    }

    @RequestMapping("/sendMultiple")
    public void sendMultiple() {
        String message = "发送到Kafka的消息";
        for (int i = 0;i < 10;i++) {
            kafkaTemplate.send("topic1", "发送到Kafka的消息" + i);
            System.out.println(message + i);
        }
    }

    @RequestMapping("/send")
    public void send() {
    	//这个User的代码我没放出来，自己随便写一个实体类，实体类一定要 implements Serializable
        User user = new User(1, "徐一杰");
        kafkaTemplate.send("topic1", user);
        kafkaTemplate.send("topic2", "发给topic2");
    }

	/**
     * Kafka提供了多种构建消息的方式
     * @throws ExecutionException
     * @throws InterruptedException
     * @throws TimeoutException
     */
    public void SendDemo() throws ExecutionException, InterruptedException, TimeoutException {
        //后面的get代表同步发送，括号内时间可选，代表超过这个时间会抛出超时异常，但是仍会发送成功
        kafkaTemplate.send("topic1", "发给topic1").get(1, TimeUnit.MILLISECONDS);

        //使用ProducerRecord发送消息
        ProducerRecord<Object, Object> producerRecord = new ProducerRecord<>("topic.quick.demo", "use ProducerRecord to send message");
        kafkaTemplate.send(producerRecord);

        //使用Message发送消息
        Map<String, Object> map = new HashMap<>();
        map.put(KafkaHeaders.TOPIC, "topic.quick.demo");
        map.put(KafkaHeaders.PARTITION_ID, 0);
        map.put(KafkaHeaders.MESSAGE_KEY, 0);
        GenericMessage<Object> message = new GenericMessage<>("use Message to send message", new MessageHeaders(map));
        kafkaTemplate.send(message);
    }
}
```

### 成功回调和异常处理
`KafkaSendResultHandler `

```java
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.springframework.kafka.support.ProducerListener;
import org.springframework.lang.Nullable;
import org.springframework.stereotype.Component;

/**
 * @author 徐一杰
 * @date 2022/10/31 15:41
 * kafka消息发送回调
 */
@Component
public class KafkaSendResultHandler implements ProducerListener<Object, Object> {

    @Override
    public void onSuccess(ProducerRecord producerRecord, RecordMetadata recordMetadata) {
        System.out.println("消息发送成功：" + producerRecord.toString());
    }

    @Override
    public void onError(ProducerRecord producerRecord, @Nullable RecordMetadata recordMetadata, Exception exception) {
        System.out.println("消息发送失败：" + producerRecord.toString() + exception.getMessage());
    }
}
```

## 消费者
### 接收
`KafkaHandler`用于接收 Kafka 里的消息

```java
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.config.KafkaListenerEndpointRegistry;
import org.springframework.kafka.support.Acknowledgment;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;
import java.util.Objects;

/**
 * @author 徐一杰
 * @date 2022/10/31 14:04
 * kafka监听消息
 */
@RestController
public class KafkaHandler {

    private final KafkaListenerEndpointRegistry kafkaListenerEndpointRegistry;

    public KafkaHandler(KafkaListenerEndpointRegistry kafkaListenerEndpointRegistry) {
        this.kafkaListenerEndpointRegistry = kafkaListenerEndpointRegistry;
    }

    /**
     * 监听kafka消息
     *
     * @param consumerRecord kafka的消息，用consumerRecord可以接收到更详细的信息，也可以用String message只接收消息
     * @param ack  kafka的消息确认
     * 使用autoStartup = "false"必须指定id
     */
    @KafkaListener(topics = {"topic1", "topic2"}, errorHandler = "myKafkaListenerErrorHandler")
//    @KafkaListener(id = "${spring.kafka.consumer.group-id}", topics = {"topic1", "topic2"}, autoStartup = "false")
    public void listen1(ConsumerRecord<Object, Objects> consumerRecord, Acknowledgment ack) {
        try {
            //用于测试异常处理
            //int i = 1 / 0;
            System.out.println(consumerRecord.get(0).value());
            //手动确认
            ack.acknowledge();
        } catch (Exception e) {
            System.out.println("消费失败：" + e);
        }
    }

    /**
     * 下面的方法可以手动操控kafka的队列监听情况
     * 先发送一条消息，因为autoStartup = "false"，所以并不会看到有消息进入监听器。
     * 接着启动监听器，/start/webGroup。可以看到有一条消息进来了。
     * pause是暂停监听，resume是继续监听
     *
     * @param listenerId consumer的group-id
     */
    @RequestMapping("/pause/{listenerId}")
    public void stop(@PathVariable String listenerId) {
        Objects.requireNonNull(kafkaListenerEndpointRegistry.getListenerContainer(listenerId)).pause();
    }

    @RequestMapping("/resume/{listenerId}")
    public void resume(@PathVariable String listenerId) {
        Objects.requireNonNull(kafkaListenerEndpointRegistry.getListenerContainer(listenerId)).resume();
    }

    @RequestMapping("/start/{listenerId}")
    public void start(@PathVariable String listenerId) {
        Objects.requireNonNull(kafkaListenerEndpointRegistry.getListenerContainer(listenerId)).start();
    }
}
```

### 异常处理
`MyKafkaListenerErrorHandler`

```java
import org.apache.kafka.clients.consumer.Consumer;
import org.springframework.kafka.listener.KafkaListenerErrorHandler;
import org.springframework.kafka.listener.ListenerExecutionFailedException;
import org.springframework.lang.NonNull;
import org.springframework.messaging.Message;
import org.springframework.stereotype.Component;

/**
 * @author 徐一杰
 * @date 2022/10/31 15:27
 * 异常处理
 */
@Component
public class MyKafkaListenerErrorHandler implements KafkaListenerErrorHandler {

    @Override
    @NonNull
    public Object handleError(@NonNull Message<?> message, @NonNull ListenerExecutionFailedException exception) {
        return new Object();
    }

    @Override
    @NonNull
    public Object handleError(@NonNull Message<?> message, @NonNull ListenerExecutionFailedException exception, Consumer<?, ?> consumer) {
        System.out.println("消息详情：" + message);
        System.out.println("异常信息：：" + exception);
        System.out.println("消费者详情：：" + consumer.groupMetadata());
        System.out.println("监听主题：：" + consumer.listTopics());
        return KafkaListenerErrorHandler.super.handleError(message, exception, consumer);
    }
}
```

---

# 七、开始测试
> 启动生产者和消费者，消费者控制台打印出我配置的 group-id `webGroup` id就是启动成功了，`如果启动报错不会解决，可以评论区留言`

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SpringbootKafka1.png)
## 测试普通单条消息

> 浏览器访问 `http://127.0.0.1:8081/provider/send` 来调用生产者发送一条消息，生产者控制台打印出回调，消费者控制台输出接收到的消息

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SpringbootKafka2.png)
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SpringbootKafka3.png)

## 测试消费者异常处理
> 把消费者里的 `listen1` 方法里的这行代码取消注释

```java
//用于测试异常处理
int i = 1 / 0;
```
> 重启消费者，访问 `http://127.0.0.1:8081/provider/send` ，发现消费者虽然报错但是没有抛出异常，而是被我们处理了

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SpringbootKafka4.png)



## 测试延时消息
> 发送延时消息要关闭事务，在生产者的 yml 和 config 配置文件里把下面代码注释掉

```yaml
# 开启事务，必须在开启了事务的方法中发送，否则报错
# transaction-id-prefix: kafkaTx-
```

```java
//开启事务，会导致 LINGER_MS_CONFIG 配置失效
//factory.setTransactionIdPrefix(transactionIdPrefix);
```
> 然后重新请求`http://127.0.0.1:8081/provider/send`，发现 5s 后消息发出，配置延迟时间的配置是`props.put(ProducerConfig.LINGER_MS_CONFIG, "5000");`，其实这个不是真正的延时消息，Kafka实现真正的延时消息要使用JDK的DelayQueue手动实现。

## 测试批量消息

> 打开消费者的 config 配置里 `setBatchListener` 这一行代码，我们定义的 `MAX_POLL_RECORDS_CONFIG` 为3，即每次批量读取3条消息，批量监听需要用List接收，`listen1`方法的参数加一个List包起来

```java
//设置为批量监听，需要用List接收
factory.setBatchListener(true);
```

```java
propsMap.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, maxPollRecords);
```

```java
public void listen1(List<ConsumerRecord<Object, Objects>> consumerRecord, Acknowledgment ack)
```

> 注意！！！Debug消费者，因为我们要打断点观察每次接收的条数
> 调用消费者接口`http://127.0.0.1:8081/provider/sendMultiple`批量发送10条，可以看到消费者每次只接收3条

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SpringbootKafka5.png)

## 测试手动控制消费者监听
> `@KafkaListener`这样写，id 和 autoStartup 是关键

```java
@KafkaListener(id = "${spring.kafka.consumer.group-id}", topics = {"topic1", "topic2"}, autoStartup = "false")
```
> 重启消费者，调用生产者接口`http://127.0.0.1:8081/provider/send`，我们发现这次消费者没有接收到消息，因为我们关闭了 `autoStartup`

> 要开始接收的话，调用消费者接口`http://127.0.0.1:8082/start/firstGroup`，这个方法可以启动 group-id 为 firstGroup 的 @KafkaListener，然后我们发现消费者控制台接收到消息

> `http://127.0.0.1:8082/pause/firstGroup`暂停接收
> `http://127.0.0.1:8082/resume/firstGroup`恢复接收

---
# 总结
你会了吗，我反正是又写了一遍博客现在刻到脑子里了，但是项目里有两个配置参数我有疑问
> `batch-size`，这个参数没有效果
> 为什么开启事务以后会让 `LINGER_MS_CONFIG`这个配置失效，这个我并没有看到文档里面有写
> 有没有知道的同学告诉我一下
