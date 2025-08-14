# Week 7: Managing Data and State in React – Lists, Lifting State, and Deployment

## Introduction

This week, we’ll build on our React knowledge by learning how to display lists of data, how to make multiple components share and update the same data (using a technique called *lifting state up*), and how to nest components inside each other. We’ll also learn how to prepare our React app for production using Vite, so you can deploy your app or integrate it with your Node/Express backend from previous weeks. By the end, you’ll create a simple to-do list application and a music search app in React, and know how to build them for deployment.

### What You’ll Learn

- **Rendering Lists in React**: How to use JavaScript’s `map()` function in JSX to display arrays of data as lists of elements, and the importance of using keys for list items.
- **Nested Components**: Creating components that render other components inside them, organizing your UI into a tree of reusable pieces.
- **Lifting State Up**: Sharing state between components by moving state to their closest common parent. This lets two or more components communicate and stay in sync (e.g., a form input updating a list displayed elsewhere).
- **Using Vite in Production**: Building your React app for production with Vite, and serving or deploying the optimized build. You’ll learn how to generate a production-ready bundle and options for hosting it (including integrating with an Express server).
- **AJAX in React**: Making asynchronous requests to an Express API to fetch and display dynamic data, such as searching for songs in a music database.

### What You’ll Do

- **Exercise 1**: Render a list of items in React. You’ll create an array of data and use JSX with the array’s `.map()` method to display each item on the page as a list element.
- **Coding Exercise A**: Create a music search app using React components that fetches songs from a new Express API and SQLite database, demonstrating AJAX, list rendering, and state management.
- **Exercise 2**: Explore *lifting state up*. Through a conceptual breakdown, you’ll see why we sometimes need to move state to a parent component so that multiple child components can work with the same data (e.g., a form component and a list component sharing the same list of items).
- **Coding Exercise B**: Implement state lifting by adding a feature to add new items to your to-do list. You’ll create a form component for adding items, lift the list state up to the parent `App` component, and update the list when the form is submitted – making the new item appear instantly.
- **Build and Deploy**: Run the Vite production build for your React app. You’ll learn how to build the app into static files, preview it, and understand how to serve those files (either with a static server or by integrating with your Node.js/Express app).

### Summary

We’ll recap the key concepts learned in Week 7 and how they fit into your growing web development skills.

### Prerequisites

- You have completed Week 6 and have a working React project set up with Vite (the “Hello React World” app). Ensure Node.js and npm are installed, and that you can run the Vite development server.
- Knowledge of basic React concepts from last week: components, props, state (`useState`), and handling events with `onChange`.
- Familiarity with JavaScript arrays and the `Array.map()` method will be helpful, as well as concepts of functions and event handling from previous exercises.
- (Optional) Your Node/Express server from Weeks 1–4 if you plan to integrate the React frontend with it after building for production.

Now, let’s dive into managing lists and shared state in React!

## Step 1: Setting Up for Week 7

Before we start coding, let’s get our project ready:

1. **Continue from Week 6**: We’ll build off the React app you created in Week 6. Locate the folder (e.g., `WAD/week6`) that contains your React project. If you prefer to keep Week 6’s code intact, you can make a copy of that folder and rename it (e.g., `WAD/week7`) to use for this week’s exercises. To copy:
   ```
   cd %userprofile%\Desktop\WAD
   xcopy week6 week7 /E /H /C /I
   cd week7
   ```
2. **Open the project**: Using your editor (VS Code or Notepad), open the project folder. If using Command Prompt:
   ```
   notepad src\App.jsx
   ```
3. **Install dependencies (if needed)**: If you copied the project to a new folder, ensure Node modules are installed:
   ```
   npm install
   ```
   This installs React, Vite, and other dependencies.
4. **Start the development server**: Run the Vite dev server to see changes live:
   ```
   npm run dev
   ```
   Alternatively, use `npx vite dev`. This starts the server (usually at `http://localhost:5173`). Open this address in your browser to see your app (e.g., “Hello React World with Vite!”).

