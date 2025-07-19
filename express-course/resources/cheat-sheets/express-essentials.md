# Express.js Essentials Cheat Sheet 🚀

## Basic Server Setup

### Minimal Express Server
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Complete Server Setup
```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');

const app = express();

// Security middleware
app.use(helmet());
app.use(cors());

// Logging
app.use(morgan('combined'));

// Body parsing
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.use('/api/users', userRoutes);

// Error handling
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

## Routing

### Basic Routes
```javascript
// HTTP Methods
app.get('/users', (req, res) => {});      // GET
app.post('/users', (req, res) => {});     // POST
app.put('/users/:id', (req, res) => {});  // PUT
app.patch('/users/:id', (req, res) => {}); // PATCH
app.delete('/users/:id', (req, res) => {}); // DELETE

// Route Parameters
app.get('/users/:id', (req, res) => {
  const { id } = req.params;
  res.json({ userId: id });
});

// Multiple Parameters
app.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  res.json({ userId, postId });
});

// Query Parameters
app.get('/search', (req, res) => {
  const { q, page, limit } = req.query;
  res.json({ query: q, page, limit });
});
```

### Express Router
```javascript
// routes/users.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json({ message: 'Get all users' });
});

router.get('/:id', (req, res) => {
  res.json({ message: `Get user ${req.params.id}` });
});

router.post('/', (req, res) => {
  res.json({ message: 'Create user', data: req.body });
});

module.exports = router;

// app.js
const userRoutes = require('./routes/users');
app.use('/api/users', userRoutes);
```

## Middleware

### Basic Middleware
```javascript
// Application-level middleware
app.use((req, res, next) => {
  console.log('Time:', Date.now());
  next();
});

// Route-specific middleware
app.get('/protected', authenticate, (req, res) => {
  res.json({ message: 'Protected route' });
});

// Multiple middleware
app.get('/admin', authenticate, authorize('admin'), (req, res) => {
  res.json({ message: 'Admin only' });
});
```

### Custom Middleware Examples
```javascript
// Request logging
const requestLogger = (req, res, next) => {
  console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
  next();
};

// Authentication
const authenticate = (req, res, next) => {
  const token = req.headers.authorization;
  if (!token) {
    return res.status(401).json({ error: 'Token required' });
  }
  // Verify token logic
  next();
};

// Authorization
const authorize = (role) => {
  return (req, res, next) => {
    if (req.user.role !== role) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
};

// Error handling middleware
const errorHandler = (err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: 'Something went wrong!',
    message: err.message
  });
};
```

## Request and Response

### Request Object Properties
```javascript
app.get('/request-info', (req, res) => {
  const info = {
    method: req.method,           // HTTP method
    url: req.url,                 // URL path
    originalUrl: req.originalUrl, // Original URL
    baseUrl: req.baseUrl,         // Base URL
    path: req.path,               // Path portion
    hostname: req.hostname,       // Host name
    ip: req.ip,                   // Client IP
    protocol: req.protocol,       // http or https
    secure: req.secure,           // true if https
    headers: req.headers,         // Request headers
    params: req.params,           // Route parameters
    query: req.query,             // Query string
    body: req.body,               // Request body
    cookies: req.cookies          // Cookies
  };
  res.json(info);
});
```

### Response Methods
```javascript
// Send responses
res.send('Hello World');                    // Send string
res.json({ message: 'Hello' });            // Send JSON
res.status(404).send('Not Found');         // Set status and send
res.status(201).json({ id: 1 });           // Set status and send JSON

// Set headers
res.set('X-Custom-Header', 'value');
res.cookie('name', 'value');
res.clearCookie('name');

// Redirects
res.redirect('/login');                     // 302 redirect
res.redirect(301, '/new-url');              // 301 redirect

// File operations
res.sendFile(path.join(__dirname, 'file.html'));
res.download('/path/to/file.pdf');
```

## Error Handling

### Basic Error Handling
```javascript
// Async error wrapper
const asyncHandler = (fn) => {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};

// Usage
app.get('/async-route', asyncHandler(async (req, res) => {
  const data = await someAsyncOperation();
  res.json(data);
}));

// 404 handler
app.use('*', (req, res) => {
  res.status(404).json({
    error: 'Route not found',
    message: `Cannot ${req.method} ${req.originalUrl}`
  });
});

