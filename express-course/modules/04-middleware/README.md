# Module 04: Middleware and Request Processing

**Duration:** 1 Week | **Difficulty:** Intermediate

## 🎯 Learning Objectives

By the end of this module, you will:
- ✅ Understand what middleware is and how it works in Express
- ✅ Create custom middleware functions for various purposes
- ✅ Use built-in and third-party middleware effectively
- ✅ Implement error handling middleware
- ✅ Control middleware execution flow
- ✅ Build authentication and authorization middleware
- ✅ Create logging and request processing middleware

## 📚 What is Middleware?

**Middleware** functions are functions that have access to the request object (`req`), the response object (`res`), and the next middleware function in the application's request-response cycle, commonly denoted by a variable named `next`.

### Middleware Analogy
Think of middleware like security checkpoints at an airport:
- **Request** = Passenger
- **Middleware** = Security checkpoints, passport control, boarding gate
- **Response** = Getting on the plane
- **next()** = Moving to the next checkpoint

Each checkpoint can:
- Inspect the passenger (request)
- Add stamps/tags (modify request)
- Allow passage (call next())
- Stop the passenger (send response)

## 🔧 How Middleware Works

### Basic Middleware Structure
```javascript
const express = require('express');
const app = express();

// Basic middleware function
const myMiddleware = (req, res, next) => {
  console.log('Middleware executed!');
  console.log('Time:', Date.now());
  next(); // Pass control to the next middleware
};

// Use middleware
app.use(myMiddleware);

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(3000);
```

### Middleware Execution Flow
```javascript
const app = express();

// Middleware 1
app.use((req, res, next) => {
  console.log('Middleware 1');
  next();
});

// Middleware 2
app.use((req, res, next) => {
  console.log('Middleware 2');
  next();
});

// Route handler
app.get('/', (req, res) => {
  console.log('Route handler');
  res.send('Response sent');
});

// This will print:
// Middleware 1
// Middleware 2
// Route handler
```

## 🛠️ Types of Middleware

### 1. Application-Level Middleware
```javascript
const app = express();

// Runs for all routes
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
  next();
});

// Runs for specific path
app.use('/api', (req, res, next) => {
  console.log('API route accessed');
  next();
});

// Runs for specific HTTP method and path
app.use('/users', (req, res, next) => {
  console.log('Users route accessed');
  next();
});
```

### 2. Router-Level Middleware
```javascript
const express = require('express');
const router = express.Router();

// Router-level middleware
router.use((req, res, next) => {
  console.log('Router middleware');
  next();
});

// Specific to this router
router.use('/users/:id', (req, res, next) => {
  console.log('User ID:', req.params.id);
  next();
});

router.get('/users/:id', (req, res) => {
  res.json({ userId: req.params.id });
});

app.use('/api', router);
```

### 3. Built-in Middleware
```javascript
const app = express();

// Parse JSON bodies
app.use(express.json());

// Parse URL-encoded bodies
app.use(express.urlencoded({ extended: true }));

// Serve static files
app.use(express.static('public'));

// Example usage
app.post('/data', (req, res) => {
  console.log('JSON body:', req.body);
  res.json({ received: req.body });
});
```

### 4. Third-Party Middleware
```bash
npm install cors helmet morgan compression
```

```javascript
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');
const compression = require('compression');

const app = express();

// Security middleware
app.use(helmet());

// CORS middleware
app.use(cors());

// Logging middleware
app.use(morgan('combined'));

// Compression middleware
app.use(compression());

// Built-in middleware
app.use(express.json());
```

## 🔨 Creating Custom Middleware

### Request Logging Middleware
```javascript
const requestLogger = (req, res, next) => {
  const timestamp = new Date().toISOString();
  const method = req.method;
  const url = req.url;
  const ip = req.ip;
  const userAgent = req.get('User-Agent');
  
  console.log(`[${timestamp}] ${method} ${url} - IP: ${ip} - ${userAgent}`);
  
  next();
};

app.use(requestLogger);
```

### Request Timing Middleware
```javascript
const requestTimer = (req, res, next) => {
  req.startTime = Date.now();
  
  // Override res.end to calculate duration
  const originalEnd = res.end;
  res.end = function(...args) {
    const duration = Date.now() - req.startTime;
    console.log(`Request completed in ${duration}ms`);
    originalEnd.apply(this, args);
  };
  
  next();
};

app.use(requestTimer);
```

