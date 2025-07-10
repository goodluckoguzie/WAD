# Week 4: Using the Document Object Model (DOM) for Dynamic Front-Ends

This guide builds on your Week 3 "HitTastic!" front-end by introducing the Document Object Model (DOM) to dynamically manipulate HTML. You’ll replace `innerHTML` with DOM methods to display search results and add "Buy" buttons for each song, sending AJAX POST requests to your Week 2 API. The server will be updated to handle multi-quantity purchases with error checking. We’ll use the Windows Command Prompt in a new `WAD/week4` folder, provide clear step-by-step instructions, and include full scripts with comments. You’ll test in a browser and save your work on GitHub.

## What You’ll Do
- **Learn the DOM**: Use nodes, `createElement`, `appendChild`, and `removeChild` to manipulate HTML dynamically.
- **Enhance the Front-End**: Display search results using DOM and add "Buy" buttons with quantity inputs.
- **Update the Server**: Modify the `/orders` route to accept a quantity parameter and reject invalid requests.
- **Handle Errors**: Display user-friendly alerts for HTTP status codes like 400.
- **Test and Deploy**: Verify functionality and commit to GitHub.

**Prerequisites**: You completed Week 3, with a `WAD/week3` folder containing `app.mjs`, `setupDB.mjs`, `package.json`, a `public` folder with `index.html` and `index.mjs`, and a GitHub repository (`WAD-week3`).

## Step 1: Set Up the Week 4 Folder
Create a new folder, `WAD/week4`, to organize your work.

1. **Open Command Prompt**:
   - Press `Windows Key + R`, type `cmd`, and press Enter.
   - This opens a command window for typing instructions.

2. **Navigate to Desktop**:
   - Type and press Enter:
     ```
     cd %userprofile%\Desktop
     ```
   - Type `dir` to verify; you should see the `WAD` folder from previous weeks.

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

4. **Create week4 Folder**:
   - Type and press Enter:
     ```
     mkdir week4
     ```

5. **Move to week4**:
   - Type and press Enter:
     ```
     cd week4
     ```

**Why this matters**: A new folder keeps Week 4 work separate, like a new section in a notebook.

**Check**: Type `dir` and press Enter. You should see no files yet.

## Step 2: Copy Week 3 Files
Copy all files from `WAD/week3` to reuse your server, database, and front-end setup.

1. **Copy Files**:
   - In `WAD/week4`, type and press Enter:
     ```
     xcopy ..\week3\*.* . /E /H /C /I
     ```
   - The `xcopy` command copies all files and subdirectories (e.g., `public`), with `/E` for empty directories, `/H` for hidden files, `/C` to continue on errors, and `/I` to assume a directory if copying multiple files.

2. **Check Files**:
   - Type `dir` and press Enter. You should see `app.mjs`, `setupDB.mjs`, `package.json`, and a `public` folder.

**Why this matters**: This preserves your Week 3 work as the foundation for Week 4 enhancements.

## Step 3: Install Dependencies
Ensure Express, `better-sqlite3`, and PM2 are installed.

1. **Install Express**:
   - Type and press Enter:
     ```
     npm install express
     ```

2. **Install better-sqlite3**:
   - Type and press Enter:
     ```
     npm install better-sqlite3
     ```

3. **Verify**:
   - Type `dir` and press Enter. Confirm `node_modules` and `package-lock.json` exist.

**Why this matters**: These tools run your server and manage the database.

## Step 4: Install PM2
PM2 keeps your server running and restarts it on file changes.

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

**Why this matters**: PM2 automates server management, saving manual restarts.

## Step 5: Verify the Database (setupDB.mjs)
The Week 2 `setupDB.mjs` remains unchanged and supports all endpoints.

1. **Open setupDB.mjs**:
   - Type and press Enter:
     ```
     notepad setupDB.mjs
     ```

