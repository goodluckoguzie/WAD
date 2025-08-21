# DiscoverHealth Part B – Building a React Front-End for the Healthcare API

## Introduction

In **DiscoverHealth Part B**, you’ll create a web front-end using **React** and **Vite** to interact with the REST API you built in Part A. This front-end will allow users to search for healthcare resources by region, add new resources, and "like" (recommend) resources, all via asynchronous (AJAX) requests to the API. You’ll use the **Windows Command Prompt** or **macOS Terminal** to set up the project, and by the end, you’ll have a dynamic web interface that communicates with your Part A server. This satisfies the assignment’s requirement for a “more advanced solution” by leveraging React’s component-based architecture.

### What You’ll Learn

* **React Components**: Build reusable UI pieces (e.g., a search form, a resource list) to create a dynamic interface.
* **Routing with React Router**: Use client-side routing to navigate between pages (Search and Add Resource) without full page reloads.
* **AJAX with Fetch**: Make asynchronous HTTP requests to the Part A API to retrieve and send data.
* **State Management**: Use React’s `useState` hook to manage form inputs, search results, and feedback messages.
* **CORS**: Configure the backend to allow cross-origin requests from the React development server.
* **Event Handling**: Respond to user actions like form submissions and button clicks to interact with the API.
* **Dynamic UI**: Display search results and update recommendation counts in real-time using React’s declarative rendering.

### What You’ll Do

1. **Exercise A**: Set up a new project folder for Part B, copy necessary files from Part A (`discoverhealth.db` and `server.js`), and install backend dependencies including `cors` for cross-origin support.
2. **Exercise B**: Modify the Part A Express server to include CORS middleware, ensuring the React front-end can communicate with it.
3. **Exercise C**: Create a React project using Vite, and build three components:
   * `App.jsx`: The root component with navigation and routing.
   * `SearchView.jsx`: A component to search for resources by region and recommend them.
   * `AddResourceView.jsx`: A form to add new healthcare resources.
4. **Exercise D**: Run both the Express server and React dev server, then test the interface in a browser and (optionally) with Postman/RESTer.
5. **Save Everything**: Organize your work and (optionally) push it to GitHub for safekeeping.

### Prerequisites

