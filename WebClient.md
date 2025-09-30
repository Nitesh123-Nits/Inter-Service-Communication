
# **Spring Boot WebClient: Full CRUD Example with Headers and Query Params**

---

## **1. Maven Dependencies**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

> `WebClient` is part of **Spring WebFlux**, so you need `spring-boot-starter-webflux`.

---

## **2. DTO Class**

Same as before:

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

## **3. WebClient Configuration**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.function.client.WebClient;

@Configuration
public class WebClientConfig {

    @Bean
    public WebClient webClient() {
        return WebClient.builder()
                .baseUrl("https://jsonplaceholder.typicode.com")
                .build();
    }
}
```

---

## **4. Service Layer with Full CRUD**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Service;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

@Service
public class PostService {

    private final WebClient webClient;

    @Autowired
    public PostService(WebClient webClient) {
        this.webClient = webClient;
    }

    // GET all posts with optional query param
    public Mono<PostDTO[]> getAllPosts(Integer userId) {
        WebClient.RequestHeadersUriSpec<?> request = webClient.get().uri(uriBuilder -> {
            if (userId != null) uriBuilder.queryParam("userId", userId);
            return uriBuilder.path("/posts").build();
        });

        return request
                .accept(MediaType.APPLICATION_JSON)
                .retrieve()
                .bodyToMono(PostDTO[].class);
    }

    // GET single post with header
    public Mono<PostDTO> getPostById(int id, String authToken) {
        return webClient.get()
                .uri("/posts/{id}", id)
                .header("Authorization", "Bearer " + authToken)
                .accept(MediaType.APPLICATION_JSON)
                .retrieve()
                .bodyToMono(PostDTO.class);
    }

    // CREATE post
    public Mono<PostDTO> createPost(PostDTO postDTO) {
        return webClient.post()
                .uri("/posts")
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue(postDTO)
                .retrieve()
                .bodyToMono(PostDTO.class);
    }

    // UPDATE post
    public Mono<PostDTO> updatePost(int id, PostDTO postDTO) {
        return webClient.put()
                .uri("/posts/{id}", id)
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue(postDTO)
                .retrieve()
                .bodyToMono(PostDTO.class);
    }

    // DELETE post with header
    public Mono<Void> deletePost(int id, String authToken) {
        return webClient.delete()
                .uri("/posts/{id}", id)
                .header("Authorization", "Bearer " + authToken)
                .retrieve()
                .bodyToMono(Void.class);
    }
}
```

---

## **5. Controller Layer**

Because `WebClient` is reactive, we return **`Mono` or `Flux`**:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/api/posts")
public class PostController {

    private final PostService postService;

    @Autowired
    public PostController(PostService postService) {
        this.postService = postService;
    }

    @GetMapping
    public Mono<PostDTO[]> getAllPosts(@RequestParam(required = false) Integer userId) {
        return postService.getAllPosts(userId);
    }

    @GetMapping("/{id}")
    public Mono<PostDTO> getPost(@PathVariable int id, @RequestHeader("Authorization") String authToken) {
        return postService.getPostById(id, authToken);
    }

    @PostMapping
    public Mono<PostDTO> createPost(@RequestBody PostDTO postDTO) {
        return postService.createPost(postDTO);
    }

    @PutMapping("/{id}")
    public Mono<PostDTO> updatePost(@PathVariable int id, @RequestBody PostDTO postDTO) {
        return postService.updatePost(id, postDTO);
    }

    @DeleteMapping("/{id}")
    public Mono<String> deletePost(@PathVariable int id, @RequestHeader("Authorization") String authToken) {
        return postService.deletePost(id, authToken)
                .thenReturn("Post with id " + id + " deleted successfully");
    }
}
```

---

## **6. Running and Testing**

Requests are almost identical to the `RestTemplate` example:

* **GET with query param** → `/api/posts?userId=1`
* **GET single post with header** → `/api/posts/1` + `Authorization` header
* **POST / PUT** → JSON body
* **DELETE** → Authorization header

Since `WebClient` is reactive, responses are **non-blocking**, so it scales better under load.

---

## **7. Advantages over RestTemplate**

* Non-blocking, reactive
* Better scalability for high-load systems
* Easier integration with **Spring WebFlux** and reactive streams
* Supports **advanced features** like retries, filters, and backpressure

---

