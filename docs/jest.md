# Jest

Jest is a delightful JavaScript testing framework with a focus on simplicity.

## Installation

```bash
# Install Jest
npm install --save-dev jest

# TypeScript support
npm install --save-dev @types/jest ts-jest

# For React
npm install --save-dev @testing-library/react @testing-library/jest-dom
```

## Configuration

```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'node',
  coverageDirectory: 'coverage',
  collectCoverageFrom: [
    'src/**/*.{js,jsx}',
    '!src/**/*.test.{js,jsx}'
  ],
  testMatch: [
    '**/__tests__/**/*.js',
    '**/*.test.js'
  ]
};
```

```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

## Basic Tests

```javascript
// sum.js
function sum(a, b) {
  return a + b;
}

module.exports = sum;

// sum.test.js
const sum = require('./sum');

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});

describe('sum function', () => {
  test('adds positive numbers', () => {
    expect(sum(1, 2)).toBe(3);
  });

  test('adds negative numbers', () => {
    expect(sum(-1, -2)).toBe(-3);
  });
});
```

## Matchers

```javascript
// Equality
expect(value).toBe(4);                    // Exact equality (===)
expect(value).toEqual({ name: 'John' });  // Deep equality
expect(value).not.toBe(5);                // Not equal

// Truthiness
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeUndefined();
expect(value).toBeDefined();

// Numbers
expect(value).toBeGreaterThan(3);
expect(value).toBeGreaterThanOrEqual(3.5);
expect(value).toBeLessThan(5);
expect(value).toBeLessThanOrEqual(4.5);
expect(0.1 + 0.2).toBeCloseTo(0.3);       // Floating point

// Strings
expect('team').not.toMatch(/I/);
expect('team').toMatch(/tea/);

// Arrays and iterables
expect(['apple', 'banana']).toContain('apple');
expect(new Set(['apple', 'banana'])).toContain('apple');

// Exceptions
expect(() => {
  throw new Error('Error message');
}).toThrow();
expect(() => {
  throw new Error('Error message');
}).toThrow('Error message');
```

## Async Testing

```javascript
// Promises
test('async test with promises', () => {
  return fetchData().then(data => {
    expect(data).toBe('peanut butter');
  });
});

// Async/Await
test('async test with async/await', async () => {
  const data = await fetchData();
  expect(data).toBe('peanut butter');
});

test('async test with error', async () => {
  expect.assertions(1);
  try {
    await fetchData();
  } catch (error) {
    expect(error).toMatch('error');
  }
});

// Alternative error testing
test('async error test', async () => {
  await expect(fetchData()).rejects.toThrow('error');
});
```

## Mocking

### Mock Functions

```javascript
// Create mock function
const mockFn = jest.fn();

// Mock implementation
const mockFn = jest.fn(x => x + 1);

// Test mock
test('mock function', () => {
  mockFn(1);
  mockFn(2);

  expect(mockFn).toHaveBeenCalled();
  expect(mockFn).toHaveBeenCalledTimes(2);
  expect(mockFn).toHaveBeenCalledWith(1);
  expect(mockFn).toHaveBeenLastCalledWith(2);
});

// Mock return values
const mockFn = jest.fn();
mockFn.mockReturnValue(42);
mockFn.mockReturnValueOnce(100).mockReturnValueOnce(200);

// Mock resolved/rejected promises
mockFn.mockResolvedValue('success');
mockFn.mockRejectedValue(new Error('error'));
```

### Mock Modules

```javascript
// axios.js
import axios from 'axios';

export async function fetchUser(id) {
  const response = await axios.get(`/users/${id}`);
  return response.data;
}

// axios.test.js
import axios from 'axios';
import { fetchUser } from './axios';

jest.mock('axios');

test('fetches user', async () => {
  const user = { id: 1, name: 'John' };
  axios.get.mockResolvedValue({ data: user });

  const result = await fetchUser(1);

  expect(result).toEqual(user);
  expect(axios.get).toHaveBeenCalledWith('/users/1');
});
```

## Setup and Teardown

```javascript
// Before each test
beforeEach(() => {
  initializeDatabase();
});

