# Project 1: CLI Task Manager 📋

**Difficulty:** Beginner | **Estimated Time:** 12-16 hours | **Technologies:** Node.js Core, File System, CLI

## 🎯 Project Overview

Build a comprehensive command-line task management application using pure Node.js. This project demonstrates core Node.js concepts including file system operations, command-line argument parsing, event-driven programming, and JSON data persistence.

## 🚀 What You'll Build

A fully functional CLI task manager with the following features:
- ✅ Add, edit, and delete tasks
- ✅ Mark tasks as complete/incomplete
- ✅ List tasks with filtering and sorting
- ✅ Search tasks by keywords
- ✅ Task categories and priorities
- ✅ Due date management
- ✅ Data persistence with JSON files
- ✅ Colorful CLI output
- ✅ Progress tracking and statistics

## 🛠️ Technical Requirements

### Core Features

#### Command Structure
```bash
# Task management
node task add "Complete Node.js course" --priority high --due 2024-07-30
node task list --status pending --sort priority
node task complete 1
node task delete 2
node task edit 3 --title "Updated task title"

# Filtering and search
node task list --category work --priority high
node task search "Node.js"
node task due --today
node task overdue

# Statistics and reporting
node task stats
node task export --format json
node task import tasks.json
```

#### Data Model
```javascript
// Task structure
{
  "id": 1,
  "title": "Complete Node.js course",
  "description": "Finish all modules and projects",
  "status": "pending", // pending, completed, cancelled
  "priority": "high", // low, medium, high
  "category": "learning",
  "dueDate": "2024-07-30T18:00:00Z",
  "tags": ["education", "programming"],
  "createdAt": "2024-07-20T10:00:00Z",
  "updatedAt": "2024-07-20T10:00:00Z",
  "completedAt": null
}
```

## 📋 Step-by-Step Implementation Guide

### Phase 1: Project Setup and Core Structure (3-4 hours)

#### 1. Initialize Project
```bash
mkdir cli-task-manager
cd cli-task-manager
npm init -y

# Install dependencies
npm install chalk commander inquirer date-fns
npm install --save-dev jest

# Create project structure
mkdir src data
touch src/task.js src/storage.js src/cli.js src/utils.js
touch data/tasks.json
```

#### 2. Basic CLI Setup
```javascript
// src/cli.js
#!/usr/bin/env node

const { Command } = require('commander');
const chalk = require('chalk');
const TaskManager = require('./task');

const program = new Command();
const taskManager = new TaskManager();

program
  .name('task')
  .description('CLI Task Manager')
  .version('1.0.0');

// Add task command
program
  .command('add <title>')
  .description('Add a new task')
  .option('-d, --description <desc>', 'Task description')
  .option('-p, --priority <priority>', 'Task priority (low, medium, high)', 'medium')
  .option('-c, --category <category>', 'Task category')
  .option('--due <date>', 'Due date (YYYY-MM-DD)')
  .action(async (title, options) => {
    try {
      const task = await taskManager.addTask(title, options);
      console.log(chalk.green('✓ Task added successfully!'));
      console.log(chalk.blue(`ID: ${task.id} - ${task.title}`));
    } catch (error) {
      console.error(chalk.red('Error:', error.message));
    }
  });

program.parse();
```

#### 3. Data Storage Layer
```javascript
// src/storage.js
const fs = require('fs').promises;
const path = require('path');

class Storage {
  constructor(filename = 'tasks.json') {
    this.filePath = path.join(__dirname, '../data', filename);
  }

  async read() {
    try {
      const data = await fs.readFile(this.filePath, 'utf8');
      return JSON.parse(data);
    } catch (error) {
      if (error.code === 'ENOENT') {
        // File doesn't exist, return empty array
        return [];
      }
      throw error;
    }
  }

  async write(data) {
    try {
      await fs.writeFile(this.filePath, JSON.stringify(data, null, 2));
    } catch (error) {
      throw new Error(`Failed to write data: ${error.message}`);
    }
  }

  async backup() {
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
    const backupPath = this.filePath.replace('.json', `_backup_${timestamp}.json`);
    
    try {
      const data = await this.read();
      await fs.writeFile(backupPath, JSON.stringify(data, null, 2));
      return backupPath;
    } catch (error) {
      throw new Error(`Failed to create backup: ${error.message}`);
    }
  }
}

module.exports = Storage;
```

### Phase 2: Core Task Management (4-5 hours)

