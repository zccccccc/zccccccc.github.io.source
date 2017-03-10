---
title: spring-boot 完成版本－不解释
date: 2017-03-09 17:47:31
tags: java
layout: clean-blog
slug: spring-boot-first2

---
## 本文为Spring-boot 完整版本，可用于基础开发
  
  
  
###### 上一文中我们介绍了如何搭建sprign-boot 服务，下面我们一起来搞出完整版本（以下是包含使用到的技术）
   
   
  1. Swagger2 是一个规范和完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务。总体目标是使客户端和文件系统作为服务器以同样的速度来更新。文件的方法，参数和模型紧密集成到服务器端的代码，允许API来始终保持同步。 
  2. Spring jap 已经实现了很多数据库操作相关功能，但是拼写SQL是个硬伤，当前项目中实现了一种方式，将 resources/dbsql下的 XXX.xml 映射到全局map中，下面会有详细介绍
 
   
---
####  项目源码:[github源码地址](https://github.com/zccccccc/cici) 
---

  


  
  ---
#### Swagger2介绍 开始
   
   
 ***pom.xml 添加***
 
 ```
 
     <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.2.2</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.2.2</version>
        </dependency>
 
 ```
   
***在MainApplication平级创建 Swagger2.java***

```

@Configuration
@EnableSwagger2
public class Swagger2 {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.springboot.demo"))
                .paths(PathSelectors.any())
                .build();
    }


    private ApiInfo apiInfo() {


                return new ApiInfoBuilder()
                .title("Spring Boot 中使用Swagger2构建RESTful APIs").version("2.0").build();

    }

}


```

*** API接口方法中加入下面注解，就可以了使用了，访问地址 http://localhost:7777 /swagger-ui.html 就可以访问了***  
 
 ```
    @ApiOperation(value = "获取用户列表", notes = "需要传入json参数")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "sysUserDomain", value = "系统用户实体", required = true, dataType = "SysUserDomain")
    })
    @ApiResponses(value = {
            @ApiResponse(code = 200, message = "Success", response = BaseOutput.class),
            @ApiResponse(code = 401, message = "Unauthorized"),
            @ApiResponse(code = 403, message = "Forbidden"),
            @ApiResponse(code = 404, message = "Not Found"),
            @ApiResponse(code = 500, message = "Failure")})
    @PostMapping(value = "list")
    public BaseOutput<SysUser> list(HttpServletRequest request, HttpServletResponse response, @RequestBody SysUserDomain sysUserDomain) {
        BaseOutput<SysUser> baseOutput = null;
        try {

            baseOutput = sysUserService.findList(sysUserDomain);

        } catch (Exception ex) {

            baseOutput = baseOutput == null ? (baseOutput = new BaseOutput<SysUser>()) : baseOutput;
            baseOutput.setReturnCode(new ReturnCode(ReturnCodeDict.ERROR.k, ReturnCodeDict.ERROR.m, ex.getMessage()));


        } finally {
            return baseOutput;
        }

    }
 
 ```
  
---  
####  将sql放到xml中，系统启动时候加载到各个模块中
*** 在resources 下创建dbsql文件夹 里面包含各种sql的xml: namespace(具体使用实现类，获取sql时使用),version(版本控制后期会加入自动热部署) ***

```

<?xml version="1.0" encoding="UTF-8"?>
<db-sql>
<mapper namespace="com.springboot.demo.usercenter.usermanager.serve.SysUserServiceImpl" version="1.0" >

    <sql id="findListNativeSql">
        SELECT
        distinct
        XX.module_id,
        XX.action_code,
        XX.action_name,
        ZZ.name module_name,
        ZZ.url,
        ZZ.order_id,
        ZZ.module_desc,
        ZZ.code module_code,
        ZZ.parent_id
        FROM SYS_PRIVILEGE XX,
        SYS_MODULE ZZ
        WHERE XX.module_id =ZZ.id
        AND ZZ.enable_state='10'
        and exists
        ( SELECT 1
        FROM SYS_USER_AUTH AA,SYS_ROLE BB
        WHERE AA.user_id =  ?1
        AND AA.role_id = BB.id
        AND BB.enable_state='10'
        and AA.role_id = XX.role_id
        AND EXISTS
        ( SELECT 1
        FROM sys_system SS
        WHERE SS.id = BB.sys_id
        AND SS.code =  ?2 )
        AND AA.role_id=BB.id)
    </sql>

</mapper>
</db-sql>

```
  
  
***读取XML并映射到HashMap中 创建LoadingSqlXml.java***

 
*** 完整版本 ***

