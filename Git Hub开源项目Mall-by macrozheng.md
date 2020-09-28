# Git Hub开源项目Mall-by **macrozheng**

​	从底层数据库操作出发，分为多个子项目，每个子项目根据需求整合相应技术。主要描述需要整合到项目的技术是如何使用的、在配置时的一些注意事项等。

# 整合SpringBoot+MyBatis

​	本子项目使用到的框架有：SpringBoot、PageHelper、Druid、Mybatis Generator。

## 框架介绍

### SpringBoot

> ​	SpringBoot可以让你快速构建基于Spring的Web应用程序，内置多种Web容器(如Tomcat)，通过启动入口程序的main函数即可运行。
>

### PageHelper

> ​	Mybatis分页插件，帮助开发者只需几行代码即可实现分页查询，在与SpringBoot整合时，主要整合了PageHelper就自动整合了Mybatis。
>

```java
PageHelper.startPage(pageNum, pageSize);
//之后进行查询操作将自动进行分页
List<PmsBrand> brandList = brandMapper.selectByExample(new PmsBrandExample());
//通过构造PageInfo对象获取分页信息，如当前页码，总页数，总条数
PageInfo<PmsBrand> pageInfo = new PageInfo<PmsBrand>(list);
```

### Druid

> ​	Alibaba开源的数据库连接池，号称Java语言中最好的数据库连接池。
>

### Mybatis Generator

> ​	MyBatis的代码生成器，可以根据数据库生成model、mapper.xml、mapper接口和Example（条件），通常情况下的单表查询不用再手写mapper。
>

## Maven依赖

```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <dependencies>
        <!--SpringBoot通用依赖模块-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--MyBatis分页插件-->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>1.2.10</version>
        </dependency>
        <!--集成druid连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>
        <!-- MyBatis 生成器 -->
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.3</version>
        </dependency>
        <!--Mysql数据库驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.15</version>
        </dependency>
    </dependencies>
```

## 使用MGB生成器

​	Mybatis Generator生成model、mapper接口以及mapper.xml 需要数据库连接、数据操作对象（表）、生成器配置文件和生成器。

### 数据库相关配置文件

​	主要配置**数据库连接相关信息**。

```properties
jdbc.driverClass=com.mysql.cj.jdbc.Driver
jdbc.connectionURL=jdbc:mysql://localhost:3306/mall?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
jdbc.userId=root
jdbc.password=root
```

### 生成器配置文件

​	主要配置**数据操作对象（表）（生成对应的model和mapper）、插件以及自定义生成器（生成注解等）**

​	关于配置详解，可参考一下链接：

[^MyBatis Generator 配置详解]: https://blog.csdn.net/testcs_dn/article/details/79295065

```properties
<generatorConfiguration>
	<properties resource="generator.properties" />
	<context id="MySqlContext" targetRuntime="MyBatis3"
		defaultModelType="flat">
		<property name="beginningDelimiter" value="`" />
		<property name="endingDelimiter" value="`" />
		<property name="javaFileEncoding" value="UTF-8" />
		<!-- 为模型生成序列化方法 -->x
		<plugin type="org.mybatis.generator.plugins.SerializablePlugin" />
		<!-- 为生成的Java模型创建一个toString方法 -->
		<plugin type="org.mybatis.generator.plugins.ToStringPlugin" />
		<!--生成mapper.xml时覆盖原文件-->
		<plugin type="org.mybatis.generator.plugins.UnmergeableXmlMappersPlugin"/>
		<!-- 可以自定义生成model的代码注释 -->
		<commentGenerator
			type="com.macro.mall.tiny.mbg.CommentGenerator">
			<!-- 是否去除自动生成的注释 true：是 ： false:否 -->
			<property name="suppressAllComments" value="true" />
			<property name="suppressDate" value="true" />
			<property name="addRemarkComments" value="true" />
		</commentGenerator>
		<!-- 配置数据库连接 -->
		<jdbcConnection driverClass="${jdbc.driverClass}"
			connectionURL="${jdbc.connectionURL}" userId="${jdbc.userId}"
			password="${jdbc.password}">
			<!-- 解决mysql驱动升级到8.0后不生成指定数据库代码的问题 -->
			<property name="nullCatalogMeansCurrent" value="true" />
		</jdbcConnection>
		<!-- 指定生成model的路径 -->
		<javaModelGenerator
			targetPackage="com.macro.mall.tiny.mbg.model"
			targetProject="mall-learning-master\mall-tiny-01\src\main\java" />
		<!-- 指定生成mapper.xml的路径 -->
		<sqlMapGenerator
			targetPackage="com.macro.mall.tiny.mbg.mapper"
			targetProject="mall-learning-master\mall-tiny-01\src\main\resources" />
		<!-- 指定生成mapper接口的的路径 -->
		<javaClientGenerator type="XMLMAPPER"
			targetPackage="com.macro.mall.tiny.mbg.mapper"
			targetProject="mall-learning-master\mall-tiny-01\src\main\java" />
		<!-- 生成全部表tableName设为% -->
		<table tableName="pms_brand">
			<generatedKey column="id" sqlStatement="MySql"
				identity="true" />
		</table>
	</context>
