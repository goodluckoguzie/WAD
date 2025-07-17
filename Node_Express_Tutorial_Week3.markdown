# Week 3: Building a Front-End for HitTastic! with Promises and AJAX

This guide extends your Week 2 music API by adding a front-end for "HitTastic!", a website for searching and adding music. You’ll use promises and AJAX with JSON to create an interactive user experience, connecting to your Week 2 REST API. We’ll use the Windows Command Prompt in a new `WAD/week3` folder, explain each step and change clearly, and provide full scripts with beginner-friendly comments. You’ll test the front-end in a browser, handle errors, and save your work on GitHub.

## What You’ll Do
- **Learn Promises and AJAX**: Use `fetch()` with `async/await` to fetch data asynchronously without reloading the page.
- **Build a Front-End**: Create HTML and JavaScript for searching music by artist, title, or year, and adding new songs.
- **Enhance the Server**: Serve static files and add a year-based search endpoint.
- **Handle Errors**: Display user-friendly messages for invalid inputs or network issues.
- **Display Results in a Table**: Show search results in a structured table with a "CLASSIC HIT!" label for songs before 2000.
- **Add Search Type Selector**: Allow users to choose between searching by title, artist, or year.
- **Test and Deploy**: Verify functionality in a browser and save to GitHub.

**Prerequisites**: You completed Week 2, with a `WAD/week2` folder containing `app.mjs`, `setupDB.mjs`, `package.json`, and a GitHub repository (`WAD-week2`).

## Step 1: Set Up the Week 3 Folder
Create a new folder, `WAD/week3`, to keep your work organized.

1. **Open Command Prompt**:
   - Press `Windows Key + R`, type `cmd`, and press Enter.
   - This opens a black window for typing commands, like giving instructions to your computer.

2. **Navigate to Desktop**:
   - Type and press Enter:
     ```
     cd %userprofile%\Desktop
     ```
   - Type `dir` to see files; you should see the `WAD` folder from Week 2.

3. **Move to WAD Folder**:
   - Type and press Enter:
     ```
     cd WAD
     ```
   - If `WAD` doesn’t exist, create it:
     ```
     mkdir WAD
     cd WAD
     ```

4. **Create week3 Folder**:
   - Type and press Enter:
     ```
     mkdir week3
     ```

5. **Move to week3**:
   - Type and press Enter:
     ```
     cd week3
     ```

**Why this matters**: A separate folder keeps Week 3 work distinct, like starting a new chapter in a notebook.

**Check**: Type `dir` and press Enter. You should see no files yet.

## Step 2: Copy Week 2 Files
Copy `app.mjs`, `setupDB.mjs`, and `package.json` from `WAD/week2` to reuse your server and database setup.

