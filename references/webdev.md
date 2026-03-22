# Web Development Architecture Reference

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
