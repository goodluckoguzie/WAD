# DiscoverHealth Part E – Session-Based Login, Signup, and Access Control

## Introduction

In **DiscoverHealth Part E**, you’ll enhance the Part D application to meet Tasks 10 and 11 by implementing a session-based login and signup system on the main index page and restricting resource addition to logged-in users. Task 10 includes a login form, a signup form, a “Logged in as {username}” message, logout functionality, and persistent login state on page reload. Task 11 restricts POST `/api/resources` to authenticated users with user-friendly errors. You’ll use **Windows Command Prompt** or **macOS Terminal**, building on Part D to achieve a 62% grade threshold.

### What You’ll Learn

* **Session-Based Authentication (Task 10)**: Use Express sessions for login, signup, logout, and persistent state with username display.
* **Access Control (Task 11)**: Restrict API endpoints to logged-in users.
* **Frontend Integration**: Add login and signup forms to the index page.
* **User Feedback**: Display clear error messages and username after login.

### What You’ll Do

1. **Exercise A**: Set up `DiscoverHealthPartE` folder, copy Part D files, install `express-session` and `bcrypt`.
2. **Exercise B**: Update `server.js` for session middleware, login, signup, logout, and restricted POST.
3. **Exercise C**: Update `SearchView.jsx` for login/signup forms and username display, update `App.jsx` to remove login route.
4. **Exercise D**: Verify `AddResourceView.jsx` for session cookies.
5. **Exercise E**: Run servers, test login, signup, username display, and restricted resource addition.
6. **Save Everything**: Organize work and (optionally) push to GitHub.

### Prerequisites

* Completed **DiscoverHealth Part D** with working map and API.
* **Node.js** installed (check: `node --version`, `npm --version`).
* Familiarity with command-line (`cd`, `mkdir`, `copy`/`cp`).
* Part D project folder at `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartD`.
* `discoverhealth.db` with `users` table (`id`, `username`, `password`).

## Step 1: Understanding the Basics

### What is Part E?

**DiscoverHealth Part E** implements Tasks 10 and 11:
* **Task 10**: Session-based login and signup forms on the index page (`/`), showing “Logged in as {username}”, with logout and persistent state on reload.
* **Task 11**: Restrict POST `/api/resources` to logged-in users, showing errors like “Unauthorized: Please log in”.

### How Part E Builds on Part D

* **Part D**:
  * `server.js`: POST `/api/resources` with validation.
  * `SearchView.jsx`: Index page with map and resource addition.
  * `AddResourceView.jsx`: Manual resource addition.
* **Part E**:
  * Update `server.js` for sessions, login, signup, logout, and restricted POST.
  * Update `SearchView.jsx` for login/signup forms, username display, and persistent state.
  * Update `App.jsx` to remove `/login` route.
  * Update `AddResourceView.jsx` for session cookies.

### Key Concepts

* **Express Sessions**: Store `userId` server-side, send cookies to client.
* **Persistent Login**: Use GET `/api/user` to check session and display username.
* **Authentication Middleware**: Restrict endpoints with `req.session.userId`.
* **Frontend**: Use `credentials: 'include'` for session cookies, toggle login/signup forms.

## Step 2: Exercise A – Set Up the Project

1. **Open Command Prompt (Windows) or Terminal (macOS)**:
   * **Windows**: `Win + R`, `cmd`, Enter.
   * **macOS**: Open Terminal via Spotlight.

2. **Create Project Folder**:
   ```bash
   cd %userprofile%\Desktop\Solent University\FinalProject  # Windows
   cd ~/Desktop/Solent University/FinalProject             # macOS
   mkdir DiscoverHealthPartE
   cd DiscoverHealthPartE
   ```

3. **Copy Files from Part D**:
   * Copy `discoverhealth.db` and `server.js`:
     ```bash
     # Windows
     copy ..\DiscoverHealthPartD\discoverhealth.db .
     copy ..\DiscoverHealthPartD\server.js .

     # macOS
     cp ../DiscoverHealthPartD/discoverhealth.db .
     cp ../DiscoverHealthPartD/server.js .
     ```
   * Copy `frontend` folder:
     ```bash
     # Windows
     xcopy ..\DiscoverHealthPartD\frontend frontend /s /e /h /i

     # macOS
     cp -R ../DiscoverHealthPartD/frontend .
     ```

