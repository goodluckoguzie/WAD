# Week 5: Modules and Bundlers; Web Mapping with OpenStreetMap and Leaflet

## Introduction

Hello! In Week 5 of your Web Application Development (WAD) course, we’re going to make a cool web map that shows places like your hometown, Owerri, Nigeria, using a tool called Leaflet. We’ll also learn about splitting our code into small pieces (called modules) and using a tool called Vite to make our app fast and fun to build. This guide is written for someone new to coding, like a 10-year-old, so we’ll explain everything step by step. You’ll use the Windows Command Prompt, just like in Week 2, to set up everything. By the end, you’ll have a map that shows Owerri, lets you add markers by clicking, and talks to your “HitTastic!” server from earlier weeks. Let’s get started!

### What You’ll Learn

- **OpenStreetMap (OSM)**: A free map of the whole world, like a giant drawing anyone can edit.
- **Leaflet**: A tool to put interactive maps (ones you can zoom and drag) on your website.
- **Latitude and Longitude**: Numbers that tell us where places are on Earth, like a treasure map’s X and Y.
- **Projections**: How we turn the round Earth into a flat map on your screen.
- **ECMAScript 6 (ES6) Modules**: A way to organize your code into small files, like sorting toys into boxes.
- **Vite**: A tool that runs your app and updates it instantly when you change the code.

### What You’ll Do

1. **Exercise A**: Set up Vite and make a simple webpage, then add a Leaflet map.
2. **Exercise B**: Add a marker for Owerri, Nigeria, and let users add markers by clicking the map.
3. **Exercise C**: Connect your map to your Express server to show artists’ hometowns.
4. **Exercise D**: Add new hometowns to your server by clicking the map.
5. **Save Everything**: Keep your work organized in a folder and save it to GitHub (optional).

### Prerequisites

- You’ve done Weeks 1–4, so you know a bit about Node.js, Express, and SQLite.
- You have Node.js installed (check by typing `node --version` in Command Prompt).
- You’re okay typing commands in the black Command Prompt window.
- You have a `mydatabase.db` file from Week 2 with an `artists` table (we’ll assume it has columns `name`, `hometown`, `latitude`, `longitude`).

## Step 1: Understanding the Basics

Before we start coding, let’s learn what we’re working with, like understanding the rules of a new game.

### OpenStreetMap (OSM)

