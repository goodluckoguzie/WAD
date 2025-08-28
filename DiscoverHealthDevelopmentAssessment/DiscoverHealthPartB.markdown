# Week 6: DiscoverHealth Part B – Building a React Front-End for the Healthcare API

## Introduction

In **DiscoverHealth Part B**, you’ll create a web front-end using **React** and **Vite** to interact with the REST API from Part A. This front-end enables users to search for healthcare resources by region, add new resources, and "like" (recommend) resources via asynchronous (AJAX) requests. You’ll use **Windows Command Prompt** or **macOS Terminal** to set up the project, resulting in a dynamic web interface communicating with your Part A server. This satisfies the assignment’s requirement for a “more advanced solution” using React’s component-based architecture.

### What You’ll Learn

* **React Components**: Build reusable UI pieces (e.g., search form, resource list).
* **Routing with React Router**: Navigate between pages (Search, Add Resource) without full reloads.
* **AJAX with Fetch**: Make asynchronous HTTP requests to the Part A API.
* **State Management**: Use React’s `useState` hook for form inputs, results, and messages.
* **CORS**: Configure the backend for cross-origin requests from the React dev server.
* **Event Handling**: Respond to form submissions and button clicks.
* **Dynamic UI**: Display search results and update recommendation counts in real-time.

### What You’ll Do

1. **Exercise A**: Set up a project folder, copy Part A files (`discoverhealth.db`, `server.js`), and install backend dependencies including `cors`.
2. **Exercise B**: Modify the Express server to include CORS middleware.
3. **Exercise C**: Create a React project with Vite, building:
   * `App.jsx`: Navigation and routing.
   * `SearchView.jsx`: Search form and resource list with recommendation functionality.
   * `AddResourceView.jsx`: Form to add resources.
4. **Exercise D**: Run both servers, test the interface in a browser, and optionally with Postman/RESTer.
5. **Save Everything**: Organize work and (optionally) push to GitHub.

### Prerequisites

