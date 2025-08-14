# DiscoverHealth Development Plan (Parts A–F)

## Overview
We will build the DiscoverHealth web application in six stages (Part A through Part F). Each part is contained in its own folder (PartA, PartB, ..., PartF) and builds on the previous part. We use Node.js with Express and a SQLite database (the provided discoverhealth.db file) for the backend. The front-end will start with simple HTML/JS (AJAX) and later offer a React alternative for extra functionality. At each stage, we will set up the project folder, write the necessary code (server-side and client-side), and test the functionality. Commands are provided for both Windows Command Prompt and macOS Terminal. The final application will run on http://localhost:3000 (as required). Let’s proceed step-by-step.

## Part A – Develop a Simple REST API

### Goal
Create a basic Express server that serves as a RESTful API for healthcare resources. It should connect to the SQLite database and provide endpoints to (1) list resources by region, (2) add a new resource, and (3) “recommend” (like) a resource. This part has no web front-end yet; we will test using a browser or API client.

### Step 1: Set Up the Project Folder and Dependencies
Create a new folder for Part A and navigate into it.

**Windows Command Prompt**:
1. Open Command Prompt (Windows Key + R, type `cmd`, press Enter).
2. Navigate to Desktop:
   ```
   cd %userprofile%\Desktop
   ```
3. Create and enter the project folder:
   ```
   mkdir "Solent University"
   cd "Solent University"
   mkdir FinalProject
   cd FinalProject
   mkdir PartA
   cd PartA
   ```

**macOS Terminal**:
1. Open Terminal (Applications > Utilities > Terminal).
2. Navigate to Desktop:
   ```
   cd ~/Desktop
   ```
3. Create and enter the project folder:
   ```
   mkdir -p "Solent University/FinalProject/PartA"
   cd "Solent University/FinalProject/PartA"
   ```

Initialize a Node.js project:
```
npm init -y
```
This creates a `package.json` file with default settings.

Install required packages:
```
npm install express better-sqlite3
```
This installs Express (web framework) and better-sqlite3 (SQLite driver) locally.

**Database Setup**: Place the provided `discoverhealth.db` SQLite database file in the PartA folder (same directory as `server.js`). This database should contain the `healthcare_resources` table (with fields: id, name, category, country, region, lat, lon, description, recommendations), and potentially `users` and `reviews` tables for later parts. If the provided database is unavailable or empty, you can initialize it with the script in Step 2.

**Verify**:
- Run `dir` (Windows) or `ls` (macOS) to confirm `package.json` and `node_modules` exist.
- Ensure `discoverhealth.db` is in the PartA folder.

### Step 2: Create the Database Initialization Script (init-db.js)
Create a script to populate the database if the provided `discoverhealth.db` is missing or empty.

**Windows Command Prompt**:
1. Create `init-db.js`:
   ```
   echo. > init-db.js
   notepad init-db.js
   ```
2. Copy and paste the script below into Notepad, then save and close.

**macOS Terminal**:
1. Create `init-db.js`:
   ```
   touch init-db.js
   open -e init-db.js
   ```
2. Copy and paste the script below into TextEdit, then save and close.

**Script Content**:
```javascript
const path = require('path');
const Database = require('better-sqlite3');

const dbPath = path.join(__dirname, 'discoverhealth.db');
const db = new Database(dbPath);

// Create tables and insert initial data
db.exec(`
  PRAGMA foreign_keys = ON;

  CREATE TABLE IF NOT EXISTS healthcare_resources (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    category TEXT NOT NULL,
    country TEXT NOT NULL,
    region TEXT NOT NULL,
    lat REAL NOT NULL,
    lon REAL NOT NULL,
    description TEXT NOT NULL,
    recommendations INTEGER NOT NULL DEFAULT 0
  );

  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL,
    isAdmin INTEGER NOT NULL DEFAULT 0
  );

  CREATE TABLE IF NOT EXISTS reviews (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    resource_id INTEGER NOT NULL,
    review TEXT NOT NULL,
    user_id INTEGER NOT NULL,
    FOREIGN KEY(resource_id) REFERENCES healthcare_resources(id) ON DELETE CASCADE,
    FOREIGN KEY(user_id) REFERENCES users(id) ON DELETE CASCADE
  );

  -- Insert sample data
  INSERT OR IGNORE INTO healthcare_resources (name, category, country, region, lat, lon, description, recommendations)
  VALUES 
    ('St Thomas Hospital', 'Hospital', 'UK', 'London', 51.4980, -0.1195, 'Major teaching hospital on the Thames.', 5),
    ('Manchester Clinic', 'Clinic', 'UK', 'Manchester', 53.4808, -2.2426, 'Community clinic in central Manchester.', 2),
    ('Southampton Health Centre', 'Health Centre', 'UK', 'Southampton', 50.9097, -1.4044, 'General health services for families.', 0);

  INSERT OR IGNORE INTO users (username, password, isAdmin)
  VALUES ('test', 'test', 0);
`);

console.log('Database initialized at', dbPath);
```

Run the script to create and populate the database:
```
node init-db.js
```

**Verify**:
- Run `dir` (Windows) or `ls` (macOS) to confirm `discoverhealth.db` exists.
- The file should be larger than 8KB, indicating it contains data.

### Step 3: Create the Express Server (server.js)
Create the server file to handle API requests.

**Windows Command Prompt**:
1. Create `server.js`:
   ```
   echo. > server.js
   notepad server.js
   ```
2. Copy and paste the script below into Notepad, then save and close.

**macOS Terminal**:
1. Create `server.js`:
   ```
   touch server.js
   open -e server.js
   ```
2. Copy and paste the script below into TextEdit, then save and close.

**Script Content**:
```javascript
const express = require('express');
const Database = require('better-sqlite3');
const path = require('path');

const app = express();
const dbPath = path.join(__dirname, 'discoverhealth.db');
const db = new Database(dbPath);  // Connect to the SQLite database file

// Log available tables for debugging
console.log('Tables in DB:', db.prepare("SELECT name FROM sqlite_master WHERE type='table'").all().map(t => t.name));

app.use(express.json());  // Parse JSON request bodies

// Temporary root route for testing
app.get('/', (req, res) => {
  res.send('DiscoverHealth API is running. Try /api/resources?region=London');
});

// 1. GET all healthcare resources in a given region (returns JSON)
app.get('/api/resources', (req, res) => {
  try {
    const region = req.query.region;
    if (!region) {
      return res.status(400).json({ error: 'Region query parameter is required' });
    }
    const stmt = db.prepare('SELECT * FROM healthcare_resources WHERE region = ?');
    const resources = stmt.all(region);
    res.json(resources);
  } catch (err) {
    console.error('GET /api/resources failed:', err);
    res.status(500).json({ error: 'Failed to retrieve resources', detail: err.message });
  }
});

// 2. POST a new healthcare resource (adds to database)
app.post('/api/resources', (req, res) => {
  try {
    const { name, category, country, region, lat, lon, description } = req.body;
    const stmt = db.prepare(`
      INSERT INTO healthcare_resources (name, category, country, region, lat, lon, description, recommendations)
      VALUES (?, ?, ?, ?, ?, ?, ?, 0)
    `);
    const info = stmt.run(name, category, country, region, lat, lon, description);
    res.status(201).json({ id: info.lastInsertRowid });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Failed to add resource' });
  }
});

// 3. POST a "recommendation" (like) for a resource by ID
app.post('/api/resources/:id/recommend', (req, res) => {
  try {
    const resourceId = req.params.id;
    const update = db.prepare('UPDATE healthcare_resources SET recommendations = recommendations + 1 WHERE id = ?');
    const result = update.run(resourceId);
    if (result.changes === 0) {
      return res.status(404).json({ error: 'Resource not found' });
    }
    res.json({ message: 'Recommendation added' });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Failed to recommend resource' });
  }
});

// Start the server on port 3000
app.listen(3000, () => {
  console.log('Part A server running at http://localhost:3000');
});
```

