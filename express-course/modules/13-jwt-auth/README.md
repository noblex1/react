# Module 13: User Authentication with JWT

**Duration:** 1 Week | **Difficulty:** Advanced

## 🎯 Learning Objectives

By the end of this module, you will:
- ✅ Understand JWT (JSON Web Tokens) and how they work
- ✅ Implement user registration and login systems
- ✅ Hash passwords securely using bcrypt
- ✅ Create and verify JWT tokens
- ✅ Build authentication middleware
- ✅ Handle token refresh and expiration
- ✅ Implement logout functionality
- ✅ Secure API endpoints with authentication

## 📚 What is JWT Authentication?

**JWT (JSON Web Token)** is a compact, URL-safe means of representing claims to be transferred between two parties. It's commonly used for authentication in modern web applications.

### JWT Structure
A JWT consists of three parts separated by dots (`.`):
```
header.payload.signature
```

**Example JWT:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### JWT vs Sessions
| JWT | Sessions |
|-----|----------|
| Stateless | Stateful |
| Stored client-side | Stored server-side |
| Self-contained | Requires database lookup |
| Scalable | Server memory dependent |
| Can't be revoked easily | Easy to revoke |

## 🛠️ Setting Up JWT Authentication

### Install Required Packages
```bash
npm install jsonwebtoken bcryptjs
npm install --save-dev @types/jsonwebtoken @types/bcryptjs
```

### Environment Variables
```javascript
// .env
JWT_SECRET=your-super-secret-jwt-key-here
JWT_EXPIRES_IN=7d
BCRYPT_SALT_ROUNDS=12
```

### User Model Structure
```javascript
// models/User.js (for now, we'll use JSON storage)
const userSchema = {
  id: "uuid",
  email: "user@example.com",
  password: "hashed-password",
  name: "John Doe",
  role: "user", // user, admin
  isActive: true,
  createdAt: "2024-07-20T10:00:00Z",
  updatedAt: "2024-07-20T10:00:00Z",
  lastLogin: "2024-07-20T10:00:00Z"
};
```

## 🔐 Password Hashing with bcrypt

### Hashing Passwords
```javascript
const bcrypt = require('bcryptjs');

class PasswordService {
  static async hashPassword(password) {
    const saltRounds = parseInt(process.env.BCRYPT_SALT_ROUNDS) || 12;
    return await bcrypt.hash(password, saltRounds);
  }
  
  static async comparePassword(password, hashedPassword) {
    return await bcrypt.compare(password, hashedPassword);
  }
  
  static validatePassword(password) {
    const errors = [];
    
    if (!password) {
      errors.push('Password is required');
    } else {
      if (password.length < 8) {
        errors.push('Password must be at least 8 characters long');
      }
      if (!/(?=.*[a-z])/.test(password)) {
        errors.push('Password must contain at least one lowercase letter');
      }
      if (!/(?=.*[A-Z])/.test(password)) {
        errors.push('Password must contain at least one uppercase letter');
      }
      if (!/(?=.*\d)/.test(password)) {
        errors.push('Password must contain at least one number');
      }
      if (!/(?=.*[@$!%*?&])/.test(password)) {
        errors.push('Password must contain at least one special character');
      }
    }
    
    return errors;
  }
}

module.exports = PasswordService;
```

## 🎫 JWT Token Management

### JWT Service
```javascript
const jwt = require('jsonwebtoken');

class JWTService {
  static generateToken(payload) {
    return jwt.sign(
      payload,
      process.env.JWT_SECRET,
      { 
        expiresIn: process.env.JWT_EXPIRES_IN || '7d',
        issuer: 'your-app-name',
        audience: 'your-app-users'
      }
    );
  }
  
  static verifyToken(token) {
    try {
      return jwt.verify(token, process.env.JWT_SECRET);
    } catch (error) {
      if (error.name === 'TokenExpiredError') {
        throw new Error('Token has expired');
      } else if (error.name === 'JsonWebTokenError') {
        throw new Error('Invalid token');
      } else {
        throw new Error('Token verification failed');
      }
    }
  }
  
  static generateRefreshToken(payload) {
    return jwt.sign(
      payload,
      process.env.JWT_REFRESH_SECRET || process.env.JWT_SECRET,
      { 
        expiresIn: '30d',
        issuer: 'your-app-name',
        audience: 'your-app-users'
      }
    );
  }
  
  static decodeToken(token) {
    return jwt.decode(token);
  }
}

module.exports = JWTService;
```

