要在Spring Boot项目中使用Redis操作，需要进行以下准备工作：

## 1. 添加依赖

在 [pom.xml](file://F:\BaiduNetdiskDownload\黑马点评\hm-dianping\pom.xml) 中添加Redis相关依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- 如果需要连接池功能 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```


## 2. 配置Redis连接

在 `application.yml` 或 `application.properties` 中配置Redis连接信息：

```yaml
spring:
  redis:
    host: 192.168.100.128
    port: 6379
    password: 1234
    database: 0 # 使用的数据库索引
    timeout: 2000ms # 连接超时时间
    lettuce:
      pool:
        max-active: 8 # 连接池最大连接数
        max-idle: 8 # 连接池最大空闲连接
        min-idle: 0 # 连接池最小空闲连接
        max-wait: -1ms # 连接池最大阻塞等待时间
```

## 3. 配置RedisTemplate

如果需要自定义序列化方式，可以配置 [RedisTemplate](file://F:\BaiduNetdiskDownload\黑马点评\hm-dianping\src\main\java\com\hmdp\config\WebMvcConfig.java#L21-L21)：

```java
@Configuration
public class RedisConfig {
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        
        // 设置key的序列化方式
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        
        // 设置value的序列化方式
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        template.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
        
        template.afterPropertiesSet();
        return template;
    }
}
```


## 4. 注入StringRedisTemplate或RedisTemplate

在需要使用Redis的类中注入相应的模板：

```java
@Service
public class RedisService {
    
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    
    // 或者使用自定义的RedisTemplate
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // 使用示例
    public void setValue(String key, String value) {
        stringRedisTemplate.opsForValue().set(key, value);
    }
    
    public String getValue(String key) {
        return stringRedisTemplate.opsForValue().get(key);
    }
}
```


## 5. 常用操作示例

配置完成后可以使用各种Redis操作：

```java
// 字符串操作
stringRedisTemplate.opsForValue().set("key", "value");
String value = stringRedisTemplate.opsForValue().get("key");

// Hash操作
stringRedisTemplate.opsForHash().put("hashKey", "field", "value");
Map<Object, Object> map = stringRedisTemplate.opsForHash().entries("hashKey");

// List操作
stringRedisTemplate.opsForList().leftPush("listKey", "value");
List<String> list = stringRedisTemplate.opsForList().range("listKey", 0, -1);

// 设置过期时间
stringRedisTemplate.expire("key", 30, TimeUnit.MINUTES);
```


## 6. 启动Redis服务

确保本地或远程Redis服务已经启动并可以正常连接。

完成以上配置后，就可以在项目中正常使用Redis进行数据操作了。从你的代码上下文可以看出，项目中已经配置并使用了 [StringRedisTemplate](file://F:\BaiduNetdiskDownload\黑马点评\hm-dianping\src\main\java\com\hmdp\config\WebMvcConfig.java#L21-L21)。