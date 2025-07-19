# Module 02: JSX and Components

**Duration:** 1 Week | **Difficulty:** Beginner

## 🎯 Learning Objectives

By the end of this module, you will:
- ✅ Understand JSX syntax and how it differs from HTML
- ✅ Create your first custom React components
- ✅ Understand component composition and reusability
- ✅ Know JSX rules and best practices
- ✅ Build a simple multi-component application

## 📚 What is JSX?

**JSX** stands for **JavaScript XML**. It's a syntax extension that lets you write HTML-like code inside JavaScript. Think of it as a way to describe what your UI should look like.

### JSX vs HTML

**HTML:**
```html
<div class="container">
  <h1>Hello World</h1>
  <p>This is HTML</p>
</div>
```

**JSX:**
```jsx
<div className="container">
  <h1>Hello World</h1>
  <p>This is JSX</p>
</div>
```

**Key Differences:**
- `class` becomes `className` in JSX
- JSX must have one parent element
- You can embed JavaScript expressions using `{}`

## 🧩 Understanding Components

A **component** is like a custom HTML element that you create. It's a piece of UI that you can reuse throughout your application.

### Function Components (Modern Way)

```jsx
// This is a component!
function Welcome() {
  return <h1>Hello, welcome to React!</h1>;
}

// Using the component
function App() {
  return (
    <div>
      <Welcome />
      <Welcome />
      <Welcome />
    </div>
  );
}
```

### Component Rules:
1. **Component names must start with a capital letter**
2. **Components must return JSX**
3. **Components can be reused multiple times**

## 💻 JSX Syntax Rules

### 1. Single Parent Element
```jsx
// ❌ Wrong - Multiple parent elements
function BadComponent() {
  return (
    <h1>Title</h1>
    <p>Paragraph</p>
  );
}

// ✅ Correct - Single parent element
function GoodComponent() {
  return (
    <div>
      <h1>Title</h1>
      <p>Paragraph</p>
    </div>
  );
}

// ✅ Also correct - Using React Fragment
function AlsoGoodComponent() {
  return (
    <>
      <h1>Title</h1>
      <p>Paragraph</p>
    </>
  );
}
```

### 2. JavaScript Expressions in JSX
```jsx
function Greeting() {
  const name = "Sarah";
  const age = 25;
  
  return (
    <div>
      <h1>Hello, {name}!</h1>
      <p>You are {age} years old.</p>
      <p>Next year you'll be {age + 1}!</p>
    </div>
  );
}
```

### 3. Conditional Rendering
```jsx
function WelcomeMessage() {
  const isLoggedIn = true;
  
  return (
    <div>
      {isLoggedIn ? (
        <h1>Welcome back!</h1>
      ) : (
        <h1>Please sign in.</h1>
      )}
    </div>
  );
}
```

### 4. Lists and Keys
```jsx
function FruitList() {
  const fruits = ['Apple', 'Banana', 'Orange'];
  
  return (
    <ul>
      {fruits.map((fruit, index) => (
        <li key={index}>{fruit}</li>
      ))}
    </ul>
  );
}
```

## 🏗️ Creating Custom Components

Let's create some practical components:

### Example 1: User Card Component
```jsx
function UserCard() {
  return (
    <div className="user-card">
      <img src="https://via.placeholder.com/100" alt="User" />
      <h3>John Doe</h3>
      <p>Software Developer</p>
      <button>Follow</button>
    </div>
  );
}
```

### Example 2: Navigation Component
```jsx
function Navigation() {
  return (
    <nav>
      <ul>
        <li><a href="#home">Home</a></li>
        <li><a href="#about">About</a></li>
        <li><a href="#contact">Contact</a></li>
      </ul>
    </nav>
  );
}
```

### Example 3: Footer Component
```jsx
function Footer() {
  const currentYear = new Date().getFullYear();
  
  return (
    <footer>
      <p>&copy; {currentYear} My React App. All rights reserved.</p>
    </footer>
  );
}
```

## 🏋️ Hands-on Exercises

### Exercise 1: Create a Profile Component
Create a component that displays your personal information:

```jsx
function Profile() {
  // Add your code here
  // Include: name, photo, bio, favorite programming language
}
```

### Exercise 2: Build a Product Card
Create a component for an e-commerce product:

```jsx
function ProductCard() {
  // Include: product image, name, price, description, buy button
}
```

### Exercise 3: Create a Weather Widget
Build a simple weather display component:

```jsx
function WeatherWidget() {
  // Include: city name, temperature, weather condition, icon
}
```

## 🎯 Challenge: Build a Personal Portfolio Page

Create a complete personal portfolio page using multiple components:

**Requirements:**
1. **Header Component** - Your name and navigation
2. **About Component** - Brief introduction about yourself
3. **Skills Component** - List of your technical skills
4. **Projects Component** - Showcase 2-3 projects
5. **Contact Component** - Your contact information
6. **Footer Component** - Copyright and social links

**File Structure:**
```
src/
├── components/
│   ├── Header.js
│   ├── About.js
│   ├── Skills.js
│   ├── Projects.js
│   ├── Contact.js
│   └── Footer.js
├── App.js
└── App.css
```

**Styling Tips:**
- Use CSS classes with `className`
- Create a clean, professional look
- Make it responsive (we'll improve this later)

## 🎨 Styling Components

### Inline Styles
```jsx
function StyledComponent() {
  const styles = {
    backgroundColor: 'blue',
    color: 'white',
    padding: '20px',
    borderRadius: '5px'
  };
  
  return <div style={styles}>Styled with inline styles</div>;
}
```

### CSS Classes
```jsx
// In your CSS file
.card {
  background-color: #f0f0f0;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

// In your component
function Card() {
  return <div className="card">This is a styled card</div>;
}
```

## 🔧 Best Practices

1. **One Component Per File** - Keep components organized
2. **Descriptive Names** - Use clear, descriptive component names
3. **Small Components** - Break large components into smaller ones
4. **Consistent Formatting** - Use Prettier for code formatting
5. **Comments** - Add comments for complex logic

## 📚 Additional Resources

### Essential Reading:
- [JSX In Depth](https://reactjs.org/docs/jsx-in-depth.html)
- [Components and Props](https://reactjs.org/docs/components-and-props.html)
- [Conditional Rendering](https://reactjs.org/docs/conditional-rendering.html)

### Practice Platforms:
- [React CodePen Collection](https://codepen.io/collection/DYVxpE/)
- [React Exercises](https://react-exercises.com/)

### Videos:
- [JSX Explained](https://www.youtube.com/watch?v=7fPXI_MnBOY)
- [React Components Tutorial](https://www.youtube.com/watch?v=Y2hgEGPzTZY)

## ✅ Module Completion Checklist

Before moving to the next module, make sure you can:
- [ ] Explain the difference between JSX and HTML
- [ ] Create functional components
- [ ] Use JavaScript expressions in JSX
- [ ] Implement conditional rendering
- [ ] Render lists of data
- [ ] Style components with CSS classes
- [ ] Organize components in separate files
- [ ] Build a multi-component application

## 🚀 What's Next?

In **Module 03**, we'll explore:
- Passing data between components with props
- Making components dynamic and reusable
- Component communication patterns
- Building more interactive applications

---

**Ready to continue?** 👉 **[Module 03: Props and Component Communication](../03-props-communication/README.md)**

---

*Estimated completion time: 4-6 hours*