</generatorConfiguration>
```

### 生成器类

​	主要用于**生成MGB代码，是一个main程序。**

```java
public class Generator {
    public static void main(String[] args) throws Exception {
        //MBG 执行过程中的警告信息
        List<String> warnings = new ArrayList<String>();
        //当生成的代码重复时，覆盖原代码
        boolean overwrite = true;
        //读取我们的 MBG 配置文件
        InputStream is = Generator.class.getResourceAsStream("/generatorConfig.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(is);
        is.close();

        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        //创建 MBG
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        //执行生成代码
        myBatisGenerator.generate(null);
        //输出警告信息
        for (String warning : warnings) {
            System.out.println(warning);
        }
    }
}
```

### 注意事项及问题

​	**注意事项：**

​	 使用Eclipse在指定参数 **targetProject** 时，需要使用绝对路径，否则会报错而无法在对应包中生成代码。

​	**问题：**	

​	在多次运行该类的情况下，会覆盖原来的 Java文件，但是**在生成的 xxxMapper.xml文件中，不会覆盖而是以追加的方式添加代码，导致代码重复**。

​	**解决方案：**

​	1.删除原来的xml文件。

​	2.升级Mybatis Generator的版本，在1.3.7版本提供了解决方案。在generatorConfig.xml文件中添加覆盖mapper.xml 的插件：

```xml
<!--生成mapper.xml时覆盖原文件-->
<plugin type="org.mybatis.generator.plugins.UnmergeableXmlMappersPlugin" />
```

## PageHelper

### **使用流程（简单）**

1.导包：与SpringBoot整合，在Maven中导入第三方starter即可使用。

2.方法调用：在需要使用到分页的业务方法中开启分页。

```java
    public List<PmsBrand> listBrand(int pageNum, int pageSize) {
        // 使用PageHelper的静态方法startPage（），传递两个参数（当前页码，每页查询条数）
        PageHelper.startPage(pageNum, pageSize); 
        // 自动的对PageHelper.startPage方法下的第一个sql查询进行分页查询，原理详情看后面。
        return brandMapper.selectByExample(new PmsBrandExample()); 
        // 此时的select的结果是分页的，是根据参数查询的结果集。
    }
```

这是分页的简单实现，也可以根据业务需求对PageHelper提供的一些API进行调用和设参。

### 分页实现原理

​	PageHelper首先将传递个start方法的参数保存到page(Page类)这个对象中，接着将page的副本存放入ThreadLoacl中，这样可以保证分页的时候，参数互不影响。接着利用了mybatis提供的**拦截器**，取得ThreadLocal的值（page副本），**重新拼装**分页SQL，完成分页。

***debug源码待研究***

## 配置文件相关

```yml
server:
  port: 8080
  
spring:
  redis:
    host: localhost # redis服务器地址
    database: 0 # redis数据库索引（默认为0）
    port: 6379 # redis服务器连接端口
    jedis: 
      pool:
        max-active: 8 # 连接池最大连接数（负值表示没有限制） 
        max-wait: -1ms # 连接池最大阻塞等待时间（负值表示没有限制）
        max-idle: 8 # 连接池中最大空闲连接数
        min-idle: 0 # 连接池中最小空闲连接数
    timeout: 3000ms # 连接超时时间（毫秒）
  datasource:
    url: jdbc:mysql://localhost:3306/mall?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
    username: root
    password: root
    
mybatis:
  mapper-locations:
    - classpath:mapper/*.xml
    - classpath*:com/**/mapper/*.xml
```



# 整合Swagger-UI实现在线API文档

## 框架介绍

### Swagger-UI

> ​	**Swagger-UI是HTML, Javascript, CSS的一个集合，可以动态地根据注解生成在线API文档。**
>

​	Swagger不单只有UI，也有一些其他主要项目，如Swagger-tools、core等。

**一些常用注解：**

- @Api：类级别上的注解，用于修饰项目中的Controller类，生成Controller相关文档信息
- @ApiOperation：方法级别上的注解，用于修饰Controller中的方法，生成接口方法相关文档信息
- @ApiParam：参数级别上的注解，用于修饰接口中的参数，生成接口参数相关文档信息
- @ApiModelProperty：字段级别上的注解，用于修饰实体类的属性，当实体类是请求参数或返回结果时，直接生成相关文档信息

PS：这里的接口不是 Interface类，而是API的意思。

## Maven依赖

```xml
<!--Swagger-UI API文档生产工具-->
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
  <version>2.7.0</version>
</dependency>
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui</artifactId>
  <version>2.7.0</version>
</dependency>
```

## Swagger-UI整合

​	使用 POJO 的方式配置，并作为配置类交给SpringBoot管理，所以在config包下添加Swagger-UI的配置类

​	主要配置bean方法：**Docket createRestApi()**，

```java

/**
 * Swagger2API文档的配置
 */
@Configuration
@EnableSwagger2
public class Swagger2Config {
    @Bean
    public Docket createRestApi(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                //为当前包下controller生成API文档
                .apis(RequestHandlerSelectors.basePackage("com.macro.mall.tiny.controller"))
                .paths(PathSelectors.any())
                .build()
                //添加登录认证
                .securitySchemes(securitySchemes())
                .securityContexts(securityContexts());
    }
	
