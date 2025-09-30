
# **Spring Boot Feign Client: Full CRUD with Global Exception Handling**

Microservices often need to communicate with each other. While **`RestTemplate`** and **`WebClient`** are popular, **Spring Cloud Feign** (declarative REST client) simplifies inter-service calls with **interface-based clients**, automatic serialization, and load balancing.

In this tutorial, we will build a **Spring Boot application using Feign Client** to perform **full CRUD operations**, include **custom headers and query parameters**, and implement **global exception handling**.

---

## **Table of Contents**

1. Introduction
2. Project Setup
3. DTO Class
4. Feign Client Configuration
5. Service Layer
6. Controller Layer
7. Global Exception Handling
8. Testing the Application
9. Conclusion

---

## **1. Introduction**

Feign provides a declarative way to define HTTP clients using Java interfaces:

* Less boilerplate than `RestTemplate`
* Supports **path variables, query parameters, headers, and request bodies**
* Integrates with **Ribbon/Spring Cloud LoadBalancer** for client-side load balancing
* Supports **custom error handling via FeignErrorDecoder**

---

## **2. Project Setup**

Add the following dependencies in `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Enable Feign Clients in the main application class:

```java
@SpringBootApplication
@EnableFeignClients
public class FeignClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(FeignClientApplication.class, args);
    }
}
```

---

## **3. DTO Class**

```java
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class PostDTO {
    private Integer id;
    private Integer userId;
    private String title;
    private String body;

    public PostDTO() {}
    public PostDTO(Integer userId, String title, String body) {
        this.userId = userId;
        this.title = title;
        this.body = body;
    }

    // Getters and Setters
}
```

---

## **4. Feign Client Interface**

Define your client using Feign annotations:

```java
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.*;

@FeignClient(name = "post-service", url = "https://jsonplaceholder.typicode.com",
             configuration = FeignConfig.class)
public interface PostClient {

    @GetMapping("/posts")
    PostDTO[] getAllPosts(@RequestParam(value = "userId", required = false) Integer userId);

    @GetMapping("/posts/{id}")
    PostDTO getPostById(@PathVariable("id") int id, @RequestHeader("Authorization") String authToken);

    @PostMapping("/posts")
    PostDTO createPost(@RequestBody PostDTO postDTO);

    @PutMapping("/posts/{id}")
    PostDTO updatePost(@PathVariable("id") int id, @RequestBody PostDTO postDTO);

    @DeleteMapping("/posts/{id}")
    void deletePost(@PathVariable("id") int id, @RequestHeader("Authorization") String authToken);
}
```

---

### **4.1 Feign Client Configuration**

Optionally, add custom error decoder and logging:

```java
import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignConfig {

    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }

    @Bean
    public CustomFeignErrorDecoder errorDecoder() {
        return new CustomFeignErrorDecoder();
    }
}
```

---

## **5. Service Layer**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class PostService {

    private final PostClient postClient;

    @Autowired
    public PostService(PostClient postClient) {
        this.postClient = postClient;
    }

    public PostDTO[] getAllPosts(Integer userId) {
        return postClient.getAllPosts(userId);
    }

    public PostDTO getPostById(int id, String authToken) {
        return postClient.getPostById(id, authToken);
    }

    public PostDTO createPost(PostDTO postDTO) {
        return postClient.createPost(postDTO);
    }

    public PostDTO updatePost(int id, PostDTO postDTO) {
        return postClient.updatePost(id, postDTO);
    }

    public void deletePost(int id, String authToken) {
        postClient.deletePost(id, authToken);
    }
}
```

---

## **6. Controller Layer**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/posts")
public class PostController {

    @Autowired
    private PostService postService;

    @GetMapping
    public PostDTO[] getAllPosts(@RequestParam(required = false) Integer userId) {
        return postService.getAllPosts(userId);
    }

    @GetMapping("/{id}")
    public PostDTO getPost(@PathVariable int id, @RequestHeader("Authorization") String authToken) {
        return postService.getPostById(id, authToken);
    }

    @PostMapping
    public PostDTO createPost(@RequestBody PostDTO postDTO) {
        return postService.createPost(postDTO);
    }

    @PutMapping("/{id}")
    public PostDTO updatePost(@PathVariable int id, @RequestBody PostDTO postDTO) {
        return postService.updatePost(id, postDTO);
    }

    @DeleteMapping("/{id}")
    public String deletePost(@PathVariable int id, @RequestHeader("Authorization") String authToken) {
        postService.deletePost(id, authToken);
        return "Post deleted successfully";
    }
}
```

---

## **7. Global Exception Handling**

Use `@ControllerAdvice` for centralized exception handling:

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(FeignException.class)
    public ResponseEntity<String> handleFeignException(FeignException ex) {
        return new ResponseEntity<>("Feign client error: " + ex.getMessage(), HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGeneralException(Exception ex) {
        return new ResponseEntity<>("Server error: " + ex.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

### **7.1 Custom Feign Error Decoder**

```java
import feign.Response;
import feign.codec.ErrorDecoder;

public class CustomFeignErrorDecoder implements ErrorDecoder {

    private final ErrorDecoder defaultDecoder = new Default();

    @Override
    public Exception decode(String methodKey, Response response) {
        switch (response.status()) {
            case 400:
                return new RuntimeException("Bad request in method " + methodKey);
            case 404:
                return new RuntimeException("Resource not found");
            default:
                return defaultDecoder.decode(methodKey, response);
        }
    }
}
```

---

## **8. Testing the Application**

**GET all posts**:

```
GET /api/posts
```

**GET post with Authorization header**:

```
GET /api/posts/1
Authorization: Bearer token123
```

**POST**:

```
POST /api/posts
{
  "userId":1,
  "title":"New Post",
  "body":"This is a test"
}
```

**PUT**:

```
PUT /api/posts/1
{
  "userId":1,
  "title":"Updated Post",
  "body":"Updated body"
}
```

**DELETE**:

```
DELETE /api/posts/1
Authorization: Bearer token123
```

---

## **9. Conclusion**

Using **Spring Cloud Feign**:

* Simplifies inter-service HTTP calls with **interfaces**
* Supports **CRUD operations, headers, path/query parameters**
* Integrates with **custom error handling** for robust production-ready applications
* Works well with **service discovery** and **load balancing**

This setup is ideal for **microservice architectures**, providing clean, declarative, and maintainable code for REST clients.

---

