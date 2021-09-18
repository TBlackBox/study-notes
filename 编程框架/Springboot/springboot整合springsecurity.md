# 简介
Spring Security 是一个框架，侧重于为 Java 应用程序提供身份验证和授权。与所有 Spring 项目一样，Spring 安全性的真正强大之处，在于它很容易扩展以满足定制需求。

# 简单集成
新建springboot工程
1. 引入pom配置
```xml
<!-- 实现对 Spring MVC 的自动化配置 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- 实现对 Spring Security 的自动化配置 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
2. 添加yml配置
```yaml
spring:
  # Spring Security 配置项，对应 SecurityProperties 配置类
  security:
    # 配置默认的 InMemoryUserDetailsManager 的用户账号与密码。
    user:
      name: test # 账号
      password: test # 密码
      roles: admin # 拥有角色

#      在 spring.security 配置项，设置 Spring Security 的配置，对应 SecurityProperties 配置类。
#      默认情况下，Spring Boot UserDetailsServiceAutoConfiguration 自动化配置类，会创建一个内存级别的 InMemoryUserDetailsManager Bean 对象，提供认证的用户信息。
#      这里，我们添加了 spring.security.user 配置项，UserDetailsServiceAutoConfiguration 会基于配置的信息创建一个用户 User 在内存中。
#      如果，我们未添加 spring.security.user 配置项，UserDetailsServiceAutoConfiguration 会自动创建一个用户名为 "user" ，密码为 UUID 随机的用户 User 在内存中。
```

3. 添加测试类
```java
@RestController
@RequestMapping("/test")
public class TestController {

    @GetMapping("/security")
    public String security(){
        return "安全测试";
    }
}
```
4. 测试
访问
```
http://127.0.0.1:8080/test/security
```
会弹出spring security内置的安全登录界面，输入登录账号和密码即可。


# 进阶集成
1. 在上一个项目下，添加如下配置文件
```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception{
        auth.
                // 使用内存中的 InMemoryUserDetailsManager
                //调用 AuthenticationManagerBuilder#inMemoryAuthentication() 方法，使用内存级别的 InMemoryUserDetailsManager Bean 对象，提供认证的用户信息。
                //    Spring 内置了两种 UserDetailsManager 实现：
                //    InMemoryUserDetailsManager，和「2. 快速入门」是一样的。
                //    JdbcUserDetailsManager ，基于 JDBC的 JdbcUserDetailsManager 。
                //    实际项目中，我们更多采用调用 AuthenticationManagerBuilder#userDetailsService(userDetailsService) 方法，使用自定义实现的 UserDetailsService 实现类，更加灵活且自由的实现认证的用户信息的读取。
                        inMemoryAuthentication()
                // 不使用 PasswordEncoder 密码编码器
                //passwordEncoder(passwordEncoder) 方法，设置 PasswordEncoder 密码编码器。
                //在这里，为了方便，我们使用 NoOpPasswordEncoder 。实际上，等于不使用 PasswordEncoder ，不配置的话会报错。
                .passwordEncoder(NoOpPasswordEncoder.getInstance())
                // 配置了「admin/admin」和「normal/normal」两个用户，分别对应 ADMIN 和 NORMAL 角色。相比「2. 快速入门」来说，可以配置更多的用户。
                .withUser("admin").password("admin").roles("ADMIN")
                .and().withUser("normal").password("normal").roles("NORMAL");
    }

    //主要配置 URL 的权限控制。
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                // <X> 配置请求地址的权限
                .authorizeRequests()
                .antMatchers("/test/all").permitAll() // 所有用户可访问
                .antMatchers("/test/admin").hasRole("ADMIN") // 需要 ADMIN 角色
                .antMatchers("/test/normal").access("hasRole('ROLE_NORMAL')") // 需要 NORMAL 角色。
                // 任何请求，访问的用户都需要经过认证
                .anyRequest().authenticated()
                .and()
                // <Y> 设置 Form 表单登录
                .formLogin()
