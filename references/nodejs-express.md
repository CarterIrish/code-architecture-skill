# Node.js / Express Architecture Reference

## Common Architectural Patterns

### MVC / Route-Controller-Service

The bread and butter of Express/Node.js apps. Separate concerns into:

- **Routes** — Define endpoints and map to controllers.
- **Controllers** — Handle request/response logic, validate input, call services.
- **Services** — Contain business logic, interact with data layer.
- **Models** — Define data schemas and database interaction.

Best for: REST APIs, server-rendered apps, CRUD-heavy applications.

### API + SPA

Decoupled backend API serving a frontend single-page application.

- Backend: Express/Fastify API with JSON responses.
- Frontend: React, Vue, or vanilla JS consuming the API.
- Communication: REST or GraphQL.

Best for: Interactive dashboards, apps with complex client-side state.

### Middleware Pipeline

Express's core pattern — chain middleware functions for cross-cutting concerns:

- Authentication (JWT, session, Duo MFA)
- Rate limiting
- Input validation/sanitization
- Error handling (centralized error middleware)
- Logging

### Event-Driven / Pub-Sub

Use when components need loose coupling:

- Node.js EventEmitter for in-process events.
- Redis pub/sub or message queues (Bull, RabbitMQ) for cross-service.
- WebSockets (Socket.io) for real-time client updates.

## Pattern Choices

These are architectural decisions where multiple approaches are equally valid. Don't default to one — present the options during the interview and let the user choose based on their preference and comfort level. Each choice is labeled with a short name for easy reference.

### Routing Style

Present when the project has more than 3-4 routes.

**A) Router-as-function** — A single `router.js` exports a function that takes the Express `app` and maps all routes. Simple, all routes visible in one file.

```js
// router.js
const controllers = require("./controllers");
module.exports = (app) => {
  app.get("/api/cats", controllers.getCats);
  app.post("/api/cats", controllers.addCat);
};
```

Best for: Small projects (<10 routes), users who like seeing all routes in one place.

**B) Express Router per resource** — Each resource/feature gets its own Router file, mounted in the entry point. Standard Express pattern.

```js
// routes/cats.js
const router = require("express").Router();
router.get("/", getCats);
router.post("/", addCat);
module.exports = router;
// app.js: app.use('/api/cats', require('./routes/cats'));
```

Best for: Medium+ projects, multiple route groups, teams. Most Express tutorials teach this.

**C) File-based routing** — Route files auto-discovered by convention (like Next.js API routes). Requires a small loader script or a library like `express-file-routing`.
Best for: Users coming from Next.js/Nuxt who prefer convention over configuration. Less common in vanilla Express projects.

### Code Style

Present when the project involves controllers, services, or models — anywhere there's a choice between OOP and functional patterns.

**A) Functional / closures** — Export plain functions. No classes, no `this`. State passed explicitly as arguments or closed over.

```js
// controllers/cats.js
const getCats = async (req, res) => { ... };
const addCat = async (req, res) => { ... };
module.exports = { getCats, addCat };
```

Best for: Users who prefer simplicity, avoid `this` confusion, most Node.js/Express conventions.

**B) Class-based** — Group related methods in a class. Use `this` for shared state/dependencies.

```js
class CatController {
  constructor(catService) { this.catService = catService; }
  async getCats(req, res) { ... }
}
module.exports = new CatController(catService);
```

Best for: Users from Java/C# backgrounds, projects with dependency injection, larger codebases where method grouping helps navigation.

### Error Handling Strategy

Present for any project with async route handlers.

**A) Try/catch per handler** — Each controller wraps its own async logic in try/catch. Explicit, easy to follow, easy to customize per-route.

```js
const getCats = async (req, res) => {
  try { ... }
  catch (err) { res.status(500).json({ error: 'Failed to fetch cats' }); }
};
```

Best for: Small projects, beginners, when each route needs different error responses.

**B) Async wrapper + centralized error middleware** — A higher-order function catches errors and passes them to Express's error middleware. Less repetition, consistent error format.

```js
const asyncHandler = (fn) => (req, res, next) => fn(req, res, next).catch(next);
router.get("/cats", asyncHandler(getCats));
// Error middleware in app.js handles all errors uniformly
```

Best for: Projects with many routes where consistent error handling matters. Slightly more abstract.

### Module System

Present when starting a new project (not restructuring an existing one).

**A) CommonJS (`require` / `module.exports`)** — The Node.js default. Works everywhere, no configuration needed.
**B) ES Modules (`import` / `export`)** — Modern JavaScript standard. Requires `"type": "module"` in package.json or `.mjs` extensions. Enables tree-shaking, aligns with browser JS.

Best default: CommonJS unless the user specifically prefers ES Modules or the project needs to share code with a frontend that uses imports.

## Tech Stack Patterns

### Node.js / Express Stack

```
project/
├── src/
│   ├── routes/          # Express route definitions
│   ├── controllers/     # Request handlers
│   ├── services/        # Business logic
│   ├── models/          # Database models/schemas
│   ├── middleware/       # Auth, validation, error handling
│   ├── config/          # Environment config, DB connection
│   └── utils/           # Shared helpers
├── tests/
├── package.json
└── .env
```

