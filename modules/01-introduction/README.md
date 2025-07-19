# Module 01: Introduction to React & Development Environment

**Duration:** 1 Week | **Difficulty:** Beginner

## 🎯 Learning Objectives

By the end of this module, you will:
- ✅ Understand what React is and why it's popular
- ✅ Set up a complete React development environment
- ✅ Create your first React application
- ✅ Understand the React ecosystem and MERN stack context
- ✅ Know the basic project structure and development workflow

## 📚 What is React?

React is a **JavaScript library** for building user interfaces, especially web applications. Think of it as a tool that helps you create interactive websites more efficiently.

### Why React?
- **Component-Based**: Build encapsulated components that manage their own state
- **Declarative**: Describe what your UI should look like, React handles the how
- **Learn Once, Write Anywhere**: Use React for web, mobile (React Native), and more
- **Huge Community**: Massive ecosystem with tons of resources and job opportunities

### React in the MERN Stack
**MERN** stands for:
- **M**ongoDB - Database
- **E**xpress.js - Backend framework
- **R**eact - Frontend library (what we're learning!)
- **N**ode.js - JavaScript runtime for backend

React handles the **frontend** (what users see and interact with), while the other technologies handle data storage and server-side logic.

## 🛠️ Development Environment Setup

### Prerequisites
Before we start, make sure you have:
- Basic knowledge of HTML, CSS, and JavaScript
- A text editor (VS Code recommended)
- A modern web browser (Chrome, Firefox, Safari, Edge)

### Step 1: Install Node.js
1. Go to [nodejs.org](https://nodejs.org/)
2. Download the LTS (Long Term Support) version
3. Install it following the installer instructions
4. Verify installation by opening terminal/command prompt and running:
   ```bash
   node --version
   npm --version
   ```

### Step 2: Install VS Code Extensions
Install these helpful extensions in VS Code:
- **ES7+ React/Redux/React-Native snippets** - Code shortcuts
- **Prettier - Code formatter** - Auto-format your code
- **Auto Rename Tag** - Automatically rename paired HTML/JSX tags
- **Bracket Pair Colorizer** - Color-code matching brackets

### Step 3: Create Your First React App
React provides a tool called **Create React App** that sets up everything for you:

```bash
# Install create-react-app globally (one-time setup)
npm install -g create-react-app

# Create a new React project
npx create-react-app my-first-react-app

# Navigate to the project folder
cd my-first-react-app

# Start the development server
npm start
```

Your browser should automatically open to `http://localhost:3000` showing your React app!

## 📁 Project Structure Explained

When you create a React app, you'll see this structure:

```
my-first-react-app/
├── public/
│   ├── index.html          # Main HTML file
│   └── favicon.ico         # Website icon
├── src/
│   ├── App.js              # Main React component
│   ├── App.css             # Styles for App component
│   ├── index.js            # Entry point of the app
│   └── index.css           # Global styles
├── package.json            # Project dependencies and scripts
└── README.md               # Project documentation
```

### Key Files to Know:
- **`src/index.js`** - Where React starts running
- **`src/App.js`** - Your main component (we'll learn about components next!)
- **`public/index.html`** - The HTML template
- **`package.json`** - Lists all the packages your project uses

## 💻 Your First React Code

Let's look at the default `App.js` file:

```jsx
import logo from './logo.svg';
import './App.css';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
      </header>
    </div>
  );
}

export default App;
```

**What's happening here?**
- We're creating a **function** called `App`
- This function **returns** what looks like HTML (it's actually JSX!)
- We **export** this function so other files can use it

## 🏋️ Hands-on Exercise

### Exercise 1: Customize Your First App
1. Open `src/App.js` in your editor
2. Change the text "Learn React" to "Hello, I'm learning React!"
3. Save the file and watch it automatically update in the browser
4. Try changing other text and see the live updates

### Exercise 2: Add Your Personal Touch
1. Replace the React logo with a welcome message
2. Add your name to the page
3. Change some colors in `App.css`

## 🎯 Challenge: Create a Personal Introduction

Create a simple personal introduction page by modifying `App.js`:

**Requirements:**
- Display your name as a heading
- Add a short bio about yourself
- Include your learning goals for React
- Style it to look nice

**Hint:** You can use regular HTML tags like `<h1>`, `<p>`, `<ul>`, etc. inside the return statement.

## 🔧 Development Workflow

Here are the essential commands you'll use:

```bash
# Start development server (run this to work on your app)
npm start

# Build for production (creates optimized version)
npm run build

# Run tests (we'll cover this later)
npm test

# Install new packages
npm install package-name
```

## 📚 Additional Resources

### Essential Reading:
- [React Official Documentation](https://reactjs.org/docs/getting-started.html)
- [Create React App Documentation](https://create-react-app.dev/)
- [MDN JavaScript Refresher](https://developer.mozilla.org/en-US/docs/Web/JavaScript)

### Helpful Tools:
- [React Developer Tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi) - Browser extension
- [CodePen](https://codepen.io/) - Online code playground
- [CodeSandbox](https://codesandbox.io/) - Online React development environment

### Videos:
- [React in 100 Seconds](https://www.youtube.com/watch?v=Tn6-PIqc4UM)
- [What is React?](https://www.youtube.com/watch?v=N3AkSS5hXMA)

## ✅ Module Completion Checklist

Before moving to the next module, make sure you can:
- [ ] Explain what React is in simple terms
- [ ] Set up a new React project using Create React App
- [ ] Navigate the basic project structure
- [ ] Make simple changes to a React component
- [ ] Start and stop the development server
- [ ] Understand how React fits into the MERN stack

## 🚀 What's Next?

In **Module 02**, we'll dive into:
- JSX syntax and how it works
- Creating your first custom components
- Understanding the difference between HTML and JSX
- Building reusable UI pieces

---

**Ready to continue?** 👉 **[Module 02: JSX and Components](../02-jsx-components/README.md)**

---

*Estimated completion time: 3-5 hours*
