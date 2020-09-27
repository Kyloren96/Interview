# Spring

## **SpringMVC**

### 原理

<img src="https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcuZ3Vveml5YW5nLnRvcC9pbWFnZXMvMjAyMC8wNi8wOC82NDAuanBn?x-oss-process=image/format,png" alt="640.jpg" style="zoom:150%;" />

1. 用户发送请求至前端控制器DispatcherServlet。
2. DispatcherServlet收到请求调用HandlerMapping处理器映射器。
3. 处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。
4. DispatcherServlet通过HandlerAdapter处理器适配器调用处理器。
5. 执行处理器(Controller，也叫后端控制器)。
6. Controller执行完成返回ModelAndView。
7. HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet。
8. DispatcherServlet将ModelAndView传给ViewReslover视图解析器。
9. ViewReslover解析后返回具体View。
10. DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）。
11. DispatcherServlet响应用户。

### Springmvc 组件

**1、**前端控制器DispatcherServlet（不需要工程师开发）,由框架提供（重要）

作用：**Spring MVC 的入口函数。接收请求，响应结果，相当于转发器，中央处理器。有了 DispatcherServlet 减少了其它组件之间的耦合度。用户请求到达前端控制器，它就相当于mvc模式中的c，DispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，DispatcherServlet的存在降低了组件之间的耦合性。**

**2、处理器映射器HandlerMapping(不需要工程师开发),由框架提供**

作用：根据请求的url查找Handler。HandlerMapping负责根据用户请求找到Handler即处理器（Controller），SpringMVC提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

**3、处理器适配器HandlerAdapter**

作用：按照特定规则（HandlerAdapter要求的规则）去执行Handler 通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

**4、处理器Handler(需要工程师开发)**

注意：编写Handler时按照HandlerAdapter的要求去做，这样适配器才可以去正确执行Handler Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。 由于Handler涉及到具体的用户业务请求，所以一般情况需要工程师根据业务需求开发Handler。

**5、视图解析器View resolver(不需要工程师开发),由框架提供**

作用：进行视图解析，根据逻辑视图名解析成真正的视图（view） View Resolver负责将处理结果生成View视图，View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。 springmvc框架提供了很多的View视图类型，包括：jstlView、freemarkerView、pdfView等。 一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由工程师根据业务需求开发具体的页面。

**6、视图View(需要工程师开发)**

View是一个接口，实现类支持不同的View类型（jsp、freemarker、pdf...）

### 常用注解

**类型类**

- @Controller：负责注册一个bean 到spring 上下文中

- @Service

- @Repository

- @Component

- @Configuration：声明当前类为配置类，相当于xml形式的Spring配置

- @Bean：注解在方法上，声明当前方法的返回值为一个 bean

  **@Bean和@Component的区别**

  - @Component 在类上使用，表示这是一个组件类，需要 Spring 为这个类创建 Bean
  - @Bean 在方法上使用，告诉 Spring 这个方法将返回一个 Bean 对象，需要把返回的对象注册到应用上下文中

**设置类**

- @Required：确保值一定被设置
- @Autowired && @Qualifier
  - @Qualifier：当一个接口有多个实现的时候，为了指名具体调用哪个类的实现。
- @Scope：生命周期

**Web类**

- @RequestMapping && @GetMapping @ PostMapping

  - RequestMapping：用于映射Web请求，包括访问路径和参数（类或方法上）

- @PathVariable && @RequestParam

  - @PathVariable：用于接收路径参数，比如@RequestMapping(“/hello/{name}”)申明的路径，将注解放在参数中前，即可获取该值，通常作为Restful的接口实现方法。

- @RequestBody && @ResponseBody

  - @RequestBody 接收JSON数据

    该注解用于读取 Request 请求的 body 部分数据。允许request的参数在request体中，而不是在直接连接在地址后面。（放在参数前）

  - @ResponseBody 将java对象转换成json格式的字符串，返回给浏览器。该注解用于将 Controller 的方法返回的对象，写入到 Response 对象的 body 数据区。 （返回值旁或方法上）

**功能类**

- @ImportResource：引用类
- @ComponentScan：自动扫描
- @EnableCaching && Cacheable： 开启注解式的缓存支持/缓存
- @Transactional：开启事务
- @Aspect && Poincut：切面和切点
- @Scheduled：来申明这是一个任务

- @RequestController **@RestController=@ResponseBody+@Controller** 意味着，该Controller的所有方法都默认加上了@ResponseBody。
- @ControllerAdvice 使一个Contoller成为全局的异常处理类,类中用@ExceptionHandler方法注解的方法可以处理所有Controller发生的异常。
- @ModelAttribute最主要的作用是将数据添加到模型对象中，用于视图页面展示时使用。 等价于 model.addAttribute("attributeName", abc)。



