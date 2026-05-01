# Beginner-Friendly Tutorial: Building a Containerized Backend Server with PostgreSQL

## Overview

In this tutorial, you will build a fully functional backend web server that:

- Runs inside Docker containers
- Connects to a PostgreSQL database
- Exposes RESTful CRUD endpoints
- Uses a clean and simple tech stack

We will use:

- **Node.js (Express)** for the backend server
- **PostgreSQL** for the database
- **Docker + Docker Compose** for containerization  
  By the end, you will have a working backend system you can extend for your own projects.

---

## Prerequisites

### Operating System

- Macbook or any Linux Distrobution

### Required Software

1. Node.js (v18 or higher)
2. Docker Desktop (latest version)
3. Git (optional but recommended)
4. VS Code (or any code editor)

### Knowledge Assumptions

- Basic familiarity with a programming language
- Basic understanding of HTTP (GET, POST, etc.)
- Very basic understanding of Docker (helpful, not required)

---

## Project Structure

We will build the following structure:

```text
project-root/
│
├── backend/
│   ├── src/
│   ├── package.json
│   ├── Dockerfile
│
├── docker-compose.yml
└── README.md
```

---

## Step-by-Step Instructions

### Step 1: Create Project Directory

We begin by creating a dedicated folder for the project. This keeps all files organized and makes it easier to manage containers and code.

```sh
mkdir simple-backend-tutorial
cd simple-backend-tutorial
```

---

### Step 2: Initialize Backend Folder

We separate backend code into its own directory to keep concerns isolated and maintain a clean structure.

```sh
mkdir backend
cd backend
```

---

### Step 3: Initialize Node Project

This creates a `package.json` file, which tracks dependencies and scripts for your Node.js application.

```sh
npm init -y
```

---

### Step 4: Install Dependencies

These libraries provide core functionality:

- `express`: web server
- `pg`: PostgreSQL client
- `cors`: handles cross-origin requests
- `dotenv`: manages environment variables

```sh
npm install express pg cors dotenv
```

---

### Step 5: Install Dev Dependency

`nodemon` automatically restarts your server when files change—very helpful during development.

```sh
npm install --save-dev nodemon
```

---

### Step 6: Update `package.json` Scripts

We define commands to start the server in development and production modes.

**File: `backend/package.json`**

```json
"scripts": {
  "start": "node src/index.js",
  "dev": "nodemon src/index.js"
}
```

---

### Step 7: Create Source Folder

This is where all application logic will live.

```sh
mkdir src
```

---

### Step 8: Create Entry File

The entry point is where the server starts execution.

```sh
touch src/index.js
```

---

### Step 9: Build Basic Express Server

We create a minimal server to confirm everything is working before adding complexity.

**File: `backend/src/index.js`**

```js
const express = require("express");
const app = express();
app.use(express.json());
app.get("/", (req, res) => {
  res.send("Server is running");
});
app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

---

### Step 10: Run Server Locally

This step verifies that Node and Express are configured correctly.

```sh
npm run dev
```

**Expected Output:**

- Terminal: `Server running on port 3000`
- Browser (http://localhost:3000):  
  → `Server is running`

---

### Step 11: Add PostgreSQL Connection File

We isolate database logic into its own file for modularity and reuse.

```sh
touch src/db.js
```

---

### Step 12: Configure Database Connection

We configure how the backend will connect to PostgreSQL. Note that `"db"` will later match the Docker service name.

**File: `backend/src/db.js`**

```js
const { Pool } = require("pg");
const pool = new Pool({
  host: "db",
  user: "postgres",
  password: "postgres",
  database: "mydb",
  port: 5432,
});
module.exports = pool;
```

---

### Step 13: Create Routes File

Separating routes improves readability and keeps the main server file clean.

```sh
touch src/routes.js
```

---

### Step 14: Add CRUD Endpoints

These endpoints allow clients to Create, Read, Update, and Delete data in the database.

**File: `backend/src/routes.js`**

```js
const express = require("express");
const router = express.Router();
const pool = require("./db");
// Create
router.post("/items", async (req, res) => {
  const { name } = req.body;
  const result = await pool.query(
    "INSERT INTO items (name) VALUES ($1) RETURNING *",
    [name],
  );
  res.json(result.rows[0]);
});
// Read
router.get("/items", async (req, res) => {
  const result = await pool.query("SELECT * FROM items");
  res.json(result.rows);
});
// Update
router.put("/items/:id", async (req, res) => {
  const { id } = req.params;
  const { name } = req.body;
  const result = await pool.query(
    "UPDATE items SET name=$1 WHERE id=$2 RETURNING *",
    [name, id],
  );
  res.json(result.rows[0]);
});
// Delete
router.delete("/items/:id", async (req, res) => {
  const { id } = req.params;
  await pool.query("DELETE FROM items WHERE id=$1", [id]);
  res.send("Deleted");
});
module.exports = router;
```

---

### Step 15: Connect Routes to Server

This tells Express to use our routes under the `/api` path.

**File: `backend/src/index.js` (add below existing code)**

```js
const routes = require("./routes");
app.use("/api", routes);
```

---

### Step 16: Create Dockerfile

The Dockerfile defines how to build the backend container image.

```sh
touch Dockerfile
```

---

### Step 17: Add Dockerfile Content

This file installs dependencies and prepares the app to run inside a container.

**File: `backend/Dockerfile`**

```dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

