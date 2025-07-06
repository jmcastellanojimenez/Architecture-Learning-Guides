# 📁 **Complete Spring Boot Project Architecture Guide**

## Java OOP Fundamentals
Interface = "I PROMISE to do these things, but I won't tell you HOW"
Implementation = "Here's HOW I actually do those things I promised"
Class = "I am the TEMPLATE/BLUEPRINT that contains the actual code and data structure"
Object = "I am the LIVING INSTANCE created from that template, with real data and running in memory"

## Annotations
Those @ symbols are annotations - they're like special instructions or labels you put on your code to tell Spring (and Java) how to handle that class or method:
- @SpringBootApplication = "Hey Spring, this is where you start the app!"
- @Controller = "Hey Spring, this handles web requests!"
- @Service = "Hey Spring, this contains business logic!"special instructions or labels you put on your code to tell Spring (and Java) how to handle that class or method.

## **Standard Spring Boot Project Layout**

```
task-management-app-lesson/
├── src/main/java/com/baeldung/
│   ├── 🚀 LsApplication.java                   # ✅ Main starter class
│   │
│   ├── 📱 controller/                          # PRESENTATION LAYER
│   │   └── ProjectController.java              # Handles HTTP requests
│   │
│   ├── 🧠 service/                             # ✅ BUSINESS LOGIC LAYER  
│   │   ├── IProjectService.java                # ✅ Interface (contract)
│   │   └── impl/ProjectServiceImpl.java        # ✅ Implementation (actual logic)
│   │
│   └── 💾 persistence/                          # ✅ PERSISTENCE LAYER
│       ├── model/Project.java                   # ✅ Data model (properties)
│       └── repository/
│           ├── IProjectRepository.java          # ✅ Interface (contract) 
│           └── impl/ProjectRepositoryImpl.java  # ✅ Implementation (database code)
│
├── src/main/resources/
│   └── application.properties                   # Configuration
│
└── pom.xml                                      # Dependencies
```

## 🔄 **How Layers Connect**

```
HTTP Request → ProjectController → ProjectService → ProjectRepository → Database
              ↓                  ↓                ↓
          Presentation       Business         Persistence
            Layer             Layer             Layer
```

## **Physical Components**

```
- Controller layer → API, Load Balancer
- Service layer → Application servers (ECS, K8s)
- Persistence layer → Database servers, but also:
	Repository classes → Database drivers, connection pools
	The actual data → PostgreSQL, MongoDB, etc.
```

---

## 📋 **Layer-by-Layer Breakdown**

### 🚀 **APPLICATION STARTER**

#### `LsApplication.java` - The Entry Point
**Purpose:** Bootstraps and starts the entire Spring Boot application
**Content:**
```java
@SpringBootApplication  // Enables auto-configuration, component scanning, configuration
public class LsApplication {
    public static void main(String[] args) {
        SpringApplication.run(LsApplication.class, args);
    }
}
```

**What it does:**
- Starts embedded Tomcat server
- Scans for components (@Controller, @Service, @Repository)
- Configures Spring context
- Loads application.properties

---

### 📱 **PRESENTATION LAYER** (Controller)
Controllers are the API-fication (love that term!) of your services.

