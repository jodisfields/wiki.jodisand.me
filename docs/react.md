# React

React is a JavaScript library for building user interfaces, particularly single-page applications.

## Quick Start

```bash
# Create new React app
npx create-react-app my-app
cd my-app
npm start

# Or with Vite
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

## Components

### Functional Components

```javascript
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

// With Arrow Function
const Welcome = ({ name }) => {
  return <h1>Hello, {name}</h1>;
};
```

### Class Components

```javascript
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

## Hooks

### useState

```javascript
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

### useEffect

```javascript
import { useEffect, useState } from 'react';

function DataFetcher() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch('https://api.example.com/data')
      .then(res => res.json())
      .then(setData);
  }, []); // Empty array = run once on mount

  return <div>{data && JSON.stringify(data)}</div>;
}
```

### useContext

```javascript
import { createContext, useContext } from 'react';

const ThemeContext = createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <ThemedButton />
    </ThemeContext.Provider>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Themed Button</button>;
}
```

### useRef

```javascript
import { useRef } from 'react';

function TextInput() {
  const inputRef = useRef(null);

  const focusInput = () => {
    inputRef.current.focus();
  };

  return (
    <>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus Input</button>
    </>
  );
}
```

### useMemo & useCallback

```javascript
import { useMemo, useCallback, useState } from 'react';

function ExpensiveComponent({ items }) {
  const [count, setCount] = useState(0);

  // Memoize expensive calculation
  const sortedItems = useMemo(() => {
    return items.sort((a, b) => a - b);
  }, [items]);

  // Memoize callback
  const handleClick = useCallback(() => {
    setCount(c => c + 1);
  }, []);

  return <div>...</div>;
}
```

## Props & State

```javascript
// Parent Component
function Parent() {
  const [data, setData] = useState('Hello');

  return <Child message={data} onUpdate={setData} />;
}

// Child Component
function Child({ message, onUpdate }) {
  return (
    <div>
      <p>{message}</p>
      <button onClick={() => onUpdate('Updated!')}>Update</button>
    </div>
  );
}
```

## Conditional Rendering

```javascript
function Greeting({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? <UserGreeting /> : <GuestGreeting />}
    </div>
  );
}

// With && operator
function Mailbox({ unreadMessages }) {
  return (
    <div>
      {unreadMessages.length > 0 && (
        <h2>You have {unreadMessages.length} unread messages.</h2>
      )}
    </div>
  );
}
```

## Lists & Keys

```javascript
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

## Forms

```javascript
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log({ email, password });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

## React Router

```javascript
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/users/:id" element={<UserProfile />} />
      </Routes>
    </BrowserRouter>
  );
}
```

## Performance Optimization

```javascript
import { memo } from 'react';

// Memoize component to prevent unnecessary re-renders
const ExpensiveComponent = memo(function ExpensiveComponent({ data }) {
  return <div>{/* Expensive rendering */}</div>;
});

// Lazy loading
import { lazy, Suspense } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>
  );
}
```

## Resources

- [Official Documentation](https://react.dev/)
- [React Patterns](https://reactpatterns.com/)
- [React DevTools](https://react.dev/learn/react-developer-tools)
