# Complete API Guide: Foundations to Implementation

## Table of Contents
1. [Understanding the API Ecosystem](#understanding-the-api-ecosystem)
2. [HTTP Protocol Foundations](#http-protocol-foundations)
3. [REST API Fundamentals](#rest-api-fundamentals)
4. [Communication Patterns: Push vs Pull](#communication-patterns-push-vs-pull)
5. [API Paradigms Comparison](#api-paradigms-comparison)
6. [Java Spring Boot Implementation](#java-spring-boot-implementation)
7. [Best Practices and Design Patterns](#best-practices-and-design-patterns)
8. [Security and Authentication](#security-and-authentication)
9. [API Documentation and Testing](#api-documentation-and-testing)
10. [Performance and Scaling](#performance-and-scaling)

---

## Understanding the API Ecosystem

### The Relationship Hierarchy
```
HTTP Protocol (Foundation)
    ↓
HTTP API (Uses HTTP for communication)
    ↓
REST API (HTTP API + REST principles)
```

**API (Application Programming Interface)** is the broadest concept - any set of rules allowing software components to communicate. APIs can use various protocols: HTTP, TCP, message queues, or even direct function calls.

**HTTP APIs** specifically use the HTTP protocol as their communication mechanism. When you make a request to `GET /users/123`, you're using HTTP as the transport layer.

**REST APIs** are a specialized type of HTTP API that follows REST architectural principles, using HTTP methods semantically and structuring URLs as resources.

**REST (Representational State Transfer)** is a software architectural style that defines a set of constraints for creating web services. The core principles of REST include client-server architecture, statelessness, cacheability, a uniform interface, a layered system and code on demand. These principles promote flexibility, scalability and maintainability in web service design.

### Key Terminology
- **Endpoint**: A specific URL where an API can be accessed
- **Resource**: A data entity that can be accessed via the API
- **Client**: The application making API requests
- **Server**: The application serving API responses
- **Payload**: The data sent in requests/responses

---

## HTTP Protocol Foundations

### HTTP Methods (Verbs)
| Method | Purpose | Safe | Idempotent | Body |
|--------|---------|------|------------|------|
| GET | Retrieve data | ✓ | ✓ | No |
| POST | Create new resource | ✗ | ✗ | Yes |
| PUT | Update/replace entire resource | ✗ | ✓ | Yes |
| PATCH | Partial update | ✗ | ✗ | Yes |
| DELETE | Remove resource | ✗ | ✓ | Optional |
| HEAD | Get headers only | ✓ | ✓ | No |
| OPTIONS | Get allowed methods | ✓ | ✓ | No |

### HTTP Status Codes
#### 2xx Success
- **200 OK**: Request successful
- **201 Created**: Resource created successfully
- **202 Accepted**: Request accepted for processing
- **204 No Content**: Success with no response body

#### 3xx Redirection
- **301 Moved Permanently**: Resource permanently moved
- **302 Found**: Resource temporarily moved
- **304 Not Modified**: Resource not modified (caching)

#### 4xx Client Errors
- **400 Bad Request**: Invalid request syntax
- **401 Unauthorized**: Authentication required
- **403 Forbidden**: Access denied
- **404 Not Found**: Resource not found
- **409 Conflict**: Resource conflict
- **422 Unprocessable Entity**: Validation failed
- **429 Too Many Requests**: Rate limit exceeded

#### 5xx Server Errors
- **500 Internal Server Error**: Generic server error
- **502 Bad Gateway**: Invalid response from upstream
- **503 Service Unavailable**: Server temporarily unavailable
- **504 Gateway Timeout**: Upstream timeout

### Essential HTTP Headers
#### Request Headers
- **Content-Type**: Format of request body (`application/json`)
- **Accept**: Expected response format
- **Authorization**: Authentication credentials
- **User-Agent**: Client identification
- **Cache-Control**: Caching directives

#### Response Headers
- **Content-Type**: Format of response body
- **Location**: URL of newly created resource
- **ETag**: Resource version identifier
- **Cache-Control**: Caching instructions
- **Access-Control-Allow-Origin**: CORS policy

### Request/Response Structure
```
REQUEST:
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

{
  "name": "John Doe",
  "email": "john@example.com"
}

RESPONSE:
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/users/123

{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "created_at": "2025-07-10T10:30:00Z"
}
```

---

## REST API Fundamentals

### Core Principles
1. **Stateless**: Each request contains all necessary information
2. **Resource-Based**: URLs represent resources, not actions
3. **HTTP Methods**: Use verbs semantically
4. **Representations**: Resources can have multiple formats (JSON, XML)
5. **HATEOAS**: Hypermedia as the Engine of Application State

### RESTful URL Design
#### Good Examples
```
GET    /users           # List all users
GET    /users/123       # Get specific user
POST   /users           # Create new user
PUT    /users/123       # Update user (full replacement)
PATCH  /users/123       # Partial update
DELETE /users/123       # Delete user

# Nested resources
GET    /users/123/orders      # User's orders
POST   /users/123/orders      # Create order for user
GET    /users/123/orders/456  # Specific order
```

#### Avoid These Patterns
```
❌ /getUsers           # Action in URL
❌ /users/delete/123   # Action in URL
❌ /users?action=delete&id=123  # Action in query
```

### CRUD Operations Mapping
| Operation | HTTP Method | URL Pattern | Purpose |
|-----------|-------------|-------------|---------|
| Create | POST | `/users` | Create new user |
| Read | GET | `/users` or `/users/123` | List or get user |
| Update | PUT/PATCH | `/users/123` | Update user |
| Delete | DELETE | `/users/123` | Delete user |

### Response Format Standards
```json
{
  "data": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com"
  },
  "meta": {
    "timestamp": "2025-07-10T10:30:00Z",
    "version": "1.0"
  }
}
```

### Error Response Format
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  }
}
```

---

## Communication Patterns: Push vs Pull

### Pull Model (Client-Initiated)
**Definition**: Client requests data when needed

#### Characteristics
- Client controls timing and frequency
- Server responds only when asked
- Stateless interactions
- Cacheable responses

#### Implementation Patterns
```javascript
// Traditional REST API call
fetch('/api/users/123')
  .then(response => response.json())
  .then(data => console.log(data));

// GraphQL query
const query = `
  query {
    user(id: 123) {
      name
      email
    }
  }
`;

// Polling for updates
setInterval(() => {
  fetch('/api/notifications')
    .then(response => response.json())
    .then(data => updateUI(data));
}, 5000);
```

#### Use Cases
- User-initiated actions (search, view profile)
- Data that doesn't change frequently
- Simple request-response patterns
- Public APIs with unknown client needs

### Push Model (Server-Initiated)
**Definition**: Server sends data to client when events occur

#### Implementation Technologies

**WebSockets**: Full-duplex communication
```javascript
const socket = new WebSocket('ws://localhost:8080');

socket.onmessage = (event) => {
  const data = JSON.parse(event.data);
  updateUI(data);
};

socket.send(JSON.stringify({
  type: 'subscribe',
  channel: 'notifications'
}));
```

**Server-Sent Events (SSE)**: Server-to-client streaming
```javascript
const eventSource = new EventSource('/api/events');

eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  updateUI(data);
};
```

**Webhooks**: Server calls client endpoint
```javascript
// Server-side webhook call
app.post('/webhook/payment', (req, res) => {
  const paymentData = req.body;
  
  // Process payment event
  processPayment(paymentData);
  
  res.status(200).send('OK');
});
```

#### Use Cases
- Real-time notifications (chat, alerts)
- Live data feeds (stock prices, sports scores)
- Event-driven architectures
- Collaborative applications

### Hybrid Approaches

**Long Polling**: Client requests, server holds until data available
```javascript
async function longPoll() {
  try {
    const response = await fetch('/api/long-poll', {
      timeout: 30000
    });
    const data = await response.json();
    
    // Process data
    updateUI(data);
    
    // Immediately start next poll
    longPoll();
  } catch (error) {
    // Retry after delay
    setTimeout(longPoll, 5000);
  }
}
```

**Polling with Backoff**: Adaptive polling frequency
```javascript
let pollInterval = 1000;
const maxInterval = 30000;

function adaptivePoll() {
  fetch('/api/status')
    .then(response => response.json())
    .then(data => {
      if (data.hasUpdates) {
        pollInterval = 1000; // Reset to frequent polling
        updateUI(data);
      } else {
        pollInterval = Math.min(pollInterval * 1.5, maxInterval);
      }
      
      setTimeout(adaptivePoll, pollInterval);
    });
}
```

---

## API Paradigms Comparison

### REST (Representational State Transfer)
**Philosophy**: Resource-oriented architecture using HTTP semantically

#### Characteristics
- **Resources**: Everything is a resource with a URL
- **HTTP Methods**: Use GET, POST, PUT, DELETE semantically
- **Stateless**: Each request is independent
- **Cacheable**: Responses can be cached

#### Example
```http
GET /api/users/123 HTTP/1.1
Accept: application/json

HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}
```

#### Strengths
- Simple and intuitive
- Widely understood and supported
- Excellent caching capabilities
- Stateless and scalable
- Great for CRUD operations

#### Weaknesses
- Over-fetching/under-fetching data
- Multiple round trips for complex operations
- Limited query capabilities
- Versioning challenges

### RPC (Remote Procedure Call)
**Philosophy**: Make remote calls feel like local function calls

#### Variants

**JSON-RPC**: Simple RPC using JSON
```json
// Request
{
  "jsonrpc": "2.0",
  "method": "getUser",
  "params": {"id": 123},
  "id": 1
}

// Response
{
  "jsonrpc": "2.0",
  "result": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com"
  },
  "id": 1
}
```

**gRPC**: High-performance RPC with Protocol Buffers
```protobuf
// user.proto
service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc CreateUser(CreateUserRequest) returns (User);
}

message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}
```

**SOAP**: XML-based enterprise protocol
```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <GetUserRequest>
      <UserId>123</UserId>
    </GetUserRequest>
  </soap:Body>
</soap:Envelope>
```

#### Strengths
- Intuitive function-like interface
- Strong typing (gRPC, SOAP)
- Efficient binary protocols (gRPC)
- Built-in code generation
- Excellent for internal services

#### Weaknesses
- Tighter coupling between client and server
- Harder to cache than REST
- Limited HTTP semantics usage
- Discovery and documentation challenges

### GraphQL
**Philosophy**: Query-oriented API with client-controlled data fetching

#### Core Concepts
- **Single Endpoint**: All operations through one URL
- **Flexible Queries**: Client specifies exact data needed
- **Type System**: Strong typing with schema definition
- **Real-time**: Built-in subscription support

#### Example Query
```graphql
query {
  user(id: 123) {
    name
    email
    orders(limit: 5) {
      id
      total
      items {
        name
        price
      }
    }
  }
}
```

#### Response
```json
{
  "data": {
    "user": {
      "name": "John Doe",
      "email": "john@example.com",
      "orders": [
        {
          "id": 456,
          "total": 99.99,
          "items": [
            {"name": "Widget", "price": 49.99},
            {"name": "Gadget", "price": 49.99}
          ]
        }
      ]
    }
  }
}
```

#### Operations
```graphql
# Query (Read)
query GetUser($id: ID!) {
  user(id: $id) {
    name
    email
  }
}

# Mutation (Write)
mutation CreateUser($input: CreateUserInput!) {
  createUser(input: $input) {
    id
    name
    email
  }
}

# Subscription (Real-time)
subscription OnUserUpdated($id: ID!) {
  userUpdated(id: $id) {
    id
    name
    email
  }
}
```

#### Strengths
- Eliminates over-fetching and under-fetching
- Single request for complex data
- Strong type system
- Self-documenting schema
- Real-time capabilities with subscriptions
- Excellent developer experience

#### Weaknesses
- Complexity in implementation
- Caching challenges
- Potential for expensive queries
- Learning curve
- HTTP caching limitations

### When to Choose Each Paradigm

| Scenario | Best Choice | Reason |
|----------|-------------|---------|
| Public API | REST | Simple, cacheable, widely understood |
| Internal microservices | RPC (gRPC) | Performance, type safety, code generation |
| Complex client data needs | GraphQL | Flexible queries, single endpoint |
| Real-time applications | GraphQL + WebSockets | Subscriptions + real-time updates |
| Legacy system integration | SOAP | Enterprise standards, reliability |
| Mobile applications | GraphQL | Reduces bandwidth, flexible queries |
| Simple CRUD operations | REST | Straightforward, well-established patterns |

---

## Java Spring Boot Implementation

### Spring Boot's REST Magic

Spring Boot revolutionizes REST API development through **convention over configuration** and powerful annotations that eliminate boilerplate code.

#### Key Annotations
```java
@RestController  // Combines @Controller + @ResponseBody
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        return ResponseEntity.ok(user);
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody @Valid User user) {
        User createdUser = userService.create(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(createdUser);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(
            @PathVariable Long id, 
            @RequestBody @Valid User user) {
        User updatedUser = userService.update(id, user);
        return ResponseEntity.ok(updatedUser);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

#### Auto-Configuration Benefits
Spring Boot automatically configures:
- **Jackson** for JSON serialization/deserialization
- **Embedded Tomcat** for serving HTTP requests
- **Exception handling** through `@ControllerAdvice`
- **Content negotiation** for multiple response formats
- **Validation** with JSR-303 annotations

#### Complete Example with Error Handling
```java
@RestController
@RequestMapping("/api/users")
@Validated
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping
    public ResponseEntity<Page<User>> getUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(required = false) String search) {
        
        Pageable pageable = PageRequest.of(page, size);
        Page<User> users = userService.findAll(search, pageable);
        return ResponseEntity.ok(users);
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody @Valid CreateUserRequest request) {
        User user = userService.create(request);
        URI location = ServletUriComponentsBuilder
                .fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(user.getId())
                .toUri();
        return ResponseEntity.created(location).body(user);
    }
    
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse("USER_NOT_FOUND", ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
}

@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationErrors(
            MethodArgumentNotValidException ex) {
        
        List<String> errors = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(error -> error.getField() + ": " + error.getDefaultMessage())
                .collect(Collectors.toList());
        
        ErrorResponse errorResponse = new ErrorResponse("VALIDATION_ERROR", 
                "Invalid input", errors);
        return ResponseEntity.badRequest().body(errorResponse);
    }
}
```

#### Spring Data JPA Integration
```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    @NotBlank(message = "Name is required")
    private String name;
    
    @Column(nullable = false, unique = true)
    @Email(message = "Invalid email format")
    private String email;
    
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    private LocalDateTime updatedAt;
    
    // Constructors, getters, setters
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    Page<User> findByNameContainingIgnoreCase(String name, Pageable pageable);
    
    Optional<User> findByEmail(String email);
    
    @Query("SELECT u FROM User u WHERE u.createdAt > :date")
    List<User> findRecentUsers(@Param("date") LocalDateTime date);
}

@Service
@Transactional
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    public Page<User> findAll(String search, Pageable pageable) {
        if (search != null && !search.trim().isEmpty()) {
            return userRepository.findByNameContainingIgnoreCase(search, pageable);
        }
        return userRepository.findAll(pageable);
    }
    
    public User findById(Long id) {
        return userRepository.findById(id)
                .orElseThrow(() -> new UserNotFoundException("User not found with id: " + id));
    }
    
    public User create(CreateUserRequest request) {
        if (userRepository.findByEmail(request.getEmail()).isPresent()) {
            throw new UserAlreadyExistsException("User with email already exists");
        }
        
        User user = new User();
        user.setName(request.getName());
        user.setEmail(request.getEmail());
        return userRepository.save(user);
    }
}
```

#### Configuration and Properties
```yaml
# application.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    username: sa
    password: 
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        format_sql: true
  jackson:
    default-property-inclusion: non_null
    serialization:
      write-dates-as-timestamps: false

server:
  port: 8080
  servlet:
    context-path: /api

logging:
  level:
    org.springframework.web: DEBUG
    org.hibernate.SQL: DEBUG
```

### Advanced Spring Boot Features

#### Custom Response Wrappers
```java
@RestController
public class ApiResponseController {
    
    @GetMapping("/users")
    public ApiResponse<List<User>> getUsers() {
        List<User> users = userService.findAll();
        return ApiResponse.<List<User>>builder()
                .success(true)
                .data(users)
                .message("Users retrieved successfully")
                .timestamp(LocalDateTime.now())
                .build();
    }
}

@Data
@Builder
public class ApiResponse<T> {
    private boolean success;
    private T data;
    private String message;
    private LocalDateTime timestamp;
    private List<String> errors;
}
```

#### HATEOAS Support
```java
@RestController
@RequestMapping("/api/users")
public class UserHateoasController {
    
    @GetMapping("/{id}")
    public EntityModel<User> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        
        return EntityModel.of(user)
                .add(linkTo(methodOn(UserController.class).getUser(id)).withSelfRel())
                .add(linkTo(methodOn(UserController.class).getUsers(0, 10, null)).withRel("users"))
                .add(linkTo(methodOn(OrderController.class).getUserOrders(id)).withRel("orders"));
    }
}
```

---

## Best Practices and Design Patterns

### API Design Principles

#### 1. Consistency
- Use consistent naming conventions
- Standardize response formats
- Maintain uniform error handling
- Apply consistent HTTP status codes

#### 2. Versioning Strategy
```
// URL versioning
/api/v1/users
/api/v2/users

// Header versioning
Accept: application/vnd.api+json;version=1
API-Version: 1

// Parameter versioning
/api/users?version=1
```

#### 3. Pagination
```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "size": 10,
    "total": 100,
    "totalPages": 10,
    "hasNext": true,
    "hasPrevious": false
  }
}
```

#### 4. Filtering and Sorting
```
GET /api/users?name=John&sort=createdAt,desc&page=0&size=10
GET /api/users?filter=active:true,role:admin&sort=name
```

### Design Patterns

#### Repository Pattern
```java
public interface UserRepository {
    List<User> findAll();
    Optional<User> findById(Long id);
    User save(User user);
    void deleteById(Long id);
}

@Repository
public class JpaUserRepository implements UserRepository {
    // JPA implementation
}
```

#### Service Layer Pattern
```java
@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    @Transactional
    public User createUser(CreateUserRequest request) {
        // Business logic
        User user = new User(request.getName(), request.getEmail());
        User savedUser = userRepository.save(user);
        emailService.sendWelcomeEmail(savedUser);
        return savedUser;
    }
}
```

#### DTO Pattern
```java
public class UserDTO {
    private Long id;
    private String name;
    private String email;
    // No sensitive fields like password
}

@RestController
public class UserController {
    
    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        UserDTO dto = UserMapper.toDTO(user);
        return ResponseEntity.ok(dto);
    }
}
```

### Error Handling Best Practices

#### Standardized Error Response
```java
@Data
@Builder
public class ErrorResponse {
    private String code;
    private String message;
    private List<String> details;
    private LocalDateTime timestamp;
    private String path;
}

@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
            ValidationException ex, HttpServletRequest request) {
        
        ErrorResponse error = ErrorResponse.builder()
                .code("VALIDATION_ERROR")
                .message(ex.getMessage())
                .details(ex.getValidationErrors())
                .timestamp(LocalDateTime.now())
                .path(request.getRequestURI())
                .build();
        
        return ResponseEntity.badRequest().body(error);
    }
}
```

### Performance Optimization

#### Caching Strategies
```java
@RestController
public class UserController {
    
    @GetMapping("/{id}")
    @Cacheable(value = "users", key = "#id")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        return ResponseEntity.ok()
                .cacheControl(CacheControl.maxAge(10, TimeUnit.MINUTES))
                .body(user);
    }
}
```

#### Async Processing
```java
@RestController
public class AsyncController {
    
    @PostMapping("/process")
    public ResponseEntity<String> processAsync(@RequestBody ProcessRequest request) {
        String taskId = UUID.randomUUID().toString();
        
        // Start async processing
        asyncService.process(taskId, request);
        
        return ResponseEntity.accepted()
                .header("Location", "/api/tasks/" + taskId)
                .body("Processing started");
    }
    
    @GetMapping("/tasks/{taskId}")
    public ResponseEntity<TaskStatus> getTaskStatus(@PathVariable String taskId) {
        TaskStatus status = asyncService.getStatus(taskId);
        return ResponseEntity.ok(status);
    }
}
```

---

## Security and Authentication

### Authentication Methods

#### 1. Basic Authentication
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults());
        
        return http.build();
    }
}
```

#### 2. JWT Token Authentication
```java
@RestController
public class AuthController {
    
    @PostMapping("/login")
    public ResponseEntity<LoginResponse> login(@RequestBody LoginRequest request) {
        // Validate credentials
        if (authService.validateCredentials(request.getUsername(), request.getPassword())) {
            String token = jwtService.generateToken(request.getUsername());
            return ResponseEntity.ok(new LoginResponse(token));
        }
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
    }
}

@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                   HttpServletResponse response, 
                                   FilterChain filterChain) throws ServletException, IOException {
        
        String token = extractTokenFromRequest(request);
        
        if (token != null && jwtService.validateToken(token)) {
            String username = jwtService.extractUsername(token);
            UsernamePasswordAuthenticationToken authentication = 
                new UsernamePasswordAuthenticationToken(username, null, null);
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }
        
        filterChain.doFilter(request, response);
    }
}
```

#### 3. OAuth 2.0
```java
@Configuration
@EnableOAuth2Client
public class OAuth2Config {
    
