---
title: Springboot 配置使用 Elasticsearch
date: 2022-09-29 21:52:07
categories: Java
tags:
    - SpringBoot
    - Elasticsearch
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/elasticsearch.jpg
---
# 前言
> `elasticsearch`：就是一个很快很快的搜索引擎，使用场景就是像网易词典这种，输入几个相关文字，就可以从几百万个单词中瞬间找到匹配的翻译，这种海量数据的模糊搜索在关系型数据库中是不可能的，所以这时候就要用到elasticsearch了，但是它和 MongoDB 这种文档型数据库有什么区别，有没有懂的可以在评论区教教我，我是没有搞懂

---


# 一、安装Elasticsearch
## 1、Windows安装
> Windows安装比较简单，官网下载压缩包，解压出来，然后里面有个 bin 目录，里面有个`elasticsearch.bat`，双击，就运行起来了，然后在浏览器输入`localhost:9200`验证，成功会返回下面的图片

## 2、Linux安装

```bash
# 导入公钥，如果报错gpg，则先apt install gpg一下就可以了，如果报错sudo，则去掉命令中的sudo
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
# 添加elasticsearch到仓库
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
# 更新仓库，安装
sudo apt update
sudo apt install elasticsearch
# 开启外网访问，把这个文件里的 network.host 改为 0.0.0.0
# elasticsearch版本 7.0、8.0 以上还需要修改 xpack-security.enable 为 false，如果不使用 ssl，也设置为 false，如下图
sudo vim /etc/elasticsearch/elasticsearch.yml
# 启动服务
sudo systemctl start elasticsearch.service
# 设置开机启动（可选）
sudo systemctl enable elasticsearch.service
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0a6849f927014a11814e0c54788f02ee.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/fc354c7212b041d1bf56be7e8ac7c1e3.png)

不要忘记开放防火墙端口，控制台输入`curl -X GET "localhost:9200/"`或浏览器访问 `IP:9200`

![在这里插入图片描述](https://img-blog.csdnimg.cn/27afbe9e882f4c6fa7f762513a23ae49.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/d50e6c08ce8c4e62b5ea12c32925cb7b.png)
> 开放防火墙 9200 端口

```bash
# Debian/Ubuntu ufw
ufw allow 9200
ufw reload
# Debian/Ubuntu iptables（这个叼毛防火墙好麻烦，我没用过，不知道是不是这样）
iptables -A INPUT -p tcp --dport 9200 -j ACCEPT
iptables-restore
# CentOS
firewall-cmd --zone=public --add-port=9200/tcp --permanent
firewall-cmd --reload
```


# 二、开始写代码

> 先给你们看一下项目结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/6e88ddbcd6454373a95addd1182643f3.png)


## 1、引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```
## 2、配置文件
`两种方式选一种就可以了`

### (1) yml 方式
> 默认的 elasticsearch 不需要密码登录，所以不需要写

```yaml
server:
  port: 8081
spring:
  elasticsearch:
  	# elasticsearch地址
    uris: 192.168.0.105:9200
#    username: elasticsearch
#    password: 3ve6fT0TQreeQslzkiojCA
    connection-timeout: 30000
    socket-timeout: 50000
    socket-keep-alive: false
```

### (1) api 方式
> 新建`ElasticSearchConfig`，这里面的配置会覆盖掉`yml`里面的

```java
package pers.xuyijie.elasticsearchdemo.config;

import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.json.jackson.JacksonJsonpMapper;
import co.elastic.clients.transport.ElasticsearchTransport;
import co.elastic.clients.transport.rest_client.RestClientTransport;
import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.context.annotation.Bean;

/**
 * @author 徐一杰
 * @date 2022/9/20 17:39
 * @description
 */
@SpringBootConfiguration
public class ElasticSearchConfig {
    @Bean
    public ElasticsearchClient elasticsearchClient(){
        RestClient client = RestClient.builder(new HttpHost("192.168.0.105", 9200))
                .setRequestConfigCallback(requestConfigBuilder -> requestConfigBuilder
                        .setConnectTimeout(30000)
                        .setSocketTimeout(50000))
                        .build();
        ElasticsearchTransport transport = new RestClientTransport(client, new JacksonJsonpMapper());
        return new ElasticsearchClient(transport);
    }
}

```