#### 1. Task Manager Class
```javascript
// src/task.js
const Storage = require('./storage');
const { formatDate, validateDate, generateId } = require('./utils');

class TaskManager {
  constructor() {
    this.storage = new Storage();
  }

  async addTask(title, options = {}) {
    const tasks = await this.storage.read();
    
    // Validate input
    if (!title || title.trim().length === 0) {
      throw new Error('Task title is required');
    }

    if (options.due && !validateDate(options.due)) {
      throw new Error('Invalid due date format. Use YYYY-MM-DD');
    }

    const task = {
      id: generateId(tasks),
      title: title.trim(),
      description: options.description || '',
      status: 'pending',
      priority: options.priority || 'medium',
      category: options.category || 'general',
      dueDate: options.due ? new Date(options.due).toISOString() : null,
      tags: options.tags ? options.tags.split(',').map(tag => tag.trim()) : [],
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString(),
      completedAt: null
    };

    tasks.push(task);
    await this.storage.write(tasks);
    
    return task;
  }

  async listTasks(filters = {}) {
    const tasks = await this.storage.read();
    let filteredTasks = [...tasks];

    // Apply filters
    if (filters.status) {
      filteredTasks = filteredTasks.filter(task => task.status === filters.status);
    }

    if (filters.priority) {
      filteredTasks = filteredTasks.filter(task => task.priority === filters.priority);
    }

    if (filters.category) {
      filteredTasks = filteredTasks.filter(task => task.category === filters.category);
    }

    if (filters.due) {
      const today = new Date().toDateString();
      filteredTasks = filteredTasks.filter(task => {
        if (!task.dueDate) return false;
        const taskDate = new Date(task.dueDate).toDateString();
        return taskDate === today;
      });
    }

    // Apply sorting
    if (filters.sort) {
      filteredTasks.sort((a, b) => {
        switch (filters.sort) {
          case 'priority':
            const priorityOrder = { high: 3, medium: 2, low: 1 };
            return priorityOrder[b.priority] - priorityOrder[a.priority];
          case 'due':
            if (!a.dueDate && !b.dueDate) return 0;
            if (!a.dueDate) return 1;
            if (!b.dueDate) return -1;
            return new Date(a.dueDate) - new Date(b.dueDate);
          case 'created':
            return new Date(b.createdAt) - new Date(a.createdAt);
          default:
            return 0;
        }
      });
    }

    return filteredTasks;
  }

  async completeTask(id) {
    const tasks = await this.storage.read();
    const taskIndex = tasks.findIndex(task => task.id === parseInt(id));
    
    if (taskIndex === -1) {
      throw new Error(`Task with ID ${id} not found`);
    }

    tasks[taskIndex].status = 'completed';
    tasks[taskIndex].completedAt = new Date().toISOString();
    tasks[taskIndex].updatedAt = new Date().toISOString();

    await this.storage.write(tasks);
    return tasks[taskIndex];
  }

  async deleteTask(id) {
    const tasks = await this.storage.read();
    const taskIndex = tasks.findIndex(task => task.id === parseInt(id));
    
    if (taskIndex === -1) {
      throw new Error(`Task with ID ${id} not found`);
    }

    const deletedTask = tasks.splice(taskIndex, 1)[0];
    await this.storage.write(tasks);
    
    return deletedTask;
  }

  async searchTasks(query) {
    const tasks = await this.storage.read();
    const searchTerm = query.toLowerCase();
    
    return tasks.filter(task => 
      task.title.toLowerCase().includes(searchTerm) ||
      task.description.toLowerCase().includes(searchTerm) ||
      task.category.toLowerCase().includes(searchTerm) ||
      (task.tags && task.tags.some(tag => tag.toLowerCase().includes(searchTerm)))
    );
  }

  async getStats() {
    const tasks = await this.storage.read();
    
    const stats = {
      total: tasks.length,
      pending: tasks.filter(t => t.status === 'pending').length,
      completed: tasks.filter(t => t.status === 'completed').length,
      cancelled: tasks.filter(t => t.status === 'cancelled').length,
      overdue: tasks.filter(t => 
        t.status === 'pending' && 
        t.dueDate && 
        new Date(t.dueDate) < new Date()
      ).length,
      byPriority: {
        high: tasks.filter(t => t.priority === 'high').length,
        medium: tasks.filter(t => t.priority === 'medium').length,
        low: tasks.filter(t => t.priority === 'low').length
      },
      byCategory: tasks.reduce((acc, task) => {
        acc[task.category] = (acc[task.category] || 0) + 1;
        return acc;
      }, {})
    };

    return stats;
  }
}

module.exports = TaskManager;
```

