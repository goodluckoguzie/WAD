# Week 7: DiscoverHealth Part D – Adding a Map with Leaflet

## Introduction

In **DiscoverHealth Part D**, you’ll enhance the Part C application to meet Tasks 8 and 9 of the assignment by integrating an OpenStreetMap-based map using Leaflet. Task 8 adds a map to the search page (Task 4) to display healthcare resources as markers with popups showing name and description. Task 9 allows users to add resources by clicking the map, collecting input via a form, sending it to the POST `/api/resources` endpoint (Task 2), and adding a marker only on server confirmation. You’ll use **Windows Command Prompt** or **macOS Terminal**, building on Part C to achieve a 52% grade threshold.

### What You’ll Learn

* **Leaflet Integration**: Use Leaflet with OpenStreetMap to display interactive maps.
* **Map Markers (Task 8)**: Display resources as markers with popups.
* **Map-Based Input (Task 9)**: Capture map clicks to set coordinates and add resources via API.
* **Dynamic UI**: Manage form visibility and marker updates in React.

### What You’ll Do

1. **Exercise A**: Set up `DiscoverHealthPartD` folder, copy Part C files, and install Leaflet.
2. **Exercise B**: Update `SearchView.jsx` to add Leaflet map and map-based resource addition.
3. **Exercise C**: Verify `server.js` and `AddResourceView.jsx` functionality.
4. **Exercise D**: Run servers, test map display, and verify resource addition via map clicks.
5. **Save Everything**: Organize work and (optionally) push to GitHub.

### Prerequisites

