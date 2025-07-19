# Module 01: Node.js Architecture and Event Loop

**Duration:** 1 Week | **Difficulty:** Intermediate

## 🎯 Learning Objectives

By the end of this module, you will:
- ✅ Understand Node.js architecture and its components
- ✅ Master the event loop and its phases
- ✅ Understand the difference between blocking and non-blocking operations
- ✅ Learn about the thread pool and libuv
- ✅ Understand how Node.js handles concurrency
- ✅ Debug and profile event loop performance
- ✅ Write efficient asynchronous code

## 📚 Node.js Architecture Overview

### What Makes Node.js Special?

**Node.js** is built on Chrome's V8 JavaScript engine and uses an event-driven, non-blocking I/O model that makes it lightweight and efficient. Unlike traditional server technologies, Node.js runs on a single thread but can handle thousands of concurrent connections.

### Core Components

```
┌─────────────────────────────────────────────────────────┐
│                    Node.js Application                  │
├─────────────────────────────────────────────────────────┤
│                    Node.js Bindings                     │
├─────────────────────────────────────────────────────────┤
│                      V8 Engine                          │
│                   (JavaScript Runtime)                  │
├─────────────────────────────────────────────────────────┤
│                        libuv                            │
│                  (Event Loop + Thread Pool)             │
├─────────────────────────────────────────────────────────┤
│                   Operating System                      │
└─────────────────────────────────────────────────────────┘
```

#### 1. V8 JavaScript Engine
- **Purpose**: Executes JavaScript code
- **Features**: Just-in-time compilation, garbage collection
- **Performance**: Highly optimized for modern JavaScript

#### 2. libuv
- **Purpose**: Provides the event loop and thread pool
- **Features**: Cross-platform asynchronous I/O
- **Handles**: File system, networking, timers, child processes

#### 3. Node.js Bindings
- **Purpose**: Bridge between JavaScript and C++
- **Provides**: Access to system APIs and native modules

## ⚡ The Event Loop Deep Dive

### Event Loop Phases

The event loop has six main phases, each with its own callback queue:

```
┌───────────────────────────┐
┌─>│           timers          │  ← setTimeout, setInterval
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  ← I/O callbacks deferred to next loop
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │  ← Internal use only
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           poll            │  ← Fetch new I/O events
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           check           │  ← setImmediate callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │  ← socket.on('close', ...)
   └───────────────────────────┘
```

### Phase Details

#### 1. Timers Phase
```javascript
// These callbacks are executed in this phase
setTimeout(() => console.log('Timer 1'), 0);
setInterval(() => console.log('Interval'), 1000);
```

#### 2. Pending Callbacks Phase
```javascript
// I/O callbacks that were deferred
const fs = require('fs');
fs.readFile('file.txt', (err, data) => {
  // This might be executed in pending callbacks phase
  console.log('File read complete');
});
```

#### 3. Poll Phase
- **Most important phase**
- Fetches new I/O events
- Executes I/O related callbacks
- Will block here if no timers are scheduled

#### 4. Check Phase
```javascript
// setImmediate callbacks are executed here
setImmediate(() => console.log('Immediate'));
```

#### 5. Close Callbacks Phase
```javascript
const server = require('http').createServer();
server.on('close', () => {
  console.log('Server closed'); // Executed in close callbacks phase
});
```

## 🔄 Event Loop in Action

### Example: Understanding Execution Order
```javascript
console.log('Start');

setTimeout(() => console.log('Timer'), 0);
setImmediate(() => console.log('Immediate'));

process.nextTick(() => console.log('Next Tick'));
Promise.resolve().then(() => console.log('Promise'));

console.log('End');

// Output:
// Start
// End
// Next Tick
// Promise
// Timer (or Immediate - order can vary)
// Immediate (or Timer - order can vary)
```

### Microtasks vs Macrotasks

**Microtasks** (Higher Priority):
- `process.nextTick()`
- `Promise.then()`
- `queueMicrotask()`

**Macrotasks** (Lower Priority):
- `setTimeout()`
- `setInterval()`
- `setImmediate()`
- I/O operations