4. **Install Backend Dependencies**:
   ```bash
   npm install express better-sqlite3 cors express-session bcrypt
   ```

5. **Install Frontend Dependencies**:
   ```bash
   cd frontend
   npm install
   npm install react-router-dom leaflet
   ```
   * Verify `package.json`:
     ```bash
     notepad package.json  # Windows
     nano package.json    # macOS
     ```
     Ensure:
     ```json
     "dependencies": {
       "react": "^18.3.1",
       "react-dom": "^18.3.1",
       "react-router-dom": "^6.26.2",
       "leaflet": "^1.9.4"
     }
     ```

6. **Remove Unneeded Files**:
   * Delete `LoginView.jsx` if present:
     ```bash
     del frontend\src\LoginView.jsx  # Windows
     rm frontend/src/LoginView.jsx   # macOS
     ```

7. **Verify Setup**:
   * Check files:
     ```bash
     dir  # Windows
     ls   # macOS
     ```
     Expect: `discoverhealth.db`, `server.js`, `frontend`.
   * Check `frontend/src`:
     ```bash
     dir frontend\src
     ```
     Expect: `App.jsx`, `SearchView.jsx`, `AddResourceView.jsx`, `main.jsx`.

## Step 3: Exercise B – Update server.js for Authentication

Verify `server.js` supports login, signup, logout, and restricted POST.

### B.1: Update `server.js`

