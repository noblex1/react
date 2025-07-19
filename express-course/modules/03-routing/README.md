# Module 03: Routing and Request Handling

**Duration:** 1 Week | **Difficulty:** Beginner to Intermediate

## 🎯 Learning Objectives

By the end of this module, you will:
- ✅ Master advanced routing patterns and techniques
- ✅ Use route parameters, wildcards, and regular expressions
- ✅ Implement router-level middleware
- ✅ Handle different types of request data (JSON, form data, files)
- ✅ Build modular and organized route handlers
- ✅ Implement proper request validation
- ✅ Create RESTful API endpoints

## 🛣️ Advanced Routing Patterns

### Route Parameters with Validation
```javascript
const express = require('express');
const app = express();

// Basic parameter
app.get('/users/:id', (req, res) => {
  const { id } = req.params;
  
  // Validate parameter
  if (isNaN(id)) {
    return res.status(400).json({ error: 'User ID must be a number' });
  }
  
  res.json({ userId: parseInt(id) });
});

// Multiple parameters
app.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  res.json({ 
    message: `User ${userId}, Post ${postId}`,
    userId: parseInt(userId),
    postId: parseInt(postId)
  });
});

// Optional parameters
app.get('/posts/:year/:month?/:day?', (req, res) => {
  const { year, month, day } = req.params;
  res.json({
    year: parseInt(year),
    month: month ? parseInt(month) : null,
    day: day ? parseInt(day) : null
  });
});
```

### Wildcard Routes
```javascript
// Catch-all route (should be last)
app.get('/files/*', (req, res) => {
  const filePath = req.params[0]; // Everything after /files/
  res.json({ requestedFile: filePath });
});

// Example: GET /files/documents/report.pdf
// filePath = "documents/report.pdf"
```

### Regular Expression Routes
```javascript
// Route that matches specific patterns
app.get(/.*fly$/, (req, res) => {
  res.send('Ends with "fly"'); // Matches: butterfly, dragonfly, etc.
});

// Route with specific format
app.get(/^\/users\/(\d+)$/, (req, res) => {
  const userId = req.params[0];
  res.json({ userId: parseInt(userId) });
});
```

### Query Parameters
```javascript
// GET /search?q=nodejs&category=tutorial&page=1&limit=10
app.get('/search', (req, res) => {
  const {
    q = '',           // Search query
    category = 'all', // Category filter
    page = 1,         // Page number
    limit = 10        // Items per page
  } = req.query;
  
  // Validate query parameters
  const pageNum = parseInt(page);
  const limitNum = parseInt(limit);
  
  if (pageNum < 1 || limitNum < 1 || limitNum > 100) {
    return res.status(400).json({ 
      error: 'Invalid pagination parameters' 
    });
  }
  
  res.json({
    query: q,
    category,
    pagination: {
      page: pageNum,
      limit: limitNum,
      offset: (pageNum - 1) * limitNum
    }
  });
});
```

## 🏗️ Express Router

### Creating Modular Routes
Create `routes/users.js`:
```javascript
const express = require('express');
const router = express.Router();

// Middleware specific to this router
router.use((req, res, next) => {
  console.log('User route accessed at:', new Date().toISOString());
  next();
});

// GET /users
router.get('/', (req, res) => {
  res.json({ 
    message: 'Get all users',
    users: [
      { id: 1, name: 'John Doe', email: 'john@example.com' },
      { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
    ]
  });
});

// GET /users/:id
router.get('/:id', (req, res) => {
  const { id } = req.params;
  
  if (isNaN(id)) {
    return res.status(400).json({ error: 'Invalid user ID' });
  }
  
  // Mock user lookup
  const user = { id: parseInt(id), name: 'John Doe', email: 'john@example.com' };
  res.json({ user });
});

// POST /users
router.post('/', (req, res) => {
  const { name, email } = req.body;
  
  // Basic validation
  if (!name || !email) {
    return res.status(400).json({ 
      error: 'Name and email are required' 
    });
  }
  
  // Mock user creation
  const newUser = {
    id: Date.now(),
    name,
    email,
    createdAt: new Date().toISOString()
  };
  
  res.status(201).json({ 
    message: 'User created successfully',
    user: newUser 
  });
});

// PUT /users/:id
router.put('/:id', (req, res) => {
  const { id } = req.params;
  const { name, email } = req.body;
  
  if (isNaN(id)) {
    return res.status(400).json({ error: 'Invalid user ID' });
  }
  
  // Mock user update
  const updatedUser = {
    id: parseInt(id),
    name,
    email,
    updatedAt: new Date().toISOString()
  };
  
  res.json({ 
    message: 'User updated successfully',
    user: updatedUser 
  });
});

// DELETE /users/:id
router.delete('/:id', (req, res) => {
  const { id } = req.params;
  
  if (isNaN(id)) {
    return res.status(400).json({ error: 'Invalid user ID' });
  }
  
  res.json({ 
    message: `User ${id} deleted successfully` 
  });
});

module.exports = router;
```

