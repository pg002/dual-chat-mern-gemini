# Dual Chat — MERN + Gemini AI

A real-time project collaboration and chat application built with the MERN stack, Socket.IO, and Google Gemini. Users can collaborate in project rooms, exchange messages, and mention `@ai` to request AI assistance. The frontend also integrates WebContainer for working with generated project files in the browser.

## Features

- **User authentication:** Register and log in with email and password.
- **JWT-protected APIs:** Middleware verifies tokens before allowing access to protected endpoints.
- **Project management:** Create projects, view projects, retrieve project details, and add users to a project.
- **Real-time project chat:** Socket.IO rooms allow connected users to exchange messages in a project.
- **Gemini AI assistance:** Mention `@ai` in a project message to send a prompt to the Gemini service.
- **AI-generated project files:** The AI service is instructed to return structured JSON that can include a response, file tree, and build/start commands.
- **Browser-based project tooling:** WebContainer integration supports working with generated files.
- **MongoDB persistence:** Stores user and project data.
- **Redis token blacklist:** Used by the logout/authentication flow to reject blacklisted tokens.

## Tech stack

### Frontend

- React 18
- Vite
- React Router
- Axios
- Socket.IO Client
- Tailwind CSS
- WebContainer API
- Highlight.js and Markdown-to-JSX

### Backend

- Node.js
- Express 4
- MongoDB with Mongoose
- Socket.IO
- JSON Web Tokens (`jsonwebtoken`)
- bcrypt
- Redis with ioredis
- Google Generative AI SDK (`@google/generative-ai`)
- dotenv, cookie-parser, CORS, Morgan, and express-validator

## Architecture overview

```text
React + Vite frontend
  ├── Axios ────────────────> Express REST API
  ├── Socket.IO Client ─────> Socket.IO server
  └── WebContainer API          │
                                ├── MongoDB (users and projects)
                                ├── Redis (JWT blacklist)
                                └── Gemini API (@ai prompts)
```

The REST API handles user and project operations. Socket.IO handles live project-room messaging. When a message contains `@ai`, the backend extracts the prompt, calls the Gemini service, and emits the generated response to the project room.

## Project structure

```text
.
├── backend/
│   ├── controllers/       # Request handlers
│   ├── db/                # MongoDB connection
│   ├── middleware/        # Authentication middleware
│   ├── models/            # Mongoose models
│   ├── routes/             # Express routes
│   ├── services/          # User, project, Redis, and AI logic
│   ├── app.js              # Express app and middleware
│   └── server.js           # HTTP and Socket.IO server
└── frontend/
    └── src/
        ├── auth/           # Authentication helpers
        ├── config/         # Axios, Socket.IO, WebContainer config
        ├── context/        # React context
        ├── routes/         # Frontend routes
        └── screens/        # Login, register, home, project screens
```

## Prerequisites

Install or prepare the following before running the app:

- Node.js and npm
- A MongoDB instance (local or hosted)
- A Redis instance
- A Google Gemini API key for AI features

## Getting started

### 1. Clone the repository

```bash
git clone <your-public-repository-url>
cd dual-chat-mern-gemini-main
```

### 2. Configure the backend

```bash
cd backend
npm install
```

Create a `.env` file inside `backend/`:

```env
PORT=3000
MONGODB_URI=mongodb://127.0.0.1:27017/dual-chat
JWT_SECRET=replace_with_a_long_random_secret
GOOGLE_AI_KEY=your_gemini_api_key
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_PASSWORD=
```

Update the MongoDB and Redis values to match your environment. Keep `.env` private and never commit API keys or secrets.

Start the backend:

```bash
node server.js
```

The backend uses port `3000` by default, or the value supplied through `PORT`.

### 3. Configure the frontend

Open a second terminal:

```bash
cd frontend
npm install
```

Create `frontend/.env`:

```env
VITE_API_URL=http://localhost:3000
```

`VITE_API_URL` is used for both Axios API requests and the Socket.IO connection. Restart Vite after changing environment variables.

Start the frontend:

```bash
npm run dev
```

Vite will print the local URL to open in your browser (usually `http://localhost:5173`).

## API overview

The backend mounts these route groups:

| Route | Purpose |
|---|---|
| `POST /users/register` | Register a user |
| `POST /users/login` | Log in |
| `GET /users/profile` | Get the authenticated user's profile |
| `GET /users/logout` | Log out / blacklist the current token |
| `GET /users/all` | List users (authenticated) |
| `POST /projects/create` | Create a project (authenticated) |
| `GET /projects/all` | List the current user's projects (authenticated) |
| `PUT /projects/add-user` | Add users to a project (authenticated) |
| `GET /projects/get-project/:projectId` | Retrieve project details (authenticated) |
| `PUT /projects/update-file-tree` | Update a project's file tree (authenticated) |
| `GET /ai/get-result` | AI result endpoint |

For exact request bodies and response formats, see the route and controller files in `backend/routes/` and `backend/controllers/`.

## How AI chat works

1. A user sends a project message containing `@ai`.
2. The Socket.IO server detects the mention and removes `@ai` from the prompt.
3. `backend/services/ai.service.js` sends the prompt to Gemini.
4. The service returns the generated response text; the frontend expects structured JSON for AI output.
5. The server emits the AI response to sockets in that project room.

The Gemini model name is configured in `backend/services/ai.service.js`. Model availability can change; if the configured model is unavailable, update it to a currently supported model for your API key and SDK.

## Available frontend scripts

Run these commands from `frontend/`:

```bash
npm run dev      # Start the Vite development server
npm run build    # Build the production frontend
npm run preview  # Preview the production build
npm run lint     # Run ESLint
```

## Security and implementation notes

- Never commit `.env` files, API keys, or JWT secrets.
- Use a strong, unpredictable `JWT_SECRET` outside local development.
- Validate and safely parse AI-generated JSON before rendering or using generated files.
- Treat AI-generated code as untrusted. Review it before executing it.
- Configure CORS and Socket.IO origins for your actual deployment rather than allowing every origin.
- Ensure users are authorised to access the requested project, not merely authenticated.
- The current frontend stores the JWT in local storage; protect against cross-site scripting and consider a secure HttpOnly-cookie design for production.
- The backend's Socket.IO connection should verify both authentication and the user's permission to access the requested project room.
- Configure timeouts and error handling for external Gemini requests.

## Current scope

This repository demonstrates project-based real-time messaging and AI-assisted code generation. Do not assume persistent chat history, message delivery/read receipts, or production deployment automation unless you implement those features separately.

## License

No project-specific open-source license is included. Add a `LICENSE` file if you intend to distribute this project under a particular license.
