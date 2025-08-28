# Week 6: DiscoverHealth Part C – Adding Error-Checking and Validation

## Introduction

In **DiscoverHealth Part C**, you’ll enhance the Part B application to meet Task 7 of the assignment by adding server-side validation for the POST `/api/resources` endpoint (Task 2 from Part A) and ensuring the frontend displays these errors in the add-resource form (Task 5 from Part B). This improves data integrity and user experience by catching invalid inputs on the server and showing clear error messages in the React UI. You’ll use **Windows Command Prompt** or **macOS Terminal**, building on Part B to achieve a 48% grade threshold.

### What You’ll Learn

* **Backend Validation**: Check for missing or empty fields in POST requests, returning 400 Bad Request errors.
* **Frontend Error Handling**: Display server errors in the React UI using state.
* **HTTP Status Codes**: Use 400 for validation failures.
* **User Feedback**: Provide specific error messages to guide users.

### What You’ll Do

1. **Exercise A**: Set up `DiscoverHealthPartC` folder, copy Part B files (`discoverhealth.db`, `server.js`, `frontend`), and install dependencies.
2. **Exercise B**: Update `server.js` to validate POST `/api/resources` data.
3. **Exercise C**: Verify `AddResourceView.jsx` handles server errors.
4. **Exercise D**: Run servers and test validation in browser and (optionally) Postman/RESTer.
5. **Save Everything**: Organize work and (optionally) push to GitHub.

### Prerequisites

