# Real-Time-Collaborative-Editor
Company Name: CODTECH IT SOLUTIONS

Name: Vedant Satish Manolkar

Intern ID: CTISAN15

Domain Name: Full Stack Web Development

Batch Duration: 6 Weeks

Mentor Name: Neela Santosh


# CollabDocs — Real-Time Collaborative Document Editor

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENT (React.js)                    │
│  ┌──────────┐  ┌───────────┐  ┌──────────────────────┐ │
│  │ Editor   │  │ Sidebar   │  │ Presence/Cursor Layer │ │
│  │ (textarea│  │ (Users,   │  │ (Live cursors, names) │ │
│  │  + OT)   │  │  History) │  │                      │ │
│  └────┬─────┘  └─────┬─────┘  └──────────────────────┘ │
│       │               │                                  │
│       └───────┬────────┘                                 │
│               │ WebSocket + REST API                     │
└───────────────┼──────────────────────────────────────────┘
                │
┌───────────────▼──────────────────────────────────────────┐
│                   SERVER (Node.js)                        │
│  ┌─────────────────┐    ┌──────────────────────────────┐ │
│  │  Express REST   │    │   WebSocket Server (ws)      │ │
│  │  /api/documents │    │   - Rooms per docId          │ │
│  │  /api/history   │    │   - Broadcast edits          │ │
│  └────────┬────────┘    │   - Cursor sync              │ │
│           │             │   - Presence tracking        │ │
│           └──────┬───────┘                             │ │
│                  │                                      │ │
│         ┌────────▼────────┐                            │ │
│         │    Mongoose ODM │                            │ │
│         └────────┬────────┘                            │ │
└──────────────────┼──────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────┐
│                   MongoDB                                │
│   documents     │   operations (history/OT log)         │
│   ─────────     │   ──────────────────────────          │
│   _id           │   docId                               │
│   title         │   userId / userName                   │
│   content       │   type (insert/delete/replace)        │
│   version       │   content (snapshot)                  │
│   collaborators │   version / timestamp                 │
└──────────────────────────────────────────────────────────┘
```

---

## Tech Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| Frontend | React.js + Vite | Fast reactivity, component model |
| Real-time | WebSocket (ws) | Bi-directional low-latency sync |
| Backend | Node.js + Express | Non-blocking I/O, event-driven |
| Database | MongoDB + Mongoose | Flexible schema, fast writes |
| Concurrency | OT (Operational Transform) | Conflict-free concurrent editing |

---

## How to Execute the Program

### Prerequisites
- Node.js v18+ installed → https://nodejs.org
- MongoDB installed and running → https://www.mongodb.com/try/download/community
- npm (comes with Node.js)

---

### Step 1 — Start MongoDB

```bash
# macOS (with Homebrew)
brew services start mongodb-community

# Ubuntu/Debian
sudo systemctl start mongod

# Windows
net start MongoDB

# OR use MongoDB Atlas (cloud) — just replace MONGO_URI in step 3
```

---

### Step 2 — Set Up the Backend

```bash
# Navigate to backend folder
cd collaborative-editor/backend

# Install dependencies
npm install

# Start the server (development with auto-reload)
npm run dev

# OR start production
npm start
```

You should see:
```
✅ MongoDB connected
🚀 Server running on http://localhost:4000
```

---

### Step 3 — Set Up the Frontend

Open a NEW terminal:

```bash
# Navigate to frontend folder
cd collaborative-editor/frontend

# Install dependencies
npm install

# Start the dev server
npm run dev
```

You should see:
```
  VITE v4.x ready
  Local: http://localhost:3000
```

---

### Step 4 — Open the App

1. Open **http://localhost:3000** in your browser
2. Open the same URL in a **second browser tab/window**
3. Start typing in one tab — see changes appear in the other in real-time!

---

## How Real-Time Sync Works (Explanation)

### WebSocket Message Flow

```
Tab A types "Hello"
    │
    ▼
ws.send({ type: "edit", content: "Hello", version: 5 })
    │
    ▼
Server receives message
    │
    ├── Saves to MongoDB (Document.findByIdAndUpdate)
    ├── Logs to Operations collection
    └── Broadcasts to ALL other clients in same room
              │
              ▼
        Tab B receives { type: "edit", content: "Hello" }
              │
              ▼
        React setState → UI updates instantly
```

### Key Message Types

| Type | Direction | Purpose |
|------|-----------|---------|
| `join` | Client→Server | Enter a document room |
| `init` | Server→Client | Send current doc state to new joiner |
| `edit` | Both | Broadcast content changes |
| `cursor` | Both | Share cursor position |
| `title` | Both | Document title change |
| `user_joined` | Server→Client | Presence notification |
| `user_left` | Server→Client | Presence notification |
| `users` | Server→Client | Full user list on join |

---

## MongoDB Collections

### `documents` collection
```json
{
  "_id": "doc-uuid-here",
  "title": "Meeting Notes",
  "content": "Today we discussed...",
  "version": 42,
  "createdAt": "2024-01-15T10:00:00Z",
  "updatedAt": "2024-01-15T11:30:00Z",
  "collaborators": [
    { "userId": "user-abc", "name": "Alice", "joinedAt": "..." }
  ]
}
```

### `operations` collection (history/audit log)
```json
{
  "docId": "doc-uuid-here",
  "userId": "user-abc",
  "userName": "Alice",
  "type": "replace",
  "content": "full document snapshot",
  "version": 42,
  "timestamp": "2024-01-15T11:30:00Z"
}
```

---

## REST API Endpoints

```
POST   /api/documents          → Create new document
GET    /api/documents          → List all documents
GET    /api/documents/:id      → Get document by ID
GET    /api/documents/:id/history → Get edit history
```

---

## Production Enhancements (Next Steps)

1. **Operational Transform / CRDT**: Replace full-snapshot sync with delta operations (use `sharedb` or `Automerge` library) for true conflict-free editing with multiple simultaneous users.

2. **Authentication**: Add JWT auth so only authenticated users can access documents.

3. **PostgreSQL Alternative**: Replace Mongoose with `pg` + `Sequelize` if you prefer relational storage:
   ```sql
   CREATE TABLE documents (id UUID, title TEXT, content TEXT, version INT);
   CREATE TABLE operations (id UUID, doc_id UUID, user_id UUID, delta JSONB, version INT, created_at TIMESTAMP);
   ```

4. **Redis Pub/Sub**: For scaling to multiple server instances, use Redis to broadcast messages across servers.

5. **Docker Compose**:
   ```yaml
   services:
     mongo: image: mongo:7
     backend: build: ./backend, ports: ["4000:4000"]
     frontend: build: ./frontend, ports: ["3000:3000"]
   ```

---

## File Structure

```
collaborative-editor/
├── backend/
│   ├── server.js          ← Express + WebSocket + MongoDB
│   └── package.json
└── frontend/
    ├── index.html
    ├── vite.config.js
    ├── package.json
    └── src/
        ├── main.jsx       ← React entry point
        └── App.jsx        ← Main editor component
```
