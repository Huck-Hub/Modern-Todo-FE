# LinearFlow — Modern Full-Stack Todo & Workflow Manager (MEVN)

A 2026-grade **Full-Stack MEVN (MongoDB, Express.js, Vue.js 3, Node.js)** Task & Workflow Manager inspired by **Linear**, **Superlist**, and **Todoist**, built using the exact vanilla patterns from **CST3144 Full Stack Development** — now featuring a **Live Browser Alarm & Reminder System** powered by the Web Audio API and Desktop Notifications API.

---

## 1. What Makes `LinearFlow` a Complete 2026 Todo App?

1. **Multi-View Workspace (List View + 3-Column Kanban Board):**
   * Switch seamlessly between a high-density **List View** and a 3-column **Kanban Board** (`To Do`, `In Progress`, `Completed`) using Vue's `v-if` / `v-else`.
2. **Real-Time Audio Alarm & Reminder System (`reminderTime` + Web Audio API):**
   * Attach an optional date/time alarm (`datetime-local`) when creating or editing a task, or click the **Bell Icon (`fa-bell-o`)** on any task card to open the **Quick Alarm Drawer** with 1-click presets (`In 10 sec (Demo)`, `+1 min`, `+5 min`, `+15 min`, `+1 hr`).
   * **Background 1-Second Watcher (`checkAlarms()`):** Runs inside Vue's `created()` hook via `setInterval` and checks if any uncompleted task has reached its `reminderTime`.
   * **Synthesized Two-Tone Digital Alarm Chime (`playAlarmChime()`):** Uses the browser's built-in `AudioContext` (`880 Hz` A5 -> `1174.66 Hz` D6 sine wave oscillators) so the alarm rings reliably offline or online without needing external `.mp3` files.
   * **OS Desktop Notifications (`sendDesktopNotification()`):** Triggers a native desktop notification banner via the browser `Notification` API.
   * **Interactive Ringing Alarm Modal Banner:** Pulses at the top of the workspace while the alarm rings, letting you click **Snooze +5m**, **Mark Done**, or **Dismiss**, plus a **Test Alarm Sound** button in the top bar for instant demoing.
3. **Inline Task Editing (`PUT /tasks/:_id`) & Nested Subtasks Checklist:**
   * Click the pencil icon to edit any task's title, category, priority, due date, alarm time, or time estimate in-place.
   * Expand any task's checklist (`0/3`) to check off individual subtask steps or add new steps (`+ Step`).
4. **Overdue Date Detection, Bulk "Clear Done", & `localStorage` Persistence:**
   * Tasks past their `dueDate` automatically highlight in crimson (`Overdue • YYYY-MM-DD`).
   * One-click **Clear Done** button (`DELETE /tasks/completed`) removes all completed tasks at once.
   * Automatically persists to `localStorage` (`linearflow_tasks_v1`) so all edits, subtasks, and alarms survive page refreshes even on GitHub Pages when the backend is offline.
5. **Full-Stack Live Search + Multi-Filter + Multi-Attribute Sorting:**
   * Typing in the search bar triggers a Vue `watch` that queries `GET /search?q=...` across title, category, priority, status, and due date.

---

## 2. UI/UX Theme: "Linear / Superlist Precision Dark-Mode Craft"

* **Canvas & Surfaces:** Deep carbon canvas (`#08090a`) paired with layered dark surfaces (`#0f1011`, `#141517`, `#18191a`) and hairline borders (`1px solid rgba(255, 255, 255, 0.07)`).
* **Chromatic Accents:**
  * **Linear Lavender-Blue (`#5e6ad2`):** Focus rings, active view buttons, and primary actions.
  * **Emerald Green (`#32d583`):** Completed checkboxes and progress bar completion gradient.
  * **Semantic Priority & Alarm Badges:** Crimson (`#f04438`) for `P1 - Urgent` and `Overdue`, Amber (`#f59e0b`) for `P2 - Medium` and active **Alarm Reminders**.

---

## 3. Line-by-Line Code Explanation

### A. Back-End (`backend/server.js`)
* **Lines 1–18 (`express`, `cors`, `dotenv`, `mongodb`):**
  * Loads environment variables from `config.env`, configures `MongoClient` and `ObjectId` from the native `mongodb` driver, and loads `tasks.json` as an automatic fallback.
* **Lines 22–37 (Middleware Stack):**
  * `app.use(cors())` enables Cross-Origin requests from GitHub Pages or Live Preview.
  * `app.use(express.json())` parses JSON bodies on `POST` and `PUT` requests.
  * **Logger Middleware** outputs every request's method, URL, and ISO timestamp to the server console and calls `next()`.
  * **Static File Middleware** serves `/images` from `staticFiles/` and returns a `404` `"Image not found!"` error if a requested file is missing.
* **Lines 48–150 (Full CRUD REST API Routes):**
  * `GET /tasks`: Fetches all tasks from `db.collection("tasks").find({}).toArray()`.
  * `GET /search?q=...`: Filters tasks on the server across `title`, `category`, `priorityLabel`, `status`, and `dueDate`.
  * `POST /tasks`: Inserts a new task document (including `reminderTime`, `reminderFired`, and `subtasks`) into MongoDB using `collection.insertOne(body)`.
  * `PUT /tasks/:_id`: Updates any task field (`completed`, `status`, `priority`, `reminderTime`, `reminderFired`, `subtasks`) by ID using `new ObjectId(tempid)` and `collection.updateOne(query, { $set: request.body })`.
  * `DELETE /tasks/completed`: Bulk-deletes all completed tasks via `collection.deleteMany({ completed: true })`.
  * `DELETE /tasks/:_id`: Removes a single task document from MongoDB using `collection.deleteOne({ _id: new ObjectId(tempid) })`.

### B. Front-End (`frontend/index.html`)
* **Header, Alarm Test Button & Ringing Alarm Banner:**
  * Displays `totalCount`, `activeCount`, `completedCount`, and `activeAlarmsCount` computed properties.
  * Clicking **Test Alarm Sound** triggers `triggerTestAlarm()`, which immediately plays the Web Audio API two-tone alarm chime and opens the pulsing **ALARM REMINDER** banner (`v-if="ringingTasks.length > 0"`).
  * When an alarm is ringing, the user can click **Snooze +5m** (`snoozeAlarm(task, 5)`), **Mark Done** (`completeFromAlarm(task)`), or **Dismiss** (`dismissAlarm(task)`).
* **Quick-Add Command Bar:**
  * Uses `v-model` and `v-model.number` to bind `newTask.title`, `newTask.category`, `newTask.priority`, `newTask.dueDate`, `newTask.reminderTime` (`type="datetime-local"`), and `newTask.estimate`.
* **Per-Task Alarm Bell Drawer (`toggleAlarmPanel(task)`):**
  * Clicking the bell icon on any task opens the inline alarm bar where you can pick an exact date/time or click **`In 10 sec (Demo)`**, **`+1 min`**, **`+5 min`**, **`+15 min`**, or **`+1 hr`** (`setQuickAlarm(task, minutes)`).
* **Web Audio API Synthesizer (`playAlarmChime()`):**
  * Creates a native `AudioContext`, schedules four alternating sine-wave tones (`880 Hz` and `1174.66 Hz`), and loops every `2.4` seconds via `startLoopingAlarmSound()` until the user snoozes or dismisses the alarm.