* Completed **DiscoverHealth Part A** with a working API, `discoverhealth.db`, and `server.js`.
* **Node.js** installed (check: `node --version`, `npm --version`).
* Familiarity with command-line commands (`cd`, `mkdir`) and basic **JavaScript**, **JSON**, and **React** concepts.
* A browser and optionally Postman/RESTer.
* Install Node.js from [nodejs.org](https://nodejs.org) if needed.

## Step 1: Understanding the Basics

### What is Part B?

**DiscoverHealth Part B** is the front-end for the Part A API, allowing users to:
* Search for resources by region, displaying name, category, description, and recommendations (GET requests).
* Add resources via a form (POST requests).
* Recommend resources with a button (POST requests).

The front-end uses **AJAX** (`fetch` API) to communicate with the API at `http://localhost:3000`. **React Router** handles navigation, and **CORS** ensures the React dev server (`http://localhost:5173`) can access the API.

### How Part B Connects to Part A

* **Part A API Endpoints**:
  * `GET /api/resources?region=RegionName`: Returns resources for a region.
  * `POST /api/resources`: Adds a resource.
  * `POST /api/resources/:id/recommend`: Increments recommendations.
* **React Front-End**:
  * Uses GET/POST requests to fetch/send data.
  * Updates UI dynamically based on API responses.
* **CORS**: Required due to different ports (`5173` vs `3000`).

### Key React Concepts

* **Components**: `SearchView` and `AddResourceView` within `App`.
* **Routing**: `react-router-dom` for navigation (`/`, `/add`).
* **State with `useState`**: Manages inputs, results, messages.
* **AJAX with `fetch`**: Asynchronous API calls with `async/await`.
* **Event Handling**: Handles `onSubmit` and `onClick`.

### Putting It Together

* Reuse Part A’s database and server, adding CORS.
* Create a React project with Vite, building components.
* Use AJAX to connect to the API, displaying results dynamically.
* Vite’s hot module reloading updates the browser during development.

## Step 2: Exercise A – Set Up the Project

1. **Open Command Prompt (Windows) or Terminal (macOS)**:
   * **Windows**: `Win + R`, `cmd`, Enter.
   * **macOS**: Open Terminal via Spotlight.
   * Verify:
     ```bash
     node --version
     npm --version
     ```
     Install Node.js from [nodejs.org](https://nodejs.org) if missing.

2. **Create a Project Folder**:
   ```bash
   cd %userprofile%\Desktop  # Windows
   cd ~/Desktop             # macOS
   mkdir DiscoverHealthPartB
   cd DiscoverHealthPartB
   ```

3. **Copy Files from Part A**:
   * Copy `discoverhealth.db`, `server.js`, and (optionally) `init-db.js`:
     ```bash
     # Windows
     copy ..\DiscoverHealthPartA\discoverhealth.db .
     copy ..\DiscoverHealthPartA\server.js .
     copy ..\DiscoverHealthPartA\init-db.js .

     # macOS
     cp ../DiscoverHealthPartA/discoverhealth.db .
     cp ../DiscoverHealthPartA/server.js .
     cp ../DiscoverHealthPartA/init-db.js .
     ```
   * Verify:
     ```bash
     dir  # Windows
     ls   # macOS
     ```

4. **Install Backend Dependencies**:
   ```bash
   npm install express better-sqlite3 cors
   ```
   * Creates `node_modules` and `package-lock.json`.

5. **Verify Setup**:
   * Optionally run:
     ```bash
     node init-db.js
     ```
     Skip if `discoverhealth.db` is populated.

## Step 3: Exercise B – Modify the Express Server for CORS

1. **Open `server.js`**:
   ```bash
   notepad server.js  # Windows
   nano server.js    # macOS
   ```

2. **Add CORS**:
   * After `require` statements:
     ```javascript
     const cors = require('cors');
     ```
   * After `const app = express();`:
     ```javascript
     app.use(cors({ origin: 'http://localhost:5173', credentials: true }));
     ```
   * Full `server.js`:
     ```javascript
     const express = require('express');
     const Database = require('better-sqlite3');
     const path = require('path');
     const fs = require('fs');
     const cors = require('cors');

     const app = express();

     // Enable CORS for React dev server
     app.use(cors({ origin: 'http://localhost:5173', credentials: true }));

     // Locate the database file
     const dbPath = path.join(__dirname, 'discoverhealth.db');
     console.log('[BOOT] Using DB at:', dbPath, fs.existsSync(dbPath) ? '(FOUND)' : '(MISSING)');
     const db = new Database(dbPath);

     // Debug: list tables so you know the DB is correct
     try {
       const tables = db.prepare("SELECT name FROM sqlite_master WHERE type='table'").all();
       console.log('[BOOT] Tables in DB:', tables.map(t => t.name));
     } catch (e) {
       console.error('[BOOT] Could not list tables:', e);
     }

     // Middleware to parse JSON in request bodies
     app.use(express.json());

     // Friendly root page
     app.get('/', (req, res) => {
       res.send('DiscoverHealth API is running. Try <code>/api/resources?region=London</code>');
     });

     /* 1. GET /api/resources?region=RegionName */
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

     /* 2. POST /api/resources */
     app.post('/api/resources', (req, res) => {
       const { name, category, country, region, lat, lon, description } = req.body;
       try {
         const stmt = db.prepare(`
           INSERT INTO healthcare_resources
             (name, category, country, region, lat, lon, description, recommendations)
           VALUES (?, ?, ?, ?, ?, ?, ?, 0)
         `);
         const result = stmt.run(name, category, country, region, lat, lon, description);
         res.status(201).json({ id: result.lastInsertRowid });
       } catch (err) {
         console.error('[POST /api/resources] DB error:', err);
         res.status(500).json({ error: 'Failed to add resource' });
       }
     });

     /* 3. POST /api/resources/:id/recommend */
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

3. **Save**:
   * **Windows**: File > Save, close.
   * **macOS**: `Ctrl+O`, Enter, `Ctrl+X`.

## Step 4: Exercise C – Create the React Front-End

### C.1: Scaffold the React App

1. **Create Project**:
   * In `DiscoverHealthPartB`:
     ```bash
     npm create vite@latest frontend -- --template react
     ```
     * Select **React**, **JavaScript**.

2. **Install Dependencies**:
   ```bash
   cd frontend
   npm install
   npm install react-router-dom
   ```
   * **Critical**: `react-router-dom` is required for routing. Skipping this causes errors like “Failed to resolve import ‘react-router-dom’”.
   * Verify in `package.json`:
     ```bash
     notepad package.json  # Windows
     nano package.json    # macOS
     ```
     Look for:
     ```json
     "dependencies": {
       "react": "^18.3.1",
       "react-dom": "^18.3.1",
       "react-router-dom": "^6.26.2"
     }
     ```

3. **Project Structure**:
   * `frontend` contains:
     * `index.html`: HTML template.
     * `src/main.jsx`: Entry point.
     * `src/App.jsx`: Root component (replace default).
     * Create `src/SearchView.jsx`, `src/AddResourceView.jsx`.

### C.2: Edit `src/App.jsx`

1. **Open**:
   ```bash
   cd frontend/src
   notepad App.jsx  # Windows
   nano App.jsx    # macOS
   ```

2. **Replace**:
   * Replace the default counter code (with Vite/React logos) with:
     ```jsx
     import { BrowserRouter as Router, Routes, Route, Link } from 'react-router-dom';
     import SearchView from './SearchView';
     import AddResourceView from './AddResourceView';

     /**
      * App is the root component. It sets up navigation and routes.
      */
     export default function App() {
       return (
         <Router>
           <div style={{ padding: '20px', fontFamily: 'Arial, sans-serif' }}>
             <h1>DiscoverHealth</h1>
             <nav>
               <Link to="/" style={{ marginRight: '10px' }}>Search Resources</Link>
               <Link to="/add">Add Resource</Link>
             </nav>
             <hr />
             <Routes>
               <Route path="/" element={<SearchView />} />
               <Route path="/add" element={<AddResourceView />} />
             </Routes>
           </div>
         </Router>
       );
     }
     ```

3. **Save**:
   * **Windows**: Save as `"App.jsx"` (All Files) to avoid `.txt`.
   * **macOS**: `Ctrl+O`, Enter, `Ctrl+X`.
   * **Verify**: Ensure it’s not `App.jsx.txt`:
     ```bash
     dir
     ren App.jsx.txt App.jsx
     ```

4. **Note**: If you see the default Vite + React page (with logos and counter), you didn’t replace `App.jsx` correctly.

### C.3: Create `src/SearchView.jsx`

1. **Create**:
   ```bash
   notepad SearchView.jsx  # Windows
   nano SearchView.jsx    # macOS
   ```

2. **Add**:
   ```jsx
   import { useState } from 'react';

   /**
    * SearchView lets users search for resources by region.
    * It displays results and allows each to be recommended.
    */
   export default function SearchView() {
     const [region, setRegion] = useState('');
     const [results, setResults] = useState([]);
     const [error, setError] = useState('');

     async function handleSearch(e) {
       e.preventDefault();
       setResults([]);
       setError('');
       if (!region.trim()) return;

       try {
         const res = await fetch(`http://localhost:3000/api/resources?region=${encodeURIComponent(region.trim())}`);
         if (!res.ok) {
           const err = await res.json();
           setError(err.error || `Error ${res.status}`);
           return;
         }
         const data = await res.json();
         setResults(data);
       } catch (err) {
         console.error(err);
         setError('An unexpected error occurred.');
       }
     }

     async function handleRecommend(id, index) {
       try {
         const res = await fetch(`http://localhost:3000/api/resources/${id}/recommend`, { method: 'POST' });
         if (res.ok) {
           setResults(old => old.map((r, i) => (
             i === index ? { ...r, recommendations: r.recommendations + 1 } : r
           )));
         } else {
           alert('Failed to recommend.');
         }
       } catch (err) {
         console.error(err);
         alert('Error recommending.');
       }
     }

     return (
       <div>
         <form onSubmit={handleSearch}>
           <label>
             Region:
             <input
               type="text"
               value={region}
               onChange={e => setRegion(e.target.value)}
               placeholder="e.g., London"
               style={{ marginLeft: '8px' }}
             />
           </label>
           <button type="submit" style={{ marginLeft: '8px' }}>Search</button>
         </form>

         {error && <p style={{ color: 'red' }}>{error}</p>}

         {results.length > 0 && (
           <div>
             <h2>Results:</h2>
             {results.map((r, index) => (
               <div key={r.id} style={{ marginBottom: '16px' }}>
                 <h3>{r.name} <small>({r.category})</small></h3>
                 <p>{r.description}</p>
                 <p>Recommendations: {r.recommendations}</p>
                 <button onClick={() => handleRecommend(r.id, index)}>Recommend</button>
                 <hr />
               </div>
             ))}
           </div>
         )}
       </div>
     );
   }
   ```

3. **Save**:
   * Save as `"SearchView.jsx"` (All Files).
   * Rename if needed:
     ```bash
     ren SearchView.jsx.txt SearchView.jsx
     ```

### C.4: Create `src/AddResourceView.jsx`

1. **Create**:
   ```bash
   notepad AddResourceView.jsx  # Windows
   nano AddResourceView.jsx    # macOS
   ```

2. **Add**:
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
           })
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

3. **Save**:
   * Save as `"AddResourceView.jsx"` (All Files).
   * Rename if needed:
     ```bash
     ren AddResourceView.jsx.txt AddResourceView.jsx
     ```

4. **Verify Files**:
   * Check:
     ```bash
     dir src
     ```
     Expect `App.jsx`, `SearchView.jsx`, `AddResourceView.jsx`, `main.jsx`.
   * If any have `.txt`, rename them.

## Step 5: Exercise D – Run and Test the Application

### D.1: Start the Express Server

1. **Open a terminal**:
   ```bash
   cd C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartB
   node server.js
   ```
   * Expect:
     ```
     [BOOT] Using DB at: .../discoverhealth.db (FOUND)
     [BOOT] Tables in DB: [ 'healthcare_resources', 'users', 'reviews' ]
     Server running at http://localhost:3000
     ```

### D.2: Start the React Dev Server

1. **Open a new terminal**:
   * Navigate:
     ```bash
     cd C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartB\frontend
     ```
   * If dependencies are installed, run:
     ```bash
     npm run dev
     ```
     * Expect:
       ```
       VITE v7.1.3  ready in ...ms
       ➜  Local:   http://localhost:5173/
       ```
   * **If errors occur** or dependencies are missing:
     ```bash
     npm install
     npm install react-router-dom
     npm run dev
     ```
     * Dependencies only need installation once unless `node_modules` is deleted.

2. **Open**:
   * Visit `http://localhost:5173`. Expect “DiscoverHealth” title, navigation links (“Search Resources”, “Add Resource”), and search form.
   * If you see a Vite + React page with logos and a counter, recheck `App.jsx`, `SearchView.jsx`, and `AddResourceView.jsx`.

### D.3: Test the Interface

1. **Search Page (`/`)**:
   * Enter “London” and click Search. Expect resources (e.g., “St Thomas Hospital”).
   * Click “Recommend” to increment counts.
   * Test other regions (e.g., “Manchester”) or invalid inputs.

2. **Add Resource Page (`/add`)**:
   * Click “Add Resource”, fill in fields (e.g., Name: “New Clinic”, Region: “Liverpool”), submit.
   * Expect green “Resource added with ID X” message.
   * Verify new resource in Search page.

3. **Optional: Test with Postman/RESTer**:
   * **GET**: `http://localhost:3000/api/resources?region=London`
   * **POST Resource**:
     * URL: `http://localhost:3000/api/resources`
     * Body:
       ```json
       {
         "name": "Test Clinic",
         "category": "Clinic",
         "country": "UK",
         "region": "Bristol",
         "lat": 51.45,
         "lon": -2.58,
         "description": "Test clinic in Bristol"
       }
       ```
   * **POST Recommend**: `http://localhost:3000/api/resources/5/recommend`

### D.4: Troubleshooting

* **Default Vite + React Page**:
  * Verify `App.jsx`, `SearchView.jsx`, `AddResourceView.jsx` contents and names (case-sensitive).
  * Rename if needed:
    ```bash
    ren src\App.jsx.txt App.jsx
    ren src\SearchView.jsx.txt SearchView.jsx
    ren src\AddResourceView.jsx.txt AddResourceView.jsx
    ```
* **“Failed to resolve react-router-dom”**:
  * Run:
    ```bash
    cd frontend
    npm install react-router-dom
    ```
  * Or reinstall:
    ```bash
    del node_modules /s /q
    del package-lock.json
    npm install
    npm install react-router-dom
    ```
* **CORS Errors**: Verify `server.js` CORS middleware. Restart server.
* **API Errors**: Ensure `discoverhealth.db` exists and Express server is running.

## Step 6: (Optional) Save to GitHub

1. **Initialize Git**:
   ```bash
   git init
   git add .
   git commit -m "DiscoverHealth Part B: React Front-End"
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
   * Create a GitHub repository `DiscoverHealthPartB`.
   * Run:
     ```bash
     git remote add origin https://github.com/<your-username>/DiscoverHealthPartB.git
     git branch -M main
     git push -u origin main
     ```

## Step 7: Summary and Next Steps

You’ve built a React front-end for your Part A API! Recap:
* **Set up**: Copied Part A files, installed `express`, `better-sqlite3`, `cors`, `react-router-dom`.
* **Modified API**: Added CORS.
* **Built React app**: Created `App.jsx`, `SearchView.jsx`, `AddResourceView.jsx`.
* **Tested**: Verified in browser and optionally Postman/RESTer.

**Next Steps**:
* **Enhance UI**: Add CSS or Tailwind.
* **Add Validation**: Check latitude/longitude ranges.
* **Explore Part C**: Use `users` and `reviews` for authentication/reviews.
* **Production**: Run `npm run build` in `frontend`.
