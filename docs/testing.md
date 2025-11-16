# Software Testing Best Practices

A comprehensive guide to software testing strategies, methodologies, and best practices for ensuring quality and reliability.

## Table of Contents

- [Testing Fundamentals](#testing-fundamentals)
- [Testing Types](#testing-types)
- [Test-Driven Development](#test-driven-development)
- [Unit Testing](#unit-testing)
- [Integration Testing](#integration-testing)
- [End-to-End Testing](#end-to-end-testing)
- [Testing Strategies](#testing-strategies)
- [Test Automation](#test-automation)
- [Performance Testing](#performance-testing)
- [Security Testing](#security-testing)
- [Best Practices](#best-practices)

## Testing Fundamentals

### Testing Pyramid

```
         /\
        /E2E\          Few, slow, expensive
       /------\
      /  Integ \       Moderate number
     /----------\
    /    Unit    \     Many, fast, cheap
   /--------------\
```

```yaml
Unit Tests (70%):
  - Test individual functions/methods
  - Fast execution
  - Easy to maintain
  - High coverage

Integration Tests (20%):
  - Test component interactions
  - Database, API, services
  - Moderate speed
  - More complex

E2E Tests (10%):
  - Test complete user flows
  - Slow execution
  - Brittle
  - High confidence
```

### Test Qualities (F.I.R.S.T)

```yaml
Fast:
  - Run quickly
  - Immediate feedback
  - Run frequently

Independent:
  - No dependencies between tests
  - Run in any order
  - Isolated state

Repeatable:
  - Same results every time
  - No flaky tests
  - Deterministic

Self-Validating:
  - Clear pass/fail
  - No manual verification
  - Automated assertions

Timely:
  - Written with code (TDD)
  - Before or with implementation
  - Not as afterthought
```

### Test Coverage Metrics

```yaml
Line Coverage:
  - Percentage of lines executed
  - Most common metric
  - 80%+ is good target

Branch Coverage:
  - Percentage of decision branches
  - Better than line coverage
  - Tests all paths

Function Coverage:
  - Percentage of functions called
  - Coarse-grained metric

Statement Coverage:
  - Individual statements executed
  - Similar to line coverage
```

## Testing Types

### By Level

```yaml
Unit Testing:
  Scope: Individual functions/classes
  Speed: Very fast (milliseconds)
  Tools: JUnit, pytest, Jest
  Example: Test a sorting function

Integration Testing:
  Scope: Multiple components together
  Speed: Moderate (seconds)
  Tools: TestNG, pytest, Mocha
  Example: Test API with database

System Testing:
  Scope: Complete application
  Speed: Slow (minutes)
  Tools: Selenium, Cypress
  Example: Test full user workflow

Acceptance Testing:
  Scope: Business requirements
  Speed: Slow
  Tools: Cucumber, SpecFlow
  Example: Test user stories
```

### By Purpose

```yaml
Functional Testing:
  - Does it work correctly?
  - Feature validation
  - Expected behavior

Non-Functional Testing:
  - Performance
  - Security
  - Usability
  - Reliability
  - Scalability

Regression Testing:
  - Existing features still work
  - After changes
  - Automated test suite

Smoke Testing:
  - Basic functionality
  - Quick validation
  - "Does it turn on?"

Sanity Testing:
  - Rational behavior
  - Quick check after build
  - Subset of regression
```

### By Knowledge

```yaml
White Box (Glass Box):
  - Internal structure known
  - Code coverage
  - Path testing
  - Developer testing

Black Box:
  - No internal knowledge
  - Input/output validation
  - Specification-based
  - QA testing

Gray Box:
  - Partial knowledge
  - Database schemas
  - API contracts
  - Hybrid approach
```

## Test-Driven Development (TDD)

### TDD Cycle (Red-Green-Refactor)

```yaml
1. Red - Write failing test:
   - Define expected behavior
   - Write test first
   - Test fails (no implementation)

2. Green - Make it pass:
   - Write minimal code
   - Test passes
   - Don't optimize yet

3. Refactor - Improve code:
   - Clean up code
   - Remove duplication
   - Tests still pass
```

### TDD Example

```python
# 1. RED - Write failing test
def test_add():
    calculator = Calculator()
    assert calculator.add(2, 3) == 5

# Test fails - Calculator doesn't exist

# 2. GREEN - Make it pass
class Calculator:
    def add(self, a, b):
        return a + b

# Test passes

# 3. REFACTOR - Improve
class Calculator:
    """A simple calculator class."""

    def add(self, a: int, b: int) -> int:
        """Add two numbers."""
        return a + b

# Test still passes
```

### Benefits of TDD

```yaml
Advantages:
  - Better design
  - Higher test coverage
  - Living documentation
  - Reduced debugging
  - Confidence in changes

Challenges:
  - Learning curve
  - Time investment
  - Legacy code
  - UI testing difficult
```

## Unit Testing

### Unit Test Structure (AAA Pattern)

```python
def test_user_creation():
    # Arrange - Setup
    username = "testuser"
    email = "test@example.com"

    # Act - Execute
    user = User.create(username, email)

    # Assert - Verify
    assert user.username == username
    assert user.email == email
    assert user.is_active == True
```

### Test Naming Conventions

```python
# Good test names
def test_empty_cart_has_zero_total():
    """Cart total should be 0 when empty."""
    pass

def test_adding_item_increases_total():
    """Adding item should increase cart total."""
    pass

def test_invalid_email_raises_validation_error():
    """Invalid email should raise ValidationError."""
    pass

# Bad test names
def test1():
    pass

def test_cart():
    pass

def testAddingStuff():
    pass
```

### Mocking and Stubbing

```python
# Python (using unittest.mock)
from unittest.mock import Mock, patch

def test_send_email():
    # Mock external service
    with patch('email_service.send') as mock_send:
        mock_send.return_value = True

        result = send_welcome_email("test@example.com")

        mock_send.assert_called_once_with(
            to="test@example.com",
            subject="Welcome"
        )
        assert result == True

# JavaScript (using Jest)
test('fetches data from API', async () => {
  // Mock fetch
  global.fetch = jest.fn(() =>
    Promise.resolve({
      json: () => Promise.resolve({ data: 'test' })
    })
  );

  const data = await fetchData();

  expect(fetch).toHaveBeenCalledWith('/api/data');
  expect(data).toEqual({ data: 'test' });
});
```

### Test Fixtures

```python
import pytest

@pytest.fixture
def sample_user():
    """Create a sample user for testing."""
    user = User(username="testuser", email="test@example.com")
    user.save()
    yield user
    user.delete()  # Cleanup

def test_user_login(sample_user):
    result = sample_user.login("password123")
    assert result == True

# JavaScript (Jest)
beforeEach(() => {
  // Setup before each test
  database.connect();
});

afterEach(() => {
  // Cleanup after each test
  database.clear();
});
```

## Integration Testing

### Database Testing

```python
import pytest

@pytest.fixture
def database():
    """Setup test database."""
    db = TestDatabase()
    db.create_tables()
    yield db
    db.drop_tables()

def test_user_repository_save(database):
    # Arrange
    user = User(username="test", email="test@example.com")
    repo = UserRepository(database)

    # Act
    saved_user = repo.save(user)

    # Assert
    assert saved_user.id is not None
    found_user = repo.find_by_id(saved_user.id)
    assert found_user.username == "test"
```

### API Testing

```python
# Python (using requests)
def test_get_users_endpoint():
    response = requests.get("http://api/users")

    assert response.status_code == 200
    assert "users" in response.json()
    assert len(response.json()["users"]) > 0

def test_create_user_endpoint():
    payload = {
        "username": "newuser",
        "email": "new@example.com"
    }

    response = requests.post("http://api/users", json=payload)

    assert response.status_code == 201
    assert response.json()["id"] is not None

# JavaScript (using Supertest)
const request = require('supertest');
const app = require('./app');

describe('GET /api/users', () => {
  it('returns list of users', async () => {
    const response = await request(app)
      .get('/api/users')
      .expect(200);

    expect(response.body.users).toBeDefined();
    expect(Array.isArray(response.body.users)).toBe(true);
  });
});
```

### Contract Testing

```yaml
Consumer-Driven Contracts:
  1. Consumer defines expectations
  2. Provider validates contract
  3. Both parties test against contract

Tools:
  - Pact
  - Spring Cloud Contract
  - Postman Contract Testing

Example (Pact):
  Consumer Test:
    .given('user exists')
    .uponReceiving('request for user')
    .withRequest({
      method: 'GET',
      path: '/users/1'
    })
    .willRespondWith({
      status: 200,
      body: { id: 1, name: 'Alice' }
    })

  Provider Verification:
    - Verify against actual API
    - Ensures contract is honored
```

## End-to-End Testing

### E2E Testing Frameworks

```yaml
Selenium:
  - Browser automation
  - Multiple languages
  - Cross-browser
  - Mature ecosystem

Cypress:
  - Modern JavaScript framework
  - Fast execution
  - Time travel debugging
  - Easy setup

Playwright:
  - Multi-browser (Chromium, Firefox, WebKit)
  - Fast and reliable
  - Auto-wait
  - API testing

Puppeteer:
  - Chrome/Chromium only
  - Headless by default
  - Google maintained
```

### E2E Test Example

```javascript
// Cypress
describe('Login Flow', () => {
  it('successfully logs in user', () => {
    cy.visit('/login');

    cy.get('input[name="email"]').type('user@example.com');
    cy.get('input[name="password"]').type('password123');
    cy.get('button[type="submit"]').click();

    cy.url().should('include', '/dashboard');
    cy.get('.welcome-message').should('contain', 'Welcome back');
  });

  it('shows error for invalid credentials', () => {
    cy.visit('/login');

    cy.get('input[name="email"]').type('wrong@example.com');
    cy.get('input[name="password"]').type('wrongpass');
    cy.get('button[type="submit"]').click();

    cy.get('.error-message').should('be.visible');
    cy.get('.error-message').should('contain', 'Invalid credentials');
  });
});

// Playwright
const { test, expect } = require('@playwright/test');

test('user can complete checkout', async ({ page }) => {
  await page.goto('/products');

  await page.click('button.add-to-cart');
  await page.click('a.cart-link');

  await expect(page.locator('.cart-items')).toHaveCount(1);

  await page.click('button.checkout');
  await page.fill('input[name="address"]', '123 Main St');
  await page.fill('input[name="card"]', '4111111111111111');
  await page.click('button.submit-order');

  await expect(page.locator('.confirmation')).toBeVisible();
});
```

### Page Object Model (POM)

```javascript
// Page Object
class LoginPage {
  constructor(page) {
    this.page = page;
    this.emailInput = 'input[name="email"]';
    this.passwordInput = 'input[name="password"]';
    this.submitButton = 'button[type="submit"]';
    this.errorMessage = '.error-message';
  }

  async navigate() {
    await this.page.goto('/login');
  }

  async login(email, password) {
    await this.page.fill(this.emailInput, email);
    await this.page.fill(this.passwordInput, password);
    await this.page.click(this.submitButton);
  }

  async getErrorMessage() {
    return await this.page.textContent(this.errorMessage);
  }
}

// Test using Page Object
test('login with invalid credentials', async ({ page }) => {
  const loginPage = new LoginPage(page);

  await loginPage.navigate();
  await loginPage.login('wrong@example.com', 'wrongpass');

  const error = await loginPage.getErrorMessage();
  expect(error).toContain('Invalid credentials');
});
```

## Testing Strategies

### Equivalence Partitioning

```yaml
Input: Age (0-120)

Valid Partitions:
  - 0-17: Minor
  - 18-64: Adult
  - 65-120: Senior

Invalid Partitions:
  - < 0: Negative age
  - > 120: Too old

Test Cases:
  - -1 (invalid)
  - 0 (valid - minor)
  - 10 (valid - minor)
  - 18 (valid - adult)
  - 50 (valid - adult)
  - 65 (valid - senior)
  - 100 (valid - senior)
  - 121 (invalid)
```

### Boundary Value Analysis

```yaml
Input: Quantity (1-100)

Test Boundaries:
  - 0 (below minimum)
  - 1 (minimum valid)
  - 2 (just above minimum)
  - 50 (middle)
  - 99 (just below maximum)
  - 100 (maximum valid)
  - 101 (above maximum)

Why: Bugs often occur at boundaries
```

### State Transition Testing

```yaml
Shopping Cart States:
  Empty → Adding → Processing → Completed

Transitions:
  Empty + Add Item → Adding
  Adding + Add Item → Adding
  Adding + Checkout → Processing
  Processing + Payment → Completed
  Adding + Clear → Empty
  Processing + Cancel → Adding

Test Cases:
  - Empty cart, add item
  - Add multiple items
  - Checkout with items
  - Complete payment
  - Cancel during checkout
  - Clear cart
```

## Test Automation

### CI/CD Integration

```yaml
# GitHub Actions
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Setup Node
        uses: actions/setup-node@v2
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm install

      - name: Run unit tests
        run: npm test

      - name: Run integration tests
        run: npm run test:integration

      - name: Upload coverage
        uses: codecov/codecov-action@v2
        with:
          files: ./coverage/lcov.info
```

### Test Parallelization

```javascript
// Jest parallel execution (default)
{
  "jest": {
    "maxWorkers": "50%",  // Use 50% of CPU cores
    "testPathIgnorePatterns": ["/node_modules/"]
  }
}

// Playwright parallel execution
const { test } = require('@playwright/test');

test.describe.parallel('Parallel Tests', () => {
  test('test 1', async ({ page }) => {
    // Runs in parallel
  });

  test('test 2', async ({ page }) => {
    // Runs in parallel
  });
});
```

### Test Reporting

```yaml
HTML Reports:
  - Visual test results
  - Screenshots/videos
  - Execution time
  - Pass/fail details

JUnit XML:
  - CI/CD integration
  - Standard format
  - Test metrics

Coverage Reports:
  - Istanbul/NYC
  - Codecov
  - SonarQube
  - Line/branch coverage
```

## Performance Testing

### Load Testing

```yaml
Objective:
  - Test system under expected load
  - Identify bottlenecks
  - Validate performance requirements

Metrics:
  - Response time
  - Throughput (requests/sec)
  - Error rate
  - Resource utilization

Tools:
  - Apache JMeter
  - Gatling
  - k6
  - Locust
```

### Stress Testing

```yaml
Objective:
  - Find breaking point
  - Test beyond normal capacity
  - Recovery behavior

Approach:
  1. Gradually increase load
  2. Monitor system behavior
  3. Identify failure point
  4. Test recovery
```

### Example (k6)

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '2m', target: 100 },   // Ramp up
    { duration: '5m', target: 100 },   // Stay at 100 users
    { duration: '2m', target: 200 },   // Ramp up more
    { duration: '5m', target: 200 },   // Stay at 200 users
    { duration: '2m', target: 0 },     // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% of requests < 500ms
    http_req_failed: ['rate<0.01'],    // Error rate < 1%
  },
};

export default function() {
  let response = http.get('https://api.example.com/users');

  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });

  sleep(1);
}
```

## Security Testing

### Common Vulnerabilities (OWASP Top 10)

```yaml
1. Injection (SQL, NoSQL, OS):
   Test:
     - Input: ' OR '1'='1
     - Input: '; DROP TABLE users--
   Prevent:
     - Parameterized queries
     - Input validation
     - ORM usage

2. Broken Authentication:
   Test:
     - Weak passwords
     - Session fixation
     - Brute force
   Prevent:
     - Strong password policy
     - MFA
     - Rate limiting

3. Sensitive Data Exposure:
   Test:
     - Unencrypted data
     - Weak encryption
     - Data in logs
   Prevent:
     - Encryption at rest/transit
     - Secure key management

4. XSS (Cross-Site Scripting):
   Test:
     - Input: <script>alert('XSS')</script>
   Prevent:
     - Input sanitization
     - Output encoding
     - CSP headers

5. Broken Access Control:
   Test:
     - Access other user data
     - Privilege escalation
   Prevent:
     - Authorization checks
     - Least privilege
     - Role-based access
```

### Security Testing Tools

```yaml
Static Analysis (SAST):
  - SonarQube
  - Checkmarx
  - ESLint (security plugins)
  - Bandit (Python)

Dynamic Analysis (DAST):
  - OWASP ZAP
  - Burp Suite
  - Nessus
  - Acunetix

Dependency Scanning:
  - npm audit
  - Snyk
  - OWASP Dependency-Check
  - GitHub Dependabot
```

## Best Practices

### General Principles

```yaml
Write Tests First:
  - TDD approach
  - Design for testability
  - Living documentation

Keep Tests Simple:
  - One assertion per test
  - Clear test names
  - Easy to understand

Fast Feedback:
  - Run tests frequently
  - Fast unit tests
  - Slow tests separate

Isolate Tests:
  - No shared state
  - Independent execution
  - Clean setup/teardown

Avoid Test Duplication:
  - DRY principle
  - Helper functions
  - Fixtures/factories
```

### Test Organization

```
tests/
├── unit/
│   ├── utils/
│   │   └── test_string_utils.py
│   └── models/
│       └── test_user.py
├── integration/
│   ├── test_api.py
│   └── test_database.py
├── e2e/
│   └── test_user_flow.py
├── fixtures/
│   └── sample_data.py
└── conftest.py  # Shared fixtures
```

### Code Coverage Guidelines

```yaml
Target Coverage:
  - Unit tests: 80-90%
  - Integration tests: 60-70%
  - E2E tests: Critical paths

What to Cover:
  - Business logic (high priority)
  - Edge cases
  - Error handling
  - Common user paths

What NOT to Cover:
  - Generated code
  - Third-party libraries
  - Simple getters/setters
  - Configuration files

Remember:
  - 100% coverage ≠ bug-free
  - Quality > quantity
  - Test meaningful scenarios
```

### Flaky Test Prevention

```yaml
Causes:
  - Timing issues
  - External dependencies
  - Shared state
  - Random data

Solutions:
  - Explicit waits (not sleep)
  - Mock external services
  - Isolated test data
  - Deterministic tests
  - Retry mechanism (last resort)

Example:
  # Bad
  sleep(1000)
  assert element.visible()

  # Good
  waitFor(element).toBeVisible()
  assert element.visible()
```

## Additional Resources

- [Testing Best Practices](https://martinfowler.com/testing/)
- [Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [Google Testing Blog](https://testing.googleblog.com/)

---

*Last updated: 2025-11-16*
