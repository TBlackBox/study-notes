[DruidDataSource配置属性列表 · alibaba/druid Wiki (github.com)](https://github.com/alibaba/druid/wiki/DruidDataSource配置属性列表)

1. 确保配置添加了如下配置，确保使用的是druid 数据源

   ```yml
       filters: stat
   ```

2. 添加如下配置

   ```java
    /**
        * druidServlet注册 配置一个管理后台的Servlet
        */
       @Bean
       public ServletRegistrationBean druidServletRegistration() {
           ServletRegistrationBean registration = new ServletRegistrationBean(new StatViewServlet());
           registration.addUrlMappings("/druid/*");
           Map<String,String> initParams = new HashMap<>();
           initParams.put("loginUsername","test");
           initParams.put("loginPassword","654321");
           initParams.put("allow","");    /**默认就是允许所有访问*/
           registration.setInitParameters(initParams);
   
           return registration;
       }
   
       /**
        * druid监控 配置URI拦截策略  配置一个web监控的filter
        */
       @Bean
       public FilterRegistrationBean druidStatFilter() {
           FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(new WebStatFilter());
   
           //添加过滤规则.
           filterRegistrationBean.addUrlPatterns("/*");
           //添加不需要忽略的格式信息.
           filterRegistrationBean.addInitParameter(
                   "exclusions", "/static/*,*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid,/druid/*");
           //用于session监控页面的用户名显示 需要登录后主动将username注入到session里
           filterRegistrationBean.addInitParameter("principalSessionName", "username");
           return filterRegistrationBean;
       }
   
       /**
        * druid数据库连接池监控
        */
       @Bean
       public DruidStatInterceptor druidStatInterceptor() {
           return new DruidStatInterceptor();
       }
   
       /**
        * druid数据库连接池监控
        */
       @Bean
       public BeanTypeAutoProxyCreator beanTypeAutoProxyCreator() {
           BeanTypeAutoProxyCreator beanTypeAutoProxyCreator = new BeanTypeAutoProxyCreator();
           beanTypeAutoProxyCreator.setTargetBeanType(DruidDataSource.class);
           beanTypeAutoProxyCreator.setInterceptorNames("druidStatInterceptor");
           return beanTypeAutoProxyCreator;
       }
   
   ```

3. 测试

   ```
   http://127.0.0.1:8080/druid/index.html
   用户名和密码，在上面的配置中
   ```

   



