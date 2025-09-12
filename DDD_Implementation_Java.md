## 🎯 **What We're Building**

This guide implements a **real-world airport operations system** using Domain-Driven Design principles in Java. Think of an airport's coordination complexity: flights must be scheduled, passengers assigned seats, gates managed, and everything synchronized without conflicts. Our domain models core airport concepts like `Flight` (aggregate root managing passenger assignments), `Passenger` (entity with unique identity), and `SeatAssignment` (value object describing seat details). The system enforces business rules (no duplicate seat bookings), publishes domain events (passenger added notifications), and organizes code into bounded contexts (flight operations, passenger services, ground services) - just like real airport departments. By the end, you'll have a working Spring Boot application that demonstrates how DDD transforms business complexity into clean, maintainable code.

---

## 🎯 **Project Structure**

### **Standard DDD Java Project Layout**
```
src/main/java/com/airport/
├── domain/
│   ├── model/          ← Entities, Value Objects, Aggregates
│   ├── events/         ← Domain Events
│   ├── services/       ← Domain Services
│   └── repository/     ← Repository Interfaces
├── application/
│   ├── services/       ← Application Services
│   └── dto/            ← Request/Response DTOs
├── infrastructure/
│   ├── persistence/    ← JPA/MongoDB Implementations
│   └── web/           ← REST Controllers
└── config/            ← Spring Configuration
```

---

## 🏗️ **Core Domain Implementation**

### **1. Value Objects (No Identity)**
```java
// SeatAssignment - describes something
public class SeatAssignment {
    private final String seatNumber;
    private final String seatClass; // Economy, Business
    
    public SeatAssignment(String seatNumber, String seatClass) {
        this.seatNumber = seatNumber;
        this.seatClass = seatClass;
    }
    
    @Override
    public boolean equals(Object obj) {
        // Value equality based on fields, not identity
        if (!(obj instanceof SeatAssignment)) return false;
        SeatAssignment other = (SeatAssignment) obj;
        return Objects.equals(seatNumber, other.seatNumber) &&
               Objects.equals(seatClass, other.seatClass);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(seatNumber, seatClass);
    }
}
```

### **2. Entities (With Identity)**
```java
// Passenger - has unique identity
@Entity
public class Passenger {
    @Id
    @GeneratedValue
    private Long id;                    // Identity
    private String name;
    @Embedded
    private SeatAssignment seatAssignment; // Value Object
    
    // Constructors, getters, setters
    
    @Override
    public boolean equals(Object obj) {
        // Entity equality based on ID, not fields
        if (!(obj instanceof Passenger)) return false;
        Passenger other = (Passenger) obj;
        return Objects.equals(id, other.id);
    }
}
```

### **3. Aggregate Root (Manages Consistency)**
```java
@Entity
public class Flight {
    @Id
    private String flightNumber;       // Business ID
    private String origin;
    private String destination;
    private LocalDateTime scheduledDeparture;
    private LocalDateTime scheduledArrival;
    
    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Passenger> passengers = new ArrayList<>();
    
    @Transient
    private List<Object> domainEvents = new ArrayList<>();
    
    // Business method - enforces rules
    public void addPassenger(Passenger passenger) {
        // Business rule: Check seat availability
        boolean seatTaken = passengers.stream()
            .anyMatch(p -> p.getSeatAssignment().equals(passenger.getSeatAssignment()));
            
        if (seatTaken) {
            throw new IllegalArgumentException("Seat already assigned");
        }
        
        passengers.add(passenger);
        
        // Publish domain event
        domainEvents.add(new PassengerAddedEvent(flightNumber, passenger.getId()));
    }
    
    // Spring Data JPA domain events support
    @DomainEvents
    Collection<Object> domainEvents() {
        return domainEvents;
    }
    
    @AfterDomainEventPublication
    void clearDomainEvents() {
        domainEvents.clear();
    }
}
```

---

## 📊 **Repository Pattern**

### **Interface in Domain Layer**
```java
// Domain layer - pure interface
public interface FlightRepository {
    Flight findByFlightNumber(String flightNumber);
    List<Flight> findByRoute(String origin, String destination);
    List<Flight> findByDepartureTime(LocalDateTime start, LocalDateTime end);
    void save(Flight flight);
    void delete(String flightNumber);
}
```

### **Implementation in Infrastructure Layer**
```java
// Infrastructure layer - Spring Data implementation
@Repository
public interface JpaFlightRepository extends JpaRepository<Flight, String>, FlightRepository {
    
    @Query("SELECT f FROM Flight f WHERE f.origin = :origin AND f.destination = :destination")
    List<Flight> findByRoute(@Param("origin") String origin, @Param("destination") String destination);
    
    @Query("SELECT f FROM Flight f WHERE f.scheduledDeparture BETWEEN :start AND :end")
    List<Flight> findByDepartureTime(@Param("start") LocalDateTime start, @Param("end") LocalDateTime end);
    
    Flight findByFlightNumber(String flightNumber);
}
```

