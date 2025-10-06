# Spring Boot Common Service Requirements

## Project Overview
Establish foundational requirements and architectural standards that apply to any Spring Boot microservice development. These common requirements ensure consistency, maintainability, and best practices across all Spring Boot services within the organization.

## Common Spring Boot Dependencies

<!-- Core Spring Boot Starters -->
- spring-boot-starter-web (REST API development)
- spring-boot-starter-data-jpa (Database operations)
- spring-boot-starter-validation (Input validation)
- spring-boot-starter-actuator (Health checks and monitoring)

<!-- Database -->
- h2 (Development and testing database)
- mysql-connector-j (Production MySQL database)

<!-- Testing -->
- spring-boot-starter-test (Comprehensive testing framework)
- testcontainers (Integration testing with real databases)

<!-- Documentation -->
- springdoc-openapi-starter-webmvc-ui (OpenAPI/Swagger documentation)

<!-- Development Tools -->
- spring-boot-devtools (Development productivity)
- lombok (Code generation and boilerplate reduction)

<!-- Monitoring and Observability -->
- micrometer-registry-prometheus (Metrics collection)
- zipkin-brave (Distributed tracing)

## Standard Spring Boot Architecture

### 1. Controller Layer

#### A. REST Controller Standards
• Base URL Structure: `/api/{service-name}/{version}`
• HTTP Method Usage: GET (retrieval), POST (creation), PUT (updates), DELETE (removal)
• Response Standards: Consistent JSON structure with proper HTTP status codes
• Error Handling: Centralized exception handling with @ControllerAdvice
• Validation: Input validation using @Valid and custom validators
• API Documentation: Complete OpenAPI/Swagger documentation for all endpoints
• CORS Configuration: Proper Cross-Origin Resource Sharing setup
• Rate Limiting: Request throttling for API protection

#### B. Controller Best Practices
• Slim controllers with business logic delegated to service layer
• Consistent request/response DTO usage
• Proper HTTP status code usage (200, 201, 400, 401, 403, 404, 500)
• Request logging and monitoring integration
• Pagination support for list endpoints
• Filtering and sorting capabilities
• Content negotiation support (JSON, XML)

### 2. Service Layer

#### A. Business Logic Standards
• Service Interface Pattern: Define interfaces for all business services
• Transaction Management: Proper @Transactional annotation usage
• Exception Handling: Business-specific exceptions with meaningful messages
• Logging: Comprehensive logging with appropriate levels (DEBUG, INFO, WARN, ERROR)
• Validation: Business rule validation and enforcement
• Caching: Strategic caching implementation with @Cacheable
• Async Processing: Non-blocking operations with @Async where appropriate

#### B. Service Layer Best Practices
• Single Responsibility Principle adherence
• Dependency injection with constructor-based injection preferred
• Stateless service design for scalability
• Proper error propagation and handling
• Business metrics and monitoring integration
• Configuration externalization
• Service-to-service communication standards

### 3. Repository Layer

#### A. Data Access Standards
• JPA Repository Pattern: Extend JpaRepository for standard CRUD operations
• Custom Queries: @Query annotations for complex database operations
• Named Queries: Externalized queries for better maintainability
• Pagination: Built-in pagination and sorting support
• Auditing: Automatic audit fields (createdAt, updatedAt, createdBy, updatedBy)
• Soft Deletes: Logical deletion with @Where annotations
• Database Migrations: Flyway or Liquibase for version control

#### B. Repository Best Practices
• Repository interfaces for better testability
• Query optimization and performance monitoring
• Database connection pooling configuration
• Proper indexing strategy
• Transaction isolation level configuration
• Lazy loading optimization
• Bulk operations for performance

### 4. Entity/Model Layer

#### A. JPA Entity Standards
• Entity Mapping: Proper @Entity, @Table, @Column annotations
• Primary Keys: Auto-generated IDs with @GeneratedValue
• Relationships: Proper @OneToMany, @ManyToOne, @ManyToMany mappings
• Validation: Bean validation annotations (@NotNull, @Size, @Email, etc.)
• Audit Fields: Common audit fields in all entities
• Serialization: JSON serialization configuration with @JsonIgnore, @JsonProperty
• Equality: Proper equals() and hashCode() implementation