```javascript
// Microtasks are executed before macrotasks
setTimeout(() => console.log('Macro 1'), 0);
Promise.resolve().then(() => console.log('Micro 1'));
setTimeout(() => console.log('Macro 2'), 0);
Promise.resolve().then(() => console.log('Micro 2'));

// Output: Micro 1, Micro 2, Macro 1, Macro 2
```

## 🧵 Thread Pool and libuv

### What Uses the Thread Pool?
```javascript
const fs = require('fs');
const crypto = require('crypto');

// File system operations use thread pool
fs.readFile('large-file.txt', (err, data) => {
  console.log('File operation complete');
});

// CPU-intensive crypto operations use thread pool
crypto.pbkdf2('secret', 'salt', 100000, 64, 'sha512', (err, key) => {
  console.log('Crypto operation complete');
});

// Network operations do NOT use thread pool (they use system async APIs)
const http = require('http');
http.get('http://example.com', (res) => {
  console.log('HTTP request complete');
});
```

### Thread Pool Size
```javascript
// Default thread pool size is 4
// You can change it with environment variable
process.env.UV_THREADPOOL_SIZE = 8;

// Or check current size
console.log('Thread pool size:', process.env.UV_THREADPOOL_SIZE || 4);
```

## 🚫 Blocking vs Non-Blocking Operations

### Blocking Operations (Avoid in Production)
```javascript
const fs = require('fs');

console.log('Before blocking operation');

// This blocks the entire event loop!
const data = fs.readFileSync('large-file.txt', 'utf8');
console.log('File content length:', data.length);

console.log('After blocking operation');
```

### Non-Blocking Operations (Preferred)
```javascript
const fs = require('fs');

console.log('Before non-blocking operation');

// This doesn't block the event loop
fs.readFile('large-file.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Error reading file:', err);
    return;
  }
  console.log('File content length:', data.length);
});

console.log('After non-blocking operation');
```

### Async/Await with Promises
```javascript
const fs = require('fs').promises;

async function readFileAsync() {
  console.log('Before async operation');
  
  try {
    const data = await fs.readFile('large-file.txt', 'utf8');
    console.log('File content length:', data.length);
  } catch (err) {
    console.error('Error reading file:', err);
  }
  
  console.log('After async operation');
}

readFileAsync();
console.log('This runs immediately');
```

## 🔍 Debugging and Profiling the Event Loop

### Event Loop Lag Detection
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

### Using process.hrtime for High-Resolution Timing
```javascript
function timeAsyncOperation() {
  const start = process.hrtime.bigint();
  
  setTimeout(() => {
    const end = process.hrtime.bigint();
    const duration = Number(end - start) / 1000000; // Convert to milliseconds
    console.log(`Operation took: ${duration.toFixed(2)}ms`);
  }, 100);
}

timeAsyncOperation();
```

### Event Loop Monitoring
```javascript
const { monitorEventLoopDelay } = require('perf_hooks');

const histogram = monitorEventLoopDelay({ resolution: 20 });
histogram.enable();

setTimeout(() => {
  console.log('Event Loop Delay Statistics:');
  console.log(`Min: ${histogram.min}ns`);
  console.log(`Max: ${histogram.max}ns`);
  console.log(`Mean: ${histogram.mean}ns`);
  console.log(`Std Dev: ${histogram.stddev}ns`);
  
  histogram.disable();
}, 5000);
```

## 🏋️ Hands-on Exercises

### Exercise 1: Event Loop Order Prediction
Predict the output of this code and then run it to verify:

```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);
setTimeout(() => console.log('3'), 0);

setImmediate(() => console.log('4'));
setImmediate(() => console.log('5'));

process.nextTick(() => console.log('6'));
process.nextTick(() => console.log('7'));

Promise.resolve().then(() => console.log('8'));
Promise.resolve().then(() => console.log('9'));

console.log('10');
```

### Exercise 2: Thread Pool Experiment
Create a script that demonstrates thread pool usage:

```javascript
const crypto = require('crypto');
const { performance } = require('perf_hooks');

function cryptoOperation(id) {
  const start = performance.now();
  
  crypto.pbkdf2('secret', 'salt', 100000, 64, 'sha512', (err, key) => {
    const duration = performance.now() - start;
    console.log(`Crypto ${id} completed in ${duration.toFixed(2)}ms`);
  });
}

// Start multiple crypto operations
for (let i = 1; i <= 8; i++) {
  cryptoOperation(i);
}
```