    // 生成API页面基本信息
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("SwaggerUI演示")
                .description("mall-tiny")
                .contact("macro")
                .version("1.0")
                .build();
    }

    private List<ApiKey> securitySchemes() {
        //设置请求头信息
        List<ApiKey> result = new ArrayList<>();
        ApiKey apiKey = new ApiKey("Authorization", "Authorization", "header");
        result.add(apiKey);
        return result;
    }

    private List<SecurityContext> securityContexts() {
        //设置需要登录认证的路径
        List<SecurityContext> result = new ArrayList<>();
        result.add(getContextByPath("/brand/.*"));
        return result;
    }

    private SecurityContext getContextByPath(String pathRegex){
        return SecurityContext.builder()
                .securityReferences(defaultAuth())
                .forPaths(PathSelectors.regex(pathRegex))
                .build();
    }

    private List<SecurityReference> defaultAuth() {
        List<SecurityReference> result = new ArrayList<>();
        AuthorizationScope authorizationScope = new AuthorizationScope("global", "accessEverything");
        AuthorizationScope[] authorizationScopes = new AuthorizationScope[1];
        authorizationScopes[0] = authorizationScope;
        result.add(new SecurityReference("Authorization", authorizationScopes));
        return result;
    }
}
```

注意：Swagger对生成API文档的范围有三种不同的选择

- 生成指定包下面的类的API文档
- 生成有指定注解的类的API文档
- 生成有指定注解的方法的API文档

敲完配置类后，在需要生成API文档的代码中添加注解（只贴出局部）：

```java
@Controller
@RequestMapping("/brand")
@Api(value = "PmsBrandController") // 类级别的注解
@SuppressWarnings("rawtypes")
public class PmsBrandController {

	// ....
    
    @ResponseBody
    @RequestMapping(value = "/list", method = RequestMethod.GET)
    @ApiOperation("分页查询品牌列表") // 方法级别的注解
    public CommonResult<CommonPage<PmsBrand>> listBrand(
            @RequestParam(value = "pageNum", defaultValue = "1") @ApiParam("页码") Integer pageNum,// 参数级别的注解
            @RequestParam(value = "pageSize", defaultValue = "3") @ApiParam("每页数量") Integer pageSize) {
        List<PmsBrand> brandList = brandService.listBrand(pageNum, pageSize);
        return CommonResult.success(CommonPage.restPage(brandList));
    }
}
```

# 整合Redis实现缓存功能

## 框架介绍

### Redis

> Redis是用C语言开发的一个高性能键值对数据库，可用于数据缓存，主要用于处理大量数据的高访问负载。

### Jedis

> Jedis是Redis官方首选的 Java 客户端开发包。

## Maven依赖

```xml
<!--redis依赖配置-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## Redis整合

​	redis是数据层相关的框架，需要在配置文件中声明一些相关参数。

### 配置文件

​	修改springboot配置文件，在application.yml中添加Redis的配置及Redis中自定义key的配置。

​	在spring节点下添加redis配置：

