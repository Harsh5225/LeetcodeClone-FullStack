# CodeBuddy — Collaborative Coding Platform

[![Node.js](https://img.shields.io/badge/Node.js-18%2B-339933?logo=node.js&logoColor=white)](https://nodejs.org/)
[![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)](https://react.dev/)
[![MongoDB](https://img.shields.io/badge/MongoDB-Database-47A248?logo=mongodb&logoColor=white)](https://www.mongodb.com/)
[![Socket.IO](https://img.shields.io/badge/Socket.IO-Realtime-black?logo=socket.io)](https://socket.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

A full-stack collaborative coding platform for practicing algorithmic problem-solving in real time — with a Monaco-based
editor, live multi-user collaboration, AI-assisted debugging, and an admin panel for problem management.

<!-- Optional: add a screenshot or GIF here once available -->
<!-- ![CodeBuddy demo](./docs/demo.gif) -->

## Table of Contents
- [Key Features](#key-features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [API Reference](#api-reference)
- [Socket.IO Events](#socketio-events)
- [Security](#security)
- [Performance](#performance-optimizations)
- [Testing](#testing)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

## Key Features

**Authentication & User Management**
- JWT-based auth with Redis session storage and token blacklisting on logout
- Role-based access control (User / Admin)
- Editable profiles with photo upload

**Problem-Solving Environment**
- Monaco Editor with syntax highlighting across JavaScript, Java, C++, and Python
- Real-time code execution and evaluation via the Judge0 API
- Visible and hidden test cases with detailed feedback

**Real-Time Collaboration**
- Live collaborative editing with typing indicators and cursor sharing
- Room-based sessions with online/away/busy/offline presence tracking
- In-room chat alongside the shared editor

**AI-Powered Assistance**
- Google Gemini integration for contextual hints and debugging help
- Step-by-step solution explanations

**Educational Content & Admin Panel**
- Video solution uploads (Cloudinary) with editorial write-ups
- Problem CRUD, bulk operations, and basic user/submission analytics for admins

## Architecture

```
Frontend/                       Backend/
├── src/                        ├── controllers/       # Business logic
│   ├── components/             ├── models/             # MongoDB schemas
│   ├── pages/                  ├── routes/             # API endpoints
│   ├── features/  (Redux)      ├── middlewares/        # Auth, roles, rate limiting
│   ├── hooks/                  ├── socket/             # Socket.IO handlers
│   └── utils/                  ├── utils/              # Judge0, Cloudinary, validation
                                 └── database/           # MongoDB + Redis connections
```

## Tech Stack

| Layer | Technologies |
|---|---|
| **Frontend** | React 19, Vite, Redux Toolkit, React Router, Tailwind CSS + DaisyUI, Monaco Editor, Socket.IO Client, Framer Motion, React Hook Form + Zod |
| **Backend** | Node.js, Express, MongoDB + Mongoose, Redis, Socket.IO, JWT, Multer |
| **External Services** | Judge0 CE (code execution), Cloudinary (media storage), Google Gemini (AI assistance) |

## Getting Started

### Prerequisites
- Node.js v18+
- MongoDB
- Redis
- Judge0 API access, a Cloudinary account, and a Google Gemini API key

### Installation

```bash
git clone https://github.com/Harsh5225/CodeBuddy.git
cd CodeBuddy

# Backend
cd Backend && npm install

# Frontend
cd ../Frontend && npm install
```

### Environment Variables

**`Backend/.env`**
```env
MONGODB_URI=mongodb://localhost:27017/codebuddy

REDIS_HOST=your-redis-host
REDIS_PASSWORD=your-redis-password

JWT_SECRET=your-jwt-secret

RAPIDAPI_KEY=your-rapidapi-key
RAPIDAPI_HOST=judge0-ce.p.rapidapi.com

CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret

GEMINI_KEY=your-gemini-api-key

PORT=3000
NODE_ENV=development
```

**`Frontend/.env`**
```env
VITE_API_BASE_URL=http://localhost:3000
```

### Run Locally

```bash
# Terminal 1 — Backend
cd Backend && npm run dev

# Terminal 2 — Frontend
cd Frontend && npm run dev
```

- Frontend: `http://localhost:5173`
- Backend API: `http://localhost:3000`

## API Reference

**Authentication**
```
POST   /user/register
POST   /user/login
GET    /user/logout
GET    /user/check
GET    /user/profile
PUT    /user/edit-profile
```

**Problems**
```
GET    /problem                      # paginated list
GET    /problem/:id
POST   /problem/create               # admin
PATCH  /problem/:id                  # admin
DELETE /problem/:id                  # admin
GET    /problem/userSolvedProblem
```

**Submissions**
```
POST   /submission/run/:id           # run against visible test cases
POST   /submission/submit/:id        # full evaluation
GET    /submission/recent
```

**Collaboration**
```
POST   /collaboration/rooms/create
GET    /collaboration/rooms/:roomId
GET    /collaboration/rooms
```

**AI Assistant**
```
POST   /ai/chat
```

**Video (Admin)**
```
GET    /video/create/:problemId      # generate upload signature
POST   /video/save
DELETE /video/delete/:problemId
```

## Socket.IO Events

**Client → Server**
```javascript
socket.emit('join-room', { roomId, userId, userName, problemId })
socket.emit('leave-room', { roomId })
socket.emit('code-change', { roomId, code, changes })
socket.emit('language-change', { roomId, language })
socket.emit('cursor-change', { roomId, position })
socket.emit('typing-start', { roomId })
socket.emit('typing-stop', { roomId })
socket.emit('status-change', { roomId, status })
socket.emit('send-message', { roomId, message })
socket.emit('share-execution', { roomId, result, type })
```

**Server → Client**
```javascript
socket.on('room-state', { code, language, users, messages, typingUsers })
socket.on('user-joined', { user, totalUsers, onlineUsers })
socket.on('user-left', { userId, userName, totalUsers })
socket.on('code-update', { code, changes, userId })
socket.on('language-update', { language })
socket.on('cursor-update', { userId, userName, position })
socket.on('user-typing', { userId, userName, isTyping, typingUsers })
socket.on('user-presence-update', { userId, status, lastActivity })
socket.on('new-message', message)
socket.on('execution-shared', executionMessage)
```

## Security

- JWT-based authentication with Redis-backed token blacklisting
- bcrypt password hashing
- Redis-based rate limiting on sensitive endpoints
- CORS configuration for cross-origin requests
- Request validation via Zod schemas
- Mongoose ODM (parameterized queries, no raw query injection surface)

## Performance Optimizations

- **Frontend**: route-based code splitting, memoized components, Vite build optimizations
- **Backend**: MongoDB indexing on frequently queried fields, Redis caching for sessions and rate limits, connection pooling
- **Real-time**: debounced socket events and selective (room-scoped) broadcasting to reduce unnecessary traffic

## Testing

The project uses Jest/Supertest for backend API testing and includes a manual QA checklist covering auth flow, the
problem-solving workflow, real-time collaboration, admin functionality, and cross-browser/mobile behavior. Run
`npm run test` in `Backend/` or `Frontend/` to execute the available test suites.

> Note: test coverage is actively being expanded — see open issues for gaps.

## Roadmap

- WebRTC-based voice/video chat for collaboration rooms
- Peer code review workflow
- Timed contest mode
- React Native mobile app
- Service decomposition into microservices for independent scaling
- GraphQL API layer
- Progressive Web App support

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes and add tests where applicable
4. Open a pull request

Please follow the existing ESLint/Prettier config and use conventional commit messages. Bug reports should include a
clear description, reproduction steps, and expected vs. actual behavior via the GitHub issue tracker.

## License

Licensed under the [MIT License](./LICENSE).

---

Built by [Harsh](https://github.com/Harsh5225)