## 👤 User Registration and Login

### Auth Controller
```javascript
const { v4: uuidv4 } = require('uuid');
const DataStore = require('../utils/dataStore');
const PasswordService = require('../services/PasswordService');
const JWTService = require('../services/JWTService');

const userStore = new DataStore('users.json');

const authController = {
  // POST /api/auth/register
  register: async (req, res, next) => {
    try {
      const { email, password, name } = req.body;
      
      // Validate input
      if (!email || !password || !name) {
        return res.status(400).json({
          success: false,
          error: {
            code: 'MISSING_FIELDS',
            message: 'Email, password, and name are required'
          }
        });
      }
      
      // Validate email format
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      if (!emailRegex.test(email)) {
        return res.status(400).json({
          success: false,
          error: {
            code: 'INVALID_EMAIL',
            message: 'Please provide a valid email address'
          }
        });
      }
      
      // Validate password
      const passwordErrors = PasswordService.validatePassword(password);
      if (passwordErrors.length > 0) {
        return res.status(400).json({
          success: false,
          error: {
            code: 'INVALID_PASSWORD',
            message: 'Password validation failed',
            details: passwordErrors
          }
        });
      }
      
      // Check if user already exists
      const users = await userStore.read();
      const existingUser = users.find(user => user.email.toLowerCase() === email.toLowerCase());
      
      if (existingUser) {
        return res.status(409).json({
          success: false,
          error: {
            code: 'USER_EXISTS',
            message: 'User with this email already exists'
          }
        });
      }
      
      // Hash password
      const hashedPassword = await PasswordService.hashPassword(password);
      
      // Create new user
      const newUser = {
        id: uuidv4(),
        email: email.toLowerCase(),
        password: hashedPassword,
        name: name.trim(),
        role: 'user',
        isActive: true,
        createdAt: new Date().toISOString(),
        updatedAt: new Date().toISOString(),
        lastLogin: null
      };
      
      users.push(newUser);
      await userStore.write(users);
      
      // Generate tokens
      const tokenPayload = {
        userId: newUser.id,
        email: newUser.email,
        role: newUser.role
      };
      
      const accessToken = JWTService.generateToken(tokenPayload);
      const refreshToken = JWTService.generateRefreshToken({ userId: newUser.id });
      
      // Remove password from response
      const { password: _, ...userResponse } = newUser;
      
      res.status(201).json({
        success: true,
        data: {
          user: userResponse,
          tokens: {
            accessToken,
            refreshToken,
            expiresIn: process.env.JWT_EXPIRES_IN || '7d'
          },
          message: 'User registered successfully'
        }
      });
      
    } catch (error) {
      next(error);
    }
  },
  
  // POST /api/auth/login
  login: async (req, res, next) => {
    try {
      const { email, password } = req.body;
      
      // Validate input
      if (!email || !password) {
        return res.status(400).json({
          success: false,
          error: {
            code: 'MISSING_CREDENTIALS',
            message: 'Email and password are required'
          }
        });
      }
      
      // Find user
      const users = await userStore.read();
      const user = users.find(u => u.email.toLowerCase() === email.toLowerCase());
      
      if (!user) {
        return res.status(401).json({
          success: false,
          error: {
            code: 'INVALID_CREDENTIALS',
            message: 'Invalid email or password'
          }
        });
      }
      
      // Check if user is active
      if (!user.isActive) {
        return res.status(401).json({
          success: false,
          error: {
            code: 'ACCOUNT_DISABLED',
            message: 'Account has been disabled'
          }
        });
      }
      
      // Verify password
      const isPasswordValid = await PasswordService.comparePassword(password, user.password);
      
      if (!isPasswordValid) {
        return res.status(401).json({
          success: false,
          error: {
            code: 'INVALID_CREDENTIALS',
            message: 'Invalid email or password'
          }
        });
      }
      
      // Update last login
      user.lastLogin = new Date().toISOString();
      await userStore.write(users);
      
      // Generate tokens
      const tokenPayload = {
        userId: user.id,
        email: user.email,
        role: user.role
      };
      
      const accessToken = JWTService.generateToken(tokenPayload);
      const refreshToken = JWTService.generateRefreshToken({ userId: user.id });
      
      // Remove password from response
      const { password: _, ...userResponse } = user;
      
      res.json({
        success: true,
        data: {
          user: userResponse,
          tokens: {
            accessToken,
            refreshToken,
            expiresIn: process.env.JWT_EXPIRES_IN || '7d'
          },
          message: 'Login successful'
        }
      });
      
    } catch (error) {
      next(error);
    }
  },
  
  // POST /api/auth/refresh
  refreshToken: async (req, res, next) => {
    try {
      const { refreshToken } = req.body;
      
      if (!refreshToken) {
        return res.status(400).json({
          success: false,
          error: {
            code: 'MISSING_REFRESH_TOKEN',
            message: 'Refresh token is required'
          }
        });
      }
      
      // Verify refresh token
      const decoded = JWTService.verifyToken(refreshToken);
      
      // Find user
      const users = await userStore.read();
      const user = users.find(u => u.id === decoded.userId);
      
      if (!user || !user.isActive) {
        return res.status(401).json({
          success: false,
          error: {
            code: 'INVALID_REFRESH_TOKEN',
            message: 'Invalid refresh token'
          }
        });
      }
      
      // Generate new access token
      const tokenPayload = {
        userId: user.id,
        email: user.email,
        role: user.role
      };
      
      const newAccessToken = JWTService.generateToken(tokenPayload);
      
      res.json({
        success: true,
        data: {
          accessToken: newAccessToken,
          expiresIn: process.env.JWT_EXPIRES_IN || '7d'
        }
      });
      
    } catch (error) {
      if (error.message.includes('Token') || error.message.includes('token')) {
        return res.status(401).json({
          success: false,
          error: {
            code: 'INVALID_REFRESH_TOKEN',
            message: error.message
          }
        });
      }
      next(error);
    }
  },
  
  // POST /api/auth/logout
  logout: async (req, res, next) => {
    try {
      // In a real app, you might want to blacklist the token
      // For now, we'll just return success
      res.json({
        success: true,
        data: {
          message: 'Logout successful'
        }
      });
    } catch (error) {
      next(error);
    }
  }
};

module.exports = authController;
```