## **Spring Boot**

**如果必须启动一个新的 Spring 项目，我们必须添加构建路径或 maven 依赖项，配置 application server，添加 Spring 配置。因此，启动一个新的 Spring 项目需要大量的工作，因为我们目前必须从头开始做所有事情**。 Spring Boot 是这个问题的解决方案。Spring boot 构建在现有 Spring 框架之上。使用 Spring boot，可以避免以前必须执行的所有样板代码和配置。

### Spring Boot 的优点是什么?

编码：减少开发、测试的时间和工作量。

配置：使用 JavaConfig 有助于避免使用 XML。没有 web.xml 文件，只需添加带 @ configuration 注释的类。

避免大量 maven 导入和各种版本冲突。

部署：不需要单独的 Web 服务器。这意味着您不再需要启动 Tomcat 或其他任何东西。



### Spring Boot 启动流程⭐

首先 prepareEnvironment 配置环境，然后准备 Context 上下文，ApplicationContext 的后置处理器，初始化 lnitializers，通知处理上下文准备和加载时间，然后开始refresh。



### Spring Boot提供了两种常用的配置文件：

- properties文件

- yml文件

  

### Spring Boot 的核心注解是哪个？⭐

启动类上面的注解是@SpringBootApplication，它也是 Spring Boot 的核心注解，主要组合包含了以下 3 个注解：

@SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能。

@EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能： @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。

@ComponentScan：Spring组件扫描。



### Spring Boot 自动配置原理是什么⭐

@EnableAutoConfiguration这个注解开启自动配置，它的作用：

- 利用EnableAutoConfigurationImportSelector给容器中导入一些组件
- 这个类父类有一个方法：selectImports()，这个方法返回 **configurations** ：
- List configurations = getCandidateConfigurations(annotationMetadata, attributes);获取候选的配置
- 将 类路径下 META-INF/spring.factories 里面配置的所有EnableAutoConfiguration的值加入到了容器中
- 加载某个组件的时候根据注解的条件判断每个加入的组件是否生效，如果生效就把类的属性和配置文件绑定起来
- 这时就读取配置文件的值加载组件



### SpringBoot 拦截器和过滤器

 1、Filter是依赖于Servlet容器，属于Servlet规范的一部分，而拦截器则是独立存在的，可以在任何情况下使用。

　　2、Filter的执行由Servlet容器回调完成，而拦截器通常通过动态代理的方式来执行。

　　3、Filter的生命周期由Servlet容器管理，而拦截器则可以通过IoC容器来管理，因此可以通过注入等方式来获取其他Bean的实例，因此使用会更方便。



### SpringBoot、SpringMVC和Spring区别

spring boot只是一个配置工具,整合工具,辅助工具.

springmvc是框架,项目中实际运行的代码

Spring 框架就像一个家族，有众多衍生产品例如 boot、security、jpa等等。但他们的基础都是Spring 的ioc和 aop，ioc 提供了依赖注入的容器， aop解决了面向横切面的编程，然后在此两者的基础上实现了其他延伸产品的高级功能。

用最简练的语言概括就是：

- Spring 是一个“引擎”；
- Spring MVC 是基于Spring的一个 MVC 框架；
- Spring Boot 是基于Spring的条件注册的一套快速开发整合包



### spring boot处理一个http请求的全过程

- 由前端发起请求
- 根据路径，Springboot会加载相应的Controller进行拦截
- 拦截处理后，跳转到相应的Service处理层
- 跳转到ServiceImplement(service实现类)
- 在执行serviceimplement时会加载Dao层，操作数据库
- 再跳到Dao层实现类
- 执行会跳转到mapper层
- 然后MallMapper会继续找对应的mapper.xml配置文件
- 之后便会跳转到第4步继续执行，执行完毕后会将结果返回到第1步，然后
- 便会将数据以JSON的形式返回到页面，同时返回状态码，正常则会返回200，便会回到步骤1中查询判断。

### **Spring事务隔离级别和事务传播属性** ⭐

**隔离级别：**

**1) DEFAULT （默认）**
 这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别。另外四个与JDBC的隔离级别相对应。

**2) READ_UNCOMMITTED （读未提交）**
 这是事务最低的隔离级别，它允许另外一个事务可以看到这个事务未提交的数据。这种隔离级别会产生脏读，不可重复读和幻像读。 

