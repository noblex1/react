# Module 01: Introduction to Node.js & Backend Development

**Duration:** 1 Week | **Difficulty:** Beginner

## 🎯 Learning Objectives

By the end of this module, you will:
- ✅ Understand what Node.js is and why it's used for backend development
- ✅ Set up a complete Node.js development environment
- ✅ Understand the difference between frontend and backend development
- ✅ Create your first Node.js application
- ✅ Understand asynchronous programming and the event loop
- ✅ Work with Node.js modules and npm packages

## 📚 What is Backend Development?

### Frontend vs Backend
**Frontend (Client-Side):**
- What users see and interact with
- Runs in the browser
- HTML, CSS, JavaScript, React
- User interface and user experience

**Backend (Server-Side):**
- Behind-the-scenes logic
- Runs on servers
- Handles data, authentication, business logic
- APIs, databases, security

### Real-World Analogy
Think of a restaurant:
- **Frontend** = Dining room (what customers see)
- **Backend** = Kitchen (where food is prepared)
- **API** = Waiters (communication between dining room and kitchen)

## 🚀 What is Node.js?

**Node.js** is a JavaScript runtime that allows you to run JavaScript on the server (outside the browser). It's built on Chrome's V8 JavaScript engine.

### Why Node.js?
- **Same Language** - Use JavaScript for both frontend and backend
- **Fast Performance** - Non-blocking, event-driven architecture
- **Large Ecosystem** - Huge npm package repository
- **Scalable** - Great for building APIs and real-time applications
- **Job Market** - High demand for Node.js developers

### Node.js vs Browser JavaScript
| Browser JavaScript | Node.js |
|-------------------|---------|
| DOM manipulation | File system access |
| Window object | Global object |
| Limited file access | Full server capabilities |
| User interactions | Server requests |

## 🛠️ Setting Up Node.js Development Environment