1. **Copy Files**:
   - In `WAD/week3`, type and press Enter for each:
     ```
     copy ..\week2\app.mjs app.mjs
     copy ..\week2\setupDB.mjs setupDB.mjs
     copy ..\week2\package.json package.json
     ```
   - The `..\week2\` means “go up one folder to WAD, then into week2.”

2. **Check Files**:
   - Type `dir` and press Enter. You should see `app.mjs`, `setupDB.mjs`, and `package.json`.

**Why this matters**: These files contain your Week 2 server, database setup, and project settings, forming the backend for your front-end.

## Step 3: Install Dependencies
Ensure Express and `better-sqlite3` are installed for your server.

1. **Install Express**:
   - Type and press Enter:
     ```
     npm install express
     ```
   - This creates `node_modules` and `package-lock.json`.

2. **Install better-sqlite3**:
   - Type and press Enter:
     ```
     npm install better-sqlite3
     ```

3. **Verify**:
   - Type `dir` and press Enter. Confirm `node_modules` and `package-lock.json` exist.

**Why this matters**: These tools run your server and connect to the database, like ingredients for a recipe.

## Step 4: Install PM2
PM2 keeps your server running and restarts it if you edit files.

1. **Install PM2 Globally**:
   - Type and press Enter:
     ```
     npm install -g pm2
     ```

2. **Check PM2**:
   - Type and press Enter:
     ```
     pm2 --version
     ```
   - Expect a version number (e.g., `5.2.0`).

**Why this matters**: PM2 simplifies server management, unlike manually restarting with `node app.mjs`.

## Step 5: Verify the Database (setupDB.mjs)
The Week 2 `setupDB.mjs` includes `students`, `songs`, and `orders` tables with sample data. Let’s verify it’s correct.

1. **Open setupDB.mjs**:
   - Type and press Enter:
     ```
     notepad setupDB.mjs
     ```

2. **Verify Content**:
   - Ensure it matches this code, which sets up:
     - **Students Table**: Stores student data (unchanged).
     - **Songs Table**: Includes `price` and `quantity_in_stock`.
     - **Orders Table**: Tracks song purchases.
     - **Sample Data**: Songs with prices and stock for testing.

   ```javascript
   import Database from 'better-sqlite3';

   const db = new Database('mydatabase.db');

   // Create students table
   const stmtStudents = db.prepare(`
       CREATE TABLE IF NOT EXISTS students (
           id INTEGER PRIMARY KEY AUTOINCREMENT,
           firstname TEXT,
           lastname TEXT,
           course TEXT
       )
   `);
   stmtStudents.run();

   // Add sample students
   const insertStudent = db.prepare(`
       INSERT INTO students (firstname, lastname, course) VALUES (?, ?, ?)
   `);
   insertStudent.run('Alice', 'Smith', 'Math');
   insertStudent.run('Bob', 'Jones', 'Science');

   // Create songs table with price and quantity_in_stock
   const stmtSongs = db.prepare(`
       CREATE TABLE IF NOT EXISTS songs (
           id INTEGER PRIMARY KEY AUTOINCREMENT,
           artist TEXT,
           title TEXT,
           year INTEGER,
           price REAL,
           quantity_in_stock INTEGER
       )
   `);
   stmtSongs.run();

   // Add sample songs
   const insertSong = db.prepare(`
       INSERT INTO songs (artist, title, year, price, quantity_in_stock) VALUES (?, ?, ?, ?, ?)
   `);
   insertSong.run('The Beatles', 'Hey Jude', 1968, 0.99, 10);
   insertSong.run('Queen', 'Bohemian Rhapsody', 1975, 1.29, 5);
   insertSong.run('The Beatles', 'Let It Be', 1970, 0.89, 8);

   // Create orders table
   const stmtOrders = db.prepare(`
       CREATE TABLE IF NOT EXISTS orders (
           id INTEGER PRIMARY KEY AUTOINCREMENT,
           song_id INTEGER,
           quantity INTEGER,
           FOREIGN KEY (song_id) REFERENCES songs(id)
       )
   `);
   stmtOrders.run();

   console.log('Database setup complete!');
   db.close();
   ```

3. **Save and Close Notepad**.

4. **Run the Script**:
   - Type and press Enter:
     ```
     node setupDB.mjs
     ```
   - Expect: `Database setup complete!`

**Why this matters**: The database supports all Week 2 endpoints and the new year-based search, ensuring your front-end can fetch data.

**Check**: Type `dir` to confirm `mydatabase.db` exists.

## Step 6: Create the Public Folder
Create a `public` folder to store front-end files (HTML, JavaScript, images).

1. **Create public Folder**:
   - In `WAD/week3`, type and press Enter:
     ```
     mkdir public
     ```

2. **Move to public**:
   - Type and press Enter:
     ```
     cd public
     ```

**Why this matters**: The `public` folder holds static files (like web pages) that your server will serve to users.

## Step 7: Create index.html
This is the main page for "HitTastic!", allowing users to search for music by artist, title, or year.

1. **Create index.html**:
   - Type and press Enter:
     ```
     notepad index.html
     ```

2. **Add Content**:
   - Copy and paste:
     ```html
     <!DOCTYPE html>
     <html lang="en">
     <head>
         <meta charset="UTF-8">
         <meta name="viewport" content="width=device-width, initial-scale=1.0">
         <title>HitTastic!</title>
         <script type="module" src="index.mjs"></script>
         <style>
             body Misses {
                 font-family: 'Berlin Sans FB', Helvetica, Arial, sans-serif;
                 background-color: #c0c0ff;
                 color: black;
                 padding: 20px;
             }
             h1 {
                 color: #333;
             }
             input, select, button {
                 margin: 5px;
                 padding: 5px;
             }
             table {
                 width: 50%;
                 border-collapse: collapse;
                 background-color: #e0e0ff;
                 border: 1px solid black;
                 margin-top: 20px;
             }
             th, td {
                 border: 1px solid #ccc;
                 padding: 8px;
                 text-align: left;
             }
             th {
                 background-color: #f2f2f2;
             }
         </style>
     </head>
     <body>
         <p><img src="images/hittastic.png" alt="HitTastic logo"></p>
         <h1>Welcome to HitTastic!</h1>
         <p>Search and shop for your favourite top 40 hits on HitTastic! Whether it's pop, rock, rap or pure liquid cheese you're into, you can be sure to find it on HitTastic! With the full range of top 40 hits since 1952 on our database, you can guarantee you'll find what you're looking for in stock. Plus, with our Year Search find out exactly what was in the chart in any year.</p>
         <div>
             Artist: <input id="theArtist">
             <button id="search">Search!</button>
         </div>
         <div>
             Search by:
             <select id="searchType">
                 <option value="artist">Artist</option>
                 <option value="title">Title</option>
                 <option value="year">Year</option>
             </select>
             <input id="searchTerm" placeholder="Enter search term">
             <button id="searchByType">Search by Type</button>
         </div>
         <table id="ht_results"></table>
     </body>
     </html>
     ```

3. **Save and Close Notepad**.

4. **Download the Logo**:
   - Visit [https://nwcourses.github.io/images/hittastic.png](https://nwcourses.github.io/images/hittastic.png).
   - Save the image as `hittastic.png` in `WAD/week3/public/images` (create the `images` folder first):
     ```
     mkdir images
     ```
   - Manually download and place the image in `public/images`.

**Why this matters**: This HTML page provides the user interface for searching music, with both an artist-specific search and a flexible search type selector. The table element ensures results are displayed in a structured format.

## Step 8: Create index.mjs
This JavaScript file uses AJAX with promises to fetch and display search results in a table.

1. **Create index.mjs**:
   - In `public`, type and press Enter:
     ```
     notepad index.mjs
     ```

2. **Add Content**:
   - Copy and paste:
     ```javascript
     // Handle artist search button click
     document.getElementById('search').addEventListener('click', async () => {
         const artist = document.getElementById('theArtist').value;
         try {
             const response = await fetch(`/songs/artist/${encodeURIComponent(artist)}`);
             if (!response.ok) {
                 throw new Error(`HTTP error! Status: ${response.status}`);
             }
             const songs = await response.json();
             displayResults(songs);
         } catch (error) {
             console.error('Error fetching data:', error);
             document.getElementById('ht_results').innerHTML = '<tr><td colspan="6">Error fetching results. Please try again.</td></tr>';
         }
     });

     // Handle search by type button click
     document.getElementById('searchByType').addEventListener('click', async () => {
         const searchType = document.getElementById('searchType').value;
         const searchTerm = document.getElementById('searchTerm').value;
         let url;
         if (searchType === 'artist') {
             url = `/songs/artist/${encodeURIComponent(searchTerm)}`;
         } else if (searchType === 'title') {
             url = `/songs/title/${encodeURIComponent(searchTerm)}`;
         } else if (searchType === 'year') {
             url = `/songs/year/${encodeURIComponent(searchTerm)}`;
         }
         try {
             const response = await fetch(url);
             if (!response.ok) {
                 throw new Error(`HTTP error! Status: ${response.status}`);
             }
             const songs = await response.json();
             displayResults(songs);
         } catch (error) {
             console.error('Error fetching data:', error);
             document.getElementById('ht_results').innerHTML = '<tr><td colspan="6">Error fetching results. Please try again.</td></tr>';
         }
     });

     // Display results in a table
     function displayResults(songs) {
         const table = document.getElementById('ht_results');
         table.innerHTML = ''; // Clear existing content
         // Create table header
         const headerRow = document.createElement('tr');
         ['Title', 'Artist', 'Year', 'Price', 'Stock', 'Status'].forEach(headerText => {
             const th = document.createElement('th');
             th.textContent = headerText;
             headerRow.appendChild(th);
         });
         table.appendChild(headerRow);
         // Create table rows
         songs.forEach(song => {
             const row = document.createElement('tr');
             const titleCell = document.createElement('td');
             titleCell.textContent = song.title;
             row.appendChild(titleCell);
             const artistCell = document.createElement('td');
             artistCell.textContent = song.artist;
             row.appendChild(artistCell);
             const yearCell = document.createElement('td');
             yearCell.textContent = song.year;
             row.appendChild(yearCell);
             const priceCell = document.createElement('td');
             priceCell.textContent = `$${song.price}`;
             row.appendChild(priceCell);
             const stockCell = document.createElement('td');
             stockCell.textContent = song.quantity_in_stock;
             row.appendChild(stockCell);
             const classicHitCell = document.createElement('td');
             classicHitCell.textContent = song.year < 2000 ? 'CLASSIC HIT!' : '';
             row.appendChild(classicHitCell);
             table.appendChild(row);
         });
     }
     ```

3. **Save and Close Notepad**.

**Why this matters**: This script uses `fetch()` with `async/await` to send AJAX requests to your server, fetching songs by artist, title, or year. It displays results in a table with a "CLASSIC HIT!" label for pre-2000 songs, using DOM methods for better performance.

## Step 9: Create addSong.html
This page lets users add new songs via a form.

1. **Create addSong.html**:
   - In `public`, type and press Enter:
     ```
     notepad addSong.html
     ```

2. **Add Content**:
   - Copy and paste:
     ```html
     <!DOCTYPE html>
     <html lang="en">
     <head>
         <meta charset="UTF-8">
         <meta name="viewport" content="width=device-width, initial-scale=1.0">
         <title>Add New Song - HitTastic!</title>
         <script type="module" src="addSong.mjs"></script>
         <style>
             body {
                 font-family: 'Berlin Sans FB', Helvetica, Arial, sans-serif;
                 background-color: #c0c0ff;
                 color: black;
                 padding: 20px;
             }
             h1 {
                 color: #333;
             }
             input, button {
                 margin: 5px;
                 padding: 5px;
             }
             #addSongResult {
                 margin-top: 10px;
                 color: red;
             }
         </style>
     </head>
     <body>
         <h1>Add New Song</h1>
         <form id="addSongForm">
             <label for="title">Title:</label>
             <input type="text" id="title" required><br>
             <label for="artist">Artist:</label>
             <input type="text" id="artist" required><br>
             <label for="year">Year:</label>
             <input type="number" id="year" required><br>
             <label for="price">Price:</label>
             <input type="number" step="0.01" id="price" required><br>
             <label for="stock">Quantity in Stock:</label>
             <input type="number" id="stock" required><br>
             <button type="submit">Add Song</button>
         </form>
         <div id="addSongResult"></div>
     </body>
     </html>
     ```

3. **Save and Close Notepad**.

**Why this matters**: This form lets users input song details, with `required` attributes ensuring fields aren’t blank before submission.

## Step 10: Create addSong.mjs
This script sends an AJAX POST request to add a song, handling errors for blank fields.

1. **Create addSong.mjs**:
   - In `public`, type and press Enter:
     ```
     notepad addSong.mjs
     ```

2. **Add Content**:
   - Copy and paste:
     ```javascript
     document.getElementById('addSongForm').addEventListener('submit', async (event) => {
         event.preventDefault(); // Prevent page reload
         const title = document.getElementById('title').value;
         const artist = document.getElementById('artist').value;
         const year = document.getElementById('year').value;
         const price = document.getElementById('price').value;
         const stock = document.getElementById('stock').value;

         // Validate inputs client-side
         if (!title || !artist || !year || !price || !stock) {
             document.getElementById('addSongResult').innerHTML = 'Please fill all fields.';
             return;
         }

         const songData = {
             title,
             artist,
             year: parseInt(year),
             price: parseFloat(price),
             quantity_in_stock: parseInt(stock)
         };

         try {
             const response = await fetch('/songs', {
                 method: 'POST',
                 headers: {
                     'Content-Type': 'application/json'
                 },
                 body: JSON.stringify(songData)
             });

             if (response.status === 400) {
                 const errorData = await response.json();
                 document.getElementById('addSongResult').innerHTML = `Error: ${errorData.error}`;
                 return;
             }
             if (!response.ok) {
                 throw new Error(`HTTP error! Status: ${response.status}`);
             }
             const data = await response.json();
             document.getElementById('addSongResult').innerHTML = `Song added with ID: ${data.id}`;
         } catch (error) {
             console.error('Error:', error);
             document.getElementById('addSongResult').innerHTML = 'An error occurred while adding the song.';
         }
     });
     ```

3. **Save and Close Notepad**.

**Why this matters**: This script uses AJAX to send a POST request, validates inputs, and provides clear feedback for success or errors.

## Step 11: Update app.mjs
Modify `app.mjs` to serve static files, add a year search endpoint, and enhance the POST `/songs` route with validation.

1. **Open app.mjs**:
   - In `WAD/week3`, type and press Enter:
     ```
     notepad app.mjs
     ```

2. **Changes from Week 2**:
   - **Add Static File Serving**: Insert `app.use(express.static('public'))` after `app.use(express.json())`.
   - **Add Year Endpoint**: Add `GET /songs/year/:year` to fetch songs by year.
   - **Enhance POST /songs**: Add validation for blank fields.
   - **Keep Existing Routes**: Retain all Week 2 routes.

3. **Full Updated Code**:
   ```javascript