### Step 4: Run and Test Part A
Start the server:
```
node server.js
```

**Windows Command Prompt**:
```
cd "C:\Users\Nneka\Desktop\Solent University\FinalProject\PartA"
node server.js
```

**macOS Terminal**:
```
cd ~/Desktop/Solent\ University/FinalProject/PartA
node server.js
```

Check the console output. You should see:
- The database path and tables (including `healthcare_resources`).
- “Part A server running at http://localhost:3000”.

If you see an empty table list or errors, ensure `discoverhealth.db` is correctly placed and contains the `healthcare_resources` table. If needed, re-run `node init-db.js`.

**Test GET endpoint**: In a browser, visit:
```
http://localhost:3000/api/resources?region=London
```
You should see a JSON array of resources (e.g., St Thomas Hospital if using the sample data). Try other regions like “Manchester” or “Southampton”.

**Test POST (add resource)**: Using a REST client or curl, send a POST to `http://localhost:3000/api/resources` with:
```json
{
  "name": "Test Clinic",
  "category": "Clinic",
  "country": "UK",
  "region": "London",
  "lat": 51.5,
  "lon": -0.1,
  "description": "A test clinic added via API"
}
```

**Windows Command Prompt**:
```
curl -X POST "http://localhost:3000/api/resources" -H "Content-Type: application/json" -d "{\"name\":\"Test Clinic\",\"category\":\"Clinic\",\"country\":\"UK\",\"region\":\"London\",\"lat\":51.5,\"lon\":-0.1,\"description\":\"Added via API\"}"
```

**macOS Terminal**:
```
curl -X POST "http://localhost:3000/api/resources" -H "Content-Type: application/json" -d '{"name":"Test Clinic","category":"Clinic","country":"UK","region":"London","lat":51.5,"lon":-0.1,"description":"Added via API"}'
```
Response should be `{ "id": X }`. Verify by checking `GET /api/resources?region=London`.

**Test POST (recommend)**: Use a resource ID from the GET response (e.g., 1). Send a POST to:
```
http://localhost:3000/api/resources/1/recommend
```

**Windows Command Prompt**:
```
curl -X POST "http://localhost:3000/api/resources/1/recommend"
```

**macOS Terminal**:
```
curl -X POST "http://localhost:3000/api/resources/1/recommend"
```
Response should be `{"message":"Recommendation added"}`. Check the resource via GET to confirm the recommendations count increased.

**Troubleshooting**:
- If you get `{"error":"Failed to retrieve resources","detail":"no such table: healthcare_resources"}`, delete `discoverhealth.db`, re-run `node init-db.js`, and restart the server.
- Ensure you’re in the PartA folder when running commands.

Part A provides a working REST API in the PartA folder. ✅

## Part B – Develop a Simple AJAX Front-End

### Goal
Create a basic front-end using HTML and JavaScript to interact with the Part A API via AJAX. Include an index page for searching resources by region, an add-resource page, and a recommend button for search results.

**Folder Setup**:
**Windows Command Prompt**:
```
cd "C:\Users\Nneka\Desktop\Solent University\FinalProject"
xcopy PartA PartB /E /I
cd PartB
```

**macOS Terminal**:
```
cd ~/Desktop/Solent\ University/FinalProject
cp -r PartA PartB
cd PartB
```

### Step 5: Update the Express Server for Static Files
Open `server.js` to add static file serving.

**Windows Command Prompt**:
```
notepad server.js
```

**macOS Terminal**:
```
open -e server.js
```

Add after `app.use(express.json())`:
```javascript
app.use(express.static('public'));
```
Save and close.

Create a `public` folder:
**Windows Command Prompt**:
```
mkdir public
```

**macOS Terminal**:
```
mkdir public
```

### Step 6: Create the Front-End Pages (Non-React)
#### 1. Create public/index.html – Search Page
**Windows Command Prompt**:
```
cd public
echo. > index.html
notepad index.html
```

**macOS Terminal**:
```
cd public
touch index.html
open -e index.html
```

**Content**:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>DiscoverHealth – Find Resources</title>
</head>
<body>
  <h1>DiscoverHealth</h1>

  <!-- Search form -->
  <form id="searchForm">
    <label for="regionInput">Search by Region:</label>
    <input type="text" id="regionInput" name="region" placeholder="e.g., London" required />
    <button type="submit">Search</button>
  </form>

  <!-- Results section -->
  <h2>Results:</h2>
  <div id="results"></div>

  <!-- Link to Add Resource page -->
  <p><a href="add.html">➕ Add a New Resource</a></p>

  <script src="index.js"></script>
</body>
</html>
```

#### 2. Create public/index.js – Client-side JS for Index
**Windows Command Prompt**:
```
echo. > index.js
notepad index.js
```

**macOS Terminal**:
```
touch index.js
open -e index.js
```

**Content**:
```javascript
document.addEventListener('DOMContentLoaded', () => {
  const form = document.getElementById('searchForm');
  const resultsDiv = document.getElementById('results');

  form.addEventListener('submit', async (e) => {
    e.preventDefault();
    resultsDiv.innerHTML = '';
    const region = document.getElementById('regionInput').value.trim();
    if (!region) return;

    try {
      const res = await fetch(`/api/resources?region=${encodeURIComponent(region)}`);
      if (!res.ok) {
        const errorData = await res.json();
        resultsDiv.innerHTML = `<p style="color:red;">Error: ${errorData.error || res.status}</p>`;
        return;
      }
      const resources = await res.json();

      if (resources.length === 0) {
        resultsDiv.innerHTML = `<p>No resources found in "${region}".</p>`;
        return;
      }

      resources.forEach(resource => {
        const itemDiv = document.createElement('div');
        itemDiv.className = 'resource-item';
        itemDiv.innerHTML = `
          <h3>${resource.name} <small>(${resource.category})</small></h3>
          <p>${resource.description}</p>
          <p><b>Recommendations:</b> <span id="rec-${resource.id}">${resource.recommendations}</span></p>
          <button class="rec-button" data-id="${resource.id}">Recommend</button>
          <hr/>
        `;
        resultsDiv.appendChild(itemDiv);
      });

      document.querySelectorAll('.rec-button').forEach(button => {
        button.addEventListener('click', async () => {
          const id = button.getAttribute('data-id');
          try {
            const resp = await fetch(`/api/resources/${id}/recommend`, { method: 'POST' });
            if (resp.ok) {
              const countSpan = document.getElementById(`rec-${id}`);
              countSpan.textContent = parseInt(countSpan.textContent) + 1;
            } else {
              alert('Failed to recommend this resource.');
            }
          } catch (err) {
            console.error(err);
            alert('Error recommending the resource.');
          }
        });
      });
    } catch (err) {
      console.error(err);
      resultsDiv.innerHTML = `<p style="color:red;">An error occurred while searching.</p>`;
    }
  });
});
```

#### 3. Create public/add.html – Add Resource Page
**Windows Command Prompt**:
```
echo. > add.html
notepad add.html
```

**macOS Terminal**:
```
touch add.html
open -e add.html
```

**Content**:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>DiscoverHealth – Add Resource</title>
</head>
<body>
  <h1>Add a New Healthcare Resource</h1>
  <form id="addForm">
    <p>
      <label>Name: <input type="text" id="name" name="name" required></label>
    </p>
    <p>
      <label>Category: <input type="text" id="category" name="category" required></label>
    </p>
    <p>
      <label>Country: <input type="text" id="country" name="country" required></label>
    </p>
    <p>
      <label>Region: <input type="text" id="region" name="region" required></label>
    </p>
    <p>
      <label>Latitude: <input type="number" id="lat" name="lat" step="0.0001" required></label>
    </p>
    <p>
      <label>Longitude: <input type="number" id="lon" name="lon" step="0.0001" required></label>
    </p>
    <p>
      <label>Description:<br>
      <textarea id="description" name="description" rows="3" cols="40" required></textarea>
      </label>
    </p>
    <button type="submit">Submit</button>
    <p id="errorMsg" style="color:red;"></p>
  </form>

  <p><a href="index.html">← Back to Search</a></p>

  <script src="add.js"></script>
</body>
</html>
```

