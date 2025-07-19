# Module 13: API Integration and HTTP Requests

**Duration:** 1 Week | **Difficulty:** Intermediate to Advanced

## 🎯 Learning Objectives

By the end of this module, you will:
- ✅ Understand RESTful APIs and HTTP methods
- ✅ Make API calls using fetch and axios
- ✅ Handle different types of API responses
- ✅ Implement proper error handling for API requests
- ✅ Manage loading states and user feedback
- ✅ Work with authentication and headers
- ✅ Understand CORS and how to handle it

## 📚 Understanding APIs and HTTP

### What is an API?
An **API** (Application Programming Interface) is a way for different software applications to communicate with each other. In web development, we typically work with **REST APIs** that use HTTP requests.

### HTTP Methods
- **GET** - Retrieve data
- **POST** - Create new data
- **PUT** - Update existing data (replace entire resource)
- **PATCH** - Update existing data (partial update)
- **DELETE** - Remove data

### HTTP Status Codes
- **200** - OK (Success)
- **201** - Created
- **400** - Bad Request
- **401** - Unauthorized
- **403** - Forbidden
- **404** - Not Found
- **500** - Internal Server Error

## 🌐 Making API Calls in React

### Using Fetch API
```jsx
import React, { useState, useEffect } from 'react';

function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUsers = async () => {
      try {
        setLoading(true);
        const response = await fetch('https://jsonplaceholder.typicode.com/users');
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        setUsers(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, []);

  if (loading) return <div>Loading users...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <h2>Users</h2>
      {users.map(user => (
        <div key={user.id}>
          <h3>{user.name}</h3>
          <p>{user.email}</p>
        </div>
      ))}
    </div>
  );
}
```

### Using Axios (Recommended)
First install axios: `npm install axios`

```jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function PostList() {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchPosts = async () => {
      try {
        setLoading(true);
        const response = await axios.get('https://jsonplaceholder.typicode.com/posts');
        setPosts(response.data);
      } catch (err) {
        setError(err.response?.data?.message || err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchPosts();
  }, []);

  if (loading) return <div>Loading posts...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h3>{post.title}</h3>
          <p>{post.body}</p>
        </article>
      ))}
    </div>
  );
}
```

## 🔧 CRUD Operations

### Create (POST)
```jsx
function CreatePost() {
  const [title, setTitle] = useState('');
  const [body, setBody] = useState('');
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    try {
      setLoading(true);
      const response = await axios.post('https://jsonplaceholder.typicode.com/posts', {
        title,
        body,
        userId: 1
      });
      
      console.log('Post created:', response.data);
      // Reset form
      setTitle('');
      setBody('');
    } catch (error) {
      console.error('Error creating post:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Post title"
        required
      />
      <textarea
        value={body}
        onChange={(e) => setBody(e.target.value)}
        placeholder="Post content"
        required
      />
      <button type="submit" disabled={loading}>
        {loading ? 'Creating...' : 'Create Post'}
      </button>
    </form>
  );
}
```

### Update (PUT/PATCH)
```jsx
function EditPost({ postId, onUpdate }) {
  const [post, setPost] = useState({ title: '', body: '' });
  const [loading, setLoading] = useState(false);

  const handleUpdate = async (e) => {
    e.preventDefault();
    
    try {
      setLoading(true);
      const response = await axios.put(
        `https://jsonplaceholder.typicode.com/posts/${postId}`,
        {
          ...post,
          userId: 1
        }
      );
      
      onUpdate(response.data);
    } catch (error) {
      console.error('Error updating post:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleUpdate}>
      <input
        type="text"
        value={post.title}
        onChange={(e) => setPost({ ...post, title: e.target.value })}
        placeholder="Post title"
      />
      <textarea
        value={post.body}
        onChange={(e) => setPost({ ...post, body: e.target.value })}
        placeholder="Post content"
      />
      <button type="submit" disabled={loading}>
        {loading ? 'Updating...' : 'Update Post'}
      </button>
    </form>
  );
}
```

### Delete
```jsx
function DeletePost({ postId, onDelete }) {
  const [loading, setLoading] = useState(false);

  const handleDelete = async () => {
    if (!window.confirm('Are you sure you want to delete this post?')) {
      return;
    }

    try {
      setLoading(true);
      await axios.delete(`https://jsonplaceholder.typicode.com/posts/${postId}`);
      onDelete(postId);
    } catch (error) {
      console.error('Error deleting post:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <button onClick={handleDelete} disabled={loading}>
      {loading ? 'Deleting...' : 'Delete'}
    </button>
  );
}
```

## 🔐 Authentication and Headers

### Setting Up Axios with Default Headers
```jsx
// api.js - API configuration
import axios from 'axios';

const API = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 10000,
});

// Add auth token to all requests
API.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('authToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Handle response errors globally
API.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Redirect to login
      localStorage.removeItem('authToken');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default API;
```

### Using the Configured API
```jsx
import API from './api';

function AuthenticatedComponent() {
  const [data, setData] = useState(null);

  useEffect(() => {
    const fetchProtectedData = async () => {
      try {
        const response = await API.get('/protected-endpoint');
        setData(response.data);
      } catch (error) {
        console.error('Error fetching protected data:', error);
      }
    };

    fetchProtectedData();
  }, []);

  return (
    <div>
      {data ? (
        <div>Protected data: {JSON.stringify(data)}</div>
      ) : (
        <div>Loading...</div>
      )}
    </div>
  );
}
```

## 🎣 Custom Hooks for API Calls

### useApi Hook
```jsx
import { useState, useEffect } from 'react';
import axios from 'axios';

function useApi(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);
        const response = await axios.get(url, options);
        setData(response.data);
      } catch (err) {
        setError(err.response?.data?.message || err.message);
      } finally {
        setLoading(false);
      }
    };

    if (url) {
      fetchData();
    }
  }, [url]);

  const refetch = () => {
    if (url) {
      fetchData();
    }
  };

  return { data, loading, error, refetch };
}

