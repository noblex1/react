# Project 1: Task Management API 📝

**Difficulty:** Beginner | **Estimated Time:** 10-14 hours | **Technologies:** Express.js, Node.js, JSON Storage

## 🎯 Project Overview

Build a comprehensive REST API for task management that serves as the backend for your React Todo application. This project demonstrates fundamental Express.js concepts including routing, middleware, validation, and data persistence using JSON files.

## 🚀 What You'll Build

A fully functional task management API with the following features:
- ✅ Complete CRUD operations for tasks
- ✅ Task filtering and searching capabilities
- ✅ Request validation and error handling
- ✅ API authentication with API keys
- ✅ Request logging and rate limiting
- ✅ Comprehensive API documentation
- ✅ JSON file-based data persistence
- ✅ RESTful endpoint design

## 🛠️ Technical Requirements

### Core API Endpoints

#### Task Management
```javascript
// Core CRUD operations
GET    /api/tasks              // Get all tasks (with filtering)
GET    /api/tasks/:id          // Get specific task
POST   /api/tasks              // Create new task
PUT    /api/tasks/:id          // Update entire task
PATCH  /api/tasks/:id          // Partial task update
DELETE /api/tasks/:id          // Delete task

// Task status operations
PATCH  /api/tasks/:id/complete   // Mark task as complete
PATCH  /api/tasks/:id/incomplete // Mark task as incomplete

// Bulk operations
POST   /api/tasks/bulk          // Create multiple tasks
DELETE /api/tasks/completed     // Delete all completed tasks
```

#### Advanced Features
```javascript
// Search and filtering
GET    /api/tasks/search?q=keyword        // Search tasks
GET    /api/tasks/filter?status=completed // Filter by status
GET    /api/tasks/due/:date               // Tasks due on date
GET    /api/tasks/overdue                 // Overdue tasks

// Statistics and analytics
GET    /api/tasks/stats                   // Task statistics
GET    /api/tasks/count                   // Task counts by status

// Categories and priorities
GET    /api/tasks/categories              // Get all categories
GET    /api/tasks/priority/:level         // Tasks by priority
```

### Data Models

#### Task Schema
```javascript
{
  "id": "uuid-string",
  "title": "Task title",
  "description": "Task description",
  "completed": false,
  "priority": "medium", // low, medium, high
  "category": "work",
  "dueDate": "2024-07-25T10:00:00Z",
  "tags": ["urgent", "important"],
  "createdAt": "2024-07-20T10:00:00Z",
  "updatedAt": "2024-07-20T10:00:00Z",
  "completedAt": null
}
```

#### API Response Format
```javascript
// Success Response
{
  "success": true,
  "data": {
    "task": { /* task object */ },
    "message": "Task created successfully"
  },
  "meta": {
    "timestamp": "2024-07-20T10:00:00Z",
    "requestId": "uuid-string"
  }
}

// Error Response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Title is required",
    "details": [
      {
        "field": "title",
        "message": "Title must be between 1 and 100 characters"
      }
    ]
  },
  "meta": {
    "timestamp": "2024-07-20T10:00:00Z",
    "requestId": "uuid-string"
  }
}
```

## 📋 Step-by-Step Implementation Guide

### Phase 1: Project Setup and Basic Structure (2-3 hours)

#### 1. Initialize Project
```bash
mkdir task-management-api
cd task-management-api
npm init -y

# Install dependencies
npm install express cors helmet morgan compression dotenv uuid
npm install --save-dev nodemon

# Create project structure
mkdir src
mkdir src/routes src/controllers src/middleware src/utils src/data
touch src/server.js src/app.js
```

#### 2. Basic Server Setup
```javascript
// src/app.js
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');
const compression = require('compression');

const app = express();

// Security middleware
app.use(helmet());
app.use(cors());

// Logging
app.use(morgan('combined'));

// Compression
app.use(compression());

// Body parsing
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));

// Health check
app.get('/health', (req, res) => {
  res.json({ 
    status: 'OK', 
    timestamp: new Date().toISOString(),
    uptime: process.uptime()
  });
});

module.exports = app;
```

#### 3. Data Storage Setup
```javascript
// src/utils/dataStore.js
const fs = require('fs').promises;
const path = require('path');

class DataStore {
  constructor(filename) {
    this.filePath = path.join(__dirname, '../data', filename);
  }

  async read() {
    try {
      const data = await fs.readFile(this.filePath, 'utf8');
      return JSON.parse(data);
    } catch (error) {
      if (error.code === 'ENOENT') {
        return [];
      }
      throw error;
    }
  }

  async write(data) {
    await fs.writeFile(this.filePath, JSON.stringify(data, null, 2));
  }
}

module.exports = DataStore;
```

### Phase 2: Core CRUD Operations (3-4 hours)

