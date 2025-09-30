
# **Spring Boot REST Client with RestTemplate: Full CRUD Example**

Integrating with external REST APIs is a common requirement in modern backend development. In Spring Boot, the **`RestTemplate`** class provides a synchronous HTTP client for making REST calls. In this tutorial, we’ll explore a **full CRUD example** using `RestTemplate`, including `GET`, `POST`, `PUT`, and `DELETE` operations.

By the end, you’ll have a reusable template for building REST clients in production-grade Spring Boot applications.

---

## **Table of Contents**

1. Introduction
2. Project Setup
3. DTO Class
4. Configuring RestTemplate
5. Service Layer
6. Controller Layer
7. Running the Application
8. Conclusion

---

## **1. Introduction**

`RestTemplate` is part of **Spring Web** and provides a convenient way to perform HTTP requests to external services. While Spring now recommends `WebClient` for reactive programming, `RestTemplate` remains widely used for simple synchronous calls.

In this tutorial, we’ll simulate CRUD operations with `https://jsonplaceholder.typicode.com/posts` — a free online REST API for testing.

---

## **2. Project Setup**

Add the following dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

This provides the `RestTemplate` class and all necessary Spring Web utilities.

---

## **3. DTO Class**

Define a **PostDTO** class to map JSON data:

```java
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class PostDTO {
    private Integer id;
    private Integer userId;
    private String title;
    private String body;

    // Constructors
    public PostDTO() {}
    public PostDTO(Integer userId, String title, String body) {
        this.userId = userId;
        this.title = title;
        this.body = body;
    }

    // Getters and Setters
    public Integer getId() { return id; }
    public void setId(Integer id) { this.id = id; }
    public Integer getUserId() { return userId; }
    public void setUserId(Integer userId) { this.userId = userId; }
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public String getBody() { return body; }
    public void setBody(String body) { this.body = body; }
}
```

This DTO ensures that our application can serialize and deserialize JSON data seamlessly.

---

## **4. Configuring RestTemplate**

We’ll configure `RestTemplate` as a Spring Bean so it can be reused across the application:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestTemplateConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

---

## **5. Service Layer**

The **PostService** class performs all REST API operations:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.*;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.client.RestClientException;

@Service
public class PostService {

    private final RestTemplate restTemplate;
    private static final String BASE_URL = "https://jsonplaceholder.typicode.com/posts";

    @Autowired
    public PostService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    // GET all posts
    public PostDTO[] getAllPosts() {
        try {
            return restTemplate.getForObject(BASE_URL, PostDTO[].class);
        } catch (RestClientException e) {
            throw new RuntimeException("Failed to fetch posts", e);
        }
    }

    // GET single post
    public PostDTO getPostById(int id) {
        try {
            return restTemplate.getForObject(BASE_URL + "/{id}", PostDTO.class, id);
        } catch (RestClientException e) {
            throw new RuntimeException("Failed to fetch post with id " + id, e);
        }
    }

    // CREATE post
    public PostDTO createPost(PostDTO postDTO) {
        try {
            return restTemplate.postForObject(BASE_URL, postDTO, PostDTO.class);
        } catch (RestClientException e) {
            throw new RuntimeException("Failed to create post", e);
        }
    }

    // UPDATE post
    public PostDTO updatePost(int id, PostDTO postDTO) {
        try {
            HttpEntity<PostDTO> requestUpdate = new HttpEntity<>(postDTO);
            ResponseEntity<PostDTO> response = restTemplate.exchange(
                    BASE_URL + "/{id}", HttpMethod.PUT, requestUpdate, PostDTO.class, id);
            return response.getBody();
        } catch (RestClientException e) {
            throw new RuntimeException("Failed to update post with id " + id, e);
        }
    }

