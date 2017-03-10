---
title: spring-boot 搭建-不解释
date: 2017-03-06 14:21:59
tags: java
layout: clean-blog
slug: spring-boot-first

---

## 本文为Spring-boot入门篇,老司机跳过
  
  1. 服务创建
  2. RestAPI创建(配置文件)
  3. 数据层（链接池）/服务层实现(事务，单表更新级查询)/整合接口服务
 
    
    
  

> ** 知识准备。 **
>  spring boot 基于 maven创建，如果不熟悉请先学习[maven 教程](http://www.yiibai.com/maven/)  

---
####  项目源码:[github源码地址](https://github.com/zccccccc/cici) 
---


### 1.创建maven web项目 (如果选择 springBoot模版创建速度非常慢，个人感觉直接创建比较简单)  
 
  * 创建maven web项目 (如果选择 springBoot模版创建速度非常慢，个人感觉直接创建比较)
  修改pom.xml,添加springboot依赖
  
  ```
   <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.1.RELEASE</version>
    </parent>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>


        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>


        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.0</version>
        </dependency>


        <dependency>
            <groupId>com.oracle</groupId>
            <artifactId>ojdbc14</artifactId>
            <version>10.2.0.4.0</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.0.18</version>
        </dependency>


    </dependencies>
  
  ```
 
  * 创建启动入口文件 运行main方法，服务搭建就OK啦
  
  ```
  @SpringBootApplication
public class MainApp extends WebMvcConfigurerAdapter {

    //第一种
    //1 cd  当前项目根目录下
    //1 mvn spring-boot:run

    //第二种
    //2 cd  当前项目根目录下
    //2. mvn install
    //2.  cd target/
    //2. java -jar cici-log.jar

    //第三种
    //直接运行main方法

    //启动时候选择配置文件启动
    //3 application.yml application-dev.yml application-pro.yml
    //3. mvn install
    //3.  cd target/
    //3. java -jar cici-log.jar -spring.profies.active=pro

    //后台启动
    /**
     *  nohup java -jar cici-log.jar &
     *  nohup java -jar cici-log.jar  /dev/null 2>&1 &
     *
     * */
    public static void main(String[] args){
        SpringApplication.run(MainApplation.class,args);
    }

  }
  
  ```
    
### 2.RestAPI创建   
   
   创建服务接口包括post/put/get/delete
   创建 SysUserController.java 
   添加注解
   
   ```
   @RestController
   @RequestMapping("/api/sysUser")
   ```
  
   完整服务接口代码
   
   ```
@RestController
@RequestMapping("/api/sysUser")
public class SysUserController {

    @GetMapping(value = "list")
    public List<CatUser> catalog(){
        return null;
    }

    //获取数据
    @GetMapping(value = "update/{id}")
    public CatUser get(@PathVariable("id")String id){
        return null;
    }

    //新增
    @PostMapping(value = "update")
    public Object add(@RequestBody Object c){

        return null;
    }

    //更新
    @PutMapping(value = "update")
    public CatUser update(@RequestBody CatUser c){

        return null;
    }

    //删除
    @DeleteMapping(value = "update/{id}")
    public void add(@PathVariable("id")String id){

    }
}

```
 
###  3. 数据层/服务层实现（事务，链接池）  
  
  当我们接口服务创建完成后开始创建服务层数据层代码
  
#### 1. 配置文件 spring boot 推荐使用.yml 配置，此方法少去很多冗余字段
#### 2. 我们创建 application.yml 
  
 ```

spring:
  profiles:
    active: dev
  jpa:
     hibernate:
       ddl-auto: none
     show-sql: true


 ```
 - spring.profiles.active: dev 读取配置文件为 application-dev.yml 的配置文件,目前用于测试环境和生   产环境配置分类 
 - jpa.hibernate.ddl-auto:  会根据绑定的实体(@Entity)生成 ,建议使用 none
 
 		+ validate   加载hibernate时，验证创建数据库表结构
 		+ create      每次加载hibernate，重新创建数据库表结构，这就是导致数据库表数据丢失的原因。
 		+ create-drop  加载hibernate时创建，退出是删除表结构
 		+ update        加载hibernate自动更新数据库结构
 - jpa.hibernate.show-sql: true 显示sql 
  
 
#### 3. 创建数据库配置 application-dev.yml 

```
server:
    port: 7777
spring:
  datasource:
      driver-class-name: oracle.jdbc.driver.OracleDriver
      url: jdbc:oracle:thin:@172.25.13.98:1521:data98
      username: HHZY_TEST
      password: HHZY_TEST
      type: com.alibaba.druid.pool.DruidDataSource
      spring.datasource.initialSize: 5
      spring.datasource.minIdle: 5
      spring.datasource.maxActive: 20
      spring.datasource.maxWait: 60000
      timeBetweenEvictionRunsMillis: 60000
      minEvictableIdleTimeMillis: 300000
      validationQuery: SELECT 1 FROM DUAL
      testWhileIdle: true
      testOnBorrow: false
      testOnReturn: false
      poolPreparedStatements: true
      maxPoolPreparedStatementPerConnectionSize: 20
      filters: stat,wall,log4j
      connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
      useGlobalDataSourceStat: true
logging.level.root: DEBUG
logging.level.org.springframework.web: DEBUG
logging.level.org.hibernate: DEBUG


```
 
   
   
- server.port: 7777 为当前服务端口


- 
spring.datasource 数据库连接配置当前使用连接池的是 [DruidDataSource](https://github.com/alibaba/druid/wiki/DruidDataSource%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7%E5%88%97%E8%A1%A8)
  + 创建 druid 访问[http://localhost:7777/druid/index.html](http://localhost:7777/druid/index.html)
  
      
  ```  
@SuppressWarnings("serial")
@WebServlet(urlPatterns = "/druid/*",  initParams = {
                @WebInitParam(name = "allow", value = "127.0.0.1"),// IP白名单 (没有配置或者为空，则允许所有访问)
                @WebInitParam(name = "deny", value = "192.168.16.111"),// IP黑名单 (存在共同时，deny优先于allow)
                @WebInitParam(name = "loginUsername", value = "zc"),// 用户名
                @WebInitParam(name = "loginPassword", value = "zc"),// 密码
                @WebInitParam(name = "resetEnable", value = "false")// 禁用HTML页面上的“Reset All”功能
        })
public class DruidStatViewServlet extends StatViewServlet {
}

  ```
    
   + 创建druid 过滤器
   
```
   
   @WebFilter(filterName = "druidWebStatFilter", urlPatterns = "/*",
        initParams = {
                @WebInitParam(name = "exclusions", value = "*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*")// 忽略资源
        })
public class DruidStatFilter extends WebStatFilter {
}

```
    
   
#### 4. 创建数据模型
   创建实体类添加数据注解（@Entity）
   SysUser.java 
   非完整代码
 
 ```

 @Entity
 public class SysUser extends BaseDomain {
 
    private String id;
    ...
    ...
  
 ```
   
   
####  5. 创建服务层(事务，单表更新级查询)及数据访问层
   创建数据库层接口 SysUserRepository.java 继承 JpaRepository<SysUser(当前对象),String(当前对象主键ID)>  
    
  ```
   
    public interface SysUserRepository  extends JpaRepository<SysUser,String> { }
   
  ```
    
   创建服务层类 SysUserService.java (新增修改删除查询，不用写sql)
   
   
   ```
   @Service
public class SysUserService {

    private final Logger logger = LoggerFactory.getLogger(this.getClass());


    @Autowired
    private SysUserRepository sysUserRepository;


    public List<SysUser> findAll() {
        return sysUserRepository.findAll();
    }

    public SysUser update(SysUser sysUser) {

        if (StringUtils.isEmpty(sysUser.getId())) {
            sysUser.setId("ID存在更新,否则新增");
        }

        logger.debug("新增了用户");
        return sysUserRepository.saveAndFlush(sysUser);
    }


    public void delete(String id) {
        logger.debug("删除了用户");
         sysUserRepository.delete(id);
    }

    public SysUser findOne(String id) {
        return sysUserRepository.findOne(id);
    }

}
  
   ```
   
 
   
     
#### 6. 控制层调用 
  创建SysUserController.java 
  
  
  
```

  
//系统用户管理
@RestController
@RequestMapping("/api/sysUser")
public class SysUserController {

    @Autowired
    SysUserService sysUserService;

    @GetMapping(value = "list")
    public List<SysUser> list(){
        return sysUserService.findAll();
    }

    //获取数据
    @GetMapping(value = "update/{id}")
    public SysUser get(@PathVariable("id")String id){
        return sysUserService.findOne(id);
    }

    //新增
    @PostMapping(value = "update")
    public Object add(@RequestBody SysUser c){
        return sysUserService.update(c);
    }

    //更新
    @PutMapping(value = "update")
    public SysUser update(@RequestBody SysUser c){
        return sysUserService.update(c);
    }

    //删除
    @DeleteMapping(value = "update/{id}")
    public void add(@PathVariable("id")String id){
        sysUserService.delete(id);
    }
}

  
```
  
  到目前为止，我们实现了对单表的增删改查所有操作，没有写一句sql，下面我们将添加复杂sql查询，以及最终版本入门框架搭建

### 后续会介绍整体架构，可以应用于项目开发


---
####  项目源码:[github源码地址](https://github.com/zccccccc/cici) 
---
   
  
    
     
   
   
  
  
 
  
  
       
  
   