---
name: test-automation-scaffolding
description: "Generate test scaffolds from acceptance criteria and user stories: JUnit 5 for Java, Jest for React, Jasmine/Karma for Angular, and Cypress/Playwright for E2E. Use for accelerating test coverage, ensuring AC traceability, and bootstrapping test suites."
argument-hint: "Acceptance criteria, user stories, tech stack (Java/Angular/React), test framework, and coverage goals"
user-invocable: true
---

# Test Automation Scaffolding (Khoi tao Khung Kiem thu Tu dong)

## What This Skill Produces
- Ready-to-run test files generated from acceptance criteria and user stories.
- AC-to-test traceability mapping for every generated test.
- Test structure following project conventions and framework best practices.
- Coverage gap analysis showing which AC lack test coverage.
- Test data fixtures and mock configurations for generated tests.

## When To Use
- Starting a new feature and need test scaffolds before or during implementation.
- BA delivers acceptance criteria and QA needs test cases quickly.
- Existing code lacks test coverage and needs bootstrap test suite.
- Refactoring code and need safety-net tests before changes.
- Sprint planning where QA estimates depend on test generation effort.

## Required Inputs
- Acceptance criteria (AC) or user stories with clear behavioral expectations.
- Target tech stack and test framework.
- Source code context (service/component being tested, if existing).
- Project conventions: naming style, directory structure, assertion library.
- Coverage goals: which AC to prioritize, which test types needed.

## Supported Frameworks

### Backend (Java)
| Component | Framework | Convention |
|---|---|---|
| Unit Tests | JUnit 5 + Mockito + AssertJ | `src/test/java/.../[Class]Test.java` |
| Integration Tests | Spring Boot Test + Testcontainers | `src/test/java/.../[Class]IntegrationTest.java` |
| API Tests | REST Assured | `src/test/java/.../[Endpoint]ApiTest.java` |

### Frontend (React)
| Component | Framework | Convention |
|---|---|---|
| Unit Tests | Jest + Testing Library | `__tests__/[Component].test.tsx` |
| Hook Tests | Jest + renderHook | `__tests__/use[Hook].test.ts` |
| E2E Tests | Cypress or Playwright | `cypress/e2e/[feature].cy.ts` or `e2e/[feature].spec.ts` |

### Frontend (Angular)
| Component | Framework | Convention |
|---|---|---|
| Unit Tests | Jasmine + Karma | `[component].spec.ts` (co-located) |
| Service Tests | Jasmine + HttpClientTestingModule | `[service].spec.ts` |
| E2E Tests | Cypress or Playwright | `cypress/e2e/[feature].cy.ts` or `e2e/[feature].spec.ts` |

## Standard Workflow

### Step 1: Parse Acceptance Criteria
1. Extract atomic behaviors from each AC (one assertion per behavior).
2. Assign test IDs: `TC-<feature>-<number>`.
3. Classify each test: unit | integration | e2e.
4. Map priority: P0 (critical path), P1 (important), P2 (edge case), P3 (nice-to-have).

### Step 2: Generate Test Structure
1. Create test file with proper imports and setup.
2. Generate `describe` / `@Nested` blocks organized by feature or component.
3. Generate test method stubs with descriptive names.
4. Add AC reference as comment or annotation.
5. Include Arrange-Act-Assert (AAA) structure.

### Step 3: Generate Test Body
For each test case:
1. **Arrange**: Setup test data, mocks, preconditions.
2. **Act**: Call the method/endpoint/component interaction.
3. **Assert**: Verify expected outcome matches AC.

### Step 4: Generate Test Data
1. Create factory methods or fixtures for test entities.
2. Include valid, invalid, boundary, and null variations.
3. Use readable test data names that describe the scenario.

### Step 5: Generate Mocks
1. Identify external dependencies (repositories, APIs, services).
2. Create mock configurations using framework conventions.
3. Define mock responses for happy path and error scenarios.

## Test Generation Templates

### JUnit 5 Template
```java
/**
 * Tests for {@link UserService}
 * Linked AC: AC-001, AC-002
 */
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserServiceImpl userService;

    // --- AC-001: User creation with valid data ---

    @Test
    @DisplayName("should create user when valid request provided")
    void should_CreateUser_when_ValidRequestProvided() {
        // Arrange
        var request = UserFixture.validCreateRequest();
        when(userRepository.save(any())).thenReturn(UserFixture.savedUser());

        // Act
        var result = userService.createUser(request);

        // Assert
        assertThat(result).isNotNull();
        assertThat(result.getEmail()).isEqualTo(request.getEmail());
        verify(userRepository).save(any(User.class));
    }

    @Test
    @DisplayName("should throw exception when email already exists")
    void should_ThrowException_when_EmailAlreadyExists() {
        // Arrange
        var request = UserFixture.validCreateRequest();
        when(userRepository.existsByEmail(request.getEmail())).thenReturn(true);

        // Act & Assert
        assertThatThrownBy(() -> userService.createUser(request))
            .isInstanceOf(DuplicateResourceException.class)
            .hasMessageContaining("email");
    }
}
```