#### `ProjectController.java` - HTTP Request Handler
**Purpose:** Receives HTTP requests and returns HTTP responses
**Content:**
```java
@RestController                    // Marks as REST API controller
@RequestMapping("/api/projects")   // Base URL path
@CrossOrigin(origins = "*")        // CORS configuration
public class ProjectController {
    
    @Autowired
    private IProjectService projectService;  // Inject service dependency
    
    @GetMapping                    // GET /api/projects
    public ResponseEntity<List<Project>> getAllProjects() {
        List<Project> projects = projectService.getAllProjects();
        return ResponseEntity.ok(projects);
    }
    
    @PostMapping                   // POST /api/projects
    public ResponseEntity<Project> createProject(@RequestBody CreateProjectRequest request) {
        Project project = projectService.createProject(request.getName(), request.getDescription());
        return ResponseEntity.status(HttpStatus.CREATED).body(project);
    }
    
    @GetMapping("/{id}")          // GET /api/projects/1
    public ResponseEntity<Project> getProjectById(@PathVariable Long id) {
        Project project = projectService.getProjectById(id);
        return ResponseEntity.ok(project);
    }
    
    @PutMapping("/{id}")          // PUT /api/projects/1
    public ResponseEntity<Project> updateProject(@PathVariable Long id, @RequestBody UpdateProjectRequest request) {
        Project project = projectService.updateProject(id, request.getName(), request.getDescription());
        return ResponseEntity.ok(project);
    }
    
    @DeleteMapping("/{id}")       // DELETE /api/projects/1
    public ResponseEntity<Void> deleteProject(@PathVariable Long id) {
        projectService.deleteProject(id);
        return ResponseEntity.noContent().build();
    }
}
```

**Key Annotations:**
- `@RestController` - Combines @Controller + @ResponseBody
- `@RequestMapping` - Maps HTTP requests to methods
- `@GetMapping/@PostMapping/@PutMapping/@DeleteMapping` - HTTP method mapping
- `@PathVariable` - Extracts URL parameters
- `@RequestBody` - Maps JSON request body to Java object
- `@Autowired` - Dependency injection

**Responsibilities:**
- Validate HTTP requests
- Call appropriate service methods
- Handle HTTP status codes
- Return JSON responses
- Handle exceptions (with @ExceptionHandler)

---

### 🧠 **BUSINESS LOGIC LAYER** (Service)
The Service layer in your Spring Boot app is NOT microservices. They're just services (lowercase 's') - internal components within a single application.
Your @Service classes are like departments in one office building (same app, direct communication), Microservices are like separate companies in different 
buildings (separate apps, network communication).

#### `IProjectService.java` - Service Contract
**Purpose:** Defines what business operations are available
**Content:**
```java
public interface IProjectService {
    // Create operations
    Project createProject(String name, String description);
    
    // Read operations
    List<Project> getAllProjects();
    Project getProjectById(Long id);
    List<Project> getProjectsByStatus(ProjectStatus status);
    
    // Update operations
    Project updateProject(Long id, String name, String description);
    Project updateProjectStatus(Long id, ProjectStatus status);
    
    // Delete operations
    void deleteProject(Long id);
    void deleteAllProjects();
    
    // Business logic operations
    List<Project> getActiveProjects();
    int countProjectsByStatus(ProjectStatus status);
    boolean isProjectNameUnique(String name);
}
```

**Benefits of Interface:**
- Loose coupling between layers
- Easy to test (can mock interface)
- Multiple implementations possible
- Clear contract definition

#### `impl/ProjectServiceImpl.java` - Business Logic Implementation
**Purpose:** Contains the actual business rules and logic
**Content:**
```java
@Service                                    // Marks as service component
@Transactional                             // Enables database transactions
public class ProjectServiceImpl implements IProjectService {
    
    @Autowired
    private IProjectRepository projectRepository;  // Inject repository
    
    @Override
    public Project createProject(String name, String description) {
        // Business validation
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Project name cannot be empty");
        }
        
        if (isProjectNameUnique(name)) {
            throw new DuplicateProjectNameException("Project name already exists");
        }
        
        // Create and save project
        Project project = new Project();
        project.setName(name.trim());
        project.setDescription(description);
        project.setStatus(ProjectStatus.ACTIVE);
        project.setCreatedAt(LocalDateTime.now());
        
        return projectRepository.save(project);
    }
    
    @Override
    public List<Project> getAllProjects() {
        return projectRepository.findAll();
    }
    
    @Override
    public Project getProjectById(Long id) {
        return projectRepository.findById(id)
            .orElseThrow(() -> new ProjectNotFoundException("Project not found with id: " + id));
    }
    
    @Override
    public Project updateProject(Long id, String name, String description) {
        Project project = getProjectById(id);
        
        // Business validation
        if (!project.getName().equals(name) && !isProjectNameUnique(name)) {
            throw new DuplicateProjectNameException("Project name already exists");
        }
        
        project.setName(name);
        project.setDescription(description);
        project.setUpdatedAt(LocalDateTime.now());
        
        return projectRepository.save(project);
    }
    
    @Override
    public void deleteProject(Long id) {
        if (!projectRepository.existsById(id)) {
            throw new ProjectNotFoundException("Project not found with id: " + id);
        }
        projectRepository.deleteById(id);
    }
    
    @Override
    public boolean isProjectNameUnique(String name) {
        return !projectRepository.existsByNameIgnoreCase(name);
    }
    
    @Override
    public List<Project> getActiveProjects() {
        return projectRepository.findByStatus(ProjectStatus.ACTIVE);
    }
}
```