```
public enum LoadingSqlXml {

    INSTANCE;

    //获取dbsql绝对路径
    private final static String dbSqlPath = ClassUtils.getDefaultClassLoader().getResource("dbsql").getPath();
    //定义私有SQLMap  XML中的namespace,version 该文件下的sql集合
    private static final Map<String, HashMap<Double, HashMap<String, String>>> myHashMaps = new HashMap<String, HashMap<Double, HashMap<String, String>>>();

    //定义私有SQLMap集合
    private static HashMap sqlMap = null;

    //定义版本集合
    private static final HashMap<Double, HashMap<String, String>> versionMap = new HashMap();


    //根据名称找到当前实现类下需要使用的SQL集合
    //如:   private HashMap<String, String> dbSqlMap = LoadingSqlXml.INSTANCE.findSqlMap(this.getClass().getName());
    //  dbSqlMap.get("sqlId");
    public HashMap<String, String> findSqlMap(String c) {

        if (myHashMaps.isEmpty()) {
            try {
                init();
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }

        HashMap<String, String> _h = null;
        for (Map.Entry<Double, HashMap<String, String>> entry0 : myHashMaps.get(c).entrySet()) {
            _h = entry0.getValue();
        }
        return _h;
    }

    //初始化执行
    public void init() throws InitDataSqlReaderException, DocumentException {
        if (!myHashMaps.isEmpty()) {
            return;
        }
        File file = new File(dbSqlPath);
        File[] files = file.listFiles();
        for (File f : files) {
            ReadDbSqlXml(f);
        }
    }

    //读取文件XML并解析到 数据集合中
    private void ReadDbSqlXml(File f) throws InitDataSqlReaderException, DocumentException {

        SAXReader reader = new SAXReader();
        Document doc = reader.read(f);
        Element root = doc.getRootElement();
        Element fo;
        Element value;
        String fileName = f.getName();

        //迭代器查看有几个
        for (Iterator i = root.elementIterator("mapper"); i.hasNext(); ) {

            fo = (Element) i.next();

            List<String> attributes = fo.attributes();
            if (attributes.size() == 0) {
                throw new InitDataSqlReaderException(fileName.concat(" 在mapper节点找不到属性,请定义 [namespace,version] "));
            }

            String namespace = fo.attribute("namespace").getValue();
            String version = fo.attribute("version").getValue();

            if (StringUtils.isEmpty(namespace)) {
                throw new InitDataSqlReaderException(fileName.concat(" 在mapper节点找不到属性,请定义 namespace "));
            }
            if (StringUtils.isEmpty(version)) {
                throw new InitDataSqlReaderException(fileName.concat(" 在mapper节点找不到属性,请定义 version "));
            }

            if (myHashMaps.get(namespace) != null) {
                throw new InitDataSqlReaderException(fileName.concat(" 在mapper节点 中的 namespace 定义重复 ==>" + namespace));
            }

            for (Iterator k = fo.elementIterator("sql"); k.hasNext(); ) {
                sqlMap = new HashMap();
                value = (Element) k.next();
                String id = value.attribute("id").getValue();

                if (id.equals("")) {
                    throw new InitDataSqlReaderException(fileName.concat("在mapper下sql未定义id"));
                }

                String sql = String.valueOf(value.getData());
                if (sqlMap.get(id) == null) {
                    sqlMap.put(id, sql);
                } else {
                    throw new InitDataSqlReaderException(fileName.concat("sql 节点 ID 定义重复 当前ID=" + id));
                }

            }
            versionMap.put(Double.parseDouble(version), sqlMap);
            myHashMaps.put(namespace, versionMap);
        }
    }
    }
    
 ```
 
 
 	 *具体实现代码就不在这里展示了直接看源码吧，比较简单，目录结构 下载到源码后从com.springboot.demo.usercenter.usermanager.web.SysUserController 开始向上阅读*
 	   
 	   
     
  *目前只是简单实现，后续会介绍完整版本 基于spring cloud 微服务架构整合*
    
    
---
####  项目源码:[github源码地址](https://github.com/zccccccc/cici) 
---

    
    
 
   







  
  