### Request ID Middleware
```javascript
const { v4: uuidv4 } = require('uuid'); // npm install uuid

const requestId = (req, res, next) => {
  req.id = uuidv4();
  res.set('X-Request-ID', req.id);
  next();
};

app.use(requestId);

app.get('/', (req, res) => {
  res.json({ 
    message: 'Hello World',
    requestId: req.id 
  });
});
```

### Rate Limiting Middleware
```javascript
const rateLimit = {};

const rateLimiter = (maxRequests = 100, windowMs = 15 * 60 * 1000) => {
  return (req, res, next) => {
    const ip = req.ip;
    const now = Date.now();
    
    if (!rateLimit[ip]) {
      rateLimit[ip] = { count: 1, resetTime: now + windowMs };
      return next();
    }
    
    if (now > rateLimit[ip].resetTime) {
      rateLimit[ip] = { count: 1, resetTime: now + windowMs };
      return next();
    }
    
    if (rateLimit[ip].count >= maxRequests) {
      return res.status(429).json({
        error: 'Too many requests',
        message: 'Rate limit exceeded'
      });
    }
    
    rateLimit[ip].count++;
    next();
  };
};

// Use rate limiter
app.use('/api', rateLimiter(50, 15 * 60 * 1000)); // 50 requests per 15 minutes
```

## 🔐 Authentication Middleware

### Basic API Key Authentication
```javascript
const apiKeyAuth = (req, res, next) => {
  const apiKey = req.get('X-API-Key') || req.query.apiKey;
  
  if (!apiKey) {
    return res.status(401).json({
      error: 'API key required',
      message: 'Please provide an API key in X-API-Key header or apiKey query parameter'
    });
  }
  
  // In real app, validate against database
  const validApiKeys = ['your-secret-api-key', 'another-valid-key'];
  
  if (!validApiKeys.includes(apiKey)) {
    return res.status(401).json({
      error: 'Invalid API key',
      message: 'The provided API key is not valid'
    });
  }
  
  // Add user info to request (if needed)
  req.user = { apiKey, role: 'user' };
  next();
};

// Protect specific routes
app.get('/api/protected', apiKeyAuth, (req, res) => {
  res.json({ 
    message: 'This is a protected route',
    user: req.user 
  });
});
```

### Role-Based Authorization
```javascript
const authorize = (roles = []) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({
        error: 'Authentication required',
        message: 'Please authenticate first'
      });
    }
    
    if (roles.length && !roles.includes(req.user.role)) {
      return res.status(403).json({
        error: 'Insufficient permissions',
        message: `Required roles: ${roles.join(', ')}`
      });
    }
    
    next();
  };
};

// Usage
app.get('/api/admin', 
  apiKeyAuth, 
  authorize(['admin']), 
  (req, res) => {
    res.json({ message: 'Admin only content' });
  }
);

app.get('/api/user', 
  apiKeyAuth, 
  authorize(['user', 'admin']), 
  (req, res) => {
    res.json({ message: 'User content' });
  }
);
```

## ❌ Error Handling Middleware

### Basic Error Handler
```javascript
const errorHandler = (err, req, res, next) => {
  console.error('Error:', err.stack);
  
  // Default error
  let error = {
    message: err.message || 'Internal Server Error',
    status: err.status || 500
  };
  
  // Handle specific error types
  if (err.name === 'ValidationError') {
    error.status = 400;
    error.message = 'Validation Error';
    error.details = err.details;
  }
  
  if (err.name === 'CastError') {
    error.status = 400;
    error.message = 'Invalid ID format';
  }
  
  // Don't leak error details in production
  if (process.env.NODE_ENV === 'production') {
    delete err.stack;
  }
  
  res.status(error.status).json({
    error: error.message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
};

// Use error handler (must be last middleware)
app.use(errorHandler);
```

### Async Error Handler
```javascript
const asyncHandler = (fn) => {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};

// Usage with async route handlers
app.get('/api/async-route', asyncHandler(async (req, res) => {
  // This will automatically catch any errors and pass to error handler
  const data = await someAsyncOperation();
  res.json(data);
}));
```

### 404 Handler
```javascript
const notFoundHandler = (req, res, next) => {
  const error = new Error(`Route ${req.originalUrl} not found`);
  error.status = 404;
  next(error);
};

// Use before error handler
app.use(notFoundHandler);
app.use(errorHandler);
```

## 🏋️ Hands-on Exercises

### Exercise 1: Request Analytics Middleware
Create middleware that tracks:
- Request count per IP address
- Most requested endpoints
- Average response time
- Request methods distribution