### Phase 3: Advanced CLI Features (3-4 hours)

#### 1. Enhanced CLI Commands
```javascript
// Add to src/cli.js

// List tasks command
program
  .command('list')
  .description('List tasks')
  .option('-s, --status <status>', 'Filter by status (pending, completed, cancelled)')
  .option('-p, --priority <priority>', 'Filter by priority (low, medium, high)')
  .option('-c, --category <category>', 'Filter by category')
  .option('--sort <field>', 'Sort by field (priority, due, created)')
  .option('--due-today', 'Show tasks due today')
  .action(async (options) => {
    try {
      const filters = {
        status: options.status,
        priority: options.priority,
        category: options.category,
        sort: options.sort,
        due: options.dueToday
      };
      
      const tasks = await taskManager.listTasks(filters);
      displayTasks(tasks);
    } catch (error) {
      console.error(chalk.red('Error:', error.message));
    }
  });

// Complete task command
program
  .command('complete <id>')
  .description('Mark task as complete')
  .action(async (id) => {
    try {
      const task = await taskManager.completeTask(id);
      console.log(chalk.green('✓ Task completed!'));
      console.log(chalk.blue(`${task.title}`));
    } catch (error) {
      console.error(chalk.red('Error:', error.message));
    }
  });

// Search command
program
  .command('search <query>')
  .description('Search tasks')
  .action(async (query) => {
    try {
      const tasks = await taskManager.searchTasks(query);
      console.log(chalk.yellow(`Found ${tasks.length} task(s) matching "${query}"`));
      displayTasks(tasks);
    } catch (error) {
      console.error(chalk.red('Error:', error.message));
    }
  });

// Stats command
program
  .command('stats')
  .description('Show task statistics')
  .action(async () => {
    try {
      const stats = await taskManager.getStats();
      displayStats(stats);
    } catch (error) {
      console.error(chalk.red('Error:', error.message));
    }
  });

// Display functions
function displayTasks(tasks) {
  if (tasks.length === 0) {
    console.log(chalk.yellow('No tasks found.'));
    return;
  }

  console.log(chalk.bold('\nTasks:\n'));
  
  tasks.forEach(task => {
    const statusIcon = task.status === 'completed' ? '✓' : 
                      task.status === 'cancelled' ? '✗' : '○';
    const statusColor = task.status === 'completed' ? 'green' : 
                       task.status === 'cancelled' ? 'red' : 'yellow';
    
    const priorityColor = task.priority === 'high' ? 'red' : 
                         task.priority === 'medium' ? 'yellow' : 'blue';
    
    console.log(chalk[statusColor](statusIcon) + ` [${task.id}] ${task.title}`);
    console.log(`   Priority: ${chalk[priorityColor](task.priority.toUpperCase())}`);
    console.log(`   Category: ${task.category}`);
    
    if (task.dueDate) {
      const dueDate = new Date(task.dueDate);
      const isOverdue = task.status === 'pending' && dueDate < new Date();
      const dueDateStr = formatDate(dueDate);
      console.log(`   Due: ${isOverdue ? chalk.red(dueDateStr + ' (OVERDUE)') : dueDateStr}`);
    }
    
    if (task.description) {
      console.log(`   ${chalk.gray(task.description)}`);
    }
    
    console.log('');
  });
}

function displayStats(stats) {
  console.log(chalk.bold('\n📊 Task Statistics\n'));
  
  console.log(`Total Tasks: ${chalk.blue(stats.total)}`);
  console.log(`Pending: ${chalk.yellow(stats.pending)}`);
  console.log(`Completed: ${chalk.green(stats.completed)}`);
  console.log(`Cancelled: ${chalk.red(stats.cancelled)}`);
  console.log(`Overdue: ${chalk.red(stats.overdue)}`);
  
  console.log(chalk.bold('\nBy Priority:'));
  console.log(`High: ${chalk.red(stats.byPriority.high)}`);
  console.log(`Medium: ${chalk.yellow(stats.byPriority.medium)}`);
  console.log(`Low: ${chalk.blue(stats.byPriority.low)}`);
  
  console.log(chalk.bold('\nBy Category:'));
  Object.entries(stats.byCategory).forEach(([category, count]) => {
    console.log(`${category}: ${count}`);
  });
}
```