#### B. Entity Best Practices
• Entity classes should be immutable where possible
• Use of @Embedded for value objects
• Proper cascade and fetch type configuration
• Entity lifecycle callbacks (@PrePersist, @PreUpdate)
• Version control with @Version for optimistic locking
• Custom entity listeners for cross-cutting concerns
• Database schema validation

## Common Configuration Standards

### A. Application Properties Structure

```properties
# Server Configuration
server.port=8080
server.servlet.context-path=/
server.error.include-message=always
server.error.include-binding-errors=always

# Database Configuration
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# JPA Configuration
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect

# H2 Console (Development only)
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# Validation Configuration
spring.mvc.throw-exception-if-no-handler-found=true
spring.web.resources.add-mappings=false

# Actuator Configuration
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.endpoint.health.show-details=when-authorized
management.health.db.enabled=true

# Logging Configuration
logging.level.com.wipro.ai.demo=${LOG_LEVEL:INFO}
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE

# OpenAPI Documentation
springdoc.api-docs.path=/api-docs
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.swagger-ui.operationsSorter=method
```

### B. Profile-Specific Configuration

#### Development Profile (application-dev.properties)
```properties
# Development Database
spring.datasource.url=jdbc:h2:mem:devdb
spring.h2.console.enabled=true

# Development Logging
logging.level.com.wipro.ai.demo=DEBUG
logging.level.root=INFO

# Development Security
security.debug=true
```

#### Production Profile (application-prod.properties)
```properties
# Production Database
spring.datasource.url=jdbc:mysql://localhost:3306/production_db
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false

# Production Security
server.ssl.enabled=true
server.ssl.key-store=${SSL_KEYSTORE_PATH}
server.ssl.key-store-password=${SSL_KEYSTORE_PASSWORD}

# Production Logging
logging.level.com.wipro.ai.demo=INFO
logging.level.root=WARN

# Production Monitoring
management.endpoints.web.exposure.include=health,metrics,prometheus
management.endpoint.health.show-details=never
```

### C. Common Bean Configuration

#### A. WebConfig.java
```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("http://localhost:3000", "http://localhost:4200")
            .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
            .allowedHeaders("*")
            .allowCredentials(true);
    }
    
    @Bean
    public ModelMapper modelMapper() {
        return new ModelMapper();
    }
}
```

#### B. DatabaseConfig.java
```java
@Configuration
@EnableJpaRepositories
@EnableJpaAuditing
public class DatabaseConfig {
    
    @Bean
    public AuditorAware<String> auditorProvider() {
        return new AuditorAwareImpl();
    }
    
    @Bean
    @Profile("prod")
    public DataSource productionDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(env.getProperty("spring.datasource.url"));
        config.setUsername(env.getProperty("spring.datasource.username"));
        config.setPassword(env.getProperty("spring.datasource.password"));
        config.setMaximumPoolSize(20);
        return new HikariDataSource(config);
    }
}
```

## Common Exception Handling

### A. Global Exception Handler
```java
@ControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ValidationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ResponseEntity<ErrorResponse> handleValidationException(ValidationException ex) {
        log.error("Validation error: {}", ex.getMessage(), ex);
        return ResponseEntity.badRequest().body(
            ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpStatus.BAD_REQUEST.value())
                .error("Validation Failed")
                .message(ex.getMessage())
                .build()
        );
    }
    
    @ExceptionHandler(EntityNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ResponseEntity<ErrorResponse> handleEntityNotFoundException(EntityNotFoundException ex) {
        log.error("Entity not found: {}", ex.getMessage(), ex);
        return ResponseEntity.notFound().build();
    }
    
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        log.error("Unexpected error: {}", ex.getMessage(), ex);
        return ResponseEntity.internalServerError().body(
            ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpStatus.INTERNAL_SERVER_ERROR.value())
                .error("Internal Server Error")
                .message("An unexpected error occurred")
                .build()
        );
    }
}
```

