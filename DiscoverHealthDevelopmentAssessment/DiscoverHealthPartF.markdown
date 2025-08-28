# DiscoverHealth Part F – Reviews and Enhanced Map Functionality with Updated Authentication

## Introduction

In **DiscoverHealth Part F**, you’ll enhance the Part E application to meet Tasks 12 and 13 by adding a reviews API endpoint and integrating review forms into map marker popups, while incorporating updates to the login and signup system from Part E. Task 12 adds a POST `/api/resources/:id/reviews` endpoint with validation for resource ID, logged-in user, and non-empty review. Task 13 enhances the map in `SearchView.jsx` with review forms in popups, handling submissions and UI updates. Part E’s updates include a signup form and fixed login display showing “Logged in as {username}”. You’ll use **Windows Command Prompt** or **macOS Terminal**, building on the updated Part E.

### What You’ll Learn

* **Reviews API (Task 12)**: Create an endpoint to store reviews, validating inputs and authentication.
* **Map Enhancements (Task 13)**: Add review forms to marker popups, handle submissions, and update UI.
* **Authentication Updates (Part E)**: Integrate signup form and ensure username displays after login.
* **Dynamic UI**: Display reviews, errors, and user status in popups and resource lists.
* **Session Integration**: Ensure reviews and resource addition are tied to authenticated users.

### What You’ll Do

1. **Exercise A**: Set up `DiscoverHealthPartF` folder, copy updated Part E files, and verify dependencies.
2. **Exercise B**: Verify `server.js` for POST `/api/resources/:id/reviews` and reviews in GET `/api/resources`.
3. **Exercise C**: Update `SearchView.jsx` for review forms in popups, signup form, and username display.
4. **Exercise D**: Verify `App.jsx` and `AddResourceView.jsx`.
5. **Exercise E**: Run servers, test signup, login, username display, and review submission.
6. **Save Everything**: Organize work and (optionally) push to GitHub.

### Prerequisites

* Completed **DiscoverHealth Part E** with updated session-based login and signup.
* **Node.js** installed (check: `node --version`, `npm --version`).
* Familiarity with command-line (`cd`, `mkdir`, `copy`/`cp`).
* Part E project folder at `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartE`.
* `discoverhealth.db` with `users`, `healthcare_resources`, and `reviews` tables.

## Step 1: Understanding the Basics

### What is Part F?

**DiscoverHealth Part F** implements Tasks 12 and 13, building on Part E’s updated authentication:
* **Task 12**: Add POST `/api/resources/:id/reviews` to accept `resource_id` and `review`, validating ID, authentication, and review text.
* **Task 13**: Add review forms to map marker popups in `SearchView.jsx`, send reviews to the API, handle errors, and update the UI.
* **Part E Updates**: Include signup form, toggle between login/signup, and display “Logged in as {username}” reliably.

### How Part F Builds on Part E

* **Part E (Updated)**:
  * `server.js`: Session-based login, signup, logout, and restricted POST `/api/resources`.
  * `SearchView.jsx`: Index page with login/signup forms, username display, map, and resource addition.
  * `AddResourceView.jsx`: Manual resource addition.
* **Part F**:
  * Update `server.js` for reviews endpoint and include reviews in GET `/api/resources`.
  * Update `SearchView.jsx` for review forms in popups and review display, retaining signup/login.
  * Keep `App.jsx` and `AddResourceView.jsx` unchanged.

### Key Concepts

* **Reviews Endpoint**: Store reviews in `reviews` table with `resource_id` and `user_id`.
* **Map Popups**: Use Leaflet’s `bindPopup` with HTML review forms.
* **Dynamic UI**: Update markers and resource list after review submission.
* **Session Authentication**: Restrict reviews and resource addition to logged-in users.
* **Updated Authentication**: Signup form and reliable username display.

## Step 2: Exercise A – Set Up the Project

1. **Open Command Prompt (Windows) or Terminal (macOS)**:
   * **Windows**: `Win + R`, `cmd`, Enter.
   * **macOS**: Open Terminal via Spotlight.

2. **Create Project Folder**:
   ```bash
   cd %userprofile%\Desktop\Solent University\FinalProject  # Windows
   cd ~/Desktop/Solent University/FinalProject             # macOS
   mkdir DiscoverHealthPartF
   cd DiscoverHealthPartF
   ```

3. **Copy Files from Part E**:
   * Copy `discoverhealth.db` and `server.js`:
     ```bash
     # Windows
     copy ..\DiscoverHealthPartE\discoverhealth.db .
     copy ..\DiscoverHealthPartE\server.js .

     # macOS
     cp ../DiscoverHealthPartE/discoverhealth.db .
     cp ../DiscoverHealthPartE/server.js .
     ```
   * Copy `frontend` folder:
     ```bash
     # Windows
     xcopy ..\DiscoverHealthPartE\frontend frontend /s /e /h /i

     # macOS
     cp -R ../DiscoverHealthPartE/frontend .
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

6. **Verify Setup**:
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

## Step 3: Exercise B – Verify server.js for Reviews

Verify `server.js` supports reviews and authentication.

### B.1: Verify `server.js`

1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartF\server.js`

