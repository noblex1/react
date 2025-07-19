# Module 03: Props and Component Communication

**Duration:** 1 Week | **Difficulty:** Beginner

## 🎯 Learning Objectives

By the end of this module, you will:
- ✅ Understand what props are and how to use them
- ✅ Pass data from parent to child components
- ✅ Make components dynamic and reusable
- ✅ Handle different types of props (strings, numbers, objects, functions)
- ✅ Use props for component communication
- ✅ Implement prop validation and default values

## 📚 What are Props?

**Props** (short for "properties") are how you pass data from one component to another in React. Think of props like **arguments** you pass to a function, or **attributes** you add to HTML elements.

### Real-World Analogy
Imagine you're ordering a custom t-shirt:
- The **component** is the t-shirt template
- The **props** are your customizations (size, color, text)
- Each t-shirt is unique based on the props you provide

## 💻 Basic Props Usage

### Passing Props
```jsx
// Parent component passing props
function App() {
  return (
    <div>
      <Greeting name="Alice" age={25} />
      <Greeting name="Bob" age={30} />
      <Greeting name="Charlie" age={22} />
    </div>
  );
}

// Child component receiving props
function Greeting(props) {
  return (
    <div>
      <h2>Hello, {props.name}!</h2>
      <p>You are {props.age} years old.</p>
    </div>
  );
}
```

### Destructuring Props (Cleaner Syntax)
```jsx
// Instead of using props.name, props.age
function Greeting({ name, age }) {
  return (
    <div>
      <h2>Hello, {name}!</h2>
      <p>You are {age} years old.</p>
    </div>
  );
}
```

## 🎨 Different Types of Props

### 1. String Props
```jsx
function Welcome({ message, color }) {
  return <h1 style={{ color: color }}>{message}</h1>;
}

// Usage
<Welcome message="Welcome to React!" color="blue" />
```

### 2. Number Props
```jsx
function Counter({ count, max }) {
  return (
    <div>
      <p>Count: {count}</p>
      <p>Progress: {(count / max) * 100}%</p>
    </div>
  );
}

// Usage
<Counter count={7} max={10} />
```

### 3. Boolean Props
```jsx
function Button({ text, disabled, primary }) {
  return (
    <button 
      disabled={disabled}
      className={primary ? 'btn-primary' : 'btn-secondary'}
    >
      {text}
    </button>
  );
}

// Usage
<Button text="Click me" disabled={false} primary={true} />
<Button text="Save" disabled={true} primary={false} />
```

### 4. Object Props
```jsx
function UserProfile({ user }) {
  return (
    <div className="user-profile">
      <img src={user.avatar} alt={user.name} />
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <p>Joined: {user.joinDate}</p>
    </div>
  );
}

// Usage
const userData = {
  name: "Sarah Johnson",
  email: "sarah@example.com",
  avatar: "https://via.placeholder.com/100",
  joinDate: "2023-01-15"
};

<UserProfile user={userData} />
```

### 5. Array Props
```jsx
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map((todo, index) => (
        <li key={index}>{todo}</li>
      ))}
    </ul>
  );
}

// Usage
const myTodos = ["Learn React", "Build a project", "Get a job"];
<TodoList todos={myTodos} />
```

### 6. Function Props
```jsx
function AlertButton({ message, onButtonClick }) {
  return (
    <button onClick={() => onButtonClick(message)}>
      Show Alert
    </button>
  );
}

// Usage
function App() {
  const handleAlert = (msg) => {
    alert(msg);
  };

  return (
    <AlertButton 
      message="Hello from props!" 
      onButtonClick={handleAlert} 
    />
  );
}
```

## 🏗️ Building Reusable Components

### Example 1: Product Card Component
```jsx
function ProductCard({ product }) {
  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p className="price">${product.price}</p>
      <p className="description">{product.description}</p>
      <button className="buy-btn">Add to Cart</button>
    </div>
  );
}

// Usage
const products = [
  {
    id: 1,
    name: "Laptop",
    price: 999,
    image: "laptop.jpg",
    description: "High-performance laptop"
  },
  {
    id: 2,
    name: "Phone",
    price: 699,
    image: "phone.jpg",
    description: "Latest smartphone"
  }
];

function ProductList() {
  return (
    <div className="product-list">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

### Example 2: Social Media Post Component
```jsx
function Post({ author, content, timestamp, likes, comments }) {
  return (
    <div className="post">
      <div className="post-header">
        <img src={author.avatar} alt={author.name} className="avatar" />
        <div>
          <h4>{author.name}</h4>
          <span className="timestamp">{timestamp}</span>
        </div>
      </div>
      <div className="post-content">
        <p>{content}</p>
      </div>
      <div className="post-actions">
        <button>👍 {likes}</button>
        <button>💬 {comments}</button>
        <button>🔄 Share</button>
      </div>
    </div>
  );
}
```

## 🏋️ Hands-on Exercises

### Exercise 1: Create a Movie Card Component
Build a component that displays movie information:

```jsx
function MovieCard({ title, year, genre, rating, poster }) {
  // Your code here
  // Display: poster image, title, year, genre, rating
  // Style it to look like a movie card
}