```yml
spring:
 ...
 redis:
    host: localhost # Redis服务器地址
    database: 0 # Redis数据库索引（默认为0）
    port: 6379 # Redis服务器连接端口
    password: # Redis服务器连接密码（默认为空）
    jedis:
      pool:
        max-active: 8 # 连接池最大连接数（使用负值表示没有限制）
        max-wait: -1ms # 连接池最大阻塞等待时间（使用负值表示没有限制）
        max-idle: 8 # 连接池中的最大空闲连接
        min-idle: 0 # 连接池中的最小空闲连接
    timeout: 3000ms # 连接超时时间（毫秒）
```

​	在根节点下添加redis自定义key的配置：

```yml
# 自定义redis key
redis:
  key:
    prefix:
      authCode: "portal:authCode:"
    expire:
      authCode: 120 # 验证码超期时间
```

### 使用redis实现service

​	注入StringRedisTemplate，实现RedisService接口：

```java
package com.macro.mall.tiny.service.impl;

import com.macro.mall.tiny.service.RedisService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

/**
 * redis操作Service的实现类
 * Created by macro on 2018/8/7.
 */
@Service
public class RedisServiceImpl implements RedisService {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

	// 存储数据
    @Override
    public void set(String key, String value) {
        stringRedisTemplate.opsForValue().set(key, value);
    }

    // 获取数据
    @Override
    public String get(String key) {
        return stringRedisTemplate.opsForValue().get(key);
    }

    // 设置超时时间
    @Override
    public boolean expire(String key, long expire) {
        return stringRedisTemplate.expire(key, expire, TimeUnit.SECONDS);
    }

    // 删除数据
    @Override
    public void remove(String key) {
        stringRedisTemplate.delete(key);
    }

    // 自步增长
    @Override
    public Long increment(String key, long delta) {
        return stringRedisTemplate.opsForValue().increment(key,delta);
    }
}

```

### 利用service实现验证码业务功能

​	***在这里提供基本思路，不贴上代码。待日后复习可以自己实现，增强思考能力***

​	要实现验证码功能，首先需要生成验证码，将生成的验证码发送给客户端显示。从客户端获取到键入的值，经过一些错误处理（是否为空），比对验证码与键入的值是否相等，根据该结果返回信息到客户端。

​	由上面的逻辑可以知道我们需要 **生成验证码 **和 **验证验证码** 的service，所以需要新创建一个service去实现这两个方法。上面提到的 **RedisService** 主要是为了在服务器中存储生成的验证码和设置超时时间。

​	前面我们在配置文件中设置的自定义key是为了设置超时时间的长度和验证码头部。超时时间容易理解，验证码头部的功能猜测是为了防止与redis其他key有冲突，所以添加头部表明该键值对是为了验证码功能而使用的。

# 整合SpringSecurity和 JWT 实现认证和授权

​	本文主要讲解mall通过整合SpringSecurity和 JWT实现后台用户的登录和授权功能，同时改造Swagger-UI的配置使其可以自动记住登录令牌进行发送。

## 框架介绍	

### Spring Security

> SpringSecurity是一个强大的可高度定制的认证和授权框架，对于Spring应用来说它是一套Web安全标准。SpringSecurity注重于为Java应用提供认证和授权功能，像所有的Spring项目一样，它对自定义需求具有强大的扩展性。

### JWT

> JWT是JSON WEB TOKEN的缩写，它是基于 RFC 7519 标准定义的一种可以安全传输的的JSON对象，由于使用了数字签名，所以是可信任和安全的。

**JWT token组成主要由三部分：头部、负载和签名**

- **头部**：用于存放签名的生成算法
- **负载**：用于存放用户名、token生成时间和过期时间、还可以存放一些其他数据
- **签名**：以header和payload生成的签名，一旦header和payload被篡改，验证将失败

**JWT实现认证和授权的原理：**

- 用户调用登录接口，登录成功后获取到 JWT 的token；
- 之后用户每次调用接口都在http的header中添加一个叫Authorization的头，值为JWT的token；
- 后台程序通过对Authorization头中信息的解码及数字签名校验来获取其中的用户信息，从而实现认证和授权。

### Hutool

> Hutool是一个丰富的Java开源工具包,它帮助我们简化每一行代码，减少每一个方法，mall项目采用了此工具包。

## Maven依赖

```xml
<!--SpringSecurity依赖配置-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<!--Hutool Java工具包-->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>4.5.7</version>
</dependency>
<!--JWT(Json Web Token)登录支持-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.0</version>
</dependency>
```