**Key Annotations:**
- `@Service` - Marks as service component for Spring
- `@Transactional` - Wraps methods in database transactions
- `@Override` - Confirms interface implementation

**Responsibilities:**
- Business rule validation
- Data transformation
- Orchestrating multiple repository calls
- Transaction management
- Exception handling for business logic

---

### 💾 **PERSISTENCE LAYER** (Repository & Model)
YES, exactly! The Persistence Layer is ALL the database stuff!
Model classes are your table blueprints (@Entity = "this becomes a database table").
Repository classes are your database toolbox (@Repository = "save, find, delete, update operations"), so basically:
Model = "what data looks like" and Repository = "how to get/put that data".


#### `model/Project.java` - Data Model
**Purpose:** Represents the data structure and maps to database table
**Content:**
```java
@Entity                                    // JPA entity
@Table(name = "projects")                  // Database table name
public class Project {
    
    @Id                                    // Primary key
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // Auto-increment
    private Long id;
    
    @Column(name = "name", nullable = false, unique = true, length = 100)
    private String name;
    
    @Column(name = "description", length = 500)
    private String description;
    
    @Enumerated(EnumType.STRING)           // Store enum as string
    @Column(name = "status", nullable = false)
    private ProjectStatus status;
    
    @Column(name = "created_at", nullable = false)
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // Relationships
    @OneToMany(mappedBy = "project", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Task> tasks = new ArrayList<>();
    
    // Constructors
    public Project() {}
    
    public Project(String name, String description) {
        this.name = name;
        this.description = description;
        this.status = ProjectStatus.ACTIVE;
        this.createdAt = LocalDateTime.now();
    }
    
    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    // ... other getters and setters
    
    // Business methods
    public void addTask(Task task) {
        tasks.add(task);
        task.setProject(this);
    }
    
    public void removeTask(Task task) {
        tasks.remove(task);
        task.setProject(null);
    }
}
```

**Key Annotations:**
- `@Entity` - JPA entity that maps to database table
- `@Table` - Specifies table name and constraints
- `@Id` - Primary key field
- `@GeneratedValue` - Auto-generated values
- `@Column` - Column mapping with constraints
- `@Enumerated` - How to store enums
- `@OneToMany/@ManyToOne` - Relationships between entities

#### `repository/IProjectRepository.java` - Data Access Contract
**Purpose:** Defines how to access project data
**Content:**
```java
@Repository
public interface IProjectRepository extends JpaRepository<Project, Long> {
    
    // Spring Data JPA provides automatically:
    // - save(Project project)
    // - findById(Long id)
    // - findAll()
    // - deleteById(Long id)
    // - existsById(Long id)
    // - count()
    
    // Custom query methods
    List<Project> findByStatus(ProjectStatus status);
    List<Project> findByNameContainingIgnoreCase(String name);
    boolean existsByNameIgnoreCase(String name);
    
    // Custom JPQL queries
    @Query("SELECT p FROM Project p WHERE p.createdAt >= :startDate")
    List<Project> findProjectsCreatedAfter(@Param("startDate") LocalDateTime startDate);
    
    // Native SQL queries
    @Query(value = "SELECT * FROM projects WHERE status = ?1 ORDER BY created_at DESC", nativeQuery = true)
    List<Project> findByStatusOrderByCreatedAtDesc(String status);
    
    // Modifying queries
    @Modifying
    @Query("UPDATE Project p SET p.status = :status WHERE p.id = :id")
    int updateProjectStatus(@Param("id") Long id, @Param("status") ProjectStatus status);
}
```

