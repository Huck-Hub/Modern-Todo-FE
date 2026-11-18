# LinearFlow — Modern Full-Stack Todo & Workflow Manager (MEVN)

A 2026-grade **Full-Stack MEVN (MongoDB, Express.js, Vue.js 3, Node.js)** Task & Workflow Manager inspired by **Linear**, **Superlist**, and **Todoist**, built using the exact vanilla patterns from **CST3144 Full Stack Development**.

---

## 1. What Makes a Modern 2026 Todo App (vs. a Basic Checklist)?

1. **Multi-View Workspace (List View + 3-Column Kanban Board):**
   * Switch seamlessly between a high-density **List View** and a 3-column **Kanban Board** (`To Do`, `In Progress`, `Completed`) using Vue's `v-if` / `v-else`.
2. **Command-Bar Quick-Add with Rich Metadata:**
   * Tasks aren't just text strings—each task tracks **Priority (`P1 - Urgent`, `P2 - Medium`, `P3 - Low`)**, **Category (`University`, `Engineering`, `Design`, `Personal`)**, **Due Date**, **Time Estimate (`1h`, `45m`)**, and **Workflow Status (`todo`, `in-progress`, `done`)**.
3. **Real-Time Workspace Analytics & Progress Bar (`computed` properties):**
   * Automatically computes **Total Tasks**, **Active Tasks**, **Completed Tasks**, **Urgent (P1) Tasks**, and a live **Completion Percentage Progress Bar** (`v-bind:style="{ width: completionPercentage + '%' }"`).
4. **Full-Stack Live Search + Multi-Filter + Multi-Attribute Sorting:**
   * Typing in the search bar triggers a Vue `watch` that queries `GET /search?q=...` across title, category, priority, status, and due date.
   * Combine Category filters, Status filters, and ascending/descending sorting by Priority, Due Date, Title, or Category.
5. **Complete CRUD REST API (`GET`, `POST`, `PUT`, `DELETE`):**
   * Connects to **MongoDB Atlas** (`ModernTodoDB` -> `tasks` collection) via the native MongoDB Node.js driver, with automatic local fallback so it also works offline on GitHub Pages out of the box.

---

## 2. UI/UX Theme: "Linear / Superlist Precision Dark-Mode Craft"

* **Canvas & Surfaces:** Deep carbon canvas (`#08090a`) paired with layered dark surfaces (`#0f1011`, `#141517`, `#18191a`) and hairline borders (`1px solid rgba(255, 255, 255, 0.07)`).
* **Chromatic Accents:**
  * **Linear Lavender-Blue (`#5e6ad2`):** Focus rings, active view buttons, and primary actions.
  * **Emerald Green (`#32d583`):** Completed checkboxes and progress bar completion gradient.
  * **Semantic Priority Badges:** Crimson (`#f04438`) for `P1 - Urgent`, Amber (`#f59e0b`) for `P2 - Medium`, and Muted Slate (`#9ba1a6`) for `P3 - Low`.

---

## 3. Line-by-Line Code Explanation

### A. Back-End (`backend/server.js`)
* **Lines 1–18 (`express`, `cors`, `dotenv`, `mongodb`):**
  * Loads environment variables from `config.env`, configures `MongoClient` and `ObjectId` from the native `mongodb` driver, and loads `tasks.json` as an automatic fallback when `config.env` hasn't been linked to Atlas yet.
* **Lines 22–37 (Middleware Stack):**
  * `app.use(cors())` enables Cross-Origin requests from GitHub Pages or Live Preview.
  * `app.use(express.json())` parses JSON bodies on `POST` and `PUT` requests.
  * **Logger Middleware** outputs every request's method, URL, and ISO timestamp to the server console and calls `next()`.
  * **Static File Middleware** serves `/images` from `staticFiles/` and returns a `404` `"Image not found!"` error if a requested file is missing.
* **Lines 48–137 (Full CRUD REST API Routes):**
  * `GET /tasks`: Fetches all tasks from `db.collection("tasks").find({}).toArray()`.
  * `GET /search?q=...`: Filters tasks on the server across `title`, `category`, `priorityLabel`, `status`, and `dueDate`.
  * `POST /tasks`: Inserts a new task document into MongoDB using `collection.insertOne(body)`.
  * `PUT /tasks/:_id`: Updates any task field (`completed`, `status`, `priority`, etc.) by ID using `new ObjectId(tempid)` and `collection.updateOne(query, { $set: request.body })`.
  * `DELETE /tasks/:_id`: Removes a task document from MongoDB using `collection.deleteOne({ _id: new ObjectId(tempid) })`.

### B. Front-End (`frontend/index.html`)
* **Header & Progress Bar (Lines 318–357):**
  * Displays `totalCount`, `activeCount`, `completedCount`, and `urgentCount` computed properties.
  * Includes the **List / Kanban** view switcher (`v-on:click="viewMode = 'list'"` / `'board'`) and the dynamic progress bar (`v-bind:style="{ width: completionPercentage + '%' }"`).
* **Quick-Add Command Bar (Lines 360–389):**
  * Uses `v-model` and `v-model.number` to bind `newTask.title`, `newTask.category`, `newTask.priority`, `newTask.dueDate`, and `newTask.estimate`.
  * Disables the **"Add Task"** button via `v-bind:disabled="!isValidTask"` until the title has at least 3 characters.
* **Search, Filter & Sort Bar (Lines 392–425):**
  * Binds `searchQuery` (watched by Vue `watch`), `filterCategory`, `filterStatus`, `sortBy`, and `sortOrder`.
* **View 1 — High-Density List View (`v-if="viewMode === 'list'"`, Lines 428–465):**
  * Renders each task in `sortedTasks` using `v-for="task in sortedTasks"`.
  * Checkbox (`v-bind:checked="task.completed"`) calls `toggleComplete(task)`, which flips `task.completed`, updates `task.status`, and sends a `PUT` request to `/tasks/:_id`.
  * Inline status dropdown (`v-model="task.status" v-on:change="updateStatus(task)"`) and delete button (`v-on:click="deleteTask(task)"` -> `DELETE /tasks/:_id`).
* **View 2 — 3-Column Kanban Board (`v-else`, Lines 468–534):**
  * Groups `sortedTasks` into `To Do` (`tasksByStatus('todo')`), `In Progress` (`tasksByStatus('in-progress')`), and `Completed` (`tasksByStatus('done')`) columns with one-click workflow transition buttons (`Start ->`, `Complete`, `Reopen`).