2. **Verify Content**:
   - Ensure it matches the Week 2 script (see Week 2 guide for full code), which includes `students`, `songs`, and `orders` tables with sample data.

3. **Save and Close Notepad**.

4. **Run the Script**:
   - Type and press Enter:
     ```
     node setupDB.mjs
     ```
   - Expect: `Database setup complete!`

**Why this matters**: The database supports existing routes and the updated `/orders` endpoint.

**Check**: Confirm `mydatabase.db` exists with `dir`.

## Step 6: Update index.html
Add a container for buy buttons and quantity inputs.

1. **Open index.html**:
   - In `WAD/week4/public`, type and press Enter:
     ```
     notepad index.html
     ```

2. **Modify Content**:
   - Update the `<div id="ht_results">` to include a wrapper:
     ```html
     <div id="ht_results_wrapper"></div>
     ```
   - Full updated file:
     ```html
     <!DOCTYPE html>
     <html lang="en">
     <head>
         <meta charset="UTF-8">
         <meta name="viewport" content="width=device-width, initial-scale=1.0">
         <title>HitTastic!</title>
         <script type="module" src="index.mjs"></script>
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
             input, select, button {
                 margin: 5px;
                 padding: 5px;
             }
             #ht_results_wrapper {
                 border: 1px solid black;
                 background-color: #e0e0ff;
                 width: 50%;
                 height: 200px;
                 overflow: auto;
                 margin-top: 20px;
             }
             table {
                 width: 100%;
                 border-collapse: collapse;
             }
             th, td {
                 border: 1px solid #ccc;
                 padding: 8px;
                 text-align: left;
             }
             th {
                 background-color: #f2f2f2;
             }
             .buy-section {
                 margin-top: 5px;
             }
         </style>
     </head>
     <body>
         <p><img src="images/hittastic.png" alt="HitTastic logo"></p>
         <h1>Welcome to HitTastic!</h1>
         <p>Search and shop for your favourite top 40 hits on HitTastic! Whether it's pop, rock, rap or pure liquid cheese you're into, you can be sure to find it on HitTastic! With the full range of top 40 hits since 1952 on our database, you can guarantee you'll find what you're looking for in stock. Plus, with our Year Search find out exactly what was in the chart in any year.</p>
         <div>
             Search by:
             <select id="searchType">
                 <option value="artist">Artist</option>
                 <option value="title">Title</option>
                 <option value="year">Year</option>
             </select>
             <input id="searchTerm" placeholder="Enter search term">
             <button id="searchByType">Search!</button>
         </div>
         <div id="ht_results_wrapper"></div>
     </body>
     </html>
     ```

3. **Save and Close Notepad**.

**Why this matters**: The `ht_results_wrapper` div will hold DOM-generated paragraphs and buy sections.

## Step 7: Update index.mjs
Replace `innerHTML` with DOM methods and add "Buy" functionality.

1. **Open index.mjs**:
   - In `WAD/week4/public`, type and press Enter:
     ```
     notepad index.mjs
     ```