## 使用 JWT 实现 token 相关功能的工具类

​	token相关功能有：

- **用户登录时生成token** ：generateToken
- **从客户端传到服务端的token中获取登录用户信息**：getUsernameFromToken
- **验证传入的token是否正确且是否过期**：validateToken

创建组件类 JwtTokenUtil ，实现以上功能：

```java
@Component
public class JwtTokenUtil {
    private static final Logger LOGGER = LoggerFactory.getLogger(JwtTokenUtil.class);
    private static final String CLAIM_KEY_USERNAME = "sub"; // 自定义负载中的信息
    private static final String CLAIM_KEY_CREATED = "created"; // 自行负载中的信息
    @Value("${jwt.secret}") // 从application.yml 中获取密钥
    private String secret;
    @Value("${jwt.expiration}") // 从application.yml 中获取过期时间
    private Long expiration;

    /**
     * 根据负责生成JWT的token
     */
    private String generateToken(Map<String, Object> claims) {
        return Jwts.builder()
                .setClaims(claims)
                .setExpiration(generateExpirationDate())
                .signWith(SignatureAlgorithm.HS512, secret)
                .compact();
    }

    /**
     * 从token中获取JWT中的负载
     */
    private Claims getClaimsFromToken(String token) {
        Claims claims = null;
        try {
            claims = Jwts.parser()
                    .setSigningKey(secret)
                    .parseClaimsJws(token)
                    .getBody();
        } catch (Exception e) {
            LOGGER.info("JWT格式验证失败:{}",token);
        }
        return claims;
    }

    /**
     * 生成token的过期时间
     */
    private Date generateExpirationDate() {
        return new Date(System.currentTimeMillis() + expiration * 1000);
    }

    /**
     * 从token中获取登录用户名
     */
    public String getUserNameFromToken(String token) {
        String username;
        try {
            Claims claims = getClaimsFromToken(token);
            username =  claims.getSubject();
        } catch (Exception e) {
            username = null;
        }
        return username;
    }

    /**
     * 验证token是否还有效
     *
     * @param token       客户端传入的token
     * @param userDetails 从数据库中查询出来的用户信息
     */
    public boolean validateToken(String token, UserDetails userDetails) {
        String username = getUserNameFromToken(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }

    /**
     * 判断token是否已经失效
     */
    private boolean isTokenExpired(String token) {
        Date expiredDate = getExpiredDateFromToken(token);
        return expiredDate.before(new Date());
    }

    /**
     * 从token中获取过期时间
     */
    private Date getExpiredDateFromToken(String token) {
        Claims claims = getClaimsFromToken(token);
        return claims.getExpiration();
    }

    /**
     * 根据用户信息生成token
     */
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put(CLAIM_KEY_USERNAME, userDetails.getUsername());
        claims.put(CLAIM_KEY_CREATED, new Date());
        return generateToken(claims);
    }

    /**
     * 判断token是否可以被刷新
     */
    public boolean canRefresh(String token) {
        return !isTokenExpired(token);
    }

    /**
     * 刷新token
     */
    public String refreshToken(String token) {
        Claims claims = getClaimsFromToken(token);
        claims.put(CLAIM_KEY_CREATED, new Date());
        return generateToken(claims);
    }
}
```

## 配置Spring Security

Spring Security的核心功能主要包括：

- 认证
- 授权
- 攻击防护

其核心就是一组过滤器链，项目启动后将会自动配置。其中最核心的就是 Basic Authentication Filter 用来认证用户的身份，一个在spring security中一种过滤器处理一种认证方式。

​	使用**@EnableWebSecurity注解** 启动Spring Security，配置类继承 **WebSecurityConfigurerAdapter** 重写其中的 **configure** 方法定义哪些URL路径应该被保护，哪些路径可以匿名访问。

​	在该项目中，我们使用Spring Security 和 JWT 共同实现认证和授权功能，因此需要关闭session功能。允许对于网站中静态资源以及swagger-ui的资源进行访问，允许对登录和注册进行访问。

​	自定义错误处理机制，如当用户访问接口没有权限时的错误信息（这里的权限是指对用户对该项目功能模块的访问权限），用户未登录或token过期时错误信息。 **RestfulAccessDeniedHandler** 和**RestAuthenticationEntryPoint**  都是使用封装好的通用返回类中的方法进行错误提示，注意的是需要返回对象转换成 JSON 格式。

​	自定义 JwtToken认证过滤器，见如下的 **JwtAuthenticationTokenFilter**。

