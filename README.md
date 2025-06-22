# time-attack

### 🧱 Core Collections

#### 1. `users` *(built-in)*
- Handles auth (email/password).
- Add optional fields like `full_name`, `role` (`freelancer`, `client`), etc.

#### 2. `clients`
```json
{
  "name": "Acme Corp",
  "email": "contact@acme.com",
  "user": "relation -> users"
}
```

#### 3. `projects`
```json
{
  "name": "Website Redesign",
  "client": "relation -> clients",
  "hourly_rate": 500,
  "user": "relation -> users"
}
```

#### 4. `time_logs`
```json
{
  "start_time": "datetime",
  "end_time": "datetime",
  "duration_minutes": "number",
  "project": "relation -> projects",
  "notes": "text",
  "tags": "json[]",
  "user": "relation -> users"
}
```

Manual entries populate `start_time`, `end_time`, and `duration_minutes`. Live timers calculate duration dynamically.

#### 5. `invoices`
```json
{
  "project": "relation -> projects",
  "time_logs": "relation -> time_logs (multi)",
  "total_amount": "number",
  "status": "select -> [draft, sent, paid]",
  "user": "relation -> users"
}
```

---

### 🧪 Feature Breakdown

#### ✅ Auth
Use Pocketbase SDK:
```js
await pb.collection("users").authWithPassword(email, password);
```

#### ⏱ Timer / Manual Logging
For timers:
```js
const startTime = new Date();
// Later…
const endTime = new Date();
const duration = (endTime - startTime) / 60000; // in minutes
```

Post to `time_logs` with this data + project ID.

#### 📅 Timesheets
Use filters like:
```js
await pb.collection("time_logs").getFullList({
  filter: `user.id = "${pb.authStore.model.id}" && created >= "2025-06-01" && created <= "2025-06-30"`
});
```

Aggregate by project or tag client-side. You can export using a simple CSV generator or trigger PDF generation via a Netlify function or similar.

#### 💸 Invoicing
To generate an invoice:
- Select all `time_logs` for a project
- Sum `duration_minutes * project.hourly_rate`
- Link them into a new `invoices` record

Mark invoice status changes with a basic dropdown.

---

### 💡 Optional Enhancements for Later
- Detect inactivity (e.g., if user didn't move mouse for X mins, suggest pausing the timer)
- Add `subtasks` under projects
- Create a global dashboard with weekly goals or income predictions
