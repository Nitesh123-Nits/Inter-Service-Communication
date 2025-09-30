
# **Spring Boot RestClient: Full CRUD with Global Exception Handling**

Spring Boot 3 introduced **`RestClient`**, a modern, type-safe, declarative replacement for `RestTemplate`. It simplifies **HTTP calls**, provides **non-blocking reactive support**, and integrates with Spring’s exception handling.

In this tutorial, we will build a **Spring Boot REST client** using **RestClient** with:

* Full CRUD operations
* Custom headers & query parameters
* Global exception handling

---

## **Table of Contents**

1. Introduction
2. Project Setup
3. DTO Class
4. RestClient Configuration
5. Service Layer
6. Controller Layer
7. Global Exception Handling
8. Testing
9. Conclusion

---

## **1. Introduction**

`RestClient` (introduced in Spring 6) allows:

* Declarative, type-safe HTTP clients
* Reactive and blocking calls
* Integration with Spring Boot exception handling
* Cleaner code compared to `RestTemplate`

---

## **2. Project Setup**

Add Spring Boot Web dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

> No separate WebFlux dependency is needed unless you want fully reactive non-blocking streams.

Enable **RestClient** in your configuration:

```java
import org.springframework.boot.web.client.RestClientBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestClient;

@Configuration
public class RestClientConfig {

    @Bean
    public RestClient restClient(RestClientBuilder builder) {
        return builder
                .baseUrl("https://jsonplaceholder.typicode.com")
                .build();
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

    // Getters and setters
}
```

---

## **4. Service Layer Using RestClient**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestClientException;
import org.springframework.web.client.RestClient;

@Service
public class PostService {

    private final RestClient restClient;

    @Autowired
    public PostService(RestClient restClient) {
        this.restClient = restClient;
    }

    // GET all posts with optional query param
    public PostDTO[] getAllPosts(Integer userId) {
        try {
            var request = restClient.get()
                    .uri(uriBuilder -> {
                        if (userId != null) uriBuilder.queryParam("userId", userId);
                        return uriBuilder.path("/posts").build();
                    })
                    .header(HttpHeaders.ACCEPT, MediaType.APPLICATION_JSON_VALUE);

            return request.retrieve().body(PostDTO[].class);
        } catch (RestClientException e) {
            throw new RuntimeException("Failed to fetch posts", e);
        }
    }

    // GET post by ID with header
    public PostDTO getPostById(int id, String authToken) {
        try {
            var request = restClient.get()
                    .uri("/posts/{id}", id)
                    .header("Authorization", "Bearer " + authToken);

            return request.retrieve().body(PostDTO.class);
        } catch (RestClientException e) {
            throw new RuntimeException("Failed to fetch post " + id, e);
        }
    }

    // CREATE post
    public PostDTO createPost(PostDTO postDTO) {
        try {
            return restClient.post()
                    .uri("/posts")
                    .body(postDTO)
                    .header(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
                    .retrieve()
                    .body(PostDTO.class);
        } catch (RestClientException e) {
            throw new RuntimeException("Failed to create post", e);
        }
    }

    // UPDATE post
    public PostDTO updatePost(int id, PostDTO postDTO) {
        try {
            return restClient.put()
                    .uri("/posts/{id}", id)
                    .body(postDTO)
                    .header(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
                    .retrieve()
                    .body(PostDTO.class);
        } catch (RestClientException e) {
            throw new RuntimeException("Failed to update post " + id, e);
        }
    }

    // DELETE post with header
    public void deletePost(int id, String authToken) {
        try {
            restClient.delete()
                    .uri("/posts/{id}", id)
                    .header("Authorization", "Bearer " + authToken)
                    .retrieve()
                    .toBodilessEntity();
        } catch (RestClientException e) {
            throw new RuntimeException("Failed to delete post " + id, e);
        }
    }
}
```

---

## **5. Controller Layer**

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

## **6. Global Exception Handling**

Centralized exception handling ensures consistent API responses:

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(RestClientException.class)
    public ResponseEntity<String> handleRestClientException(RestClientException ex) {
        return ResponseEntity.status(HttpStatus.BAD_GATEWAY)
                .body("Error while calling external service: " + ex.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGeneralException(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body("Server error: " + ex.getMessage());
    }
}
```

---

## **7. Testing the Application**

Example requests:

* **GET all posts**: `/api/posts` or `/api/posts?userId=1`
* **GET single post with header**: `/api/posts/1` + `Authorization`
* **POST**: `/api/posts` with JSON body

```json
{
  "userId": 1,
  "title": "New Post",
  "body": "This is a test post"
}
```

* **PUT**: `/api/posts/1` with JSON body
* **DELETE**: `/api/posts/1` + `Authorization`

---

## **8. Conclusion**

With **Spring Boot RestClient**:

* You can implement **full CRUD** operations in a clean, modern, and type-safe way
* Supports **headers, query parameters, and path variables**
* Integrates seamlessly with **global exception handling**
* Works for both **blocking and reactive** calls

This approach is ideal for **microservices integration**, modernizing legacy `RestTemplate` codebases, and building **production-ready HTTP clients**.

