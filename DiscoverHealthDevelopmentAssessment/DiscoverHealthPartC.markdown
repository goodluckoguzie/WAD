# Week 6: DiscoverHealth Part C – Adding Error-Checking and Validation

## Introduction

In **DiscoverHealth Part C**, you’ll enhance the application from Part B by implementing error-checking for Task 7 of the assignment. This involves validating input data in the backend API for adding healthcare resources (Task 2 from Part A) and displaying server-side errors in the frontend add-resource form (Task 5 from Part B). You’ll use **Windows Command Prompt** or **macOS Terminal** to set up the project, building on Part B to achieve a 48% grade threshold.

### What You’ll Learn

* **Backend Validation**: Validate POST data for required fields, returning HTTP 400 errors if invalid.
* **Frontend Error Handling**: Display server error messages in the React UI using state.
* **HTTP Status Codes**: Use 400 Bad Request for validation failures.
* **User Feedback**: Provide clear error messages to improve usability.

### What You’ll Do

1. **Exercise A**: Set up a new project folder `DiscoverHealthPartC`, copy files from Part B (`discoverhealth.db`, `server.js`, `frontend` folder), and install dependencies.
2. **Exercise B**: Update `server.js` to validate POST `/api/resources` data, checking for missing fields.
3. **Exercise C**: Verify `AddResourceView.jsx` handles server errors correctly.
4. **Exercise D**: Run both servers and test validation in the browser and (optionally) Postman/RESTer.
5. **Save Everything**: Organize work and (optionally) push to GitHub.

### Prerequisites

* Completed **DiscoverHealth Part B** with a working React frontend, Express server, and `discoverhealth.db`.
* **Node.js** installed (check: `node --version`, `npm --version`).
* Familiarity with command-line commands (`cd`, `mkdir`, `copy`/`cp`).
* Part B project folder available for copying.
* Install Node.js from [nodejs.org](https://nodejs.org) if needed.

## Step 1: Understanding the Basics

### What is Part C?

**DiscoverHealth Part C** adds error-checking to meet Task 7:
* **Backend**: Validate POST data for adding resources (name, category, country, region, lat, lon, description) and return 400 errors if fields are missing or empty.
* **Frontend**: Display these server errors in the add-resource form for user feedback.

### How Part C Builds on Part B

* **Part B**:
  * POST `/api/resources`: Accepts resource data without validation.
  * `AddResourceView.jsx`: Has client-side validation and basic server error handling.
* **Part C**:
  * Backend: Add server-side validation in `server.js`.
  * Frontend: Leverage existing error handling in `AddResourceView.jsx` to show server errors.

### Key Concepts

* **Validation**: Check `req.body` fields in Express using `if` conditions.
* **HTTP Errors**: Return `res.status(400).json({ error: '...' })` for invalid data.
* **Frontend Error Display**: Use React `useState` to show `data.error` from server responses.

## Step 2: Exercise A – Set Up the Project

Create a new folder and copy Part B files.

1. **Open Command Prompt (Windows) or Terminal (macOS)**:
   * **Windows**: `Win + R`, `cmd`, Enter.
   * **macOS**: Open Terminal via Spotlight.

2. **Create Project Folder**:
   ```bash
   cd %userprofile%\Desktop  # Windows
   cd ~/Desktop             # macOS
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

Update `server.js` to validate POST data.

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

5. **Changes**:
   * Added validation in POST `/api/resources` to check for missing/empty fields.
   * Returns 400 with specific error message.

## Step 4: Exercise C – Verify Frontend Error Handling

Confirm `AddResourceView.jsx` handles server errors.

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
   * Save as `"AddResourceView.jsx"` (All Files).
   * Verify:
     ```bash
     dir
     ren AddResourceView.jsx.txt AddResourceView.jsx
     ```

5. **Note**: No changes needed; the script already handles server errors via `data.error`.

## Step 5: Exercise D – Run and Test the Application

1. **Start Express Server**:
   ```bash
   cd C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartC
   node server.js
   ```

2. **Start React Dev Server** (new terminal):
   * If dependencies are installed:
     ```bash
     cd C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartC\frontend
     npm run dev
     ```
   * If errors occur:
     ```bash
     npm install
     npm install react-router-dom
     npm run dev
     ```

3. **Test Validation**:
   * Visit `http://localhost:5173/add`.
   * Submit form with missing fields (e.g., no name).
   * Expect red error: “All fields are required: name, category, country, region, lat, lon, description”.
   * Test with Postman:
     * POST `http://localhost:3000/api/resources`
     * Body: `{ "name": "Test", "category": "Clinic" }` (missing fields)
     * Expect: 400, `{ "error": "All fields are required..." }`

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

You’ve enhanced DiscoverHealth with validation! Recap:
* Copied Part B files.
* Updated `server.js` for POST validation.
* Verified `AddResourceView.jsx` for error display.
* Tested error messages.

**Next Steps**:
* Proceed to Part D (Leaflet map integration).
* Add advanced validation (e.g., numeric checks for lat/lon).
* Use browser console (F12) for debugging.