#### 1. Task Controller
```javascript
// src/controllers/taskController.js
const { v4: uuidv4 } = require('uuid');
const DataStore = require('../utils/dataStore');

const taskStore = new DataStore('tasks.json');

const taskController = {
  // GET /api/tasks
  getAllTasks: async (req, res, next) => {
    try {
      const tasks = await taskStore.read();
      const { status, priority, category, page = 1, limit = 10 } = req.query;
      
      let filteredTasks = tasks;
      
      // Apply filters
      if (status) {
        filteredTasks = filteredTasks.filter(task => 
          status === 'completed' ? task.completed : !task.completed
        );
      }
      
      if (priority) {
        filteredTasks = filteredTasks.filter(task => task.priority === priority);
      }
      
      if (category) {
        filteredTasks = filteredTasks.filter(task => task.category === category);
      }
      
      // Pagination
      const startIndex = (page - 1) * limit;
      const endIndex = startIndex + parseInt(limit);
      const paginatedTasks = filteredTasks.slice(startIndex, endIndex);
      
      res.json({
        success: true,
        data: {
          tasks: paginatedTasks,
          pagination: {
            page: parseInt(page),
            limit: parseInt(limit),
            total: filteredTasks.length,
            pages: Math.ceil(filteredTasks.length / limit)
          }
        }
      });
    } catch (error) {
      next(error);
    }
  },

  // POST /api/tasks
  createTask: async (req, res, next) => {
    try {
      const { title, description, priority = 'medium', category, dueDate, tags = [] } = req.body;
      
      const newTask = {
        id: uuidv4(),
        title,
        description,
        completed: false,
        priority,
        category,
        dueDate,
        tags,
        createdAt: new Date().toISOString(),
        updatedAt: new Date().toISOString(),
        completedAt: null
      };
      
      const tasks = await taskStore.read();
      tasks.push(newTask);
      await taskStore.write(tasks);
      
      res.status(201).json({
        success: true,
        data: {
          task: newTask,
          message: 'Task created successfully'
        }
      });
    } catch (error) {
      next(error);
    }
  }
  
  // Add other CRUD methods...
};

module.exports = taskController;
```

#### 2. Task Routes
```javascript
// src/routes/taskRoutes.js
const express = require('express');
const taskController = require('../controllers/taskController');
const validateTask = require('../middleware/validateTask');

const router = express.Router();

// Core CRUD routes
router.get('/', taskController.getAllTasks);
router.get('/:id', taskController.getTaskById);
router.post('/', validateTask, taskController.createTask);
router.put('/:id', validateTask, taskController.updateTask);
router.patch('/:id', taskController.patchTask);
router.delete('/:id', taskController.deleteTask);

// Status operations
router.patch('/:id/complete', taskController.completeTask);
router.patch('/:id/incomplete', taskController.incompleteTask);

// Advanced operations
router.get('/search', taskController.searchTasks);
router.get('/stats', taskController.getStats);
router.post('/bulk', taskController.createBulkTasks);
router.delete('/completed', taskController.deleteCompleted);

module.exports = router;
```

### Phase 3: Validation and Middleware (2-3 hours)

#### 1. Task Validation Middleware
```javascript
// src/middleware/validateTask.js
const validateTask = (req, res, next) => {
  const { title, description, priority, dueDate } = req.body;
  const errors = [];
  
  // Title validation
  if (!title || title.trim().length === 0) {
    errors.push({ field: 'title', message: 'Title is required' });
  } else if (title.length > 100) {
    errors.push({ field: 'title', message: 'Title must be less than 100 characters' });
  }
  
  // Description validation
  if (description && description.length > 500) {
    errors.push({ field: 'description', message: 'Description must be less than 500 characters' });
  }
  
  // Priority validation
  const validPriorities = ['low', 'medium', 'high'];
  if (priority && !validPriorities.includes(priority)) {
    errors.push({ field: 'priority', message: 'Priority must be low, medium, or high' });
  }
  
  // Due date validation
  if (dueDate && isNaN(Date.parse(dueDate))) {
    errors.push({ field: 'dueDate', message: 'Due date must be a valid date' });
  }
  
  if (errors.length > 0) {
    return res.status(400).json({
      success: false,
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Validation failed',
        details: errors
      }
    });
  }
  
  next();
};

module.exports = validateTask;
```

#### 2. Authentication Middleware
```javascript
// src/middleware/auth.js
const apiKeyAuth = (req, res, next) => {
  const apiKey = req.get('X-API-Key') || req.query.apiKey;
  
  if (!apiKey) {
    return res.status(401).json({
      success: false,
      error: {
        code: 'MISSING_API_KEY',
        message: 'API key is required'
      }
    });
  }
  
  // In a real app, validate against database
  const validApiKeys = [process.env.API_KEY || 'your-secret-api-key'];
  
  if (!validApiKeys.includes(apiKey)) {
    return res.status(401).json({
      success: false,
      error: {
        code: 'INVALID_API_KEY',
        message: 'Invalid API key'
      }
    });
  }
  
  next();
};

module.exports = apiKeyAuth;
```