​	SecurityConfig 类如下：

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private UmsAdminService adminService; 
    @Autowired
    private RestfulAccessDeniedHandler restfulAccessDeniedHandler; 
    @Autowired
    private RestAuthenticationEntryPoint restAuthenticationEntryPoint; 

    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.csrf()// 由于使用的是JWT，我们这里不需要csrf
                .disable().sessionManagement()// 基于token，所以不需要session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS).and().authorizeRequests()
                .antMatchers(HttpMethod.GET, // 允许对于网站静态资源的无授权访问
                        "/", "/*.html", "/favicon.ico", "/**/*.html", "/**/*.css", "/**/*.js", "/swagger-resources/**",
                        "/v2/api-docs/**")
                .permitAll().antMatchers("/admin/login", "/admin/register")// 对登录注册要允许匿名访问
                .permitAll().antMatchers(HttpMethod.OPTIONS)// 跨域请求会先进行一次options请求
                .permitAll()
//                .antMatchers("/**")//测试时全部运行访问
//                .permitAll()
                .anyRequest()// 除上面外的所有请求全部需要鉴权认证
                .authenticated();
        // 禁用缓存
        httpSecurity.headers().cacheControl();
        // 添加JWT filter
        httpSecurity.addFilterBefore(jwtAuthenticationTokenFilter(), UsernamePasswordAuthenticationFilter.class);
        // 添加自定义未授权和未登录结果返回
     httpSecurity.exceptionHandling().accessDeniedHandler(restfulAccessDeniedHandler)
                .authenticationEntryPoint(restAuthenticationEntryPoint);
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService()).passwordEncoder(passwordEncoder());
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /**
     * 加载用户特定数据的核心接口。
     * 它作为用户DAO在整个框架中使用，也是DaoAuthenticationProvider使用的策略。
     * 该接口只有一个通过用户名获取UserDetails的方法，这简化了对新数据访问策略的支持。
     */
    @Bean
    public UserDetailsService userDetailsService() {
        // 获取登录用户信息
        // 使用lambda表达式实现接口中的方法，返回一个实现了UserDetails接口的AdminUserDetails实例（自定义）
        return username -> {
            UmsAdmin admin = adminService.getAdminByUsername(username);
            if (admin != null) {
                List<UmsPermission> permissionList = adminService.getPermissionList(admin.getId());
                return new AdminUserDetails(admin, permissionList);
            }
            throw new UsernameNotFoundException("用户名或密码错误");
        };
    }

    @Bean
    public JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter() {
        return new JwtAuthenticationTokenFilter();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

}

```

**JwtAuthenticationTokenFilter** 的实现：获取请求中的请求头中key为 **Authorization** 的value（值），去除tokenHead后得到从客户端请求的 token，根据token获取用户信息，然后根据该用户信息进行认证判断。

```java
/**
 * JWT登录授权过滤器 Created by macro on 2018/4/26.
 */
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {
    private static final Logger LOGGER = LoggerFactory.getLogger(JwtAuthenticationTokenFilter.class);
    @Autowired
    private UserDetailsService userDetailsService;
    @Autowired
    private JwtTokenUtil jwtTokenUtil;
    @Value("${jwt.tokenHeader}") // 请求头中带有token值的key
    private String tokenHeader;
    @Value("${jwt.tokenHead}")
    private String tokenHead; // token头

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        String authHeader = request.getHeader(this.tokenHeader);
        if (authHeader != null && authHeader.startsWith(this.tokenHead)) {
            // The part after "Bearer "
            String authToken = authHeader.substring(this.tokenHead.length());
            String username = jwtTokenUtil.getUserNameFromToken(authToken);
            LOGGER.info("checking username:{}", username);
            if (username != null && 
    SecurityContextHolder.getContext().getAuthentication() == null) {
                UserDetails userDetails = 
                    this.userDetailsService.loadUserByUsername(username);
                if (jwtTokenUtil.validateToken(authToken, userDetails)) {
                    UsernamePasswordAuthenticationToken authentication = 
                        new UsernamePasswordAuthenticationToken(
                            userDetails, null, userDetails.getAuthorities());
                    authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                    LOGGER.info("authenticated user:{}", username);
                    SecurityContextHolder.getContext().setAuthentication(authentication);
                }
            }
        }
        chain.doFilter(request, response);
    }
}
```

关于service和controller的实现就是方法之间的调用问题和错误处理了。

## 配置文件相关

主要是关于token的相关数据，放在配置文件中是为了修改方便

```yml
jwt:
  tokenHeader: Authorization #JWT存储的请求头
  secret: mySecret #JWT加密、解密使用的密钥
  expiration: 604800 #JWT的超期时间（60*60*24）
  tokenHead: Bearer #JWT负载中拿到开头