## 🛡️ Authentication Middleware

### JWT Authentication Middleware
```javascript
const JWTService = require('../services/JWTService');
const DataStore = require('../utils/dataStore');

const userStore = new DataStore('users.json');

const authenticateToken = async (req, res, next) => {
  try {
    // Get token from header
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN
    
    if (!token) {
      return res.status(401).json({
        success: false,
        error: {
          code: 'MISSING_TOKEN',
          message: 'Access token is required'
        }
      });
    }
    
    // Verify token
    const decoded = JWTService.verifyToken(token);
    
    // Find user
    const users = await userStore.read();
    const user = users.find(u => u.id === decoded.userId);
    
    if (!user) {
      return res.status(401).json({
        success: false,
        error: {
          code: 'USER_NOT_FOUND',
          message: 'User not found'
        }
      });
    }
    
    if (!user.isActive) {
      return res.status(401).json({
        success: false,
        error: {
          code: 'ACCOUNT_DISABLED',
          message: 'Account has been disabled'
        }
      });
    }
    
    // Add user to request object
    req.user = {
      id: user.id,
      email: user.email,
      name: user.name,
      role: user.role
    };
    
    next();
    
  } catch (error) {
    if (error.message.includes('Token') || error.message.includes('token')) {
      return res.status(401).json({
        success: false,
        error: {
          code: 'INVALID_TOKEN',
          message: error.message
        }
      });
    }
    next(error);
  }
};

// Optional authentication (doesn't fail if no token)
const optionalAuth = async (req, res, next) => {
  try {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    
    if (token) {
      const decoded = JWTService.verifyToken(token);
      const users = await userStore.read();
      const user = users.find(u => u.id === decoded.userId);
      
      if (user && user.isActive) {
        req.user = {
          id: user.id,
          email: user.email,
          name: user.name,
          role: user.role
        };
      }
    }
    
    next();
  } catch (error) {
    // Ignore token errors for optional auth
    next();
  }
};

module.exports = {
  authenticateToken,
  optionalAuth
};
```