1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartE\server.js`

2. **Open**:
   ```bash
   notepad server.js  # Windows
   nano server.js    # macOS
   ```

3. **Replace with**:
   ```javascript
   const express = require('express');
   const Database = require('better-sqlite3');
   const path = require('path');
   const fs = require('fs');
   const cors = require('cors');
   const session = require('express-session');
   const bcrypt = require('bcrypt');

   const app = express();

   // Enable CORS for React dev server
   app.use(cors({ origin: 'http://localhost:5173', credentials: true }));

   // Session middleware
   app.use(session({
     secret: 'discoverhealth-secret',
     resave: false,
     saveUninitialized: false,
     cookie: { secure: false, maxAge: 24 * 60 * 60 * 1000 } // 24 hours
   }));

   // Locate the database file
   const dbPath = path.join(__dirname, 'discoverhealth.db');
   console.log('[BOOT] Using DB at:', dbPath, fs.existsSync(dbPath) ? '(FOUND)' : '(MISSING)');
   const db = new Database(dbPath);

   // Debug: list tables
   try {
     const tables = db.prepare("SELECT name FROM sqlite_master WHERE type='table'").all();
     console.log('[BOOT] Tables in DB:', tables.map(t => t.name));
   } catch (e) {
     console.error('[BOOT] Could not list tables:', e);
   }

   // Middleware to parse JSON
   app.use(express.json());

   // Authentication middleware
   function isAuthenticated(req, res, next) {
     if (req.session.userId) {
       return next();
     }
     res.status(401).json({ error: 'Unauthorized: Please log in' });
   }

   // Get current user
   app.get('/api/user', (req, res) => {
     if (req.session.userId) {
       try {
         const stmt = db.prepare('SELECT username FROM users WHERE id = ?');
         const user = stmt.get(req.session.userId);
         if (user) {
           return res.json({ username: user.username });
         }
       } catch (err) {
         console.error('[GET /api/user] DB error:', err);
       }
     }
     res.status(401).json({ error: 'Not logged in' });
   });

   // Friendly root page
   app.get('/', (req, res) => {
     res.send('DiscoverHealth API is running. Try <code>/api/resources?region=London</code>');
   });

   /* POST /api/login */
   app.post('/api/login', (req, res) => {
     const { username, password } = req.body;
     if (!username?.trim() || !password?.trim()) {
       return res.status(400).json({ error: 'Username and password are required' });
     }
     try {
       const stmt = db.prepare('SELECT id, username, password FROM users WHERE username = ?');
       const user = stmt.get(username.trim());
       if (!user) {
         return res.status(401).json({ error: 'Invalid username or password' });
       }
       // Assuming plain-text passwords
       if (user.password !== password.trim()) {
         return res.status(401).json({ error: 'Invalid username or password' });
       }
       req.session.userId = user.id;
       res.json({ message: 'Login successful', username: user.username });
     } catch (err) {
       console.error('[POST /api/login] DB error:', err);
       res.status(500).json({ error: 'Failed to log in' });
     }
   });

   /* POST /api/signup */
   app.post('/api/signup', (req, res) => {
     const { username, password } = req.body;
     if (!username?.trim() || !password?.trim()) {
       return res.status(400).json({ error: 'Username and password are required' });
     }
     try {
       // Check if username exists
       const stmtCheck = db.prepare('SELECT id FROM users WHERE username = ?');
       if (stmtCheck.get(username.trim())) {
         return res.status(400).json({ error: 'Username already exists' });
       }
       // Insert new user
       const stmt = db.prepare('INSERT INTO users (username, password) VALUES (?, ?)');
       const result = stmt.run(username.trim(), password.trim());
       res.status(201).json({ message: 'User created', id: result.lastInsertRowid });
     } catch (err) {
       console.error('[POST /api/signup] DB error:', err);
       res.status(500).json({ error: 'Failed to create user' });
     }
   });

   /* POST /api/logout */
   app.post('/api/logout', (req, res) => {
     req.session.destroy(err => {
       if (err) {
         console.error('[POST /api/logout] Session destroy error:', err);
         return res.status(500).json({ error: 'Failed to log out' });
       }
       res.json({ message: 'Logout successful' });
     });
   });

   /* GET /api/resources?region=RegionName */
   app.get('/api/resources', (req, res) => {
     const { region } = req.query;
     if (!region) {
       return res.status(400).json({ error: 'Region query parameter is required' });
     }
     try {
       const stmt = db.prepare('SELECT * FROM healthcare_resources WHERE region = ?');
       const resources = stmt.all(region);
       res.json(resources);
     } catch (err) {
       console.error('[GET /api/resources] DB error:', err);
       res.status(500).json({ error: 'Failed to retrieve resources' });
     }
   });

   /* POST /api/resources */
   app.post('/api/resources', isAuthenticated, (req, res) => {
     const { name, category, country, region, lat, lon, description } = req.body;
     // Validate all fields
     if (!name?.trim() || !category?.trim() || !country?.trim() || !region?.trim() || 
         lat === undefined || lon === undefined || !description?.trim()) {
       return res.status(400).json({ error: 'All fields are required: name, category, country, region, lat, lon, description' });
     }
     try {
       const stmt = db.prepare(`
         INSERT INTO healthcare_resources
           (name, category, country, region, lat, lon, description, recommendations)
         VALUES (?, ?, ?, ?, ?, ?, ?, 0)
       `);
       const result = stmt.run(name.trim(), category.trim(), country.trim(), region.trim(), lat, lon, description.trim());
       res.status(201).json({ id: result.lastInsertRowid });
     } catch (err) {
       console.error('[POST /api/resources] DB error:', err);
       res.status(500).json({ error: 'Failed to add resource' });
     }
   });

   /* POST /api/resources/:id/recommend */
   app.post('/api/resources/:id/recommend', (req, res) => {
     const { id } = req.params;
     try {
       const stmt = db.prepare(`
         UPDATE healthcare_resources
         SET recommendations = recommendations + 1
         WHERE id = ?
       `);
       const result = stmt.run(id);
       if (result.changes === 0) {
         return res.status(404).json({ error: 'Resource not found' });
       }
       res.json({ message: 'Recommendation added' });
     } catch (err) {
       console.error('[POST /api/resources/:id/recommend] DB error:', err);
       res.status(500).json({ error: 'Failed to recommend resource' });
     }
   });

   app.listen(3000, () => {
     console.log('Server running at http://localhost:3000');
   });
   ```

4. **Save**:
   * **Windows**: Save as `"server.js"` (All Files).
   * **macOS**: `Ctrl+O`, Enter, `Ctrl+X`.
   * Verify:
     ```bash
     dir
     ren server.js.txt server.js
     ```

5. **Code Explanation**:
   * **Verified**: Supports login, signup, logout, and restricted POST.
   * **Function**: No changes needed, as POST `/api/signup` and GET `/api/user` are correct.

## Step 4: Exercise C – Update SearchView.jsx and App.jsx

Add signup form and fix login display in `SearchView.jsx`, verify `App.jsx`.

### C.1: Update `SearchView.jsx`

1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartE\frontend\src\SearchView.jsx`