#### `repository/impl/ProjectRepositoryImpl.java` - Custom Repository Implementation
**Purpose:** For complex queries that can't be expressed with method names
**Content:**
```java
@Repository
public class ProjectRepositoryImpl {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    public List<Project> findProjectsWithComplexCriteria(ProjectSearchCriteria criteria) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Project> query = cb.createQuery(Project.class);
        Root<Project> root = query.from(Project.class);
        
        List<Predicate> predicates = new ArrayList<>();
        
        if (criteria.getName() != null) {
            predicates.add(cb.like(cb.lower(root.get("name")), 
                "%" + criteria.getName().toLowerCase() + "%"));
        }
        
        if (criteria.getStatus() != null) {
            predicates.add(cb.equal(root.get("status"), criteria.getStatus()));
        }
        
        if (criteria.getStartDate() != null) {
            predicates.add(cb.greaterThanOrEqualTo(root.get("createdAt"), criteria.getStartDate()));
        }
        
        query.where(predicates.toArray(new Predicate[0]));
        query.orderBy(cb.desc(root.get("createdAt")));
        
        return entityManager.createQuery(query).getResultList();
    }
}
```

---

### ⚙️ **CONFIGURATION**

#### `application.properties` - Application Configuration
**Purpose:** Configures all aspects of the application
**Content:**
```properties
# Server Configuration
server.port=8080
server.servlet.context-path=/api

# Database Configuration
spring.datasource.url=jdbc:postgresql://localhost:5432/task_management_db
spring.datasource.username=admin
spring.datasource.password=password
spring.datasource.driver-class-name=org.postgresql.Driver

# JPA/Hibernate Configuration
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

# Logging Configuration
logging.level.com.baeldung=DEBUG
logging.level.org.springframework=INFO
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} - %msg%n

# JSON Configuration
spring.jackson.serialization.write-dates-as-timestamps=false
spring.jackson.default-property-inclusion=NON_NULL
```

#### `pom.xml` - Maven Dependencies
**Purpose:** Defines project dependencies and build configuration
**Content:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.baeldung</groupId>
    <artifactId>task-management-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>
    
    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <!-- Database -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## 🔄 **Complete Request Flow Example**

### Scenario: Creating a new project

1. **HTTP Request:** `POST /api/projects`
   ```json
   {
     "name": "New Website",
     "description": "Build company website"
   }
   ```

2. **Controller Layer:** `ProjectController.createProject()`
   - Receives HTTP request
   - Validates request format
   - Calls service layer

3. **Service Layer:** `ProjectServiceImpl.createProject()`
   - Validates business rules (name uniqueness)
   - Creates Project entity
   - Calls repository layer

4. **Repository Layer:** `IProjectRepository.save()`
   - Converts entity to SQL
   - Executes database insert
   - Returns saved entity

5. **Response Flow:** Database → Repository → Service → Controller → HTTP Response
   ```json
   {
     "id": 1,
     "name": "New Website",
     "description": "Build company website",
     "status": "ACTIVE",
     "createdAt": "2024-01-15T10:30:00"
   }
   ```

---

## 🎯 **Key Principles**

### **Separation of Concerns**
- Each layer has a single responsibility
- Changes in one layer don't affect others
- Easy to test each layer independently

### **Dependency Injection**
- Components depend on interfaces, not implementations
- Spring manages object creation and wiring
- Loose coupling between components

### **Transaction Management**
- Service layer manages database transactions
- Ensures data consistency
- Automatic rollback on exceptions

### **Error Handling**
- Each layer handles its own type of errors
- Controller: HTTP errors
- Service: Business logic errors
- Repository: Database errors

This architecture provides a solid foundation for building scalable, maintainable Spring Boot applications!