### Step 1: Install Node.js
1. Go to [nodejs.org](https://nodejs.org/)
2. Download the LTS (Long Term Support) version
3. Install following the installer instructions
4. Verify installation:
   ```bash
   node --version
   npm --version
   ```

### Step 2: Choose a Code Editor
**Recommended: Visual Studio Code**
- Download from [code.visualstudio.com](https://code.visualstudio.com/)
- Install useful extensions:
  - **Node.js Extension Pack**
  - **REST Client** (for testing APIs)
  - **Thunder Client** (Postman alternative)
  - **Auto Rename Tag**

### Step 3: Install Additional Tools
```bash
# Install nodemon globally (auto-restart server during development)
npm install -g nodemon

# Install Postman or use Thunder Client for API testing
```

## 💻 Your First Node.js Application

### Hello World Server
Create a file called `server.js`:

```javascript
// server.js
console.log('Hello from Node.js!');
console.log('Node.js version:', process.version);
console.log('Platform:', process.platform);
```

Run it:
```bash
node server.js
```

### Working with Modules
Node.js uses modules to organize code. There are three types:

#### 1. Built-in Modules
```javascript
// File system module
const fs = require('fs');

// HTTP module for creating servers
const http = require('http');

// Path module for working with file paths
const path = require('path');

// OS module for operating system info
const os = require('os');

console.log('OS Platform:', os.platform());
console.log('Free Memory:', os.freemem());
```

#### 2. Local Modules (Your Own Files)
Create `math.js`:
```javascript
// math.js
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

function multiply(a, b) {
  return a * b;
}

// Export functions
module.exports = {
  add,
  subtract,
  multiply
};
```

Use it in `app.js`:
```javascript
// app.js
const math = require('./math');

console.log('5 + 3 =', math.add(5, 3));
console.log('10 - 4 =', math.subtract(10, 4));
console.log('6 * 7 =', math.multiply(6, 7));
```

#### 3. Third-Party Modules (npm packages)
```bash
# Initialize a new Node.js project
npm init -y

# Install a package (example: colors for colorful console output)
npm install colors
```

Use the package:
```javascript
// app.js
const colors = require('colors');

console.log('Hello World!'.rainbow);
console.log('This is red text'.red);
console.log('This is green text'.green);
```

## 🌐 Creating Your First HTTP Server

```javascript
// server.js
const http = require('http');

// Create a server
const server = http.createServer((req, res) => {
  // Set response headers
  res.writeHead(200, { 'Content-Type': 'text/html' });
  
  // Send response
  res.end('<h1>Hello from Node.js Server!</h1>');
});

// Start the server
const PORT = 3000;
server.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

Run the server:
```bash
node server.js
```

Visit `http://localhost:3000` in your browser!

## ⚡ Understanding Asynchronous Programming

Node.js is **non-blocking** and **asynchronous**. This means it can handle multiple operations without waiting for each one to complete.

### Synchronous vs Asynchronous
```javascript
// Synchronous (blocking)
console.log('Start');
console.log('Middle');
console.log('End');

// Output: Start, Middle, End (in order)
```

```javascript
// Asynchronous (non-blocking)
console.log('Start');

setTimeout(() => {
  console.log('Middle (after 2 seconds)');
}, 2000);

console.log('End');

// Output: Start, End, Middle (after 2 seconds)
```

### Callbacks
```javascript
const fs = require('fs');

// Asynchronous file reading
fs.readFile('example.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Error reading file:', err);
    return;
  }
  console.log('File content:', data);
});

console.log('This runs immediately');
```

### Promises
```javascript
const fs = require('fs').promises;

// Using promises
fs.readFile('example.txt', 'utf8')
  .then(data => {
    console.log('File content:', data);
  })
  .catch(err => {
    console.error('Error reading file:', err);
  });
```

### Async/Await (Modern Approach)
```javascript
const fs = require('fs').promises;

async function readFileAsync() {
  try {
    const data = await fs.readFile('example.txt', 'utf8');
    console.log('File content:', data);
  } catch (err) {
    console.error('Error reading file:', err);
  }
}

readFileAsync();
```

## 🏋️ Hands-on Exercises

### Exercise 1: File System Operations
Create a Node.js script that:
1. Creates a new directory called 'my-app'
2. Creates a file called 'info.txt' with your name and today's date
3. Reads the file and displays its contents
4. Lists all files in the current directory

```javascript
// file-operations.js
const fs = require('fs').promises;
const path = require('path');

async function fileOperations() {
  // Your code here
}

fileOperations();
```

### Exercise 2: Simple Calculator Module
Create a calculator module with functions for basic math operations:
1. Create `calculator.js` with add, subtract, multiply, divide functions
2. Create `app.js` that imports and uses the calculator
3. Handle division by zero errors
4. Add a function that performs multiple operations

### Exercise 3: HTTP Server with Routes
Create an HTTP server that responds differently based on the URL:
- `/` - Welcome message
- `/about` - Information about yourself
- `/contact` - Your contact information
- Any other route - 404 Not Found message

```javascript
// server.js
const http = require('http');
const url = require('url');

const server = http.createServer((req, res) => {
  const pathname = url.parse(req.url).pathname;
  
  // Your routing logic here
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

## 🎯 Challenge: Personal Information API

Build a simple API server that returns information about yourself in JSON format:

**Requirements:**
1. **GET /api/profile** - Return your personal information (name, skills, experience)
2. **GET /api/skills** - Return a list of your technical skills
3. **GET /api/projects** - Return a list of projects you've worked on
4. **GET /api/contact** - Return your contact information
5. **Handle 404** - Return appropriate error for unknown routes

**Data Structure Example:**
```javascript
const profile = {
  name: "Your Name",
  title: "Aspiring Full-Stack Developer",
  experience: "Learning MERN Stack",
  location: "Your City"
};

const skills = [
  "JavaScript",
  "Node.js",
  "React",
  "HTML/CSS"
];
```

**Bonus Features:**
- Add proper HTTP status codes
- Include CORS headers
- Add request logging
- Format JSON responses nicely

## 📦 Package Management with npm

### Understanding package.json
```bash
# Create a new Node.js project
npm init -y
```

This creates a `package.json` file:
```json
{
  "name": "my-node-app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "dependencies": {},
  "devDependencies": {}
}
```

### Installing Packages
```bash
# Install a package for production
npm install express

# Install a package for development only
npm install --save-dev nodemon

# Install a package globally
npm install -g nodemon

# Install a specific version
npm install express@4.18.0
```

### Using Scripts
```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "echo \"No tests yet\""
  }
}
```

Run scripts:
```bash
npm start
npm run dev
npm test
```

## 🔧 Development Best Practices

### 1. Project Structure
```
my-node-app/
├── package.json
├── server.js          # Main server file
├── routes/            # Route handlers
├── models/            # Data models
├── middleware/        # Custom middleware
├── utils/             # Utility functions
├── config/            # Configuration files
└── tests/             # Test files
```

### 2. Environment Variables
Create a `.env` file:
```
PORT=3000
NODE_ENV=development
DATABASE_URL=mongodb://localhost:27017/myapp
```

Use with `dotenv` package:
```javascript
require('dotenv').config();

const PORT = process.env.PORT || 3000;
console.log('Environment:', process.env.NODE_ENV);
```

### 3. Error Handling
```javascript
process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err);
  process.exit(1);
});

process.on('unhandledRejection', (err) => {
  console.error('Unhandled Rejection:', err);
  process.exit(1);
});
```

## 📚 Additional Resources

### Essential Reading:
- [Node.js Official Documentation](https://nodejs.org/en/docs/)
- [npm Documentation](https://docs.npmjs.com/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)

### Helpful Tools:
- [Postman](https://www.postman.com/) - API testing
- [Insomnia](https://insomnia.rest/) - API client
- [MongoDB Compass](https://www.mongodb.com/products/compass) - Database GUI

### Videos:
- [Node.js Crash Course](https://www.youtube.com/watch?v=fBNz5xF-Kx4)
- [What is Node.js?](https://www.youtube.com/watch?v=uVwtVBpw7RQ)

## ✅ Module Completion Checklist

Before moving to the next module, make sure you can:
- [ ] Explain what Node.js is and its advantages
- [ ] Set up a Node.js development environment
- [ ] Create and run Node.js applications
- [ ] Work with built-in Node.js modules
- [ ] Create and use custom modules
- [ ] Install and use npm packages
- [ ] Create a basic HTTP server
- [ ] Understand asynchronous programming concepts
- [ ] Handle errors properly in Node.js applications

## 🚀 What's Next?

In **Module 02**, we'll dive into:
- Express.js framework and why it's essential
- Setting up Express servers
- Basic routing and middleware
- Request and response handling

---

**Ready to continue?** 👉 **[Module 02: Express.js Setup and Basic Server](../02-express-setup/README.md)**

---

*Estimated completion time: 6-8 hours*