### Using Router in Main App
```javascript
// server.js
const express = require('express');
const userRoutes = require('./routes/users');
const postRoutes = require('./routes/posts');

const app = express();

// Middleware to parse JSON
app.use(express.json());

// Use routers
app.use('/api/users', userRoutes);
app.use('/api/posts', postRoutes);

// Root route
app.get('/', (req, res) => {
  res.json({ 
    message: 'Welcome to the API',
    endpoints: [
      'GET /api/users',
      'POST /api/users',
      'GET /api/users/:id',
      'PUT /api/users/:id',
      'DELETE /api/users/:id'
    ]
  });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

## 📨 Handling Request Data

### JSON Data
```javascript
const express = require('express');
const app = express();

// Middleware to parse JSON bodies
app.use(express.json());

app.post('/api/data', (req, res) => {
  console.log('Request body:', req.body);
  console.log('Content-Type:', req.get('Content-Type'));
  
  // Validate JSON data
  if (!req.body || Object.keys(req.body).length === 0) {
    return res.status(400).json({ error: 'Request body is required' });
  }
  
  res.json({
    message: 'Data received successfully',
    receivedData: req.body,
    timestamp: new Date().toISOString()
  });
});
```

### Form Data (URL-encoded)
```javascript
// Middleware to parse URL-encoded bodies
app.use(express.urlencoded({ extended: true }));

app.post('/api/form', (req, res) => {
  console.log('Form data:', req.body);
  
  const { name, email, message } = req.body;
  
  if (!name || !email || !message) {
    return res.status(400).json({ 
      error: 'All fields are required' 
    });
  }
  
  res.json({
    message: 'Form submitted successfully',
    data: { name, email, message }
  });
});
```

### Headers and Metadata
```javascript
app.get('/api/request-info', (req, res) => {
  const requestInfo = {
    method: req.method,
    url: req.url,
    originalUrl: req.originalUrl,
    baseUrl: req.baseUrl,
    path: req.path,
    hostname: req.hostname,
    ip: req.ip,
    protocol: req.protocol,
    secure: req.secure,
    headers: req.headers,
    userAgent: req.get('User-Agent'),
    contentType: req.get('Content-Type'),
    params: req.params,
    query: req.query,
    body: req.body
  };
  
  res.json(requestInfo);
});
```

## 🔍 Request Validation

### Basic Validation Function
```javascript
function validateUser(req, res, next) {
  const { name, email, age } = req.body;
  const errors = [];
  
  if (!name || name.trim().length < 2) {
    errors.push('Name must be at least 2 characters long');
  }
  
  if (!email || !email.includes('@')) {
    errors.push('Valid email is required');
  }
  
  if (age && (isNaN(age) || age < 0 || age > 150)) {
    errors.push('Age must be a number between 0 and 150');
  }
  
  if (errors.length > 0) {
    return res.status(400).json({ 
      error: 'Validation failed',
      details: errors 
    });
  }
  
  next();
}

// Use validation middleware
app.post('/api/users', validateUser, (req, res) => {
  res.json({ 
    message: 'User data is valid',
    data: req.body 
  });
});
```

### Advanced Validation with express-validator
```bash
npm install express-validator
```

```javascript
const { body, validationResult } = require('express-validator');

// Validation rules
const userValidationRules = () => {
  return [
    body('name')
      .isLength({ min: 2 })
      .withMessage('Name must be at least 2 characters long')
      .trim(),
    body('email')
      .isEmail()
      .withMessage('Must be a valid email')
      .normalizeEmail(),
    body('age')
      .optional()
      .isInt({ min: 0, max: 150 })
      .withMessage('Age must be between 0 and 150')
  ];
};

// Check validation results
const validate = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({
      error: 'Validation failed',
      details: errors.array()
    });
  }
  next();
};