* Completed **DiscoverHealth Part B** with working React frontend, Express server, and `discoverhealth.db`.
* **Node.js** installed (check: `node --version`, `npm --version`).
* Familiarity with command-line (`cd`, `mkdir`, `copy`/`cp`).
* Part B project folder available at `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartB`.
* Install Node.js from [nodejs.org](https://nodejs.org) if needed.

## Step 1: Understanding the Basics

### What is Part C?

**DiscoverHealth Part C** implements Task 7:
* **Backend**: Validate POST `/api/resources` data, ensuring all fields (`name`, `category`, `country`, `region`, `lat`, `lon`, `description`) are present and non-empty, returning 400 errors if invalid.
* **Frontend**: Display server errors in the add-resource form for user feedback.

### How Part C Builds on Part B

* **Part B**:
  * POST `/api/resources`: Accepts data without server-side validation.
  * `AddResourceView.jsx`: Has client-side validation and basic server error handling.
* **Part C**:
  * Add server-side validation in `server.js`.
  * Use existing `AddResourceView.jsx` error handling to show server errors.

### Key Concepts

* **Validation**: Check `req.body` fields in Express using `if` conditions.
* **HTTP Errors**: Return `res.status(400).json({ error: '...' })`.
* **Frontend Error Display**: Use `useState` to show `data.error`.

## Step 2: Exercise A – Set Up the Project

1. **Open Command Prompt (Windows) or Terminal (macOS)**:
   * **Windows**: `Win + R`, `cmd`, Enter.
   * **macOS**: Open Terminal via Spotlight.

2. **Create Project Folder**:
   ```bash
   cd %userprofile%\Desktop\Solent University\FinalProject  # Windows
   cd ~/Desktop/Solent University/FinalProject             # macOS
   mkdir DiscoverHealthPartC
   cd DiscoverHealthPartC
   ```

3. **Copy Files from Part B**:
   * Copy `discoverhealth.db` and `server.js`:
     ```bash
     # Windows
     copy ..\DiscoverHealthPartB\discoverhealth.db .
     copy ..\DiscoverHealthPartB\server.js .

     # macOS
     cp ../DiscoverHealthPartB/discoverhealth.db .
     cp ../DiscoverHealthPartB/server.js .
     ```
   * Copy `frontend` folder:
     ```bash
     # Windows
     xcopy ..\DiscoverHealthPartB\frontend frontend /s /e /h /i

     # macOS
     cp -R ../DiscoverHealthPartB/frontend .
     ```

4. **Install Backend Dependencies**:
   ```bash
   npm install express better-sqlite3 cors
   ```
   * Creates `node_modules` and `package-lock.json`.

5. **Install Frontend Dependencies**:
   ```bash
   cd frontend
   npm install
   npm install react-router-dom
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
       "react-router-dom": "^6.26.2"
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

## Step 3: Exercise B – Add Validation to the Express Server

Update `server.js` to validate POST data, adding server-side checks for missing or empty fields.

### B.1: Update `server.js`

1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartC\server.js`

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
     // Validate all fields are present and non-empty
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

4. **Save**:
   * **Windows**: Save as `"server.js"` (All Files).
   * **macOS**: `Ctrl+O`, Enter, `Ctrl+X`.
   * Verify:
     ```bash
     dir
     ren server.js.txt server.js
     ```

5. **Code Explanation**:
   * **Imports and Setup**:
     - `const express = require('express');`: Imports Express for the web server.
     - `const Database = require('better-sqlite3');`: Imports better-sqlite3 for database operations.
     - `const path = require('path');`, `const fs = require('fs');`: Locate the database file.
     - `const cors = require('cors');`: Enables CORS for frontend communication.
     - `app.use(cors({ origin: 'http://localhost:5173', credentials: true }));`: Allows requests from React dev server on port 5173.
     - `const dbPath = path.join(__dirname, 'discoverhealth.db');`: Constructs path to `discoverhealth.db`.
     - `const db = new Database(dbPath);`: Opens SQLite database.
     - `console.log('[BOOT] ...');`: Logs database file status.
     - `try { ... } catch (e) { ... }`: Lists database tables for debugging.
     - `app.use(express.json());`: Parses JSON requests.
   * **GET `/` Route**: Returns a friendly message for the root URL.
   * **GET `/api/resources` Route**:
     - Extracts `region` query parameter.
     - Returns 400 if missing, else queries resources and returns JSON.
   * **POST `/api/resources` Route (Updated)**:
     - Destructures `req.body` for fields.
     - **New Addition**: Validates fields with `if (!name?.trim() || ...)` to check for missing (`undefined`) or empty (`""` or `" "`) fields, using optional chaining (`?.`) for safety.
     - Returns 400 with error message if validation fails.
     - Inserts validated data, trimming strings to clean inputs.
     - Returns 201 with new resource ID or 500 on database error.
   * **Function of Addition**: Ensures data integrity by preventing invalid insertions and provides clear error messages (e.g., “All fields are required...”) for the frontend.
   * **POST `/api/resources/:id/recommend` Route**: Increments recommendations, handling errors.
   * **Server Start**: Listens on port 3000.

## Step 4: Exercise C – Verify Frontend Error Handling

Confirm `AddResourceView.jsx` handles server errors correctly.

### C.1: Verify `AddResourceView.jsx`

1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartC\frontend\src\AddResourceView.jsx`

2. **Open**:
   ```bash
   cd frontend/src
   notepad AddResourceView.jsx  # Windows
   nano AddResourceView.jsx    # macOS
   ```

3. **Ensure Content**:
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

4. **Save**:
   * **Windows**: Save as `"AddResourceView.jsx"` (All Files).
   * **macOS**: `Ctrl+O`, Enter, `Ctrl+X`.
   * Verify:
     ```bash
     dir
     ren AddResourceView.jsx.txt AddResourceView.jsx
     ```

5. **Code Explanation**:
   * **Imports**: `useState` for managing form state, success, and error messages.
   * **State**:
     - `form`: Stores input fields.
     - `message`: Shows success (e.g., “Resource added...”).
     - `error`: Shows errors from client or server.
   * **Event Handlers**:
     - `handleChange`: Updates form state on input changes.
     - `handleSubmit`:
       - Prevents page reload.
       - Clears previous messages.
       - Client-side validation checks for empty fields.
       - Sends POST request with JSON data, converting `lat`/`lon` to numbers.
       - On success (201), shows success message and clears form.
       - On failure, sets `error` to `data.error` (e.g., “All fields are required...”) or fallback.
       - Catches network errors.
   * **JSX**: Renders form with inputs, displays errors in red, success in green.
   * **Function**: Displays server errors from `server.js`, enhancing user feedback.

## Step 5: Exercise D – Run and Test the Application

1. **Start Express Server**:
   ```bash
   cd C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartC
   node server.js
   ```
   * Expect:
     ```
     [BOOT] Using DB at: .../discoverhealth.db (FOUND)
     [BOOT] Tables in DB: [ 'healthcare_resources', 'users', 'reviews' ]
     Server running at http://localhost:3000
     ```

2. **Start React Dev Server** (new terminal):
   * If dependencies installed:
     ```bash
     cd C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartC\frontend
     npm run dev
     ```
     * Expect:
       ```
       VITE v7.1.3  ready in ...ms
       ➜  Local:   http://localhost:5173/
       ```
   * If errors:
     ```bash
     npm install
     npm install react-router-dom
     npm run dev
     ```

3. **Test Validation**:
   * Visit `http://localhost:5173/add`.
   * Submit form with missing fields (e.g., no name or lat).
   * Expect red error: “All fields are required: name, category, country, region, lat, lon, description”.
   * Submit valid data (e.g., Name: “Test Clinic”, Category: “Clinic”, Country: “UK”, Region: “London”, Lat: 51.5, Lon: -0.1, Description: “Test”) to confirm success message.
   * Test with Postman:
     * POST `http://localhost:3000/api/resources`
     * Body: `{ "name": "Test", "category": "Clinic" }`
     * Expect: 400, `{ "error": "All fields are required..." }`
     * Body: `{ "name": "Test Clinic", "category": "Clinic", "country": "UK", "region": "London", "lat": 51.5, "lon": -0.1, "description": "Test" }`
     * Expect: 201, `{ "id": X }`

4. **Troubleshooting**:
   * **No Errors Displayed**: Check browser console (F12) and server logs.
   * **Default Vite Page**: Verify `App.jsx`, `SearchView.jsx`, `AddResourceView.jsx` contents and names (not `.txt`).
   * **CORS Errors**: Confirm `cors` middleware in `server.js`.
   * **Dependency Issues**:
     ```bash
     cd frontend
     del node_modules /s /q
     del package-lock.json
     npm install
     npm install react-router-dom
     ```

## Step 6: (Optional) Save to GitHub

1. **Initialize Git**:
   ```bash
   git init
   git add .
   git commit -m "DiscoverHealth Part C: Error-Checking and Validation"
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
   git remote add origin https://github.com/<your-username>/DiscoverHealthPartC.git
   git branch -M main
   git push -u origin main
   ```

## Step 7: Summary and Next Steps

You’ve enhanced DiscoverHealth with server-side validation, meeting Task 7! Recap:
* Copied Part B files.
* Added validation in `server.js` for POST `/api/resources`, ensuring all fields are present and non-empty.
* Verified `AddResourceView.jsx` displays server errors like “All fields are required...”.
* Tested error handling with invalid and valid inputs.

**Next Steps**:
* Proceed to Part D (Tasks 8-9: Leaflet map integration).
* Enhance validation (e.g., check `lat`/`lon` are valid numbers using `isNaN`).
* Debug using browser console (F12) and server logs.
