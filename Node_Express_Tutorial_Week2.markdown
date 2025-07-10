# Week 2: Building a RESTful API with POST, PUT, and DELETE Endpoints

This guide extends your Week 1 music API using Node.js, Express, and SQLite in a new `WAD/week2` folder. You’ll add features to manage songs (add, update, delete) and create a buy song feature with an orders table. We’ll use the Windows Command Prompt, explain every step and change in detail, and provide full scripts with comments to show what’s new. You’ll test the API with RESTer or REST Tester and save your work on GitHub.

---

## What You’ll Do
- **REST Basics**: Use HTTP methods (POST to create, PUT to update, DELETE to remove data).
- **New Endpoints**: Add routes to:
  - Add a song (POST).
  - Update a song’s price and stock (PUT).
  - Delete a song (DELETE).
  - Buy a song, reducing stock and creating an order (POST).
- **Database Updates**: Add an `orders` table and extend the `songs` table.
- **Testing**: Check your API with RESTer (personal computer) or REST Tester (university computer).
- **GitHub**: Save your work online.

**Prerequisites**: You completed Week 1, with a `WAD/week1` folder containing `app.mjs`, `setupDB.mjs`, `package.json`, and a GitHub repository (`WAD-week1`).

---

## Step 1: Set Up the Week 2 Folder
We need a new folder, `WAD/week2`, to keep Week 2 work separate from Week 1, like organizing notes in a new notebook section.

1. **Open Command Prompt**:
   - Press `Windows Key + R`, type `cmd`, and press Enter.
   - This opens a black window for typing commands, like giving your computer instructions.

2. **Go to Desktop**:
   - Type and press Enter:
     ```
     cd %userprofile%\Desktop
     ```
   - This moves you to your Desktop. Type `dir` and press Enter to see files; you should see the `WAD` folder from Week 1.

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

4. **Create week2 Folder**:
   - Type and press Enter:
     ```
     mkdir week2
     ```

5. **Move to week2**:
   - Type and press Enter:
     ```
     cd week2
     ```
   - You’re now in `WAD/week2`, where we’ll work.

**Check**: Type `dir` and press Enter. You should see no files yet, as we haven’t added anything.

**Why this matters**: A separate folder keeps Week 2 organized and prevents overwriting Week 1 files.

---

## Step 2: Copy Week 1 Files
We’ll copy `app.mjs`, `setupDB.mjs`, and `package.json` from `WAD/week1` to `WAD/week2` to reuse the Week 1 server and database setup.

