---
title: Spring Boot笔记
date: 2017-07-20 10:00:00
tags: [Spring Boot]
categories: Spring Boot
---

# 静态资源访问
----

Spring Boot默认提供静态资源目录位置需置于classpath下，目录名需符合如下规则：

- /static
- /public
- /resources
- /META-INF/resources

举例：我们可以在src/main/resources/目录下创建static，在该位置放置一个图片文件。启动程序后，尝试访问http://localhost:8080/D.jpg


# 模板引擎
----

Spring Boot提供了默认配置的模板引擎主要有以下几种：

- Thymeleaf
- FreeMarker
- Velocity
- Groovy
- Mustache

** 模板默认配置路径为：src/main/resources/templates **


# Thymeleaf
----

Thymeleaf是一个XML/XHTML/HTML5模板引擎，可用于Web与非Web环境中的应用开发。

Spring Boot中使用Thymeleaf，只需要引入下面依赖，并在默认的模板路径src/main/resources/templates下编写模板文件即可完成。
```bash
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```


示例模板：
```bash
<table>
  <thead>
    <tr>
      <th th:text="#{msgs.headers.name}">Name</td>
      <th th:text="#{msgs.headers.price}">Price</td>
    </tr>
  </thead>
  <tbody>
    <tr th:each="prod : ${allProducts}">
      <td th:text="${prod.name}">Oranges</td>
      <td th:text="${#numbers.formatDecimal(prod.price,1,2)}">0.99</td>
    </tr>
  </tbody>
</table>
```

# Thymeleaf的默认参数配置
----

如有需要修改默认配置的时候，只需复制下面要修改的属性到application.properties中，并修改成需要的值，如修改模板文件的扩展名，修改默认的模板路径等。
```bash
# Enable template caching.
spring.thymeleaf.cache=true 
# Check that the templates location exists.
spring.thymeleaf.check-template-location=true 
# Content-Type value.
spring.thymeleaf.content-type=text/html 
# Enable MVC Thymeleaf view resolution.
spring.thymeleaf.enabled=true 
# Template encoding.
spring.thymeleaf.encoding=UTF-8 
# Comma-separated list of view names that should be excluded from resolution.
spring.thymeleaf.excluded-view-names= 
# Template mode to be applied to templates. See also StandardTemplateModeHandlers.
spring.thymeleaf.mode=HTML5 
# Prefix that gets prepended to view names when building a URL.
spring.thymeleaf.prefix=classpath:/templates/ 
# Suffix that gets appended to view names when building a URL.
spring.thymeleaf.suffix=.html
# Order of the template resolver in the chain.
spring.thymeleaf.template-resolver-order= 
# Comma-separated list of view names that can be resolved.
spring.thymeleaf.view-names= 
```


# Thymeleaf示例
----
```bash
@Controller
public class HelloController {
    @RequestMapping("/")
    public String index(ModelMap map) {
        // 加入一个属性，用来在模板中读取
        map.addAttribute("host", "http://blog.didispace.com");
        // return模板文件的名称，对应src/main/resources/templates/index.html
        return "index";  
    }
}
```

```bash
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8" />
    <title></title>
</head>
<body>
<h1 th:text="${host}">Hello World</h1>
</body>
</html>
```

# 整合Spring Security
----

## 核心概念
Principle(User), Authority(Role)和Permission是Spring Security的3个核心概念。跟通常理解上Role和Permission之间一对多的关系不同，在Spring Security中，Authority和Permission是两个完全独立的概念，两者并没有必然的联系，但可以通过配置进行关联。

## 添加依赖
在pom.xml中添加如下配置，引入对Spring Security的依赖。
```bash
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

## Spring Security配置
创建Spring Security的配置类WebSecurityConfig，具体如下：
```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/", "/home").permitAll()
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login")
                .permitAll()
                .and()
            .logout()
                .permitAll();
    }

    /**
    * Web层面的配置:一般用来配置无需安全检查的资源路径
    */
    @Override
    public void configure(WebSecurity web) throws Exception {
      web.ignoring().antMatchers("/js/**", "/css/**", "/images/**");
    }

    /**
    * 身份验证配置，用于注入自定义身份验证Bean和密码校验规则
    */
    /*@Override
    public void configure(AuthenticationManagerBuilder auth) throws Exception {
      auth.userDetailsService(detailsService).passwordEncoder(new BCryptPasswordEncoder());
    }*/

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth
            .inMemoryAuthentication()
                .withUser("user").password("password").roles("USER");
    }
}
```
- 通过@EnableWebSecurity注解开启Spring Security的功能
- 继承WebSecurityConfigurerAdapter，并重写它的方法来设置一些web安全的细节
- configure(HttpSecurity http)方法
	* 通过authorizeRequests()定义哪些URL需要被保护、哪些不需要被保护。例如以上代码指定了/和/home不需要任何认证就可以访问，其他的路径都必须通过身份验证。
	* 通过formLogin()定义当需要用户登录时候，转到的登录页面。
- configureGlobal(AuthenticationManagerBuilder auth)方法，在内存中创建了一个用户，该用户的名称为user，密码为password，用户角色为USER。