### B. Standard Error Response
```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ErrorResponse {
    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private String path;
    private Map<String, String> validationErrors;
}
```

## Common Testing Standards

### A. Unit Testing Framework
```java
@ExtendWith(MockitoExtension.class)
class ServiceTest {
    
    @Mock
    private Repository repository;
    
    @InjectMocks
    private ServiceImpl service;
    
    @Test
    void shouldReturnEntityWhenValidIdProvided() {
        // Given
        Long id = 1L;
        Entity entity = new Entity();
        when(repository.findById(id)).thenReturn(Optional.of(entity));
        
        // When
        Entity result = service.findById(id);
        
        // Then
        assertThat(result).isNotNull();
        verify(repository).findById(id);
    }
}
```

### B. Integration Testing Framework
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class IntegrationTest {
    
    @Container
    static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.0")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    void shouldCreateEntitySuccessfully() {
        // Given
        CreateRequest request = new CreateRequest("test");
        
        // When
        ResponseEntity<CreateResponse> response = restTemplate.postForEntity(
            "/api/entities", request, CreateResponse.class);
        
        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody().getName()).isEqualTo("test");
    }
}
```

### C. Repository Testing Framework
```java
@DataJpaTest
class RepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private EntityRepository repository;
    
    @Test
    void shouldFindEntityByName() {
        // Given
        Entity entity = new Entity("test");
        entityManager.persistAndFlush(entity);
        
        // When
        Optional<Entity> found = repository.findByName("test");
        
        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getName()).isEqualTo("test");
    }
}
```

## Test Case Generation Standards

### A. Automated Test Coverage Requirements

#### Mandatory Test Coverage:
- **Unit Tests**: Minimum 80% code coverage for service and utility classes
- **Integration Tests**: Complete API endpoint coverage with positive and negative scenarios
- **Repository Tests**: Database operation validation for all custom queries
- **Controller Tests**: HTTP request/response validation for all REST endpoints
- **Security Tests**: Authentication and authorization validation
- **Performance Tests**: Load testing for critical business operations

#### Test Case Generation Framework:
```java
@ExtendWith(MockitoExtension.class)
class AutoGeneratedServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @ParameterizedTest
    @ValueSource(strings = {"", " ", "invalid-email"})
    void shouldThrowValidationException_WhenEmailIsInvalid(String email) {
        // Given
        UserRegistrationRequest request = UserRegistrationRequest.builder()
            .email(email)
            .build();
        
        // When & Then
        assertThrows(ValidationException.class, 
            () -> userService.createUser(request));
    }
    
    @Test
    void shouldCreateUser_WhenValidDataProvided() {
        // Given
        UserRegistrationRequest request = createValidUserRequest();
        User expectedUser = createUserEntity();
        when(userRepository.save(any(User.class))).thenReturn(expectedUser);
        
        // When
        UserRegistrationResponse response = userService.createUser(request);
        
        // Then
        assertThat(response).isNotNull();
        assertThat(response.getUsername()).isEqualTo(expectedUser.getUsername());
        verify(userRepository).save(any(User.class));
    }
}
```

### B. API Test Automation Standards

#### RestAssured Integration Testing:
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@TestMethodOrder(OrderAnnotation.class)
class ApiIntegrationTest {
    
    @LocalServerPort
    private int port;
    
    @Test
    @Order(1)
    void shouldRegisterNewUser() {
        UserRegistrationRequest request = UserRegistrationRequest.builder()
            .username("testuser")
            .email("test@example.com")
            .password("password123")
            .firstName("Test")
            .lastName("User")
            .build();
        
        given()
            .port(port)
            .contentType(ContentType.JSON)
            .body(request)
        .when()
            .post("/api/v1/users/register")
        .then()
            .statusCode(201)
            .body("username", equalTo("testuser"))
            .body("email", equalTo("test@example.com"));
    }
    
    @Test
    @Order(2)
    void shouldGetUserById() {
        given()
            .port(port)
            .header("X-User-ID", "1")
            .header("X-User-Roles", "USER")
        .when()
            .get("/api/v1/users/1")
        .then()
            .statusCode(200)
            .body("id", notNullValue())
            .body("username", notNullValue());
    }
}
```