// After each test
afterEach(() => {
  clearDatabase();
});

// Before all tests
beforeAll(() => {
  connectToDatabase();
});

// After all tests
afterAll(() => {
  disconnectFromDatabase();
});

// Scoped to describe block
describe('database tests', () => {
  beforeEach(() => {
    initializeDatabase();
  });

  test('test 1', () => {
    // Database initialized
  });
});
```

## Snapshot Testing

```javascript
// Component
function Link({ page, children }) {
  return <a href={page}>{children}</a>;
}

// Test
test('renders correctly', () => {
  const tree = renderer
    .create(<Link page="http://example.com">Example</Link>)
    .toJSON();

  expect(tree).toMatchSnapshot();
});

// Update snapshots
// npm test -- -u
```

## Testing React Components

```javascript
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import '@testing-library/jest-dom';
import Button from './Button';

test('renders button', () => {
  render(<Button>Click me</Button>);

  const button = screen.getByText('Click me');
  expect(button).toBeInTheDocument();
});

test('button click', () => {
  const handleClick = jest.fn();
  render(<Button onClick={handleClick}>Click me</Button>);

  const button = screen.getByText('Click me');
  fireEvent.click(button);

  expect(handleClick).toHaveBeenCalledTimes(1);
});

test('async component', async () => {
  render(<UserList />);

  const user = await screen.findByText('John Doe');
  expect(user).toBeInTheDocument();
});
```

## Coverage

```bash
# Run with coverage
npm test -- --coverage

# Coverage thresholds in jest.config.js
module.exports = {
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
};
```

## Watch Mode

```bash
# Watch mode
npm test -- --watch

# Watch all files
npm test -- --watchAll

# Commands in watch mode:
# › Press f to run only failed tests.
# › Press o to only run tests related to changed files.
# › Press p to filter by a filename regex pattern.
# › Press t to filter by a test name regex pattern.
# › Press q to quit watch mode.
# › Press Enter to trigger a test run.
```

## Timer Mocks

```javascript
jest.useFakeTimers();

test('waits 1 second', () => {
  const callback = jest.fn();

  setTimeout(callback, 1000);

  // Fast-forward time
  jest.runAllTimers();

  expect(callback).toHaveBeenCalled();
});

test('advances timers by time', () => {
  const callback = jest.fn();

  setTimeout(callback, 1000);

  // Advance by 500ms
  jest.advanceTimersByTime(500);
  expect(callback).not.toHaveBeenCalled();

  // Advance by another 500ms
  jest.advanceTimersByTime(500);
  expect(callback).toHaveBeenCalled();
});
```

## Custom Matchers

```javascript
// Custom matcher
expect.extend({
  toBeWithinRange(received, floor, ceiling) {
    const pass = received >= floor && received <= ceiling;
    if (pass) {
      return {
        message: () =>
          `expected ${received} not to be within range ${floor} - ${ceiling}`,
        pass: true,
      };
    } else {
      return {
        message: () =>
          `expected ${received} to be within range ${floor} - ${ceiling}`,
        pass: false,
      };
    }
  },
});

test('numeric ranges', () => {
  expect(100).toBeWithinRange(90, 110);
  expect(101).not.toBeWithinRange(0, 100);
});
```

## Test Structure Best Practices

```javascript
describe('UserService', () => {
  describe('getUser', () => {
    test('returns user when user exists', async () => {
      // Arrange
      const userId = 1;
      const expectedUser = { id: 1, name: 'John' };

      // Act
      const result = await UserService.getUser(userId);

      // Assert
      expect(result).toEqual(expectedUser);
    });

    test('throws error when user does not exist', async () => {
      // Arrange
      const userId = 999;

      // Act & Assert
      await expect(UserService.getUser(userId))
        .rejects
        .toThrow('User not found');
    });
  });
});
```

## Resources

- [Official Documentation](https://jestjs.io/)
- [Testing Library](https://testing-library.com/)
- [Jest Cheat Sheet](https://github.com/sapegin/jest-cheat-sheet)