1. **Copy Files**:
   - In `WAD/week2`, type and press Enter for each:
     ```
     copy ..\week1\app.mjs app.mjs
     copy ..\week1\setupDB.mjs setupDB.mjs
     copy ..\week1\package.json package.json
     ```
   - The `..\week1\` means “go up one folder to WAD, then into week1” to find the files.

2. **Check Files**:
   - Type `dir` and press Enter. You should see `app.mjs`, `setupDB.mjs`, and `package.json`.

**Why this matters**: These files contain your Week 1 server (`app.mjs`), database setup (`setupDB.mjs`), and project settings (`package.json`). Copying them lets us build on existing work.

---

## Step 3: Install Dependencies
We need Express (for the web server) and `better-sqlite3` (for the database), which were used in Week 1. We’ll reinstall them in `week2` to ensure everything works.

1. **Install Express**:
   - Type and press Enter:
     ```
     npm install express
     ```
   - This creates a `node_modules` folder (stores Express code) and a `package-lock.json` file (tracks exact versions).

2. **Install better-sqlite3**:
   - Type and press Enter:
     ```
     npm install better-sqlite3
     ```

3. **Verify**:
   - Type `dir` and press Enter. You should see `node_modules` and `package-lock.json`.

**Why this matters**: Without these tools, your server can’t run or connect to the database, like needing ingredients to cook a meal.

---

## Step 4: Update the Database (`setupDB.mjs`)
We’ll modify `setupDB.mjs` to support the new features. Compared to Week 1, we’ll:
- Keep the `students` table unchanged.
- Add `price` and `quantity_in_stock` columns to the `songs` table for updating and buying songs.
- Add a new `orders` table to track song purchases.
- Include sample song data with prices and stock.

### What’s Changing from Week 1
- **Week 1 `setupDB.mjs`** (assumed): Created `students` and `songs` tables with `id`, `artist`, `title`, and `year` for songs, and sample data.
- **Week 2 Changes**:
  - **Songs Table**: Add `price` (cost, e.g., 0.99) and `quantity_in_stock` (available copies, e.g., 10) to support updating and buying.
  - **Orders Table**: New table with `id`, `song_id` (links to a song), and `quantity` (number bought, set to 1).
  - **Sample Data**: Update songs with price and stock values.

### Steps to Update
1. **Open setupDB.mjs**:
   - Type and press Enter:
     ```
     notepad setupDB.mjs
     ```

2. **Update the File**:
   - Replace the existing code with the new version below. Here’s what each part does:
     - **Import Database**: Loads `better-sqlite3` to work with SQLite.
     - **Create Database**: Opens `mydatabase.db`.
     - **Students Table**: Same as Week 1, stores student info (unchanged).
     - **Songs Table**: Adds `price` (REAL, for decimal numbers) and `quantity_in_stock` (INTEGER, for whole numbers).
     - **Orders Table**: New table with `song_id` linking to `songs` via a FOREIGN KEY to ensure valid song IDs.
     - **Sample Data**: Adds songs with prices and stock for testing.
     - **Close Database**: Ensures the database saves properly.

   ```javascript
   import Database from 'better-sqlite3';

   const db = new Database('mydatabase.db');

   // Create students table (unchanged from Week 1)
   const stmtStudents = db.prepare(`
       CREATE TABLE IF NOT EXISTS students (
           id INTEGER PRIMARY KEY AUTOINCREMENT,
           firstname TEXT,
           lastname TEXT,
           course TEXT
       )
   `);
   stmtStudents.run();

   // Add sample students (unchanged from Week 1)
   const insertStudent = db.prepare(`
       INSERT INTO students (firstname, lastname, course) VALUES (?, ?, ?)
   `);
   insertStudent.run('Alice', 'Smith', 'Math');
   insertStudent.run('Bob', 'Jones', 'Science');

   // Create songs table with new price and quantity_in_stock columns
   const stmtSongs = db.prepare(`
       CREATE TABLE IF NOT EXISTS songs (
           id INTEGER PRIMARY KEY AUTOINCREMENT,
           artist TEXT,
           title TEXT,
           year INTEGER,
           price REAL, -- New: cost of the song
           quantity_in_stock INTEGER -- New: available copies
       )
   `);
   stmtSongs.run();

   // Add sample songs with price and stock
   const insertSong = db.prepare(`
       INSERT INTO songs (artist, title, year, price, quantity_in_stock) VALUES (?, ?, ?, ?, ?)
   `);
   insertSong.run('The Beatles', 'Hey Jude', 1968, 0.99, 10);
   insertSong.run('Queen', 'Bohemian Rhapsody', 1975, 1.29, 5);
   insertSong.run('The Beatles', 'Let It Be', 1970, 0.89, 8);

   // Create orders table (new for Week 2)
   const stmtOrders = db.prepare(`
       CREATE TABLE IF NOT EXISTS orders (
           id INTEGER PRIMARY KEY AUTOINCREMENT,
           song_id INTEGER, -- Links to songs table
           quantity INTEGER, -- Number of copies bought
           FOREIGN KEY (song_id) REFERENCES songs(id) -- Ensures valid song_id
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
   - You should see: `Database setup complete!`

**Why these changes matter**:
- **Price and Quantity**: Allow updating song prices and tracking stock for purchases, like a store inventory.
- **Orders Table**: Records each purchase, linking to a song, like an order receipt.
- **Sample Data**: Gives us songs to test adding, updating, deleting, and buying.
- **FOREIGN KEY**: Prevents orders for non-existent songs, keeping data reliable.

**Check**: The script creates `mydatabase.db` in `WAD/week2`. If you rerun it, it won’t duplicate tables due to `IF NOT EXISTS`.

**Full Script for `setupDB.mjs`**:
```javascript
import Database from 'better-sqlite3';

const db = new Database('mydatabase.db');

// Create students table (unchanged from Week 1)
const stmtStudents = db.prepare(`
    CREATE TABLE IF NOT EXISTS students (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        firstname TEXT,
        lastname TEXT,
        course TEXT
    )
`);
stmtStudents.run();

// Add sample students (unchanged from Week 1)
const insertStudent = db.prepare(`
    INSERT INTO students (firstname, lastname, course) VALUES (?, ?, ?)
`);
insertStudent.run('Alice', 'Smith', 'Math');
insertStudent.run('Bob', 'Jones', 'Science');

// Create songs table with new price and quantity_in_stock columns
const stmtSongs = db.prepare(`
    CREATE TABLE IF NOT EXISTS songs (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        artist TEXT,
        title TEXT,
        year INTEGER,
        price REAL, -- New: cost of the song
        quantity_in_stock INTEGER -- New: available copies
    )
`);
stmtSongs.run();

// Add sample songs with price and stock
const insertSong = db.prepare(`
    INSERT INTO songs (artist, title, year, price, quantity_in_stock) VALUES (?, ?, ?, ?, ?)
`);
insertSong.run('The Beatles', 'Hey Jude', 1968, 0.99, 10);
insertSong.run('Queen', 'Bohemian Rhapsody', 1975, 1.29, 5);
insertSong.run('The Beatles', 'Let It Be', 1970, 0.89, 8);

// Create orders table (new for Week 2)
const stmtOrders = db.prepare(`
    CREATE TABLE IF NOT EXISTS orders (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        song_id INTEGER, -- Links to songs table
        quantity INTEGER, -- Number of copies bought
        FOREIGN KEY (song_id) REFERENCES songs(id) -- Ensures valid song_id
    )
`);
stmtOrders.run();

console.log('Database setup complete!');
db.close();
```

---

## Step 5: Install PM2
PM2 runs our server continuously and restarts it if it crashes or if we edit the code, making development easier.

1. **Install PM2 Globally**:
   - Type and press Enter:
     ```
     npm install -g pm2
     ```
   - The `-g` means PM2 is available for all projects on your computer.

2. **Check PM2**:
   - Type and press Enter:
     ```
     pm2 --version
     ```
   - You should see a version number (e.g., `5.2.0`). If not, ensure Node.js is installed.

**Why this matters**: PM2 saves time by keeping the server running, unlike manually restarting with `node app.mjs`.

---

## Step 6: Update the Express Server (`app.mjs`)
We’ll modify `app.mjs` to add four new routes for the Week 2 exercise, while keeping Week 1 routes. The new routes use REST principles (POST, PUT, DELETE) to manage songs and orders.

### What’s Changing from Week 1
- **Week 1 `app.mjs`** (assumed): Included routes for:
  - `GET /`: Returns “Hello World”.
  - `GET /time`: Returns milliseconds since 1970.
  - `GET /greet/:name`: Greets a name.
  - `GET /students`: Lists all students.
  - `GET /students/:lastname`: Finds students by last name.
  - `POST /student/create`: Adds a student.
  - `GET /songs/artist/:artist`, `GET /songs/title/:title`, `GET /songs/artist/:artist/title/:title`: Find songs.
- **Week 2 Changes**:
  - **Add JSON Parsing**: Add `app.use(express.json())` to read JSON data in POST and PUT requests (e.g., song details).
  - **New GET Route**: Add `GET /songs/id/:id` to find a song by ID, useful for testing updates and deletes.
  - **New POST Route**: Add `POST /songs` to create a song with title, artist, year, price, and stock.
  - **New PUT Route**: Add `PUT /songs/:id` to update a song’s price and stock.
  - **New DELETE Route**: Add `DELETE /songs/:id` to delete a song.
  - **New POST Route**: Add `POST /orders` to buy a song, reducing stock and creating an order.

### Steps to Update
1. **Open app.mjs**:
   - Type and press Enter:
     ```
     notepad app.mjs
     ```

2. **Update the File**:
   - We’ll modify `app.mjs` by adding new code while keeping existing routes. Here’s what each change does:
     - **JSON Parsing**: `app.use(express.json())` lets the server understand JSON data sent in requests, like a form with song details.
     - **GET /songs/id/:id**: Retrieves a single song by ID, returns 404 if not found.
     - **POST /songs**: Inserts a new song into the database using JSON data, returns the new ID with 201 status (means “created”).
     - **PUT /songs/:id**: Updates price and stock for a song, checks if the song exists (returns 404 if not).
     - **DELETE /songs/:id**: Deletes a song by ID, checks if deleted (returns 404 if not).
     - **POST /orders**: Checks if a song exists and has stock, reduces stock by 1, adds an order, returns 201.
     - **Error Handling**: Uses `try/catch` to catch database errors and returns appropriate HTTP status codes (400, 404, 500).

   ```javascript
   import express from 'express';
   import Database from 'better-sqlite3';

   const app = express();
   const db = new Database('mydatabase.db');

   // Allow JSON data in POST and PUT requests (new for Week 2)
   app.use(express.json());

   // Root route (unchanged from Week 1)
   app.get('/', (req, res) => {
       res.send('Hello World from Express!');
   });

   // Time route (unchanged from Week 1)
   app.get('/time', (req, res) => {
       res.send(`There have been ${Date.now()} milliseconds since 1/1/70.`);
   });

   // Greet route (unchanged from Week 1)
   app.get('/greet/:name', (req, res) => {
       res.send(`Hello ${req.params.name}!`);
   });

   // Get all students (unchanged from Week 1)
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

   // Get students by last name (unchanged from Week 1)
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

   // Add a new student (unchanged from Week 1)
   app.post('/student/create', (req, res) => {
       try {
           const stmt = db.prepare('INSERT INTO students (firstname, lastname, course) VALUES (?, ?, ?)');
           const info = stmt.run(req.body.firstname, req.body.lastname, req.body.course);
           res.json({ id: info.lastInsertRowid });
       } catch (error) {
           console.log(error);
           res.status(500).json({ error: 'Something went wrong!' });
       }
   });

   // Get songs by artist (unchanged from Week 1)
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

   // Get songs by title (unchanged from Week 1)
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

   // Get songs by artist and title (unchanged from Week 1)
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

   // Get song by ID (new for Week 2, helps test updates and deletes)
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

   // Add a new song (new for Week 2)
   app.post('/songs', (req, res) => {
       try {
           const { title, artist, year, price, quantity_in_stock } = req.body;
           const stmt = db.prepare('INSERT INTO songs (title, artist, year, price, quantity_in_stock) VALUES (?, ?, ?, ?, ?)');
           const info = stmt.run(title, artist, year, price, quantity_in_stock);
           res.status(201).json({ id: info.lastInsertRowid }); // 201 means created
       } catch (error) {
           console.log(error);
           res.status(500).json({ error: 'Failed to add song' });
       }
   });

   // Update song price and quantity (new for Week 2)
   app.put('/songs/:id', (req, res) => {
       try {
           const { price, quantity_in_stock } = req.body;
           const stmt = db.prepare('UPDATE songs SET price = ?, quantity_in_stock = ? WHERE id = ?');
           const info = stmt.run(price, quantity_in_stock, req.params.id);
           if (info.changes === 0) {
               res.status(404).json({ error: 'Song not found' });
           } else {
               res.json({ message: 'Song updated successfully' });
           }
       } catch (error) {
           console.log(error);
           res.status(500).json({ error: 'Failed to update song' });
       }
   });

   // Delete a song (new for Week 2)
   app.delete('/songs/:id', (req, res) => {
       try {
           const stmt = db.prepare('DELETE FROM songs WHERE id = ?');
           const info = stmt.run(req.params.id);
           if (info.changes === 0) {
               res.status(404).json({ error: 'Song not found' });
           } else {
               res.json({ message: 'Song deleted successfully' });
           }
       } catch (error) {
           console.log(error);
           res.status(500).json({ error: 'Failed to delete song' });
       }
   });

   // Buy a song (new for Week 2, reduces stock and creates order)
   app.post('/orders', (req, res) => {
       try {
           const { song_id } = req.body;
           const checkStmt = db.prepare('SELECT quantity_in_stock FROM songs WHERE id = ?');
           const song = checkStmt.get(song_id);
           if (!song) {
               return res.status(404).json({ error: 'Song not found' });
           }
           if (song.quantity_in_stock <= 0) {
               return res.status(400).json({ error: 'Song is out of stock' });
           }
           const updateStmt = db.prepare('UPDATE songs SET quantity_in_stock = quantity_in_stock - 1 WHERE id = ?');
           updateStmt.run(song_id);
           const orderStmt = db.prepare('INSERT INTO orders (song_id, quantity) VALUES (?, ?)');
           orderStmt.run(song_id, 1);
           res.status(201).json({ message: 'Order created successfully' });
       } catch (error) {
           console.log(error);
           res.status(500).json({ error: 'Failed to create order' });
       }
   });

   app.listen(3000, () => {
       console.log('Server is running on http://localhost:3000');
   });
   ```

3. **Save and Close Notepad**.

4. **Run with PM2**:
   - Type and press Enter:
     ```
     pm2 start app.mjs --name Week2Server --watch
     ```
   - The `--watch` flag restarts the server if you edit `app.mjs`. You’ll see output like “Server is running on http://localhost:3000”.

**Why these changes matter**:
- **JSON Parsing**: Enables reading data sent to POST/PUT routes, like a form for adding a song.
- **GET /songs/id/:id**: Helps verify updates and deletes by checking a song’s details.
- **POST /songs**: Adds songs to the database, like adding a new product to a store.
- **PUT /songs/:id**: Updates price and stock, like changing a product’s price in a shop.
- **DELETE /songs/:id**: Removes a song, like deleting an item from inventory.
- **POST /orders**: Simulates buying a song, updating stock and recording the purchase, like a checkout process.
- **Status Codes**: Use 201 (created), 404 (not found), 400 (bad request), and 500 (server error) to follow REST standards, making the API clear to users.

**Check**: Open a browser to `http://localhost:3000` to see “Hello World from Express!”.

**Full Script for `app.mjs`**:
```javascript
import express from 'express';
import Database from 'better-sqlite3';

const app = express();
const db = new Database('mydatabase.db');

// Allow JSON data in POST and PUT requests (new for Week 2)
app.use(express.json());

// Root route (unchanged from Week 1)
app.get('/', (req, res) => {
    res.send('Hello World from Express!');
});

// Time route (unchanged from Week 1)
app.get('/time', (req, res) => {
    res.send(`There have been ${Date.now()} milliseconds since 1/1/70.`);
});

// Greet route (unchanged from Week 1)
app.get('/greet/:name', (req, res) => {
    res.send(`Hello ${req.params.name}!`);
});

// Get all students (unchanged from Week 1)
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

// Get students by last name (unchanged from Week 1)
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

// Add a new student (unchanged from Week 1)
app.post('/student/create', (req, res) => {
    try {
        const stmt = db.prepare('INSERT INTO students (firstname, lastname, course) VALUES (?, ?, ?)');
        const info = stmt.run(req.body.firstname, req.body.lastname, req.body.course);
        res.json({ id: info.lastInsertRowid });
    } catch (error) {
        console.log(error);
        res.status(500).json({ error: 'Something went wrong!' });
    }
});

// Get songs by artist (unchanged from Week 1)
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

// Get songs by title (unchanged from Week 1)
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

// Get songs by artist and title (unchanged from Week 1)
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

// Get song by ID (new for Week 2, helps test updates and deletes)
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

// Add a new song (new for Week 2)
app.post('/songs', (req, res) => {
    try {
        const { title, artist, year, price, quantity_in_stock } = req.body;
        const stmt = db.prepare('INSERT INTO songs (title, artist, year, price, quantity_in_stock) VALUES (?, ?, ?, ?, ?)');
        const info = stmt.run(title, artist, year, price, quantity_in_stock);
        res.status(201).json({ id: info.lastInsertRowid }); // 201 means created
    } catch (error) {
        console.log(error);
        res.status(500).json({ error: 'Failed to add song' });
    }
});

// Update song price and quantity (new for Week 2)
app.put('/songs/:id', (req, res) => {
    try {
        const { price, quantity_in_stock } = req.body;
        const stmt = db.prepare('UPDATE songs SET price = ?, quantity_in_stock = ? WHERE id = ?');
        const info = stmt.run(price, quantity_in_stock, req.params.id);
        if (info.changes === 0) {
            res.status(404).json({ error: 'Song not found' });
        } else {
            res.json({ message: 'Song updated successfully' });
        }
    } catch (error) {
        console.log(error);
        res.status(500).json({ error: 'Failed to update song' });
    }
});

// Delete a song (new for Week 2)
app.delete('/songs/:id', (req, res) => {
    try {
        const stmt = db.prepare('DELETE FROM songs WHERE id = ?');
        const info = stmt.run(req.params.id);
        if (info.changes === 0) {
            res.status(404).json({ error: 'Song not found' });
        } else {
            res.json({ message: 'Song deleted successfully' });
        }
    } catch (error) {
        console.log(error);
        res.status(500).json({ error: 'Failed to delete song' });
    }
});

// Buy a song (new for Week 2, reduces stock and creates order)
app.post('/orders', (req, res) => {
    try {
        const { song_id } = req.body;
        const checkStmt = db.prepare('SELECT quantity_in_stock FROM songs WHERE id = ?');
        const song = checkStmt.get(song_id);
        if (!song) {
            return res.status(404).json({ error: 'Song not found' });
        }
        if (song.quantity_in_stock <= 0) {
            return res.status(400).json({ error: 'Song is out of stock' });
        }
        const updateStmt = db.prepare('UPDATE songs SET quantity_in_stock = quantity_in_stock - 1 WHERE id = ?');
        updateStmt.run(song_id);
        const orderStmt = db.prepare('INSERT INTO orders (song_id, quantity) VALUES (?, ?)');
        orderStmt.run(song_id, 1);
        res.status(201).json({ message: 'Order created successfully' });
    } catch (error) {
        console.log(error);
        res.status(500).json({ error: 'Failed to create order' });
    }
});

app.listen(3000, () => {
    console.log('Server is running on http://localhost:3000');
});
```

---

## Step 7: Test the API
Browsers only send GET requests, so we’ll use RESTer (personal computer) or REST Tester (university computer) to test POST, PUT, and DELETE routes.

### Option 1: RESTer (Personal Computer)
1. **Install RESTer**:
   - Open Chrome or Firefox.
   - Search for “RESTer” in the Chrome Web Store ([RESTer for Chrome](https://chrome.google.com/webstore/detail/rester/pllhjjegkmlljlglcclnjfhklhnfihag)) or Firefox Add-ons ([RESTer for Firefox](https://addons.mozilla.org/en-US/firefox/addon/rester/)).
   - Click “Add to Chrome” or “Add to Firefox” and follow prompts.
   - Find RESTer in your toolbar (a plug icon).

2. **Test Endpoints**:
   - Open RESTer and configure:
     - **POST /songs** (add a song):
       - URL: `http://localhost:3000/songs`
       - Method: POST
       - Headers: `Content-Type: application/json`
       - Body: `{"title":"New Song","artist":"Artist","year":2023,"price":0.99,"quantity_in_stock":5}`
       - Click Send. Expect: Status 201, response `{ "id": 4 }` (ID may vary).
     - **PUT /songs/1** (update song):
       - URL: `http://localhost:3000/songs/1`
       - Method: PUT
       - Headers: `Content-Type: application/json`
       - Body: `{"price":1.49,"quantity_in_stock":7}`
       - Click Send. Expect: Status 200, response `{ "message": "Song updated successfully" }`.
     - **DELETE /songs/1** (delete song):
       - URL: `http://localhost:3000/songs/1`
       - Method: DELETE
       - Click Send. Expect: Status 200, response `{ "message": "Song deleted successfully" }`.
     - **POST /orders** (buy song):
       - URL: `http://localhost:3000/orders`
       - Method: POST
       - Headers: `Content-Type: application/json`
       - Body: `{"song_id":2}`
       - Click Send. Expect: Status 201, response `{ "message": "Order created successfully" }`.

3. **Test Errors**:
   - Try `PUT /songs/999` (invalid ID): Expect 404 `{ "error": "Song not found" }`.
   - Try `POST /orders` with `{"song_id":2}` after stock reaches 0: Expect 400 `{ "error": "Song is out of stock" }`.

**Why this matters**: RESTer lets you send and test non-GET requests, like submitting forms or updating inventory.

### Option 2: REST Tester (University Computer)
1. **Set Up REST Tester**:
   - Open a new Command Prompt:
     ```
     cd %userprofile%\Desktop
     ```
   - Clone the repository:
     ```
     git clone https://github.com/nickw1/resttest.git
     ```
   - Move to `resttest`:
     ```
     cd resttest
     ```
   - Install dependencies:
     ```
     npm install
     ```
   - Run:
     ```
     node app.mjs
     ```
   - Open a browser to `http://localhost:3200`.

2. **Test Endpoints**:
   - In REST Tester’s web interface, use the same URLs, methods, headers, and bodies as above.
   - Verify responses match expected status codes and messages.

**Why this matters**: REST Tester is a fallback for university computers where you can’t install RESTer.

---

## Step 8: Set Up GitHub
We’ll create a new GitHub repository, `WAD-week2`, to store Week 2 files, keeping them separate from Week 1.

1. **Check Git**:
   - Type and press Enter:
     ```
     git --version
     ```
   - If it fails, download Git from [git-scm.com](https://git-scm.com) and install it.

2. **Create .gitignore**:
   - Type and press Enter:
     ```
     notepad .gitignore
     ```
   - Add:
     ```
     node_modules/
     mydatabase.db
     ```
   - Save and close. This prevents uploading large or temporary files.

3. **Initialize Git**:
   - In `WAD/week2`, type and press Enter:
     ```
     git init
     ```

4. **Commit Files**:
   - Type and press Enter for each:
     ```
     git add .
     git commit -m "Week 2: Added REST API with POST, PUT, DELETE endpoints and orders"
     ```

5. **Create GitHub Repository**:
   - Go to [github.com](https://github.com), sign in, click **New Repository**.
   - Name it `WAD-week2`, keep it public or private, and click **Create Repository**.
   - Copy the “push an existing repository” commands, e.g.:
     ```
     git remote add origin https://github.com/your-username/WAD-week2.git
     git branch -M main
     git push -u origin main
     ```

6. **Push to GitHub**:
   - Run the copied commands in Command Prompt.

7. **Verify**:
   - Visit `https://github.com/your-username/WAD-week2` to see `app.mjs`, `setupDB.mjs`, `package.json`, and `.gitignore`.

**Why this matters**: GitHub stores your code online, like a cloud backup, and lets you share or revert changes.

---

## Step 9: Stop the Server
When done, stop PM2 to free up your computer:
- Type and press Enter:
  ```
  pm2 stop Week2Server
  pm2 delete Week2Server
  ```

**Why this matters**: Stops the server from running in the background, like turning off a machine when you’re done.

---

## Step 10: Summary and Next Steps
You’ve extended your Week 1 API to:
- Add songs with POST `/songs`.
- Update song price and stock with PUT `/songs/:id`.
- Delete songs with DELETE `/songs/:id`.
- Buy songs with POST `/orders`, managing stock and orders.
- Tested endpoints with RESTer or REST Tester.
- Saved work to GitHub.

**Try Next**:
- Test error cases in RESTer (e.g., invalid song ID or out-of-stock).
- Explore [Express](https://expressjs.com), [better-sqlite3](https://github.com/JoshuaWise/better-sqlite3), or [PM2](https://pm2.keymetrics.io) for more details.

---

## Addressing the Exercise 1 Quiz
The quiz questions are theoretical but included in the Week 2 material. Here are the answers with explanations:
1. **Create a booking**: POST (creates new data, like adding a reservation).
2. **Look up bookings**: GET (retrieves data, like viewing reservations).
3. **Modify a booking**: PUT (updates data, like changing a booking date).
4. **Artist doesn’t exist**: 404 Not Found (the artist isn’t in the database).
5. **Year before 1952**: 400 Bad Request (invalid year, as charts started in 1952).
6. **Delete song without admin**: 401 Unauthorized (you need admin rights).
7. **REST URL for cafes**: `https://pointsofinterest.example.com/type/cafe` (clear, descriptive URL).
8. **Cafes in London**: `https://pointsofinterest.example.com/type/cafe/in/London` (specific to location).
9. **Create POI**: `https://pointsofinterest.example.com/poi` (POST implies creating a new point).
10. **GET /product/23255**: Returns product details in JSON/XML or 404 if not found.
11. **PUT /product/23255**: Updates product, requires JSON body (e.g., `{"name":"New Name"}`) with new details.
12. **Teapot brewing coffee**: 418 I’m a Teapot (a humorous HTTP code for an invalid action).

**Why included**: Ensures all Week 2 requirements are covered, including theoretical understanding.