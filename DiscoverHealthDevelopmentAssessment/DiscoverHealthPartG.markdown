# Week 10: DiscoverHealth Part G – Extra Credit for React Implementation

## Introduction

In **DiscoverHealth Part G**, you'll finalize the project by ensuring the frontend is fully implemented in React, as per the extra credit option mentioned in the assessment brief. Since we chose to use React from Part B onward, this part focuses on summarizing how React was used to build the front-end for all tasks, providing extra credit. This includes component-based structure, state management, routing, and AJAX integration. You'll use **Windows Command Prompt** or **macOS Terminal**, building on Part F to achieve extra credit.

### What You’ll Learn

* **React Extra Credit**: Understand how using React enhances the front-end, allowing for reusable components, dynamic UI updates, and better code organization.
* **Project Summary**: Review how React was applied across parts B-F.
* **Final Polish**: Ensure all features are integrated in React without a non-React alternative.

### What You’ll Do

1. **Exercise A**: Set up `DiscoverHealthPartG` folder, copy Part F files, and verify dependencies.
2. **Exercise B**: Verify all frontend components are in React (`SearchView.jsx`, `AddResourceView.jsx`, `App.jsx`).
3. **Exercise C**: Summarize React usage in a report section for extra credit.
4. **Exercise D**: Run servers and test the full React frontend.
5. **Save Everything**: Organize work and (optionally) push to GitHub.

### Prerequisites

* Completed **DiscoverHealth Part F** with working reviews and authentication.
* **Node.js** installed (check: `node --version`, `npm --version`).
* Familiarity with command-line (`cd`, `mkdir`, `copy`/`cp`).
* Part F project folder at `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartF`.
* `discoverhealth.db` with all tables.

## Step 1: Understanding the Basics

### What is Part G?

**DiscoverHealth Part G** is the extra credit section for using React in the frontend. As per the brief, using React is more difficult but allows extra credit. Since we used React for Parts B-F, this part summarizes its application and ensures the project is complete.

### How Part G Builds on Part F

* **Part F**:
  * `server.js`: Reviews endpoint and authentication.
  * `SearchView.jsx`: Map with review forms, login/signup.
  * `AddResourceView.jsx`: Manual resource addition.
* **Part G**:
  - Verify React implementation across all frontend tasks.
  - Provide a report summary for extra credit.

### Key Concepts

* **React Advantages**: Component-based, state management (`useState`), effects (`useEffect`), routing (`react-router-dom`), and AJAX (`fetch`).
* **Extra Credit**: The brief notes React for extra credit; summarize how it was used.

## Step 2: Exercise A – Set Up the Project

1. **Open Command Prompt (Windows) or Terminal (macOS)**:
   * **Windows**: `Win + R`, `cmd`, Enter.
   * **macOS**: Open Terminal via Spotlight.

2. **Create Project Folder**:
   ```bash
   cd %userprofile%\Desktop\Solent University\FinalProject  # Windows
   cd ~/Desktop/Solent University/FinalProject             # macOS
   mkdir DiscoverHealthPartG
   cd DiscoverHealthPartG
   ```

3. **Copy Files from Part F**:
   * Copy `discoverhealth.db` and `server.js`:
     ```bash
     # Windows
     copy ..\DiscoverHealthPartF\discoverhealth.db .
     copy ..\DiscoverHealthPartF\server.js .

     # macOS
     cp ../DiscoverHealthPartF/discoverhealth.db .
     cp ../DiscoverHealthPartF/server.js .
     ```
   * Copy `frontend` folder:
     ```bash
     # Windows
     xcopy ..\DiscoverHealthPartF\frontend frontend /s /e /h /i

     # macOS
     cp -R ../DiscoverHealthPartF/frontend .
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

## Step 3: Exercise B – Verify Frontend in React

Verify all frontend components are in React for extra credit.

### B.1: Verify `App.jsx`

1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartG\frontend\src\App.jsx`

2. **Open**:
   ```bash
   cd frontend/src
   notepad App.jsx  # Windows
   nano App.jsx    # macOS
   ```

3. **Verify Content**:
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

5. **Code Explanation**:
   * **Routing**: Uses `react-router-dom` for client-side routing to Search and Add Resource pages.
   * **Function**: Provides navigation and renders components based on URL, demonstrating React's routing capabilities for extra credit.

### B.2: Verify `SearchView.jsx`

1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartG\frontend\src\SearchView.jsx`

2. **Open**:
   ```bash
   notepad SearchView.jsx  # Windows
   nano SearchView.jsx    # macOS
   ```

3. **Verify Content**:
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
2. **Check**: Matches Part F code.

### D.2: Verify `AddResourceView.jsx`
1. **Path**: `C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartF\frontend\src\AddResourceView.jsx`
2. **Check**: Matches Part F code.

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
   * **Authentication**:
     - See login form; click “Need an account? Sign Up” to show signup form.
     - Sign up with new credentials (e.g., username: “newuser”, password: “test”).
     - Expect “Signup successful! Please log in as newuser.” and switch to login form.
     - Log in with valid credentials (e.g., username: “jsmith”, password: “test”).
     - Expect “Logged in as jsmith” and logout button.
     - Reload page; expect “Logged in as jsmith” to persist.
     - Click logout; expect login form to reappear.
     - Test invalid credentials; expect error (e.g., “Invalid username or password”).
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
   * **Username Not Showing**: Check GET `/api/user` response, ensure `credentials: 'include'` in fetch.
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
   git commit -m "DiscoverHealth Part F: Reviews and Map Enhancements with Updated Authentication"
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