要使用 MyBatis-Plus 查询数据库，需要进行以下准备工作：

## 1. 添加依赖

在 [pom.xml](file://F:\BaiduNetdiskDownload\黑马点评\hm-dianping\pom.xml) 中添加 MyBatis-Plus 依赖：

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.3.1</version>
</dependency>
```


## 2. 配置数据库连接

在 `application.yml` 或 `application.properties` 中配置数据库连接信息：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/your_database?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
    username: your_username
    password: your_password
    driver-class-name: com.mysql.cj.jdbc.Driver
```


## 3. 创建实体类

创建与数据库表对应的实体类，使用 MyBatis-Plus 注解：

```java
@Data
@TableName("user") // 对应数据库表名
public class User {
    @TableId(type = IdType.AUTO) // 主键策略
    private Long id;
    
    private String name;
    private Integer age;
    private String email;
}
```


## 4. 创建 Mapper 接口

创建继承自 `BaseMapper` 的 Mapper 接口：

```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
    // 继承 BaseMapper 后，可以直接使用常见的 CRUD 操作
    // 也可以在这里定义自定义查询方法
}
```


## 5. 启用 Mapper 扫描

可以通过以下方式之一启用 Mapper 扫描：

### 方式一：在启动类上添加注解
```java
@SpringBootApplication
@MapperScan("com.yourpackage.mapper")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```


### 方式二：在每个 Mapper 接口上添加 @Mapper 注解
```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
    // ...
}
```


## 6. 在 Service 中使用

在 Service 类中注入 Mapper 并使用：

```java
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;
    
    public List<User> getAllUsers() {
        return userMapper.selectList(null); // 查询所有用户
    }
    
    public User getUserById(Long id) {
        return userMapper.selectById(id);
    }
}
```

## 7. 常用配置

可以在 `application.yml` 中添加 MyBatis-Plus 相关配置：

```yaml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl # 打印SQL日志
  global-config:
    db-config:
      id-type: auto # 全局ID策略
```


完成以上配置后，就可以使用 MyBatis-Plus 提供的各种便捷的数据库操作功能了。