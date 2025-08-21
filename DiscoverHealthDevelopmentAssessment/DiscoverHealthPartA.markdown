# Week 6: DiscoverHealth Part A – Building a REST API with Node.js and SQLite

## Introduction

In this week’s project, we’re building **DiscoverHealth Part A**, a REST API to manage healthcare resources using **Node.js**, **Express**, and **SQLite**. You’ll create an API that allows clients to list healthcare resources by region, add new resources, and "like" (recommend) resources. We’ll use the **Command Prompt** (Windows) or **Terminal** (macOS) to set up the project and test it with tools like a browser, Postman, or RESTer. By the end, you’ll have a working server that handles JSON data and interacts with a SQLite database.

### What You’ll Learn

* **REST API**: A way for clients (like apps or browsers) to interact with a server using standard HTTP methods (GET, POST). You’ll build endpoints to retrieve and modify data.
* **Express**: A Node.js framework for creating web servers and APIs. It simplifies handling HTTP requests and responses.
* **SQLite**: A lightweight database that stores data in a single file (`discoverhealth.db`). You’ll create tables and use SQL queries to manage data.
* **better-sqlite3**: A Node.js library for working with SQLite databases. It allows synchronous database operations, which are simpler for beginners.
* **JSON**: The format used to send and receive data in your API. You’ll return JSON for resource lists and accept JSON for adding resources.
* **Testing APIs**: How to test your API using a browser (for GET requests) and tools like Postman or RESTer (for POST requests).

### What You’ll Do

1. **Exercise A**: Set up Node.js, create a project folder, and initialize a Node.js project with Express and better-sqlite3. Verify your environment is ready.
2. **Exercise B**: Create and seed a SQLite database (`discoverhealth.db`) with tables for healthcare resources, users, and reviews, plus sample data for testing.
3. **Exercise C**: Build a REST API with three endpoints:
   * GET `/api/resources?region=RegionName` to list resources in a region.
   * POST `/api/resources` to add a new resource.
   * POST `/api/resources/:id/recommend` to increment a resource’s recommendation count.
4. **Exercise D**: Test your API using a browser (for listing) and Postman/RESTer (for adding and liking resources).
5. **Save Everything**: Keep your work organized in a folder and (optionally) save it to GitHub to preserve it.

### Prerequisites

