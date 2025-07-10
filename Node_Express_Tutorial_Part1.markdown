# Part 1: Building a Web API with Node.js, Express, SQLite, and GitHub

This guide is for beginners learning to create a web server using Node.js, Express, and SQLite. We'll use the Windows Command Prompt to set up the project in a `WAD/week1` folder, write code, and push it to GitHub. Each step includes clear explanations, and we'll detail all scripts, including their purpose and how they work. We'll use Notepad via the terminal to write code and provide full scripts at the end of relevant steps.

---

## Step 1: Check if Node.js is Installed
Node.js runs JavaScript to build servers and includes `npm` to install packages like Express.

1. Open the **Command Prompt**:
   - Press `Windows Key + R`, type `cmd`, and press Enter.
   - The terminal lets you type commands to manage files and run programs.

2. Check Node.js:
   - Type and press Enter:
     ```
     node --version
     ```
   - A version number (e.g., `v16.13.0`) means Node.js is installed. If you see an error like "node is not recognized," download Node.js from [nodejs.org](https://nodejs.org), install it, and retry.

3. Check `npm`:
   - Type and press Enter:
     ```
     npm --version
     ```
   - A version number (e.g., `8.1.0`) confirms `npm` (Node Package Manager) is ready.

**Why this matters**: Node.js runs your server code, and `npm` installs tools like Express.

---

## Step 2: Create the Project Folders
We'll create a `WAD` folder with a `week1` subfolder.

1. Navigate to your Desktop:
   - Type and press Enter:
     ```
     cd %userprofile%\Desktop
     ```
   - Type `dir` to see files and folders.

2. Create the `WAD` folder:
   - Type and press Enter:
     ```
     mkdir WAD
     ```

3. Move into `WAD`:
   - Type and press Enter:
     ```
     cd WAD
     ```

4. Create the `week1` folder:
   - Type and press Enter:
     ```
     mkdir week1
     ```

5. Move into `week1`:
   - Type and press Enter:
     ```
     cd week1
     ```

**Why this matters**: Organized folders keep your project tidy, like sorting notes into binders.

---

## Step 3: Set Up the Node.js Project
Initialize a Node.js project in `week1`.

1. Initialize the project:
   - Type and press Enter:
     ```
     npm init -y
     ```
   - This creates a `package.json` file with default settings, like a project blueprint.

2. Verify `package.json`:
   - Type `dir` and press Enter to see `package.json`.

**Why this matters**: `package.json` lists your project's details and dependencies.

---

## Step 4: Install Express
Express simplifies building web servers.

1. Install Express:
   - Type and press Enter:
     ```
     npm install express
     ```
   - This creates `node_modules` (stores Express) and `package-lock.json` (tracks versions).

2. Verify installation:
   - Type `dir` and press Enter to see `node_modules` and `package-lock.json`.

**Why this matters**: Express handles web requests easily.

---

## Step 5: Create a Simple Express Server
Create `app.mjs` with routes for a welcome message and time, using the `.mjs` extension for ECMAScript modules.

1. Open Notepad:
   - Type and press Enter:
     ```
     notepad app.mjs
     ```

2. Copy and paste:
   ```javascript
   import express from 'express';
   const app = express();

   app.get('/', (req, res) => {
       res.send('Hello World from Express!');
   });

   app.get('/time', (req, res) => {
       res.send(`There have been ${Date.now()} milliseconds since 1/1/70.`);
   });

   app.listen(3000, () => {
       console.log('Server is running on http://localhost:3000');
   });
   ```

3. Save and close Notepad.

4. Run the server:
   - Type and press Enter:
     ```
     node app.mjs
     ```
   - See: `Server is running on http://localhost:3000`.

5. Test the server:
   - In a browser, visit:
     - `http://localhost:3000` (shows "Hello World from Express!").
     - `http://localhost:3000/time` (shows milliseconds since 1970).

6. Stop the server:
   - Press `Ctrl + C` in the Command Prompt.

**Explanation of `app.mjs`**:
- `import express from 'express';`: Imports Express.
- `const app = express();`: Creates a web server.
- `app.get('/', ...)`: Responds with a welcome message for the main page.
- `app.get('/time', ...)`: Shows milliseconds since 1970 using `Date.now()`.
- `app.listen(3000, ...)`: Starts the server on port 3000.
- `req` holds request details; `res` sends responses.

**Why this matters**: This sets up a server with two routes using modern JavaScript.

**Full Script for `app.mjs`**:
```javascript
import express from 'express';
const app = express();

app.get('/', (req, res) => {
    res.send('Hello World from Express!');
});

app.get('/time', (req, res) => {
    res.send(`There have been ${Date.now()} milliseconds since 1/1/70.`);
});

app.listen(3000, () => {
    console.log('Server is running on http://localhost:3000');
});
```

---

## Step 6: Install SQLite
Use SQLite for data storage with `better-sqlite3`.

1. Install `better-sqlite3`:
   - Type and press Enter:
     ```
     npm install better-sqlite3
     ```

**Why this matters**: SQLite stores data like a spreadsheet, and `better-sqlite3` connects it to Node.js.

---

## Step 7: Set Up the SQLite Database
Create `setupDB.mjs` to make a `mydatabase.db` file with a `students` table.

1. Create `setupDB.mjs`:
   - Type and press Enter:
     ```
     notepad setupDB.mjs
     ```
   - Copy and paste:
     ```javascript
     import Database from 'better-sqlite3';

     const db = new Database('mydatabase.db');

     // Create a students table
     const stmt = db.prepare(`
         CREATE TABLE IF NOT EXISTS students (
             id INTEGER PRIMARY KEY AUTOINCREMENT,
             firstname TEXT,
             lastname TEXT,
             course TEXT
         )
     `);
     stmt.run();

     // Add sample students
     const insert = db.prepare(`
         INSERT INTO students (firstname, lastname, course) VALUES (?, ?, ?)
     `);
     insert.run('Alice', 'Smith', 'Math');
     insert.run('Bob', 'Jones', 'Science');

     console.log('Database setup complete!');
     db.close();
     ```

2. Run:
   - Type and press Enter:
     ```
     node setupDB.mjs
     ```
   - See `Database setup complete!`.

**Explanation of `setupDB.mjs`**:
- `import Database from 'better-sqlite3';`: Imports SQLite tools.
- `const db = new Database('mydatabase.db');`: Creates/opens the database.
- `db.prepare(...)`: Creates a `students` table with `id`, `firstname`, `lastname`, `course`.
- `stmt.run()`: Executes table creation.
- `insert.run(...)`: Adds two students.
- `db.close()`: Closes the database.
- `console.log`: Confirms setup.

**Why this matters**: This sets up a database for student data.

**Full Script for `setupDB.mjs`**:
```javascript
import Database from 'better-sqlite3';

const db = new Database('mydatabase.db');

// Create a students table
const stmt = db.prepare(`
    CREATE TABLE IF NOT EXISTS students (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        firstname TEXT,
        lastname TEXT,
        course TEXT
    )
`);
stmt.run();

// Add sample students
const insert = db.prepare(`
    INSERT INTO students (firstname, lastname, course) VALUES (?, ?, ?)
`);
insert.run('Alice', 'Smith', 'Math');
insert.run('Bob', 'Jones', 'Science');

console.log('Database setup complete!');
db.close();
```

---

## Step 8: Update the Express Server with Database Routes
Add student-related API endpoints to `app.mjs`.

1. Open `app.mjs`:
   - Type and press Enter:
     ```
     notepad app.mjs
     ```
   - Replace with:
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

     app.listen(3000, () => {
         console.log('Server is running on http://localhost:3000');
     });
     ```

2. Run:
   - Type and press Enter:
     ```
     node app.mjs
     ```

3. Test:
   - In a browser, visit:
     - `http://localhost:3000/students` (lists all students).
     - `http://localhost:3000/students/Smith` (lists students with last name "Smith").
   - Use [Postman](https://www.postman.com/) for `/student/create` (e.g., `{"firstname":"Charlie","lastname":"Brown","course":"Art"}`).

**Explanation of Updated `app.mjs`**:
- **New**:
  - `import Database from 'better-sqlite3';`: Imports SQLite.
  - `const db = new Database('mydatabase.db');`: Connects to the database.
  - `app.use(express.json());`: Parses JSON for POST requests.
  - `app.get('/students', ...)`: Fetches all students.
  - `app.get('/students/:lastname', ...)`: Fetches students by last name, using `?` for security.
  - `app.post('/student/create', ...)`: Adds a student, returning the new ID.
  - `try/catch`: Handles errors with a 500 status.
- **Unchanged**: Root and `/time` routes.

**Why this matters**: This creates a REST API for student data.

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

app.listen(3000, () => {
    console.log('Server is running on http://localhost:3000');
});
```

---

## Step 9: Initial GitHub Setup
Set up a GitHub repository for the project.

1. Check for Git:
   - Type and press Enter:
     ```
     git --version
     ```
   - If no version appears, download Git from [git-scm.com](https://git-scm.com).

2. Initialize Git:
   - Type and press Enter:
     ```
     git init
     ```

3. Create `.gitignore`:
   - Type and press Enter:
     ```
     notepad .gitignore
     ```
   - Copy and paste:
     ```
     node_modules/
     mydatabase.db
     ```

4. Commit:
   - Type and press Enter for each:
     ```
     git add .
     git commit -m "Initial commit with Express and SQLite setup"
     ```

5. Create GitHub repository:
   - Go to [github.com](https://github.com), sign in, click **New Repository**, name it `WAD-week1`, and create it.
   - Copy the "…or push an existing repository" commands.

6. Push:
   - Run the copied commands, e.g.:
     ```
     git remote add origin https://github.com/your-username/WAD-week1.git
     git branch -M main
     git push -u origin main
     ```

7. Verify:
   - Visit your repository (e.g., `https://github.com/your-username/WAD-week1`).

**Why this matters**: GitHub stores your code online.

---

## Step 10: Part 1 Summary
You’ve built a web server with Node.js, Express, and SQLite, with routes for `/`, `/time`, `/students`, `/students/:lastname`, and `/student/create`, organized in `WAD/week1`, and pushed to GitHub.