// Global error handler (must be last)
app.use((err, req, res, next) => {
  const status = err.status || 500;
  res.status(status).json({
    error: err.message || 'Internal Server Error',
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
});
```

## Validation

### Manual Validation
```javascript
const validateUser = (req, res, next) => {
  const { name, email, age } = req.body;
  const errors = [];
  
  if (!name || name.length < 2) {
    errors.push('Name must be at least 2 characters');
  }
  
  if (!email || !email.includes('@')) {
    errors.push('Valid email is required');
  }
  
  if (age && (isNaN(age) || age < 0)) {
    errors.push('Age must be a positive number');
  }
  
  if (errors.length > 0) {
    return res.status(400).json({ errors });
  }
  
  next();
};
```

### Using express-validator
```javascript
const { body, validationResult } = require('express-validator');

// Validation rules
const userValidation = [
  body('name').isLength({ min: 2 }).trim(),
  body('email').isEmail().normalizeEmail(),
  body('age').optional().isInt({ min: 0 })
];

// Check validation results
const validate = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  next();
};

// Usage
app.post('/users', userValidation, validate, (req, res) => {
  res.json({ message: 'User created', data: req.body });
});
```

## Authentication with JWT

### JWT Setup
```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');

// Generate token
const generateToken = (payload) => {
  return jwt.sign(payload, process.env.JWT_SECRET, { expiresIn: '7d' });
};

// Verify token
const verifyToken = (token) => {
  return jwt.verify(token, process.env.JWT_SECRET);
};

// Hash password
const hashPassword = async (password) => {
  return await bcrypt.hash(password, 12);
};

// Compare password
const comparePassword = async (password, hash) => {
  return await bcrypt.compare(password, hash);
};
```

### Auth Middleware
```javascript
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'Access token required' });
  }
  
  try {
    const decoded = verifyToken(token);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(403).json({ error: 'Invalid token' });
  }
};
```

### Login/Register Routes
```javascript
// Register
app.post('/auth/register', async (req, res) => {
  try {
    const { email, password, name } = req.body;
    
    // Check if user exists
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(409).json({ error: 'User already exists' });
    }
    
    // Hash password and create user
    const hashedPassword = await hashPassword(password);
    const user = await User.create({ email, password: hashedPassword, name });
    
    // Generate token
    const token = generateToken({ userId: user.id, email: user.email });
    
    res.status(201).json({
      message: 'User created successfully',
      token,
      user: { id: user.id, email: user.email, name: user.name }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Login
app.post('/auth/login', async (req, res) => {
  try {
    const { email, password } = req.body;
    
    // Find user
    const user = await User.findOne({ email });
    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Check password
    const isValidPassword = await comparePassword(password, user.password);
    if (!isValidPassword) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Generate token
    const token = generateToken({ userId: user.id, email: user.email });
    
    res.json({
      message: 'Login successful',
      token,
      user: { id: user.id, email: user.email, name: user.name }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

## Database Integration (MongoDB with Mongoose)

### Basic Setup
```javascript
const mongoose = require('mongoose');

// Connect to MongoDB
mongoose.connect(process.env.MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

// User Schema
const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  role: { type: String, enum: ['user', 'admin'], default: 'user' },
  createdAt: { type: Date, default: Date.now }
});

const User = mongoose.model('User', userSchema);
```

### CRUD Operations
```javascript
// Create
app.post('/users', async (req, res) => {
  try {
    const user = new User(req.body);
    await user.save();
    res.status(201).json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// Read (all)
app.get('/users', async (req, res) => {
  try {
    const users = await User.find().select('-password');
    res.json(users);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Read (one)
app.get('/users/:id', async (req, res) => {
  try {
    const user = await User.findById(req.params.id).select('-password');
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Update
app.put('/users/:id', async (req, res) => {
  try {
    const user = await User.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true }
    ).select('-password');
    
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// Delete
app.delete('/users/:id', async (req, res) => {
  try {
    const user = await User.findByIdAndDelete(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json({ message: 'User deleted successfully' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

## Environment Configuration

### .env File
```bash
# Server
PORT=3000
NODE_ENV=development

# Database
MONGODB_URI=mongodb://localhost:27017/myapp

# JWT
JWT_SECRET=your-super-secret-jwt-key
JWT_EXPIRES_IN=7d

# Email (if using)
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your-email@gmail.com
EMAIL_PASS=your-app-password
```

### Using Environment Variables
```javascript
require('dotenv').config();

const config = {
  port: process.env.PORT || 3000,
  mongoUri: process.env.MONGODB_URI || 'mongodb://localhost:27017/myapp',
  jwtSecret: process.env.JWT_SECRET || 'fallback-secret',
  jwtExpiresIn: process.env.JWT_EXPIRES_IN || '7d',
  nodeEnv: process.env.NODE_ENV || 'development'
};

// Use in app
app.listen(config.port, () => {
  console.log(`Server running on port ${config.port}`);
});
```

## Common HTTP Status Codes

```javascript
// Success
200 // OK - Request successful
201 // Created - Resource created
204 // No Content - Request successful, no content to return

// Client Errors
400 // Bad Request - Invalid request data
401 // Unauthorized - Authentication required
403 // Forbidden - Access denied
404 // Not Found - Resource not found
409 // Conflict - Resource already exists
422 // Unprocessable Entity - Validation error

// Server Errors
500 // Internal Server Error - Generic server error
502 // Bad Gateway - Invalid response from upstream
503 // Service Unavailable - Server temporarily unavailable

// Usage examples
res.status(201).json({ message: 'Created' });
res.status(400).json({ error: 'Bad request' });
res.status(404).json({ error: 'Not found' });
res.status(500).json({ error: 'Server error' });
```

## Testing with curl

```bash
# GET request
curl http://localhost:3000/api/users

# POST request with JSON
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "John Doe", "email": "john@example.com"}'

# PUT request with authentication
curl -X PUT http://localhost:3000/api/users/123 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-jwt-token" \
  -d '{"name": "Jane Doe"}'

# DELETE request
curl -X DELETE http://localhost:3000/api/users/123 \
  -H "Authorization: Bearer your-jwt-token"

# Query parameters
curl "http://localhost:3000/api/users?page=1&limit=10&sort=name"
```

---

**Keep this cheat sheet handy while building Express.js applications!** 📚