import express from 'express';
import Database from 'better-sqlite3';

const app = express();
const db = new Database('mydatabase.db');

// Allow JSON data in POST and PUT requests
app.use(express.json());

// Serve static files from public folder
app.use(express.static('public'));

// Root route
app.get('/', (req, res) => {
    res.send('Hello World from Express!');
});

// Time route
app.get('/time', (req, res) => {
    res.send(`There have been ${Date.now()} milliseconds since 1/1/70.`);
});

// Greet route
app.get('/greet/:name', (req, res) => {
    res.send(`Hello ${req.params.name}!`);
});

// Get all students
app.get('/students', (req, res) => {
    try {
        const stmt = db.prepare('SELECT * FROM students');
        const results = stmt.all();
        res.json(results);
    } catch (error) {
        console.log(error);
        res.status(500).json({ error: 'Something went wrong!' });
    }
});

// Get students by last name
app.get('/students/:lastname', (req, res) => {
    try {
        const stmt = db.prepare('SELECT * FROM students WHERE lastname = ?');
        const results = stmt.all(req.params.lastname);
        res.json(results);
    } catch (error) {
        console.log(error);
        res.status(500).json({ error: 'Something went wrong!' });
    }
});

// Add a new student
app.post('/student/create', (req, res) => {
    try {
        const { firstname, lastname, course } = req.body;
        if (!firstname || !lastname || !course) {
            return res.status(400).json({ error: 'All fields are required' });
        }
        const stmt = db.prepare('INSERT INTO students (firstname, lastname, course) VALUES (?, ?, ?)');
        const info = stmt.run(firstname, lastname, course);
        res.json({ id: info.lastInsertRowid });
    } catch (error) {
        console.log(error);
        res.status(500).json({ error: 'Something went wrong!' });
    }
});