With the dev server running, you’re ready to start this week’s tasks. Keep the browser open to see live updates (thanks to Vite’s hot reload).

## Step 2: Exercise 1 – Rendering a List of Items

In this exercise, we’ll learn how to display a list of data in a React component using the array `map()` function in JSX to create repeated elements. We’ll render a static list of to-do items in our `App` component.

### 1.1: Create an Array of Data in State

First, add sample data to the `App` component using state to hold an array of to-do items. Using state will make it easier to expand our example later when we add or modify items.

Edit `src/App.jsx`:
1. Import the `useState` hook from React (if not already imported).
2. Inside the `App` component, create a state variable `items` with an initial array of strings (e.g., to-do tasks).

```javascript
import React, { useState } from 'react';

function App() {
  // Initial list of tasks
  const [items, setItems] = useState([
    'Learn React basics',
    'Build a React app',
    'Deploy the app'
  ]);

  return (
    <div className="App">
      <h1>My Task List</h1>
    </div>
  );
}

export default App;
```

**Breakdown**:
- Added `useState` import from `'react'`.
- Created `items` state with three example tasks.
- Included an `<h1>` heading “My Task List” to label the UI.
- Save the file; the browser should update to show the heading.

### 1.2: Use map() to Render `<li>` Elements

Now, display the `items` array as an unordered list (`<ul>`) by mapping each item to an `<li>` element.

Update the `return` in `App.jsx`:
```javascript
return (
  <div className="App">
    <h1>My Task List</h1>
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ul>
  </div>
);
```

**Breakdown**:
- `{items.map(...)}` loops over the `items` array in JSX.
- For each `item`, returns an `<li>` with the item’s text.
- `key={index}` ensures each `<li>` has a unique key (using array index for this static list).
- Save the file; the browser should show a bulleted list:
  - Learn React basics
  - Build a React app
  - Deploy the app

**Why use .map()**? It transforms the `items` array into an array of JSX elements, which React renders efficiently. **Keys**: Using `index` as the key is fine for static lists, but for dynamic lists (e.g., adding/removing items), a unique ID is better to avoid React re-rendering issues.

## Step 3: Coding Exercise A – Creating a Music Search App with AJAX

Instead of creating components for the to-do list, we’ll build a new React app from scratch for the HitTastic! music search, using AJAX to fetch songs from an Express API with a SQLite database. This demonstrates list rendering, state management, and AJAX integration, aligning with Week 6 concepts.

### A.1: Create a New React/Vite Project

1. **Create project folder**:
   ```
   cd %userprofile%\Desktop\WAD
   mkdir week7-hittastic
   cd week7-hittastic
   ```
2. **Initialize React/Vite project**:
   ```
   npm create vite@latest hittastic -- --template react
   cd hittastic
   ```
   Select “React” and “JavaScript” when prompted.
3. **Install server dependencies**:
   ```
   npm install express vite-express better-sqlite3
   ```
4. **Create server folder**:
   ```
   mkdir server
   ```

### A.2: Set Up SQLite Database

Create `server/create-database.mjs` to initialize `wadsongs.db`:
```
notepad server\create-database.mjs
```
Add:
```javascript
import Database from 'better-sqlite3';

// Initialize SQLite database
const db = new Database('wadsongs.db');

// Create songs table
const stmtSongs = db.prepare(`
    CREATE TABLE IF NOT EXISTS songs (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        artist TEXT,
        title TEXT,
        year INTEGER,
        price REAL,
        quantity_in_stock INTEGER
    )
`);
stmtSongs.run();

// Add sample songs
const insertSong = db.prepare(`
    INSERT INTO songs (artist, title, year, price, quantity_in_stock) 
    VALUES (?, ?, ?, ?, ?)
`);
insertSong.run('The Beatles', 'Hey Jude', 1968, 1.29, 5);
insertSong.run('The Beatles', 'Let It Be', 1970, 0.99, 3);
insertSong.run('Queen', 'Bohemian Rhapsody', 1975, 1.49, 4);
insertSong.run('Adele', 'Hello', 2015, 1.29, 6);

console.log('Database setup complete!');
db.close();
```
Run:
```
node server\create-database.mjs
```