```

# 整合Spring Task实现定时任务

暂

# 整合Elasticsearch实现商品搜索

> 本文主要讲解mall整合Elasticsearch的过程，以实现商品信息在Elasticsearch中的导入、查询、修改、删除为例。

## 框架介绍

### Elasticsearch

> Elasticsearch 是一个分布式、可扩展、实时的搜索与数据分析引擎。它能从项目一开始就赋予你的数据以搜索、分析和探索的能力，可用于实现全文搜索和实时数据统计。

#### 基本概念

- index：相当于关系型数据库的库，名称必须小写。
- type：相当于关系型数据库的表。
- document：相当于关系型数据库的行。
- field：相当于关系型数据库的列（字段）。

### Spring Data Elasticsearch

> Sptring data elasticsearh 是spring 提供的一种以Spring Data风格来操作数据存储的方式，它可以避免编写大量的样板代码。

常用注解

- @Document：可以指定index和type
- @Id：文档的Id，主键的概念
- @Field：字段，可以指定是否建立**倒排索引**（重点）等。
- @Query：可以自定义用Elasticsearch的DSL语句进行查询。

## Maven依赖

```xml
<!--Elasticsearch相关依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch<artifactId>
</dependency>
```

## 配置文件先关

​	修改Spring boot配置文件，在spring节点下添加data节点（spring data），再在data节点下添加Elasticsearch相关配置。

```yaml
spring:
 ...
 data:
  elasticsearch:
    repositories:
      enabled: true
    cluster-nodes: 127.0.0.1:9300 # es的连接地址及端口号
    cluster-name: elasticsearch # es集群的名称
```

## 实现商品搜索

### 实体类

**EsProduct**：商品文档对象

```java
package com.macro.mall.tiny.nosql.elasticsearch.document;

import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

import java.io.Serializable;
import java.math.BigDecimal;
import java.util.List;

/**
 * 搜索中的商品信息
 * Created by macro on 2018/6/19.
 */
@Document(indexName = "pms", type = "product",shards = 1,replicas = 0)
public class EsProduct implements Serializable {
    private static final long serialVersionUID = -1L;
    @Id
    private Long id;
    @Field(type = FieldType.Keyword)
    private String productSn;
    private Long brandId;
    @Field(type = FieldType.Keyword)
    private String brandName;
    private Long productCategoryId;
    @Field(type = FieldType.Keyword)
    private String productCategoryName;
    private String pic;
    @Field(analyzer = "ik_max_word",type = FieldType.Text)
    private String name;
    @Field(analyzer = "ik_max_word",type = FieldType.Text)
    private String subTitle;
    @Field(analyzer = "ik_max_word",type = FieldType.Text)
    private String keywords;
    private BigDecimal price;
    private Integer sale;
    private Integer newStatus;
    private Integer recommandStatus;
    private Integer stock;
    private Integer promotionType;
    private Integer sort;
    @Field(type =FieldType.Nested)
    private List<EsProductAttributeValue> attrValueList;

    //省略了所有getter和setter方法
}
```

### Dao

​	使用MyBatis从数据库中获取商品信息（联合查询）

```java
package com.macro.mall.tiny.dao;

import com.macro.mall.tiny.nosql.elasticsearch.document.EsProduct;
import org.apache.ibatis.annotations.Param;

import java.util.List;

/**
 * 搜索系统中的商品管理自定义Dao
 * Created by macro on 2018/6/19.
 */