// Test with this data:
const movie = {
  title: "The Matrix",
  year: 1999,
  genre: "Sci-Fi",
  rating: 8.7,
  poster: "matrix-poster.jpg"
};
```

### Exercise 2: Build a Weather Card Component
Create a weather display component:

```jsx
function WeatherCard({ city, temperature, condition, humidity, windSpeed }) {
  // Your code here
  // Display: city name, temperature, weather condition, humidity, wind speed
  // Add appropriate styling and maybe weather icons
}
```

### Exercise 3: Create a Team Member Component
Build a component for displaying team member information:

```jsx
function TeamMember({ name, role, bio, skills, socialLinks }) {
  // Your code here
  // Display: photo, name, role, bio, skills list, social media links
}
```

## 🎯 Challenge: Build a Blog Post System

Create a complete blog post system with multiple components:

**Components to Build:**
1. **BlogPost** - Main post component
2. **Author** - Author information component
3. **TagList** - Component for displaying post tags
4. **CommentSection** - Comments display component

**Requirements:**
```jsx
// Sample data structure
const blogPost = {
  id: 1,
  title: "Getting Started with React",
  content: "React is a powerful library...",
  author: {
    name: "Jane Doe",
    avatar: "jane.jpg",
    bio: "Frontend Developer"
  },
  publishDate: "2024-01-15",
  tags: ["React", "JavaScript", "Frontend"],
  comments: [
    {
      id: 1,
      author: "John Smith",
      content: "Great article!",
      date: "2024-01-16"
    }
  ],
  likes: 42,
  readTime: "5 min read"
};
```

**Features to Implement:**
- Display full blog post with all information
- Reusable components for different sections
- Clean, readable layout
- Responsive design (basic)

## 🔧 Props Best Practices

### 1. Default Props
```jsx
function Button({ text = "Click me", color = "blue", size = "medium" }) {
  return (
    <button 
      className={`btn btn-${color} btn-${size}`}
    >
      {text}
    </button>
  );
}
```

### 2. Props Validation (Optional but Recommended)
```jsx
import PropTypes from 'prop-types';

function UserCard({ name, age, email }) {
  return (
    <div>
      <h3>{name}</h3>
      <p>Age: {age}</p>
      <p>Email: {email}</p>
    </div>
  );
}

UserCard.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number.isRequired,
  email: PropTypes.string.isRequired
};
```

### 3. Destructuring with Default Values
```jsx
function ProfileCard({ 
  name = "Anonymous", 
  role = "User", 
  isOnline = false 
}) {
  return (
    <div className="profile-card">
      <h3>{name}</h3>
      <p>{role}</p>
      <span className={isOnline ? "online" : "offline"}>
        {isOnline ? "🟢 Online" : "🔴 Offline"}
      </span>
    </div>
  );
}
```

## 📚 Additional Resources

### Essential Reading:
- [Components and Props - React Docs](https://reactjs.org/docs/components-and-props.html)
- [Typechecking With PropTypes](https://reactjs.org/docs/typechecking-with-proptypes.html)

### Practice Exercises:
- [React Props Exercises](https://react-exercises.com/props)
- [Component Communication Patterns](https://kentcdodds.com/blog/application-state-management-with-react)

### Videos:
- [React Props Explained](https://www.youtube.com/watch?v=PHaECbrKgs0)
- [Component Communication](https://www.youtube.com/watch?v=dyL99ACQfsM)

## ✅ Module Completion Checklist

Before moving to the next module, make sure you can:
- [ ] Pass props from parent to child components
- [ ] Use different types of props (string, number, boolean, object, array, function)
- [ ] Destructure props for cleaner code
- [ ] Create reusable components with props
- [ ] Set default prop values
- [ ] Build complex UIs with multiple components
- [ ] Understand the flow of data in React (top-down)

## 🚀 What's Next?

In **Module 04**, we'll learn about:
- Component state and how it differs from props
- Event handling in React
- Making components interactive
- Building your first interactive React application

---

**Ready to continue?** 👉 **[Module 04: State and Event Handling](../04-state-events/README.md)**

---

*Estimated completion time: 5-7 hours*