### Exercise 3: Event Loop Blocking Detection
Create a function that detects when the event loop is blocked:

```javascript
function createBlockingDetector(threshold = 100) {
  let lastCheck = Date.now();
  
  function check() {
    const now = Date.now();
    const lag = now - lastCheck - 10; // Expected 10ms interval
    
    if (lag > threshold) {
      console.warn(`Event loop blocked for ${lag}ms`);
    }
    
    lastCheck = now;
    setTimeout(check, 10);
  }
  
  check();
}

createBlockingDetector(50);

// Test with blocking operation
setTimeout(() => {
  const start = Date.now();
  while (Date.now() - start < 200) {
    // Blocking operation
  }
}, 2000);
```

## 🎯 Challenge: Event Loop Performance Monitor

Build a comprehensive event loop monitoring system:

**Requirements:**
1. **Lag Monitoring** - Detect event loop delays
2. **Phase Timing** - Measure time spent in different phases
3. **Memory Usage** - Track memory consumption
4. **Active Handles** - Monitor active timers and handles
5. **Reporting** - Generate performance reports

**Implementation Structure:**
```javascript
class EventLoopMonitor {
  constructor(options = {}) {
    this.interval = options.interval || 1000;
    this.threshold = options.threshold || 100;
    this.isRunning = false;
  }
  
  start() {
    // Implement monitoring logic
  }
  
  stop() {
    // Stop monitoring
  }
  
  getReport() {
    // Generate performance report
  }
  
  measureLag() {
    // Measure event loop lag
  }
  
  trackMemory() {
    // Track memory usage
  }
  
  countActiveHandles() {
    // Count active handles and requests
  }
}
```

## 🔧 Performance Best Practices

### 1. Avoid Blocking Operations
```javascript
// ❌ Bad - Blocks event loop
const data = fs.readFileSync('file.txt');

// ✅ Good - Non-blocking
fs.readFile('file.txt', (err, data) => {
  // Handle data
});

// ✅ Better - Promise-based
const data = await fs.promises.readFile('file.txt');
```

### 2. Use setImmediate for CPU-Intensive Tasks
```javascript
function processLargeArray(array, callback) {
  if (array.length === 0) {
    return callback();
  }
  
  // Process one item
  processItem(array.shift());
  
  // Yield control back to event loop
  setImmediate(() => processLargeArray(array, callback));
}
```

### 3. Monitor Event Loop Health
```javascript
const eventLoopUtilization = require('perf_hooks').eventLoopUtilization;

setInterval(() => {
  const elu = eventLoopUtilization();
  console.log(`Event loop utilization: ${(elu.utilization * 100).toFixed(2)}%`);
}, 5000);
```

## 📚 Additional Resources

### Essential Reading:
- [Node.js Event Loop Documentation](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
- [libuv Documentation](http://docs.libuv.org/)
- [V8 JavaScript Engine](https://v8.dev/)

### Advanced Topics:
- [Event Loop Best Practices](https://nodejs.org/en/docs/guides/dont-block-the-event-loop/)
- [Performance Timing API](https://nodejs.org/api/perf_hooks.html)
- [Worker Threads](https://nodejs.org/api/worker_threads.html)

### Tools:
- [clinic.js](https://clinicjs.org/) - Performance profiling
- [0x](https://github.com/davidmarkclements/0x) - Flamegraph profiler
- [autocannon](https://github.com/mcollina/autocannon) - HTTP benchmarking

## ✅ Module Completion Checklist

Before moving to the next module, make sure you can:
- [ ] Explain Node.js architecture components
- [ ] Understand event loop phases and execution order
- [ ] Distinguish between blocking and non-blocking operations
- [ ] Use microtasks and macrotasks appropriately
- [ ] Monitor event loop performance
- [ ] Debug event loop issues
- [ ] Write efficient asynchronous code
- [ ] Understand thread pool usage

## 🚀 What's Next?

In **Module 02**, we'll explore:
- Node.js module system (CommonJS and ES modules)
- Package management with npm and yarn
- Creating and publishing packages
- Module resolution and caching

---

**Ready to continue?** 👉 **[Module 02: Modules and Package Management](../02-modules-packages/README.md)**

---

*Estimated completion time: 8-10 hours*
