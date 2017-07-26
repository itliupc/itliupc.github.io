---
title: 自定义Spring Boot Starter
date: 2017-07-26 10:00:00
tags: [Spring Boot]
categories: Spring Boot
---


Spring Boot由众多Starter组成，随着版本的推移Starter家族成员也与日俱增。在传统Maven项目中通常将一些层、组件拆分为模块来管理，以便相互依赖复用，在Spring Boot项目中我们则可以创建自定义Spring Boot Starter来达成该目的。

# 创建Maven项目
----

创建Maven项目并引入依赖：

```bash
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.test</groupId>
    <artifactId>test-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <!-- Import dependency management from Spring Boot -->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>1.5.2.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>

```

artifactId命名问题:
- spring官方Starter通常命名为spring-boot-starter-{name}
- Spring官方建议非官方Starter命名应遵循{name}-spring-boot-starter的格式


# 接收properties中配置的信息
----
```java
import org.springframework.boot.context.properties.ConfigurationProperties;

/**
 *接收配置文件中以test开头的message信息
 */
@ConfigurationProperties(prefix = "test")  
public class TestServiceProperties {
  
  private String message;

  public String getMessage() {
    return message;
  }

  public void setMessage(String message) {
    this.message = message;
  }
}
```
** @ConfigurationProperties：标识这个类是用来接收指定前缀为（test）的资源配置值 **

# 创建Service提供服务
----
```java
/**
 * 定义Service类对外提供服务（输出配置文件的信息）
 */
public class TestService {
  private String message;

  public String getMessage() {
    return message;
  }

  public void setMessage(String message) {
    this.message = message;
  }
  
  public String sayMessage(){
    return message;
  }
}
```

# 配置AutoConfigure
----

```java
/**
 * 用来读取TestServiceProprties配置信息注入到TestService
 */
@Configuration                                                                                                                              
@EnableConfigurationProperties(value = TestServiceProperties.class)                                                                        
@ConditionalOnClass(TestService.class)                                                                                                     
@ConditionalOnProperty(prefix = "test", value = "enable", matchIfMissing = true)  
public class TestAutoConfiguration {

  @Autowired                                                                                                                              
  private TestServiceProperties testServiceProperteis;                                                                                  
                                                                                                                                          
  @Bean                                                                                                                                   
  @ConditionalOnMissingBean(TestService.class)                                                                                           
  public TestService testService() {                                                                                                    
    TestService testService = new TestService();                                                                                     
    testService.setMessage(testServiceProperteis.getMessage());                                                                               
    return testService;                                                                                                                
  }                     
}
```

- ** @Configuration：标识此类为一个spring配置类 **

- ** @EnableConfigurationProperties(value = TestServiceProperties.class):启动配置文件，value用来指定我们要启用的配置类，多个时配置类时value={xxProperties1.class,xxProperteis2.class....} **

- ** @ConditionalOnClass(TestService.class):表示当classPath下存在TestService.class文件时该配置文件类才有效 **

- ** @ConditionalOnProperty(prefix = "test", value = "enable", matchIfMissing = true):表示只有我们的配置文件是否配置了以test为前缀的资源项值，并且在该资源项值为enable，如果没有配置我们默认设置为enable **

# 新建spring.factories
----
在src/main/resources 文件夹下新建文件夹 META-INF，在新建的META-INF文件夹下新建 spring.factories。
```bash
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.test.TestAutoConfiguration
```

# 测试运行
----

在SpringBoot项目中引入test-spring-boot-starter依赖
```bash
<dependency>
    <groupId>com.test</groupId>
    <artifactId>test-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

在application.properties进行配置
```bash
test.message = test spring boot starter
```

注入Starter提供的TestService看它是否正常
```bash
@Autowired
private TestService testService;

@@RequestMapping("/test")
public String test(){
	return testService.sayMessage();
}
```

