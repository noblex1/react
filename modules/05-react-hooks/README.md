# Module 05: React Hooks (useState, useEffect)

**Duration:** 1 Week | **Difficulty:** Intermediate

## 🎯 Learning Objectives

By the end of this module, you will:
- ✅ Master the useState hook for complex state management
- ✅ Understand and use the useEffect hook for side effects
- ✅ Handle component lifecycle with hooks
- ✅ Fetch data from APIs using useEffect
- ✅ Clean up resources and prevent memory leaks
- ✅ Understand dependency arrays and optimization

## 📚 Introduction to React Hooks

**Hooks** are functions that let you "hook into" React features from functional components. They were introduced in React 16.8 and have become the modern way to write React components.

### Why Hooks?
- **Simpler code** - No need for class components
- **Reusable logic** - Share stateful logic between components
- **Better performance** - Easier optimization
- **Modern React** - Industry standard approach

## 🎣 useState Hook (Advanced)

You've already learned basic useState. Let's explore advanced patterns:

### Multiple State Variables
```jsx
function UserProfile() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);
  const [isLoading, setIsLoading] = useState(false);

  return (
    <div>
      <input 
        value={name} 
        onChange={(e) => setName(e.target.value)}
        placeholder="Name"
      />
      <input 
        value={email} 
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input 
        type="number"
        value={age} 
        onChange={(e) => setAge(parseInt(e.target.value))}
        placeholder="Age"
      />
    </div>
  );
}
```

### Object State
```jsx
function UserForm() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });

  const updateUser = (field, value) => {
    setUser(prevUser => ({
      ...prevUser,
      [field]: value
    }));
  };

  return (
    <div>
      <input 
        value={user.name}
        onChange={(e) => updateUser('name', e.target.value)}
        placeholder="Name"
      />
      <input 
        value={user.email}
        onChange={(e) => updateUser('email', e.target.value)}
        placeholder="Email"
      />
    </div>
  );
}
```

### Array State
```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);

  const addTodo = (text) => {
    const newTodo = {
      id: Date.now(),
      text,
      completed: false
    };
    setTodos(prevTodos => [...prevTodos, newTodo]);
  };

  const removeTodo = (id) => {
    setTodos(prevTodos => prevTodos.filter(todo => todo.id !== id));
  };

  const toggleTodo = (id) => {
    setTodos(prevTodos =>
      prevTodos.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  };

  return (
    <div>
      {/* Todo UI here */}
    </div>
  );
}
```

## ⚡ useEffect Hook

The `useEffect` hook lets you perform side effects in functional components. Think of it as a combination of `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount` from class components.

### Basic useEffect
```jsx
import React, { useState, useEffect } from 'react';

function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(prevSeconds => prevSeconds + 1);
    }, 1000);

    // Cleanup function (like componentWillUnmount)
    return () => clearInterval(interval);
  }, []); // Empty dependency array = run once on mount

  return <div>Timer: {seconds} seconds</div>;
}
```

### useEffect with Dependencies
```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setLoading(true);
    
    // Fetch user data when userId changes
    fetch(`/api/users/${userId}`)
      .then(response => response.json())
      .then(userData => {
        setUser(userData);
        setLoading(false);
      });
  }, [userId]); // Run when userId changes

  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

### Multiple useEffect Hooks
```jsx
function Dashboard() {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [notifications, setNotifications] = useState([]);

  // Effect for user data
  useEffect(() => {
    fetchUserData().then(setUser);
  }, []);

  // Effect for posts
  useEffect(() => {
    if (user) {
      fetchUserPosts(user.id).then(setPosts);
    }
  }, [user]);

  // Effect for notifications
  useEffect(() => {
    const interval = setInterval(() => {
      fetchNotifications().then(setNotifications);
    }, 30000); // Check every 30 seconds

    return () => clearInterval(interval);
  }, []);

  return (
    <div>
      {/* Dashboard UI */}
    </div>
  );
}
```

## 🌐 Data Fetching with useEffect

### Basic API Call
```jsx
function PostList() {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchPosts = async () => {
      try {
        setLoading(true);
        const response = await fetch('https://jsonplaceholder.typicode.com/posts');
        
        if (!response.ok) {
          throw new Error('Failed to fetch posts');
        }
        
        const data = await response.json();
        setPosts(data);
      } catch (err) {
        setError(err.message);
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
        <div key={post.id}>
          <h3>{post.title}</h3>
          <p>{post.body}</p>
        </div>
      ))}
    </div>
  );
}
```

### Search with Debouncing
```jsx
function SearchPosts() {
  const [query, setQuery] = useState('');
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (!query) {
      setPosts([]);
      return;
    }

    const timeoutId = setTimeout(() => {
      setLoading(true);
      
      fetch(`/api/posts/search?q=${query}`)
        .then(response => response.json())
        .then(data => {
          setPosts(data);
          setLoading(false);
        });
    }, 500); // Debounce for 500ms

    return () => clearTimeout(timeoutId);
  }, [query]);

  return (
    <div>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search posts..."
      />
      
      {loading && <div>Searching...</div>}
      
      {posts.map(post => (
        <div key={post.id}>
          <h3>{post.title}</h3>
          <p>{post.body}</p>
        </div>
      ))}
    </div>
  );
}
```

## 🏋️ Hands-on Exercises

### Exercise 1: Weather App Component
Create a component that fetches and displays weather data:

```jsx
function WeatherWidget({ city }) {
  // State for: weather data, loading, error
  // useEffect to fetch weather when city changes
  // Display: temperature, description, humidity, wind speed
  // Handle loading and error states
}
```

### Exercise 2: Real-time Clock
Build a clock that updates every second:

```jsx
function DigitalClock() {
  // State for current time
  // useEffect with setInterval to update time
  // Display formatted time (HH:MM:SS)
  // Cleanup interval on unmount
}
```

### Exercise 3: User Profile with Posts
Create a component that shows user info and their posts:

```jsx
function UserDashboard({ userId }) {
  // State for: user data, posts, loading states
  // Multiple useEffect hooks for different data
  // Handle dependencies properly
  // Display user info and posts list
}
```

## 🎯 Challenge: Build a News Reader App

Create a news reader application with the following features:

**Requirements:**
1. **Fetch News Articles** - Use a news API (NewsAPI, Guardian, etc.)
2. **Category Filter** - Filter by different news categories
3. **Search Functionality** - Search articles with debouncing
4. **Infinite Scroll** - Load more articles as user scrolls
5. **Bookmarks** - Save favorite articles to local storage
6. **Loading States** - Show loading indicators
7. **Error Handling** - Handle API errors gracefully

**Component Structure:**
```jsx
function NewsApp() {
  // State management for articles, categories, search, etc.
  // Multiple useEffect hooks for different data fetching
}

