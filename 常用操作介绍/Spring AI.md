要在Spring Boot项目中使用Spring AI操作AI模型，需要进行以下准备工作：

## 1. 添加Spring AI依赖

在 [pom.xml](file://F:\BaiduNetdiskDownload\黑马点评\hm-dianping\pom.xml) 中添加Spring AI相关依赖。首先需要添加Spring AI的BOM（Bill of Materials）：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>0.8.1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```


然后添加具体的AI模块依赖，根据需要选择：

```xml
<!-- OpenAI支持 -->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai</artifactId>
</dependency>

<!-- 或者Azure OpenAI支持 -->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-azure-openai</artifactId>
</dependency>

<!-- 或者Anthropic Claude支持 -->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-anthropic</artifactId>
</dependency>

<!-- 或者Ollama本地模型支持 -->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-ollama</artifactId>
</dependency>
```


## 2. 配置API密钥和端点

在 `application.yml` 中配置相应的AI服务密钥和端点：

### OpenAI配置：
```yaml
spring:
  ai:
    openai:
      api-key: your-openai-api-key
      chat:
        options:
          model: gpt-3.5-turbo
      embedding:
        options:
          model: text-embedding-ada-002
```


### Azure OpenAI配置：
```yaml
spring:
  ai:
    azure:
      openai:
        api-key: your-azure-openai-api-key
        endpoint: https://your-resource-name.openai.azure.com
        chat:
          options:
            deployment-name: gpt-35-turbo
        embedding:
          options:
            deployment-name: text-embedding-ada-002
```


### Ollama本地模型配置：
```yaml
spring:
  ai:
    ollama:
      base-url: http://localhost:11434
      chat:
        options:
          model: llama2
      embedding:
        options:
          model: nomic-embed-text
```


## 3. 创建AI服务组件

### 注入并使用ChatModel：
```java
@Service
public class AIService {
    
    private final ChatClient chatClient;
    
    public AIService(ChatClient.Builder chatClientBuilder) {
        this.chatClient = chatClientBuilder.build();
    }
    
    public String getChatResponse(String message) {
        return chatClient.prompt()
                .user(message)
                .call()
                .content();
    }
    
    public ChatResponse getChatResponseWithDetails(String message) {
        return chatClient.prompt()
                .user(message)
                .call()
                .chatResponse();
    }
}
```


### 使用EmbeddingClient：
```java
@Service
public class EmbeddingService {
    
    private final EmbeddingClient embeddingClient;
    
    public EmbeddingService(EmbeddingClient embeddingClient) {
        this.embeddingClient = embeddingClient;
    }
    
    public List<Double> getEmbedding(String text) {
        EmbeddingResponse response = embeddingClient.embedForResponse(List.of(text));
        return response.getResult().getOutput();
    }
}
```


## 4. 配置向量存储（如果需要）

如果需要使用向量存储功能，添加相应的依赖：

```xml
<!-- Redis向量存储 -->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-redis</artifactId>
</dependency>

<!-- PostgreSQL向量存储 -->
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-pgvector</artifactId>
</dependency>
```


配置向量存储：

```yaml
spring:
  ai:
    vectorstore:
      redis:
        initialize-schema: true
        index-name: vector_index
      pgvector:
        dimensions: 1536
```


## 5. 创建Controller提供AI服务接口

```java
@RestController
@RequestMapping("/api/ai")
public class AIController {
    
    private final AIService aiService;
    
    public AIController(AIService aiService) {
        this.aiService = aiService;
    }
    
    @PostMapping("/chat")
    public ResponseEntity<String> chat(@RequestBody String message) {
        String response = aiService.getChatResponse(message);
        return ResponseEntity.ok(response);
    }
    
    @PostMapping("/embedding")
    public ResponseEntity<List<Double>> embedding(@RequestBody String text) {
        List<Double> embedding = aiService.getEmbedding(text);
        return ResponseEntity.ok(embedding);
    }
}
```


## 6. 配置Spring AI Beans（可选）

如果需要自定义配置，可以创建配置类：

```java
@Configuration
public class AIConfig {
    
    @Bean
    public ChatClient chatClient(ChatModel chatModel) {
        return ChatClient.builder(chatModel)
                .defaultSystem("你是一个有用的助手")
                .build();
    }
    
    @Bean
    public VectorStore vectorStore(RedisTemplate<String, Object> redisTemplate) {
        return new RedisVectorStore(redisTemplate);
    }
}
```


## 7. 启动相关服务

### 如果使用Ollama本地模型：
1. 下载并安装Ollama：https://ollama.com/
2. 拉取模型：`ollama pull llama2`
3. 启动Ollama服务

### 如果使用OpenAI：
1. 确保有有效的OpenAI API密钥
2. 确保网络可以访问OpenAI API

### 如果使用Azure OpenAI：
1. 在Azure门户中创建OpenAI服务
2. 获取API密钥和端点URL

完成以上配置后，就可以在Spring Boot应用中使用Spring AI来操作各种AI模型，实现聊天、文本生成、嵌入向量计算等功能了。