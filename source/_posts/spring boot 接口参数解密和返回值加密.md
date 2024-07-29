---
title: spring boot 接口参数解密和返回值加密
date: 2024-07-05 16:22:48
categories:
  - Java
tags:
  - SpringBoot
  - 数据加密
cover: https://qiniuoss.xuyijie.icu/SecureApiDoc/img/logo/logo.png
---

# 开发背景
> - 虽然使用 HTTPS 已经可以基本保证传输数据的安全性，但是很多国企、医疗、股票项目等仍然要求对接口数据进行自行加密，作者最近就遇到了，是支付宝对接国家医保的接口时，医保接口要求进行接口加密。
>
> - 所以鉴于这个需求的存在，作者也找过现在的一些接口加密组件，发现都是3.4年前就已经停止更新了，并且不接受 pr 也不回复 issue 了，用起来也是功能简陋配置复杂，所以就开发了一套全新的组件。
>
> - 作者对接口加密的各种需求了解的并不是很多，如果在使用中有功能缺失，欢迎来 [Github](https://github.com/BubblingXuYijie/secure-api-spring-boot) 提 issue 或者 pr ❤️。

---

# 简介

> - SecureApi 是一款接口参数和返回值加解密工具，高性能、轻量化，无任何外部依赖；
>
> - spring boot 场景启动器设计（支持spring boot2和3），完全自动化，支持 param、body 参数（暂不支持 path 参数），用户无需关心加密解密和密钥匹配过程；
>
> - 配置灵活，配置文件支持 yml 和 bean 方式，支持注解、url正则进行接口匹配，支持 AES、SM4、RSA 等多种加密方式，支持 DH 前后端密钥协商方式。

`用户增量趋势`

此组件发布已经一两年了，发现有那么多用户，才重新整理了命名、仓库和文档

![组件maven下载量.jpg](https://qiniuoss.xuyijie.icu/SecureApiDoc/img/组件maven下载量.jpg)

==本篇只是先体验功能，使用请看官方文档：== [https://doc.xuyijie.icu/secure-api-doc/](https://doc.xuyijie.icu/secure-api-doc/)

==Github地址：== [https://github.com/BubblingXuYijie/secure-api-spring-boot](https://github.com/BubblingXuYijie/secure-api-spring-boot)

==Gitee地址(非主要，issue还是集中在Github比较好，大家都可以看到)== [https://gitee.com/BubblingXuYijie/secure-api-spring-boot](https://gitee.com/BubblingXuYijie/secure-api-spring-boot)

一些问题你们会在`文档`中或者Github的 [issue](https://github.com/BubblingXuYijie/secure-api-spring-boot/issues?q=) 里找到答案，如果你不知道前端如何配合，那么`文档里有你想要的东西`

---
# 安装

> 环境要求：
> - jdk8+（spring boot 3 请使用 jdk17+）
> - Maven/Gradle
> - spring boot 2+（spring boot 3 请引入 SecureApi 的 3.0.0+ 版本）

`Maven`
```xml
<!-- https://mvnrepository.com/artifact/icu.xuyijie/secure-api-spring-boot-starter -->
<dependency>
    <groupId>icu.xuyijie</groupId>
    <artifactId>secure-api-spring-boot-starter</artifactId>
    <!--spring boot 3 请引入 3.0.7 版本-->
    <version>2.1.4</version>
</dependency>
```

`Gradle`

```groovy
// spring boot 3 请引入 3.0.7 版本
implementation 'icu.xuyijie:secure-api-spring-boot-starter:2.1.4'
```

---


# 配置

> 有两种配置方法， `yml` 和 `Bean` 方式，看你喜欢哪一个，如果是对安全性要求较高，建议使用 `Bean` 方式动态设置密钥，而不是写在 `yml` 里。

## yml 方式

> 下面是 yml 的完整配置，有些配置项是可选的，在注解中均已解释

```yml
secure-api:
  # 开启SecureApi功能，如果为false则其余配置项均不生效
  enabled: true
  # 生成的key和密文等是否是符合url规范的
  url-safe: true
  # 开启加解密日志打印，会打印出接口名、加密模式、算法、明文和密文等信息
  show-log: true
  # 加密模式，common和session_key可选，session_key是会话密钥模式，用于每次请求都使用不同的密钥，需要前端配合
  mode: common
  # 加密算法
  cipher-algorithm: rsa_ecb_sha256
  # session_key模式配置项，与前端协商的会话密钥类型，common模式下此配置不生效
  session-key-cipher-algorithm: aes_ecb_pkcs5
  # 对称算法用于加解密的密钥，Base64格式，cipher-algorithm选择对称加密算法时配置，也可为空，组件会随机生成一个
  key:
  # 对称算法用于加解密的偏移量，Base64格式，cipher-algorithm选择对称加密算法时配置，也可为空，组件会随机生成一个
  iv:
  # 非对称算法用于加密的公钥，Base64格式，cipher-algorithm选择RSA非对称加密算法时配置，也可为空，组件会随机生成一对
  public-key:
  # 非对称算法用于解密的私钥，Base64格式，cipher-algorithm选择RSA非对称加密算法时配置，也可为空，组件会随机生成一对
  private-key:
  # 需要加密的接口路径匹配，遵循spring boot拦截器的正则规则，留空或者不配置代表不使用url匹配，只对注解的接口进行解密
  encrypt-url:
    # 配置了此项，接口有无注解都将进行返回值加密
    include-urls: /**
    # 即使配置了排除，注解的优先级也高于此项
    exclude-urls:
  # 需要解密的接口路径匹配，遵循spring boot拦截器的正则规则，留空或者不配置代表不使用url匹配，只对注解的接口、参数、字段进行解密
  decrypt-url:
    # 配置了此项，接口、参数、字段有无注解都将进行解密
    include-urls: /**
    # 即使配置了排除，注解的优先级也高于此项
    exclude-urls:
```

## Bean 方式

> 注意，一旦使用了 `Bean` 方式来配置，`yml` 里的配置项都将`失效`

```java
import icu.xuyijie.secureapi.cipher.CipherAlgorithmEnum;
import icu.xuyijie.secureapi.cipher.CipherUtils;
import icu.xuyijie.secureapi.cipher.RsaKeyPair;
import icu.xuyijie.secureapi.model.SecureApiProperties;
import icu.xuyijie.secureapi.model.SecureApiPropertiesConfig;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.context.annotation.Bean;

import java.util.ArrayList;
import java.util.Arrays;

@SpringBootConfiguration
public class SecureApiConfig {
    /**
     * 这里的配置了Bean会导致yml配置的数据失效
     * @return SecureApiPropertiesConfig
     */
    @Bean
    public SecureApiPropertiesConfig secureApiPropertiesConfig() {
        SecureApiPropertiesConfig secureApiPropertiesConfig = new SecureApiPropertiesConfig();
        secureApiPropertiesConfig.setEnabled(true);
        secureApiPropertiesConfig.setUrlSafe(true);
        secureApiPropertiesConfig.setShowLog(true);
        secureApiPropertiesConfig.setMode(SecureApiProperties.Mode.COMMON);
        secureApiPropertiesConfig.setCipherAlgorithmEnum(CipherAlgorithmEnum.RSA_ECB_SHA256);
        
        // 密钥可以不设置，组件会自动生成一个，并打印在控制台，如果需要手动生成，只需要使用组件提供的CipherUtils
        CipherUtils cipherUtils = new CipherUtils(CipherAlgorithmEnum.RSA_ECB_SHA256);
        // 因为我们选择的是非对称加密RSA，所以生成一个密钥对，getRandomRsaKeyPair("1")可传入seed参数，在测试时可用于控制每次生成的密钥相同
        RsaKeyPair randomRsaKeyPair = cipherUtils.getRandomRsaKeyPair();
        // 把生成的密钥对设置到secureApiPropertiesConfig
        secureApiPropertiesConfig.setPublicKey(randomRsaKeyPair.getPublicKey());
        secureApiPropertiesConfig.setPrivateKey(randomRsaKeyPair.getPrivateKey());
        
        // 不需要使用url匹配功能可以删除掉下面两行，或者传入空数组
        // secureApiPropertiesConfig.setEncryptUrl(new SecureApiProperties.UrlPattern(Arrays.asList("/**"), new ArrayList<>()));
        // secureApiPropertiesConfig.setDecryptUrl(new SecureApiProperties.UrlPattern(Arrays.asList("/**"), Arrays.asList("/secureApiTest/testForm")));
        return secureApiPropertiesConfig;
    }
}
```

---


# 试一下

> 好了各位大佬们，到了这里，配置以及完成，接下来可以进行效果体验了，后面有时间会提供前后端的 demo。

==本篇只是先体验功能，请务必和我的代码相同，不然也许你会遇到一些问题，代码里使用的注解等看文档会介绍，可以解决你的疑惑。==
## 启动项目

> SecureApi的 `enable` 设置为 `true` 时，控制台会打印以下信息，代表开启接口加解密功能，我这里没有指定 key ，所以组件为我自动生成了，然后你可以把密钥设置到前端或者和前端进行密钥协商以追求更安全的传输。

> 建议测试的时候使用 `CipherUtils` 手动设置密钥，可以指定 `seed` 保证每次生成的密钥相同，更加方便。

![后端启动打印信息](https://qiniuoss.xuyijie.icu/SecureApiDoc/img/后端Demo启动打印信息.png)

## 返回值加密

> 前面我们没有配置url匹配，需要在接口上或者接口所在类上添加 `@EncryptApi` 注解，即可实现返回值加密

![返回值加密接口](https://qiniuoss.xuyijie.icu/SecureApiDoc/img/后端测试返回值加密接口.png)

> 可以看到由于我开启了日志打印功能，控制台打印出一些信息

![返回值加密日志](https://qiniuoss.xuyijie.icu/SecureApiDoc/img/后端测试返回值加密.png)

> 接口返回的是一个 json 字符串，然后前端使用对应密钥解密这个字符串（注意这是个json字符串，前端处理时应该去除前后两端引号）就可以拿到 `{"code":200,"message":"哈哈哈","data":null}` 这样的对象了

![接口返回值密文](https://qiniuoss.xuyijie.icu/SecureApiDoc/img/后端测试返回值加密接口返回值.png)

---

## 参数解密

### body 参数解密

> 组件可以对json参数体进行解密，这次我们传入上一步中加密的返回值，看一看解密结果，接口需要添加 `@DecryptApi` 注解，这样这个接口既会解密参数，也会加密返回值

![参数解密接口](https://qiniuoss.xuyijie.icu/SecureApiDoc/img/后端测试Body参数解密接口.png)

> 使用 `Postman` 发送密文body参数

![发送body密文](https://qiniuoss.xuyijie.icu/SecureApiDoc/img/后端测试Body参数解密接口返回值.png)

> 可以看到密文参数正常解密为 `{"code":200,"message":"哈哈哈","data":null}`,返回值也成功加密了

![body解密](https://qiniuoss.xuyijie.icu/SecureApiDoc/img/后端测试Body参数解密.png)

### param和form-data参数解密

> 这一次我们整复杂一点，各种类型的参数都整上，没有开启url匹配，我们要给字段加上 `@DecryptParam` 注解，注意 `@DecryptParam` 不能和 `@RequestParam` 同时使用，`@DecryptParam` 已经替代了后者功能。

![param和form-data参数解密接口](https://qiniuoss.xuyijie.icu/SecureApiDoc/img/测试param和form-data参数解密接口.png)

> 实体类本身不需要加注解，要加在里面的字段上，注意，没有加注解的字段不会解密（配置了url匹配的话，加不加注解全部字段都会解密）

![实体类中字段](https://qiniuoss.xuyijie.icu/SecureApiDoc/img/测试param和form-data参数解密实体类.png)

> 发送请求，请求中这些密文都是我提前用代码生成好的（注意，param里的密文要是url safe的），这些参数放在 param 里或者放在 form-data 里发送都是可以的

![发送请求](https://qiniuoss.xuyijie.icu/SecureApiDoc/img/测试param和form-data参数解密postman.png)

> 成功解密加了注解的参数，没有加注解和为null的参数不解密

![param和form-data参数解密日志](https://qiniuoss.xuyijie.icu/SecureApiDoc/img/测试param和form-data参数解密结果.png)

---
# 总结

==本篇只是先体验功能，使用请看官方文档：== [https://doc.xuyijie.icu/secure-api-doc/](https://doc.xuyijie.icu/secure-api-doc/)

==Github地址：== [https://github.com/BubblingXuYijie/secure-api-spring-boot](https://github.com/BubblingXuYijie/secure-api-spring-boot)

==Gitee地址(非主要，issue还是集中在Github比较好，大家都可以看到)== [https://gitee.com/BubblingXuYijie/secure-api-spring-boot](https://gitee.com/BubblingXuYijie/secure-api-spring-boot)

一些问题你们会在`文档`中或者Github的 [issue](https://github.com/BubblingXuYijie/secure-api-spring-boot/issues?q=) 里找到答案，如果你不知道前端如何配合，那么`文档里有你想要的东西`