    // DELETE post
    public void deletePost(int id) {
        try {
            restTemplate.delete(BASE_URL + "/{id}", id);
        } catch (RestClientException e) {
            throw new RuntimeException("Failed to delete post with id " + id, e);
        }
    }
}
```

---

## **6. Controller Layer**

Expose REST endpoints in your application to interact with the service:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/posts")
public class PostController {

    private final PostService postService;

    @Autowired
    public PostController(PostService postService) {
        this.postService = postService;
    }

    @GetMapping
    public PostDTO[] getAllPosts() {
        return postService.getAllPosts();
    }

    @GetMapping("/{id}")
    public PostDTO getPost(@PathVariable int id) {
        return postService.getPostById(id);
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
    public String deletePost(@PathVariable int id) {
        postService.deletePost(id);
        return "Post with id " + id + " deleted successfully";
    }
}
```

---

## **7. Running the Application**

Run the Spring Boot application:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class RestTemplateCrudApplication {

    public static void main(String[] args) {
        SpringApplication.run(RestTemplateCrudApplication.class, args);
    }
}
```

You can now test your CRUD operations:

* `GET /api/posts` → Fetch all posts
* `GET /api/posts/{id}` → Fetch single post
* `POST /api/posts` → Create a post
* `PUT /api/posts/{id}` → Update a post
* `DELETE /api/posts/{id}` → Delete a post

---

## **8. Conclusion**

In this tutorial, we:

* Configured **RestTemplate** as a Spring Bean.
* Created a **DTO class** for JSON mapping.
* Built a **service layer** to handle GET, POST, PUT, DELETE requests.
* Exposed REST endpoints via a **controller**.

This setup provides a **reusable template** for integrating any REST API with Spring Boot. For production systems, you may add **logging, error handling, and retry mechanisms** for more robust API communication.









-----------------------------------------------------------------------------------------------------------------------------
* Full CRUD examples
* **Custom headers** (Authorization, Content-Type)
* Query parameters
* Path variables
* Exception handling
* Logging

This will demonstrate **real-world use cases for RestTemplate**.

---

# **Spring Boot REST Client with RestTemplate: Full CRUD with Headers, Query Params, and Exception Handling**

Integrating with external REST APIs is a fundamental requirement in modern applications. Spring Boot’s **`RestTemplate`** provides a simple, synchronous way to interact with REST services.

In this tutorial, we’ll implement a **full CRUD example** covering all common use cases, including **custom headers, query parameters, exception handling, and logging**.

---

## **Table of Contents**

1. Introduction
2. Project Setup
3. DTO Class
4. RestTemplate Configuration
5. Service Layer with Full Use Cases
6. Controller Layer
7. Running and Testing the Application
8. Advanced Tips
9. Conclusion

---

## **1. Introduction**

`RestTemplate` allows you to make HTTP requests such as **GET, POST, PUT, DELETE**.

In production, APIs often require:

* Custom headers (e.g., Authorization tokens)
* Query parameters
* Logging and proper error handling

We’ll use `https://jsonplaceholder.typicode.com/posts` as the example API.

---

## **2. Project Setup**