function ArticleList({ articles, onLoadMore }) {
  // Display articles with infinite scroll
}

function SearchBar({ onSearch }) {
  // Search input with debouncing
}

function CategoryFilter({ categories, selectedCategory, onCategoryChange }) {
  // Category selection buttons
}
```

**API Integration:**
- Use NewsAPI or similar service
- Handle API rate limits
- Implement proper error handling
- Cache results when possible

## 🔧 useEffect Best Practices

### 1. Dependency Arrays
```jsx
// ❌ Missing dependencies
useEffect(() => {
  fetchUserData(userId);
}, []); // userId should be in dependencies

// ✅ Correct dependencies
useEffect(() => {
  fetchUserData(userId);
}, [userId]);
```

### 2. Cleanup Functions
```jsx
useEffect(() => {
  const subscription = subscribeToData();
  
  return () => {
    subscription.unsubscribe(); // Always cleanup
  };
}, []);
```

### 3. Conditional Effects
```jsx
useEffect(() => {
  if (user && user.id) {
    fetchUserPosts(user.id);
  }
}, [user]);
```

### 4. Avoiding Infinite Loops
```jsx
// ❌ This will cause infinite loop
const [user, setUser] = useState({});
useEffect(() => {
  setUser({ ...user, lastSeen: new Date() });
}, [user]);

// ✅ Use functional update or separate the dependency
useEffect(() => {
  setUser(prevUser => ({ ...prevUser, lastSeen: new Date() }));
}, []); // Or use a different dependency
```

## 📚 Additional Resources

### Essential Reading:
- [Using the Effect Hook](https://reactjs.org/docs/hooks-effect.html)
- [Rules of Hooks](https://reactjs.org/docs/hooks-rules.html)
- [Hooks FAQ](https://reactjs.org/docs/hooks-faq.html)

### Advanced Topics:
- [useEffect Complete Guide](https://overreacted.io/a-complete-guide-to-useeffect/)
- [Data Fetching with Hooks](https://www.robinwieruch.de/react-hooks-fetch-data)

### Practice APIs:
- [JSONPlaceholder](https://jsonplaceholder.typicode.com/) - Fake REST API
- [NewsAPI](https://newsapi.org/) - News articles
- [OpenWeatherMap](https://openweathermap.org/api) - Weather data
- [REST Countries](https://restcountries.com/) - Country information

## ✅ Module Completion Checklist

Before moving to the next module, make sure you can:
- [ ] Use useState for complex state management (objects, arrays)
- [ ] Understand useEffect and its dependency array
- [ ] Fetch data from APIs using useEffect
- [ ] Handle loading and error states
- [ ] Implement cleanup functions to prevent memory leaks
- [ ] Use multiple useEffect hooks in one component
- [ ] Implement debouncing for search functionality
- [ ] Handle component lifecycle events with hooks
- [ ] Follow hooks best practices and rules

## 🚀 What's Next?

In **Module 06**, we'll explore:
- Advanced hooks (useContext, useReducer, useMemo, useCallback)
- Creating custom hooks
- Performance optimization techniques
- Complex state management patterns

---

**Ready to continue?** 👉 **[Module 06: Advanced Hooks and Custom Hooks](../06-advanced-hooks/README.md)**

---

*Estimated completion time: 6-8 hours*