## Postman Collection Generation Standards

### A. Automated Postman Script Generation

#### Collection Structure Requirements:
- **Environment Variables**: Separate dev, test, and prod environments
- **Authentication Setup**: Pre-request scripts for header-based auth
- **Test Scripts**: Automated response validation in each request
- **Data Management**: Dynamic test data generation and cleanup
- **Error Scenarios**: Negative test cases for error handling validation

#### Sample Postman Collection Structure:
```json
{
  "info": {
    "name": "User Management Service API",
    "description": "Complete API testing collection for User Management Service",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "auth": {
    "type": "noauth"
  },
  "event": [
    {
      "listen": "prerequest",
      "script": {
        "exec": [
          "// Set dynamic headers for user authentication",
          "pm.request.headers.add({key: 'X-User-ID', value: pm.environment.get('current_user_id')});",
          "pm.request.headers.add({key: 'X-User-Roles', value: pm.environment.get('user_roles')});"
        ]
      }
    }
  ],
  "variable": [
    {
      "key": "baseUrl",
      "value": "{{base_url}}/api/v1"
    }
  ]
}
```

### B. Postman Test Scripts Standards

#### Standard Test Script Template:
```javascript
// Response validation template
pm.test("Status code validation", function () {
    pm.expect(pm.response.code).to.be.oneOf([200, 201, 204]);
});

pm.test("Response time validation", function () {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});

pm.test("Response structure validation", function () {
    const responseJson = pm.response.json();
    pm.expect(responseJson).to.have.property('id');
    pm.expect(responseJson.id).to.be.a('number');
});

pm.test("Business logic validation", function () {
    const responseJson = pm.response.json();
    pm.expect(responseJson.email).to.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/);
});

// Dynamic data extraction for chaining requests
if (pm.response.code === 201) {
    const responseJson = pm.response.json();
    pm.environment.set("created_user_id", responseJson.id);
    pm.environment.set("created_username", responseJson.username);
}
```

### C. Environment Configuration Standards

#### Development Environment:
```json
{
  "name": "Development",
  "values": [
    {
      "key": "base_url",
      "value": "http://localhost:8080",
      "enabled": true
    },
    {
      "key": "current_user_id",
      "value": "1",
      "enabled": true
    },
    {
      "key": "user_roles",
      "value": "USER,ADMIN",
      "enabled": true
    }
  ]
}
```

#### Production Environment:
```json
{
  "name": "Production",
  "values": [
    {
      "key": "base_url",
      "value": "https://api.production.com",
      "enabled": true
    },
    {
      "key": "current_user_id",
      "value": "{{$randomInt}}",
      "enabled": true
    },
    {
      "key": "user_roles",
      "value": "USER",
      "enabled": true
    }
  ]
}
```

## Common Monitoring and Observability

### A. Health Check Configuration
```java
@Component
public class CustomHealthIndicator implements HealthIndicator {
    
    @Override
    public Health health() {
        // Custom health check logic
        boolean healthy = checkExternalService();
        
        if (healthy) {
            return Health.up()
                .withDetail("service", "Available")
                .withDetail("timestamp", LocalDateTime.now())
                .build();
        } else {
            return Health.down()
                .withDetail("service", "Unavailable")
                .withDetail("error", "External service connection failed")
                .build();
        }
    }
}
```

### B. Metrics Configuration
```java
@Component
public class CustomMetrics {
    
    private final Counter requestCounter;
    private final Timer requestTimer;
    
    public CustomMetrics(MeterRegistry meterRegistry) {
        this.requestCounter = Counter.builder("api.requests.total")
            .description("Total API requests")
            .register(meterRegistry);
            
        this.requestTimer = Timer.builder("api.requests.duration")
            .description("API request duration")
            .register(meterRegistry);
    }
    
    public void incrementRequestCounter() {
        requestCounter.increment();
    }
    
    public Timer.Sample startTimer() {
        return Timer.start(requestTimer);
    }
}
```

