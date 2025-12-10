要在Spring Boot项目中使用AOP（面向切面编程），需要进行以下准备工作：

## 1. 添加AOP依赖

在 [pom.xml](file://F:\BaiduNetdiskDownload\黑马点评\hm-dianping\pom.xml) 中添加AOP相关依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```


这个依赖会自动引入AspectJ的相关库。

## 2. 启用AOP支持

在Spring Boot中，如果添加了 `spring-boot-starter-aop` 依赖，默认会自动启用AOP支持。但也可以显式地在配置类或启动类上添加注解：

```java
@SpringBootApplication
@EnableAspectJAutoProxy  // 显式启用AspectJ自动代理
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```


## 3. 创建切面类

创建一个切面类来定义横切关注点：

```java
@Aspect
@Component
public class LoggingAspect {
    
    private static final Logger logger = LoggerFactory.getLogger(LoggingAspect.class);
    
    // 定义切点
    @Pointcut("execution(* com.yourpackage.service.*.*(..))")
    public void serviceLayerExecution() {}
    
    // 前置通知
    @Before("serviceLayerExecution()")
    public void beforeServiceMethod(JoinPoint joinPoint) {
        logger.info("即将执行方法: " + joinPoint.getSignature().getName());
    }
    
    // 后置通知
    @After("serviceLayerExecution()")
    public void afterServiceMethod(JoinPoint joinPoint) {
        logger.info("方法执行完成: " + joinPoint.getSignature().getName());
    }
    
    // 返回通知
    @AfterReturning(pointcut = "serviceLayerExecution()", returning = "result")
    public void afterReturningServiceMethod(JoinPoint joinPoint, Object result) {
        logger.info("方法返回值: " + result);
    }
    
    // 异常通知
    @AfterThrowing(pointcut = "serviceLayerExecution()", throwing = "exception")
    public void afterThrowingServiceMethod(JoinPoint joinPoint, Exception exception) {
        logger.error("方法抛出异常: " + exception.getMessage());
    }
    
    // 环绕通知
    @Around("serviceLayerExecution()")
    public Object aroundServiceMethod(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();
        try {
            Object result = joinPoint.proceed();
            long endTime = System.currentTimeMillis();
            logger.info("方法执行时间: " + (endTime - startTime) + "ms");
            return result;
        } catch (Throwable throwable) {
            logger.error("方法执行出错", throwable);
            throw throwable;
        }
    }
}
```


## 4. 常用AOP注解说明

### 切面相关注解：
- `@Aspect`: 标识一个类为切面类
- `@Pointcut`: 定义切点表达式
- `@Before`: 前置通知，在目标方法执行前执行
- `@After`: 后置通知，在目标方法执行后执行（无论是否抛出异常）
- `@AfterReturning`: 返回通知，在目标方法成功执行后执行
- `@AfterThrowing`: 异常通知，在目标方法抛出异常后执行
- `@Around`: 环绕通知，可以控制目标方法的执行

### 常用切点表达式：
```java
// 执行任何public方法
@Pointcut("execution(public * *(..))")

// 执行service包及其子包下的任何方法
@Pointcut("execution(* com.yourpackage.service..*.*(..))")

// 执行带有特定注解的方法
@Pointcut("@annotation(com.yourpackage.annotation.MyAnnotation)")

// 执行特定类的所有方法
@Pointcut("within(com.yourpackage.service.UserService)")

// 执行实现了特定接口的类的方法
@Pointcut("this(com.yourpackage.service.UserService)")
```


## 5. 自定义注解配合AOP使用

创建自定义注解：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime {
}
```


在切面中使用自定义注解：

```java
@Aspect
@Component
public class ExecutionTimeAspect {
    
    private static final Logger logger = LoggerFactory.getLogger(ExecutionTimeAspect.class);
    
    @Around("@annotation(LogExecutionTime)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();
        Object result = joinPoint.proceed();
        long endTime = System.currentTimeMillis();
        logger.info("{} 执行时间: {} ms", joinPoint.getSignature(), endTime - startTime);
        return result;
    }
}
```


在业务方法上使用注解：

```java
@Service
public class UserService {
    
    @LogExecutionTime
    public User getUserById(Long id) {
        // 业务逻辑
        return user;
    }
}
```


## 6. 配置AOP代理模式（可选）

在 `application.yml` 中可以配置AOP代理模式：

```yaml
spring:
  aop:
    proxy-target-class: true  # true使用CGLIB代理，false使用JDK动态代理
    auto: true  # 是否启用AOP自动代理
```


完成以上配置后，就可以在项目中使用AOP来实现日志记录、性能监控、权限检查、事务管理等横切关注点功能了。