public interface EsProductDao {
    List<EsProduct> getAllEsProductList(@Param("id") Long id);
}
```

注意：需要配置与dao相应的xml文件，如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.macro.mall.tiny.dao.EsProductDao">
    <resultMap id="esProductListMap" type="com.macro.mall.tiny.nosql.elasticsearch.document.EsProduct" autoMapping="true">
        <id column="id" jdbcType="BIGINT" property="id" />
        <collection property="attrValueList" columnPrefix="attr_" ofType="com.macro.mall.tiny.nosql.elasticsearch.document.EsProductAttributeValue">
            <id column="id" property="id" jdbcType="BIGINT"/>
            <result column="product_attribute_id" property="productAttributeId" jdbcType="BIGINT"/>
            <result column="value" property="value" jdbcType="VARCHAR"/>
            <result column="type" property="type"/>
            <result column="name" property="name"/>
        </collection>
    </resultMap>
    <select id="getAllEsProductList" resultMap="esProductListMap">
        select
            p.id id,
            p.product_sn productSn,
            p.brand_id brandId,
            p.brand_name brandName,
            p.product_category_id productCategoryId,
            p.product_category_name productCategoryName,
            p.pic pic,
            p.name name,
            p.sub_title subTitle,
            p.price price,
            p.sale sale,
            p.new_status newStatus,
            p.recommand_status recommandStatus,
            p.stock stock,
            p.promotion_type promotionType,
            p.keywords keywords,
            p.sort sort,
            pav.id attr_id,
            pav.value attr_value,
            pav.product_attribute_id attr_product_attribute_id,
            pa.type attr_type,
            pa.name attr_name
        from pms_product p
        left join pms_product_attribute_value pav on p.id = pav.product_id
        left join pms_product_attribute pa on pav.product_attribute_id= pa.id
        where delete_status = 0 and publish_status = 1
        <if test="id!=null">
            and p.id=#{id}
        </if>
    </select>
</mapper>
```

联合pms_product、pms_product_attribute_value 和 pms_product_attribute 三个表的信息生成 EsProduct。

### Spring Data Repository

​	继承ElasticsearchRepository接口，这样就拥有了一些基本的Elasticsearch数据操作方法，同时定义了一个衍生查询方法。

```java
package com.macro.mall.tiny.nosql.elasticsearch.repository;

import com.macro.mall.tiny.nosql.elasticsearch.document.EsProduct;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;

/**
 * 商品ES操作类
 * Created by macro on 2018/6/19.
 */
public interface EsProductRepository extends ElasticsearchRepository<EsProduct, Long> {
    /**
     * 搜索查询（自定义方法，spring data会根据方法名和返回值自动实现）
     *
     * @param name              商品名称
     * @param subTitle          商品标题
     * @param keywords          商品关键字
     * @param page              分页信息
     * @return
     */
    Page<EsProduct> findByNameOrSubTitleOrKeywords(String name, String subTitle, String keywords, Pageable page);

}
```

### Service

省略 service接口

```java
package com.macro.mall.tiny.service.impl;

/**
 * 商品搜索管理Service实现类
 * Created by macro on 2018/6/19.
 */
@Service
public class EsProductServiceImpl implements EsProductService {
    private static final Logger LOGGER =
        LoggerFactory.getLogger(EsProductServiceImpl.class);
    @Autowired
    private EsProductDao productDao;
    @Autowired
    private EsProductRepository productRepository;
    
    /**
     * 从数据库中导入所有商品到ES
     */
    @Override
    public int importAll() {
        List<EsProduct> esProductList = productDao.getAllEsProductList(null);
        Iterable<EsProduct> esProductIterable = productRepository.saveAll(esProductList);
        Iterator<EsProduct> iterator = esProductIterable.iterator();
        int result = 0;
        while (iterator.hasNext()) {
            result++;
            iterator.next();
        }
        return result;
    }

    /**
     * 根据id删除商品
     */
    @Override
    public void delete(Long id) {
        productRepository.deleteById(id);
    }

    /**
     * 根据id创建商品
     */
    @Override
    public EsProduct create(Long id) {
        EsProduct result = null;
        List<EsProduct> esProductList = productDao.getAllEsProductList(id);
        if (esProductList.size() > 0) {
            EsProduct esProduct = esProductList.get(0);
            result = productRepository.save(esProduct);
        }
        return result;
    }

    /**
     * 根据id批量删除商品
     */
    @Override
    public void delete(List<Long> ids) {
        if (!CollectionUtils.isEmpty(ids)) {
            List<EsProduct> esProductList = new ArrayList<>();
            for (Long id : ids) {
                EsProduct esProduct = new EsProduct();
                esProduct.setId(id);
                esProductList.add(esProduct);
            }
            productRepository.deleteAll(esProductList);
        }
    }

    /**
     * 根据关键字搜索名称或者副标题
     */
    @Override
    public Page<EsProduct> search(String keyword, Integer pageNum, Integer pageSize) {
        Pageable pageable = PageRequest.of(pageNum, pageSize);
        return productRepository.findByNameOrSubTitleOrKeywords(keyword, keyword, keyword, pageable);
    }
}
```

### Controller

不贴出代码，可以根据service层编写出的简单逻辑。

# 参考资料

[mall学习教程]: https://github.com/macrozheng/mall-learning