// Get songs by artist
app.get('/songs/artist/:artist', (req, res) => {
    try {
        const stmt = db.prepare('SELECT * FROM songs WHERE artist = ?');
        const results = stmt.all(req.params.artist);
        res.json(results);
    } catch (error) {
        console.log(error);
        res.status(500).json({ error: 'Something went wrong!' });
    }
});

// Get songs by title
app.get('/songs/title/:title', (req, res) => {
    try {
        const stmt = db.prepare('SELECT * FROM songs WHERE title = ?');
        const results = stmt.all(req.params.title);
        res.json(results);
    } catch (error) {
        console.log(error);
        res.status(500).json({ error: 'Something went wrong!' });
    }
});

// Get songs by artist and title
app.get('/songs/artist/:artist/title/:title', (req, res) => {
    try {
        const stmt = db.prepare('SELECT * FROM songs WHERE artist = ? AND title = ?');
        const results = stmt.all(req.params.artist, req.params.title);
        res.json(results);
    } catch (error) {
        console.log(error);
        res.status(500).json({ error: 'Something went wrong!' });
    }
});

// Get song by ID
app.get('/songs/id/:id', (req, res) => {
    try {
        const stmt = db.prepare('SELECT * FROM songs WHERE id = ?');
        const result = stmt.get(req.params.id);
        if (result) {
            res.json(result);
        } else {
            res.status(404).json({ error: 'Song not found' });
        }
    } catch (error) {
        console.log(error);
        res.status(500).json({ error: 'Something went wrong!' });
    }
});

