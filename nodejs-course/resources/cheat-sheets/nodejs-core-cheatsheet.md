# Node.js Core Concepts Cheat Sheet 🚀

## Event Loop and Asynchronous Programming

### Event Loop Phases
```javascript
// Timer phase - setTimeout, setInterval
setTimeout(() => console.log('Timer'), 0);

// Pending callbacks phase - I/O callbacks
const fs = require('fs');
fs.readFile('file.txt', () => console.log('File read'));

// Poll phase - Fetch new I/O events
// Check phase - setImmediate
setImmediate(() => console.log('Immediate'));

// Close callbacks phase
server.on('close', () => console.log('Server closed'));
```

### Microtasks vs Macrotasks
```javascript
// Microtasks (higher priority)
process.nextTick(() => console.log('Next Tick'));
Promise.resolve().then(() => console.log('Promise'));

// Macrotasks (lower priority)
setTimeout(() => console.log('Timeout'), 0);
setImmediate(() => console.log('Immediate'));

// Output: Next Tick, Promise, Timeout/Immediate (order varies)
```

### Async Patterns
```javascript
// Callbacks
fs.readFile('file.txt', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Promises
const readFilePromise = util.promisify(fs.readFile);
readFilePromise('file.txt')
  .then(data => console.log(data))
  .catch(err => console.error(err));

// Async/Await
async function readFile() {
  try {
    const data = await fs.promises.readFile('file.txt');
    console.log(data);
  } catch (err) {
    console.error(err);
  }
}
```

## File System Operations

### Reading Files
```javascript
const fs = require('fs');
const path = require('path');

// Synchronous (blocking)
const data = fs.readFileSync('file.txt', 'utf8');

// Asynchronous (non-blocking)
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Promise-based
const data = await fs.promises.readFile('file.txt', 'utf8');

// Stream-based (for large files)
const readStream = fs.createReadStream('large-file.txt');
readStream.on('data', chunk => console.log(chunk));
readStream.on('end', () => console.log('File read complete'));
```

### Writing Files
```javascript
// Write string to file
fs.writeFile('output.txt', 'Hello World', (err) => {
  if (err) throw err;
  console.log('File saved');
});

// Append to file
fs.appendFile('log.txt', 'New log entry\n', (err) => {
  if (err) throw err;
});

// Write JSON data
const data = { name: 'John', age: 30 };
fs.writeFile('data.json', JSON.stringify(data, null, 2), (err) => {
  if (err) throw err;
});
```

### Directory Operations
```javascript
// Create directory
fs.mkdir('new-folder', { recursive: true }, (err) => {
  if (err) throw err;
});

// Read directory
fs.readdir('.', (err, files) => {
  if (err) throw err;
  console.log(files);
});

// Get file stats
fs.stat('file.txt', (err, stats) => {
  if (err) throw err;
  console.log('Is file:', stats.isFile());
  console.log('Is directory:', stats.isDirectory());
  console.log('Size:', stats.size);
});
```

## Streams and Buffers

### Readable Streams
```javascript
const { Readable } = require('stream');

// Create readable stream
class MyReadable extends Readable {
  constructor(options) {
    super(options);
    this.current = 0;
  }
  
  _read() {
    if (this.current < 5) {
      this.push(`data-${this.current++}\n`);
    } else {
      this.push(null); // End stream
    }
  }
}

const readable = new MyReadable();
readable.on('data', chunk => console.log(chunk.toString()));
readable.on('end', () => console.log('Stream ended'));
```

### Writable Streams
```javascript
const { Writable } = require('stream');

class MyWritable extends Writable {
  _write(chunk, encoding, callback) {
    console.log(`Writing: ${chunk.toString()}`);
    callback();
  }
}

const writable = new MyWritable();
writable.write('Hello ');
writable.write('World');
writable.end();
```

### Transform Streams
```javascript
const { Transform } = require('stream');

class UpperCaseTransform extends Transform {
  _transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
}

const upperCase = new UpperCaseTransform();
process.stdin.pipe(upperCase).pipe(process.stdout);
```

### Buffer Operations
```javascript
// Create buffers
const buf1 = Buffer.alloc(10); // 10 bytes of zeros
const buf2 = Buffer.from('Hello World', 'utf8');
const buf3 = Buffer.from([1, 2, 3, 4, 5]);

// Buffer operations
console.log(buf2.toString()); // 'Hello World'
console.log(buf2.length); // 11
console.log(buf2.slice(0, 5).toString()); // 'Hello'

// Concatenate buffers
const combined = Buffer.concat([buf2, buf3]);
```

## HTTP and Networking

### Basic HTTP Server
```javascript
const http = require('http');
const url = require('url');

const server = http.createServer((req, res) => {
  const parsedUrl = url.parse(req.url, true);
  const path = parsedUrl.pathname;
  const method = req.method;
  
  // Set CORS headers
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Content-Type', 'application/json');
  
  if (path === '/api/users' && method === 'GET') {
    res.statusCode = 200;
    res.end(JSON.stringify({ users: [] }));
  } else {
    res.statusCode = 404;
    res.end(JSON.stringify({ error: 'Not Found' }));
  }
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### HTTP Client Requests
```javascript
const http = require('http');
const https = require('https');

// GET request
const options = {
  hostname: 'api.example.com',
  port: 443,
  path: '/users',
  method: 'GET',
  headers: {
    'Content-Type': 'application/json'
  }
};

const req = https.request(options, (res) => {
  let data = '';
  
  res.on('data', chunk => {
    data += chunk;
  });
  
  res.on('end', () => {
    console.log(JSON.parse(data));
  });
});

req.on('error', (err) => {
  console.error(err);
});