### Phase 4: Advanced Features and Testing (3-4 hours)

#### 1. Search and Statistics
```javascript
// Add to taskController.js
searchTasks: async (req, res, next) => {
  try {
    const { q } = req.query;
    const tasks = await taskStore.read();
    
    if (!q) {
      return res.json({ success: true, data: { tasks: [] } });
    }
    
    const searchResults = tasks.filter(task =>
      task.title.toLowerCase().includes(q.toLowerCase()) ||
      (task.description && task.description.toLowerCase().includes(q.toLowerCase())) ||
      (task.tags && task.tags.some(tag => tag.toLowerCase().includes(q.toLowerCase())))
    );
    
    res.json({
      success: true,
      data: {
        tasks: searchResults,
        query: q,
        count: searchResults.length
      }
    });
  } catch (error) {
    next(error);
  }
},

getStats: async (req, res, next) => {
  try {
    const tasks = await taskStore.read();
    
    const stats = {
      total: tasks.length,
      completed: tasks.filter(t => t.completed).length,
      pending: tasks.filter(t => !t.completed).length,
      overdue: tasks.filter(t => 
        !t.completed && t.dueDate && new Date(t.dueDate) < new Date()
      ).length,
      byPriority: {
        high: tasks.filter(t => t.priority === 'high').length,
        medium: tasks.filter(t => t.priority === 'medium').length,
        low: tasks.filter(t => t.priority === 'low').length
      },
      byCategory: tasks.reduce((acc, task) => {
        if (task.category) {
          acc[task.category] = (acc[task.category] || 0) + 1;
        }
        return acc;
      }, {})
    };
    
    res.json({ success: true, data: { stats } });
  } catch (error) {
    next(error);
  }
}
```

## 🎨 API Documentation

### Authentication
All API endpoints require authentication using an API key:

```bash
# Header method (recommended)
curl -H "X-API-Key: your-secret-api-key" http://localhost:3000/api/tasks

# Query parameter method
curl http://localhost:3000/api/tasks?apiKey=your-secret-api-key
```

### Example Requests

#### Create Task
```bash
curl -X POST http://localhost:3000/api/tasks \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-secret-api-key" \
  -d '{
    "title": "Complete Express.js course",
    "description": "Finish all modules and projects",
    "priority": "high",
    "category": "learning",
    "dueDate": "2024-07-30T18:00:00Z",
    "tags": ["education", "programming"]
  }'
```

#### Get Tasks with Filtering
```bash
# Get completed tasks
curl "http://localhost:3000/api/tasks?status=completed&apiKey=your-secret-api-key"

# Get high priority tasks
curl "http://localhost:3000/api/tasks?priority=high&apiKey=your-secret-api-key"

# Search tasks
curl "http://localhost:3000/api/tasks/search?q=express&apiKey=your-secret-api-key"
```

## 🧪 Testing Your API

### Manual Testing Checklist
- [ ] Create a new task
- [ ] Get all tasks
- [ ] Get specific task by ID
- [ ] Update task
- [ ] Mark task as complete
- [ ] Delete task
- [ ] Search tasks
- [ ] Filter tasks by status/priority
- [ ] Get task statistics
- [ ] Test error handling (invalid data, missing API key)

### Postman Collection
Create a Postman collection with all endpoints for easy testing.

## 🚀 Deployment Preparation

### Environment Configuration
```javascript
// .env
PORT=3000
NODE_ENV=development
API_KEY=your-secret-api-key-here
```

### Production Considerations
- Use a real database (MongoDB, PostgreSQL)
- Implement proper logging
- Add request rate limiting
- Use HTTPS in production
- Implement proper error monitoring

## 🔗 React Integration

This API is designed to work seamlessly with your React Todo application:

```javascript
// React service example
const API_BASE_URL = 'http://localhost:3000/api';
const API_KEY = 'your-secret-api-key';

const taskService = {
  async getAllTasks() {
    const response = await fetch(`${API_BASE_URL}/tasks`, {
      headers: { 'X-API-Key': API_KEY }
    });
    return response.json();
  },
  
  async createTask(task) {
    const response = await fetch(`${API_BASE_URL}/tasks`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-API-Key': API_KEY
      },
      body: JSON.stringify(task)
    });
    return response.json();
  }
};
```

## 📚 Learning Outcomes

After completing this project, you will have:
- Built a complete REST API from scratch
- Implemented CRUD operations with Express.js
- Created custom middleware for authentication and validation
- Handled errors gracefully with proper HTTP status codes
- Implemented search and filtering functionality
- Used JSON files for data persistence
- Created comprehensive API documentation
- Prepared an API for production deployment

---

**Ready to build your first backend API?** This project will give you hands-on experience with Express.js fundamentals and prepare you for more advanced backend development.

**Next Project:** [Weather Data API](../02-weather-api/README.md) - Learn external API integration and caching strategies.

---

*This project demonstrates essential backend skills that employers look for in Node.js developers. Take your time to build something robust and well-documented!*