2. **Open**:
   ```bash
   cd frontend/src
   notepad SearchView.jsx  # Windows
   nano SearchView.jsx    # macOS
   ```

3. **Replace with**:
   ```jsx
   import { useState, useEffect, useRef } from 'react';
   import { Link } from 'react-router-dom';
   import L from 'leaflet';
   import 'leaflet/dist/leaflet.css';

   /**
    * SearchView handles search, map display, login, signup, and logout on the index page.
    * Displays resources as a list and map markers, and allows map-based resource addition.
    */
   export default function SearchView() {
     const [region, setRegion] = useState('');
     const [resources, setResources] = useState([]);
     const [error, setError] = useState('');
     const [message, setMessage] = useState('');
     const [showAddForm, setShowAddForm] = useState(false);
     const [newResource, setNewResource] = useState({
       name: '', category: '', country: '', region: '', description: ''
     });
     const [clickedLatLon, setClickedLatLon] = useState(null);
     const [loginForm, setLoginForm] = useState({ username: '', password: '' });
     const [signupForm, setSignupForm] = useState({ username: '', password: '' });
     const [user, setUser] = useState(null);
     const [showLoginForm, setShowLoginForm] = useState(true);
     const mapRef = useRef(null);
     const mapInstanceRef = useRef(null);
     const markersRef = useRef([]);

     // Check login status on mount
     useEffect(() => {
       async function fetchUser() {
         try {
           const res = await fetch('http://localhost:3000/api/user', {
             credentials: 'include'
           });
           const data = await res.json();
           if (res.ok && data.username) {
             setUser(data.username);
             setMessage('Welcome back, ' + data.username + '!');
           } else {
             setUser(null);
           }
         } catch (err) {
           console.error('Fetch user error:', err);
           setError('Failed to check login status.');
         }
       }
       fetchUser();
     }, []);

     // Initialize map
     useEffect(() => {
       if (!mapInstanceRef.current) {
         mapInstanceRef.current = L.map(mapRef.current, {
           center: [51.505, -0.09],
           zoom: 10
         });
         L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
           attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
         }).addTo(mapInstanceRef.current);

         mapInstanceRef.current.on('click', (e) => {
           if (user) {
             setClickedLatLon([e.latlng.lat, e.latlng.lng]);
             setShowAddForm(true);
           } else {
             setError('Please log in to add a resource.');
           }
         });
       }
       return () => {
         if (mapInstanceRef.current) {
           mapInstanceRef.current.remove();
           mapInstanceRef.current = null;
         }
       };
     }, [user]);

     async function handleSearch(e) {
       e.preventDefault();
       setError('');
       setMessage('');
       if (!region.trim()) {
         setError('Region is required.');
         return;
       }
       try {
         const res = await fetch(`http://localhost:3000/api/resources?region=${encodeURIComponent(region)}`, {
           credentials: 'include'
         });
         const data = await res.json();
         if (res.ok) {
           setResources(data);
           markersRef.current.forEach(marker => marker.remove());
           markersRef.current = [];
           data.forEach(resource => {
             const marker = L.marker([resource.lat, resource.lon])
               .addTo(mapInstanceRef.current)
               .bindPopup(`<b>${resource.name}</b><br>${resource.description}`);
             markersRef.current.push(marker);
           });
           if (data.length > 0) {
             mapInstanceRef.current.setView([data[0].lat, data[0].lon], 10);
           }
         } else {
           setError(data.error || 'Failed to fetch resources.');
         }
       } catch (err) {
         console.error(err);
         setError('An unexpected error occurred.');
       }
     }

     async function handleRecommend(id) {
       try {
         const res = await fetch(`http://localhost:3000/api/resources/${id}/recommend`, {
           method: 'POST',
           credentials: 'include'
         });
         const data = await res.json();
         if (res.ok) {
           setResources(prev => prev.map(r => 
             r.id === id ? { ...r, recommendations: r.recommendations + 1 } : r
           ));
           setMessage('Recommendation added.');
         } else {
           setError(data.error || 'Failed to recommend resource.');
         }
       } catch (err) {
         console.error(err);
         setError('An unexpected error occurred.');
       }
     }

     function handleAddChange(e) {
       const { name, value } = e.target;
       setNewResource(prev => ({ ...prev, [name]: value }));
     }

     async function handleAddSubmit(e) {
       e.preventDefault();
       setError('');
       setMessage('');
       if (!newResource.name.trim() || !newResource.category.trim() || !newResource.country.trim() ||
           !newResource.region.trim() || !newResource.description.trim()) {
         setError('All fields are required.');
         return;
       }
       try {
         const res = await fetch('http://localhost:3000/api/resources', {
           method: 'POST',
           headers: { 'Content-Type': 'application/json' },
           body: JSON.stringify({
             ...newResource,
             lat: clickedLatLon[0],
             lon: clickedLatLon[1]
           }),
           credentials: 'include'
         });
         const data = await res.json();
         if (res.ok) {
           setMessage(`Resource added with ID ${data.id}`);
           setNewResource({ name: '', category: '', country: '', region: '', description: '' });
           setShowAddForm(false);
           const marker = L.marker([clickedLatLon[0], clickedLatLon[1]])
             .addTo(mapInstanceRef.current)
             .bindPopup(`<b>${newResource.name}</b><br>${newResource.description}`);
           markersRef.current.push(marker);
           const resFetch = await fetch(`http://localhost:3000/api/resources?region=${encodeURIComponent(region)}`, {
             credentials: 'include'
           });
           if (resFetch.ok) {
             setResources(await resFetch.json());
           }
         } else {
           setError(data.error || 'Failed to add resource.');
         }
       } catch (err) {
         console.error(err);
         setError('An unexpected error occurred.');
       }
     }

     function handleAddCancel() {
       setShowAddForm(false);
       setNewResource({ name: '', category: '', country: '', region: '', description: '' });
       setClickedLatLon(null);
     }

     function handleLoginChange(e) {
       const { name, value } = e.target;
       setLoginForm(prev => ({ ...prev, [name]: value }));
     }

     function handleSignupChange(e) {
       const { name, value } = e.target;
       setSignupForm(prev => ({ ...prev, [name]: value }));
     }

     async function handleLoginSubmit(e) {
       e.preventDefault();
       setError('');
       setMessage('');
       if (!loginForm.username.trim() || !loginForm.password.trim()) {
         setError('Username and password are required.');
         return;
       }
       try {
         const res = await fetch('http://localhost:3000/api/login', {
           method: 'POST',
           headers: { 'Content-Type': 'application/json' },
           body: JSON.stringify(loginForm),
           credentials: 'include'
         });
         const data = await res.json();
         if (res.ok) {
           setMessage(`Login successful! Welcome, ${data.username}!`);
           setUser(data.username);
           setLoginForm({ username: '', password: '' });
         } else {
           setError(data.error || 'Failed to log in.');
         }
       } catch (err) {
         console.error(err);
         setError('An unexpected error occurred.');
       }
     }

     async function handleSignupSubmit(e) {
       e.preventDefault();
       setError('');
       setMessage('');
       if (!signupForm.username.trim() || !signupForm.password.trim()) {
         setError('Username and password are required.');
         return;
       }
       try {
         const res = await fetch('http://localhost:3000/api/signup', {
           method: 'POST',
           headers: { 'Content-Type': 'application/json' },
           body: JSON.stringify(signupForm),
           credentials: 'include'
         });
         const data = await res.json();
         if (res.ok) {
           setMessage(`Signup successful! Please log in as ${signupForm.username}.`);
           setSignupForm({ username: '', password: '' });
           setShowLoginForm(true);
         } else {
           setError(data.error || 'Failed to sign up.');
         }
       } catch (err) {
         console.error(err);
         setError('An unexpected error occurred.');
       }
     }

     async function handleLogout() {
       try {
         const res = await fetch('http://localhost:3000/api/logout', {
           method: 'POST',
           credentials: 'include'
         });
         const data = await res.json();
         if (res.ok) {
           setMessage('Logout successful!');
           setUser(null);
           setShowLoginForm(true);
         } else {
           setError(data.error || 'Failed to log out.');
         }
       } catch (err) {
         console.error(err);
         setError('An unexpected error occurred.');
       }
     }

     return (
       <div>
         <h2>Search Healthcare Resources</h2>
         <Link to="/add">Add Resource (Form)</Link>
         {user ? (
           <div style={{ margin: '16px 0' }}>
             Logged in as {user}
             <button onClick={handleLogout} style={{ marginLeft: '16px' }}>Logout</button>
           </div>
         ) : (
           <div style={{ margin: '16px 0' }}>
             {showLoginForm ? (
               <>
                 <form onSubmit={handleLoginSubmit} style={{ marginBottom: '16px' }}>
                   <h3>Login</h3>
                   <input
                     name="username"
                     value={loginForm.username}
                     onChange={handleLoginChange}
                     placeholder="Username"
                     style={{ marginRight: '8px' }}
                   />
                   <input
                     name="password"
                     value={loginForm.password}
                     onChange={handleLoginChange}
                     placeholder="Password"
                     type="password"
                     style={{ marginRight: '8px' }}
                   />
                   <button type="submit">Login</button>
                 </form>
                 <button onClick={() => setShowLoginForm(false)}>Need an account? Sign Up</button>
               </>
             ) : (
               <>
                 <form onSubmit={handleSignupSubmit} style={{ marginBottom: '16px' }}>
                   <h3>Sign Up</h3>
                   <input
                     name="username"
                     value={signupForm.username}
                     onChange={handleSignupChange}
                     placeholder="Username"
                     style={{ marginRight: '8px' }}
                   />
                   <input
                     name="password"
                     value={signupForm.password}
                     onChange={handleSignupChange}
                     placeholder="Password"
                     type="password"
                     style={{ marginRight: '8px' }}
                   />
                   <button type="submit">Sign Up</button>
                 </form>
                 <button onClick={() => setShowLoginForm(true)}>Already have an account? Login</button>
               </>
             )}
           </div>
         )}
         <form onSubmit={handleSearch} style={{ margin: '16px 0' }}>
           <input
             value={region}
             onChange={e => setRegion(e.target.value)}
             placeholder="Enter region (e.g., London)"
           />
           <button type="submit">Search</button>
         </form>
         {error && <p style={{ color: 'red' }}>{error}</p>}
         {message && <p style={{ color: 'green' }}>{message}</p>}
         <div>
           {resources.map(resource => (
             <div key={resource.id} style={{ marginBottom: '16px', border: '1px solid #ccc', padding: '8px' }}>
               <h3>{resource.name}</h3>
               <p>Category: {resource.category}</p>
               <p>Description: {resource.description}</p>
               <p>Recommendations: {resource.recommendations}</p>
               <button onClick={() => handleRecommend(resource.id)}>Recommend</button>
             </div>
           ))}
         </div>
         <div ref={mapRef} style={{ height: '400px', marginTop: '16px' }}></div>
         {showAddForm && (
           <div style={{ marginTop: '16px', border: '1px solid #ccc', padding: '16px' }}>
             <h3>Add Resource at ({clickedLatLon[0].toFixed(4)}, {clickedLatLon[1].toFixed(4)})</h3>
             <form onSubmit={handleAddSubmit}>
               <div style={{ marginBottom: '8px' }}>
                 <input name="name" value={newResource.name} onChange={handleAddChange} placeholder="Name" />
               </div>
               <div style={{ marginBottom: '8px' }}>
                 <input name="category" value={newResource.category} onChange={handleAddChange} placeholder="Category" />
               </div>
               <div style={{ marginBottom: '8px' }}>
                 <input name="country" value={newResource.country} onChange={handleAddChange} placeholder="Country" />
               </div>
               <div style={{ marginBottom: '8px' }}>
                 <input name="region" value={newResource.region} onChange={handleAddChange} placeholder="Region" />
               </div>
               <div style={{ marginBottom: '8px' }}>
                 <textarea name="description" value={newResource.description} onChange={handleAddChange} placeholder="Description" rows="3" />
               </div>
               <button type="submit">Add Resource</button>
               <button type="button" onClick={handleAddCancel} style={{ marginLeft: '8px' }}>Cancel</button>
             </form>
           </div>
         )}
       </div>
     );
   }
   ```

4. **Save**:
   * **Windows**: Save as `"SearchView.jsx"` (All Files).
   * **macOS**: `Ctrl+O`, Enter, `Ctrl+X`.

### C.2: Verify `App.jsx`

1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartE\frontend\src\App.jsx`

