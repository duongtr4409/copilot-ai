---
description: "Use when Java Backend Developer implements Spring Boot services: project structure conventions, coding standards, transaction management, exception handling, logging, test patterns, API response envelopes, and performance optimization rules."
name: "Java Backend Engineering Standards"
---

# Java Backend Engineering Standards Instruction

## Table of Contents
1. Project Structure Convention
2. Naming Conventions
3. Spring Boot Patterns
4. Transaction Management Rules
5. Exception Handling Strategy
6. Logging Standards
7. Unit Test Patterns
8. Integration Test Patterns
9. API Response Envelope Pattern
10. Performance Patterns
11. Output Quality Gate

## 1. Project Structure Convention
Follow clean architecture layering:

```
src/main/java/com/project/
├── config/              # Spring configuration, security config, WebMVC config
├── common/              # Shared utilities, constants, base classes
│   ├── exception/       # Custom exceptions and global handler
│   ├── dto/             # Shared DTOs (PageResponse, ErrorResponse, etc.)
│   └── util/            # Utility classes (DateUtil, StringUtil, etc.)
├── module/              # Feature modules (domain-driven)
│   ├── user/
│   │   ├── controller/  # REST controllers
│   │   ├── service/     # Business logic interfaces and implementations
│   │   ├── repository/  # JPA repositories
│   │   ├── entity/      # JPA entities
│   │   ├── dto/         # Request/Response DTOs
│   │   ├── mapper/      # Entity-DTO mappers
│   │   └── validator/   # Custom validators
│   └── order/
│       └── ...
└── infrastructure/      # External integrations
    ├── client/           # HTTP/REST clients
    ├── messaging/        # Message producers/consumers
    └── storage/          # File/blob storage
```

### Layer Rules
- Controllers MUST NOT contain business logic.
- Services MUST NOT import Controller or Repository from other modules directly.
- Repositories MUST NOT contain business logic (only queries).
- DTOs MUST NOT use JPA entity classes — always map between layers.
- Cross-module communication MUST go through service interfaces, not direct repository access.

## 2. Naming Conventions

### Classes
| Type | Pattern | Example |
|---|---|---|
| Controller | `[Entity]Controller` | `UserController` |
| Service Interface | `[Entity]Service` | `UserService` |
| Service Impl | `[Entity]ServiceImpl` | `UserServiceImpl` |
| Repository | `[Entity]Repository` | `UserRepository` |
| Entity | `[Entity]` (singular) | `User` |
| Request DTO | `[Action][Entity]Request` | `CreateUserRequest` |
| Response DTO | `[Entity]Response` | `UserResponse` |
| Mapper | `[Entity]Mapper` | `UserMapper` |
| Exception | `[Domain]Exception` | `UserNotFoundException` |
| Config | `[Feature]Config` | `SecurityConfig` |
| Constant | `[Domain]Constants` | `UserConstants` |

### Methods
| Type | Pattern | Example |
|---|---|---|
| GET one | `get[Entity]ById` | `getUserById(Long id)` |
| GET list | `get[Entity]List` or `search[Entity]` | `getUserList(Pageable p)` |
| POST create | `create[Entity]` | `createUser(CreateUserRequest req)` |
| PUT update | `update[Entity]` | `updateUser(Long id, UpdateUserRequest req)` |
| DELETE | `delete[Entity]` | `deleteUser(Long id)` |
| Validate | `validate[Rule]` | `validateEmailUnique(String email)` |

### Database
- Table names: `snake_case`, plural (e.g., `users`, `order_items`).
- Column names: `snake_case` (e.g., `created_at`, `first_name`).
- PK: `id` (BIGINT AUTO_INCREMENT or UUID).
- FK: `[referenced_table_singular]_id` (e.g., `user_id`).
- Index: `idx_[table]_[columns]` (e.g., `idx_users_email`).
- Unique: `uk_[table]_[columns]` (e.g., `uk_users_email`).

## 3. Spring Boot Patterns

### Dependency Injection
- ALWAYS use constructor injection (not field injection with `@Autowired`).
- Use `@RequiredArgsConstructor` (Lombok) for concise constructor injection.
- Mark dependencies as `final`.

```java
@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;
    private final UserMapper userMapper;
    // NO @Autowired on fields
}
```