**3) READ_COMMITTED （读已提交）**
 保证一个事务修改的数据提交后才能被另外一个事务读取，另外一个事务不能读取该事务未提交的数据。这种事务隔离级别可以避免脏读出现，但是可能会出现不可重复读和幻像读。 

**4) REPEATABLE_READ （可重复读）**
 这种事务隔离级别可以防止脏读、不可重复读，但是可能出现幻像读。它除了保证一个事务不能读取另一个事务未提交的数据外，还保证了不可重复读。

**5) SERIALIZABLE（串行化）**
 这是花费最高代价但是最可靠的事务隔离级别，事务被处理为顺序执行。除了防止脏读、不可重复读外，还避免了幻像读。 



**Spring事务传播属性：**

**1) REQUIRED（默认属性）**
 如果存在一个事务，则支持当前事务。如果没有事务则开启一个新的事务。 被设置成这个级别时，会为每一个被调用的方法创建一个逻辑事务域。如果前面的方法已经创建了事务，那么后面的方法支持当前的事务，如果当前没有事务会重新建立事务。 

**2) MANDATORY**
 支持当前事务，如果当前没有事务，就抛出异常。 

**3) NEVER**
 以非事务方式执行，如果当前存在事务，则抛出异常。 

**4) NOT_SUPPORTED**
 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 

**5) REQUIRES_NEW**
 新建事务，如果当前存在事务，把当前事务挂起。 

**6) SUPPORTS**
 支持当前事务，如果当前没有事务，就以非事务方式执行。 

**7) NESTED**
 支持当前事务，新增Savepoint点，与当前事务同步提交或回滚。 嵌套事务一个非常重要的概念就是内层事务依赖于外层事务。外层事务失败时，会回滚内层事务所做的动作。而内层事务操作失败并不会引起外层事务的回滚。



### Spring 框架中用到了哪些设计模式？

- **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。



## MyBatis

### **防止sql注入：**

 在编写mybatis的映射语句时，尽量采用“#{xxx}”这样的格式

### **#和$区别：** 

| #                                                       | $                                   |
| ------------------------------------------------------- | ----------------------------------- |
| 相当于对数据加上双引号                                  | 相当于直接显示数据                  |
| 很大程度上防止SQL注入                                   | 无法防止SQL注入                     |
| #{xxx},使用的是PreparedStatement,会有类型转换，比较安全 | ${xxx}，使用字符串拼接，容易SQL注入 |

 简单的说就是#{}是经过预编译的，是安全的，**$**{}是未经过预编译的，仅仅是取变量的值，是非安全的，存在SQL注入。

### 要实现动态传入表名、列名：

**添加属性statementType="STATEMENT"**，**同时sql里的属有变量取值都改成${xxxx}**

### **Mybatis和Hibernate的区别** 

**Hibernate 框架：** 

 **Hibernate**是一个开放源代码的对象关系映射框架,它对JDBC进行了非常轻量级的对象封装,建立对象与数据库表的映射。是一个全自动的、完全面向对象的持久层框架。

**Mybatis框架：**

 **Mybatis**是一个开源对象关系映射框架，原名：ibatis,2010年由谷歌接管以后更名。是一个半自动化的持久层框架。

**区别：**

####  **开发方面**

 在项目开发过程当中，就速度而言：

 hibernate开发中，sql语句已经被封装，直接可以使用，加快系统开发；

 Mybatis 属于半自动化，sql需要手工完成，稍微繁琐；

 但是，凡事都不是绝对的，如果对于庞大复杂的系统项目来说，复杂语句较多，hibernate 就不是好方案。

####  **sql优化方面**

 Hibernate 自动生成sql,有些语句较为繁琐，会多消耗一些性能；

 Mybatis 手动编写sql，可以避免不需要的查询，提高系统性能；

####  **对象管理比对**

 Hibernate 是完整的对象-关系映射的框架，开发工程中，无需过多关注底层实现，只要去管理对象即可；

 Mybatis 需要自行管理 映射关系；

## JDBC连接数据库  

•创建一个以JDBC连接数据库的程序，包含7个步骤：  

####  1、加载JDBC驱动程序：  

  在连接数据库之前，首先要加载想要连接的数据库的驱动到JVM（Java虚拟机），  

  这通过java.lang.Class类的静态方法forName(String className)实现。  

  例如：  

  **try**{  

  //加载MySql的驱动类  

  Class.forName("com.mysql.jdbc.Driver") ;  

  }**catch**(ClassNotFoundException e){  

  System.out.println("找不到驱动程序类 ，加载驱动失败！");  

  e.printStackTrace() ;  

  }  

  成功加载后，会将Driver类的实例注册到DriverManager类中。  