## 🏋️ Hands-on Exercises

### Exercise 1: User Profile Management
Create endpoints for:
- `GET /api/auth/profile` - Get current user profile
- `PUT /api/auth/profile` - Update user profile
- `POST /api/auth/change-password` - Change password
- `DELETE /api/auth/account` - Delete account

### Exercise 2: Password Reset Flow
Implement:
- `POST /api/auth/forgot-password` - Request password reset
- `POST /api/auth/reset-password` - Reset password with token
- Email simulation (console.log for now)

### Exercise 3: Account Verification
Add:
- Email verification during registration
- `POST /api/auth/verify-email` - Verify email with token
- `POST /api/auth/resend-verification` - Resend verification email

## 🎯 Challenge: Complete Authentication System

Build a comprehensive authentication system with:

**Core Features:**
- User registration with email verification
- User login with JWT tokens
- Token refresh mechanism
- Password reset flow
- User profile management
- Account deactivation

**Security Features:**
- Password strength validation
- Rate limiting for auth endpoints
- Account lockout after failed attempts
- Token blacklisting for logout
- Secure password reset tokens

**Advanced Features:**
- Two-factor authentication (2FA)
- Social login integration
- Role-based permissions
- Session management
- Audit logging

## 🔧 Using Authentication in Routes

### Protecting Routes
```javascript
const express = require('express');
const { authenticateToken } = require('../middleware/auth');
const taskController = require('../controllers/taskController');

const router = express.Router();

// Public routes
router.post('/register', authController.register);
router.post('/login', authController.login);

// Protected routes
router.use(authenticateToken); // All routes below require authentication

router.get('/profile', authController.getProfile);
router.put('/profile', authController.updateProfile);
router.post('/change-password', authController.changePassword);

module.exports = router;
```

### Frontend Integration
```javascript
// React authentication service
class AuthService {
  static setTokens(accessToken, refreshToken) {
    localStorage.setItem('accessToken', accessToken);
    localStorage.setItem('refreshToken', refreshToken);
  }
  
  static getAccessToken() {
    return localStorage.getItem('accessToken');
  }
  
  static async makeAuthenticatedRequest(url, options = {}) {
    const token = this.getAccessToken();
    
    const response = await fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      }
    });
    
    if (response.status === 401) {
      // Token expired, try to refresh
      const refreshed = await this.refreshToken();
      if (refreshed) {
        // Retry request with new token
        return this.makeAuthenticatedRequest(url, options);
      } else {
        // Redirect to login
        window.location.href = '/login';
      }
    }
    
    return response;
  }
  
  static async refreshToken() {
    const refreshToken = localStorage.getItem('refreshToken');
    if (!refreshToken) return false;
    
    try {
      const response = await fetch('/api/auth/refresh', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ refreshToken })
      });
      
      if (response.ok) {
        const data = await response.json();
        localStorage.setItem('accessToken', data.data.accessToken);
        return true;
      }
    } catch (error) {
      console.error('Token refresh failed:', error);
    }
    
    return false;
  }
}
```

## 📚 Additional Resources

### Essential Reading:
- [JWT.io](https://jwt.io/) - JWT debugger and documentation
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [bcrypt Documentation](https://github.com/kelektiv/node.bcrypt.js)

### Security Best Practices:
- Never store passwords in plain text
- Use strong JWT secrets
- Implement token expiration
- Validate all user inputs
- Use HTTPS in production
- Implement rate limiting
- Log authentication events

## ✅ Module Completion Checklist

Before moving to the next module, make sure you can:
- [ ] Implement user registration with password hashing
- [ ] Create login functionality with JWT tokens
- [ ] Build authentication middleware
- [ ] Handle token refresh and expiration
- [ ] Validate passwords and user input
- [ ] Protect API routes with authentication
- [ ] Handle authentication errors gracefully
- [ ] Integrate authentication with frontend applications
- [ ] Understand JWT security considerations

## 🚀 What's Next?

In **Module 14**, we'll explore:
- Role-based authorization systems
- Permission management
- Admin panels and user management
- Advanced security patterns

---

**Ready to continue?** 👉 **[Module 14: Authorization and Role-Based Access](../14-authorization/README.md)**

---

*Estimated completion time: 10-12 hours*