// Usage
function UserProfile({ userId }) {
  const { data: user, loading, error } = useApi(`/api/users/${userId}`);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

### useFetch Hook with More Features
```jsx
import { useState, useEffect, useCallback } from 'react';

function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      
      const response = await fetch(url, {
        headers: {
          'Content-Type': 'application/json',
          ...options.headers,
        },
        ...options,
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [url, options]);

  useEffect(() => {
    if (url) {
      fetchData();
    }
  }, [fetchData]);

  return { data, loading, error, refetch: fetchData };
}
```

## 🏋️ Hands-on Exercises

### Exercise 1: User Management System
Build a component that can:
- Fetch and display a list of users
- Add new users
- Edit existing users
- Delete users
- Handle all loading and error states

### Exercise 2: Search with Debouncing
Create a search component that:
- Searches through posts/users as you type
- Debounces API calls (wait 500ms after user stops typing)
- Shows loading state during search
- Handles empty results

### Exercise 3: Pagination
Implement a paginated list that:
- Fetches data in pages (10 items per page)
- Shows page numbers and navigation
- Handles loading states for each page
- Remembers current page on component re-render

## 🎯 Challenge: Build a Blog Management System

Create a complete blog management system with these features:

**Requirements:**
1. **Posts List** - Display all blog posts with pagination
2. **Post Details** - View individual post with comments
3. **Create Post** - Form to create new posts
4. **Edit Post** - Edit existing posts
5. **Delete Post** - Remove posts with confirmation
6. **Comments** - Display and manage comments
7. **Search** - Search posts by title/content
8. **Categories** - Filter posts by category

**API Endpoints to Use:**
- JSONPlaceholder API for testing
- Posts: `/posts`
- Comments: `/comments`
- Users: `/users`

**Component Structure:**
```jsx
function BlogApp() {
  // Main app component with routing
}

function PostsList() {
  // List all posts with pagination and search
}

function PostDetail({ postId }) {
  // Show single post with comments
}

function PostForm({ postId, onSave }) {
  // Create/edit post form
}

function CommentsList({ postId }) {
  // Display comments for a post
}
```

## 🔧 Error Handling Best Practices

### Comprehensive Error Handling
```jsx
function ApiComponent() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const handleApiCall = async () => {
    try {
      setLoading(true);
      setError(null);
      
      const response = await axios.get('/api/data');
      setData(response.data);
    } catch (err) {
      // Handle different types of errors
      if (err.response) {
        // Server responded with error status
        setError(`Server Error: ${err.response.status} - ${err.response.data.message}`);
      } else if (err.request) {
        // Request was made but no response received
        setError('Network Error: Unable to reach server');
      } else {
        // Something else happened
        setError(`Error: ${err.message}`);
      }
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      {loading && <div>Loading...</div>}
      {error && (
        <div className="error">
          {error}
          <button onClick={handleApiCall}>Retry</button>
        </div>
      )}
      {data && <div>Data: {JSON.stringify(data)}</div>}
    </div>
  );
}
```

### Global Error Boundary
```jsx
import React from 'react';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>Something went wrong.</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={() => this.setState({ hasError: false, error: null })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

## 📚 Additional Resources

### Essential Reading:
- [Fetch API Documentation](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [Axios Documentation](https://axios-http.com/docs/intro)
- [REST API Best Practices](https://restfulapi.net/)

### Practice APIs:
- [JSONPlaceholder](https://jsonplaceholder.typicode.com/) - Fake REST API
- [ReqRes](https://reqres.in/) - Test API with real responses
- [Dog API](https://dog.ceo/dog-api/) - Fun API for practice
- [News API](https://newsapi.org/) - Real news data

### Advanced Topics:
- [React Query](https://react-query.tanstack.com/) - Advanced data fetching
- [SWR](https://swr.vercel.app/) - Data fetching with caching
- [GraphQL with Apollo](https://www.apollographql.com/docs/react/)

## ✅ Module Completion Checklist

Before moving to the next module, make sure you can:
- [ ] Make GET requests to fetch data
- [ ] Implement POST requests to create data
- [ ] Handle PUT/PATCH requests for updates
- [ ] Implement DELETE requests
- [ ] Handle loading states properly
- [ ] Implement comprehensive error handling
- [ ] Work with authentication headers
- [ ] Create reusable API hooks
- [ ] Handle different HTTP status codes
- [ ] Implement retry logic for failed requests

## 🚀 What's Next?

In **Module 14**, we'll explore:
- User authentication and authorization
- JWT tokens and session management
- Protected routes and role-based access
- Login/logout functionality
- Password reset and user registration

---

**Ready to continue?** 👉 **[Module 14: Authentication and Authorization](../14-authentication/README.md)**

---

*Estimated completion time: 8-10 hours*