### A.3: Set Up Express Server with vite-express

Create `server.mjs`:
```
notepad server.mjs
```
Add:
```javascript
import express from 'express';
import ViteExpress from 'vite-express';
import Database from 'better-sqlite3';

// Initialize Express and database
const app = express();
app.use(express.json());

const db = new Database('wadsongs.db');

// API route: GET /songs/artist/:artist
app.get('/songs/artist/:artist', (req, res) => {
    try {
        const stmt = db.prepare('SELECT * FROM songs WHERE artist = ?');
        const songs = stmt.all(req.params.artist);
        if (songs.length === 0) {
            res.status(404).json({ error: 'No songs found for this artist' });
        } else {
            res.json(songs);
        }
    } catch (err) {
        res.status(500).json({ error: 'Server error: ' + err.message });
    }
});

// Start server with Vite middleware
ViteExpress.listen(app, 3000, () => console.log('Server is listening on port 3000...'));
```

### A.4: Create Search Component

Update `src/App.jsx`:
```
notepad src\App.jsx
```
Replace with:
```javascript
import { useState } from 'react';

function App() {
    const [artist, setArtist] = useState('');
    const [results, setResults] = useState([]);

    const handleSearch = async () => {
        if (!artist) {
            setResults([]);
            return;
        }
        try {
            const response = await fetch(`/songs/artist/${encodeURIComponent(artist)}`);
            if (!response.ok) {
                throw new Error(`HTTP error: ${response.status}`);
            }
            const data = await response.json();
            setResults(data);
        } catch (err) {
            console.error('Search failed:', err);
            setResults([]);
        }
    };

    return (
        <div style={{ padding: '20px' }}>
            <h2>HitTastic Music Search</h2>
            <input
                type="text"
                placeholder="Enter artist name..."
                value={artist}
                onChange={(e) => setArtist(e.target.value)}
            />
            <button onClick={handleSearch}>Search</button>
            <div style={{ marginTop: '10px', fontStyle: 'italic' }}>
                {artist ? `You entered: ${artist}` : 'Please enter an artist name.'}
            </div>
            <div style={{ marginTop: '20px' }}>
                {results.length > 0 ? (
                    results.map((song) => (
                        <div key={song.id}>
                            {song.title} by <strong>{song.artist}</strong> ({song.year})
                            {song.price ? ` - £${song.price}` : ''}
                            {song.quantity_in_stock !== undefined ? ` [Stock: ${song.quantity_in_stock}]` : ''}
                        </div>
                    ))
                ) : (
                    <p>{artist ? `No songs found for "${artist}".` : 'No results to display.'}</p>
                )}
            </div>
        </div>
    );
}

export default App;
```

### A.5: Test the App

1. Install dependencies:
   ```
   npm install
   ```
2. Start the server:
   ```
   node server.mjs
   ```
3. Visit `http://localhost:3000`. Test searching for “The Beatles” (should show songs) and “Unknown” (should show “No songs found”).
4. Check the console (F12) for errors.

### A.6: Build and Run in Production

1. Build:
   ```
   npm run build
   ```
2. Run in production (Windows):
   ```
   set NODE_ENV=production && node server.mjs
   ```
   Verify at `http://localhost:3000`. Check `dist` folder for bundled files.

**Why this matters**: This creates a standalone music search app using React, AJAX, and Express, demonstrating list rendering (`map()`), state, and production builds.

## Step 4: Exercise 2 – Lifting State Up (Sharing State Between Components)

Our music search app uses a single component, but the to-do list app will require multiple components to share state. Let’s explore *lifting state up* for the to-do list.

### 2.1: What Does "Lifting State Up" Mean?

Lifting state up means moving state to the nearest common ancestor of components that need it. Instead of each component managing its own data, the parent holds the state and passes it down via props. Children communicate updates via callback functions.

For the to-do list:
- A `NewTaskForm` component adds tasks.
- A `TaskList` component displays tasks.
- Both need access to the task list, so the state lives in their parent (`App`).