OSM is like a giant map that anyone can draw on, like coloring a picture book with friends. People use GPS devices (like in phones) to trace roads or paths, then add them to OSM. It’s free, so we can use it to make our own maps, like showing where your favorite singer is from. See more at [https://www.openstreetmap.org](https://www.openstreetmap.org).

### Leaflet

Leaflet is a JavaScript tool that lets us put a map on a webpage. It’s like a magic paintbrush that draws maps you can zoom and move. We’ll use it with OSM to show places like Owerri.

### Latitude and Longitude

Imagine the Earth is a big ball with lines drawn on it:

- **Latitude**: Lines going side to side, like the equator (0°). North is positive (like 50.9° for Southampton), south is negative. Owerri is at about 5.476310° North.
- **Longitude**: Lines going up and down, starting at 0° in Greenwich, London. East is positive, west is negative (like -1.4° for Southampton). Owerri is at about 7.025853° East. These numbers are like an address for any spot on Earth, and we’ll use them to put Owerri on our map.

### Projections

The Earth is round, but screens are flat, so we need to “flatten” the map. This is called a projection. Leaflet uses the “Web Mercator” projection (also called “Google Projection”), which splits the world into square tiles, like puzzle pieces. Zoom level 0 shows the whole world in one tiny square (256x256 pixels). Zoom level 14, which we’ll use, shows a city like Owerri clearly.

### ECMAScript 6 (ES6) Modules

Big programs can get messy if all the code is in one file, like a toy box with everything jumbled. ES6 modules let us split code into small files and share only what we need. For example:

- In `mymaths.mjs`:

  ```javascript
  export function square(n) { return n * n; }
  export function cube(n) { return n * n * n; }
  ```
  - `export` means other files can use these functions.
- In `index.mjs`:

  ```javascript
  import { square, cube } from './mymaths.mjs';
  console.log(`The square of 3 is: ${square(3)}`); // Outputs: 9
  console.log(`The cube of 2 is: ${cube(2)}`); // Outputs: 8
  ```
- We use `.mjs` files and `<script type="module">` in HTML to load them, requiring a web server.

### Vite

Vite is like a helper who runs your website while you build it. It:

- Runs a server at `http://localhost:5173` so you can see your map.
- Uses Hot Module Reloading (HMR) to update your webpage instantly when you change code.
- Can bundle your app into one file for sharing later (we’ll try this at the end).

## Step 2: Set Up Your Project

Let’s create a new folder and set up our tools, like getting ready to build a model airplane.

1. **Open Command Prompt**:

   - Press `Win + R`, type `cmd`, and press Enter. This opens a black window where you type commands, like telling your computer what to do.
   - Check Node.js:

     ```bash
     node --version
     npm --version
     ```
     - You should see version numbers (e.g., `v18.17.0`). If not, download Node.js from [nodejs.org](https://nodejs.org).

2. **Create a Project Folder**:

   - Go to your Desktop:

     ```bash
     cd %userprofile%\Desktop
     ```
     - This is like walking to your desk where you’ll work.
   - Make a folder called `WAD-week5`:

     ```bash
     mkdir WAD-week5
     cd WAD-week5
     ```
     - `mkdir` makes the folder, and `cd` takes you inside it, like opening a new notebook.

3. **Initialize npm**:

   - Type:

     ```bash
     npm init -y
     ```
     - This creates a `package.json` file, like a list of ingredients for your project. The `-y` means “use defaults” to save time.
   - Check it worked:

     ```bash
     dir
     ```
     - You should see `package.json`.

4. **Install Vite and Leaflet**:

   - Type:

     ```bash
     npm install vite --save-dev
     npm install leaflet@1.9.4
     ```
     - **Vite**: Helps run and build your app. `--save-dev` means it’s only for building, not the final app.
     - **Leaflet 1.9.4**: The map tool, version 1.9.4 is stable (per [Leaflet download page](https://leafletjs.com)).
   - Check:

     ```bash
     dir
     ```
     - You’ll see `node_modules` (where Vite and Leaflet live) and `package-lock.json` (tracks exact versions).

## Step 3: Coding Exercise A - Working with Vite

This exercise has two parts: making a simple webpage and then adding a Leaflet map.

### A.1: Serve a Static HTML Page

Let’s make a simple webpage that says “Hello World!” to test Vite.

1. **Create** `index.html`:

   - Type:

     ```bash
     notepad index.html
     ```
   - Copy and paste:

     ```html
     <!DOCTYPE html>
     <html lang="en">
     <head>
         <meta charset="UTF-8">
         <meta name="viewport" content="width=device-width, initial-scale=1.0">
         <title>Hello World</title>
     </head>
     <body>
         <h1>Hello World!</h1>
     </body>
     </html>
     ```
   - Save (File > Save) and close Notepad.
   - **Explanation**:
     - `<!DOCTYPE html>`: Tells browsers this is a modern webpage.
     - `<html lang="en">`: Says the page uses English.
     - `<meta charset="UTF-8">`: Allows special characters like accents.
     - `<meta name="viewport" ...>`: Makes the page look good on phones.
     - `<title>Hello World</title>`: Names the browser tab.
     - `<h1>Hello World!</h1>`: Shows big text on the page.

2. **Run Vite**:

   - Type:

     ```bash
     npx vite dev
     ```
     - This starts Vite’s server on port 5173, so you’ll visit `http://localhost:5173` in Chrome or another browser.
   - Open your browser and go to `http://localhost:5173`. You should see “Hello World!”.
   - **Test Hot Module Reloading (HMR)**:
     - Open `index.html`:

       ```bash
       notepad index.html
       ```
     - Change `<h1>Hello World!</h1>` to `<h1>Hello Vite!</h1>`, save, and watch the browser update instantly without refreshing.

3. **Why This Matters**:

   - You’ve set up Vite to serve a webpage, like opening a toy store to show off your toys.
   - HMR is like having a magic toy box that updates your toys as you add new ones.

### A.2: Serve a Leaflet Page

Now let’s make a map showing Southampton, then tweak it to test other places.

1. **Create** `src` **Folder and** `main.mjs`:

   - Type:

     ```bash
     mkdir src
     notepad src\main.mjs
     ```
   - Copy and paste:

     ```javascript
     import 'leaflet';
     import 'leaflet/dist/leaflet.css';

     const map = L.map('map').setView([50.908, -1.4], 14);

     L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
         attribution: 'Map data © <a href="https://www.openstreetmap.org">OpenStreetMap</a> contributors'
     }).addTo(map);

     L.marker([50.908, -1.4]).addTo(map)
         .bindPopup('Southampton')
         .openPopup();
     ```
   - Save and close.
   - **Line-by-Line Explanation**:
     - `import 'leaflet';`: Loads the Leaflet library, like borrowing a map-making tool.
     - `import 'leaflet/dist/leaflet.css';`: Loads Leaflet’s styles, so the map looks nice.
     - `const map = L.map('map')`: Makes a map linked to a `<div id="map">` in HTML.
     - `.setView([50.908, -1.4], 14)`: Centers the map at Southampton (latitude 50.908, longitude -1.4) with zoom level 14 (good for city views).
     - `L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', ...)`: Gets map images (tiles) from OSM’s server. `{s}`, `{z}`, `{x}`, `{y}` are placeholders for tile details.
     - `{ attribution: 'Map data © ...' }`: Credits OSM for the map data, linking to `https://www.openstreetmap.org`.
     - `.addTo(map)`: Puts the tiles on your map.
     - `L.marker([50.908, -1.4])`: Adds a pin at Southampton’s coordinates.
     - `.addTo(map)`: Puts the pin on the map.
     - `.bindPopup('Southampton')`: Adds a bubble that says “Southampton” when you click the pin.
     - `.openPopup()`: Opens the bubble right away.

2. **Update** `index.html`:

   - Type:

     ```bash
     notepad index.html
     ```
   - Replace with:

     ```html
     <!DOCTYPE html>
     <html lang="en">
     <head>
         <meta charset="UTF-8">
         <meta name="viewport" content="width=device-width, initial-scale=1.0">
         <title>Leaflet Map</title>
         <script type="module" src="src/main.mjs"></script>
     </head>
     <body>
         <h1>Leaflet Map Example</h1>
         <div id="map" style="width: 100%; height: 600px;"></div>
     </body>
     </html>
     ```
   - Save and close.
   - **Explanation**:
     - `<script type="module" src="src/main.mjs">`: Links your JavaScript module to the page.
     - `<div id="map" style="width: 100%; height: 600px;">`: Creates a big box (100% wide, 600 pixels tall) for the map.

3. **Test the Map**:

   - If Vite isn’t running, start it:

     ```bash
     npx vite dev
     ```
   - Go to `http://localhost:5173`. You’ll see a map with a marker at Southampton labeled “Southampton.”
   - **Test Hot Module Reloading (HMR)**:
     - Open `main.mjs`:

       ```bash
       notepad src\main.mjs
       ```
     - Change coordinates to University of Southampton (50.9079, -1.4015):

       ```javascript
       const map = L.map('map').setView([50.9079, -1.4015], 14);
       L.marker([50.9079, -1.4015]).addTo(map).bindPopup('University').openPopup();
       ```
     - Save, and the map updates to the university.
     - Try New York (40.75, -74):

       ```javascript
       const map = L.map('map').setView([40.75, -74], 14);
       L.marker([40.75, -74]).addTo(map).bindPopup('New York').openPopup();
       ```
     - Save, and the map moves to New York.

4. **Why This Matters**:

   - You’ve made a map with Leaflet, like drawing a city on your computer.
   - Vite’s HMR makes testing changes super fast, like seeing your drawing update as you color.

## Step 4: Exercise 1 - Trying the Leaflet Example

The handout asks you to try the Leaflet example by adding `map.html` and `map.mjs` to your Weeks 3–4 project and serving it via your Express server. Let’s do it and discuss why it doesn’t work as expected.

1. **Set Up in Weeks 3–4 Project**:

   - Go to your Weeks 3–4 folder (assuming it’s `WAD-week3-4`):

     ```bash
     cd %userprofile%\Desktop\WAD-week3-4
     ```
   - Create a `public` folder if it doesn’t exist:

     ```bash
     mkdir public
     ```
   - Create `public/map.html`:

     ```bash
     notepad public\map.html
     ```
     - Paste:

       ```html
       <html>
       <head>
           <title>Leaflet Example</title>
           <script type='module' src='map.mjs'></script>
       </head>
       <body>
           <h1>Leaflet Test</h1>
           <div id="map1" style="width:800px; height:600px"></div>
       </body>
       </html>
       ```
     - Save and close.
   - Create `public/map.mjs`:

     ```bash
     notepad public\map.mjs
     ```
     - Paste:

       ```javascript
       import 'leaflet';
       import 'leaflet/dist/leaflet.css';

       const map = L.map("map1");

       const attrib="Map data copyright OpenStreetMap contributors, Open Database Licence";

       L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
           attribution: attrib
       }).addTo(map);

       map.setView([50.908,-1.4], 14);
       ```
     - Save and close.

2. **Run Express Server**:

   - Assuming your Express server from Week 2 is in `WAD-week3-4`:

     ```bash
     node app.mjs
     ```
   - Visit `http://localhost:3000/map.html`. The map won’t load.

3. **Discussion: Why It Doesn’t Work**:

   - Open Chrome, press `F12`, and click the “Console” tab. You’ll see an error like: `Failed to resolve module specifier "leaflet"`.
   - **Reason**: Browsers can’t understand `import 'leaflet'` because it refers to an npm package, not a file. Unlike Node.js, browsers need a bundler (like Vite) to combine your code with libraries like Leaflet into one file they can read.
   - **Solution**: Use Vite, as we did in Exercise A, to handle these imports. That’s why we’re building a new project in `WAD-week5` with Vite.

4. **Why This Matters**:

   - This shows why bundlers like Vite are needed for modern web apps with libraries, like packing all your toys into one box for a friend to use.

## Step 5: Coding Exercise B - Adding Markers and Interactivity

This exercise has three parts: add a marker for Owerri, make the map interactive with click events, and add popups with user text.

1. **Part 1: Add Marker for Owerri**:

   - Owerri’s coordinates are 5.476310, 7.025853 (from [https://www.latlong.net](https://www.latlong.net)).
   - Go back to `WAD-week5`:

     ```bash
     cd %userprofile%\Desktop\WAD-week5
     ```
   - Open `main.mjs`:

     ```bash
     notepad src\main.mjs
     ```
   - Replace with:

     ```javascript
     import 'leaflet';
     import 'leaflet/dist/leaflet.css';

     const map = L.map('map').setView([5.476310, 7.025853], 14);

     L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
         attribution: 'Map data © <a href="https://www.openstreetmap.org">OpenStreetMap</a> contributors'
     }).addTo(map);

     L.marker([5.476310, 7.025853]).addTo(map)
         .bindPopup('Owerri, Nigeria')
         .openPopup();
     ```
   - Save and close.
   - **Explanation** (same as A.2, updated coordinates):
     - `[5.476310, 7.025853]`: Owerri’s latitude and longitude.
     - `bindPopup('Owerri, Nigeria')`: Shows “Owerri, Nigeria” when you click the marker.
   - Test:
     - Ensure Vite is running (`npx vite dev`).
     - Visit `http://localhost:5173`. See the map centered on Owerri with a marker.

2. **Part 2: Add Click Event for Markers**:

   - Open `main.mjs`:

     ```bash
     notepad src\main.mjs
     ```
   - Add at the end:

     ```javascript
     map.on('click', (e) => {
         const lat = e.latlng.lat;
         const lng = e.latlng.lng;
         const text = prompt('Enter text for the popup:');
         if (text) {
             L.marker([lat, lng]).addTo(map).bindPopup(text).openPopup();
         }
     });
     ```
   - Full code:

     ```javascript
     import 'leaflet';
     import 'leaflet/dist/leaflet.css';

     const map = L.map('map').setView([5.476310, 7.025853], 14);

     L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
         attribution: 'Map data © <a href="https://www.openstreetmap.org">OpenStreetMap</a> contributors'
     }).addTo(map);

     L.marker([5.476310, 7.025853]).addTo(map)
         .bindPopup('Owerri, Nigeria')
         .openPopup();

     map.on('click', (e) => {
         const lat = e.latlng.lat;
         const lng = e.latlng.lng;
         const text = prompt('Enter text for the popup:');
         if (text) {
             L.marker([lat, lng]).addTo(map).bindPopup(text).openPopup();
         }
     });
     ```
   - Save and close.
   - **Explanation**:
     - `map.on('click', (e) => {`: Runs code when you click the map. `e` is the click event.
     - `const lat = e.latlng.lat;`: Gets the latitude where you clicked.
     - `const lng = e.latlng.lng;`: Gets the longitude.
     - `const text = prompt('Enter text for the popup:');`: Shows a box asking for text.
     - `if (text) {`: Checks if you typed something (not just clicked “Cancel”).
     - `L.marker([lat, lng]).addTo(map)`: Adds a marker where you clicked.
     - `.bindPopup(text).openPopup()`: Shows your text in a bubble.
   - Test:
     - Click the map, type something like “My House,” and see a new marker with your text.

3. **Why This Matters**:

   - You’ve made your map interactive, like a game where you can drop pins anywhere and name them.

## Step 6: Coding Exercise C - Connecting to a Web API

Now we’ll connect your map to your “HitTastic!” server to show where artists are from.

1. **Set Up Express Server**:

   - Create a `server` folder:

     ```bash
     cd %userprofile%\Desktop\WAD-week5
     mkdir server
     cd server
     ```
   - Initialize npm:

     ```bash
     npm init -y
     ```
   - Install dependencies:

     ```bash
     npm install express better-sqlite3 cors
     ```
     - `express`: Runs the server.
     - `better-sqlite3`: Connects to your database.
     - `cors`: Lets your map talk to the server (solves same-origin policy issues).
   - Copy your `mydatabase.db` from Week 2 to `WAD-week5/server` (assumes an `artists` table with `name`, `hometown`, `latitude`, `longitude`).
   - Create `app.mjs`:

     ```bash
     notepad app.mjs
     ```
     - Paste:

       ```javascript
       import express from 'express';
       import sqlite3 from 'better-sqlite3';
       import cors from 'cors';

       const app = express();
       app.use(cors());
       app.use(express.json());

       const db = new sqlite3('mydatabase.db');

       app.get('/hometown/:artist', (req, res) => {
           const artist = req.params.artist;
           const row = db.prepare('SELECT hometown, latitude, longitude FROM artists WHERE name = ?').get(artist);
           if (row) {
               res.json(row);
           } else {
               res.status(404).send('Artist not found');
           }
       });

       const PORT = 3000;
       app.listen(PORT, () => {
           console.log(`Server running on port ${PORT}`);
       });
       ```
     - Save and close.
     - **Explanation**:
       - `import express ...`: Loads tools for the server.
       - `const app = express();`: Creates the server.
       - `app.use(cors());`: Allows your map (port 5173) to talk to the server (port 3000).
       - `app.use(express.json());`: Reads JSON data sent to the server.
       - `const db = new sqlite3('mydatabase.db');`: Opens your database.
       - `app.get('/hometown/:artist', ...)`: Creates a route to find an artist’s hometown.
       - `req.params.artist`: Gets the artist name from the URL (e.g., `/hometown/Queen`).
       - `db.prepare(...).get(artist)`: Finds one row in the `artists` table.
       - `if (row) { res.json(row); }`: Sends the hometown data as JSON.
       - `else { res.status(404).send('Artist not found'); }`: Says “not found” if no match.
       - `app.listen(PORT, ...)`: Starts the server at `http://localhost:3000`.

2. **Install PM2**:

   - Type:

     ```bash
     npm install -g pm2
     ```
     - `-g` makes PM2 available everywhere on your computer.
   - Check:

     ```bash
     pm2 --version
     ```
     - You should see a version number (e.g., `5.2.0`).

3. **Run Server with PM2**:

   - Type:

     ```bash
     pm2 start app.mjs --name Week5Server --watch
     ```
     - `--watch` restarts the server if you change `app.mjs`.
   - Check it’s running:

     ```bash
     pm2 list
     ```

4. **Modify Leaflet App**:

   - Go back:

     ```bash
     cd ..
     ```
   - Update `index.html`:

     ```bash
     notepad index.html
     ```
     - Replace with:

       ```html
       <!DOCTYPE html>
       <html lang="en">
       <head>
           <meta charset="UTF-8">
           <meta name="viewport" content="width=device-width, initial-scale=1.0">
           <title>Leaflet Map</title>
           <script type="module" src="src/main.mjs"></script>
       </head>
       <body>
           <h1>Leaflet Map Example</h1>
           <input type="text" id="artist-input" placeholder="Enter artist name">
           <button id="search-button">Where is this artist from?</button>
           <div id="map" style="width: 100%; height: 600px;"></div>
       </body>
       </html>
       ```
     - Save and close.
     - **Explanation**:
       - `<input type="text" id="artist-input" ...>`: Adds a box to type an artist’s name.
       - `<button id="search-button">`: Adds a button to search for the artist.
   - Update `main.mjs`:

     ```bash
     notepad src\main.mjs
     ```
     - Replace with:

       ```javascript
       import 'leaflet';
       import 'leaflet/dist/leaflet.css';

       const map = L.map('map').setView([5.476310, 7.025853], 14);

       L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
           attribution: 'Map data © <a href="https://www.openstreetmap.org">OpenStreetMap</a> contributors'
       }).addTo(map);

       L.marker([5.476310, 7.025853]).addTo(map)
           .bindPopup('Owerri, Nigeria')
           .openPopup();

       map.on('click', (e) => {
           const lat = e.latlng.lat;
           const lng = e.latlng.lng;
           const text = prompt('Enter text for the popup:');
           if (text) {
               L.marker([lat, lng]).addTo(map).bindPopup(text).openPopup();
           }
       });

       const searchButton = document.getElementById('search-button');
       searchButton.addEventListener('click', () => {
           const artist = document.getElementById('artist-input').value;
           fetch(`http://localhost:3000/hometown/${artist}`)
               .then(response => {
                   if (!response.ok) throw new Error('Artist not found');
                   return response.json();
               })
               .then(data => {
                   map.setView([data.latitude, data.longitude], 14);
                   L.marker([data.latitude, data.longitude]).addTo(map)
                       .bindPopup(data.hometown)
                       .openPopup();
               })
               .catch(error => {
                   console.error('Error:', error);
                   alert('Artist not found');
               });
       });
       ```
     - Save and close.
     - **Explanation**:
       - `const searchButton = document.getElementById('search-button');`: Finds the button.
       - `searchButton.addEventListener('click', () => {`: Runs code when the button is clicked.
       - `const artist = document.getElementById('artist-input').value;`: Gets the artist name you typed.
       - `fetch('http://localhost:3000/hometown/' + artist)`: Asks the server for the artist’s hometown.
       - `.then(response => { if (!response.ok) ...`: Checks if the server found the artist.
       - `return response.json()`: Turns the server’s answer into data we can use.
       - `map.setView([data.latitude, data.longitude], 14)`: Moves the map to the artist’s hometown.
       - `L.marker([data.latitude, data.longitude])...`: Adds a marker with the hometown name.
       - `.catch(error => { ...`: Shows an alert if the artist isn’t found.

5. **Exercise 2: Test and Discuss CORS**:

   - Test:
     - Ensure Vite (`npx vite dev`) and PM2 (`pm2 start app.mjs --name Week5Server --watch`) are running.
     - Go to `http://localhost:5173`, type an artist (e.g., “The Beatles”), click the button, and see the map move to their hometown.
   - **Discussion**:
     - Without CORS, you’d see an error in the browser console (F12, “Console” tab) like “CORS policy: No ‘Access-Control-Allow-Origin’ header.”
     - **Why**: The map (port 5173) and server (port 3000) are on different ports, breaking the same-origin policy.
     - **Fix**: The `cors` middleware (`app.use(cors());`) allows the map to talk to the server.
   - **Why This Matters**: CORS is like giving your map permission to call your server, like asking your parents to let a friend visit.

## Step 7: Coding Exercise D - Adding New Hometowns

Let’s let users add new artists and hometowns by clicking the map.

1. **Update Express Server**:

   - Go to `server`:

     ```bash
     cd server
     ```
   - Open `app.mjs`:

     ```bash
     notepad app.mjs
     ```
   - Add the POST route before `app.listen`:

     ```javascript
     app.post('/hometown', (req, res) => {
         const { latitude, longitude, artist, hometown } = req.body;
         try {
             const stmt = db.prepare('INSERT INTO artists (name, hometown, latitude, longitude) VALUES (?, ?, ?, ?)');
             stmt.run(artist, hometown, latitude, longitude);
             res.status(201).send('Hometown added');
         } catch (error) {
             res.status(500).send('Error adding hometown');
         }
     });
     ```
   - Full `app.mjs`:

     ```javascript
     import express from 'express';
     import sqlite3 from 'better-sqlite3';
     import cors from 'cors';

     const app = express();
     app.use(cors());
     app.use(express.json());

     const db = new sqlite3('mydatabase.db');

     app.get('/hometown/:artist', (req, res) => {
         const artist = req.params.artist;
         const row = db.prepare('SELECT hometown, latitude, longitude FROM artists WHERE name = ?').get(artist);
         if (row) {
             res.json(row);
         } else {
             res.status(404).send('Artist not found');
         }
     });

     app.post('/hometown', (req, res) => {
         const { latitude, longitude, artist, hometown } = req.body;
         try {
             const stmt = db.prepare('INSERT INTO artists (name, hometown, latitude, longitude) VALUES (?, ?, ?, ?)');
             stmt.run(artist, hometown, latitude, longitude);
             res.status(201).send('Hometown added');
         } catch (error) {
             res.status(500).send('Error adding hometown');
         }
     });

     const PORT = 3000;
     app.listen(PORT, () => {
         console.log(`Server running on port ${PORT}`);
     });
     ```
   - Save, close, restart:

     ```bash
     pm2 restart Week5Server
     ```
   - **Explanation**:
     - `app.post('/hometown', ...)`: Creates a route to add a new hometown.
     - `const { latitude, longitude, artist, hometown } = req.body;`: Gets data sent from the map.
     - `db.prepare('INSERT INTO ...')`: Adds a new row to the `artists` table.
     - `res.status(201).send('Hometown added')`: Says “success” with code 201.
     - `catch (error) { ...`: Handles errors, like if the database fails.

2. **Update Leaflet App**:

   - Go back:

     ```bash
     cd ..
     ```
   - Open `main.mjs`:

     ```bash
     notepad src\main.mjs
     ```
   - Replace the click event with:

     ```javascript
     map.on('click', (e) => {
         const lat = e.latlng.lat;
         const lng = e.latlng.lng;
         const artist = prompt('Enter artist name:');
         const hometown = prompt('Enter hometown name:');
         if (artist && hometown) {
             fetch('http://localhost:3000/hometown', {
                 method: 'POST',
                 headers: {
                     'Content-Type': 'application/json',
                 },
                 body: JSON.stringify({ latitude: lat, longitude: lng, artist, hometown }),
             })
             .then(response => {
                 if (response.ok) {
                     L.marker([lat, lng]).addTo(map).bindPopup(hometown).openPopup();
                 } else {
                     alert('Failed to add hometown');
                 }
             })
             .catch(error => {
                 console.error('Error:', error);
                 alert('Error adding hometown');
             });
         }
     });
     ```
   - Full `main.mjs`:

     ```javascript
     import 'leaflet';
     import 'leaflet/dist/leaflet.css';

     const map = L.map('map').setView([5.476310, 7.025853], 14);

     L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
         attribution: 'Map data © <a href="https://www.openstreetmap.org">OpenStreetMap</a> contributors'
     }).addTo(map);

     L.marker([5.476310, 7.025853]).addTo(map)
         .bindPopup('Owerri, Nigeria')
         .openPopup();

     map.on('click', (e) => {
         const lat = e.latlng.lat;
         const lng = e.latlng.lng;
         const artist = prompt('Enter artist name:');
         const hometown = prompt('Enter hometown name:');
         if (artist && hometown) {
             fetch('http://localhost:3000/hometown', {
                 method: 'POST',
                 headers: {
                     'Content-Type': 'application/json',
                 },
                 body: JSON.stringify({ latitude: lat, longitude: lng, artist, hometown }),
             })
             .then(response => {
                 if (response.ok) {
                     L.marker([lat, lng]).addTo(map).bindPopup(hometown).openPopup();
                 } else {
                     alert('Failed to add hometown');
                 }
             })
             .catch(error => {
                 console.error('Error:', error);
                 alert('Error adding hometown');
             });
         }
     });

     const searchButton = document.getElementById('search-button');
     searchButton.addEventListener('click', () => {
         const artist = document.getElementById('artist-input').value;
         fetch(`http://localhost:3000/hometown/${artist}`)
             .then(response => {
                 if (!response.ok) throw new Error('Artist not found');
                 return response.json();
             })
             .then(data => {
                 map.setView([data.latitude, data.longitude], 14);
                 L.marker([data.latitude, data.longitude]).addTo(map)
                     .bindPopup(data.hometown)
                     .openPopup();
             })
             .catch(error => {
                 console.error('Error:', error);
                 alert('Artist not found');
             });
     });
     ```
   - Save and close.
   - **Explanation**:
     - `const artist = prompt('Enter artist name:');`: Asks for an artist name.
     - `const hometown = prompt('Enter hometown name:');`: Asks for a hometown.
     - `if (artist && hometown) {`: Checks you entered both.
     - `fetch('http://localhost:3000/hometown', ...)`: Sends data to the server.
     - `method: 'POST'`: Says we’re adding new data.
     - `headers: { 'Content-Type': 'application/json' }`: Tells the server we’re sending JSON.
     - `body: JSON.stringify(...)`: Turns data into JSON format.
     - `if (response.ok) {`: Checks if the server saved the data.
     - `L.marker([lat, lng])...`: Adds a marker if successful.

3. **Test**:

   - Click the map, enter an artist (e.g., “Davido”) and hometown (e.g., “Lagos”), and see a new marker.
   - Check the database to confirm the new entry.

## Step 8: Optional - Building for Production

To make your app ready for sharing:

- Type:

  ```bash
  npx vite build
  ```
- This creates a `dist` folder with a bundled version of your app, like packing your map into a box to send to a friend.

## Step 9: Optional - Save to GitHub

1. **Initialize Git**:

   ```bash
   cd %userprofile%\Desktop\WAD-week5
   git init
   ```
2. **Create** `.gitignore`:

   ```bash
   notepad .gitignore
   ```
   - Add:

     ```
     node_modules/
     dist/
     mydatabase.db
     ```
   - Save and close.
3. **Commit and Push**:

   ```bash
   git add .
   git commit -m "Week 5: Leaflet map with Owerri marker and API integration"
   ```
   - Create a GitHub repo (`WAD-week5`), then run:

     ```bash
     git remote add origin https://github.com/your-username/WAD-week5.git
     git branch -M main
     git push -u origin main
     ```

## Step 10: Summary and Next Steps

You’ve completed Week 5! You:

- Set up Vite and Leaflet.
- Made a map with a marker for Owerri (coordinates from [https://www.latlong.net](https://www.latlong.net)).
- Added click events for new markers with popups.
- Connected to an Express server to show and add artist hometowns.
- Learned why bundlers like Vite are needed (Exercise 1) and how CORS fixes server issues (Exercise 2).

**Next Steps**:

- Try adding more Leaflet features, like drawing lines (see [Leaflet tutorials](https://leafletjs.com)).
- Deploy your `dist` folder to a web server.
- If anything breaks, check the browser console (F12) or ask for help!

**Code Explanation Table**:

| File            | Purpose                                                                 |
|-----------------|-------------------------------------------------------------------------|
| `index.html`    | Shows the map, input field, and button; links to `main.mjs`.            |
| `src/main.mjs`  | Sets up Leaflet map, Owerri marker, click events, and API calls.        |
| `server/app.mjs`| Express server with routes to get and add artist hometowns.             |