// Get songs by year
app.get('/songs/year/:year', (req, res) => {
    try {
        const stmt = db.prepare('SELECT * FROM songs WHERE year = ?');
        const results = stmt.all(req.params.year);
        res.json(results);
    } catch (error) {
        console.log(error);
        res.status(500).json({ error: 'Something went wrong!' });
    }
});

// Add a new song with validation
app.post('/songs', (req, res) => {
    try {
        const { title, artist, year, price, quantity_in_stock } = req.body;
        if (!title || !artist || !year || !price || !quantity_in_stock) {
            return res.status(400).json({ error: 'All fields are required' });
        }
        const stmt = db.prepare('INSERT INTO songs (title, artist, year, price, quantity_in_stock) VALUES (?, ?, ?, ?, ?)');
        const info = stmt.run(title, artist, year, price, quantity_in_stock);
        res.json({ id: info.lastInsertRowid });
    } catch (error) {
        console.log(error);
        res.status(500).json({ error: 'Something went wrong!' });
    }
});

// Start the server
const port = 3000;
app.listen(port, () => {
    console.log(`Server running on http://localhost:${port}`);
});

```

4. **Save and Close Notepad**.

**Why this matters**: These updates enable the server to serve front-end files, support year-based searches, and validate song additions, completing the Week 3 requirements.

## Step 12: Test the Application
Verify that everything works as expected.

1. **Start the Server**:
   - In `WAD/week3`, type and press Enter:
     ```
     pm2 start app.mjs
     ```

2. **Test the Search Page**:
   - Open a browser and go to `http://localhost:3000/index.html`.
   - Test artist search (e.g., "The Beatles") and search by type (e.g., year "1968").

3. **Test Adding a Song**:
   - Go to `http://localhost:3000/addSong.html`.
   - Enter valid data (e.g., Title: "Happy", Artist: "Pharrell Williams", Year: 2013, Price: 0.99, Stock: 100) and submit.
   - Try with a blank field to test error handling.

**Why this matters**: Testing ensures the front-end and server work together seamlessly.

## Step 13: Save to GitHub
Save your work to your GitHub repository.

1. **Initialize Git**:
   - Type and press Enter:
     ```
     git init
     ```

2. **Add Files**:
   - Type and press Enter:
     ```
     git add .
     ```

3. **Commit Changes**:
   - Type and press Enter:
     ```
     git commit -m "Completed Week 3 front-end for HitTastic!"
     ```

4. **Push to GitHub**:
   - If you have a `WAD-week3` repo, link and push:
     ```
     git remote add origin https://github.com/yourusername/WAD-week3.git
     git push -u origin main
     ```

**Why this matters**: GitHub backs up your work and allows sharing or collaboration.

---

Congratulations! You’ve built a front-end for HitTastic! with AJAX, promises, and a Node.js server, completing Week 3.