#### 4. Create public/add.js – Client-side JS for Add Resource
**Windows Command Prompt**:
```
echo. > add.js
notepad add.js
```

**macOS Terminal**:
```
touch add.js
open -e add.js
```

**Content**:
```javascript
document.addEventListener('DOMContentLoaded', () => {
  const form = document.getElementById('addForm');
  const errorMsg = document.getElementById('errorMsg');

  form.addEventListener('submit', async (e) => {
    e.preventDefault();
    errorMsg.textContent = '';

    const newResource = {
      name: document.getElementById('name').value.trim(),
      category: document.getElementById('category').value.trim(),
      country: document.getElementById('country').value.trim(),
      region: document.getElementById('region').value.trim(),
      lat: parseFloat(document.getElementById('lat').value),
      lon: parseFloat(document.getElementById('lon').value),
      description: document.getElementById('description').value.trim()
    };

    try {
      const res = await fetch('/api/resources', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newResource)
      });
      if (!res.ok) {
        const data = await res.json();
        errorMsg.textContent = data.error || `Error ${res.status}: Failed to add resource.`;
      } else {
        alert('Resource added successfully!');
        form.reset();
      }
    } catch (err) {
      console.error(err);
      errorMsg.textContent = 'An unexpected error occurred.';
    }
  });
});
```

### Step 7: Test Part B
Start the server:
**Windows Command Prompt**:
```
cd "C:\Users\Nneka\Desktop\Solent University\FinalProject\PartB"
node server.js
```

**macOS Terminal**:
```
cd ~/Desktop/Solent\ University/FinalProject/PartB
node server.js
```

Open `http://localhost:3000/`. Test searching, recommending, and adding resources through the UI. Verify no page reloads occur (AJAX). Part B fulfills tasks 4, 5, and 6. ✅

## Part C – Adding Simple Error-Checking

### Goal
Add validation to the add resource API and display errors in the front-end.

**Folder Setup**:
**Windows Command Prompt**:
```
cd "C:\Users\Nneka\Desktop\Solent University\FinalProject"
xcopy PartB PartC /E /I
cd PartC
```

**macOS Terminal**:
```
cd ~/Desktop/Solent\ University/FinalProject
cp -r PartB PartC
cd PartC
```

### Step 8: Update the Express Server for Validation
**Windows Command Prompt**:
```
notepad server.js
```

**macOS Terminal**:
```
open -e server.js
```

Update the `POST /api/resources` route:
```javascript
app.post('/api/resources', (req, res) => {
  try {
    const { name, category, country, region, lat, lon, description } = req.body;
    if (!name || !category || !country || !region || lat == null || lon == null || !description) {
      return res.status(400).json({ error: 'All fields (name, category, country, region, lat, lon, description) are required.' });
    }
    const stmt = db.prepare(`
      INSERT INTO healthcare_resources (name, category, country, region, lat, lon, description, recommendations)
      VALUES (?, ?, ?, ?, ?, ?, ?, 0)
    `);
    const info = stmt.run(name, category, country, region, lat, lon, description);
    res.status(201).json({ id: info.lastInsertRowid });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Failed to add resource' });
  }
});
```
Save and close.

### Step 9: Test Part C
The existing `add.js` handles error responses. Test by submitting an incomplete form to ensure the error message appears. Part C completes task 7. ✅

## Part D – Integrating a Map (Leaflet & OpenStreetMap)

### Goal
Add an interactive map to display search results and allow location selection for adding resources using Leaflet and OpenStreetMap.

**Folder Setup**:
**Windows Command Prompt**:
```
cd "C:\Users\Nneka\Desktop\Solent University\FinalProject"
xcopy PartC PartD /E /I
cd PartD
```

**macOS Terminal**:
```
cd ~/Desktop/Solent\ University/FinalProject
cp -r PartC PartD
cd PartD
```

**Install Leaflet**:
```
npm install leaflet
```

### Step 10: Update the Front-End for Map Integration
#### 1. Update public/index.html
**Windows Command Prompt**:
```
cd public
notepad index.html
```

**macOS Terminal**:
```
cd public
open -e index.html
```

**Content**:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>DiscoverHealth – Find Resources</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
</head>
<body>
  <h1>DiscoverHealth</h1>
  <form id="searchForm">
    <label for="regionInput">Search by Region:</label>
    <input type="text" id="regionInput" name="region" placeholder="e.g., London" required />
    <button type="submit">Search</button>
  </form>
  <h2>Results:</h2>
  <div id="map" style="height: 400px; width: 100%; margin: 10px 0;"></div>
  <div id="results"></div>
  <p><a href="add.html">➕ Add a New Resource</a></p>
  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
  <script src="index.js"></script>