* Completed **DiscoverHealth Part C** with working React frontend, Express server, and `discoverhealth.db`.
* **Node.js** installed (check: `node --version`, `npm --version`).
* Familiarity with command-line (`cd`, `mkdir`, `copy`/`cp`).
* Part C project folder at `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartC`.
* Install Node.js from [nodejs.org](https://nodejs.org) if needed.

## Step 1: Understanding the Basics

### What is Part D?

**DiscoverHealth Part D** implements Tasks 8 and 9:
* **Task 8**: Add a Leaflet map with OpenStreetMap tiles to `SearchView.jsx`, displaying resources as markers. Clicking a marker shows name and description.
* **Task 9**: Allow users to click the map to set coordinates, show a form, send data to POST `/api/resources`, and add a marker on success.

### How Part D Builds on Part C

* **Part C**:
  * `server.js`: Has POST `/api/resources` with validation.
  * `SearchView.jsx`: Displays resources as a list with recommend buttons.
  * `AddResourceView.jsx`: Manual resource addition form.
* **Part D**:
  * Update `SearchView.jsx` to add map and map-based addition.
  * Reuse `server.js` and `AddResourceView.jsx` unchanged.

### Key Concepts

* **Leaflet**: JavaScript library for interactive maps.
* **OpenStreetMap**: Free map tile provider.
* **Map Markers**: Use `L.marker` to plot resources.
* **Map Clicks**: Capture `latlng` to set coordinates for new resources.
* **React Integration**: Use `useEffect` for map initialization, `useRef` for map instance.

## Step 2: Exercise A – Set Up the Project

1. **Open Command Prompt (Windows) or Terminal (macOS)**:
   * **Windows**: `Win + R`, `cmd`, Enter.
   * **macOS**: Open Terminal via Spotlight.

2. **Create Project Folder**:
   ```bash
   cd %userprofile%\Desktop\Solent University\FinalProject  # Windows
   cd ~/Desktop/Solent University/FinalProject             # macOS
   mkdir DiscoverHealthPartD
   cd DiscoverHealthPartD
   ```

3. **Copy Files from Part C**:
   * Copy `discoverhealth.db` and `server.js`:
     ```bash
     # Windows
     copy ..\DiscoverHealthPartC\discoverhealth.db .
     copy ..\DiscoverHealthPartC\server.js .

     # macOS
     cp ../DiscoverHealthPartC/discoverhealth.db .
     cp ../DiscoverHealthPartC/server.js .
     ```
   * Copy `frontend` folder:
     ```bash
     # Windows
     xcopy ..\DiscoverHealthPartC\frontend frontend /s /e /h /i

     # macOS
     cp -R ../DiscoverHealthPartC/frontend .
     ```

4. **Install Backend Dependencies**:
   ```bash
   npm install express better-sqlite3 cors
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

## Step 3: Exercise B – Update SearchView.jsx for Map Functionality

Add Leaflet map and map-based resource addition to `SearchView.jsx`.

### B.1: Update `SearchView.jsx`

1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartD\frontend\src\SearchView.jsx`

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
    * SearchView lets users search for healthcare resources by region and displays them
    * as a list and on a Leaflet map. Users can recommend resources and add new ones by clicking the map.
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
     const mapRef = useRef(null);
     const mapInstanceRef = useRef(null);
     const markersRef = useRef([]);

     useEffect(() => {
       // Initialize Leaflet map
       if (!mapInstanceRef.current) {
         mapInstanceRef.current = L.map(mapRef.current, {
           center: [51.505, -0.09], // Default: London
           zoom: 10
         });
         L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
           attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
         }).addTo(mapInstanceRef.current);

         // Handle map clicks for adding resources
         mapInstanceRef.current.on('click', (e) => {
           setClickedLatLon([e.latlng.lat, e.latlng.lng]);
           setShowAddForm(true);
         });
       }

       return () => {
         if (mapInstanceRef.current) {
           mapInstanceRef.current.remove();
           mapInstanceRef.current = null;
         }
       };
     }, []);

     async function handleSearch(e) {
       e.preventDefault();
       setError('');
       setMessage('');
       if (!region.trim()) {
         setError('Region is required.');
         return;
       }
       try {
         const res = await fetch(`http://localhost:3000/api/resources?region=${encodeURIComponent(region)}`);
         const data = await res.json();
         if (res.ok) {
           setResources(data);
           // Update map: clear existing markers
           markersRef.current.forEach(marker => marker.remove());
           markersRef.current = [];
           // Add new markers
           data.forEach(resource => {
             const marker = L.marker([resource.lat, resource.lon])
               .addTo(mapInstanceRef.current)
               .bindPopup(`<b>${resource.name}</b><br>${resource.description}`);
             markersRef.current.push(marker);
           });
           // Center map on first resource or keep default
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
           method: 'POST'
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
           })
         });
         const data = await res.json();
         if (res.ok) {
           setMessage(`Resource added with ID ${data.id}`);
           setNewResource({ name: '', category: '', country: '', region: '', description: '' });
           setShowAddForm(false);
           // Add new marker
           const marker = L.marker([clickedLatLon[0], clickedLatLon[1]])
             .addTo(mapInstanceRef.current)
             .bindPopup(`<b>${newResource.name}</b><br>${newResource.description}`);
           markersRef.current.push(marker);
           // Refresh resources
           const resFetch = await fetch(`http://localhost:3000/api/resources?region=${encodeURIComponent(region)}`);
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

     return (
       <div>
         <h2>Search Healthcare Resources</h2>
         <Link to="/add">Add Resource (Form)</Link>
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
   * Verify:
     ```bash
     dir
     ren SearchView.jsx.txt SearchView.jsx
     ```

5. **Code Explanation**:
   * **Imports**: Adds `leaflet` and its CSS for map functionality.
   * **State/Refs**: Manages map state (`showAddForm`, `newResource`, `clickedLatLon`) and refs for map and markers.
   * **useEffect (Task 8)**: Initializes Leaflet map with OpenStreetMap tiles, adds click handler for Task 9.
   * **handleSearch (Task 8)**: Fetches resources, adds markers with popups (`name`, `description`), centers map.
   * **handleRecommend**: Updates recommendations.
   * **handleAddSubmit (Task 9)**: Sends map-click coordinates and form data to API, adds marker on success.
   * **JSX**: Adds map div and conditional form for map-based addition.
   * **New Additions**:
     - **Task 8**: Leaflet map with markers and popups.
     - **Task 9**: Map-click form for adding resources, with marker addition on success.
   * **Function**: Visualizes resources geographically and enables intuitive location-based addition.

## Step 4: Exercise C – Verify Server and AddResourceView

Ensure `server.js` and `AddResourceView.jsx` are unchanged and functional.

### C.1: Verify `server.js`

1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartD\server.js`

2. **Check Content**: Matches code above (unchanged from Part C).

### C.2: Verify `AddResourceView.jsx`

1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartD\frontend\src\AddResourceView.jsx`

2. **Check Content**: Matches code above (unchanged).

## Step 5: Exercise D – Run and Test the Application

1. **Start Express Server**:
   ```bash
   cd C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartD
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
     cd C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartD\frontend
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
     npm install react-router-dom leaflet
     npm run dev
     ```

3. **Test Tasks 8 and 9**:
   * Visit `http://localhost:5173/`.
   * **Task 8**:
     - Enter region (e.g., “London”).
     - Submit to see resources in list and as map markers.
     - Click a marker to verify popup with name and description.
   * **Task 9**:
     - Click map to open form with coordinates.
     - Enter data (e.g., Name: “Test Clinic”, Category: “Clinic”, Country: “UK”, Region: “London”, Description: “Test”).
     - Submit to see success message and new marker.
     - Submit with missing fields to see error.
     - Cancel form to close it.
   * **Postman Test**:
     - GET `http://localhost:3000/api/resources?region=London`
     - POST `http://localhost:3000/api/resources` with valid/invalid data.

4. **Troubleshooting**:
   * **Map Not Displaying**: Check `leaflet` import, CSS, and `<div ref={mapRef}>` height.
   * **CORS Errors**: Verify `cors` in `server.js`.
   * **No Markers**: Ensure `discoverhealth.db` has data for the region.
   * **Dependency Issues**:
     ```bash
     cd frontend
     del node_modules /s /q
     del package-lock.json
     npm install
     npm install react-router-dom leaflet
     ```

## Step 6: Save to GitHub (Optional)

1. **Initialize Git**:
   ```bash
   git init
   git add .
   git commit -m "DiscoverHealth Part D: Leaflet Map Integration"
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
   git remote add origin https://github.com/<your-username>/DiscoverHealthPartD.git
   git branch -M main
   git push -u origin main
   ```

## Step 7: Summary and Next Steps

You’ve added a Leaflet map to DiscoverHealth, meeting Tasks 8 and 9! Recap:
* Copied Part C files.
* Installed `leaflet` and updated `SearchView.jsx` for map display and addition.
* Tested markers, popups, and map-based resource addition.
* Verified `server.js` and `AddResourceView.jsx`.

**Next Steps**:
* Proceed to Part E (Tasks 10-11: session-based login).
* Enhance map (e.g., validate lat/lon ranges).
* Debug with browser console (F12) and server logs.