####  2、提供JDBC连接的URL  

  •连接URL定义了连接数据库时的协议、子协议、数据源标识。  

  •书写形式：协议：子协议：数据源标识  

  协议：在JDBC中总是以jdbc开始  

  子协议：是桥连接的驱动程序或是数据库管理系统名称。  

  数据源标识：标记找到数据库来源的地址与连接端口。  

  例如：（MySql的连接URL）  

  jdbc:mysql:  

​    //localhost:3306/test?useUnicode=true&characterEncoding=gbk ;  

  useUnicode=**true**：表示使用Unicode字符集。如果characterEncoding设置为  

  gb2312或GBK，本参数必须设置为**true** 。characterEncoding=gbk：字符编码方式。  

####  3、创建数据库的连接  

  •要连接数据库，需要向java.sql.DriverManager请求并获得Connection对象，  

   该对象就代表一个数据库的连接。  

  •使用DriverManager的getConnectin(String url , String username ,  

  String password )方法传入指定的欲连接的数据库的路径、数据库的用户名和  

   密码来获得。  

   例如：  

   //连接MySql数据库，用户名和密码都是root  

   String url = "jdbc:mysql://localhost:3306/test" ;  

   String username = "root" ;  

   String password = "root" ;  

   **try**{  

  Connection con =  

​       DriverManager.getConnection(url , username , password ) ;  

   }**catch**(SQLException se){  

  System.out.println("数据库连接失败！");  

  se.printStackTrace() ;  

   }  

####  4、创建一个Statement  

  •要执行SQL语句，必须获得java.sql.Statement实例，Statement实例分为以下3 

   种类型：  

   1、执行静态SQL语句。通常通过Statement实例实现。  

   2、执行动态SQL语句。通常通过PreparedStatement实例实现。  

   3、执行数据库存储过程。通常通过CallableStatement实例实现。  

  具体的实现方式：  

​    Statement stmt = con.createStatement() ;  

​    PreparedStatement pstmt = con.prepareStatement(sql) ;  

​    CallableStatement cstmt =  

​              con.prepareCall("{CALL demoSp(? , ?)}") ;  

####  5、执行SQL语句  

  Statement接口提供了三种执行SQL语句的方法：executeQuery 、executeUpdate  

  和execute  

  1、ResultSet executeQuery(String sqlString)：执行查询数据库的SQL语句  

​    ，返回一个结果集（ResultSet）对象。  

   2、**int** executeUpdate(String sqlString)：用于执行INSERT、UPDATE或  

​    DELETE语句以及SQL DDL语句，如：CREATE TABLE和DROP TABLE等  

   3、execute(sqlString):用于执行返回多个结果集、多个更新计数或二者组合的  

​    语句。  

  具体实现的代码：  

​     ResultSet rs = stmt.executeQuery("SELECT * FROM ...") ;  

  **int** rows = stmt.executeUpdate("INSERT INTO ...") ;  

  **boolean** flag = stmt.execute(String sql) ;  

####  6、处理结果  

  两种情况：  

   1、执行更新返回的是本次操作影响到的记录数。  

   2、执行查询返回的结果是一个ResultSet对象。  

  • ResultSet包含符合SQL语句中条件的所有行，并且它通过一套get方法提供了对这些  

   行中数据的访问。  

  • 使用结果集（ResultSet）对象的访问方法获取数据：  

   **while**(rs.next()){  

​     String name = rs.getString("name") ;  

  String pass = rs.getString(1) ; // 此方法比较高效  

   }  

  （列是从左到右编号的，并且从列1开始）  

####  7、关闭JDBC对象  

   操作完成以后要把所有使用的JDBC对象全都关闭，以释放JDBC资源，关闭顺序和声  

   明顺序相反：  

   1、关闭记录集  

   2、关闭声明  

   3、关闭连接对象  

​     **if**(rs != **null**){  // 关闭记录集  

​    **try**{  

​      rs.close() ;  

​    }**catch**(SQLException e){  

​      e.printStackTrace() ;  

​    }  

​     }  

​     **if**(stmt != **null**){  // 关闭声明  

​    **try**{  

​      stmt.close() ;  

​    }**catch**(SQLException e){  

​      e.printStackTrace() ;  

​    }  

​     }  

​     **if**(conn != **null**){ // 关闭连接对象  

​     **try**{  

​      conn.close() ;  

​     }**catch**(SQLException e){  

​      e.printStackTrace() ;  

​     }  

​     } 