//                    .loginPage("/login") // 登录 URL 地址
                .permitAll() // 所有用户可访问
                .and()
                // 配置退出相关
                //自定义退出 可以通过 #logoutUrl(String logoutUrl) 方法，来进行设置。不设置使用默认的退出
                .logout()
//                    .logoutUrl("/logout") // 退出 URL 地址
                .permitAll(); // 所有用户可访问
    }

//    调用 HttpSecurity#authorizeRequests() 方法，开始配置 URL 的权限控制。
//            #(String... antPatterns) 方法，配置匹配的 URL 地址，基于 Ant 风格路径表达式 ，可传入多个。
//            【常用】#permitAll() 方法，所有用户可访问。
//            【常用】#denyAll() 方法，所有用户不可访问。
//            【常用】#authenticated() 方法，登录用户可访问。
//            #anonymous() 方法，无需登录，即匿名用户可访问。
//            #rememberMe() 方法，通过 remember me 登录的用户可访问。
//            #fullyAuthenticated() 方法，非 remember me 登录的用户可访问。
//            #hasIpAddress(String ipaddressExpression) 方法，来自指定 IP 表达式的用户可访问。
//            【常用】#hasRole(String role) 方法， 拥有指定角色的用户可访问。
//            【常用】#hasAnyRole(String... roles) 方法，拥有指定任一角色的用户可访问。
//            【常用】#hasAuthority(String authority) 方法，拥有指定权限(authority)的用户可访问。
//            【常用】#hasAuthority(String... authorities) 方法，拥有指定任一权限(authority)的用户可访问。
//            【最牛】#access(String attribute) 方法，当 Spring EL 表达式的执行结果为 true 时，可以访问。
}
```
覆盖上面的两个方法，实现用户和路径的权限控制

2. 添加测试方法
```java
@RestController
@RequestMapping("/test")
public class TestController {

    @GetMapping("/security")
    public String security(){
        return "安全测试";
    }

    @GetMapping("/all")
    public String all(){
        return "所有用户都能访问";
    }

    @GetMapping("/admin")
    public String admin(){
        return "admin用户访问";
    }

    @GetMapping("/normal")
    public String normal(){
        return "normal角色的能访问";
    }

}
```

3. 进行相应的测试
如果访问`http://127.0.0.1:8080/test/admin`，登录normal账号和密码，将会出现403紧致访问。

# 更上一层楼
通过注解的形式，实现权限控制
1. 在配置类上添加如下注解`@EnableGlobalMethodSecurity(prePostEnabled = true)`

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter
```

2. 通过注解的形式实现权限控制
```java
  //@PermitAll 注解，等价于 #permitAll() 方法，所有用户可访问。
    //因为在SecurityConfig中，配置了 .anyRequest().authenticated() ，任何请求，访问的用户都需要经过认证。所以这里 @PermitAll 注解实际是不生效的。
    //也就是说，Java Config 配置的权限，和注解配置的权限，两者是叠加的。
    @PermitAll
    @GetMapping("/all1")
    public String all1() {
        return "所有用户都能访问";
    }

    @GetMapping("/home")
    public String home() {
        return "我是首页";
    }

    @PreAuthorize("hasRole('ROLE_ADMIN')")
    @GetMapping("/admin1")
    public String admin1() {
        return "admin1用户访问";
    }

    //@PreAuthorize 注解，等价于 #access(String attribute) 方法，，当 Spring EL 表达式的执行结果为 true 时，可以访问。
    @PreAuthorize("hasRole('ROLE_NORMAL')")
    @GetMapping("/normal1")
    public String normal1() {
        return "normal1角色的能访问";
    }

    //还有一些其他的注解可以到官网学习
```

3. 进行相应的测试

# 相应资料
https://www.iocoder.cn/Spring-Boot/Spring-Security/?yudao
- 整合 Spring Session  
https://www.iocoder.cn/Spring-Boot/Distributed-Session/?self

- 整合 OAuth2  
https://www.iocoder.cn/Spring-Security/OAuth2-learning/?self

- 整合 JWT  
https://www.iocoder.cn/Fight/Separate-SpringBoot-SpringSecurity-JWT-RBAC-from-front-and-rear-to-achieve-user-stateless-request-authentication/?self