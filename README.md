# Web Application Development (WAD)

This repository contains my coursework for **Web Application Development (WAD)**. It walks through building a RESTful web API with Node.js, Express, and SQLite, plus a progressively enhanced front-end. Each week's folder has its own step-by-step tutorial and working code.

---

## 📂 Folder Structure

- **week1/**: 
  - Sets up a basic Node.js + Express server.
  - Connects to SQLite using better-sqlite3.
  - Defines a `students` table and simple API endpoints (`/`, `/time`, `/students`, etc.).
  - Teaches using npm, creating `package.json`, installing Express, and writing basic routes.
  - Adds database setup with scripts to create and seed the database.

- **week2/**: 
  - Expands the API with full REST principles (POST, PUT, DELETE).
  - Adds a `songs` table with `price` and `quantity_in_stock`.
  - Introduces an `orders` table to track song purchases.
  - Implements routes to:
    - Add, update, delete songs
    - Handle song purchases, reduce stock, and record orders
  - Emphasizes error handling and using tools like Postman/RESTer for testing.

- **week3/**:
  - Adds a **front-end** for the music store called **HitTastic!**.
  - Uses **HTML, CSS, and JavaScript with fetch/AJAX**.
  - Lets users search songs by artist, title, or year.
  - Displays search results in an interactive table.
  - Teaches promises and async/await.
  - Serves static files via Express.
  - Includes `public/` folder with:
    - `index.html`
    - `index.mjs` (front-end JavaScript)
    - Images and styling.

- **week4/**:
  - Enhances the front-end with **DOM manipulation**.
  - Replaces innerHTML with `createElement`, `appendChild`, etc.
  - Adds **Buy** buttons with quantity inputs next to songs.
  - Connects Buy buttons to POST requests to the `/orders` endpoint.
  - Updates server route to support multi-quantity purchases and error checking.
  - Focuses on clean, dynamic user interactions.

---

## ✅ How to Run

1. Clone the repo:

   ```bash
   git clone https://github.com/goodluckoguzie/WAD.git
   cd WAD