2. **Modify Content**:
   - Comment out the old `displayResults` function and replace with DOM-based code.
   - Add "Buy" buttons with quantity inputs and a standalone `buySong` function.

   ```javascript
   // Handle search button click
   async function searchByType() {
       const type = document.getElementById('searchType').value;
       const term = document.getElementById('searchTerm').value;
       let url;
       if (type === 'artist') {
           url = `/songs/artist/${encodeURIComponent(term)}`;
       } else if (type === 'title') {
           url = `/songs/title/${encodeURIComponent(term)}`;
       } else if (type === 'year') {
           url = `/songs/year/${encodeURIComponent(term)}`;
       }
       try {
           const response = await fetch(url);
           if (!response.ok) {
               throw new Error(`HTTP error! Status: ${response.status}`);
           }
           const data = await response.json();
           displayResults(data);
       } catch (error) {
           console.error('Error fetching data:', error);
           document.getElementById('ht_results_wrapper').innerHTML = 'Error fetching data. Please try again.';
       }
   }

   // Display results using DOM
   function displayResults(songs) {
       const resultsWrapper = document.getElementById('ht_results_wrapper');
       // Clear existing content
       while (resultsWrapper.firstChild) {
           resultsWrapper.removeChild(resultsWrapper.firstChild);
       }

       songs.forEach(song => {
           // Create paragraph for song details
           const paragraph = document.createElement('p');
           paragraph.innerHTML = `${song.title} by ${song.artist} (${song.year}), Price: $${song.price}, Stock: ${song.quantity_in_stock}, Type: ${song.year < 2000 ? 'CLASSIC HIT!' : ''}`;

           // Create buy section
           const buySection = document.createElement('div');
           buySection.className = 'buy-section';

           // Create quantity input
           const quantityInput = document.createElement('input');
           quantityInput.type = 'number';
           quantityInput.min = '1';
           quantityInput.value = '1';
           quantityInput.style.width = '50px';

           // Create buy button
           const buyButton = document.createElement('button');
           const buttonText = document.createTextNode('Buy');
           buyButton.appendChild(buttonText);
           buyButton.setAttribute('data-song-id', song.id);

           // Add event listener to buy button
           buyButton.addEventListener('click', (e) => buySong(e, quantityInput));

           // Append quantity and button to buy section
           buySection.appendChild(quantityInput);
           buySection.appendChild(buyButton);

           // Append paragraph and buy section to results
           resultsWrapper.appendChild(paragraph);
           resultsWrapper.appendChild(buySection);
       });
   }

   // Standalone buy function
   async function buySong(event, quantityInput) {
       const songId = event.target.getAttribute('data-song-id');
       const quantity = parseInt(quantityInput.value) || 1;
       try {
           const response = await fetch(`/orders/${songId}`, {
               method: 'POST',
               headers: {
                   'Content-Type': 'application/json'
               },
               body: JSON.stringify({ quantity })
           });
           if (response.ok) {
               alert('Purchase successful!');
           } else if (response.status === 400) {
               const errorData = await response.json();
               alert(`Error: ${errorData.error}`);
           } else {
               alert(`Error: HTTP Status ${response.status}`);
           }
       } catch (error) {
           console.error('Error buying song:', error);
           alert('An error occurred while processing your purchase.');
       }
   }

   // Add event listener to search button
   document.getElementById('searchByType').addEventListener('click', searchByType);
   ```

3. **Save and Close Notepad**.

**Why this matters**: This script uses DOM to create paragraphs and buy sections dynamically, replacing `innerHTML`. The `buySong` function handles purchases with quantity inputs, using data attributes for song IDs and error handling for HTTP statuses.

## Step 8: Update app.mjs
Modify the `/orders` route to handle multi-quantity purchases and validate quantities.

1. **Open app.mjs**:
   - In `WAD/week4`, type and press Enter:
     ```
     notepad app.mjs
     ```