2. **Open**:
   ```bash
   notepad server.js  # Windows
   nano server.js    # macOS
   ```

3. **Verify Content**:
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
       // Fetch reviews for each resource
       const reviewStmt = db.prepare('SELECT review FROM reviews WHERE resource_id = ?');
       const resourcesWithReviews = resources.map(resource => ({
         ...resource,
         reviews: reviewStmt.all(resource.id).map(r => r.review)
       }));
       res.json(resourcesWithReviews);
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

   /* POST /api/resources/:id/reviews */
   app.post('/api/resources/:id/reviews', isAuthenticated, (req, res) => {
     const { id } = req.params;
     const { review } = req.body;
     if (!review?.trim()) {
       return res.status(400).json({ error: 'Review text is required' });
     }
     try {
       // Validate resource_id
       const resourceStmt = db.prepare('SELECT id FROM healthcare_resources WHERE id = ?');
       if (!resourceStmt.get(id)) {
         return res.status(404).json({ error: 'Resource not found' });
       }
       // Insert review
       const stmt = db.prepare(`
         INSERT INTO reviews (resource_id, user_id, review)
         VALUES (?, ?, ?)
       `);
       const result = stmt.run(id, req.session.userId, review.trim());
       res.status(201).json({ id: result.lastInsertRowid, message: 'Review added' });
     } catch (err) {
       console.error('[POST /api/resources/:id/reviews] DB error:', err);
       res.status(500).json({ error: 'Failed to add review' });
     }
   });

   app.listen(3000, () => {
     console.log('Server running at http://localhost:3000');
   });
   ```

4. **Save**: No changes needed.

5. **Code Explanation**:
   * **Verified**: Supports Task 12 (POST `/api/resources/:id/reviews`) and Part E’s signup/login.
   * **Function**: Handles reviews, authentication, and resource retrieval with reviews.

## Step 4: Exercise C – Update SearchView.jsx for Reviews and Authentication

Update `SearchView.jsx` to include review forms, signup form, and fixed login display.

### C.1: Update `SearchView.jsx`

1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartF\frontend\src\SearchView.jsx`

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
    * SearchView handles search, map display, login, signup, logout, and reviews on the index page.
    * Displays resources as a list and map markers with review forms in popups.
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
             const reviewForm = user ? `
               <div>
                 <h4>Add Review</h4>
                 <textarea id="review-${resource.id}" placeholder="Write your review" rows="2" style="width:100%;"></textarea>
                 <button onclick="submitReview(${resource.id})">Submit Review</button>
               </div>
             ` : '<p>Please log in to add a review.</p>';
             const reviewsList = resource.reviews.length > 0
               ? `<h4>Reviews:</h4><ul>${resource.reviews.map(r => `<li>${r}</li>`).join('')}</ul>`
               : '<p>No reviews yet.</p>';
             const popupContent = `
               <b>${resource.name}</b><br>
               ${resource.description}<br>
               ${reviewsList}
               ${reviewForm}
             `;
             const marker = L.marker([resource.lat, resource.lon])
               .addTo(mapInstanceRef.current)
               .bindPopup(popupContent);
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

     // Global function for review submission (called from popup)
     window.submitReview = async function(resourceId) {
       const textarea = document.getElementById(`review-${resourceId}`);
       const review = textarea.value.trim();
       if (!review) {
         alert('Review cannot be empty.');
         return;
       }
       try {
         const res = await fetch(`http://localhost:3000/api/resources/${resourceId}/reviews`, {
           method: 'POST',
           headers: { 'Content-Type': 'application/json' },
           body: JSON.stringify({ review }),
           credentials: 'include'
         });
         const data = await res.json();
         if (res.ok) {
           const resFetch = await fetch(`http://localhost:3000/api/resources?region=${encodeURIComponent(region)}`, {
             credentials: 'include'
           });
           if (resFetch.ok) {
             const updatedResources = await resFetch.json();
             setResources(updatedResources);
             setMessage('Review added successfully.');
             markersRef.current.forEach(marker => marker.remove());
             markersRef.current = [];
             updatedResources.forEach(resource => {
               const reviewForm = user ? `
                 <div>
                   <h4>Add Review</h4>
                   <textarea id="review-${resource.id}" placeholder="Write your review" rows="2" style="width:100%;"></textarea>
                   <button onclick="submitReview(${resource.id})">Submit Review</button>
                 </div>
               ` : '<p>Please log in to add a review.</p>';
               const reviewsList = resource.reviews.length > 0
                 ? `<h4>Reviews:</h4><ul>${resource.reviews.map(r => `<li>${r}</li>`).join('')}</ul>`
                 : '<p>No reviews yet.</p>';
               const popupContent = `
                 <b>${resource.name}</b><br>
                 ${resource.description}<br>
                 ${reviewsList}
                 ${reviewForm}
               `;
               const marker = L.marker([resource.lat, resource.lon])
                 .addTo(mapInstanceRef.current)
                 .bindPopup(popupContent);
               markersRef.current.push(marker);
             });
           }
         } else {
           setError(data.error || 'Failed to add review.');
         }
       } catch (err) {
         console.error(err);
         setError('An unexpected error occurred.');
       }
     };

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
             .bindPopup(`
               <b>${newResource.name}</b><br>
               ${newResource.description}<br>
               <p>No reviews yet.</p>
               ${user ? `
                 <div>
                   <h4>Add Review</h4>
                   <textarea id="review-${data.id}" placeholder="Write your review" rows="2" style="width:100%;"></textarea>
                   <button onclick="submitReview(${data.id})">Submit Review</button>
                 </div>
               ` : '<p>Please log in to add a review.</p>'}
             `);
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
               {resource.reviews.length > 0 && (
                 <>
                   <h4>Reviews:</h4>
                   <ul>
                     {resource.reviews.map((review, index) => (
                       <li key={index}>{review}</li>
                     ))}
                   </ul>
                 </>
               )}
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