// Use validation
app.post('/api/users', 
  userValidationRules(), 
  validate, 
  (req, res) => {
    res.json({ 
      message: 'User created successfully',
      data: req.body 
    });
  }
);
```

## 🏋️ Hands-on Exercises

### Exercise 1: Blog API Routes
Create a complete blog API with the following routes:
- `GET /api/posts` - Get all posts (with pagination)
- `GET /api/posts/:id` - Get specific post
- `POST /api/posts` - Create new post
- `PUT /api/posts/:id` - Update post
- `DELETE /api/posts/:id` - Delete post
- `GET /api/posts/search` - Search posts by title/content

Include proper validation and error handling.

### Exercise 2: Product Catalog API
Build a product catalog API with:
- `GET /api/products` - Get products (with filtering and sorting)
- `GET /api/products/:id` - Get specific product
- `GET /api/products/category/:category` - Get products by category
- `POST /api/products` - Add new product
- `PUT /api/products/:id` - Update product
- `DELETE /api/products/:id` - Delete product

### Exercise 3: User Profile Management
Create user profile routes with:
- `GET /api/profile/:userId` - Get user profile
- `PUT /api/profile/:userId` - Update profile
- `POST /api/profile/:userId/avatar` - Upload profile picture
- `GET /api/profile/:userId/posts` - Get user's posts
- `GET /api/profile/search` - Search users by name

## 🎯 Challenge: Complete Task Management API

Expand your task management API with advanced routing:

**Core Routes:**
- `GET /api/tasks` - Get all tasks (with filtering, sorting, pagination)
- `GET /api/tasks/:id` - Get specific task
- `POST /api/tasks` - Create new task
- `PUT /api/tasks/:id` - Update task
- `DELETE /api/tasks/:id` - Delete task
- `PATCH /api/tasks/:id/complete` - Mark task as complete
- `PATCH /api/tasks/:id/incomplete` - Mark task as incomplete

**Advanced Routes:**
- `GET /api/tasks/search` - Search tasks by title/description
- `GET /api/tasks/due/:date` - Get tasks due on specific date
- `GET /api/tasks/overdue` - Get overdue tasks
- `GET /api/tasks/stats` - Get task statistics
- `POST /api/tasks/bulk` - Create multiple tasks
- `DELETE /api/tasks/completed` - Delete all completed tasks

**Query Parameters:**
- `status` - Filter by completion status
- `priority` - Filter by priority level
- `sort` - Sort by date, priority, title
- `page` & `limit` - Pagination
- `search` - Search in title/description

**Validation Requirements:**
- Title: required, 1-100 characters
- Description: optional, max 500 characters
- Priority: low, medium, high
- Due date: valid date format
- Status: pending, completed

## 🔧 Route Organization Best Practices

### Controller Pattern
Create `controllers/userController.js`:
```javascript
const userController = {
  // GET /api/users
  getAllUsers: (req, res) => {
    const { page = 1, limit = 10 } = req.query;
    
    // Mock data
    const users = [
      { id: 1, name: 'John Doe', email: 'john@example.com' },
      { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
    ];
    
    res.json({
      users,
      pagination: {
        page: parseInt(page),
        limit: parseInt(limit),
        total: users.length
      }
    });
  },
  
  // GET /api/users/:id
  getUserById: (req, res) => {
    const { id } = req.params;
    
    if (isNaN(id)) {
      return res.status(400).json({ error: 'Invalid user ID' });
    }
    
    // Mock user lookup
    const user = { id: parseInt(id), name: 'John Doe', email: 'john@example.com' };
    res.json({ user });
  },
  
  // POST /api/users
  createUser: (req, res) => {
    const { name, email } = req.body;
    
    const newUser = {
      id: Date.now(),
      name,
      email,
      createdAt: new Date().toISOString()
    };
    
    res.status(201).json({
      message: 'User created successfully',
      user: newUser
    });
  }
};

module.exports = userController;
```

Use in routes:
```javascript
const express = require('express');
const userController = require('../controllers/userController');
const router = express.Router();

router.get('/', userController.getAllUsers);
router.get('/:id', userController.getUserById);
router.post('/', userController.createUser);

module.exports = router;
```

### Route Documentation
```javascript
/**
 * @route   GET /api/users
 * @desc    Get all users
 * @access  Public
 * @params  page (query) - Page number (default: 1)
 *          limit (query) - Items per page (default: 10)
 * @returns Array of user objects
 */
router.get('/', userController.getAllUsers);

/**
 * @route   POST /api/users
 * @desc    Create new user
 * @access  Public
 * @body    name (string, required) - User's full name
 *          email (string, required) - User's email address
 * @returns Created user object
 */
router.post('/', userController.createUser);
```

## 📚 Additional Resources

### Essential Reading:
- [Express Routing Guide](https://expressjs.com/en/guide/routing.html)
- [Express Router Documentation](https://expressjs.com/en/4x/api.html#router)
- [HTTP Status Codes Reference](https://httpstatuses.com/)

### Validation Libraries:
- [express-validator](https://express-validator.github.io/docs/) - Validation middleware
- [Joi](https://joi.dev/) - Object schema validation
- [Yup](https://github.com/jquense/yup) - Schema validation

### Testing Tools:
- [Postman](https://www.postman.com/) - API testing
- [Insomnia](https://insomnia.rest/) - REST client
- [HTTPie](https://httpie.io/) - Command line HTTP client

## ✅ Module Completion Checklist

Before moving to the next module, make sure you can:
- [ ] Create advanced routing patterns with parameters and wildcards
- [ ] Use Express Router to organize routes modularly
- [ ] Handle different types of request data (JSON, form data)
- [ ] Implement proper request validation
- [ ] Create RESTful API endpoints
- [ ] Use query parameters for filtering and pagination
- [ ] Organize routes using the controller pattern
- [ ] Handle errors and edge cases in routes
- [ ] Test API endpoints using Postman or similar tools

## 🚀 What's Next?

In **Module 04**, we'll explore:
- Middleware concepts and types
- Creating custom middleware functions
- Error handling middleware
- Third-party middleware integration

---

**Ready to continue?** 👉 **[Module 04: Middleware and Request Processing](../04-middleware/README.md)**

---

*Estimated completion time: 8-10 hours*