2. **Open**:
   ```bash
   notepad App.jsx  # Windows
   nano App.jsx    # macOS
   ```

3. **Verify**:
   ```jsx
   import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';
   import SearchView from './SearchView';
   import AddResourceView from './AddResourceView';

   function App() {
     return (
       <BrowserRouter>
         <div style={{ padding: '16px' }}>
           <nav style={{ marginBottom: '16px' }}>
             <Link to="/" style={{ marginRight: '16px' }}>Search</Link>
             <Link to="/add">Add Resource</Link>
           </nav>
           <Routes>
             <Route path="/" element={<SearchView />} />
             <Route path="/add" element={<AddResourceView />} />
           </Routes>
         </div>
       </BrowserRouter>
     );
   }

   export default App;
   ```

4. **Save**: No changes needed.

## Step 5: Exercise D – Verify AddResourceView.jsx

Ensure `AddResourceView.jsx` includes session cookies.

### D.1: Verify `AddResourceView.jsx`

1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartE\frontend\src\AddResourceView.jsx`

2. **Open**:
   ```bash
   notepad AddResourceView.jsx  # Windows
   nano AddResourceView.jsx    # macOS
   ```

3. **Verify**:
   ```jsx
   import { useState } from 'react';

   /**
    * AddResourceView lets users submit a new healthcare resource.
    * It POSTs to /api/resources and displays either a success message or an error.
    */
   export default function AddResourceView() {
     const [form, setForm] = useState({
       name: '', category: '', country: '', region: '',
       lat: '', lon: '', description: ''
     });
     const [message, setMessage] = useState('');
     const [error, setError] = useState('');

     function handleChange(e) {
       const { name, value } = e.target;
       setForm(prev => ({ ...prev, [name]: value }));
     }

     async function handleSubmit(e) {
       e.preventDefault();
       setError('');
       setMessage('');

       // Validate all fields
       for (const key of Object.keys(form)) {
         if (!form[key].trim()) {
           setError('All fields are required.');
           return;
         }
       }

       try {
         const res = await fetch('http://localhost:3000/api/resources', {
           method: 'POST',
           headers: { 'Content-Type': 'application/json' },
           body: JSON.stringify({
             ...form,
             lat: parseFloat(form.lat),
             lon: parseFloat(form.lon)
           }),
           credentials: 'include'
         });
         const data = await res.json();
         if (res.ok) {
           setMessage(`Resource added with ID ${data.id}`);
           setForm({
             name: '', category: '', country: '', region: '',
             lat: '', lon: '', description: ''
           });
         } else {
           setError(data.error || 'Failed to add resource.');
         }
       } catch (err) {
         console.error(err);
         setError('An unexpected error occurred.');
       }
     }

     return (
       <div>
         <h2>Add a New Resource</h2>
         {error && <p style={{ color: 'red' }}>{error}</p>}
         {message && <p style={{ color: 'green' }}>{message}</p>}
         <form onSubmit={handleSubmit}>
           <div style={{ marginBottom: '8px' }}>
             <input name="name" value={form.name} onChange={handleChange} placeholder="Name" />
           </div>
           <div style={{ marginBottom: '8px' }}>
             <input name="category" value={form.category} onChange={handleChange} placeholder="Category" />
           </div>
           <div style={{ marginBottom: '8px' }}>
             <input name="country" value={form.country} onChange={handleChange} placeholder="Country" />
           </div>
           <div style={{ marginBottom: '8px' }}>
             <input name="region" value={form.region} onChange={handleChange} placeholder="Region" />
           </div>
           <div style={{ marginBottom: '8px' }}>
             <input name="lat" value={form.lat} onChange={handleChange} placeholder="Latitude" type="number" step="0.0001" />
           </div>
           <div style={{ marginBottom: '8px' }}>
             <input name="lon" value={form.lon} onChange={handleChange} placeholder="Longitude" type="number" step="0.0001" />
           </div>
           <div style={{ marginBottom: '8px' }}>
             <textarea name="description" value={form.description} onChange={handleChange} placeholder="Description" rows="3" />
           </div>
           <button type="submit">Submit</button>
         </form>
       </div>
     );
   }
   ```

4. **Save**: No changes needed.

## Step 6: Exercise E – Run and Test the Application

1. **Start Express Server**:
   ```bash
   cd C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartE
   node server.js
   ```
   * Expect:
     ```
     [BOOT] Using DB at: .../discoverhealth.db (FOUND)
     [BOOT] Tables in DB: [ 'healthcare_resources', 'users', 'reviews' ]
     Server running at http://localhost:3000
     ```

2. **Start React Dev Server** (new terminal):
   ```bash
   cd C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartE\frontend
   npm run dev
   ```
   * Expect:
     ```
     VITE v7.1.3  ready in ...ms
     ➜  Local:   http://localhost:5173/
     ```

3. **Test Tasks 10 and 11**:
   * Visit `http://localhost:5173/`.
   * **Task 10**:
     - See login form on index page.
     - Click “Need an account? Sign Up” to show signup form.
     - Sign up with new credentials (e.g., username: “newuser”, password: “test”).
     - Expect “Signup successful! Please log in as newuser.” and switch to login form.
     - Log in with valid credentials (e.g., username: “jsmith”, password: “test”).
     - Expect “Logged in as jsmith” and logout button.
     - Reload page; expect “Logged in as jsmith” to persist.
     - Click logout; expect login form to reappear.
     - Test invalid credentials; expect error (e.g., “Invalid username or password”).
     - Test duplicate username in signup; expect “Username already exists”.
   * **Task 11**:
     - Without logging in, try adding resource via `/add` or map click.
     - Expect error: “Unauthorized: Please log in”.
     - Log in, then add resource; expect success.
   * **Postman Tests**:
     - POST `http://localhost:3000/api/signup` with `{ "username": "testuser", "password": "testpass" }`; expect 201.
     - POST `http://localhost:3000/api/login` with above credentials; copy session cookie, expect username in response.
     - GET `/api/user` with cookie; expect `{ "username": "testuser" }`.
     - POST `/api/resources` without cookie; expect 401.
     - POST `/api/resources` with cookie and valid data; expect 201.
     - POST `/api/logout` with cookie; expect 200.