### Configuration
- Use `@ConfigurationProperties` for structured config (not scattered `@Value`).
- Use profiles for environment-specific config: `application-dev.yml`, `application-prod.yml`.
- Never hardcode environment-specific values in source code.

### Validation
- Use Bean Validation annotations (`@NotNull`, `@Size`, `@Email`, etc.) on DTOs.
- Use `@Valid` or `@Validated` on controller method parameters.
- Create custom validators for complex business rules.
- Validate at the API boundary (controller), not deep in service layer.

## 4. Transaction Management Rules

### Core Rules
- Use `@Transactional` on SERVICE methods, not on controllers or repositories.
- Use `@Transactional(readOnly = true)` for read-only operations.
- Keep transactions short — avoid external HTTP calls inside transactions.
- Avoid `@Transactional` on private methods (Spring proxy does not intercept them).

### Isolation and Propagation
| Scenario | Isolation | Propagation |
|---|---|---|
| Standard CRUD | Default (READ_COMMITTED) | REQUIRED |
| Financial/critical ops | SERIALIZABLE or explicit locking | REQUIRED |
| Read-only queries | Default | REQUIRED, readOnly=true |
| Nested independent ops | Default | REQUIRES_NEW |
| Audit logging | Default | REQUIRES_NEW (never rollback audit) |

### Concurrency Safety
- Use optimistic locking (`@Version`) for entities with concurrent update risk.
- Use pessimistic locking only when optimistic locking is insufficient (high contention).
- Document locking strategy in code comments and blueprint alignment section.

## 5. Exception Handling Strategy

### Exception Hierarchy
```
RuntimeException
└── BaseException (abstract)
    ├── ResourceNotFoundException (404)
    ├── BusinessValidationException (400/422)
    ├── DuplicateResourceException (409)
    ├── UnauthorizedException (401)
    ├── ForbiddenException (403)
    └── ExternalServiceException (502/503)
```

### Global Exception Handler
```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    // Handle each exception type
    // Return standardized ErrorResponse
    // Log with appropriate level (WARN for 4xx, ERROR for 5xx)
    // Include correlation ID for tracing
}
```

### Rules
- NEVER return raw exception messages to clients.
- NEVER catch generic `Exception` and silently swallow it.
- ALWAYS log exceptions with context (user ID, request ID, operation).
- Map exceptions to appropriate HTTP status codes.
- Include error code (machine-readable) and message (human-readable) in responses.

## 6. Logging Standards

### Structured Logging
- Use SLF4J + Logback with JSON format for production.
- Use MDC (Mapped Diagnostic Context) for request context.

### MDC Fields (Mandatory)
| Field | Source | Purpose |
|---|---|---|
| `correlationId` | Request header or generated UUID | Trace requests across services |
| `userId` | Authentication context | Identify acting user |
| `requestPath` | HTTP request | Identify operation |
| `sessionId` | Session context | Group user actions |

### Log Levels
| Level | When to Use | Example |
|---|---|---|
| ERROR | System failure, data corruption, unrecoverable state | DB connection failed, transaction rollback |
| WARN | Expected error handled, degraded behavior | User not found, validation failure, retry |
| INFO | Business events, state changes, milestones | User created, order completed, payment processed |
| DEBUG | Technical details for troubleshooting | Query parameters, cache hit/miss, timing |

### Logging Rules
- NEVER log sensitive data: passwords, tokens, PII, credit card numbers.
- ALWAYS log at service entry/exit for critical operations.
- ALWAYS include correlation ID in cross-service calls.
- Log level for 4xx responses: WARN. Log level for 5xx responses: ERROR.

## 7. Unit Test Patterns

### Framework
- JUnit 5 + Mockito + AssertJ.
- Test class naming: `[ClassName]Test` (e.g., `UserServiceImplTest`).

### Test Method Naming
Pattern: `should_[ExpectedResult]_when_[Condition]`

Examples:
- `should_ReturnUser_when_ValidIdProvided`
- `should_ThrowNotFoundException_when_UserDoesNotExist`
- `should_ReturnEmptyList_when_NoCriteriaMatch`

