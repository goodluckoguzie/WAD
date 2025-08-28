# Week 6: DiscoverHealth Part C – Adding Error-Checking and Validation

## Introduction

In **DiscoverHealth Part C**, you'll enhance the application from Part B by adding simple error-checking and validation. Specifically, you'll validate input data in the backend API for adding new healthcare resources (Task 2 from Part A) and display appropriate error messages in the frontend (Task 5 from Part B). This ensures the system handles invalid or missing data gracefully, returning HTTP errors from the server and showing user-friendly messages in the React interface. You'll use the **Windows Command Prompt** or **macOS Terminal** to set up the project, building on Part B.

### What You’ll Learn

* **Backend Validation**: Check for missing or invalid data in API requests and return appropriate HTTP status codes and error messages.
* **Error Handling in Frontend**: Capture and display server-side errors in the React UI using state management.
* **HTTP Status Codes**: Use codes like 400 (Bad Request) for validation failures.
* **Improved User Experience**: Provide feedback to users on invalid inputs without crashing the app.

### What You’ll Do

1. **Exercise A**: Set up a new project folder for Part C, copy necessary files from Part B (`discoverhealth.db`, `server.js`, and the `frontend` folder), and verify dependencies.
2. **Exercise B**: Modify the Express server to add validation in the POST `/api/resources` endpoint, checking for missing fields and sending HTTP 400 errors.
3. **Exercise C**: Update the React frontend (`AddResourceView.jsx`) to handle and display server error messages.
4. **Exercise D**: Run both the Express server and React dev server, then test the validation in the browser and (optionally) with Postman/RESTer.
5. **Save Everything**: Organize your work and (optionally) push it to GitHub.

### Prerequisites

* You completed **DiscoverHealth Part B**, with a working React front-end, Express server, and `discoverhealth.db`.
* **Node.js** installed (check with `node --version` and `npm --version`).
* Familiarity with command-line commands (`cd`, `mkdir`, `copy`/`cp`) from previous parts.
* The Part B project folder is available for copying files.
* If Node.js isn’t installed, download it from [nodejs.org](https://nodejs.org).

## Step 1: Understanding the Basics

Before coding, let's understand the enhancements for Part C.

### What is Part C?

**DiscoverHealth Part C** focuses on error-checking:
* **Backend (Task 7, Part A)**: Validate the POST data for adding resources (e.g., check if `name`, `category`, etc., are provided). If invalid, return a 400 Bad Request with an error message.
* **Frontend (Task 7, Part B)**: Display these server errors in the add-resource form, improving user feedback.

This builds on Part B's React app and API, ensuring robustness against invalid inputs.

### How Part C Builds on Part B

* **Part B's POST Endpoint**: Accepts resource data but doesn't validate for missing fields.
* **Part C Enhancements**:
  * Backend: Add checks in `server.js` for required fields.
  * Frontend: In `AddResourceView.jsx`, expand error handling to show server-specific messages.
* No new components needed; just modifications to existing code.

### Key Concepts

* **Validation**: In Express, use `if` statements to check `req.body` fields.
* **HTTP Errors**: Use `res.status(400).json({ error: 'Message' })` for bad requests.
* **Frontend Error Display**: Use React state to show server errors in the UI.

### Putting It Together

* Copy Part B as the base.
* Add backend validation.
* Enhance frontend to handle errors.
* Test with invalid data to verify messages.

## Step 2: Exercise A – Set Up the Project

Create a new folder for Part C and copy from Part B.

1. **Open Command Prompt (Windows) or Terminal (macOS)**.

2. **Create a Project Folder**:
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
   * Copy the entire `frontend` folder:
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

5. **Install Frontend Dependencies** (in `frontend`):
   ```bash
   cd frontend
   npm install
   npm install react-router-dom
   ```
   * Verify `package.json` has `react-router-dom`.

6. **Verify Setup**:
   * Check files with `dir` or `ls`.
   * Ensure `frontend/src` has `App.jsx`, `SearchView.jsx`, `AddResourceView.jsx`.

## Step 3: Exercise B – Add Validation to the Express Server

Modify `server.js` to validate POST data for adding resources.

### B.1: Update `server.js`

1. **Open `server.js`**:
   ```bash
   notepad server.js  # Windows
   nano server.js    # macOS
   ```

2. **Modify the POST `/api/resources` Endpoint**:
   * Update to check for missing fields:
     ```javascript
     app.post('/api/resources', (req, res) => {
       const { name, category, country, region, lat, lon, description } = req.body;
       if (!name || !category || !country || !region || lat === undefined || lon === undefined || !description) {
         return res.status(400).json({ error: 'All fields are required: name, category, country, region, lat, lon, description' });
       }
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
     ```
   * This checks if any required field is missing or undefined, returning 400 with a message.

3. **Save the file**.

4. **Why This Matters**:
   * Prevents invalid data from being inserted into the database.
   * Returns meaningful errors for the frontend to display.

## Step 4: Exercise C – Update the React Frontend for Error Display

Modify `AddResourceView.jsx` to handle server errors.

### C.1: Update `src/AddResourceView.jsx`

1. **Open the file**:
   ```bash
   cd frontend/src
   notepad AddResourceView.jsx  # Windows
   nano AddResourceView.jsx    # macOS
   ```

2. **Enhance Error Handling**:
   * The code already handles !res.ok with `setError(data.error || 'Failed to add resource.')`. No major changes needed, but ensure it's present:
     ```jsx
     // Inside handleSubmit, after fetch:
     const data = await res.json();
     if (res.ok) {
       setMessage(`Resource added with ID ${data.id}`);
       setForm({ name: '', category: '', country: '', region: '', lat: '', lon: '', description: '' });
     } else {
       setError(data.error || 'Failed to add resource.');
     }
     ```
   * This displays the server's error message (e.g., "All fields are required...").

3. **Save the file**.

4. **Why This Matters**:
   * Users see specific feedback (e.g., "All fields are required") instead of generic errors.

## Step 5: Exercise D – Run and Test the Application

1. **Start the Express Server**:
   ```bash
   cd C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartC
   node server.js
   ```

2. **Start the React Dev Server** (new terminal):
   ```bash
   cd C:\Users\Nneka\Desktop\Solent University\FinalProject\DiscoverHealthPartC\frontend
   npm run dev
   ```

3. **Test Validation**:
   * Go to `http://localhost:5173/add`.
   * Submit the form with missing fields (e.g., no name).
   * Expect a red error message like "All fields are required...".
   * Test with Postman: POST to `http://localhost:3000/api/resources` with missing fields. Expect 400 and JSON error.

## Step 6: (Optional) Save Everything to GitHub

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
   * Create GitHub repository `DiscoverHealthPartC`.
   * Run:
     ```bash
     git remote add origin https://github.com/<your-username>/DiscoverHealthPartC.git
     git branch -M main
     git push -u origin main
     ```

## Step 7: Summary and Next Steps

You've added validation to DiscoverHealth! Recap:
* **Set up**: Copied from Part B.
* **Backend**: Added checks in `server.js` for missing fields.
* **Frontend**: Handled server errors in `AddResourceView.jsx`.
* **Tested**: Verified error messages for invalid inputs.

**Next Steps**:
* Proceed to Part D (adding a map) or further parts.
* Enhance validation (e.g., check data types like lat/lon as numbers).
* Debug with browser console (F12) and server logs.