req.end();
```

## Events and EventEmitter

### Basic EventEmitter
```javascript
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}
const myEmitter = new MyEmitter();

// Add listener
myEmitter.on('event', (data) => {
  console.log('Event received:', data);
});

// Emit event
myEmitter.emit('event', { message: 'Hello World' });

// One-time listener
myEmitter.once('startup', () => {
  console.log('App started');
});

// Remove listener
const listener = (data) => console.log(data);
myEmitter.on('data', listener);
myEmitter.removeListener('data', listener);
```

### Custom EventEmitter Class
```javascript
class Logger extends EventEmitter {
  log(message) {
    console.log(message);
    this.emit('logged', { message, timestamp: new Date() });
  }
  
  error(message) {
    console.error(message);
    this.emit('error', { message, timestamp: new Date() });
  }
}

const logger = new Logger();

logger.on('logged', (data) => {
  // Save to file or database
});

logger.on('error', (data) => {
  // Send alert or notification
});

logger.log('Application started');
```

## Child Processes

### Spawn Process
```javascript
const { spawn } = require('child_process');

const ls = spawn('ls', ['-la']);

ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

ls.stderr.on('data', (data) => {
  console.error(`stderr: ${data}`);
});

ls.on('close', (code) => {
  console.log(`Process exited with code ${code}`);
});
```

### Execute Command
```javascript
const { exec } = require('child_process');

exec('ls -la', (error, stdout, stderr) => {
  if (error) {
    console.error(`Error: ${error}`);
    return;
  }
  console.log(`stdout: ${stdout}`);
  if (stderr) {
    console.error(`stderr: ${stderr}`);
  }
});
```

### Fork Process
```javascript
// main.js
const { fork } = require('child_process');

const child = fork('./worker.js');

child.send({ task: 'process-data', data: [1, 2, 3, 4, 5] });

child.on('message', (result) => {
  console.log('Result from child:', result);
});

// worker.js
process.on('message', (msg) => {
  if (msg.task === 'process-data') {
    const result = msg.data.map(x => x * 2);
    process.send({ result });
  }
});
```

## Clustering

### Basic Cluster Setup
```javascript
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);
  
  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork(); // Restart worker
  });
} else {
  // Worker process
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`Hello from worker ${process.pid}\n`);
  }).listen(3000);
  
  console.log(`Worker ${process.pid} started`);
}
```

## Performance Monitoring

### Memory Usage
```javascript
function getMemoryUsage() {
  const usage = process.memoryUsage();
  return {
    rss: Math.round(usage.rss / 1024 / 1024) + ' MB',
    heapTotal: Math.round(usage.heapTotal / 1024 / 1024) + ' MB',
    heapUsed: Math.round(usage.heapUsed / 1024 / 1024) + ' MB',
    external: Math.round(usage.external / 1024 / 1024) + ' MB'
  };
}

console.log('Memory usage:', getMemoryUsage());
```

### CPU Usage
```javascript
function getCPUUsage() {
  const startUsage = process.cpuUsage();
  
  // Simulate some work
  setTimeout(() => {
    const endUsage = process.cpuUsage(startUsage);
    console.log('CPU Usage:', {
      user: endUsage.user / 1000, // microseconds to milliseconds
      system: endUsage.system / 1000
    });
  }, 1000);
}
```

### Event Loop Lag
```javascript
const { performance } = require('perf_hooks');

function measureEventLoopLag() {
  const start = performance.now();
  
  setImmediate(() => {
    const lag = performance.now() - start;
    console.log(`Event loop lag: ${lag.toFixed(2)}ms`);
    
    // Schedule next measurement
    setTimeout(measureEventLoopLag, 1000);
  });
}

measureEventLoopLag();
```

## Error Handling

### Global Error Handlers
```javascript
// Uncaught exceptions
process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err);
  process.exit(1);
});

// Unhandled promise rejections
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection at:', promise, 'reason:', reason);
  process.exit(1);
});

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('SIGTERM received, shutting down gracefully');
  server.close(() => {
    process.exit(0);
  });
});
```

### Custom Error Classes
```javascript
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}

class DatabaseError extends Error {
  constructor(message, code) {
    super(message);
    this.name = 'DatabaseError';
    this.code = code;
  }
}

// Usage
try {
  throw new ValidationError('Invalid email format', 'email');
} catch (err) {
  if (err instanceof ValidationError) {
    console.log(`Validation error in field ${err.field}: ${err.message}`);
  }
}
```

## Utilities and Helpers

### Path Operations
```javascript
const path = require('path');

console.log(path.join('/users', 'john', 'documents')); // /users/john/documents
console.log(path.resolve('file.txt')); // Absolute path
console.log(path.extname('file.txt')); // .txt
console.log(path.basename('/path/to/file.txt')); // file.txt
console.log(path.dirname('/path/to/file.txt')); // /path/to
```

### URL Parsing
```javascript
const url = require('url');

const myURL = new URL('https://example.com:8080/path?query=value#hash');
console.log(myURL.hostname); // example.com
console.log(myURL.pathname); // /path
console.log(myURL.search); // ?query=value
console.log(myURL.searchParams.get('query')); // value
```

### Crypto Operations
```javascript
const crypto = require('crypto');

// Generate hash
const hash = crypto.createHash('sha256');
hash.update('Hello World');
console.log(hash.digest('hex'));

// Generate random bytes
const randomBytes = crypto.randomBytes(16);
console.log(randomBytes.toString('hex'));

// HMAC
const hmac = crypto.createHmac('sha256', 'secret-key');
hmac.update('data to sign');
console.log(hmac.digest('hex'));
```

---

**Keep this cheat sheet handy while developing Node.js applications!** 📚
