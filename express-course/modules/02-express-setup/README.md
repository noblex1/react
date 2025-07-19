# Module 02: Express.js Setup and Basic Server

**Duration:** 1 Week | **Difficulty:** Beginner

## 🎯 Learning Objectives

By the end of this module, you will:
- ✅ Understand what Express.js is and why it's the most popular Node.js framework
- ✅ Set up Express.js applications from scratch
- ✅ Create basic Express servers and handle HTTP requests
- ✅ Understand Express application structure and organization
- ✅ Handle different HTTP methods (GET, POST, PUT, DELETE)
- ✅ Work with request and response objects
- ✅ Set up development workflows with nodemon

## 📚 What is Express.js?

**Express.js** is a minimal and flexible Node.js web application framework that provides a robust set of features for web and mobile applications. Think of it as a layer on top of Node.js that makes building web servers much easier.

### Why Express.js?
- **Simplicity** - Minimal setup, easy to learn
- **Flexibility** - Unopinionated, you choose your tools
- **Performance** - Fast and lightweight
- **Ecosystem** - Huge community and middleware library
- **Industry Standard** - Used by Netflix, Uber, WhatsApp, and more

### Express vs Raw Node.js
**Raw Node.js HTTP Server:**
```javascript
const http = require('http');
const url = require('url');

const server = http.createServer((req, res) => {
  const pathname = url.parse(req.url).pathname;
  
  if (pathname === '/' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end('<h1>Home Page</h1>');
  } else if (pathname === '/about' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end('<h1>About Page</h1>');
  } else {
    res.writeHead(404, { 'Content-Type': 'text/html' });
    res.end('<h1>Page Not Found</h1>');
  }
});

server.listen(3000);
```

**Express.js Server:**
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('<h1>Home Page</h1>');
});

app.get('/about', (req, res) => {
  res.send('<h1>About Page</h1>');
});

app.listen(3000);
```

Much cleaner and easier to read!

## 🛠️ Setting Up Express.js

### Step 1: Initialize a New Project
```bash
# Create a new directory
mkdir my-express-app
cd my-express-app

# Initialize npm project
npm init -y
```

### Step 2: Install Express
```bash
# Install Express as a dependency
npm install express

# Install nodemon as a development dependency
npm install --save-dev nodemon
```

### Step 3: Update package.json Scripts
```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  }
}
```

### Step 4: Create Your First Express Server
Create `server.js`:
```javascript
const express = require('express');

// Create Express application
const app = express();

// Define a route
app.get('/', (req, res) => {
  res.send('Hello from Express!');
});

// Start the server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

### Step 5: Run Your Server
```bash
# Development mode (auto-restart on changes)
npm run dev

# Production mode
npm start
```

Visit `http://localhost:3000` to see your Express server in action!

## 🛣️ Basic Routing

### HTTP Methods
Express provides methods corresponding to HTTP verbs:

```javascript
const express = require('express');
const app = express();

// GET request
app.get('/', (req, res) => {
  res.send('GET request to homepage');
});

// POST request
app.post('/', (req, res) => {
  res.send('POST request to homepage');
});

// PUT request
app.put('/user', (req, res) => {
  res.send('PUT request to /user');
});

// DELETE request
app.delete('/user', (req, res) => {
  res.send('DELETE request to /user');
});

app.listen(3000);
```

### Route Parameters
```javascript
// Route with parameter
app.get('/users/:id', (req, res) => {
  const userId = req.params.id;
  res.send(`User ID: ${userId}`);
});

// Multiple parameters
app.get('/users/:id/posts/:postId', (req, res) => {
  const { id, postId } = req.params;
  res.send(`User: ${id}, Post: ${postId}`);
});

// Optional parameters
app.get('/posts/:year/:month?', (req, res) => {
  const { year, month } = req.params;
  res.send(`Year: ${year}, Month: ${month || 'All'}`);
});
```