### Jest/React Template
```typescript
/**
 * Tests for LoginForm component
 * Linked AC: AC-010, AC-011
 */
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  // --- AC-010: Successful login ---

  it('should submit form and redirect on valid credentials', async () => {
    // Arrange
    const mockLogin = jest.fn().mockResolvedValue({ token: 'abc' });
    render(<LoginForm onLogin={mockLogin} />);

    // Act
    fireEvent.change(screen.getByLabelText(/email/i), {
      target: { value: 'test@example.com' }
    });
    fireEvent.change(screen.getByLabelText(/password/i), {
      target: { value: 'SecurePass1!' }
    });
    fireEvent.click(screen.getByRole('button', { name: /login/i }));

    // Assert
    await waitFor(() => {
      expect(mockLogin).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'SecurePass1!'
      });
    });
  });

  // --- AC-011: Validation errors ---

  it('should show error when email is invalid', async () => {
    // Arrange
    render(<LoginForm onLogin={jest.fn()} />);

    // Act
    fireEvent.change(screen.getByLabelText(/email/i), {
      target: { value: 'invalid-email' }
    });
    fireEvent.click(screen.getByRole('button', { name: /login/i }));

    // Assert
    expect(await screen.findByText(/valid email/i)).toBeInTheDocument();
  });
});
```

### Angular Spec Template
```typescript
/**
 * Tests for UserService
 * Linked AC: AC-020
 */
import { TestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
import { UserService } from './user.service';

describe('UserService', () => {
  let service: UserService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService]
    });
    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => httpMock.verify());

  // --- AC-020: Fetch user by ID ---

  it('should return user when valid ID provided', () => {
    const mockUser = { id: 1, name: 'Test User', email: 'test@example.com' };

    service.getUserById(1).subscribe(user => {
      expect(user).toEqual(mockUser);
    });

    const req = httpMock.expectOne('/api/users/1');
    expect(req.request.method).toBe('GET');
    req.flush(mockUser);
  });
});
```

## Decision Logic And Branching

1. If AC is ambiguous or incomplete:
   - Generate test stub with `// TODO: clarify AC requirement` comment.
   - Add to coverage gap report.

2. If implementation does not exist yet (TDD approach):
   - Generate test scaffolds with expected behavior.
   - Mark tests as `@Disabled` or `it.todo()` with implementation notes.

3. If existing tests cover the AC partially:
   - Generate only missing test scenarios.
   - Add cross-reference to existing tests.

4. If test framework is not specified:
   - Java → JUnit 5 + Mockito + AssertJ.
   - React → Jest + Testing Library.
   - Angular → Jasmine + Karma.
   - E2E → Playwright (default) or Cypress.

## Output Contract
When invoked, return these sections in order:

1. AC-to-Test Mapping Table
2. Generated Test Files (with file paths)
3. Test Data Fixtures
4. Mock Configurations
5. Coverage Gap Analysis
6. Execution Instructions

### AC-to-Test Mapping
| AC ID | Test Case ID | Test Type | File Path | Status |
|---|---|---|---|---|
| AC-001 | TC-user-001 | Unit | `UserServiceTest.java` | Generated |
| AC-001 | TC-user-002 | Unit (negative) | `UserServiceTest.java` | Generated |
| AC-002 | TC-user-003 | Integration | `UserApiTest.java` | Generated |

## Completion Criteria
- Every AC has at least one test case (positive path).
- Critical AC have both positive and negative tests.
- Test files compile/parse without errors.
- Test names clearly describe the scenario.
- AC traceability is documented in comments or annotations.
- Coverage gap report highlights untested behaviors.

## Example Invocations
- /test-automation-scaffolding Generate JUnit 5 tests for user registration AC in the UserService.
- /test-automation-scaffolding Create React Jest tests from login form acceptance criteria.
- /test-automation-scaffolding Bootstrap Angular spec files for order management component.
- /test-automation-scaffolding Generate E2E Playwright tests for checkout user journey.