2. **Changes from Week 3**:
   - Update the `/orders/:id` route to accept a `quantity` parameter and validate it.
   - Keep all other routes unchanged.

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

   // Add a new song
   app.post('/songs', (req, res) => {
       try {
           const { title, artist, year, price, quantity_in_stock } = req.body;
           if (!title || !artist || !year || !price || !quantity_in_stock) {
               return res.status(400).json({ error: 'All fields are required' });
           }
           const stmt = db.prepare('INSERT INTO songs (title, artist, year, price, quantity_in_stock) VALUES (?, ?, ?, ?, ?)');
           const info = stmt.run(title, artist, year, price, quantity_in_stock);
           res.status(201).json({ id: info.lastInsertRowid });
       } catch (error) {
           console.log(error);
           res.status(500).json({ error: 'Failed to add song' });
       }
   });

   // Update song price and quantity
   app.put('/songs/:id', (req, res) => {
       try {
           const { price, quantity_in_stock } = req.body;
           if (price == null || quantity_in_stock == null) {
               return res.status(400).json({ error: 'Price and quantity are required' });
           }
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

   // Delete a song
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

   // Buy a song with quantity
   app.post('/orders/:id', (req, res) => {
       try {
           const songId = req.params.id;
           const { quantity } = req.body;
           if (!quantity || quantity <= 0) {
               return res.status(400).json({ error: 'Quantity must be greater than 0' });
           }
           const checkStmt = db.prepare('SELECT quantity_in_stock FROM songs WHERE id = ?');
           const song = checkStmt.get(songId);
           if (!song) {
               return res.status(404).json({ error: 'Song not found' });
           }
           if (song.quantity_in_stock < quantity) {
               return res.status(400).json({ error: 'Insufficient stock' });
           }
           const updateStmt = db.prepare('UPDATE songs SET quantity_in_stock = quantity_in_stock - ? WHERE id = ?');
           updateStmt.run(quantity, songId);
           const orderStmt = db.prepare('INSERT INTO orders (song_id, quantity) VALUES (?, ?)');
           orderStmt.run(songId, quantity);
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

4. **Save and Close Notepad**.

**Why this matters**: The updated `/orders/:id` route handles multi-quantity purchases, validates quantities, and checks stock availability, improving the buying process.

## Step 9: Test the Application
Test the DOM-based search and buy functionality.

1. **Start Server with PM2**:
   - Type and press Enter:
     ```
     pm2 start app.mjs --name Week4Server --watch
     ```

2. **Test in Browser**:
   - Open `http://localhost:3000/index.html`.
   - Search for songs (e.g., "The Beatles").
   - Verify results display as paragraphs with "Buy" buttons and quantity inputs.
   - Click "Buy" with valid (e.g., 2) and invalid (e.g., 0) quantities, checking alerts.
   - Test stock depletion (buy more than available stock).

**Why this matters**: Testing ensures DOM manipulation works, "Buy" buttons function, and error handling is effective.

## Step 10: Set Up GitHub
Commit your changes to a new `WAD-week4` repository.

1. **Create .gitignore**:
   - Type and press Enter:
     ```
     notepad .gitignore
     ```
   - Add:
     ```
     node_modules/
     mydatabase.db
     ```
   - Save and close.

2. **Initialize Git**:
   - Type and press Enter:
     ```
     git init
     ```

3. **Commit Files**:
   - Type and press Enter for each:
     ```
     git add .
     git commit -m "Week 4: Added DOM manipulation and buy functionality"
     ```

4. **Create GitHub Repository**:
   - Go to [github.com](https://github.com), sign in, click **New Repository**.
   - Name it `WAD-week4`, keep it public or private, and click **Create Repository**.
   - Copy the “push an existing repository” commands, e.g.:
     ```
     git remote add origin https://github.com/your-username/WAD-week4.git
     git branch -M main
     git push -u origin main
     ```

5. **Push to GitHub**:
   - Run the copied commands.

6. **Verify**:
   - Visit `https://github.com/your-username/WAD-week4` to see your files.

**Why this matters**: GitHub backs up your code and tracks changes, like a cloud notebook.

## Step 11: Stop the Server
When done, stop PM2 to free resources.

- Type and press Enter:
  ```
  pm2 stop Week4Server
  pm2 delete Week4Server
  ```

**Why this matters**: Stops the server from running in the background.

## Step 12: Summary and Next Steps
You’ve enhanced "HitTastic!" by:
- Using DOM to dynamically display search results.
- Adding "Buy" buttons with quantity inputs, linked to a standalone `buySong` function.
- Updating the server to handle multi-quantity purchases with validation.
- Testing and saving to GitHub.

**Try Next**:
- Add more DOM features (e.g., `querySelectorAll` for styling).
- Explore React for advanced front-end development (upcoming topic).

---

**Date and Time**: July 10, 2025, 08:57 AM BST