### Query Parameters
```javascript
// URL: /search?q=nodejs&category=tutorial
app.get('/search', (req, res) => {
  const { q, category } = req.query;
  res.send(`Search: ${q}, Category: ${category}`);
});
```

## 📨 Request and Response Objects

### Request Object (req)
The request object represents the HTTP request and contains properties for the request query string, parameters, body, HTTP headers, and more.

```javascript
app.get('/request-info', (req, res) => {
  const requestInfo = {
    method: req.method,
    url: req.url,
    headers: req.headers,
    params: req.params,
    query: req.query,
    ip: req.ip,
    userAgent: req.get('User-Agent')
  };
  
  res.json(requestInfo);
});
```

### Response Object (res)
The response object represents the HTTP response that Express sends when it gets an HTTP request.

```javascript
app.get('/response-examples', (req, res) => {
  // Send plain text
  // res.send('Hello World');
  
  // Send JSON
  // res.json({ message: 'Hello World', status: 'success' });
  
  // Send with status code
  // res.status(201).json({ message: 'Created successfully' });
  
  // Set headers
  res.set('X-Custom-Header', 'MyValue');
  res.json({ message: 'Response with custom header' });
});
```

### Common Response Methods
```javascript
// Send different types of responses
app.get('/responses', (req, res) => {
  const type = req.query.type;
  
  switch (type) {
    case 'text':
      res.send('Plain text response');
      break;
    case 'json':
      res.json({ message: 'JSON response', data: [1, 2, 3] });
      break;
    case 'html':
      res.send('<h1>HTML Response</h1><p>This is HTML</p>');
      break;
    case 'status':
      res.status(404).json({ error: 'Not found' });
      break;
    case 'redirect':
      res.redirect('/');
      break;
    default:
      res.json({ message: 'Default response' });
  }
});
```

## 🏗️ Application Structure

### Basic Project Structure
```
my-express-app/
├── package.json
├── server.js          # Main server file
├── routes/            # Route definitions
│   ├── users.js
│   └── posts.js
├── controllers/       # Route handlers
│   ├── userController.js
│   └── postController.js
├── models/           # Data models
├── middleware/       # Custom middleware
├── config/          # Configuration files
├── public/          # Static files (CSS, JS, images)
└── views/           # Template files
```

### Organizing Routes
Create `routes/users.js`:
```javascript
const express = require('express');
const router = express.Router();

// GET /users
router.get('/', (req, res) => {
  res.json({ message: 'Get all users' });
});

// GET /users/:id
router.get('/:id', (req, res) => {
  const { id } = req.params;
  res.json({ message: `Get user ${id}` });
});

// POST /users
router.post('/', (req, res) => {
  res.json({ message: 'Create new user' });
});

module.exports = router;
```

Use routes in `server.js`:
```javascript
const express = require('express');
const userRoutes = require('./routes/users');

const app = express();

// Use routes
app.use('/users', userRoutes);

app.listen(3000);
```

## 🏋️ Hands-on Exercises

### Exercise 1: Personal API Server
Create an Express server with the following routes:
- `GET /` - Welcome message
- `GET /about` - Information about yourself
- `GET /skills` - List of your technical skills
- `GET /projects` - List of your projects
- `GET /contact` - Your contact information

### Exercise 2: Calculator API
Build a calculator API with these endpoints:
- `GET /calc/add/:a/:b` - Add two numbers
- `GET /calc/subtract/:a/:b` - Subtract two numbers
- `GET /calc/multiply/:a/:b` - Multiply two numbers
- `GET /calc/divide/:a/:b` - Divide two numbers (handle division by zero)

### Exercise 3: Blog API Structure
Create a basic blog API structure with routes for:
- `GET /posts` - Get all posts
- `GET /posts/:id` - Get specific post
- `POST /posts` - Create new post
- `PUT /posts/:id` - Update post
- `DELETE /posts/:id` - Delete post

For now, just return mock JSON responses.

## 🎯 Challenge: Task Management API (Basic)

Build the foundation for a task management API that will complement your React Todo app:

**Requirements:**
1. **GET /api/tasks** - Return list of tasks
2. **GET /api/tasks/:id** - Return specific task
3. **POST /api/tasks** - Create new task
4. **PUT /api/tasks/:id** - Update task
5. **DELETE /api/tasks/:id** - Delete task

**Task Data Structure:**
```javascript
const tasks = [
  {
    id: 1,
    title: "Learn Express.js",
    description: "Complete the Express.js course",
    completed: false,
    createdAt: "2024-01-15T10:00:00Z",
    updatedAt: "2024-01-15T10:00:00Z"
  }
];
```

**Features to Implement:**
- Proper HTTP status codes
- JSON responses
- Error handling for invalid IDs
- Input validation (basic)

**Bonus Features:**
- Query parameters for filtering (completed/pending)
- Pagination support
- Search functionality
- Sorting options

## 🔧 Development Tools and Workflow

### Using Nodemon for Development
```bash
# Install nodemon globally
npm install -g nodemon

# Or use npx (no global install needed)
npx nodemon server.js

# Or add to package.json scripts
"scripts": {
  "dev": "nodemon server.js"
}
```

### Environment Variables
Create `.env` file:
```
PORT=3000
NODE_ENV=development
```

Install and use dotenv:
```bash
npm install dotenv
```

```javascript
require('dotenv').config();

const PORT = process.env.PORT || 3000;
const NODE_ENV = process.env.NODE_ENV || 'development';

console.log(`Running in ${NODE_ENV} mode on port ${PORT}`);
```

### Basic Error Handling
```javascript
const express = require('express');
const app = express();

// Routes
app.get('/', (req, res) => {
  res.json({ message: 'Hello World' });
});

// 404 handler (should be last)
app.use('*', (req, res) => {
  res.status(404).json({
    error: 'Route not found',
    message: `Cannot ${req.method} ${req.originalUrl}`
  });
});

// Error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: 'Something went wrong!',
    message: err.message
  });
});

app.listen(3000);
```

## 📊 Testing Your API

### Using curl
```bash
# GET request
curl http://localhost:3000/

# POST request with data
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"name": "John Doe", "email": "john@example.com"}'

# GET with query parameters
curl "http://localhost:3000/search?q=nodejs&category=tutorial"
```

### Using Postman or Thunder Client
1. Install Postman or Thunder Client VS Code extension
2. Create requests for each endpoint
3. Test different HTTP methods
4. Save requests in collections for reuse

## 📚 Additional Resources

### Essential Reading:
- [Express.js Official Documentation](https://expressjs.com/)
- [Express.js Guide](https://expressjs.com/en/guide/routing.html)
- [HTTP Status Codes](https://httpstatuses.com/)

### Helpful Tools:
- [Postman](https://www.postman.com/) - API development and testing
- [Insomnia](https://insomnia.rest/) - REST client
- [Thunder Client](https://www.thunderclient.com/) - VS Code extension

### Videos:
- [Express.js Crash Course](https://www.youtube.com/watch?v=L72fhGm1tfE)
- [Building REST APIs with Express](https://www.youtube.com/watch?v=pKd0Rpw7O48)

## ✅ Module Completion Checklist

Before moving to the next module, make sure you can:
- [ ] Set up Express.js applications from scratch
- [ ] Create basic routes with different HTTP methods
- [ ] Handle route parameters and query strings
- [ ] Use request and response objects effectively
- [ ] Organize routes in separate files
- [ ] Set up development workflow with nodemon
- [ ] Handle basic errors and 404 responses
- [ ] Test APIs using Postman or similar tools
- [ ] Understand Express application structure

## 🚀 What's Next?

In **Module 03**, we'll explore:
- Advanced routing patterns and techniques
- Route parameters, wildcards, and regular expressions
- Router-level middleware
- Building modular route handlers

---

**Ready to continue?** 👉 **[Module 03: Routing and Request Handling](../03-routing/README.md)**

---

*Estimated completion time: 6-8 hours*