* You’ve completed previous weeks’ exercises, so you have experience with Node.js and have it installed. Check by typing `node --version` in Command Prompt/Terminal.
* You know basic command-line navigation (`cd`, `mkdir`, etc.) from earlier weeks.
* You understand basic JavaScript (variables, objects, functions) and have some familiarity with JSON.
* You have a basic understanding of HTTP methods (GET, POST) from prior lessons. If not, we’ll explain as we go.
* If Node.js isn’t installed, download it from [nodejs.org](https://nodejs.org) and install it before starting.

## Step 1: Understanding the Basics

Before we start coding, let’s clarify what we’re building and the tools we’re using. This is like reading the rules of a board game before playing.

### What is a REST API?

A **REST API** (Representational State Transfer Application Programming Interface) is a way for clients (like a web app or mobile app) to communicate with a server over HTTP. It uses standard HTTP methods:

* **GET**: Retrieve data (e.g., list resources in a region).
* **POST**: Send data to create or update something (e.g., add a resource or increment a like).
* The API we build will respond with **JSON** (JavaScript Object Notation), a format that’s easy for both humans and machines to read, like `{ "id": 1, "name": "St Thomas Hospital" }`.

Our API will have three endpoints, as specified:
* `GET /api/resources?region=RegionName`: Returns a JSON array of healthcare resources in the specified region.
* `POST /api/resources`: Accepts a JSON object with resource details and adds it to the database.
* `POST /api/resources/:id/recommend`: Increments the "recommendations" count for a resource by its ID.

### Database Schema

Our SQLite database (`discoverhealth.db`) will have three tables, as described in the assignment brief:

* **healthcare_resources**:
  * `id` (INTEGER, auto-incrementing primary key)
  * `name` (TEXT, e.g., “St Thomas Hospital”)
  * `category` (TEXT, e.g., “Hospital”)
  * `country` (TEXT, e.g., “UK”)
  * `region` (TEXT, e.g., “London”)
  * `lat` (REAL, latitude coordinate)
  * `lon` (REAL, longitude coordinate)
  * `description` (TEXT, e.g., “Major teaching hospital”)
  * `recommendations` (INTEGER, default 0, for like counts)
* **users**:
  * `id` (INTEGER, auto-incrementing primary key)
  * `username` (TEXT, unique, e.g., “admin”)
  * `password` (TEXT, e.g., “password”)
  * `isAdmin` (INTEGER, 0 or 1, default 0)
* **reviews**:
  * `id` (INTEGER, auto-incrementing primary key)
  * `resource_id` (INTEGER, foreign key to `healthcare_resources.id`)
  * `review` (TEXT, the review content)
  * `user_id` (INTEGER, foreign key to `users.id`)

The `reviews` table links to `healthcare_resources` and `users` with foreign keys, which enforce referential integrity (e.g., you can’t add a review for a non-existent resource).

### Tools Overview

* **Node.js**: The runtime that lets us run JavaScript on the server.
* **Express**: A framework for Node.js that simplifies creating APIs. It handles HTTP requests and routes them to our code.
* **better-sqlite3**: A Node.js library for SQLite. It’s synchronous (easier to use than async alternatives) and lets us run SQL queries to manage the database.
* **SQLite**: A lightweight database stored in a single file (`discoverhealth.db`). No need to install a separate database server.
* **Postman/RESTer**: Tools for testing API endpoints, especially POST requests, which are harder to test in a browser.

### Putting It Together

* We’ll set up a Node.js project with Express to create a web server.
* We’ll initialize a SQLite database with the required tables and some sample data (e.g., hospitals in London, Manchester, etc.).
* We’ll implement three API endpoints to handle the tasks.
* We’ll test everything to ensure it works as expected.
* Express will handle HTTP requests, better-sqlite3 will manage database operations, and JSON will be the data format for communication.

Now, let’s set up the project and start coding.

## Step 2: Exercise A – Set Up Your Environment

Let’s create a project folder, initialize a Node.js project, and install the necessary packages. This is like setting up your workbench before starting a project.

1. **Open Command Prompt (Windows) or Terminal (macOS)**:

   * **Windows**: Press `Win + R`, type `cmd`, and press Enter to open Command Prompt.
   * **macOS**: Search for “Terminal” in Spotlight or find it in Applications > Utilities.
   * Verify Node.js and npm are installed:

     ```bash
     node --version
     npm --version
     ```

     You should see version numbers (e.g., `v18.x.x` for Node.js). If not, download and install Node.js from [nodejs.org](https://nodejs.org), then reopen your command line.

2. **Create a Project Folder**:

   * Navigate to your Desktop (or preferred location):

     ```bash
     cd ~/Desktop  # macOS
     cd %userprofile%\Desktop  # Windows
     ```

   * Create and enter a new folder for the project:

     ```bash
     mkdir DiscoverHealthPartA
     cd DiscoverHealthPartA
     ```

     This creates a folder called `DiscoverHealthPartA` and moves you into it.

3. **Initialize a Node.js Project**:

   * Run:

     ```bash
     npm init -y
     ```

     This creates a `package.json` file with default settings, marking this folder as a Node.js project. You can check it with:

     ```bash
     dir  # Windows
     ls   # macOS
     ```

     You should see `package.json`.

4. **Install Dependencies**:

   * Install Express and better-sqlite3:

     ```bash
     npm install express better-sqlite3
     ```

     * This downloads and installs:
       * **Express**: For creating the API server.
       * **better-sqlite3**: For interacting with the SQLite database.
     * After installation, run `dir` (Windows) or `ls` (macOS) to see a new `node_modules` folder and `package-lock.json`. These contain the libraries and dependency details.

5. **Open the Project in an Editor (Optional)**:

   * You can use Notepad (Windows) or nano (macOS) via the command line for editing files, as we’ll show below. If you prefer a code editor like VS Code, open the `DiscoverHealthPartA` folder in it, but we’ll assume command-line editing for consistency.

Your environment is now ready! You have Node.js, Express, and better-sqlite3 installed, and a project folder set up.

## Step 3: Exercise B – Create and Seed the Database

We need to create the SQLite database (`discoverhealth.db`) with the required tables and populate it with sample data for testing. This ensures our API has data to work with.

### B.1: Create `init-db.js`

1. **Create the file**:

   * **Windows**:

     ```bash
     notepad init-db.js
     ```

     Notepad will ask if you want to create a new file; click Yes.

   * **macOS**:

     ```bash
     nano init-db.js
     ```

2. **Add the database initialization code**:

   * Copy and paste the following into `init-db.js`:

     ```javascript
     const Database = require('better-sqlite3');
     const path = require('path');

     // Determine the database file path in this folder
     const dbPath = path.join(__dirname, 'discoverhealth.db');
     const db = new Database(dbPath);

     // Turn on foreign key constraints
     db.exec('PRAGMA foreign_keys = ON;');

     // Create the tables if they do not exist
     db.exec(`
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
     `);

     // Insert sample resources for testing
     db.exec(`
       INSERT INTO healthcare_resources
         (name, category, country, region, lat, lon, description, recommendations)
       VALUES
         ('St Thomas Hospital', 'Hospital', 'UK', 'London', 51.4980, -0.1195,
          'Major teaching hospital on the Thames.', 5),
         ('Manchester Clinic', 'Clinic', 'UK', 'Manchester', 53.4808, -2.2426,
          'Community clinic in central Manchester.', 2),
         ('Southampton Health Centre', 'Health Centre', 'UK', 'Southampton', 50.9097, -1.4044,
          'General health services for families.', 0);
     `);

     // Insert sample users (useful for later parts)
     db.exec(`
       INSERT OR IGNORE INTO users (username, password, isAdmin)
       VALUES ('admin', 'password', 1), ('user', 'password', 0);
     `);

     console.log('Database created and seeded successfully at', dbPath);
     ```

   * **Save the file**:
     * **Windows**: File > Save in Notepad, then close.
     * **macOS**: Press `Ctrl+O`, Enter to save, then `Ctrl+X` to exit.

3. **What this code does**:

   * Imports `better-sqlite3` and `path` (to locate the database file).
   * Creates `discoverhealth.db` in the project folder (or opens it if it exists).
   * Enables foreign key constraints (`PRAGMA foreign_keys = ON`) for referential integrity.
   * Creates three tables: `healthcare_resources`, `users`, and `reviews`, matching the schema from the brief.
   * Inserts sample data:
     * Three healthcare resources (in London, Manchester, Southampton) for testing the GET endpoint.
     * Two users (`admin` and `user`) for potential future use (e.g., Part B).
   * Logs a success message with the database path.

### B.2: Run the Initialization Script

* Run the script to create and seed the database:

  ```bash
  node init-db.js
  ```

* You should see output like:

  ```
  Database created and seeded successfully at .../DiscoverHealthPartA/discoverhealth.db
  ```

* Check that the database file was created:

  ```bash
  dir  # Windows
  ls   # macOS
  ```

  You should see `discoverhealth.db` in the folder. This file now contains the tables and sample data.

### B.3: Why This Matters

* The database is the backbone of our API, storing all healthcare resources.
* The sample data ensures we can test the GET endpoint immediately (e.g., querying resources in “London”).
* Foreign keys in the `reviews` table ensure data consistency (though we won’t use reviews in Part A).
* Running this script only once sets up the database. If you run it again, it won’t duplicate tables (due to `IF NOT EXISTS`) or users (due to `INSERT OR IGNORE`), but it will add duplicate resources unless we clear the table first (not needed for this exercise).

Now our database is ready with sample data to test our API.

## Step 4: Exercise C – Build the REST API

We’ll create a `server.js` file to implement the Express server with the three required API endpoints. This is where we define how the server responds to client requests.

### C.1: Create `server.js`

1. **Create the file**:

   * **Windows**:

     ```bash
     notepad server.js
     ```

     Click Yes to create the new file.

   * **macOS**:

     ```bash
     nano server.js
     ```

2. **Add the server code**:

   * Copy and paste the following into `server.js`:

     ```javascript
     const express = require('express');
     const Database = require('better-sqlite3');
     const path = require('path');
     const fs = require('fs');

     const app = express();

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

3. **Understanding the code**:

   * **Imports**: Loads Express, better-sqlite3, `path` (for file paths), and `fs` (to check if the database exists).
   * **Database Setup**: Opens `discoverhealth.db` and logs its path and whether it exists. Lists all tables for debugging.
   * **Middleware**: `app.use(express.json())` parses incoming JSON request bodies (needed for POST requests).
   * **Root Route**: `GET /` returns a friendly message suggesting how to use the API.
   * **Endpoint 1: GET /api/resources**:
     * Reads the `region` query parameter (e.g., `?region=London`).
     * If no region is provided, returns a 400 error with a JSON error message.
     * Queries the database for resources matching the region and returns them as JSON.
   * **Endpoint 2: POST /api/resources**:
     * Expects a JSON body with `name`, `category`, `country`, `region`, `lat`, `lon`, and `description`.
     * Inserts a new row into `healthcare_resources` with `recommendations` set to 0.
     * Returns a 201 status with the new resource’s `id`.
   * **Endpoint 3: POST /api/resources/:id/recommend**:
     * Takes an `id` from the URL path (e.g., `/api/resources/4/recommend`).
     * Increments the `recommendations` field for that resource.
     * If the `id` doesn’t exist, returns a 404 error.
     * Otherwise, returns a success message.
   * **Server Start**: Listens on port 3000 and logs the URL.

### C.2: Run the Server

* Start the server:

  ```bash
  node server.js
  ```

* You should see:

  ```
  [BOOT] Using DB at: .../DiscoverHealthPartA/discoverhealth.db (FOUND)
  [BOOT] Tables in DB: [ 'healthcare_resources', 'users', 'reviews' ]
  Server running at http://localhost:3000
  ```

* If you see “(MISSING)” or table errors, ensure you ran `node init-db.js` and that `discoverhealth.db` is in the folder.

The server is now running, ready to handle API requests.

## Step 5: Exercise D – Test Your API

Now we’ll test the three endpoints to ensure they work as expected. We’ll use a browser for the GET request and Postman or RESTer for POST requests.

### D.1: Test Task 1 – Listing Resources by Region

1. **Browser Test**:

   * Open a browser and navigate to:

     ```
     http://localhost:3000/api/resources?region=London
     ```

   * You should see a JSON response like:

     ```json
     [
       {
         "id": 1,
         "name": "St Thomas Hospital",
         "category": "Hospital",
         "country": "UK",
         "region": "London",
         "lat": 51.498,
         "lon": -0.1195,
         "description": "Major teaching hospital on the Thames.",
         "recommendations": 5
       }
     ]
     ```

   * Try other regions like `?region=Manchester` or `?region=Southampton` to see their resources. If you omit `region` (e.g., `http://localhost:3000/api/resources`), you should get:

     ```json
     { "error": "Region query parameter is required" }
     ```

2. **Postman Test**:

   * Open Postman and create a new GET request.
   * URL: `http://localhost:3000/api/resources`
   * In the **Params** tab, add:
     * Key: `region`
     * Value: `London`
   * Click Send. You should see the same JSON response as above.

3. **RESTer Test**:

   * In the RESTer browser extension, create a new request:
     * Method: GET
     * URL: `http://localhost:3000/api/resources?region=London`
   * Click Send. The JSON response appears.

### D.2: Test Task 2 – Adding a Resource

1. **Postman Test**:

   * Create a new POST request in Postman.
   * URL: `http://localhost:3000/api/resources`
   * Headers:
     * Key: `Content-Type`
     * Value: `application/json`
   * Body tab > raw > JSON, enter:

     ```json
     {
       "name": "New Clinic",
       "category": "Clinic",
       "country": "UK",
       "region": "Liverpool",
       "lat": 53.41,
       "lon": -2.98,
       "description": "A new clinic in Liverpool"
     }
     ```

   * Click Send. You should get:

     ```json
     { "id": 4 }
     ```

   * To verify, send a GET request to `http://localhost:3000/api/resources?region=Liverpool`. You should see the new resource in the JSON response.

2. **RESTer Test**:

   * Method: POST
   * URL: `http://localhost:3000/api/resources`
   * Headers:
     * Name: `Content-Type`
     * Value: `application/json`
   * Body (raw JSON): Paste the same JSON as above.
   * Click Send. You should see the new ID.
   * Verify with a GET request as above.

### D.3: Test Task 3 – Liking a Resource

1. **Postman Test**:

   * Create a new POST request.
   * URL: `http://localhost:3000/api/resources/4/recommend` (use the ID from the previous step, e.g., 4).
   * No body or headers needed (though `Content-Type: application/json` is fine).
   * Click Send. You should get:

     ```json
     { "message": "Recommendation added" }
     ```

   * Verify by sending a GET request to `http://localhost:3000/api/resources?region=Liverpool`. The `recommendations` field for the new clinic should now be 1 (or higher if you “liked” multiple times).

2. **RESTer Test**:

   * Method: POST
   * URL: `http://localhost:3000/api/resources/4/recommend`
   * Click Send. You should see the success message.
   * Verify as above.

### D.4: Troubleshooting

* **Cannot GET /**: This is expected; the root route (`http://localhost:3000/`) just shows a hint message.
* **Failed to retrieve resources**: Check that `discoverhealth.db` exists and was initialized with `node init-db.js`.
* **404 Resource not found**: Ensure the ID exists when recommending (check IDs via GET requests).
* **500 errors**: Look at the server logs in the terminal for database errors. Ensure the database schema matches the queries.

## Step 6: (Optional) Save Everything to GitHub

To preserve your work, you can initialize a Git repository and push to GitHub. This is optional but recommended.

1. **Initialize a Git repository**:

   ```bash
   git init
   git add .
   git commit -m "DiscoverHealth Part A: REST API with SQLite"
   ```

2. **Create a `.gitignore` file**:

   ```bash
   notepad .gitignore  # Windows
   nano .gitignore     # macOS
   ```

   Add:

   ```
   node_modules/
   ```

   Save and close. This ignores the large `node_modules` folder.

3. **Push to GitHub**:

   * Create a new repository on GitHub named `DiscoverHealthPartA`.
   * Link and push:

     ```bash
     git remote add origin https://github.com/<your-username>/DiscoverHealthPartA.git
     git branch -M main
     git push -u origin main
     ```

     Replace `<your-username>` with your GitHub username.

Your code is now safely stored on GitHub.

## Step 7: Summary and Next Steps

Congratulations on completing DiscoverHealth Part A! 🎉 You’ve built a fully functional REST API. Let’s recap:

* **Set up a Node.js project**: You created a project with Express and better-sqlite3.
* **Created a SQLite database**: You initialized `discoverhealth.db` with tables and sample data.
* **Built a REST API**: You implemented three endpoints:
  * GET `/api/resources?region=RegionName` to list resources.
  * POST `/api/resources` to add a new resource.
  * POST `/api/resources/:id/recommend` to increment likes.
* **Tested the API**: You verified functionality using a browser, Postman, or RESTer.
* **Learned key concepts**: Express routing, SQLite queries, JSON handling, and API testing.

**Next Steps**:

* Experiment with the API: Try adding more resources or testing different regions.
* Add validation: Enhance `POST /api/resources` to check for missing fields or invalid lat/lon values.
* Prepare for Part B: The users and reviews tables suggest future tasks like authentication or adding reviews. Your database is ready for those.
* Check the terminal and browser console (F12) for errors if something doesn’t work.
* Explore Express middleware or SQLite features for more advanced functionality.

You now have a solid foundation in building REST APIs with Node.js and SQLite. This is a critical skill for web development, and you’re ready to tackle more complex backend tasks. Great work, and see you in the next part of DiscoverHealth!