---

## ⚙️ **Domain Services**

### **Business Logic That Doesn't Belong to Single Entity**
```java
@Service
public class FlightService {
    private final FlightRepository flightRepository;
    
    public FlightService(FlightRepository flightRepository) {
        this.flightRepository = flightRepository;
    }
    
    public void addPassengerToFlight(String flightNumber, Passenger passenger) {
        Flight flight = flightRepository.findByFlightNumber(flightNumber);
        
        if (flight == null) {
            throw new FlightNotFoundException("Flight not found: " + flightNumber);
        }
        
        // Delegate to aggregate root
        flight.addPassenger(passenger);
        flightRepository.save(flight);
    }
    
    public boolean isRouteAvailable(String origin, String destination, LocalDateTime date) {
        List<Flight> flights = flightRepository.findByRoute(origin, destination);
        return flights.stream()
            .anyMatch(f -> f.getScheduledDeparture().toLocalDate().equals(date.toLocalDate()));
    }
}
```

---

## 🏭 **Factory Pattern**

### **Complex Object Creation**
```java
@Component
public class FlightFactory {
    
    public Flight createFlight(String flightNumber, String origin, String destination, 
                              LocalDateTime departure, LocalDateTime arrival) {
        
        // Validation logic
        if (departure.isAfter(arrival)) {
            throw new IllegalArgumentException("Departure cannot be after arrival");
        }
        
        if (departure.isBefore(LocalDateTime.now())) {
            throw new IllegalArgumentException("Cannot schedule flights in the past");
        }
        
        // Create with business rules applied
        Flight flight = new Flight();
        flight.setFlightNumber(flightNumber);
        flight.setOrigin(origin);
        flight.setDestination(destination);
        flight.setScheduledDeparture(departure);
        flight.setScheduledArrival(arrival);
        
        return flight;
    }
}
```

---

## 🔔 **Domain Events**

### **Event Definition**
```java
public class PassengerAddedEvent {
    private final String flightNumber;
    private final Long passengerId;
    private final LocalDateTime occurredAt;
    
    public PassengerAddedEvent(String flightNumber, Long passengerId) {
        this.flightNumber = flightNumber;
        this.passengerId = passengerId;
        this.occurredAt = LocalDateTime.now();
    }
    
    // Getters...
}
```

### **Event Handler**
```java
@Component
public class PassengerEventHandler {
    
    @EventListener
    @Async
    public void handlePassengerAdded(PassengerAddedEvent event) {
        // React to business event
        System.out.println("Passenger " + event.getPassengerId() + 
                          " added to flight " + event.getFlightNumber());
        
        // Could trigger: manifest updates, notifications, etc.
    }
}
```

---

## 🌐 **Application Layer**

### **REST Controller (Thin Layer)**
```java
@RestController
@RequestMapping("/api/flights")
public class FlightController {
    private final FlightService flightService;
    private final FlightFactory flightFactory;
    private final FlightRepository flightRepository;
    
    @PostMapping
    public ResponseEntity<Flight> createFlight(@RequestBody FlightRequest request) {
        Flight flight = flightFactory.createFlight(
            request.getFlightNumber(),
            request.getOrigin(),
            request.getDestination(),
            request.getScheduledDeparture(),
            request.getScheduledArrival()
        );
        
        flightRepository.save(flight);
        return ResponseEntity.ok(flight);
    }
    
    @PostMapping("/{flightNumber}/passengers")
    public ResponseEntity<String> addPassenger(@PathVariable String flightNumber, 
                                              @RequestBody PassengerRequest request) {
        try {
            Passenger passenger = new Passenger();
            passenger.setName(request.getName());
            passenger.setSeatAssignment(new SeatAssignment(
                request.getSeatNumber(), 
                request.getSeatClass()
            ));
            
            flightService.addPassengerToFlight(flightNumber, passenger);
            return ResponseEntity.ok("Passenger added successfully");
            
        } catch (IllegalArgumentException e) {
            return ResponseEntity.status(HttpStatus.CONFLICT).body(e.getMessage());
        }
    }
    
    @GetMapping("/{flightNumber}")
    public ResponseEntity<Flight> getFlight(@PathVariable String flightNumber) {
        Flight flight = flightRepository.findByFlightNumber(flightNumber);
        return flight != null ? ResponseEntity.ok(flight) : ResponseEntity.notFound().build();
    }
}
```

### **DTOs (Data Transfer Objects)**
```java
public class FlightRequest {
    private String flightNumber;
    private String origin;
    private String destination;
    private LocalDateTime scheduledDeparture;
    private LocalDateTime scheduledArrival;
    
    // Getters, setters, validation annotations
}

public class PassengerRequest {
    private String name;
    private String seatNumber;
    private String seatClass;
    
    // Getters, setters
}
```

---

## 📦 **Bounded Contexts**