---


## 3、新建 User 实体类

> `id`这个字段一定要有，作为主键索引，这个`@Document`里面的`indexName`就相当于mysql里面的表名，在elasticsearch里面叫索引

```java
package pers.xuyijie.elasticsearchdemo.document;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;

/**
 * @author 徐一杰
 * @date 2022/9/20 17:58
 * @description
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
@Document(indexName = "user")
public class User {
    @Id
    private String id;
    private String name;
    private String sex;
    private Integer age;
}

```

---


## 4、新建 UserRepository
> 这里就和 mybatis 一样了，继承一下可以使用内置方法

```java
package pers.xuyijie.elasticsearchdemo.repository;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.elasticsearch.annotations.Query;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
import pers.xuyijie.elasticsearchdemo.document.User;

import java.util.List;

/**
 * @author 徐一杰
 * @date 2022/9/20 18:25
 * @description
 */
public interface UserRepository extends ElasticsearchRepository<User, String> {

}

```

## 5、新建 Controller
> 下面我把这3个controller的代码放出来，方法很简单，都是增删改查，都有写注释，等下测试用
>
![在这里插入图片描述](https://img-blog.csdnimg.cn/02140214eb2b491db530d6098f5f056c.png)
`DocumentTestController.java`

```java
package pers.xuyijie.elasticsearchdemo.controller;

import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch._types.SortOrder;
import co.elastic.clients.elasticsearch.core.*;
import co.elastic.clients.elasticsearch.core.bulk.BulkOperation;
import co.elastic.clients.elasticsearch.core.search.Hit;
import co.elastic.clients.transport.endpoints.BooleanResponse;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import pers.xuyijie.elasticsearchdemo.document.User;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * @author 徐一杰
 * @date 2022/9/20 18:02
 * @description
 */
@RestController
@RequestMapping("/documentTest")
public class DocumentTestController {

    private final ElasticsearchClient elasticsearchClient;

    public DocumentTestController(ElasticsearchClient elasticsearchClient){
        this.elasticsearchClient = elasticsearchClient;
    }

    @RequestMapping("/insert")
    public void insert() throws IOException {
        User user = new User("1", "user1", "男", 18);
        IndexResponse indexResponse = elasticsearchClient.index(i -> i
                .index("user")
                //设置id
                .id("1")
                //传入user对象
                .document(user));
        System.out.println(indexResponse);
    }

    @RequestMapping("/update")
    public void update() throws IOException {
        UpdateResponse<User> updateResponse = elasticsearchClient.update(u -> u
                        .index("user")
                        .id("1")
                        .doc(new User("1", "user2", "男", 19))
                , User.class);
        System.out.println(updateResponse);
    }

    @RequestMapping("/isExists")
    public void isExists() throws IOException {
        BooleanResponse indexResponse = elasticsearchClient.exists(e -> e.index("user").id("1"));
        System.out.println(indexResponse.value());
    }

    @RequestMapping("/get")
    public void get() throws IOException {
        GetResponse<User> getResponse = elasticsearchClient.get(g -> g
                        .index("user")
                        .id("1")
                , User.class
        );
        System.out.println(getResponse.source());
    }

    @RequestMapping("/delete")
    public void delete() throws IOException {
        DeleteResponse deleteResponse = elasticsearchClient.delete(d -> d
                .index("user")
                .id("1")
        );
        System.out.println(deleteResponse.id());
    }

    @RequestMapping("/insertMultiple")
    public void insertMultiple() throws IOException {
        List<User> userList = new ArrayList<>();
        userList.add(new User("10", "user1", "男", 18));
        userList.add(new User("11", "user2", "男",19));
        userList.add(new User("12", "user3", "女",20));
        userList.add(new User("13", "user4", "女",21));
        userList.add(new User("14", "user5", "女",22));
        List<BulkOperation> bulkOperationArrayList = new ArrayList<>();
        //遍历添加到bulk中
        for(User user : userList){
            bulkOperationArrayList.add(BulkOperation.of(o->o.index(i->i.document(user))));
        }

        BulkResponse bulkResponse = elasticsearchClient.bulk(b -> b.index("user")
                .operations(bulkOperationArrayList));
        System.out.println(bulkResponse);
    }

    @RequestMapping("/getComplex")
    public void getComplex() throws IOException {
        SearchResponse<User> search = elasticsearchClient.search(s -> s
                .index("user")
                //查询name字段包含hello的document(不使用分词器精确查找)
                .query(q -> q
                        .term(t -> t
                                .field("name")
                                .value(v -> v.stringValue("hello"))
                        ))
                //分页查询，从第0页开始查询3个document
                .from(0)
                .size(3)
                //按age降序排序
                .sort(f->f.field(o->o.field("age").order(SortOrder.Desc))),User.class
        );
        for (Hit<User> hit : search.hits().hits()) {
            System.out.println(hit.source());
        }
    }

}

```

`IndexTestController.java`

```java
package pers.xuyijie.elasticsearchdemo.controller;

import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.elasticsearch.indices.CreateIndexResponse;
import co.elastic.clients.elasticsearch.indices.DeleteIndexResponse;
import co.elastic.clients.elasticsearch.indices.GetIndexResponse;
import co.elastic.clients.transport.endpoints.BooleanResponse;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.io.IOException;

/**
 * @author 徐一杰
 * @date 2022/9/20 17:52
 * @description Index的增删查，Index类似于数据库名
 */
@RestController
@RequestMapping("/indexTest")
public class IndexTestController {

    private final ElasticsearchClient elasticsearchClient;

    public IndexTestController(ElasticsearchClient elasticsearchClient){
        this.elasticsearchClient = elasticsearchClient;
    }

    @GetMapping("/insert")
    public void insert() throws IOException {
        CreateIndexResponse createIndexResponse = elasticsearchClient.indices().create(c -> c.index("user"));
        System.out.println(createIndexResponse);
    }

    @GetMapping("/get")
    public void get() throws IOException {
        GetIndexResponse getIndexResponse = elasticsearchClient.indices().get(c -> c.index("user"));
        System.out.println(getIndexResponse);
    }

    @GetMapping("/delete")
    public void delete() throws IOException {
        DeleteIndexResponse deleteIndexResponse = elasticsearchClient.indices().delete(c -> c.index("user"));
        System.out.println(deleteIndexResponse.acknowledged());
    }

    @GetMapping("/isExists")
    public void isExists() throws IOException {
        BooleanResponse booleanResponse = elasticsearchClient.indices().exists(c -> c.index("user"));
        System.out.println(booleanResponse.value());
    }

}

```

`UserController.java`

```java
package pers.xuyijie.elasticsearchdemo.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import pers.xuyijie.elasticsearchdemo.document.User;
import pers.xuyijie.elasticsearchdemo.repository.UserRepository;

/**
 * @author 徐一杰
 * @date 2022/9/20 18:28
 * @description
 */
@RestController
@RequestMapping("/user")
public class UserController {

    private final UserRepository userRepository;

    public UserController(UserRepository userRepository){
        this.userRepository = userRepository;
    }

    /**
     * 添加
     */
    @RequestMapping("/insert")
    public String insert() {
        User user = new User();
        user.setId("1");
        user.setName("徐一杰");
        user.setSex("男");
        user.setAge(22);
        userRepository.save(user);
        return "success";
    }

    /**
     * 删除
     */
    @RequestMapping("/delete")
    public String delete() {
        User user = userRepository.getUserById("1");
        userRepository.delete(user);
        return "success";
    }

    /**
     * 局部更新
     */
    @RequestMapping("/update")
    public String update() {
        User user = userRepository.getUserById("1");
        user.setName("泡泡");
        userRepository.save(user);
        return "success";
    }
    /**
     * 查询
     */
    @RequestMapping("/get")
    public User get() {
        User user = userRepository.getUserById("1");
        System.out.println(user);
        return user;
    }

	@RequestMapping("/findByQuery")
    public Page<User> findByQuery() {
        Pageable pageable = PageRequest.of(1,20, Sort.by(new Sort.Order(Sort.Direction.ASC, "age")));
        Page<User> userList = userRepository.findByName("1", pageable);
        System.out.println(userList);
        return userList;
    }

}

```


---

## 6、开始测试

### (1) 启动项目
> 可以看到，我们的`user`索引自动添加到`elasticsearch`里面了

![在这里插入图片描述](https://img-blog.csdnimg.cn/bb3919ef2a244a3c9c035210f8cd320a.png)

### (2) 查询索引

> 我们调用 `http://127.0.0.1:8081/indexTest/get`，可以看到，控制台打印出 user 的信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/ae74bbc4b3b94462b771164a0852355c.png)
> `IndexTestController`里面的方法都是操作索引的，删除、新增等

### (3) 新增数据

> 我们先调用`http://127.0.0.1:8081/documentTest/insert`，往`user`里面添加一条数据，这是`DocumentTestController`里面的方法，这个controller里面的方法可以操作没有继承 `ElasticsearchRepository` 的实体类

![在这里插入图片描述](https://img-blog.csdnimg.cn/7e8208f32a5a4775a484b2b9ff062814.png)

> 下面我们查询出来我们刚刚插入的数据，这次我们调用 `repository`里面的内置方法，`http://127.0.0.1:8081/user/get`，查询成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/3be22e490e124316904ebfe18951cb70.png)

---

## 7、复杂查询条件

> 对于多条件查询，我们可以使用`ElasticsearchRepository`自带的`衍生查询`，关键词 find by 等等，有代码提示

![在这里插入图片描述](https://img-blog.csdnimg.cn/84a7f95839f44627a3a427104b22fea6.png)


```java
package pers.xuyijie.elasticsearchdemo.repository;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.elasticsearch.annotations.Query;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
import pers.xuyijie.elasticsearchdemo.document.User;

import java.util.List;

/**
 * @author 徐一杰
 * @date 2022/9/20 18:25
 * @description
 */
public interface UserRepository extends ElasticsearchRepository<User, String> {

    /**
     * 衍生查询，直接指定查询方法名称便可查询，无需进行实现
     * 以下为根据用户名字和年龄不超过多少，并按年龄正序排列的例子
     * @param name
     * @param age
     * @return
     */
    List<User> findUsersByNameAndAgeBeforeOrderByAgeAsc(String name, int age);

    /**
     * 衍生查询，根据id查询
     * @param id
     * @return
     */
    User getUserById(String id);

    /**
     * 使用@Query注解可以用Elasticsearch的DSL语句进行查询
     * 根据名字查找（分页）
     * @param name
     * @param pageable
     * @return
     */
    @Query("'{\"bool\" : {\"must\" : {\"field\" : {\"name\" : \"?0\"}}}}'")
    Page<User> findByName(String name, Pageable pageable);

}

```
> 然后还有在`DocumentTestController `里面的`getComplex`方法，也可以和构建复杂条件

```java
@RequestMapping("/getComplex")
    public void getComplex() throws IOException {
        SearchResponse<User> search = elasticsearchClient.search(s -> s
                .index("user")
                //查询name字段包含hello的document(不使用分词器精确查找)
                .query(q -> q
                        .term(t -> t
                                .field("name")
                                .value(v -> v.stringValue("hello"))
                        ))
                //分页查询，从第0页开始查询3个document
                .from(0)
                .size(3)
                //按age降序排序
                .sort(f->f.field(o->o.field("age").order(SortOrder.Desc))),User.class
        );
        for (Hit<User> hit : search.hits().hits()) {
            System.out.println(hit.source());
        }
    }

```


# 总结
