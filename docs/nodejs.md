# Node.js

Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine for building server-side applications.

## Installation

```bash
# Using nvm (recommended)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install node
nvm use node

# Check version
node --version
npm --version
```

## Basic HTTP Server

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello World\n');
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000/');
});
```

## Modules

### CommonJS (require)

```javascript
// math.js
function add(a, b) {
  return a + b;
}

module.exports = { add };

// app.js
const { add } = require('./math');
console.log(add(2, 3)); // 5
```

### ES Modules (import)

```javascript
// math.js
export function add(a, b) {
  return a + b;
}

// app.js
import { add } from './math.js';
console.log(add(2, 3)); // 5
```

## File System Operations

```javascript
const fs = require('fs');
const fsPromises = require('fs').promises;

// Read file synchronously
const data = fs.readFileSync('file.txt', 'utf8');

// Read file asynchronously (callback)
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Read file with promises
fsPromises.readFile('file.txt', 'utf8')
  .then(data => console.log(data))
  .catch(err => console.error(err));

// Write file
fs.writeFile('output.txt', 'Hello World', (err) => {
  if (err) throw err;
  console.log('File saved!');
});

// Append to file
fs.appendFile('log.txt', 'New log entry\n', (err) => {
  if (err) throw err;
});

// Check if file exists
if (fs.existsSync('file.txt')) {
  console.log('File exists');
}
```

## Path Module

```javascript
const path = require('path');

// Join paths
const filePath = path.join(__dirname, 'files', 'data.txt');

// Get file extension
const ext = path.extname('file.txt'); // '.txt'

// Get filename
const name = path.basename('/path/to/file.txt'); // 'file.txt'

// Get directory name
const dir = path.dirname('/path/to/file.txt'); // '/path/to'

// Parse path
const parsed = path.parse('/path/to/file.txt');
// { root: '/', dir: '/path/to', base: 'file.txt', ext: '.txt', name: 'file' }
```

## Events

```javascript
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

// Listen for event
myEmitter.on('event', (data) => {
  console.log('Event occurred:', data);
});

// Emit event
myEmitter.emit('event', { message: 'Hello' });

// One-time listener
myEmitter.once('onetime', () => {
  console.log('This will only run once');
});
```

## Streams

```javascript
const fs = require('fs');

// Readable stream
const readStream = fs.createReadStream('input.txt', 'utf8');

readStream.on('data', (chunk) => {
  console.log('Chunk:', chunk);
});

readStream.on('end', () => {
  console.log('Finished reading');
});

// Writable stream
const writeStream = fs.createWriteStream('output.txt');

writeStream.write('Hello ');
writeStream.write('World\n');
writeStream.end();

// Pipe streams
fs.createReadStream('input.txt')
  .pipe(fs.createWriteStream('output.txt'));
```

## Process

```javascript
// Command line arguments
console.log(process.argv);

// Environment variables
console.log(process.env.NODE_ENV);

// Exit process
process.exit(0);

// Current directory
console.log(process.cwd());

// Memory usage
console.log(process.memoryUsage());

// Handle uncaught exceptions
process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err);
  process.exit(1);
});
```

## Child Processes

```javascript
const { exec, spawn } = require('child_process');

// exec - buffer output
exec('ls -la', (error, stdout, stderr) => {
  if (error) {
    console.error(`Error: ${error}`);
    return;
  }
  console.log(stdout);
});

// spawn - stream output
const child = spawn('ls', ['-la']);

child.stdout.on('data', (data) => {
  console.log(`Output: ${data}`);
});

child.stderr.on('data', (data) => {
  console.error(`Error: ${data}`);
});

child.on('close', (code) => {
  console.log(`Process exited with code ${code}`);
});
```

## HTTP Requests

```javascript
const https = require('https');

// GET request
https.get('https://api.example.com/data', (res) => {
  let data = '';

  res.on('data', (chunk) => {
    data += chunk;
  });

  res.on('end', () => {
    console.log(JSON.parse(data));
  });
}).on('error', (err) => {
  console.error('Error:', err);
});

// POST request
const postData = JSON.stringify({ key: 'value' });

const options = {
  hostname: 'api.example.com',
  port: 443,
  path: '/data',
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Content-Length': postData.length
  }
};

const req = https.request(options, (res) => {
  res.on('data', (chunk) => {
    console.log(chunk.toString());
  });
});

req.write(postData);
req.end();
```

## Timers

```javascript
// setTimeout
const timeoutId = setTimeout(() => {
  console.log('Executed after 2 seconds');
}, 2000);

// Clear timeout
clearTimeout(timeoutId);

// setInterval
const intervalId = setInterval(() => {
  console.log('Executed every 1 second');
}, 1000);

// Clear interval
clearInterval(intervalId);

// setImmediate
setImmediate(() => {
  console.log('Executed immediately after I/O events');
});
```

## Util Module

```javascript
const util = require('util');
const fs = require('fs');

// Promisify callback-based functions
const readFile = util.promisify(fs.readFile);

async function readData() {
  const data = await readFile('file.txt', 'utf8');
  console.log(data);
}

// Format strings
const formatted = util.format('%s:%s', 'foo', 'bar'); // 'foo:bar'

// Inspect objects
const obj = { a: 1, b: { c: 2 } };
console.log(util.inspect(obj, { depth: null, colors: true }));
```

## Package Management

```bash
# Initialize project
npm init -y

# Install dependencies
npm install express
npm install --save-dev nodemon

# Install globally
npm install -g typescript

# Update packages
npm update

# Run scripts
npm run dev
npm test
```

## package.json

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.0"
  }
}
```

## Resources

- [Official Documentation](https://nodejs.org/docs/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [NPM Registry](https://www.npmjs.com/)
