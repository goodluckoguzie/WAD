# Week 12: DiscoverHealth Part I – Bonus "Use My Location" Map Control

## Introduction

In **DiscoverHealth Part I**, you'll enhance the project with a bonus "Use My Location" map control, adding a smart feature to recenter the map to the user's current location. This builds on the React frontend from Parts B-H, using the `navigator.geolocation` API. The feature is optional, adding ~10 lines of code for extra credit and improving user experience.

### What You’ll Learn

* **Geolocation**: Use `navigator.geolocation.getCurrentPosition` to access user location.
* **Map Control**: Update Leaflet map view dynamically.
* **Bonus Feature**: Enhance functionality with minimal code.

### What You’ll Do

1. **Exercise A**: Set up `DiscoverHealthPartI` folder, copy Part H files, and verify dependencies.
2. **Exercise B**: Update `SearchView.jsx` to add the "Use My Location" button and functionality.
3. **Exercise C**: Test the new feature and integrate with existing code.
4. **Exercise D**: Document the bonus feature in the report.
5. **Save Everything**: Organize work and (optionally) update GitHub.

### Prerequisites

* Completed **DiscoverHealth Part H** with working React frontend and backend.
* **Node.js** installed (check: `node --version`, `npm --version`).
* Familiarity with command-line (`cd`, `mkdir`, `copy`/`cp`).
* Part H project folder at `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartH`.
* `discoverhealth.db` with all tables.

## Step 1: Understanding the Basics

### What is Part I?

**DiscoverHealth Part I** is a bonus enhancement adding a "Use My Location" button to the map in `SearchView.jsx`. It uses geolocation to recenter the Leaflet map, making it more user-friendly. This requires ~10 lines of code and integrates with the existing React setup.

### How Part I Builds on Part H

* **Part H**:
  * `server.js`: Full REST API and database.
  * `SearchView.jsx`: Map with search, add, and review features.
* **Part I**:
  - Add a button to recenter the map using geolocation.
  - Update the report to include this bonus feature.

### Key Concepts

* **Geolocation API**: `navigator.geolocation.getCurrentPosition` retrieves latitude and longitude.
* **Leaflet Integration**: `map.setView([lat, lon], 14)` adjusts the map view.
* **User Experience**: Enhances navigation with a smart control.

## Step 2: Exercise A – Set Up the Project

1. **Open Command Prompt (Windows) or Terminal (macOS)**:
   * **Windows**: `Win + R`, `cmd`, Enter.
   * **macOS**: Open Terminal via Spotlight.

2. **Create Project Folder**:
   ```bash
   cd %userprofile%\Desktop\Solent University\FinalProject  # Windows
   cd ~/Desktop/Solent University/FinalProject             # macOS
   mkdir DiscoverHealthPartI
   cd DiscoverHealthPartI
   ```

3. **Copy Files from Part H**:
   * Copy `discoverhealth.db` and `server.js`:
     ```bash
     # Windows
     copy ..\DiscoverHealthPartH\discoverhealth.db .
     copy ..\DiscoverHealthPartH\server.js .

     # macOS
     cp ../DiscoverHealthPartH/discoverhealth.db .
     cp ../DiscoverHealthPartH/server.js .
     ```
   * Copy `frontend` folder:
     ```bash
     # Windows
     xcopy ..\DiscoverHealthPartH\frontend frontend /s /e /h /i

     # macOS
     cp -R ../DiscoverHealthPartH/frontend .
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

## Step 3: Exercise B – Update `SearchView.jsx`

1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartI\frontend\src\SearchView.jsx`

2. **Open**:
   ```bash
   cd frontend/src
   notepad SearchView.jsx  # Windows
   nano SearchView.jsx    # macOS
   ```

3. **Update Content**:
   - Add the "Use My Location" button and function after the map initialization `useEffect`:
   ```jsx
   import { useState, useEffect, useRef } from 'react';
   import { Link } from 'react-router-dom';
   import L from 'leaflet';
   import 'leaflet/dist/leaflet.css';

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

     const handleUseMyLocation = () => {
       if (navigator.geolocation) {
         navigator.geolocation.getCurrentPosition(
           (position) => {
             const { latitude, longitude } = position.coords;
             mapInstanceRef.current.setView([latitude, longitude], 14);
             setMessage('Map centered to your location!');
           },
           (error) => {
             setError('Unable to retrieve your location: ' + error.message);
           }
         );
       } else {
         setError('Geolocation is not supported by your browser.');
       }
     };

     // [Rest of the existing handleSearch, submitReview, handleRecommend, etc. functions remain unchanged]

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
         <button onClick={handleUseMyLocation} style={{ margin: '16px 0' }}>Use My Location</button>
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

## Step 4: Exercise C – Test the Feature

1. **Start Express Server**:
   ```bash
   cd C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartI
   node server.js
   ```

2. **Start React Dev Server**:
   ```bash
   cd frontend
   npm run dev
   ```

3. **Test**:
   - Visit `http://localhost:5173/`.
   - Click the "Use My Location" button.
   - Expect the map to recenter to your location (zoom level 14) with a success message.
   - If geolocation is denied or unsupported, expect an error message.
   - Verify existing features (search, add, recommend, reviews) still work.

## Step 5: Exercise D – Update the Report

1. **Open Report File**:
   ```bash
   cd C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartI
   notepad report.md  # Windows
   nano report.md    # macOS
   ```

2. **Update Content**:
   - Add to the end of the "Task C-G: Feature Enhancements" section:
   ```markdown
   Additionally, a bonus "Use My Location" feature was implemented in `SearchView.jsx` using `navigator.geolocation.getCurrentPosition` to recenter the Leaflet map. This required adding a button and a function to update `map.setView([lat, lon], 14)`. The feature enhances user experience by providing a smart navigation option, completed in ~10 lines of code. Testing confirmed the map recenters correctly when geolocation is available, with error handling for unsupported browsers or denied permissions.
   ```

3. **Save**: As `report.md`.

## Step 6: Save to GitHub (Optional)

1. **Initialize Git**:
   ```bash
   git init
   git add .
   git commit -m "DiscoverHealth Part I: Added 'Use My Location' Bonus Feature"
   ```

2. **Create `.gitignore`**:
   ```bash
   notepad .gitignore
   ```
   * Add:
     ```
     node_modules/
     frontend/dist/
     ```

3. **Push**:
   ```bash
   git remote add origin https://github.com/<your-username>/DiscoverHealthPartI.git
   git branch -M main
   git push -u origin main
   ```

## Step 7: Summary and Next Steps

You’ve completed DiscoverHealth Part I with the bonus "Use My Location" feature! Recap:
* Added a button and geolocation functionality to `SearchView.jsx`.
* Updated the report to document the bonus feature.
* Tested and integrated with the existing application.

**Next Steps**:
- Submit the final version by Friday, September 26, 2025, 16:00.
- Review feedback on or after October 16, 2025.