### Database Selection Guide

- **PostgreSQL** — Relational data with complex queries, joins, transactions. Default recommendation for most projects.
- **MongoDB** — Document-oriented, flexible schemas, good for rapid prototyping or genuinely document-shaped data.
- **SQLite** — Embedded, zero-config, great for local tools, CLIs, and small self-hosted apps.
- **Redis** — Caching layer, session store, pub/sub. Complement to a primary DB, not a replacement.

### Authentication Patterns

- **Session-based** (express-session + PostgreSQL/Redis store) — Simple, stateful, good for server-rendered or same-origin apps.
- **JWT** — Stateless, good for decoupled API + SPA, mobile clients.
- **OAuth/SSO** — For third-party login or enterprise integration.
- **MFA** (Duo, TOTP) — Layer on top of any primary auth method.

### Process Management & Deployment

- **PM2** — Process manager for Node.js, handles clustering, restarts, log management.
- **Docker** — Containerized deployment, consistent environments.
- **Reverse proxy** (Nginx, Caddy) — TLS termination, static file serving, load balancing.
- **Cloudflare Tunnels** — Expose local services without port forwarding.

## Common Pitfalls (Web)

- **Fat controllers**: Business logic bleeding into route handlers. Push logic into services.
- **Missing error handling middleware**: Unhandled promise rejections crash the server. Always have a centralized error handler.
- **Hardcoded config**: Use environment variables via `.env` and a config module. Never commit secrets.
- **No input validation**: Validate and sanitize all user input at the controller/middleware layer.
- **Monolith that should be modular**: Even in a single codebase, organize by feature or domain, not by file type alone, as the project grows.
- **Ignoring connection pooling**: Use a connection pool for PostgreSQL (e.g., `pg` Pool). One connection per request doesn't scale.

## API Versioning Patterns

- **URL-based versioning** (`/api/v1/books`, `/api/v2/books`) — Simple, explicit, easy to route. Best for most projects. Use Express Router mounting: `app.use('/api/v1', v1Router)`.
- **Header-based versioning** (`Accept: application/vnd.myapi.v2+json`) — Cleaner URLs but harder to test in a browser. Better for public APIs with many consumers.
- **No versioning** — Fine for personal/internal APIs with one consumer. Don't add versioning infrastructure until you actually have two versions. This is the default recommendation for solo projects.

When to version: When you need to make breaking changes to an endpoint's request/response shape while keeping old consumers working. If you only have one consumer (your own frontend), just change both sides together.

## Rate Limiting Patterns

- **express-rate-limit** — Simple in-memory rate limiting. Good enough for most self-hosted projects.
  - Per-IP limiting for public endpoints.
  - Per-user limiting for authenticated endpoints (key by session/user ID).
  - Apply selectively: auth endpoints get strict limits (5 attempts/15 min), read endpoints get generous limits (100/min), write endpoints in between.
- **Redis-backed rate limiting** (rate-limit-redis) — Required when running multiple instances behind a load balancer. Overkill for a single-server deployment.
- **Reverse proxy rate limiting** (Nginx/Caddy) — Handle rate limiting at the proxy layer instead of in Express. Good when you already have a reverse proxy and want to keep the app simple.

Default recommendation: `express-rate-limit` with in-memory store for self-hosted single-instance apps. Move to Redis-backed only when you horizontally scale.

## WebSocket Architecture

### When to Use WebSockets

- **Real-time dashboards** — Server pushes updates to the client (server status, metrics, live data).
- **Collaborative features** — Multiple users editing the same resource.
- **Chat or notification systems** — Bidirectional, low-latency messaging.
- Don't use for: Data that changes infrequently (poll instead), one-shot request/response (use REST).

### Socket.io Integration with Express

```
project/
├── src/
│   ├── routes/          # REST endpoints (Express)
│   ├── sockets/         # WebSocket event handlers
│   │   ├── index.js     # Socket.io setup, namespace registration
│   │   └── handlers/    # Per-namespace event handlers
│   ├── services/        # Shared business logic (used by both REST and WS)
│   ├── middleware/       # Express middleware
│   └── config/
```

Key pattern: REST and WebSocket handlers should share the same service layer. Don't duplicate business logic — routes call services for request/response, socket handlers call services for real-time events.

### Authentication with WebSockets

- Share the Express session with Socket.io using `express-session` and a shared session store.
- Authenticate on connection (in the `io.use()` middleware), not on every message.
- Reject unauthenticated connections at the handshake level.

## Monorepo Patterns (Node.js)

- **npm workspaces** — Built into npm 7+. Good for 2-5 packages sharing code. Define workspaces in root `package.json`, shared dependencies are hoisted.
- **When to use**: You have a shared library used by both an API and a CLI tool, or a backend and frontend that share types/validators.
- **When NOT to use**: Single-purpose projects. A personal API with no shared consumers doesn't need a monorepo. The overhead isn't justified until you have at least two packages that share code.