## Common Security Standards

### A. Basic Security Configuration
```java
@Configuration
@EnableWebSecurity
public class BasicSecurityConfig {
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers("/swagger-ui/**", "/api-docs/**").permitAll()
                .anyRequest().authenticated()
            );
        
        return http.build();
    }
}
```

## Common Logging Standards

### A. Structured Logging Configuration
```properties
# Logback configuration in logback-spring.xml
logging.pattern.console=%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
logging.file.name=logs/application.log
logging.file.max-size=10MB
logging.file.max-history=30
```

### B. Application Logging Standards
```java
@Slf4j
@Service
public class ExampleService {
    
    public void performOperation(String input) {
        log.info("Starting operation with input: {}", input);
        
        try {
            // Business logic
            log.debug("Processing step completed successfully");
            
        } catch (Exception e) {
            log.error("Operation failed for input: {}", input, e);
            throw new ServiceException("Operation failed", e);
        }
        
        log.info("Operation completed successfully for input: {}", input);
    }
}
```
## Git Repository Management Process

### A. Repository Setup Standards

#### Initial Repository Setup
```bash
# 1. Initialize Git repository
git init
git remote add origin https://github.com/{username}/{service-name}.git
git branch -M main

# 2. Create initial commit with standard structure
git add .
git commit -m "feat: initial Spring Boot service setup"
git push -u origin main
```

#### Required Repository Structure
```
service-root/
├── .github/
│   ├── workflows/          # GitHub Actions CI/CD
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── copilot-instructions.md
├── src/main/java/          # Application source code
├── src/test/java/          # Test source code
├── .gitignore             # Git exclusions
├── README.md              # Service documentation
├── pom.xml                # Maven configuration
└── CHANGELOG.md           # Version history
```

### B. Branching Strategy (GitFlow)

#### Branch Structure
```
main                # Production-ready code
├── develop        # Integration branch
├── feature/*      # Feature development
├── release/*      # Release preparation  
└── hotfix/*       # Critical fixes
```

#### Branch Naming Conventions
```bash
# Feature branches
feature/user-authentication
feature/JIRA-123-api-endpoints

# Bug fixes  
bugfix/password-validation-fix

# Release branches
release/v1.2.0

# Hotfix branches
hotfix/security-patch
```

### C. Commit Message Standards

#### Conventional Commit Format
```
<type>(<scope>): <description>

Types: feat, fix, docs, style, refactor, test, chore
Examples:
feat(auth): add JWT token validation
fix(user): resolve email uniqueness issue
docs: update API documentation
chore(deps): update Spring Boot to 3.2.0
```

### D. Development Workflow

#### Daily Development Process
```bash
# 1. Start with latest code
git checkout main && git pull origin main
git checkout develop && git pull origin develop

# 2. Create feature branch
git checkout -b feature/new-feature develop

# 3. Development and commits
git add . && git commit -m "feat: implement new feature"
git push -u origin feature/new-feature

# 4. Create Pull Request (GitHub UI)
# 5. After approval, merge and cleanup
git checkout develop && git pull origin develop
git branch -d feature/new-feature
```

#### Release Process
```bash
# 1. Create release branch
git checkout -b release/v1.0.0 develop

# 2. Update version and documentation
# Edit pom.xml, CHANGELOG.md
git commit -m "chore: prepare release v1.0.0"

# 3. Merge to main and tag
git checkout main && git merge --no-ff release/v1.0.0
git tag -a v1.0.0 -m "Release v1.0.0"

# 4. Merge back to develop and push
git checkout develop && git merge --no-ff release/v1.0.0
git push origin main develop --tags
```

### E. Code Quality Gates

#### Pre-commit Checks (Required)
- ✅ All tests pass (`mvn test`)
- ✅ Code formatting correct (`mvn spring-javaformat:validate`)
- ✅ Static analysis clean (`mvn checkstyle:check`)
- ✅ No security vulnerabilities detected

