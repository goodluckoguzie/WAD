# Part 2: Enhancing the Web API with PM2 and Coding Exercises

This guide builds on the `WAD/week1` project, adding PM2 to manage the server and implementing coding exercises (a `/greet/:name` route and song-related API endpoints). We'll use the Windows Command Prompt, explain all scripts, provide full scripts, and update GitHub.

---

## Step 1: Install PM2
PM2 keeps the server running and restarts it if it crashes.

1. Install PM2 globally:
   - In `WAD/week1`, type and press Enter:
     ```
     npm install -g pm2
     ```

2. Verify:
   - Type and press Enter:
     ```
     pm2 --version
     ```

**Why this matters**: PM2 ensures the server runs continuously.

---

## Step 2: Update the Database for Songs
Modify `setupDB.mjs` to add a `songs` table.

1. Open `setupDB.mjs`:
   - Type and press Enter:
     ```
     notepad setupDB.mjs
     ```

2. Replace with:
   ```javascript
   import Database from 'better-sqlite3';

   const db = new Database('mydatabase.db');

   // Create a students table
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

   // Create a songs table
   const stmtSongs = db.prepare(`
       CREATE TABLE IF NOT EXISTS songs (
           id INTEGER PRIMARY KEY AUTOINCREMENT,
           artist TEXT,
           title TEXT,
           year INTEGER
       )
   `);
   stmtSongs.run();

   // Add sample songs
   const insertSong = db.prepare(`
       INSERT INTO songs (artist, title, year) VALUES (?, ?, ?)
   `);
   insertSong.run('The Beatles', 'Hey Jude', 1968);
   insertSong.run('Queen', 'Bohemian Rhapsody', 1975);
   insertSong.run('The Beatles', 'Let It Be', 1970);

   console.log('Database setup complete!');
   db.close();
   ```

3. Run:
   - Type and press Enter:
     ```
     node setupDB.mjs
     ```

**Explanation**:
- **New**: Adds a `songs` table with `id`, `artist`, `title`, `year` and sample data.
- **Unchanged**: Keeps the `students` table.

**Why this matters**: The `songs` table supports the exercise endpoints.

**Full Script for `setupDB.mjs`**:
```javascript
import Database from 'better-sqlite3';

const db = new Database('mydatabase.db');

// Create a students table
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

// Create a songs table
const stmtSongs = db.prepare(`
    CREATE TABLE IF NOT EXISTS songs (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        artist TEXT,
        title TEXT,
        year INTEGER
    )
`);
stmtSongs.run();

// Add sample songs
const insertSong = db.prepare(`
    INSERT INTO songs (artist, title, year) VALUES (?, ?, ?)
`);
insertSong.run('The Beatles', 'Hey Jude', 1968);
insertSong.run('Queen', 'Bohemian Rhapsody', 1975);
insertSong.run('The Beatles', 'Let It Be', 1970);

console.log('Database setup complete!');
db.close();
```

---

## Step 3: Update the Express Server
Add the `/greet/:name` route and song endpoints to `app.mjs`.

1. Open `app.mjs`:
   - Type and press Enter:
     ```
     notepad app.mjs
     ```

2. Replace with:
   ```javascript
   import express from 'express';
   import Database from 'better-sqlite3';

   const app = express();
   const db = new Database('mydatabase.db');

   // Allow JSON data in POST requests
   app.use(express.json());

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
           const stmt = db.prepare('INSERT INTO students (firstname, lastname, course) VALUES (?, ?, ?)');
           const info = stmt.run(req.body.firstname, req.body.lastname, req.body.course);
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
       } telligent (error) {
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

   app.listen(3000, () => {
       console.log('Server is running on http://localhost:3000');
   });
   ```

3. Run with PM2:
   - Type and press Enter:
     ```
     pm2 start app.mjs --name MyServer --watch
     ```

4. Test:
   - In a browser, visit:
     - `http://localhost:3000/greet/Alice` (shows "Hello Alice!").
     - `http://localhost:3000/songs/artist/The%20Beatles` (lists Beatles songs).
     - `http://localhost:3000/songs/title/Hey%20Jude` (lists "Hey Jude").
     - `http://localhost:3000/songs/artist/The%20Beatles/title/Hey%20Jude` (lists matching songs).
     - `http://localhost:3000/songs/id/1` (shows song with ID 1).
   - Use `%20` for spaces in URLs.

5. Manage PM2:
   - List processes: `pm2 list`
   - Stop: `pm2 stop MyServer`
   - Delete: `pm2 delete MyServer`

**Explanation of Updated `app.mjs`**:
- **New**:
  - `app.get('/greet/:name', ...)`: Returns a greeting with the name from the URL.
  - `app.get('/songs/artist/:artist', ...)`: Fetches songs by artist.
  - `app.get('/songs/title/:title', ...)`: Fetches songs by title.
  - `app.get('/songs/artist/:artist/title/:title', ...)`: Fetches songs matching both.
  - `app.get('/songs/id/:id', ...)`: Fetches a song by ID using `get()` for a single row.
- **Unchanged**: Student routes and others.
- Uses `try/catch` and `stmt.get()` for the ID endpoint.

**Why this matters**: Completes the coding exercises for a full REST API.

**Full Script for `app.mjs`**:
```javascript
import express from 'express';
import Database from 'better-sqlite3';

const app = express();
const db = new Database('mydatabase.db');

// Allow JSON data in POST requests
app.use(express.json());

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
        const stmt = db.prepare('INSERT INTO students (firstname, lastname, course) VALUES (?, ?, ?)');
        const info = stmt.run(req.body.firstname, req.body.lastname, req.body.course);
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

app.listen(3000, () => {
    console.log('Server is running on http://localhost:3000');
});
```

---

## Step 4: Update GitHub
Commit and push the changes.

1. Verify `.gitignore`:
   - Type and press Enter:
     ```
     notepad .gitignore
     ```
   - Ensure it contains:
     ```
     node_modules/
     mydatabase.db
     ```

2. Commit:
   - Type and press Enter for each:
     ```
     git add .
     git commit -m "Added PM2, greet route, and song endpoints"
     ```

3. Push:
   - Type and press Enter:
     ```
     git push origin main
     ```

4. Verify:
   - Visit your repository (e.g., `https://github.com/your-username/WAD-week1`).

**Why this matters**: Updates GitHub with the new features.

---

## Step 5: Part 2 Summary
You’ve added PM2, the `/greet/:name` route, a `songs` table, and song-related endpoints, and updated GitHub.

**Next steps**:
- Test POST routes with Postman.
- Explore [Express](https://expressjs.com), [better-sqlite3](https://github.com/JoshuaWise/better-sqlite3), or [PM2](https://pm2.keymetrics.io/).