---

### Step 18: Return to Root Directory

We move back to define multi-container orchestration.

```sh
cd ..
```

---

### Step 19: Create Docker Compose File

Docker Compose lets us run both the backend and database together.

```sh
touch docker-compose.yml
```

---

### Step 20: Add Docker Compose Configuration

This defines two services: database (`db`) and backend (`backend`).

**File: `docker-compose.yml`**

```yaml
version: "3.8"
services:
  db:
    image: postgres:15
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
  backend:
    build: ./backend
    ports:
      - "3000:3000"
    depends_on:
      - db
```

---

### Step 21: Start Containers

This builds and launches both services.

```sh
docker-compose up --build
```

**Expected Output:**

- Logs showing PostgreSQL initializing
- Logs showing Node server starting
- No red error messages

---

### Step 22: Connect to Database

We now need to access the running PostgreSQL container to create our table.
First, list all running containers:

```sh
docker ps
```

**What to look for:**

- A container with image `postgres:15`
- A name like `simple-backend-tutorial-db-1`  
  Example output:

```text
CONTAINER ID   IMAGE         NAMES
abc123def456   postgres:15   simple-backend-tutorial-db-1
```

The **CONTAINER ID** (e.g., `abc123def456`) is your `<db_container_id>`.
Now connect:

```sh
docker exec -it <db_container_id> psql -U postgres -d mydb
```

---

### Step 23: Create Table

This table will store our data.

```sql
CREATE TABLE items (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL
);
```

**Expected Output:**

- `CREATE TABLE`

---

### Step 24: Test API - Create Item

We insert a new record via HTTP.

```sh
curl -X POST http://localhost:3000/api/items \
-H "Content-Type: application/json" \
-d '{"name":"Test Item"}'
```

**Expected Output:**

```json
{ "id": 1, "name": "Test Item" }
```

---

### Step 25: Test API - Get Items

We retrieve all records from the database.

```sh
curl http://localhost:3000/api/items
```

**Expected Output:**

```json
[{ "id": 1, "name": "Test Item" }]
```

---

### Step 26: Test API - Update Item

We modify an existing record.

```sh
curl -X PUT http://localhost:3000/api/items/1 \
-H "Content-Type: application/json" \
-d '{"name":"Updated Item"}'
```

**Expected Output:**

```json
{ "id": 1, "name": "Updated Item" }
```

---

### Step 27: Test API - Delete Item

We remove a record.

```sh
curl -X DELETE http://localhost:3000/api/items/1
```

**Expected Output:**

```text
Deleted
```

---

### Step 28: Stop Containers

This shuts everything down cleanly.

```sh
docker-compose down
```

---

### Step 29: Create `.env` File and Add Environment Variables

Using environment variables helps avoid hardcoding sensitive information and makes your app easier to configure.
You must create a file named `.env` inside the `backend` folder.

**File: `backend/.env`**

```env
DB_HOST=db
DB_USER=postgres
DB_PASSWORD=postgres
DB_NAME=mydb
DB_PORT=5432
```

---

### Step 30: Refactor DB Config to Use `.env`

Now we update our database connection file to use the environment variables instead of hardcoded values.

**File: `backend/src/db.js` (replace contents)**

```js
require("dotenv").config();
const { Pool } = require("pg");
const pool = new Pool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  port: process.env.DB_PORT,
});
module.exports = pool;
```

---

### Step 31: Rebuild Containers

We rebuild so changes take effect.

```sh
docker-compose up --build
```

---

### Step 32: Final Verification

We confirm everything still works after refactoring.
Visit:

```text
http://localhost:3000/api/items
```

**Expected Output:**

- JSON array (possibly empty `[]` if no items exist)

---

## Final Result

You now have:

- A containerized backend server
- A PostgreSQL database running in Docker
- Fully working CRUD endpoints
- A reproducible development environment

---

## Next Steps

Try extending this project:

- Add input validation
- Use an ORM like Prisma
- Add authentication (JWT)
- Deploy to a cloud provider

---

## Closing Thought

Each piece you built serves a purpose:

- Docker ensures consistency
- Express handles HTTP requests
- PostgreSQL manages data
- Docker Compose connects everything  
  Understanding how these pieces interact is the foundation of modern backend development.