### Phase 4: Utility Functions and Testing (2-3 hours)

#### 1. Utility Functions
```javascript
// src/utils.js
const { format, isValid, parseISO } = require('date-fns');

function generateId(tasks) {
  if (tasks.length === 0) return 1;
  return Math.max(...tasks.map(task => task.id)) + 1;
}

function validateDate(dateString) {
  const date = new Date(dateString);
  return isValid(date) && dateString.match(/^\d{4}-\d{2}-\d{2}$/);
}

function formatDate(date) {
  return format(date, 'MMM dd, yyyy');
}

function formatDateTime(date) {
  return format(date, 'MMM dd, yyyy HH:mm');
}

function isOverdue(dueDate, status) {
  if (!dueDate || status !== 'pending') return false;
  return new Date(dueDate) < new Date();
}

function getPriorityWeight(priority) {
  const weights = { high: 3, medium: 2, low: 1 };
  return weights[priority] || 1;
}

module.exports = {
  generateId,
  validateDate,
  formatDate,
  formatDateTime,
  isOverdue,
  getPriorityWeight
};
```

## 🎨 Advanced Features

### Interactive Mode
```javascript
// Add interactive command
program
  .command('interactive')
  .alias('i')
  .description('Interactive task management')
  .action(async () => {
    const inquirer = require('inquirer');
    
    const { action } = await inquirer.prompt([
      {
        type: 'list',
        name: 'action',
        message: 'What would you like to do?',
        choices: [
          'Add Task',
          'List Tasks',
          'Complete Task',
          'Delete Task',
          'View Statistics',
          'Exit'
        ]
      }
    ]);
    
    // Handle interactive actions
    switch (action) {
      case 'Add Task':
        await interactiveAddTask();
        break;
      // ... other cases
    }
  });
```

### Export/Import Functionality
```javascript
// Export tasks
program
  .command('export')
  .description('Export tasks to file')
  .option('-f, --format <format>', 'Export format (json, csv)', 'json')
  .option('-o, --output <file>', 'Output file name')
  .action(async (options) => {
    try {
      const tasks = await taskManager.listTasks();
      const filename = options.output || `tasks_export_${Date.now()}.${options.format}`;
      
      if (options.format === 'csv') {
        await exportToCSV(tasks, filename);
      } else {
        await exportToJSON(tasks, filename);
      }
      
      console.log(chalk.green(`✓ Tasks exported to ${filename}`));
    } catch (error) {
      console.error(chalk.red('Error:', error.message));
    }
  });
```

## 🧪 Testing Your CLI

### Manual Testing Checklist
- [ ] Add tasks with different priorities and categories
- [ ] List tasks with various filters
- [ ] Complete and delete tasks
- [ ] Search functionality
- [ ] Due date handling and overdue detection
- [ ] Statistics display
- [ ] Error handling for invalid inputs

### Automated Testing
```javascript
// tests/task.test.js
const TaskManager = require('../src/task');
const Storage = require('../src/storage');

describe('TaskManager', () => {
  let taskManager;
  
  beforeEach(() => {
    taskManager = new TaskManager();
    // Mock storage for testing
  });
  
  test('should add a new task', async () => {
    const task = await taskManager.addTask('Test task', { priority: 'high' });
    expect(task.title).toBe('Test task');
    expect(task.priority).toBe('high');
    expect(task.status).toBe('pending');
  });
  
  test('should complete a task', async () => {
    const task = await taskManager.addTask('Test task');
    const completedTask = await taskManager.completeTask(task.id);
    expect(completedTask.status).toBe('completed');
    expect(completedTask.completedAt).toBeTruthy();
  });
});
```

## 📚 Learning Outcomes

After completing this project, you will have:
- Mastered Node.js file system operations
- Built a complete CLI application with argument parsing
- Implemented data persistence with JSON files
- Used event-driven programming patterns
- Created a modular, testable Node.js application
- Handled errors and edge cases gracefully
- Built an interactive command-line interface

---

**Ready to build your CLI task manager?** This project will give you hands-on experience with core Node.js concepts and prepare you for more advanced backend development.

**Next Project:** [Real-time Chat Server](../02-realtime-chat/README.md) - Learn networking, WebSockets, and event-driven architecture.

---

*This project demonstrates essential Node.js skills that employers look for in backend developers. Take your time to build something robust and well-tested!*
