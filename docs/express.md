# Express.js

Express is a minimal and flexible Node.js web application framework.

## Installation

```bash
npm install express
```

## Basic Server

```javascript
const express = require('express');
const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

## Routing

### Basic Routes

```javascript
// GET request
app.get('/users', (req, res) => {
  res.json({ users: [] });
});

// POST request
app.post('/users', (req, res) => {
  res.status(201).json({ message: 'User created' });
});

// PUT request
app.put('/users/:id', (req, res) => {
  res.json({ message: 'User updated' });
});

// DELETE request
app.delete('/users/:id', (req, res) => {
  res.json({ message: 'User deleted' });
});
```

### Route Parameters

```javascript
app.get('/users/:id', (req, res) => {
  const { id } = req.params;
  res.json({ userId: id });
});

// Multiple parameters
app.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  res.json({ userId, postId });
});
```

### Query Parameters

```javascript
// /search?q=express&page=1
app.get('/search', (req, res) => {
  const { q, page } = req.query;
  res.json({ query: q, page });
});
```

## Middleware

### Application-level Middleware

```javascript
// Logger middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

// JSON body parser
app.use(express.json());

// URL-encoded body parser
app.use(express.urlencoded({ extended: true }));

// Static files
app.use(express.static('public'));
```

### Route-level Middleware

```javascript
const authenticate = (req, res, next) => {
  const token = req.headers.authorization;

  if (!token) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  // Verify token
  next();
};

app.get('/protected', authenticate, (req, res) => {
  res.json({ message: 'Protected resource' });
});
```

### Error Handling Middleware

```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: 'Internal Server Error',
    message: err.message
  });
});
```

## Request Object

```javascript
app.get('/info', (req, res) => {
  console.log(req.method);        // GET
  console.log(req.url);           // /info
  console.log(req.params);        // Route parameters
  console.log(req.query);         // Query parameters
  console.log(req.body);          // Request body
  console.log(req.headers);       // Request headers
  console.log(req.cookies);       // Cookies
  console.log(req.ip);            // IP address
  console.log(req.hostname);      // Hostname

  res.send('Info logged');
});
```

## Response Object

```javascript
app.get('/response', (req, res) => {
  // Send text
  res.send('Hello');

  // Send JSON
  res.json({ message: 'Success' });

  // Set status code
  res.status(201).json({ created: true });

  // Send file
  res.sendFile(__dirname + '/index.html');

  // Download file
  res.download('/path/to/file.pdf');

  // Redirect
  res.redirect('/new-url');

  // Set headers
  res.set('Content-Type', 'application/json');

  // Set cookie
  res.cookie('name', 'value', { maxAge: 900000 });
});
```

## Router

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json({ users: [] });
});

router.get('/:id', (req, res) => {
  res.json({ user: { id: req.params.id } });
});

router.post('/', (req, res) => {
  res.status(201).json({ message: 'User created' });
});

module.exports = router;

// app.js
const userRoutes = require('./routes/users');
app.use('/api/users', userRoutes);
```

## Template Engines

### EJS

```javascript
app.set('view engine', 'ejs');

app.get('/', (req, res) => {
  res.render('index', {
    title: 'My App',
    users: ['Alice', 'Bob']
  });
});
```

```html
<!-- views/index.ejs -->
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
</head>
<body>
  <h1><%= title %></h1>
  <ul>
    <% users.forEach(user => { %>
      <li><%= user %></li>
    <% }); %>
  </ul>
</body>
</html>
```

## File Upload

```javascript
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });

app.post('/upload', upload.single('file'), (req, res) => {
  console.log(req.file);
  res.json({ filename: req.file.filename });
});

// Multiple files
app.post('/uploads', upload.array('files', 5), (req, res) => {
  res.json({ files: req.files });
});
```

## CORS

```javascript
const cors = require('cors');

// Enable all CORS requests
app.use(cors());

// Configure CORS
app.use(cors({
  origin: 'http://example.com',
  methods: ['GET', 'POST'],
  credentials: true
}));
```

## Sessions

```javascript
const session = require('express-session');

app.use(session({
  secret: 'secret-key',
  resave: false,
  saveUninitialized: true,
  cookie: { maxAge: 60000 }
}));

app.get('/login', (req, res) => {
  req.session.user = { id: 1, name: 'John' };
  res.send('Logged in');
});

app.get('/profile', (req, res) => {
  if (req.session.user) {
    res.json(req.session.user);
  } else {
    res.status(401).send('Not authenticated');
  }
});
```

## Environment Variables

```javascript
require('dotenv').config();

const PORT = process.env.PORT || 3000;
const DB_URL = process.env.DATABASE_URL;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

```bash
# .env
PORT=3000
DATABASE_URL=mongodb://localhost:27017/mydb
SECRET_KEY=mysecret
```

## Example API

```javascript
const express = require('express');
const app = express();

app.use(express.json());

let users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' }
];

// Get all users
app.get('/api/users', (req, res) => {
  res.json(users);
});

// Get user by ID
app.get('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));

  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  res.json(user);
});

// Create user
app.post('/api/users', (req, res) => {
  const user = {
    id: users.length + 1,
    name: req.body.name
  };

  users.push(user);
  res.status(201).json(user);
});

// Update user
app.put('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));

  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  user.name = req.body.name;
  res.json(user);
});

// Delete user
app.delete('/api/users/:id', (req, res) => {
  const index = users.findIndex(u => u.id === parseInt(req.params.id));

  if (index === -1) {
    return res.status(404).json({ error: 'User not found' });
  }

  users.splice(index, 1);
  res.status(204).send();
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

## Resources

- [Official Documentation](https://expressjs.com/)
- [Express API Reference](https://expressjs.com/en/4x/api.html)
- [Awesome Express](https://github.com/rajikaimal/awesome-express)