### Exercise 2: Content Validation Middleware
Build middleware that:
- Validates JSON content type for POST/PUT requests
- Checks for required fields
- Sanitizes input data
- Handles malformed JSON

### Exercise 3: Security Middleware Suite
Create a security middleware that:
- Adds security headers
- Validates request size
- Blocks suspicious requests
- Implements basic DDoS protection

## 🎯 Challenge: Complete Middleware Stack

Build a comprehensive middleware stack for your task management API:

**Required Middleware:**
1. **Request Logger** - Log all requests with timestamp, method, URL, IP
2. **Request ID** - Add unique ID to each request
3. **Rate Limiter** - Limit requests per IP
4. **API Key Authentication** - Protect API endpoints
5. **Request Validator** - Validate request data
6. **Error Handler** - Handle all errors gracefully
7. **404 Handler** - Handle unknown routes

**Advanced Middleware:**
1. **Request Timer** - Measure and log response times
2. **CORS Handler** - Handle cross-origin requests
3. **Compression** - Compress responses
4. **Security Headers** - Add security headers
5. **Request Size Limiter** - Limit request body size

**Implementation Requirements:**
```javascript
const app = express();

// Security middleware
app.use(helmet());
app.use(cors());

// Request processing
app.use(requestLogger);
app.use(requestId);
app.use(requestTimer);

// Rate limiting
app.use('/api', rateLimiter(100, 15 * 60 * 1000));

// Body parsing
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// Authentication
app.use('/api', apiKeyAuth);

// Routes
app.use('/api/tasks', taskRoutes);

// Error handling
app.use(notFoundHandler);
app.use(errorHandler);
```

## 🔧 Middleware Best Practices

### 1. Order Matters
```javascript
// Correct order
app.use(helmet());           // Security first
app.use(cors());             // CORS handling
app.use(morgan('combined')); // Logging
app.use(express.json());     // Body parsing
app.use(authenticate);       // Authentication
app.use('/api', routes);     // Routes
app.use(notFound);          // 404 handler
app.use(errorHandler);      // Error handler (last)
```

### 2. Conditional Middleware
```javascript
// Only in development
if (process.env.NODE_ENV === 'development') {
  app.use(morgan('dev'));
}

// Only in production
if (process.env.NODE_ENV === 'production') {
  app.use(compression());
}
```

### 3. Middleware Configuration
```javascript
const middlewareConfig = {
  cors: {
    origin: process.env.ALLOWED_ORIGINS?.split(',') || '*',
    credentials: true
  },
  rateLimit: {
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: process.env.NODE_ENV === 'production' ? 100 : 1000
  },
  helmet: {
    contentSecurityPolicy: false // Disable for API
  }
};

app.use(cors(middlewareConfig.cors));
app.use(rateLimit(middlewareConfig.rateLimit));
app.use(helmet(middlewareConfig.helmet));
```

## 📚 Additional Resources

### Essential Reading:
- [Express Middleware Guide](https://expressjs.com/en/guide/using-middleware.html)
- [Writing Middleware](https://expressjs.com/en/guide/writing-middleware.html)
- [Error Handling](https://expressjs.com/en/guide/error-handling.html)

### Popular Middleware Packages:
- [helmet](https://helmetjs.github.io/) - Security headers
- [cors](https://github.com/expressjs/cors) - Cross-origin resource sharing
- [morgan](https://github.com/expressjs/morgan) - HTTP request logger
- [compression](https://github.com/expressjs/compression) - Response compression
- [express-rate-limit](https://github.com/nfriedly/express-rate-limit) - Rate limiting

### Security Resources:
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Express Security Best Practices](https://expressjs.com/en/advanced/best-practice-security.html)

## ✅ Module Completion Checklist

Before moving to the next module, make sure you can:
- [ ] Understand how middleware works in Express
- [ ] Create custom middleware functions
- [ ] Use built-in and third-party middleware
- [ ] Implement authentication and authorization middleware
- [ ] Create error handling middleware
- [ ] Control middleware execution flow with next()
- [ ] Organize middleware in the correct order
- [ ] Handle async errors in middleware
- [ ] Build a complete middleware stack for an API

## 🚀 What's Next?

In **Module 05**, we'll explore:
- RESTful API design principles
- Building complete REST APIs
- HTTP status codes and best practices
- API versioning and documentation

---

**Ready to continue?** 👉 **[Module 05: Building REST APIs](../05-rest-apis/README.md)**

---

*Estimated completion time: 8-10 hours*