Add the Spring Web dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Optionally, for logging:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
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

    // Getters and setters omitted for brevity
}
```

---

## **4. RestTemplate Configuration**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestTemplateConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

---

## **5. Service Layer with Full Use Cases**

This service demonstrates **all CRUD operations with headers, query params, and exception handling**:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.*;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestClientException;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.util.UriComponentsBuilder;

import java.util.Collections;
import java.util.Map;

@Service
public class PostService {

    private final RestTemplate restTemplate;
    private static final String BASE_URL = "https://jsonplaceholder.typicode.com/posts";

    @Autowired
    public PostService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    // GET all posts with optional query param
    public PostDTO[] getAllPosts(Integer userId) {
        try {
            UriComponentsBuilder uriBuilder = UriComponentsBuilder.fromHttpUrl(BASE_URL);
            if (userId != null) {
                uriBuilder.queryParam("userId", userId);
            }

            HttpHeaders headers = new HttpHeaders();
            headers.setAccept(Collections.singletonList(MediaType.APPLICATION_JSON));

            HttpEntity<String> entity = new HttpEntity<>(headers);
            ResponseEntity<PostDTO[]> response = restTemplate.exchange(
                    uriBuilder.toUriString(),
                    HttpMethod.GET,
                    entity,
                    PostDTO[].class
            );

            return response.getBody();
        } catch (RestClientException e) {
            throw new RuntimeException("Failed to fetch posts", e);
        }
    }

    // GET single post with custom header
    public PostDTO getPostById(int id, String authToken) {
        try {
            HttpHeaders headers = new HttpHeaders();
            headers.set("Authorization", "Bearer " + authToken);
            HttpEntity<String> entity = new HttpEntity<>(headers);

            ResponseEntity<PostDTO> response = restTemplate.exchange(
                    BASE_URL + "/{id}",
                    HttpMethod.GET,
                    entity,
                    PostDTO.class,
                    id
            );
            return response.getBody();
        } catch (RestClientException e) {
            throw new RuntimeException("Failed to fetch post with id " + id, e);
        }
    }

    // CREATE post with headers
    public PostDTO createPost(PostDTO postDTO) {
        try {
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);
            HttpEntity<PostDTO> request = new HttpEntity<>(postDTO, headers);

            return restTemplate.postForObject(BASE_URL, request, PostDTO.class);
        } catch (RestClientException e) {
            throw new RuntimeException("Failed to create post", e);
        }
    }

    // UPDATE post with headers
    public PostDTO updatePost(int id, PostDTO postDTO) {
        try {
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);
            HttpEntity<PostDTO> request = new HttpEntity<>(postDTO, headers);

            ResponseEntity<PostDTO> response = restTemplate.exchange(
                    BASE_URL + "/{id}",
                    HttpMethod.PUT,
                    request,
                    PostDTO.class,
                    id
            );
            return response.getBody();
        } catch (RestClientException e) {
            throw new RuntimeException("Failed to update post with id " + id, e);
        }
    }

    // DELETE post with header
    public void deletePost(int id, String authToken) {
        try {
            HttpHeaders headers = new HttpHeaders();
            headers.set("Authorization", "Bearer " + authToken);
            HttpEntity<String> request = new HttpEntity<>(headers);

            restTemplate.exchange(BASE_URL + "/{id}", HttpMethod.DELETE, request, Void.class, id);
        } catch (RestClientException e) {
            throw new RuntimeException("Failed to delete post with id " + id, e);
        }
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

    private final PostService postService;

    @Autowired
    public PostController(PostService postService) {
        this.postService = postService;
    }

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
        return "Post with id " + id + " deleted successfully";
    }
}
```

---

## **7. Running and Testing the Application**

### Example Requests

**GET with query param**

```http
GET /api/posts?userId=1
```

**GET with custom header**

```http
GET /api/posts/1
Authorization: Bearer token123
```

**POST with JSON body**

```json
POST /api/posts
{
  "userId": 1,
  "title": "New Post",
  "body": "This is a test post"
}
```

**PUT with JSON body**

```json
PUT /api/posts/1
{
  "userId": 1,
  "title": "Updated Title",
  "body": "Updated body"
}
```

**DELETE with custom header**

```http
DELETE /api/posts/1
Authorization: Bearer token123
```

---

## **8. Advanced Tips**

* **Logging**: Add `Slf4j` logger in the service for request/response logging.
* **Retries**: Use `Spring Retry` to handle transient errors.
* **Timeouts**: Configure `RestTemplate` with `HttpComponentsClientHttpRequestFactory` for connection and read timeouts.
* **Error Handling**: Wrap exceptions in custom exceptions for better API responses.

---

## **9. Conclusion**

This tutorial demonstrates **full CRUD operations with `RestTemplate`**, including:

* Custom headers
* Query parameters
* Path variables
* Proper exception handling

This setup can be extended to **secure APIs, external microservices, or third-party integrations**.

---

I can also **create a ready-to-publish Medium/Dev.to version with diagrams, curl examples, and formatted code blocks**, making it a **complete production blog**.

Do you want me to do that next?