#### Pull Request Requirements
- ✅ Descriptive title and description
- ✅ Linked to relevant issues
- ✅ Code review by at least one developer
- ✅ All CI checks pass
- ✅ Documentation updated (if needed)

### F. Repository Security

#### Essential Security Configuration
```bash
# Set up user identity
git config user.name "Your Name"
git config user.email "your.email@company.com"

# Enable credential caching
git config credential.helper cache

# Set up SSH keys (recommended)
ssh-keygen -t ed25519 -C "your.email@company.com"
```

#### Enhanced .gitignore Configuration
```gitignore
# Compiled class files
*.class
target/
*.jar
*.war
*.nar
*.ear

# Log files
*.log
logs/
/logs

# IDE files
.idea/
.vscode/
*.iml
*.iws
*.ipr
.settings/
.project
.classpath

# OS files
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# Environment variables
.env
.env.*
!.env.example
.envrc

# Database files
*.db
*.sqlite
*.sqlite3
h2/

# Security files
*.pem
*.key
*.p12
*.jks
application-prod.properties
application-prod.yml

# Build tools
.mvn/wrapper/maven-wrapper.jar
.gradle/
build/
gradle-app.setting

# Docker
*.tar
*.tar.gz

# Backup files
*.bak
*.backup
*.tmp
*~

# Spring Boot
spring.log
/tmp

# Test results
/test-output/
/test-results/

# Coverage reports
.nyc_output
coverage/
*.lcov

# Dependency directories
node_modules/

# Package files
*.tgz
*.tar.gz

# Runtime data
pids
*.pid
*.seed
*.pid.lock

# Optional npm cache directory
.npm

# Optional eslint cache
.eslintcache

# Temporary folders
tmp/
temp/
### G. CI/CD Integration

#### GitHub Actions Workflow (Required)
```yaml
# .github/workflows/ci.yml
name: CI Pipeline
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-java@v4
      with:
        java-version: '17'
    - run: mvn clean test
    - run: mvn clean package -DskipTests
```

### H. Repository Maintenance

#### Weekly Maintenance Tasks
```bash
# Clean up merged branches
git branch --merged | grep -v "main\|develop" | xargs git branch -d

# Update dependencies
mvn versions:display-dependency-updates

# Run security audit
mvn dependency:analyze
```

#### Release Checklist
- [ ] All tests pass
- [ ] Version numbers updated
- [ ] CHANGELOG.md updated
- [ ] Documentation current
- [ ] Security scan completed
- [ ] Performance tested
- [ ] Tagged release created

## Expected Common Deliverables

1. Project Structure following Spring Boot best practices
2. Common Dependencies configured and optimized
3. Layered Architecture with proper separation of concerns
4. Configuration Management with profile-specific properties
5. Exception Handling with global error handling
6. Testing Framework with unit, integration, and repository tests
7. Monitoring Setup with health checks and metrics
8. Security Foundation with basic security configuration
9. Logging Framework with structured logging
10. Documentation with OpenAPI/Swagger integration
11. **Comprehensive Test Suite** with automated test case generation covering all API endpoints
12. **API Testing Collection** with Postman scripts for manual and automated API testing

## Success Criteria

• ✅ Standard Spring Boot project structure established
• ✅ Common dependencies configured and working
• ✅ Layered architecture implemented correctly
• ✅ Configuration profiles working (dev, test, prod)
• ✅ Global exception handling implemented
• ✅ Testing framework setup and functional
• ✅ Health checks and monitoring operational
• ✅ Basic security configuration applied
• ✅ Structured logging implemented
• ✅ API documentation generated and accessible
• ✅ Database connectivity and JPA working
• ✅ Build and deployment process functional
• ✅ **Comprehensive test suite with 80%+ code coverage implemented**
• ✅ **Automated test cases covering all positive and negative scenarios**
• ✅ **Postman collection with complete API endpoint coverage generated**
• ✅ **Environment-specific Postman configurations created**
• ✅ **API test automation scripts functional and validated**
• ✅ **Test data management and cleanup processes established**
