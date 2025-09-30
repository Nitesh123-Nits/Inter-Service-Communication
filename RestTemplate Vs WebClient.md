
# **Spring Boot REST Client: RestTemplate vs WebClient – Full CRUD with Headers, Query Params, and Error Handling**

Integrating with external REST APIs is a core skill for backend developers. Spring Boot provides two main ways to do this:

1. **RestTemplate** – Synchronous, blocking, and simple.
2. **WebClient** – Reactive, non-blocking, and scalable.

In this tutorial, we’ll implement **full CRUD operations** (GET, POST, PUT, DELETE) with **custom headers, query parameters, and proper error handling** using both RestTemplate and WebClient. By the end, you’ll know which client to choose depending on your use case.

---

## **Table of Contents**

1. Introduction
2. Project Setup
3. DTO Class
4. RestTemplate Implementation
5. WebClient Implementation
6. Comparison: RestTemplate vs WebClient
7. Advanced Production Considerations
8. Conclusion

---

## **1. Introduction**

Many real-world APIs require:

* Custom headers (Authorization, API key)
* Query parameters
* Path variables
* Error handling and logging

This blog will demonstrate how to handle all these requirements with **RestTemplate** and **WebClient**, using the example API `https://jsonplaceholder.typicode.com/posts`.

---

## **2. Project Setup**

Add Maven dependencies:

```xml
<!-- Spring Web for RestTemplate -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- Spring WebFlux for WebClient -->
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

    // Getters and Setters
}
```

This DTO works for both clients.

---

## **4. RestTemplate Implementation**

### **4.1 Configuration**

```java
@Configuration
public class RestTemplateConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

### **4.2 Service Layer**

```java
@Service
public class RestTemplatePostService {
    private final RestTemplate restTemplate;
    private static final String BASE_URL = "https://jsonplaceholder.typicode.com/posts";

    @Autowired
    public RestTemplatePostService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    public PostDTO[] getAllPosts(Integer userId) {
        UriComponentsBuilder uriBuilder = UriComponentsBuilder.fromHttpUrl(BASE_URL);
        if (userId != null) uriBuilder.queryParam("userId", userId);

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
    }

    public PostDTO getPostById(int id, String authToken) {
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
    }

    public PostDTO createPost(PostDTO postDTO) {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity<PostDTO> request = new HttpEntity<>(postDTO, headers);
        return restTemplate.postForObject(BASE_URL, request, PostDTO.class);
    }

    public PostDTO updatePost(int id, PostDTO postDTO) {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity<PostDTO> request = new HttpEntity<>(postDTO, headers);

        ResponseEntity<PostDTO> response = restTemplate.exchange(
                BASE_URL + "/{id}", HttpMethod.PUT, request, PostDTO.class, id
        );
        return response.getBody();
    }

    public void deletePost(int id, String authToken) {
        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "Bearer " + authToken);
        HttpEntity<String> request = new HttpEntity<>(headers);

        restTemplate.exchange(BASE_URL + "/{id}", HttpMethod.DELETE, request, Void.class, id);
    }
}
```

### **4.3 Controller Layer**

```java
@RestController
@RequestMapping("/api/resttemplate/posts")
public class RestTemplatePostController {
    @Autowired
    private RestTemplatePostService postService;

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
        return "Post deleted";
    }
}
```

---

## **5. WebClient Implementation**

### **5.1 Configuration**

```java
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

### **5.2 Service Layer**

```java
@Service
public class WebClientPostService {
    private final WebClient webClient;

    @Autowired
    public WebClientPostService(WebClient webClient) {
        this.webClient = webClient;
    }

    public Mono<PostDTO[]> getAllPosts(Integer userId) {
        return webClient.get()
                .uri(uriBuilder -> {
                    if (userId != null) uriBuilder.queryParam("userId", userId);
                    return uriBuilder.path("/posts").build();
                })
                .retrieve()
                .bodyToMono(PostDTO[].class);
    }

    public Mono<PostDTO> getPostById(int id, String authToken) {
        return webClient.get()
                .uri("/posts/{id}", id)
                .header("Authorization", "Bearer " + authToken)
                .retrieve()
                .bodyToMono(PostDTO.class);
    }

    public Mono<PostDTO> createPost(PostDTO postDTO) {
        return webClient.post()
                .uri("/posts")
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue(postDTO)
                .retrieve()
                .bodyToMono(PostDTO.class);
    }

    public Mono<PostDTO> updatePost(int id, PostDTO postDTO) {
        return webClient.put()
                .uri("/posts/{id}", id)
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue(postDTO)
                .retrieve()
                .bodyToMono(PostDTO.class);
    }

    public Mono<Void> deletePost(int id, String authToken) {
        return webClient.delete()
                .uri("/posts/{id}", id)
                .header("Authorization", "Bearer " + authToken)
                .retrieve()
                .bodyToMono(Void.class);
    }
}
```

### **5.3 Controller Layer**

```java
@RestController
@RequestMapping("/api/webclient/posts")
public class WebClientPostController {
    @Autowired
    private WebClientPostService postService;

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
        return postService.deletePost(id, authToken).thenReturn("Post deleted");
    }
}
```

---

## **6. Comparison: RestTemplate vs WebClient**

| Feature                  | RestTemplate       | WebClient                                         |
| ------------------------ | ------------------ | ------------------------------------------------- |
| Nature                   | Synchronous        | Reactive, Non-blocking                            |
| Threading                | Blocking           | Non-blocking, reactive                            |
| Scalability              | Limited under load | High, handles concurrency better                  |
| Reactive Streams Support | No                 | Yes                                               |
| Suitable for             | Simple API calls   | High-load APIs, reactive microservices            |
| Error Handling           | Exception-based    | Can handle reactively, supports retries & filters |

---

## **7. Advanced Production Considerations**

* **Logging**: Use `Slf4j` or WebClient filters for logging requests/responses.
* **Timeouts**: Configure RestTemplate or WebClient with connection/read timeouts.
* **Retries**: WebClient supports `retryWhen` for transient errors.
* **Custom Headers**: Pass Authorization, API keys, or custom metadata.
* **Error Handling**: Wrap exceptions in custom classes or return meaningful messages.

---

## **8. Conclusion**

Both **RestTemplate** and **WebClient** are valid HTTP clients in Spring Boot.

* Use **RestTemplate** for simple, blocking calls or legacy systems.
* Use **WebClient** for modern, reactive, scalable microservices.

This tutorial showed:

* Full CRUD examples for both clients
* Custom headers, query parameters, and path variables
* Exception handling and logging tips
* Comparison table for decision-making

By following this, you can build robust **REST clients in Spring Boot** for any production use case.



Do you want me to do that next?