</body>
</html>
```

#### 2. Update public/index.js
**Windows Command Prompt**:
```
notepad index.js
```

**macOS Terminal**:
```
open -e index.js
```

**Content**:
```javascript
document.addEventListener('DOMContentLoaded', () => {
  const map = L.map('map').setView([51.505, -0.09], 6);
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors'
  }).addTo(map);

  const form = document.getElementById('searchForm');
  const resultsDiv = document.getElementById('results');

  form.addEventListener('submit', async (e) => {
    e.preventDefault();
    resultsDiv.innerHTML = '';
    map.eachLayer(layer => {
      if (layer instanceof L.Marker) map.removeLayer(layer);
    });

    const region = document.getElementById('regionInput').value.trim();
    if (!region) return;

    try {
      const res = await fetch(`/api/resources?region=${encodeURIComponent(region)}`);
      if (!res.ok) {
        const errorData = await res.json();
        resultsDiv.innerHTML = `<p style="color:red;">Error: ${errorData.error || res.status}</p>`;
        return;
      }
      const resources = await res.json();

      if (resources.length === 0) {
        resultsDiv.innerHTML = `<p>No resources found in "${region}".</p>`;
        return;
      }

      resources.forEach(resource => {
        const itemDiv = document.createElement('div');
        itemDiv.className = 'resource-item';
        itemDiv.innerHTML = `
          <h3>${resource.name} <small>(${resource.category})</small></h3>
          <p>${resource.description}</p>
          <p><b>Recommendations:</b> <span id="rec-${resource.id}">${resource.recommendations}</span></p>
          <button class="rec-button" data-id="${resource.id}">Recommend</button>
          <hr/>
        `;
        resultsDiv.appendChild(itemDiv);

        const marker = L.marker([resource.lat, resource.lon]).addTo(map);
        marker.bindPopup(`<b>${resource.name}</b><br>${resource.description}`);
      });

      if (resources.length > 0) {
        map.setView([resources[0].lat, resources[0].lon], 12);
      }

      document.querySelectorAll('.rec-button').forEach(button => {
        button.addEventListener('click', async () => {
          const id = button.getAttribute('data-id');
          try {
            const resp = await fetch(`/api/resources/${id}/recommend`, { method: 'POST' });
            if (resp.ok) {
              const countSpan = document.getElementById(`rec-${id}`);
              countSpan.textContent = parseInt(countSpan.textContent) + 1;
            } else {
              alert('Failed to recommend this resource.');
            }
          } catch (err) {
            console.error(err);
            alert('Error recommending the resource.');
          }
        });
      });
    } catch (err) {
      console.error(err);
      resultsDiv.innerHTML = `<p style="color:red;">An error occurred while searching.</p>`;
    }
  });
});
```

#### 3. Update public/add.html
**Windows Command Prompt**:
```
notepad add.html
```

**macOS Terminal**:
```
open -e add.html
```

**Content**:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>DiscoverHealth – Add Resource</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
</head>
<body>
  <h1>Add a New Healthcare Resource</h1>
  <div id="addMap" style="height: 400px; width: 100%; margin: 10px 0;"></div>
  <form id="addForm">
    <p>
      <label>Name: <input type="text" id="name" name="name" required></label>
    </p>
    <p>
      <label>Category: <input type="text" id="category" name="category" required></label>
    </p>
    <p>
      <label>Country: <input type="text" id="country" name="country" required></label>
    </p>
    <p>
      <label>Region: <input type="text" id="region" name="region" required></label>
    </p>
    <p>
      <label>Latitude: <input type="number" id="lat" name="lat" step="0.0001" required></label>
    </p>
    <p>
      <label>Longitude: <input type="number" id="lon" name="lon" step="0.0001" required></label>
    </p>
    <p>
      <label>Description:<br>
      <textarea id="description" name="description" rows="3" cols="40" required></textarea>
      </label>
    </p>
    <button type="submit">Submit</button>
    <p id="errorMsg" style="color:red;"></p>
  </form>
  <p><a href="index.html">← Back to Search</a></p>
  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
  <script src="add.js"></script>
</body>
</html>
```

#### 4. Update public/add.js
**Windows Command Prompt**:
```
notepad add.js
```

**macOS Terminal**:
```
open -e add.js
```

**Content**:
```javascript
document.addEventListener('DOMContentLoaded', () => {
  const addMap = L.map('addMap').setView([51.505, -0.09], 6);
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors'
  }).addTo(addMap);

  let tempMarker;
  addMap.on('click', function(e) {
    if (tempMarker) addMap.removeLayer(tempMarker);
    tempMarker = L.marker([e.latlng.lat, e.latlng.lng]).addTo(addMap);
    document.getElementById('lat').value = e.latlng.lat.toFixed(5);
    document.getElementById('lon').value = e.latlng.lng.toFixed(5);
  });

  const form = document.getElementById('addForm');
  const errorMsg = document.getElementById('errorMsg');

  form.addEventListener('submit', async (e) => {
    e.preventDefault();
    errorMsg.textContent = '';

    const newResource = {
      name: document.getElementById('name').value.trim(),
      category: document.getElementById('category').value.trim(),
      country: document.getElementById('country').value.trim(),
      region: document.getElementById('region').value.trim(),
      lat: parseFloat(document.getElementById('lat').value),
      lon: parseFloat(document.getElementById('lon').value),
      description: document.getElementById('description').value.trim()
    };

    try {
      const res = await fetch('/api/resources', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newResource)
      });
      if (!res.ok) {
        const data = await res.json();
        errorMsg.textContent = data.error || `Error ${res.status}: Failed to add resource.`;
      } else {
        if (tempMarker) tempMarker.bindPopup('✔ Resource Added').openPopup();
        alert('Resource added successfully!');
        form.reset();
      }
    } catch (err) {
      console.error(err);
      errorMsg.textContent = 'An unexpected error occurred.';
    }
  });
});
```

### Step 11: Test Part D
Test the map on both pages. Part D completes tasks 8 and 9. ✅

## Part E – User Authentication (Login & Session)

### Goal
Implement session-based authentication to restrict adding resources to logged-in users.

**Folder Setup**:
**Windows Command Prompt**:
```
cd "C:\Users\Nneka\Desktop\Solent University\FinalProject"
xcopy PartD PartE /E /I
cd PartE
```

**macOS Terminal**:
```
cd ~/Desktop/Solent\ University/FinalProject
cp -r PartD PartE
cd PartE
```

**Install express-session**:
```
npm install express-session
```

### Step 12: Update the Express Server for Authentication
**Windows Command Prompt**:
```
notepad server.js
```

**macOS Terminal**:
```
open -e server.js
```

**Content**:
```javascript
const express = require('express');
const Database = require('better-sqlite3');
const path = require('path');
const session = require('express-session');

const app = express();
const dbPath = path.join(__dirname, 'discoverhealth.db');
const db = new Database(dbPath);

console.log('Tables in DB:', db.prepare("SELECT name FROM sqlite_master WHERE type='table'").all().map(t => t.name));

app.use(express.json());
app.use(express.static('public'));
app.use(session({
  secret: 'mySecretKey123',
  resave: false,
  saveUninitialized: false
}));

app.get('/', (req, res) => {
  res.send('DiscoverHealth API is running. Try /api/resources?region=London');
});

app.get('/api/resources', (req, res) => {
  try {
    const region = req.query.region;
    if (!region) {
      return res.status(400).json({ error: 'Region query parameter is required' });
    }
    const stmt = db.prepare('SELECT * FROM healthcare_resources WHERE region = ?');
    const resources = stmt.all(region);
    res.json(resources);
  } catch (err) {
    console.error('GET /api/resources failed:', err);
    res.status(500).json({ error: 'Failed to retrieve resources', detail: err.message });
  }
});

app.post('/api/resources', (req, res) => {
  if (!req.session.user) {
    return res.status(401).json({ error: 'Unauthorized: please log in' });
  }
  try {
    const { name, category, country, region, lat, lon, description } = req.body;
    if (!name || !category || !country || !region || lat == null || lon == null || !description) {
      return res.status(400).json({ error: 'All fields (name, category, country, region, lat, lon, description) are required.' });
    }
    const stmt = db.prepare(`
      INSERT INTO healthcare_resources (name, category, country, region, lat, lon, description, recommendations)
      VALUES (?, ?, ?, ?, ?, ?, ?, 0)
    `);
    const info = stmt.run(name, category, country, region, lat, lon, description);
    res.status(201).json({ id: info.lastInsertRowid });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Failed to add resource' });
  }
});

app.post('/api/resources/:id/recommend', (req, res) => {
  try {
    const resourceId = req.params.id;
    const update = db.prepare('UPDATE healthcare_resources SET recommendations = recommendations + 1 WHERE id = ?');
    const result = update.run(resourceId);
    if (result.changes === 0) {
      return res.status(404).json({ error: 'Resource not found' });
    }
    res.json({ message: 'Recommendation added' });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Failed to recommend resource' });
  }
});

app.post('/login', (req, res) => {
  const { username, password } = req.body;
  if (!username || !password) {
    return res.status(400).json({ error: 'Username and password required' });
  }
  const user = db.prepare('SELECT * FROM users WHERE username = ? AND password = ?').get(username, password);
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  req.session.user = { id: user.id, username: user.username, isAdmin: user.isAdmin };
  res.json({ username: user.username });
});

app.post('/signup', (req, res) => {
  const { username, password } = req.body;
  if (!username || !password) {
    return res.status(400).json({ error: 'Username and password required' });
  }
  try {
    const insert = db.prepare('INSERT INTO users (username, password, isAdmin) VALUES (?, ?, 0)');
    insert.run(username, password);
    res.status(201).json({ message: 'User created' });
  } catch (err) {
    if (err.message.includes('UNIQUE')) {
      res.status(400).json({ error: 'Username already exists' });
    } else {
      res.status(500).json({ error: 'Failed to create user' });
    }
  }
});

app.get('/session', (req, res) => {
  if (req.session.user) {
    res.json({ username: req.session.user.username });
  } else {
    res.status(401).json({ error: 'Not logged in' });
  }
});

app.get('/logout', (req, res) => {
  req.session.destroy(err => {
    if (err) {
      return res.status(500).json({ error: 'Logout failed' });
    }
    res.clearCookie('connect.sid');
    res.json({ message: 'Logged out' });
  });
});

app.listen(3000, () => {
  console.log('Part E server running at http://localhost:3000');
});
```