**Example**:
- `App` holds `items` state (the task list).
- `NewTaskForm` calls a function (via props) to add tasks to `items`.
- `TaskList` receives `items` via props to display.

### 2.2: Identifying When to Lift State

Lift state up when:
- Sibling components need the same data (e.g., form and list sharing tasks).
- A child needs to update parent-owned data (e.g., form adding tasks).
- You need a single source of truth to avoid sync issues.

In our to-do list:
- Keep `items` in `App`.
- Pass `items` to `TaskList` as a prop.
- Pass an `addTask` function to `NewTaskForm` to update `items`.

## Step 5: Coding Exercise B – Adding New Items with Lifted State

Let’s make the to-do list interactive by adding a form to add tasks, with state in `App`.

### B.1: Create AddTaskForm Component

Create `src/AddTaskForm.jsx`:
```
notepad src\AddTaskForm.jsx
```
Add:
```javascript
import React, { useState } from 'react';

function AddTaskForm(props) {
  const { onAddTask } = props;
  const [inputValue, setInputValue] = useState('');

  const handleChange = (e) => {
    setInputValue(e.target.value);
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    if (inputValue.trim() === '') {
      return;
    }
    onAddTask(inputValue.trim());
    setInputValue('');
  };

  return (
    <form onSubmit={handleSubmit}>
      <input 
        type="text" 
        placeholder="New task..." 
        value={inputValue} 
        onChange={handleChange} 
      />
      <button type="submit">Add Task</button>
    </form>
  );
}

export default AddTaskForm;
```

### B.2: Update App to Use AddTaskForm

Update `src/App.jsx`:
```
notepad src\App.jsx
```
Replace with:
```javascript
import React, { useState } from 'react';
import TodoItem from './TodoItem';
import AddTaskForm from './AddTaskForm';

function App() {
  const [items, setItems] = useState([
    'Learn React basics',
    'Build a React app',
    'Deploy the app'
  ]);

  function addTask(taskText) {
    setItems(prevItems => [...prevItems, taskText]);
  }

  return (
    <div className="App">
      <h1>My Task List</h1>
      <AddTaskForm onAddTask={addTask} />
      <ul>
        {items.map((item, index) => (
          <TodoItem key={index} item={item} />
        ))}
      </ul>
    </div>
  );
}

export default App;
```

### B.3: Create TodoItem Component

Create `src/TodoItem.jsx`:
```
notepad src\TodoItem.jsx
```
Add:
```javascript
import React from 'react';

function TodoItem(props) {
  const { item } = props;
  return (
    <li>{item}</li>
  );
}

export default TodoItem;
```

### B.4: Test Adding a New Item

1. Run:
   ```
   npm run dev
   ```
2. Visit `http://localhost:5173`. Add a task (e.g., “Learn lifting state”) and verify it appears in the list.

## Step 6: Using Vite in Production

### 6.1: Building the Application

1. Stop the dev server (`Ctrl + C`).
2. Run:
   ```
   npm run build
   ```
   This creates a `dist` folder with optimized files.

### 6.2: Previewing the Production Build

Run:
```
npm run preview
```
Visit `http://localhost:4173` to test the production build.

### 6.3: Deploying or Serving the Build

- **Static Hosting**: Upload `dist` to services like Netlify or GitHub Pages.
- **Express Integration**: Copy `dist` to your Express server’s static folder (e.g., `public`) and use `app.use(express.static('public'))`.
- **Local Server**: Use `serve -s dist` (after `npm install -g serve`).

## Step 7: Summary and Key Takeaways

You’ve completed Week 7! You:
- Rendered lists using `map()` and keys.
- Built a music search app with AJAX, state, and list rendering.
- Created nested components (`TodoItem`, `AddTaskForm`).
- Lifted state up to share data between components.
- Built and previewed a production-ready app with Vite.

**Next Steps**:
- Enhance the to-do list with delete/edit features.
- Integrate the music search with the Express server from Weeks 1–4.
- Explore React Router or state management libraries.

Happy coding, and see you next week!