### Test Structure (AAA Pattern)
```java
@Test
void should_CreateUser_when_ValidRequestProvided() {
    // Arrange
    CreateUserRequest request = buildValidRequest();
    when(userRepository.save(any())).thenReturn(buildUserEntity());

    // Act
    UserResponse result = userService.createUser(request);

    // Assert
    assertThat(result).isNotNull();
    assertThat(result.getEmail()).isEqualTo(request.getEmail());
    verify(userRepository).save(any(User.class));
}
```

### Coverage Rules
- Test positive path (happy path).
- Test negative path (validation failure, not found, conflict).
- Test boundary values (min, max, empty, null).
- DO NOT test getters/setters, framework behavior, or private methods directly.

## 8. Integration Test Patterns

### Framework
- `@SpringBootTest` + Testcontainers for database.
- `@WebMvcTest` for controller-only tests.
- `@DataJpaTest` for repository-only tests.

### Test Database
- Use Testcontainers (same DB engine as production) for integration tests.
- NEVER use H2 for integration tests if production uses PostgreSQL/MySQL.
- Each test class should use `@Transactional` with rollback for data isolation.

### API Integration Test Pattern
```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@Testcontainers
class UserControllerIntegrationTest {
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");

    @Autowired
    TestRestTemplate restTemplate;

    @Test
    void should_CreateAndRetrieveUser() {
        // POST create
        // GET retrieve
        // Assert data consistency
    }
}
```

## 9. API Response Envelope Pattern

### Success Response
```json
{
  "success": true,
  "data": { ... },
  "timestamp": "2026-04-15T00:00:00+07:00"
}
```

### Paginated Response
```json
{
  "success": true,
  "data": [ ... ],
  "pagination": {
    "page": 0,
    "size": 20,
    "totalElements": 150,
    "totalPages": 8
  },
  "timestamp": "2026-04-15T00:00:00+07:00"
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with ID 123 was not found.",
    "details": []
  },
  "timestamp": "2026-04-15T00:00:00+07:00"
}
```

### Validation Error Response
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Request validation failed.",
    "details": [
      { "field": "email", "message": "must be a valid email address" },
      { "field": "name", "message": "must not be blank" }
    ]
  },
  "timestamp": "2026-04-15T00:00:00+07:00"
}
```

## 10. Performance Patterns

### N+1 Prevention
- Use `@EntityGraph` or `JOIN FETCH` for eagerly needed associations.
- Use `@BatchSize` for lazy collections that are loaded in bulk.
- Default: all associations are LAZY. Never use EAGER on `@OneToMany` or `@ManyToMany`.

### Pagination
- ALWAYS paginate list endpoints (no unbounded result sets).
- Use Spring `Pageable` with default page size (e.g., 20, max 100).
- Add index support for paginated query patterns.

### Caching
- Use Spring Cache abstraction (`@Cacheable`, `@CacheEvict`).
- Cache read-heavy, write-light data (config, lookup tables, user profiles).
- Define TTL and eviction strategy per cache.
- NEVER cache sensitive or frequently changing transactional data.

### Query Optimization
- Review generated SQL with `spring.jpa.show-sql=true` in dev.
- Use DTO projections for read-only queries (avoid loading full entities).
- Use native queries only when JPQL/Criteria API cannot express the query efficiently.

### Connection Pooling
- Use HikariCP (Spring Boot default) with appropriate pool size.
- Formula: `pool_size = (core_count * 2) + effective_spindle_count`.
- Set connection timeout, idle timeout, and max lifetime.

## 11. Output Quality Gate (Mandatory)
Implementation task is NOT complete unless:

- [ ] Code follows project structure convention (Section 1).
- [ ] Naming conventions are consistent (Section 2).
- [ ] Constructor injection used throughout (Section 3).
- [ ] Transaction boundaries are correct (Section 4).
- [ ] Exception handling uses project hierarchy, no swallowed exceptions (Section 5).
- [ ] Logging follows standards with MDC context (Section 6).
- [ ] Unit tests cover positive, negative, and boundary cases (Section 7).
- [ ] Integration tests use Testcontainers with production-like DB (Section 8).
- [ ] API responses follow envelope pattern (Section 9).
- [ ] No N+1 queries, all list endpoints paginated (Section 10).
- [ ] Blueprint alignment verified (API contract, DB schema, security rules).
- [ ] `/PROJECT_STATE.md` updated with implementation status.