    @Bean
    public OAuth2AuthorizedClientManager authorizedClientManager(
            ClientRegistrationRepository clientRegistrationRepository,
            OAuth2AuthorizedClientRepository authorizedClientRepository) {
        
        OAuth2AuthorizedClientProvider authorizedClientProvider =
            OAuth2AuthorizedClientProviderBuilder.builder()
                .authorizationCode()
                .refreshToken()
                .build();
        
        DefaultOAuth2AuthorizedClientManager authorizedClientManager =
            new DefaultOAuth2AuthorizedClientManager(
                clientRegistrationRepository, authorizedClientRepository);
        
        authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);
        
        return authorizedClientManager;
    }
}
```

### Authorization Strategies

#### Role-Based Access Control (RBAC)
```java
@RestController
@RequestMapping("/api/admin")
@PreAuthorize("hasRole('ADMIN')")
public class AdminController {
    
    @PostMapping("/users")
    @PreAuthorize("hasAuthority('CREATE_USER')")
    public ResponseEntity<User> createUser(@RequestBody User user) {
        // Only users with CREATE_USER authority can access
        return ResponseEntity.ok(userService.create(user));
    }
}
```

#### Method-Level Security
```java
@Service
public class UserService {
    
    @PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
    public User getUserById(Long userId) {
        return userRepository.findById(userId).orElseThrow();
    }
    
    @PostAuthorize("returnObject.owner == authentication.principal.username")
    public Document getDocument(Long documentId) {
        return documentRepository.findById(documentId).orElseThrow();
    }
}
```

### Security Best Practices

#### Input Validation
```java
@RestController
public class UserController {
    
    @PostMapping("/users")
    public ResponseEntity<User> createUser(@RequestBody @Valid CreateUserRequest request) {
        // Validation annotations automatically applied
        User user = userService.create(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
}

public class CreateUserRequest {
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be between 2 and 100 characters")
    private String name;
    
    @Email(message = "Invalid email format")
    @NotBlank(message = "Email is required")
    private String email;
