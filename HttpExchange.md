
# **Spring Boot HttpExchange: Full CRUD with Global Exception Handling**

With **Spring Framework 6 / Spring Boot 3**, the **`@HttpExchange`** annotation allows you to define **type-safe, declarative HTTP clients** directly in Java interfaces. This approach is a modern replacement for `RestTemplate` and works seamlessly with **blocking and reactive clients**.

In this tutorial, we’ll build a **Spring Boot application** using `HttpExchange` to perform **full CRUD operations** with **headers, query parameters**, and **global exception handling**.

---

## **Table of Contents**

1. Introduction
2. Project Setup
3. DTO Class
4. HTTP Interface Using `@HttpExchange`
5. Configuration (RestTemplate or WebClient)
6. Service Layer
7. Controller Layer
8. Global Exception Handling
9. Testing
10. Conclusion

---

## **1. Introduction**

`@HttpExchange` provides:

* Declarative HTTP calls like **Feign**
* Full **type-safety** for request/response bodies
* Support for **path variables, query parameters, headers, and body**
* Integration with **blocking RestTemplate** or **reactive WebClient**

This makes your inter-service communication **clean, maintainable, and modern**.

---

## **2. Project Setup**

Add Spring Boot Web dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

(Optional) Add **WebFlux** if you want reactive:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
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

    // Getters and setters
}
```

---

## **4. HTTP Interface Using `@HttpExchange`**

```java
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.web.service.annotation.HttpExchange;
import org.springframework.web.service.annotation.GetExchange;
import org.springframework.web.service.annotation.PostExchange;
import org.springframework.web.service.annotation.RequestBody;
import org.springframework.web.service.annotation.RequestHeader;
import org.springframework.web.service.annotation.PathVariable;
import org.springframework.web.service.annotation.RequestParam;

public interface PostClient {

    @GetExchange("/posts")
    PostDTO[] getAllPosts(@RequestParam(name = "userId", required = false) Integer userId);

    @GetExchange("/posts/{id}")
    PostDTO getPostById(@PathVariable int id,
                        @RequestHeader(HttpHeaders.AUTHORIZATION) String authToken,
                        @RequestHeader("X-Custom-Header") String customHeader);

    @PostExchange(value = "/posts", contentType = MediaType.APPLICATION_JSON_VALUE)
    PostDTO createPost(@RequestBody PostDTO post);

    @HttpExchange(method = "PUT", value = "/posts/{id}", contentType = MediaType.APPLICATION_JSON_VALUE)
    PostDTO updatePost(@PathVariable int id, @RequestBody PostDTO post);

    @HttpExchange(method = "DELETE", value = "/posts/{id}")
    void deletePost(@PathVariable int id,
                    @RequestHeader(HttpHeaders.AUTHORIZATION) String authToken,
                    @RequestHeader("X-Custom-Header") String customHeader);
}
```

---

## **5. Configuration**

### **5.1 Using RestTemplate (Blocking)**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.service.invoker.HttpServiceProxyFactory;

@Configuration
public class HttpClientConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @Bean
    public PostClient postClient(RestTemplate restTemplate) {
        HttpServiceProxyFactory factory = HttpServiceProxyFactory.builder(restTemplate).build();
        return factory.createClient(PostClient.class);
    }
}
```

### **5.2 Using WebClient (Reactive)**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.function.client.WebClient;
import org.springframework.web.service.invoker.HttpServiceProxyFactory;

@Configuration
public class HttpClientConfigReactive {

    @Bean
    public WebClient webClient() {
        return WebClient.builder().baseUrl("https://jsonplaceholder.typicode.com").build();
    }

    @Bean
    public PostClient postClient(WebClient webClient) {
        HttpServiceProxyFactory factory = HttpServiceProxyFactory.builder(webClient).build();
        return factory.createClient(PostClient.class);
    }
}
```

---

## **6. Service Layer**

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

    public PostDTO getPostById(int id, String authToken, String customHeader) {
        return postClient.getPostById(id, authToken, customHeader);
    }

    public PostDTO createPost(PostDTO post) {
        return postClient.createPost(post);
    }

    public PostDTO updatePost(int id, PostDTO post) {
        return postClient.updatePost(id, post);
    }

    public void deletePost(int id, String authToken, String customHeader) {
        postClient.deletePost(id, authToken, customHeader);
    }
}
```

---

## **7. Controller Layer**

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
    public PostDTO getPost(@PathVariable int id,
                           @RequestHeader("Authorization") String authToken,
                           @RequestHeader("X-Custom-Header") String customHeader) {
        return postService.getPostById(id, authToken, customHeader);
    }

    @PostMapping
    public PostDTO createPost(@RequestBody PostDTO post) {
        return postService.createPost(post);
    }

    @PutMapping("/{id}")
    public PostDTO updatePost(@PathVariable int id, @RequestBody PostDTO post) {
        return postService.updatePost(id, post);
    }

    @DeleteMapping("/{id}")
    public String deletePost(@PathVariable int id,
                             @RequestHeader("Authorization") String authToken,
                             @RequestHeader("X-Custom-Header") String customHeader) {
        postService.deletePost(id, authToken, customHeader);
        return "Post deleted successfully";
    }
}
```

---

## **8. Global Exception Handling**

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.client.RestClientException;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(RestClientException.class)
    public ResponseEntity<String> handleHttpExchangeException(RestClientException ex) {
        return ResponseEntity.status(HttpStatus.BAD_GATEWAY)
                .body("HTTP client error: " + ex.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGeneralException(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body("Server error: " + ex.getMessage());
    }
}
```

---

## **9. Testing the Application**

**GET all posts:**

```
GET /api/posts
GET /api/posts?userId=1
```

**GET post by ID with multiple headers:**

```
GET /api/posts/1
Authorization: Bearer token123
X-Custom-Header: my-value
```

**POST / PUT / DELETE** works similarly, passing JSON body and headers as shown above.

---

## **10. Conclusion**

Using **`@HttpExchange`**:

* Enables **modern, declarative HTTP clients** without external libraries
* Supports **full CRUD, headers, query parameters, path variables**
* Works with **blocking (`RestTemplate`) or reactive (`WebClient`) clients**
* Integrates with **global exception handling** for production-ready reliability

This approach is ideal for **synchronous inter-service communication** in **Spring Boot microservices**, providing **clean, type-safe, and maintainable code**.



Do you want me to do that next?
