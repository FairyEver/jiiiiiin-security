- ![代码结构](https://ws2.sinaimg.cn/large/006tNbRwgy1fue02z4h20j31kw0rc0y8.jpg)
- ![属性配置](https://ws1.sinaimg.cn/large/006tNbRwgy1fuilgv8orxj30rx0emgm6.jpg)


### 关键点

+ ![RESTFul API](https://ws3.sinaimg.cn/large/006tNbRwgy1fufeoc5gxdj31kw0yswnl.jpg)

+ spring security 相关：

    > https://spring.io/projects/spring-security
    > https://docs.spring.io/spring-security/site/docs/4.2.7.RELEASE/reference/htmlsingle/

    - 核心功能：
        
        ![](https://ws2.sinaimg.cn/large/006tNbRwgy1fuia8dpyrej30dv0bvdg4.jpg)
        
        ![](https://ws3.sinaimg.cn/large/006tNbRwgy1fuib0js6rhj31kw0ju0vk.jpg)
        
    + 浏览器相关security配置参考: bean::BrowserSecurityConfig、MyUserDetailsService
    
        - 个性化用户认证流程
            
            - 自定义登录页面
            
                ![](https://ws3.sinaimg.cn/large/006tNbRwgy1fuiiuwx7wxj31kw0s5wjo.jpg)
                为了区分渠道，需要像上图那样去重新定义security框架的处理逻辑；
                
            - 自定义登录成功处理流程
            - 自定义登录失败处理流程


+ 使用Swagger自动生成文档
    
    根据代码自动生成文档，提供给前端开发人员识别接口；
    
    [集成](https://mvnrepository.com/artifact/io.springfox/springfox-swagger2)
    
    ![](https://ws1.sinaimg.cn/large/006tNbRwgy1fui7xqrglgj31kw0ocab6.jpg)
    
    ```java
        @SpringBootApplication
        @RestController
        @EnableSwagger2
        public class App {}
      
      // 常用注解：@ApiParam、@ApiOperation
    
        @GetMapping("/{id:\\d+}")
        @JsonView(User.UserDetailView.class)
        @ApiOperation(value = "用户查询服务")
        public User getUserInfo(@ApiParam(value = "用户id") @PathVariable String id, @PathVariable(name = "id") Long idddd) {
    
      // 常用注解：@ApiModelProperty
      public class UserQryCondition {
  
          private String username;
          @ApiModelProperty(value = "年龄起始值")
          private int age;
          // 年龄区间
          @ApiModelProperty(value = "年龄终止值")
    ```
    
    - 自定义
    
        `UserDetailsService`用来通过用户名获取`UserDetails`用户标识对象；
        

+ 异步处理REST服务

    - 使用Callable来处理
    
    ![]((https://ws4.sinaimg.cn/large/006tNbRwgy1fuhcqkylckj31kw0rm0v2.jpg))
    
    如果使用异步方式处理请求，那么请求就不占用容器的【主线程】的资源数，使得容器可以处理更多请求而不被阻塞，提升服务器的吞吐量；
    对于浏览器来说，还是一个【正常】的请求，因为耗时还是1.x秒得到响应；
    
    特点：
        子线程必须是在主线程中开启
        
    - 有一种情况：
    
    ![](https://ws1.sinaimg.cn/large/006tNbRwgy1fuhdliloxxj30x40ezmxz.jpg)
    
    即接收请求的服务器是一个前置服务器，真正进行耗时操作的是外围一台服务器， 应用1只负责接收消息，之后将消息放到消息队列，而真正处理业务是在应用2中去监控消息队列，取出并处理，处理完毕之后将结果返给消息队列，应用1在实时的去取结果；
    而真正的请求的响应式在应用1从队列中取出处理结果之后返回给前端的；
       
    针对这种场景，上面的处理方式就不能满足需求，需要使用DeferredResult来进行处理，参考`@RequestMapping("/async/order2")处理逻辑
    
    还需要注意：
    
    ```java
        /**
         * 针对异步接口的拦截器配置需要通过下面的接口进行注册（相应的拦截器也需要重写）
         * 否则常规的拦截器是拦截不到异步的接口
         *
         * @param configurer
         */
        @Override
        public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
            super.configureAsyncSupport(configurer);
    //        configurer.registerDeferredResultInterceptors()
    //        configurer.registerCallableInterceptors()
        }
    ```
       

+ springboot aop、拦截器、过滤器相关:

    ![springboot aop、拦截器、过滤器相关](https://ws3.sinaimg.cn/large/006tNbRwgy1fuh5paluivj30n90glgm0.jpg)
    
    `@ControllerAdvice`就是如控制器异常处理类上面声明的注解；
    
    如果控制器中抛出的异常，在外围还是抛出的话，那就会进行层层传递；
    
    如果filter还没有处理异常，就会抛到容器（如：tomcat）最终显示到前端；
        

+ springboot 默认错误处理控制器 `org.springframework.boot.autoconfigure.web.BasicErrorController`

    ```java
    
    // /error是默认错误视图的路径
    @Controller
    @RequestMapping("${server.error.path:${error.path:/error}}")
    public class BasicErrorController extends AbstractErrorController {
    
        // 表示请求参数头中的Accept被认定为html的时候进入该接口进行错误处理
        @RequestMapping(produces = "text/html")
        
        // 否则进入：
        @RequestMapping
        @ResponseBody
        public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
    ```
    
    手动处理404网页版：需要在`/jiiiiiin-security/jiiiiiin-security-demo/src/main/resources/resources/error/404.html`创建对应状态码的页面；
    这样对应状态码的网页版的错误页面就可以被订制；
            

### 问题集合：

+ spring security 常见错误：

    + 默认开启了csrf，但是没有做合理配置，登录的时候就报：
        ```xml
        There was an unexpected error (type=Forbidden, status=403).
        Could not verify the provided CSRF token because your session was not found.
        ```
        调试期间可以先关闭这个特性：
        ```java
          .and()
                .csrf().disable();
        ```

**需要先创建对应数据库，启动redis服务**

+ [redis安装](https://www.youtube.com/watch?v=JGvbEk4jtrU)

+ `Cannot determine embedded database driver class for database type NONE`：

    因为在`jiiiiiin-security-core`中声明了jdbc依赖，但是没有配置数据库链接配置则会报错：
    
    ```xml
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-jdbc</artifactId>
            </dependency>
    ```
    
    解决，配置jdbc链接配置信息：
    
    ```properties
    spring:
      datasource:
        driver-class-name: com.mysql.jdbc.Driver
    #    http://database.51cto.com/art/201005/199278.htm
        url: jdbc:mysql://127.0.0.1:3306/jiiiiiin-security-demo?&useUnicode=true&characterEncoding=utf8&autoReconnect=true&failOverReadOnly=false
        username: root
        password: root

    ```
    
+ `Caused by: java.lang.IllegalArgumentException: No Spring Session store is configured: set the 'spring.session.store-type' property`
    
    因为`jiiiiiin-security-browser`中声明了spring session的依赖，但是没有配置相关依赖配置故报错：
    解决先关闭依赖配置：
    
    ```properties
    spring:
      session:
        store-type: none
    ```
    
+ 项目启动访问需要“登录信息”，因为spring security默认开启认证，可以先关闭默认配置：

    ```properties
    security:
      basic:
        enabled: false
    ```

+ 怎么将demo项目打出可执行jar：
    
    解决：
    ```xml
         <plugins>
            <!--将项目打包成一个可执行的jar https://docs.spring.io/spring-boot/docs/1.5.15.RELEASE/reference/htmlsingle/#getting-started-first-application-executable-jar-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

        </plugins>
    ```
    
+ `java.lang.AssertionError: No value at JSON path "$.length()": java.lang.IllegalArgumentException: json can not be null or empty`

    因为测试的方法没有返回正确的json数据；
    
        
    