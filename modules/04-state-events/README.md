# Module 04: State and Event Handling

**Duration:** 1 Week | **Difficulty:** Beginner to Intermediate

## 🎯 Learning Objectives

By the end of this module, you will:
- ✅ Understand what state is and how it differs from props
- ✅ Use the useState hook to manage component state
- ✅ Handle user events (clicks, form inputs, etc.)
- ✅ Build interactive React applications
- ✅ Understand when and how to update state
- ✅ Create controlled components for forms

## 📚 What is State?

**State** is data that can change over time in your component. Unlike props (which are passed down and don't change), state is **internal** to a component and can be **updated**.

### State vs Props
| State | Props |
|-------|-------|
| Internal to component | Passed from parent |
| Can be changed | Read-only |
| Causes re-render when updated | Causes re-render when changed |
| Managed by the component itself | Managed by parent component |

### Real-World Analogy
Think of state like the **settings** on your phone:
- **Props** are like the phone model (fixed, can't change)
- **State** is like brightness, volume, wifi status (can change)

## 🎣 The useState Hook

The `useState` hook is how we add state to functional components.

### Basic Syntax
```jsx
import React, { useState } from 'react';

function Counter() {
  // useState returns [currentValue, setterFunction]
  const [count, setCount] = useState(0); // 0 is the initial value
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

### How useState Works
1. **`useState(0)`** - Initialize state with value 0
2. **`count`** - Current state value
3. **`setCount`** - Function to update the state
4. When state updates, component re-renders with new value

## 🖱️ Event Handling

Events are how users interact with your application (clicks, typing, hovering, etc.).

### Basic Event Handling
```jsx
function Button() {
  const handleClick = () => {
    alert('Button was clicked!');
  };

  return <button onClick={handleClick}>Click me!</button>;
}
```

### Inline Event Handlers
```jsx
function Button() {
  return (
    <button onClick={() => alert('Clicked!')}>
      Click me!
    </button>
  );
}
```

### Event Object
```jsx
function InputExample() {
  const handleChange = (event) => {
    console.log('Input value:', event.target.value);
  };

  return (
    <input 
      type="text" 
      onChange={handleChange}
      placeholder="Type something..."
    />
  );
}
```

## 💻 Practical Examples

### Example 1: Toggle Visibility
```jsx
function ToggleExample() {
  const [isVisible, setIsVisible] = useState(true);

  return (
    <div>
      <button onClick={() => setIsVisible(!isVisible)}>
        {isVisible ? 'Hide' : 'Show'} Content
      </button>
      {isVisible && <p>This content can be toggled!</p>}
    </div>
  );
}
```

### Example 2: Input Field with State
```jsx
function NameInput() {
  const [name, setName] = useState('');

  return (
    <div>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Enter your name"
      />
      <p>Hello, {name || 'stranger'}!</p>
    </div>
  );
}
```

### Example 3: Counter with Multiple Buttons
```jsx
function AdvancedCounter() {
  const [count, setCount] = useState(0);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);
  const reset = () => setCount(0);
  const double = () => setCount(count * 2);

  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={increment}>+1</button>
      <button onClick={decrement}>-1</button>
      <button onClick={reset}>Reset</button>
      <button onClick={double}>Double</button>
    </div>
  );
}
```

### Example 4: Shopping Cart
```jsx
function ShoppingCart() {
  const [items, setItems] = useState([]);
  const [inputValue, setInputValue] = useState('');

  const addItem = () => {
    if (inputValue.trim()) {
      setItems([...items, inputValue]);
      setInputValue('');
    }
  };

  const removeItem = (index) => {
    const newItems = items.filter((_, i) => i !== index);
    setItems(newItems);
  };

  return (
    <div>
      <h3>Shopping Cart ({items.length} items)</h3>
      <div>
        <input
          type="text"
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          placeholder="Add item..."
        />
        <button onClick={addItem}>Add</button>
      </div>
      <ul>
        {items.map((item, index) => (
          <li key={index}>
            {item}
            <button onClick={() => removeItem(index)}>Remove</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## 🏋️ Hands-on Exercises

### Exercise 1: Like Button
Create a like button that shows the number of likes and toggles between liked/unliked states:

```jsx
function LikeButton() {
  // State for: likes count, isLiked boolean
  // Button should show: "❤️ 5 likes" or "🤍 5 likes"
  // Clicking should toggle like status and update count
}
```

### Exercise 2: Color Picker
Build a component that changes background color:

```jsx
function ColorPicker() {
  // State for: current color
  // Buttons for: red, blue, green, yellow, purple
  // Display current color name
  // Change background color of a div
}
```

### Exercise 3: Simple Calculator
Create a basic calculator:

```jsx
function Calculator() {
  // State for: first number, second number, operation, result
  // Input fields for numbers
  // Buttons for: +, -, *, /
  // Display result
}
```

## 🎯 Challenge: Build a Todo Application

Create a fully functional todo application with the following features:

**Requirements:**
1. **Add todos** - Input field and add button
2. **Display todos** - List all todos
3. **Mark complete** - Toggle todo completion status
4. **Delete todos** - Remove todos from list
5. **Filter todos** - Show all, active, or completed
6. **Todo counter** - Show number of remaining todos

**Component Structure:**
```jsx
function TodoApp() {
  // State for: todos array, input value, filter
  
  // Functions for:
  // - addTodo
  // - toggleTodo
  // - deleteTodo
  // - setFilter
  
  // Render:
  // - Input section
  // - Todo list
  // - Filter buttons
  // - Todo counter
}
```

**Todo Object Structure:**
```javascript
{
  id: 1,
  text: "Learn React",
  completed: false,
  createdAt: new Date()
}
```

**Bonus Features:**
- Edit existing todos
- Drag and drop reordering
- Local storage persistence
- Due dates for todos

## 🎨 Styling Interactive Components

### CSS for Interactive States
```css
.button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
}

.button:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.button:active {
  transform: translateY(0);
}

.button.primary {
  background-color: #007bff;
  color: white;
}

.button.danger {
  background-color: #dc3545;
  color: white;
}
```

### Conditional Styling with State
```jsx
function StatusButton() {
  const [isActive, setIsActive] = useState(false);

  return (
    <button
      className={`button ${isActive ? 'active' : 'inactive'}`}
      onClick={() => setIsActive(!isActive)}
    >
      {isActive ? 'Active' : 'Inactive'}
    </button>
  );
}
```

## 🔧 State Best Practices

### 1. Don't Mutate State Directly
```jsx
// ❌ Wrong - Mutating state directly
const [items, setItems] = useState([1, 2, 3]);
items.push(4); // Don't do this!

// ✅ Correct - Create new array
setItems([...items, 4]);
```

### 2. Use Functional Updates for Complex State
```jsx
// When new state depends on previous state
setCount(prevCount => prevCount + 1);

// For objects
setUser(prevUser => ({
  ...prevUser,
  name: 'New Name'
}));
```

### 3. Keep State Simple
```jsx
// ❌ Complex nested state
const [state, setState] = useState({
  user: { name: '', email: '' },
  posts: [],
  loading: false
});

// ✅ Separate state variables
const [user, setUser] = useState({ name: '', email: '' });
const [posts, setPosts] = useState([]);
const [loading, setLoading] = useState(false);
```

## 📚 Additional Resources

### Essential Reading:
- [State and Lifecycle - React Docs](https://reactjs.org/docs/state-and-lifecycle.html)
- [Handling Events - React Docs](https://reactjs.org/docs/handling-events.html)
- [Using the State Hook](https://reactjs.org/docs/hooks-state.html)

### Interactive Tutorials:
- [React State Tutorial](https://react-tutorial.app/app.html)
- [Event Handling Practice](https://react-exercises.com/events)

### Videos:
- [React useState Hook](https://www.youtube.com/watch?v=O6P86uwfdR0)
- [Event Handling in React](https://www.youtube.com/watch?v=Znqv84xi8Vs)

## ✅ Module Completion Checklist

Before moving to the next module, make sure you can:
- [ ] Use useState to add state to components
- [ ] Handle user events (clicks, input changes, form submissions)
- [ ] Update state based on user interactions
- [ ] Create controlled components for form inputs
- [ ] Build interactive applications with multiple state variables
- [ ] Understand when components re-render
- [ ] Follow state management best practices
- [ ] Build a complete todo application

## 🚀 What's Next?

Congratulations! You've completed the React Fundamentals phase. You're now ready to build your first portfolio project.

**🎯 Next Steps:**
1. **[Project 1: Todo List Application](../../projects/01-todo-app/README.md)** - Apply everything you've learned
2. **[Module 05: React Hooks](../05-react-hooks/README.md)** - Learn more powerful hooks

---

**Ready for your first project?** 👉 **[Build Your Todo App](../../projects/01-todo-app/README.md)**

---

*Estimated completion time: 6-8 hours*