* You completed **DiscoverHealth Part A**, so you have a working REST API with `discoverhealth.db` and `server.js`.
* You have **Node.js** installed (check with `node --version` and `npm --version`).
* You’re familiar with basic command-line commands (`cd`, `mkdir`, etc.) from previous weeks.
* You understand basic **JavaScript**, **JSON**, and **React** concepts (components, state, JSX) from earlier exercises (e.g., Week 6 React tutorial).
* You have a browser and optionally Postman or RESTer for testing API calls.
* If Node.js isn’t installed, download it from [nodejs.org](https://nodejs.org) before starting.

## Step 1: Understanding the Basics

Let’s review the key concepts and how Part B builds on Part A. This is like understanding the blueprint before constructing the interface.

### What is Part B?

**DiscoverHealth Part B** is the front-end counterpart to the Part A REST API. Instead of directly calling API endpoints with tools like Postman, users will interact through a web interface built with **React**. The interface will:

* Allow users to enter a region (e.g., “London”) and see a list of healthcare resources (name, category, description, recommendations) fetched via a GET request.
* Provide a form to add a new resource, sending data to the API via a POST request.
* Include a “Recommend” button for each resource to increment its recommendation count via another POST request.

This front-end will use **AJAX** (via JavaScript’s `fetch` API) to communicate with the Part A server running at `http://localhost:3000`. React’s component-based structure makes it ideal for organizing the UI, and **React Router** will handle navigation between the search and add-resource pages.

### How Part B Connects to Part A

* The **Part A API** provides three endpoints:
  * `GET /api/resources?region=RegionName`: Returns a JSON array of resources for the given region.
  * `POST /api/resources`: Accepts a JSON object to add a new resource.
  * `POST /api/resources/:id/recommend`: Increments the `recommendations` count for a resource by ID.
* The **React front-end** will:
  * Make **GET** requests to fetch and display resources.
  * Send **POST** requests to add resources or recommend them.
  * Update the UI dynamically based on API responses (e.g., show new resources or updated recommendation counts).
* The backend needs a **CORS** (Cross-Origin Resource Sharing) configuration because the React dev server (on `http://localhost:5173`) is on a different port than the API (`http://localhost:3000`). Without CORS, browsers will block requests due to the Same-Origin Policy.

### Key React Concepts for Part B

* **Components**: You’ll create `SearchView` (for searching and recommending) and `AddResourceView` (for adding resources), rendered within the `App` component.
* **Routing**: `react-router-dom` will let users switch between the search page (`/`) and add-resource page (`/add`) without reloading the browser.
* **State with `useState`**: Manage form inputs, search results, and success/error messages. When state changes, React updates the UI automatically.
* **AJAX with `fetch`**: Use JavaScript’s `fetch` API to make HTTP requests to the backend. These are asynchronous, so we’ll use `async/await` for cleaner code.
* **Event Handling**: Handle form submissions (`onSubmit`) and button clicks (`onClick`) to trigger API calls and update state.

### Putting It Together

* You’ll reuse the Part A database and server, with a small tweak to add CORS support.
* You’ll create a new React project with Vite, defining components to meet the assignment requirements.
* The front-end will make AJAX calls to the API, displaying results and feedback in a user-friendly way.
* Vite’s hot module reloading will make development smooth, updating the browser as you edit code.

Let’s get started by setting up the project.

## Step 2: Exercise A – Set Up the Project

We’ll create a new folder for Part B, copy necessary files from Part A, and install dependencies. This keeps Part B separate but leverages Part A’s backend.

1. **Open Command Prompt (Windows) or Terminal (macOS)**:

   * **Windows**: Press `Win + R`, type `cmd`, and press Enter.
   * **macOS**: Open Terminal from Applications > Utilities or Spotlight.
   * Verify Node.js and npm:

     ```bash
     node --version
     npm --version
     ```

     Ensure you see version numbers. If not, install Node.js from [nodejs.org](https://nodejs.org).

2. **Create a Project Folder**:

   * Navigate to your preferred location (e.g., Desktop):

     ```bash
     cd ~/Desktop  # macOS
     cd %userprofile%\Desktop  # Windows
     ```

   * Create and enter a new folder:

     ```bash
     mkdir DiscoverHealthPartB
     cd DiscoverHealthPartB
     ```

3. **Copy Files from Part A**:

   * Copy `discoverhealth.db`, `server.js`, and (optionally) `init-db.js` from your Part A folder. Assuming Part A is in a sibling folder:

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

   * Check the files are copied:

     ```bash
     dir  # Windows
     ls   # macOS
     ```

     You should see `discoverhealth.db`, `server.js`, and (if copied) `init-db.js`.

4. **Install Backend Dependencies**:

   * Install Express, better-sqlite3, and cors:

     ```bash
     npm install express better-sqlite3 cors
     ```

     * **express**: For the API server.
     * **better-sqlite3**: For database access.
     * **cors**: To allow the React front-end to call the API.
     * After installation, you’ll see `node_modules` and `package-lock.json`.

5. **Verify Setup**:

   * Optionally, run the database initialization script to ensure the database is ready (not needed if `discoverhealth.db` is already populated):

     ```bash
     node init-db.js
     ```

     This recreates the tables and sample data if something went wrong with the copy.

Your Part B folder is now set up with the backend files and dependencies.

## Step 3: Exercise B – Modify the Express Server for CORS

The React front-end will run on a different port (`http://localhost:5173`) than the API (`http://localhost:3000`). We need to add **CORS** to the Express server to allow these cross-origin requests.

### B.1: Update `server.js`

1. **Open `server.js`**:

   * **Windows**:

     ```bash
     notepad server.js
     ```

   * **macOS**:

     ```bash
     nano server.js
     ```

2. **Add CORS Support**:

   * Modify `server.js` to include the `cors` package. Add the following lines:

     * At the top, after other `require` statements:

       ```javascript
       const cors = require('cors');
       ```

     * After `const app = express();`, add:

       ```javascript
       app.use(cors({ origin: 'http://localhost:5173', credentials: true }));
       ```

   * Ensure the rest of the file (routes, database setup, etc.) remains unchanged from Part A. The full updated `server.js` should look like:

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

     /* 1. GET /api/resources?region=RegionName
        Returns all resources in a given region (task 1). */
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

     /* 2. POST /api/resources
        Adds a new resource (task 2). */
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

     /* 3. POST /api/resources/:id/recommend
        Increments the like count for a resource (task 3). */
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

   * **Save the file**:
     * **Windows**: File > Save, then close Notepad.
     * **macOS**: `Ctrl+O`, Enter, then `Ctrl+X`.

3. **Why This Matters**:

   * The `cors` middleware allows the React app (running on `http://localhost:5173`) to make requests to the API (`http://localhost:3000`).
   * Without CORS, browsers would block these requests due to security restrictions.
   * The rest of the server code remains identical to Part A, ensuring the API endpoints work as before.

## Step 4: Exercise C – Create the React Front-End

We’ll scaffold a React project using Vite and build three components to meet the assignment requirements: a root component with navigation, a search interface, and a form to add resources.

### C.1: Scaffold the React App

1. **Create the React Project**:

   * In the `DiscoverHealthPartB` folder, run:

     ```bash
     npm create vite@latest frontend -- --template react
     ```

     * This creates a `frontend` subfolder with a React template.
     * If prompted, select **React** and **JavaScript** (not TypeScript).

2. **Install Dependencies**:

   * Navigate to the `frontend` folder and install dependencies:

     ```bash
     cd frontend
     npm install
     ```

   * Install `react-router-dom` for navigation:

     ```bash
     npm install react-router-dom
     ```

3. **Project Structure**:

   * After setup, the `frontend` folder contains:
     * `index.html`: The HTML template for the React app.
     * `src/main.jsx`: The entry point that renders the `App` component.
     * `src/App.jsx`: The root component (we’ll modify this).
     * We’ll create `src/SearchView.jsx` and `src/AddResourceView.jsx` for the two main views.

### C.2: Edit `src/App.jsx`

This component sets up navigation and routing for the app.

1. **Open `src/App.jsx`**:

   * Navigate to the `frontend/src` folder:

     ```bash
     cd frontend/src
     ```

   * Open the file:
     * **Windows**:

       ```bash
       notepad App.jsx
       ```

     * **macOS**:

       ```bash
       nano App.jsx
       ```

2. **Replace with the following code**:

   * ```jsx
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
             {/* Navigation bar */}
             <nav>
               <Link to="/" style={{ marginRight: '10px' }}>Search Resources</Link>
               <Link to="/add">Add Resource</Link>
             </nav>
             <hr />
             {/* Routes determine which component renders for each URL */}
             <Routes>
               <Route path="/" element={<SearchView />} />
               <Route path="/add" element={<AddResourceView />} />
             </Routes>
           </div>
         </Router>
       );
     }
     ```

   * **Save the file**:
     * **Windows**: File > Save, then close Notepad.
     * **macOS**: `Ctrl+O`, Enter, then `Ctrl+X`.

3. **Understanding the code**:

   * Imports `BrowserRouter`, `Routes`, `Route`, and `Link` from `react-router-dom` for client-side routing.
   * Wraps the app in `<Router>` to enable navigation.
   * Creates a simple navigation bar with `<Link>` components to switch between the search page (`/`) and add-resource page (`/add`).
   * Uses `<Routes>` and `<Route>` to render `SearchView` at `/` and `AddResourceView` at `/add`.
   * Applies basic inline styles (`padding`, `fontFamily`) for a clean look.

### C.3: Create `src/SearchView.jsx`

This component handles searching for resources by region and recommending them.

1. **Create the file**:

   * In `frontend/src`:
     * **Windows**:

       ```bash
       notepad SearchView.jsx
       ```

       Click Yes to create the new file.

     * **macOS**:

       ```bash
       nano SearchView.jsx
       ```

2. **Add the following code**:

   * ```jsx
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

   * **Save the file**:
     * **Windows**: File > Save, then close Notepad.
     * **macOS**: `Ctrl+O`, Enter, then `Ctrl+X`.

3. **Understanding the code**:

   * Uses `useState` for three state variables:
     * `region`: The user’s input for the region search.
     * `results`: An array of resources fetched from the API.
     * `error`: For displaying error messages.
   * `handleSearch`:
     * Triggered on form submission.
     * Makes a GET request to `http://localhost:3000/api/resources?region=...` using `fetch`.
     * Uses `encodeURIComponent` to safely encode the region name in the URL.
     * Updates `results` with the API response or sets an `error` if the request fails.
   * `handleRecommend`:
     * Triggered when a Recommend button is clicked.
     * Sends a POST request to `http://localhost:3000/api/resources/:id/recommend`.
     * Updates the `results` state to increment the `recommendations` count for the clicked resource, keeping the UI in sync.
   * JSX:
     * Renders a form with a text input for the region and a Search button.
     * Displays errors in red if present.
     * Shows a list of results (if any), with each resource showing its name, category, description, recommendations, and a Recommend button.

### C.4: Create `src/AddResourceView.jsx`

This component provides a form to add a new healthcare resource.

1. **Create the file**:

   * In `frontend/src`:
     * **Windows**:

       ```bash
       notepad AddResourceView.jsx
       ```

       Click Yes to create the new file.

     * **macOS**:

       ```bash
       nano AddResourceView.jsx
       ```

2. **Add the following code**:

   * ```jsx
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

   * **Save the file**:
     * **Windows**: File > Save, then close Notepad.
     * **macOS**: `Ctrl+O`, Enter, then `Ctrl+X`.

3. **Understanding the code**:

   * Uses `useState` for:
     * `form`: An object holding all form fields (`name`, `category`, etc.).
     * `message`: Success message (e.g., “Resource added with ID 4”).
     * `error`: Error message if the submission fails.
   * `handleChange`:
     * Updates the `form` state when any input changes, using the input’s `name` attribute to identify the field.
   * `handleSubmit`:
     * Validates that all fields are non-empty.
     * Sends a POST request to `http://localhost:3000/api/resources` with the form data as JSON.
     * Converts `lat` and `lon` to numbers using `parseFloat`.
     * On success, shows a success message and resets the form.
     * On failure, shows an error message.
   * JSX:
     * Renders a form with inputs for all required fields (name, category, country, region, lat, lon, description).
     * Displays success (green) or error (red) messages.
     * Uses a `textarea` for the description and `type="number"` for lat/lon with `step="0.0001"` for precision.

## Step 5: Exercise D – Run and Test the Application

You’ll run both the Express server (backend) and Vite server (frontend), then test the interface in a browser and optionally with Postman/RESTer.

### D.1: Start the Express Server

1. **Open a terminal in `DiscoverHealthPartB`**:

   ```bash
   cd /path/to/DiscoverHealthPartB
   ```

2. **Run the server**:

   ```bash
   node server.js
   ```

   * You should see:

     ```
     [BOOT] Using DB at: .../DiscoverHealthPartB/discoverhealth.db (FOUND)
     [BOOT] Tables in DB: [ 'healthcare_resources', 'users', 'reviews' ]
     Server running at http://localhost:3000
     ```

   * Keep this terminal running.

### D.2: Start the React Dev Server

1. **Open a second terminal** and navigate to the `frontend` folder:

   ```bash
   cd /path/to/DiscoverHealthPartB/frontend
   ```

2. **Run the Vite dev server**:

   ```bash
   npm run dev
   ```

   * You should see:

     ```
     VITE vX.X.X  ready in ...ms
     ➜  Local:   http://localhost:5173/
     ```

   * Keep this terminal running too.

3. **Open the app**:

   * In a browser, go to `http://localhost:5173`. You should see the “DiscoverHealth” title, navigation links (“Search Resources” and “Add Resource”), and the search form.

### D.3: Test the Interface

1. **Search Page (`/`)**:

   * Click “Search Resources” (or ensure you’re at `http://localhost:5173/`).
   * Enter “London” in the region input and click Search.
   * You should see a list of resources (e.g., “St Thomas Hospital”) with name, category, description, and recommendations.
   * Click a “Recommend” button. The recommendation count should increment immediately in the UI.
   * Try other regions like “Manchester” or “Southampton”. If you enter an invalid region or leave it blank, an error message should appear.

2. **Add Resource Page (`/add`)**:

   * Click “Add Resource” in the navigation.
   * Fill in all fields, e.g.:
     * Name: “New Clinic”
     * Category: “Clinic”
     * Country: “UK”
     * Region: “Liverpool”
     * Latitude: `53.41`
     * Longitude: `-2.98`
     * Description: “A new clinic in Liverpool”
   * Click Submit. You should see a green message like “Resource added with ID 4” and the form should reset.
   * If you leave a field blank, a red error message (“All fields are required.”) appears.
   * Go back to the Search page, enter “Liverpool”, and verify the new resource appears.

3. **Optional: Test with Postman/RESTer**:

   * **GET Resources**:
     * URL: `http://localhost:3000/api/resources?region=London`
     * Method: GET
     * Expect a JSON array of resources.
   * **POST Resource**:
     * URL: `http://localhost:3000/api/resources`
     * Method: POST
     * Headers: `Content-Type: application/json`
     * Body (JSON):
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
     * Expect: `{ "id": 5 }`
   * **POST Recommend**:
     * URL: `http://localhost:3000/api/resources/5/recommend`
     * Method: POST
     * Expect: `{ "message": "Recommendation added" }`
   * These confirm the API is working, but the React interface should handle most interactions.

### D.4: Troubleshooting

* **CORS Errors**: If you see “Access-Control-Allow-Origin” errors in the browser console (F12), ensure the `cors` middleware is in `server.js`.
* **API Errors**: Check the Express server terminal for logs (e.g., database errors). Ensure `discoverhealth.db` is in the `DiscoverHealthPartB` folder.
* **Blank Page or JSX Errors**: Verify file names (`SearchView.jsx`, `AddResourceView.jsx`) match imports exactly (case-sensitive). Check the Vite terminal and browser console for errors.
* **File Extensions**: If Notepad adds `.txt` (e.g., `App.jsx.txt`), rename the file or save with quotes (e.g., `"App.jsx"`) and select “All Files”.

## Step 6: (Optional) Save Everything to GitHub

To back up your work, initialize a Git repository and push to GitHub.

1. **Initialize a Git repository**:

   * In `DiscoverHealthPartB`:

     ```bash
     git init
     git add .
     git commit -m "DiscoverHealth Part B: React Front-End"
     ```

2. **Create a `.gitignore` file**:

   * ```bash
     notepad .gitignore  # Windows
     nano .gitignore     # macOS
     ```

   * Add:

     ```
     node_modules/
     frontend/dist/
     ```

   * Save and close. This ignores `node_modules` and Vite’s build output.

3. **Push to GitHub**:

   * Create a new GitHub repository named `DiscoverHealthPartB`.
   * Link and push:

     ```bash
     git remote add origin https://github.com/<your-username>/DiscoverHealthPartB.git
     git branch -M main
     git push -u origin main
     ```

     Replace `<your-username>` with your GitHub username.

## Step 7: Summary and Next Steps

Congratulations on completing **DiscoverHealth Part B**! 🎉 You’ve built a dynamic React front-end that interacts with your Part A API. Let’s recap:

* **Set up the project**: Created a new folder, copied Part A files, and installed backend (`express`, `better-sqlite3`, `cors`) and frontend (`react-router-dom`) dependencies.
* **Modified the API**: Added CORS to `server.js` to allow React requests.
* **Built a React app**: Created components:
  * `App.jsx`: Navigation and routing.
  * `SearchView.jsx`: Search form and resource list with Recommend buttons.
  * `AddResourceView.jsx`: Form to add new resources.
* **Used AJAX**: Made asynchronous `fetch` requests to the API for searching, adding, and recommending resources.
* **Tested the app**: Verified functionality in the browser and optionally with Postman/RESTer.
* **Learned key concepts**: React components, state, routing, AJAX, CORS, and dynamic UI updates.

**Next Steps**:

* **Enhance the UI**: Add CSS (e.g., via a `src/App.css` file) or a library like Tailwind for better styling.
* **Add Validation**: Strengthen `AddResourceView` with more checks (e.g., valid latitude/longitude ranges).
* **Explore Part C (if applicable)**: The `users` and `reviews` tables suggest future features like user authentication or review submission.
* **Debugging**: Use the browser console (F12) and Vite/Express terminals to troubleshoot issues.
* **Production Build**: Run `npm run build` in the `frontend` folder to generate static files for deployment (not required for Part B).

You’ve now built a complete full-stack application with a Node.js/Express backend and a React front-end. This is a powerful skillset for modern web development. Great job, and keep experimenting with React and APIs!