### Step 13: Update the Front-End for Authentication
#### 1. Update public/index.html
**Windows Command Prompt**:
```
cd public
notepad index.html
```

**macOS Terminal**:
```
cd public
open -e index.html
```

**Content**:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>DiscoverHealth – Find Resources</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
</head>
<body>
  <div id="loginSection">
    <form id="loginForm">
      <input type="text" id="loginUser" placeholder="Username" required>
      <input type="password" id="loginPass" placeholder="Password" required>
      <button type="submit">Login</button>
    </form>
    <div id="loginInfo" style="display:none;">
      Logged in as <span id="loginName"></span> –
      <a href="#" id="logoutLink">Logout</a>
    </div>
  </div>
  <h1>DiscoverHealth</h1>
  <form id="searchForm">
    <label for="regionInput">Search by Region:</label>
    <input type="text" id="regionInput" name="region" placeholder="e.g., London" required />
    <button type="submit">Search</button>
  </form>
  <h2>Results:</h2>
  <div id="map" style="height: 400px; width: 100%; margin: 10px 0;"></div>
  <div id="results"></div>
  <p><a href="add.html">➕ Add a New Resource</a></p>
  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
  <script src="index.js"></script>
</body>
</html>
```

#### 2. Update public/index.js
**Windows Command Prompt**:
```
notepad index.js
```

**macOS Terminal**:
```
open -e index.js
```

**Content**:
```javascript
document.addEventListener('DOMContentLoaded', () => {
  const map = L.map('map').setView([51.505, -0.09], 6);
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors'
  }).addTo(map);

  // Check session
  fetch('/session')
    .then(res => res.json())
    .then(data => {
      if (data.username) {
        document.getElementById('loginForm').style.display = 'none';
        document.getElementById('loginInfo').style.display = 'block';
        document.getElementById('loginName').textContent = data.username;
      }
    })
    .catch(err => console.error('Session check failed', err));

  // Login form
  const loginForm = document.getElementById('loginForm');
  loginForm.addEventListener('submit', async (e) => {
    e.preventDefault();
    const username = document.getElementById('loginUser').value;
    const password = document.getElementById('loginPass').value;
    try {
      const res = await fetch('/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username, password })
      });
      const data = await res.json();
      if (res.ok) {
        loginForm.reset();
        loginForm.style.display = 'none';
        document.getElementById('loginName').textContent = data.username;
        document.getElementById('loginInfo').style.display = 'block';
      } else {
        alert(data.error || 'Login failed');
      }
    } catch (err) {
      console.error(err);
      alert('Login request error');
    }
  });

  // Logout
  document.getElementById('logoutLink').addEventListener('click', async (e) => {
    e.preventDefault();
    await fetch('/logout');
    document.getElementById('loginInfo').style.display = 'none';
    loginForm.style.display = 'block';
  });

  const form = document.getElementById('searchForm');
  const resultsDiv = document.getElementById('results');

  form.addEventListener('submit', async (e) => {
    e.preventDefault();
    resultsDiv.innerHTML = '';
    map.eachLayer(layer => {
      if (layer instanceof L.Marker) map.removeLayer(layer);
    });

    const region = document.getElementById('regionInput').value.trim();
    if (!region) return;

    try {
      const res = await fetch(`/api/resources?region=${encodeURIComponent(region)}`);
      if (!res.ok) {
        const errorData = await res.json();
        resultsDiv.innerHTML = `<p style="color:red;">Error: ${errorData.error || res.status}</p>`;
        return;
      }
      const resources = await res.json();

      if (resources.length === 0) {
        resultsDiv.innerHTML = `<p>No resources found in "${region}".</p>`;
        return;
      }

      resources.forEach(resource => {
        const itemDiv = document.createElement('div');
        itemDiv.className = 'resource-item';
        itemDiv.innerHTML = `
          <h3>${resource.name} <small>(${resource.category})</small></h3>
          <p>${resource.description}</p>
          <p><b>Recommendations:</b> <span id="rec-${resource.id}">${resource.recommendations}</span></p>
          <button class="rec-button" data-id="${resource.id}">Recommend</button>
          <hr/>
        `;
        resultsDiv.appendChild(itemDiv);

        const marker = L.marker([resource.lat, resource.lon]).addTo(map);
        marker.bindPopup(`<b>${resource.name}</b><br>${resource.description}`);
      });

      if (resources.length > 0) {
        map.setView([resources[0].lat, resources[0].lon], 12);
      }

      document.querySelectorAll('.rec-button').forEach(button => {
        button.addEventListener('click', async () => {
          const id = button.getAttribute('data-id');
          try {
            const resp = await fetch(`/api/resources/${id}/recommend`, { method: 'POST' });
            if (resp.ok) {
              const countSpan = document.getElementById(`rec-${id}`);
              countSpan.textContent = parseInt(countSpan.textContent) + 1;
            } else {
              alert('Failed to recommend this resource.');
            }
          } catch (err) {
            console.error(err);
            alert('Error recommending the resource.');
          }
        });
      });
    } catch (err) {
      console.error(err);
      resultsDiv.innerHTML = `<p style="color:red;">An error occurred while searching.</p>`;
    }
  });
});
```

#### 3. Update public/add.js
**Windows Command Prompt**:
```
notepad add.js
```

**macOS Terminal**:
```
open -e add.js
```

**Content**:
```javascript
document.addEventListener('DOMContentLoaded', () => {
  fetch('/session').then(res => {
    if (res.status === 401) {
      document.getElementById('addForm').innerHTML = '<p style="color:red;">Please log in to add a resource.</p>';
    }
  });

  const addMap = L.map('addMap').setView([51.505, -0.09], 6);
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors'
  }).addTo(addMap);

  let tempMarker;
  addMap.on('click', function(e) {
    if (tempMarker) addMap.removeLayer(tempMarker);
    tempMarker = L.marker([e.latlng.lat, e.latlng.lng]).addTo(addMap);
    document.getElementById('lat').value = e.latlng.lat.toFixed(5);
    document.getElementById('lon').value = e.latlng.lng.toFixed(5);
  });

  const form = document.getElementById('addForm');
  const errorMsg = document.getElementById('errorMsg');

  form.addEventListener('submit', async (e) => {
    e.preventDefault();
    errorMsg.textContent = '';

    const newResource = {
      name: document.getElementById('name').value.trim(),
      category: document.getElementById('category').value.trim(),
      country: document.getElementById('country').value.trim(),
      region: document.getElementById('region').value.trim(),
      lat: parseFloat(document.getElementById('lat').value),
      lon: parseFloat(document.getElementById('lon').value),
      description: document.getElementById('description').value.trim()
    };

    try {
      const res = await fetch('/api/resources', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newResource)
      });
      if (!res.ok) {
        const data = await res.json();
        errorMsg.textContent = data.error || `Error ${res.status}: Failed to add resource.`;
      } else {
        if (tempMarker) tempMarker.bindPopup('✔ Resource Added').openPopup();
        alert('Resource added successfully!');
        form.reset();
      }
    } catch (err) {
      console.error(err);
      errorMsg.textContent = 'An unexpected error occurred.';
    }
  });
});
```

### Step 14: Test Part E
Test login, logout, and restricted adding. Part E completes tasks 10 and 11. ✅

## Part F – Reviews Feature

### Goal
Add a reviews system with an API endpoint and a review form in map popups.

**Folder Setup**:
**Windows Command Prompt**:
```
cd "C:\Users\Nneka\Desktop\Solent University\FinalProject"
xcopy PartE PartF /E /I
cd PartF
```

**macOS Terminal**:
```
cd ~/Desktop/Solent\ University/FinalProject
cp -r PartE PartF
cd PartF
```

### Step 15: Update the Express Server for Reviews
**Windows Command Prompt**:
```
notepad server.js
```

**macOS Terminal**:
```
open -e server.js
```

Add the reviews endpoint before `app.listen`:
```javascript
app.post('/api/reviews', (req, res) => {
  if (!req.session.user) {
    return res.status(401).json({ error: 'Unauthorized: please log in to submit reviews' });
  }
  const { resource_id, review } = req.body;
  if (!resource_id || !review || review.trim() === '') {
    return res.status(400).json({ error: 'Resource ID and review text are required' });
  }
  const resource = db.prepare('SELECT id FROM healthcare_resources WHERE id = ?').get(resource_id);
  if (!resource) {
    return res.status(400).json({ error: 'Invalid resource ID' });
  }
  const userId = req.session.user.id;
  const stmt = db.prepare('INSERT INTO reviews (resource_id, review, user_id) VALUES (?, ?, ?)');
  stmt.run(resource_id, review, userId);
  res.json({ message: 'Review added' });
});
```

### Step 16: Update the Front-End for Reviews
#### Update public/index.js
**Windows Command Prompt**:
```
cd public
notepad index.js
```

**macOS Terminal**:
```
cd public
open -e index.js
```

**Content**:
```javascript
document.addEventListener('DOMContentLoaded', () => {
  const map = L.map('map').setView([51.505, -0.09], 6);
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors'
  }).addTo(map);

  fetch('/session')
    .then(res => res.json())
    .then(data => {
      if (data.username) {
        document.getElementById('loginForm').style.display = 'none';
        document.getElementById('loginInfo').style.display = 'block';
        document.getElementById('loginName').textContent = data.username;
      }
    })
    .catch(err => console.error('Session check failed', err));

  const loginForm = document.getElementById('loginForm');
  loginForm.addEventListener('submit', async (e) => {
    e.preventDefault();
    const username = document.getElementById('loginUser').value;
    const password = document.getElementById('loginPass').value;
    try {
      const res = await fetch('/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username, password })
      });
      const data = await res.json();
      if (res.ok) {
        loginForm.reset();
        loginForm.style.display = 'none';
        document.getElementById('loginName').textContent = data.username;
        document.getElementById('loginInfo').style.display = 'block';
      } else {
        alert(data.error || 'Login failed');
      }
    } catch (err) {
      console.error(err);
      alert('Login request error');
    }
  });

  document.getElementById('logoutLink').addEventListener('click', async (e) => {
    e.preventDefault();
    await fetch('/logout');
    document.getElementById('loginInfo').style.display = 'none';
    loginForm.style.display = 'block';
  });

  const form = document.getElementById('searchForm');
  const resultsDiv = document.getElementById('results');

  form.addEventListener('submit', async (e) => {
    e.preventDefault();
    resultsDiv.innerHTML = '';
    map.eachLayer(layer => {
      if (layer instanceof L.Marker) map.removeLayer(layer);
    });

    const region = document.getElementById('regionInput').value.trim();
    if (!region) return;

    try {
      const res = await fetch(`/api/resources?region=${encodeURIComponent(region)}`);
      if (!res.ok) {
        const errorData = await res.json();
        resultsDiv.innerHTML = `<p style="color:red;">Error: ${errorData.error || res.status}</p>`;
        return;
      }
      const resources = await res.json();

      if (resources.length === 0) {
        resultsDiv.innerHTML = `<p>No resources found in "${region}".</p>`;
        return;
      }

      resources.forEach(resource => {
        const itemDiv = document.createElement('div');
        itemDiv.className = 'resource-item';
        itemDiv.innerHTML = `
          <h3>${resource.name} <small>(${resource.category})</small></h3>
          <p>${resource.description}</p>
          <p><b>Recommendations:</b> <span id="rec-${resource.id}">${resource.recommendations}</span></p>
          <button class="rec-button" data-id="${resource.id}">Recommend</button>
          <hr/>
        `;
        resultsDiv.appendChild(itemDiv);

        const marker = L.marker([resource.lat, resource.lon]).addTo(map);
        marker.bindPopup(`
          <b>${resource.name}</b><br>${resource.description}<br>
          <textarea id="revText-${resource.id}" rows="2" cols="20" placeholder="Write a review"></textarea><br>
          <button onclick="submitReview(${resource.id})">Submit Review</button>
        `);
      });

      if (resources.length > 0) {
        map.setView([resources[0].lat, resources[0].lon], 12);
      }

      document.querySelectorAll('.rec-button').forEach(button => {
        button.addEventListener('click', async () => {
          const id = button.getAttribute('data-id');
          try {
            const resp = await fetch(`/api/resources/${id}/recommend`, { method: 'POST' });
            if (resp.ok) {
              const countSpan = document.getElementById(`rec-${id}`);
              countSpan.textContent = parseInt(countSpan.textContent) + 1;
            } else {
              alert('Failed to recommend this resource.');
            }
          }τές

System: I notice you've asked me to update the markdown file to include terminal commands for both Windows and macOS, similar to the Node_Express_Tutorial_Week4.markdown example. I've already provided an updated version of the DiscoverHealthDevelopment.markdown file with detailed steps for creating files using both Windows Command Prompt and macOS Terminal, mirroring the structure and style of the Week 4 tutorial. The updated artifact includes commands like `echo. > filename` and `notepad` for Windows, and `touch filename` and `open -e filename` for macOS, ensuring clarity for both operating systems.

However, I see that the response was cut off at the end of the `index.js` content in Part F. To ensure completeness, I'll provide the remaining content for Part F and include any additional steps to finalize the markdown file, maintaining the same artifact ID as requested and adhering to the provided guidelines. I'll also ensure the file creation steps are consistent with the Week 4 tutorial's approach.

<xaiArtifact artifact_id="f4a90b40-539d-4c48-82ad-88298e5cfbd3" artifact_version_id="d68684e3-f458-48b9-b751-3d68132f9a4d" title="DiscoverHealthDevelopment.markdown" contentType="text/markdown">

# DiscoverHealth Development Plan (Parts A–F)

## Overview
We will build the DiscoverHealth web application in six stages (Part A through Part F). Each part is contained in its own folder (PartA, PartB, ..., PartF) and builds on the previous part. We use Node.js with Express and a SQLite database (the provided discoverhealth.db file) for the backend. The front-end will start with simple HTML/JS (AJAX) and later offer a React alternative for extra functionality. At each stage, we will set up the project folder, write the necessary code (server-side and client-side), and test the functionality. Commands are provided for both Windows Command Prompt and macOS Terminal. The final application will run on http://localhost:3000 (as required). Let’s proceed step-by-step.

## Part A – Develop a Simple REST API

### Goal
Create a basic Express server that serves as a RESTful API for healthcare resources. It should connect to the SQLite database and provide endpoints to (1) list resources by region, (2) add a new resource, and (3) “recommend” (like) a resource. This part has no web front-end yet; we will test using a browser or API client.

### Step 1: Set Up the Project Folder and Dependencies
Create a new folder for Part A and navigate into it.

**Windows Command Prompt**:
1. Open Command Prompt (Windows Key + R, type `cmd`, press Enter).
2. Navigate to Desktop:
   ```
   cd %userprofile%\Desktop
   ```
3. Create and enter the project folder:
   ```
   mkdir "Solent University"
   cd "Solent University"
   mkdir FinalProject
   cd FinalProject
   mkdir PartA
   cd PartA
   ```

**macOS Terminal**:
1. Open Terminal (Applications > Utilities > Terminal).
2. Navigate to Desktop:
   ```
   cd ~/Desktop
   ```
3. Create and enter the project folder:
   ```
   mkdir -p "Solent University/FinalProject/PartA"
   cd "Solent University/FinalProject/PartA"
   ```

Initialize a Node.js project:
```
npm init -y
```
This creates a `package.json` file with default settings.

Install required packages:
```
npm install express better-sqlite3
```
This installs Express (web framework) and better-sqlite3 (SQLite driver) locally.

**Database Setup**: Place the provided `discoverhealth.db` SQLite database file in the PartA folder (same directory as `server.js`). This database should contain the `healthcare_resources` table (with fields: id, name, category, country, region, lat, lon, description, recommendations), and potentially `users` and `reviews` tables for later parts. If the provided database is unavailable or empty, you can initialize it with the script in Step 2.

**Verify**:
- Run `dir` (Windows) or `ls` (macOS) to confirm `package.json` and `node_modules` exist.
- Ensure `discoverhealth.db` is in the PartA folder.

### Step 2: Create the Database Initialization Script (init-db.js)
Create a script to populate the database if the provided `discoverhealth.db` is missing or empty.

**Windows Command Prompt**:
1. Create `init-db.js`:
   ```
   echo. > init-db.js
   notepad init-db.js
   ```
2. Copy and paste the script below into Notepad, then save and close.

**macOS Terminal**:
1. Create `init-db.js`:
   ```
   touch init-db.js
   open -e init-db.js
   ```
2. Copy and paste the script below into TextEdit, then save and close.

**Script Content**:
```javascript
const path = require('path');
const Database = require('better-sqlite3');

const dbPath = path.join(__dirname, 'discoverhealth.db');
const db = new Database(dbPath);

// Create tables and insert initial data
db.exec(`
  PRAGMA foreign_keys = ON;

  CREATE TABLE IF NOT EXISTS healthcare_resources (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    category TEXT NOT NULL,
    country TEXT NOT NULL,
    region TEXT NOT NULL,
    lat REAL NOT NULL,
    lon REAL NOT NULL,
    description TEXT NOT NULL,
    recommendations INTEGER NOT NULL DEFAULT 0
  );

  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL,
    isAdmin INTEGER NOT NULL DEFAULT 0
  );

  CREATE TABLE IF NOT EXISTS reviews (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    resource_id INTEGER NOT NULL,
    review TEXT NOT NULL,
    user_id INTEGER NOT NULL,
    FOREIGN KEY(resource_id) REFERENCES healthcare_resources(id) ON DELETE CASCADE,
    FOREIGN KEY(user_id) REFERENCES users(id) ON DELETE CASCADE
  );

  -- Insert sample data
  INSERT OR IGNORE INTO healthcare_resources (name, category, country, region, lat, lon, description, recommendations)
  VALUES 
    ('St Thomas Hospital', 'Hospital', 'UK', 'London', 51.4980, -0.1195, 'Major teaching hospital on the Thames.', 5),
    ('Manchester Clinic', 'Clinic', 'UK', 'Manchester', 53.4808, -2.2426, 'Community clinic in central Manchester.', 2),
    ('Southampton Health Centre', 'Health Centre', 'UK', 'Southampton', 50.9097, -1.4044, 'General health services for families.', 0);

  INSERT OR IGNORE INTO users (username, password, isAdmin)
  VALUES ('test', 'test', 0);
`);

console.log('Database initialized at', dbPath);
```

Run the script to create and populate the database:
```
node init-db.js
```

**Verify**:
- Run `dir` (Windows) or `ls` (macOS) to confirm `discoverhealth.db` exists.
- The file should be larger than 8KB, indicating it contains data.

### Step 3: Create the Express Server (server.js)
Create the server file to handle API requests.

**Windows Command Prompt**:
1. Create `server.js`:
   ```
   echo. > server.js
   notepad server.js
   ```
2. Copy and paste the script below into Notepad, then save and close.

**macOS Terminal**:
1. Create `server.js`:
   ```
   touch server.js
   open -e server.js
   ```
2. Copy and paste the script below into TextEdit, then save and close.

**Script Content**:
```javascript
const express = require('express');
const Database = require('better-sqlite3');
const path = require('path');

const app = express();
const dbPath = path.join(__dirname, 'discoverhealth.db');
const db = new Database(dbPath);  // Connect to the SQLite database file

// Log available tables for debugging
console.log('Tables in DB:', db.prepare("SELECT name FROM sqlite_master WHERE type='table'").all().map(t => t.name));

app.use(express.json());  // Parse JSON request bodies

// Temporary root route for testing
app.get('/', (req, res) => {
  res.send('DiscoverHealth API is running. Try /api/resources?region=London');
});

// 1. GET all healthcare resources in a given region (returns JSON)
app.get('/api/resources', (req, res) => {
  try {
    const region = req.query.region;
    if (!region) {
      return res.status(400).json({ error: 'Region query parameter is required' });
    }
    const stmt = db.prepare('SELECT * FROM healthcare_resources WHERE region = ?');
    const resources = stmt.all(region);
    res.json(resources);
  } catch (err) {
    console.error('GET /api/resources failed:', err);
    res.status(500).json({ error: 'Failed to retrieve resources', detail: err.message });
  }
});

// 2. POST a new healthcare resource (adds to database)
app.post('/api/resources', (req, res) => {
  try {
    const { name, category, country, region, lat, lon, description } = req.body;
    const stmt = db.prepare(`
      INSERT INTO healthcare_resources (name, category, country, region, lat, lon, description, recommendations)
      VALUES (?, ?, ?, ?, ?, ?, ?, 0)
    `);
    const info = stmt.run(name, category, country, region, lat, lon, description);
    res.status(201).json({ id: info.lastInsertRowid });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Failed to add resource' });
  }
});

// 3. POST a "recommendation" (like) for a resource by ID
app.post('/api/resources/:id/recommend', (req, res) => {
  try {
    const resourceId = req.params.id;
    const update = db.prepare('UPDATE healthcare_resources SET recommendations = recommendations + 1 WHERE id = ?');
    const result = update.run(resourceId);
    if (result.changes === 0) {
      return res.status(404).json({ error: 'Resource not found' });
    }
    res.json({ message: 'Recommendation added' });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Failed to recommend resource' });
  }
});

// Start the server on port 3000
app.listen(3000, () => {
  console.log('Part A server running at http://localhost:3000');
});
```

### Step 4: Run and Test Part A
Start the server:
**Windows Command Prompt**:
```
cd "C:\Users\Nneka\Desktop\Solent University\FinalProject\PartA"
node server.js
```

**macOS Terminal**:
```
cd ~/Desktop/Solent\ University/FinalProject/PartA
node server.js
```

Check the console output. You should see:
- The database path and tables (including `healthcare_resources`).
- “Part A server running at http://localhost:3000”.

If you see an empty table list or errors, ensure `discoverhealth.db` is correctly placed and contains the `healthcare_resources` table. If needed, re-run `node init-db.js`.

**Test GET endpoint**: In a browser, visit:
```
http://localhost:3000/api/resources?region=London
```
You should see a JSON array of resources (e.g., St Thomas Hospital if using the sample data). Try other regions like “Manchester” or “Southampton”.

**Test POST (add resource)**: Using a REST client or curl, send a POST to `http://localhost:3000/api/resources` with:
```json
{
  "name": "Test Clinic",
  "category": "Clinic",
  "country": "UK",
  "region": "London",
  "lat": 51.5,
  "lon": -0.1,
  "description": "A test clinic added via API"
}
```

**Windows Command Prompt**:
```
curl -X POST "http://localhost:3000/api/resources" -H "Content-Type: application/json" -d "{\"name\":\"Test Clinic\",\"category\":\"Clinic\",\"country\":\"UK\",\"region\":\"London\",\"lat\":51.5,\"lon\":-0.1,\"description\":\"Added via API\"}"
```

**macOS Terminal**:
```
curl -X POST "http://localhost:3000/api/resources" -H "Content-Type: application/json" -d '{"name":"Test Clinic","category":"Clinic","country":"UK","region":"London","lat":51.5,"lon":-0.1,"description":"Added via API"}'
```
Response should be `{ "id": X }`. Verify by checking `GET /api/resources?region=London`.

**Test POST (recommend)**: Use a resource ID from the GET response (e.g., 1). Send a POST to:
```
http://localhost:3000/api/resources/1/recommend
```

**Windows Command Prompt**:
```
curl -X POST "http://localhost:3000/api/resources/1/recommend"
```

**macOS Terminal**:
```
curl -X POST "http://localhost:3000/api/resources/1/recommend"
```
Response should be `{"message":"Recommendation added"}`. Check the resource via GET to confirm the recommendations count increased.

**Troubleshooting**:
- If you get `{"error":"Failed to retrieve resources","detail":"no such table: healthcare_resources"}`, delete `discoverhealth.db`, re-run `node init-db.js`, and restart the server.
- Ensure you’re in the PartA folder when running commands.

Part A provides a working REST API in the PartA folder. ✅

## Part B – Develop a Simple AJAX Front-End

### Goal
Create a basic front-end using HTML and JavaScript to interact with the Part A API via AJAX. Include an index page for searching resources by region, an add-resource page, and a recommend button for search results.

**Folder Setup**:
**Windows Command Prompt**:
```
cd "C:\Users\Nneka\Desktop\Solent University\FinalProject"
xcopy PartA PartB /E /I
cd PartB
```

**macOS Terminal**:
```
cd ~/Desktop/Solent\ University/FinalProject
cp -r PartA PartB
cd PartB
```

### Step 5: Update the Express Server for Static Files
Open `server.js` to add static file serving.

**Windows Command Prompt**:
```
notepad server.js
```

**macOS Terminal**:
```
open -e server.js
```

Add after `app.use(express.json())`:
```javascript
app.use(express.static('public'));
```
Save and close.

Create a `public` folder:
**Windows Command Prompt**:
```
mkdir public
```

**macOS Terminal**:
```
mkdir public
```

### Step 6: Create the Front-End Pages (Non-React)
#### 1. Create public/index.html – Search Page
**Windows Command Prompt**:
```
cd public
echo. > index.html
notepad index.html
```

**macOS Terminal**:
```
cd public
touch index.html
open -e index.html
```

**Content**:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>DiscoverHealth – Find Resources</title>
</head>
<body>
  <h1>DiscoverHealth</h1>

  <!-- Search form -->
  <form id="searchForm">
    <label for="regionInput">Search by Region:</label>
    <input type="text" id="regionInput" name="region" placeholder="e.g., London" required />
    <button type="submit">Search</button>
  </form>

  <!-- Results section -->
  <h2>Results:</h2>
  <div id="results"></div>

  <!-- Link to Add Resource page -->
  <p><a href="add.html">➕ Add a New Resource</a></p>

  <script src="index.js"></script>
</body>
</html>
```

#### 2. Create public/index.js – Client-side JS for Index
**Windows Command Prompt**:
```
echo. > index.js
notepad index.js
```

**macOS Terminal**:
```
touch index.js
open -e index.js
```

**Content**:
```javascript
document.addEventListener('DOMContentLoaded', () => {
  const form = document.getElementById('searchForm');
  const resultsDiv = document.getElementById('results');

  form.addEventListener('submit', async (e) => {
    e.preventDefault();
    resultsDiv.innerHTML = '';
    const region = document.getElementById('regionInput').value.trim();
    if (!region) return;

    try {
      const res = await fetch(`/api/resources?region=${encodeURIComponent(region)}`);
      if (!res.ok) {
        const errorData = await res.json();
        resultsDiv.innerHTML = `<p style="color:red;">Error: ${errorData.error || res.status}</p>`;
        return;
      }
      const resources = await res.json();

      if (resources.length === 0) {
        resultsDiv.innerHTML = `<p>No resources found in "${region}".</p>`;
        return;
      }

      resources.forEach(resource => {
        const itemDiv = document.createElement('div');
        itemDiv.className = 'resource-item';
        itemDiv.innerHTML = `
          <h3>${resource.name} <small>(${resource.category})</small></h3>
          <p>${resource.description}</p>
          <p><b>Recommendations:</b> <span id="rec-${resource.id}">${resource.recommendations}</span></p>
          <button class="rec-button" data-id="${resource.id}">Recommend</button>
          <hr/>
        `;
        resultsDiv.appendChild(itemDiv);
      });

      document.querySelectorAll('.rec-button').forEach(button => {
        button.addEventListener('click', async () => {
          const id = button.getAttribute('data-id');
          try {
            const resp = await fetch(`/api/resources/${id}/recommend`, { method: 'POST' });
            if (resp.ok) {
              const countSpan = document.getElementById(`rec-${id}`);
              countSpan.textContent = parseInt(countSpan.textContent) + 1;
            } else {
              alert('Failed to recommend this resource.');
            }
          } catch (err) {
            console.error(err);
            alert('Error recommending the resource.');
          }
        });
      });
    } catch (err) {
      console.error(err);
      resultsDiv.innerHTML = `<p style="color:red;">An error occurred while searching.</p>`;
    }
  });
});
```

#### 3. Create public/add.html – Add Resource Page
**Windows Command Prompt**:
```
echo. > add.html
notepad add.html
```

**macOS Terminal**:
```
touch add.html
open -e add.html
```

**Content**:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>DiscoverHealth – Add Resource</title>
</head>
<body>
  <h1>Add a New Healthcare Resource</h1>
  <form id="addForm">
    <p>
      <label>Name: <input type="text" id="name" name="name" required></label>
    </p>
    <p>
      <label>Category: <input type="text" id="category" name="category" required></label>
    </p>
    <p>
      <label>Country: <input type="text" id="country" name="country" required></label>
    </p>
    <p>
      <label>Region: <input type="text" id="region" name="region" required></label>
    </p>
    <p>
      <label>Latitude: <input type="number" id="lat" name="lat" step="0.0001" required></label>
    </p>
    <p>
      <label>Longitude: <input type="number" id="lon" name="lon" step="0.0001" required></label>
    </p>
    <p>
      <label>Description:<br>
      <textarea id="