## Step 5: Exercise D – Verify App.jsx and AddResourceView.jsx

Ensure unchanged files are intact.

### D.1: Verify `App.jsx`
1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartF\frontend\src\App.jsx`
2. **Check**: Matches Part F code (same as Part E).

### D.2: Verify `AddResourceView.jsx`
1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartF\frontend\src\AddResourceView.jsx`
2. **Check**: Matches Part F code (same as Part E).

## Step 6: Exercise E – Run and Test the Application

1. **Start Express Server**:
   ```bash
   cd C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartF
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
   cd C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartF\frontend
   npm run dev
   ```
   * Expect:
     ```
     VITE v7.1.3  ready in ...ms
     ➜  Local:   http://localhost:5173/
     ```

3. **Test Tasks 12, 13, and Updated Authentication**:
   * Visit `http://localhost:5173/`.
   * **Authentication (Part E Updates)**:
     - See login form; click “Need an account? Sign Up” to show signup form.
     - Sign up with new credentials (e.g., username: “newuser”, password: “test”).
     - Expect “Signup successful! Please log in as newuser.” and switch to login form.
     - Log in with valid credentials (e.g., username: “jsmith”, password: “test”).
     - Expect “Logged in as jsmith” with logout button.
     - Reload page; expect “Logged in as jsmith” to persist.
     - Test invalid credentials; expect error (e.g., “Invalid username or password”).
     - Test duplicate username in signup; expect “Username already exists”.
   * **Task 12**:
     - Log in, search for a region (e.g., “London”).
     - Click a marker to open popup; submit a review.
     - Expect review to appear in popup and resource list.
     - Try without login; expect “Please log in to add a review.”
     - Try empty review; expect alert: “Review cannot be empty.”
   * **Task 13**:
     - Verify review form in popups for logged-in users.
     - Submit review; expect “Review added successfully” and updated popup.
     - Try invalid resource ID via Postman; expect 404.
   * **Postman Tests**:
     - POST `/api/signup` with `{ "username": "testuser", "password": "testpass" }`; expect 201.
     - POST `/api/login` with above credentials; copy session cookie.
     - POST `/api/resources/:id/reviews` with `{ "review": "Great service!" }` and cookie; expect 201.
     - POST without cookie; expect 401.
     - POST with invalid `id`; expect 404.
     - POST with empty review; expect 400.

4. **Troubleshooting**:
   * **Username Not Showing**: Check GET `/api/user` response, ensure `credentials: 'include'`.
   * **Reviews Not Showing**: Verify `reviews` table and GET `/api/resources` response.
   * **Popup Errors**: Check `window.submitReview` and `credentials: 'include'`.
   * **Signup Fails**: Verify POST `/api/signup` response, check for duplicates.
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
   git commit -m "DiscoverHealth Part F: Updated with Signup and Login Display"
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
   git remote add origin https://github.com/<your-username>/DiscoverHealthPartF.git
   git branch -M main
   git push -u origin main
   ```

## Step 8: Summary and Next Steps

You’ve updated DiscoverHealth Part F, meeting Tasks 12 and 13 with Part E’s authentication enhancements! Recap:
* Copied updated Part E files.
* Verified `server.js` for reviews and authentication.
* Updated `SearchView.jsx` for review forms, signup form, and fixed login display.
* Tested signup, login, username display, and review submission.

**Next Steps**:
* Add review editing/deletion.
* Enhance popup styling with CSS.
* Implement review pagination for large datasets.
* Add password hashing with `bcrypt` for security.
