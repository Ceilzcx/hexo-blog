---
title: SpringBoot
date: 2021-07-15 15:56:32
tags: java
---

## SpringBoot



### 一、跨域问题 —— CORS

```java
// 拦截器和其他配置
WebMvcConfigurer
public void addCorsMappings(CorsRegistry registry);
```



### 二、读取配置的几种方式

配置文件：Resources下的文件，一般为yml文件或者properties文件

+ 资源文件注解：`@PropertySource`

+ 默认yml文件的注解：`@ConfigurationProperties `
+ 读取对应的值：`@Value`



### 三、拦截器和过滤器

+ 相同点：都使用AOP编程思想

+ 不同点：
  + 1、Filter是Servlet规范定义的，拦截器是Spring框架的
  + 2、过滤器在进入tomcat容器后，servlet之前的预处理；拦截器在servlet处理后执行
  + 3、拦截器可以使用Spring中的各个Bean；Filter依赖Servlet容器（可以操作Request和Response）
  + 4、过滤器的实现基于回调；拦截器的实现基于反射



### 四、Spring Event

观察者模式、监听器模式

主要组成：事件（ApplicationEvent）、监听器（ApplicationListener）、事件发布操作（publisher方法）

```java
public class TestEvent extends ApplicationEvent {
    private final String msg;

    public TestEvent(Object source, String msg) {
        super(source);
        this.msg = msg;
    }

    public String getMsg() {
        return msg;
    }
}
```

```java
@Component	// 需要将监听器添加到Spring容器
public class TestEventListener implements ApplicationListener<TestEvent> {
    @Override
    public void onApplicationEvent(TestEvent testEvent) {
        System.out.println("触发监听器：" + testEvent.getMsg());
    }
}
```

```
@Configuration
public class EventConfig {
	// 与上述代码实现效果一样，使用注解和接口的区别
    @EventListener(classes = {TestEvent.class})	
    public void listen(TestEvent event) {
        System.out.println("触发监听器：" + event.getMsg());
    }
}
```

```java
@Component
public class TestPublisher {
    @Autowired
    private ApplicationEventPublisher publisher;

    public void publish(String msg) {
        publisher.publishEvent(new TestEvent(this, msg));
    }
}
```



### 其他

#### 一、Servlet主要处理 doGet和doPost













#### 配置文件脱敏

##### 2.6.0版本前可以使用 `jasypt`

```xml
<dependency>
	<groupId>com.github.ulisesbocchio</groupId>
	<artifactId>jasypt-spring-boot-starter</artifactId>
	<version>3.0.3</version>
</dependency>
```

**配置密钥**

```yaml
jasypt:
  encryptor:
    password: XXX
```

**密钥放在配置文件也不是一件很安全的事情，可以通过启动时传参**

```shell
java -jar xxx.jar -Djasypt.encryptor.password=***
```

**对需要加密的属性进行加密**

```java
@Autowired
private StringEncryptor encryptor;
encryptor.encrypt("xxx")
```

如果遇到以下异常

```tex
Encryption raised an exception. A possible cause is you are using strong encryption algorithms and you have not installed the Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files in this Java Virtual Machine
```

下载 `jce_policy-8`，将文件放到对应的 `jre/lib/security` 路径下，覆盖原有的文件

**加密后的密文写入配置**

通过ENC()包裹密文并写入



##### 2.6.0+ 后使用 `SanitizingFunction`

springboot 的接口，可以自定义指定配置文件的指定规则，对配置文件进行脱敏。

还在研究中，目前启动并没有执行里面的代码

```java
@Configuration
public class MySanitizingFunction {
    
    @Bean
    public SanitizingFunction sanitizingFunction() {
        return data -> {
            PropertySource<?> propertySource = data.getPropertySource();
            if (propertySource.getName().contains("xxx")) {
                // 执行相关操作
            }
            return data;
        };
    }
}
```