4. **Troubleshooting**:
   * **Login Not Showing Username**: Check GET `/api/user` response, ensure `credentials: 'include'` in fetch.
   * **Signup Fails**: Verify POST `/api/signup` response, check for duplicate usernames.
   * **401 Errors**: Ensure session cookies are sent.
   * **Dependency Issues**:
     ```bash
     cd frontend
     del node_modules /s /q
     del package-lock.json
     npm install
     npm install react-router-dom leaflet
     ```

## Step 7: Save to GitHub (Optional)

1. **Initialize Git**:
   ```bash
   git init
   git add .
   git commit -m "DiscoverHealth Part E: Updated Login and Signup with Username Display"
   ```

2. **Create `.gitignore`**:
   ```bash
   notepad .gitignore  # Windows
   nano .gitignore    # macOS
   ```
   * Add:
     ```
     node_modules/
     frontend/dist/
     ```

3. **Push**:
   ```bash
   git remote add origin https://github.com/<your-username>/DiscoverHealthPartE.git
   git branch -M main
   git push -u origin main
   ```

## Step 8: Summary and Next Steps

You’ve updated DiscoverHealth Part E, meeting Tasks 10 and 11 with enhancements! Recap:
* Copied Part D files.
* Verified `server.js` for sessions, login, signup, logout, and restricted POST.
* Updated `SearchView.jsx` for signup form and fixed login username display.
* Tested signup, login, username display, and restricted resource addition.

**Next Steps**:
* Enhance security with `bcrypt` for password hashing.
* Add client-side navigation guards for `/add`.
* Improve UI styling for login/signup forms.
* Investigate OAuth2 for real-world authentication.