### **Package Organization**
```
com.airport.flightops/          ← Flight Operations Context
├── domain/
├── application/
└── infrastructure/

com.airport.passengerservices/  ← Passenger Services Context
├── domain/
├── application/
└── infrastructure/

com.airport.groundservices/     ← Ground Services Context
├── domain/
├── application/
└── infrastructure/
```

### **Context Integration**
```java
// Flight Operations Context publishes events
@EventListener
public void handleFlightDelayed(FlightDelayedEvent event) {
    // Passenger Services Context reacts
    notificationService.sendDelayNotifications(event.getFlightNumber());
}
```

---

## 🧪 **Testing DDD Components**

### **Domain Logic Tests**
```java
@SpringBootTest
class FlightTest {
    
    @Test
    void shouldAddPassengerWithAvailableSeat() {
        // Given
        Flight flight = new Flight("UA101", "JFK", "LAX");
        Passenger passenger = new Passenger("John Doe", new SeatAssignment("12A", "Economy"));
        
        // When
        flight.addPassenger(passenger);
        
        // Then
        assertThat(flight.getPassengers()).hasSize(1);
        assertThat(flight.getPassengers().get(0).getName()).isEqualTo("John Doe");
    }
    
    @Test
    void shouldThrowExceptionForDuplicateSeat() {
        // Given
        Flight flight = new Flight("UA101", "JFK", "LAX");
        Passenger passenger1 = new Passenger("John", new SeatAssignment("12A", "Economy"));
        Passenger passenger2 = new Passenger("Jane", new SeatAssignment("12A", "Economy"));
        
        flight.addPassenger(passenger1);
        
        // When/Then
        assertThatThrownBy(() -> flight.addPassenger(passenger2))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessage("Seat already assigned");
    }
}
```

### **Integration Tests**
```java
@SpringBootTest
@TestMethodOrder(OrderAnnotation.class)
class FlightIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    @Order(1)
    void shouldCreateFlight() {
        FlightRequest request = new FlightRequest("UA101", "JFK", "LAX", /* dates */);
        
        ResponseEntity<Flight> response = restTemplate.postForEntity("/api/flights", request, Flight.class);
        
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody().getFlightNumber()).isEqualTo("UA101");
    }
}
```

---

## 🚀 **Spring Boot Configuration**

### **Application Properties**
```yaml
# application.yml
spring:
  datasource:
    url: jdbc:h2:mem:airport
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
  h2:
    console:
      enabled: true

logging:
  level:
    com.airport: DEBUG
```

### **Main Application Class**
```java
@SpringBootApplication
@EnableJpaRepositories
@EnableAsync
public class AirportApplication {
    public static void main(String[] args) {
        SpringApplication.run(AirportApplication.class, args);
    }
}
```

---

## ⚠️ **Common Pitfalls & Solutions**

### **❌ DON'T - Anemic Domain Model**
```java
// Bad - no business logic
public class Flight {
    private String flightNumber;
    private List<Passenger> passengers;
    // Only getters/setters
}

// Service does everything
public class FlightService {
    public void addPassenger(Flight flight, Passenger passenger) {
        // All business logic in service
    }
}
```

### **✅ DO - Rich Domain Model**
```java
// Good - business logic in domain
public class Flight {
    public void addPassenger(Passenger passenger) {
        validateSeatAvailable(passenger.getSeatAssignment());
        passengers.add(passenger);
        publishEvent(new PassengerAddedEvent(this.flightNumber, passenger.getId()));
    }
    
    private void validateSeatAvailable(SeatAssignment seat) {
        // Business rules here
    }
}
```

### **❌ DON'T - Repository in Domain**
```java
// Bad - infrastructure dependency in domain
public class Flight {
    public void addPassenger(Passenger passenger, FlightRepository repo) {
        passengers.add(passenger);
        repo.save(this); // Wrong layer!
    }
}
```

### **✅ DO - Keep Domain Pure**
```java
// Good - domain only contains business logic
public class Flight {
    public void addPassenger(Passenger passenger) {
        // Pure business logic only
        passengers.add(passenger);
    }
}

// Application service handles persistence
@Service
public class FlightService {
    public void addPassengerToFlight(String flightNumber, Passenger passenger) {
        Flight flight = flightRepository.findByFlightNumber(flightNumber);
        flight.addPassenger(passenger);
        flightRepository.save(flight); // Persistence here
    }
}
```

---

## 💡 **TL;DR**

**Java DDD Implementation**:
- **Entities**: Have identity, use ID for equality
- **Value Objects**: No identity, use value equality  
- **Aggregates**: Consistency boundaries with business methods
- **Repositories**: Interfaces in domain, implementations in infrastructure
- **Domain Services**: Business logic spanning multiple entities
- **Factories**: Complex object creation with validation
- **Domain Events**: Publish business occurrences
- **Bounded Contexts**: Separate packages/modules per business area

**Spring Boot Integration**: Use JPA for persistence, Spring Events for domain events, standard REST controllers for API

**Key Success Factors**: Rich domain models, pure domain layer, clear boundaries, ubiquitous language in code

---

*Make your Java code speak the language of your business, not your database*
