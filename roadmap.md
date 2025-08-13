# Backend-First Roadmap (with Extended Frontend On-Ramp)

**Goal:** Build a production-style Spring Boot backend first (demo-able via Swagger/Postman), then add a React frontend with a structured learning path. This plan assumes minimal React experience and gives you extra time to learn while shipping a working product.

## Phase 0 — Setup (2–3 days)

- **Install:** Java 21, Maven/Gradle, Node.js LTS, Docker Desktop, Git.
- **IDEs:** IntelliJ IDEA Community (backend), VS Code (frontend).
- **Repo:** Create GitHub repo with `/backend`, `/frontend`, `/docs`, `/ops`, `.github/workflows`.
- **Docker:** `postgres:16`, `mailpit`. Prepare `docker-compose.yml` with volumes and env files.
- **CI:** GitHub Actions workflow to run backend `mvn -B verify` on push.

## Phase 1 — Backend Foundations (Week 1)

- **Scaffold** Spring Boot (Web, Security, Data JPA, Validation, Cache, Actuator).
- **Flyway:** create `V1__init.sql` with `users`, `refresh_tokens` tables.
- **Auth:** Register/Login/JWT/Refresh (rotate on use), BCrypt hashing.
- **Mailpit:** Email verification + password reset flows.
- **OpenAPI:** springdoc at `/swagger-ui`. Export `openapi.json` to `/docs` (commit it).
- **CORS:** Allow only your future FE origin (temporarily allow localhost).
- **Tests:** JUnit + Testcontainers (Postgres) for user/auth flows.

## Phase 2 — Movies & TMDB (Week 2)

- **Entities:** `movies`, `genres`, `people`, `movie_genres`, `movie_cast`.
- **TMDB client** (WebClient). Admin endpoint to import one movie by `tmdbId`.
- **List + details endpoints:** `GET /api/v1/movies`, `GET /api/v1/movies/{id}`.
- **Search/filters/pagination:** by title/genre/year/rating; add proper DB indexes.
- **Caching:** Caffeine cache for popular lists and movie details (short TTL).
- **Tests:** Mock TMDB client; integration tests with seeded data.

## Phase 3 — Reviews & Moderation (Week 3)

- **Entities:** `reviews`, `review_votes`. Enforce one review per user per movie (unique index).
- **Endpoints:** list reviews (paginated), create/edit/delete (owner), vote helpful.
- **Moderation:** statuses `PENDING/APPROVED/HIDDEN`. Admin approve/hide endpoints.
- **Security:** Method-level `@PreAuthorize` for roles; rate limit writes with Bucket4j.
- **Errors:** Implement RFC 7807 ProblemDetail across controllers.

## Phase 4 — Watchlists & Mobile Readiness (Week 4)

- **Watchlists:** `watchlists` + `watchlist_items`. Endpoints to manage items.
- **Similar movies:** simple recs by shared genres + popularity proximity.
- **Mobile-friendly:** ETag/Last-Modified on GETs; optional cursor pagination for `/movies` and `/movies/{id}/reviews`.
- **Observability:** Actuator health/info, JSON logs with request ID (MDC).
- **Docs:** Update OpenAPI, seed script, and Postman collection.

## Phase 5 — Polish & Demo (Week 5)

- **One-command up:** `docker compose up` brings Postgres, Mailpit, backend; seed data auto-runs.
- **Newman in CI:** run Postman flows headless (commit `collection.json` in `/docs/postman`).
- **Short demo video:** Swagger → auth → list → details → review → admin approve.
- **Optional:** Cloudflare Tunnel for a temporary HTTPS URL to share the API.

## Frontend Learning Plan — Extra Time Built In (Weeks 6–9)

### Week 6 — React Basics (No API yet)

- **Initialize** Vite React + TypeScript: `npm create vite@latest frontend -- --template react-ts`.
- **Learn:** components, props, state, events, lists, conditional rendering, CSS/Tailwind basics.
- **Router:** set up `react-router-dom` with routes for Home, Search, Movie, Login, Register, Dashboard, Admin.
- **Forms:** React Hook Form + Zod for client-side validation (practice with dummy forms).
- **UI:** Choose Tailwind or MUI; build a basic layout (header, footer, grid/card components).
- **Testing:** Vitest + Testing Library for 1–2 simple component tests.

### Week 7 — Talking to the Backend (Generated Client)

- **Export** `openapi.json` from backend (fresh).
- **Generate TS client:** `npx @openapitools/openapi-generator-cli generate -g typescript-axios -i docs/openapi.json -o frontend/src/api`.
- **Axios interceptors:** attach access token; refresh logic on 401 (call `/auth/refresh`).
- **Auth pages:** hook up Register, Login. Store access token in memory; refresh via HttpOnly cookie (if you choose cookies for refresh).
- **TanStack Query:** set up a QueryClient; fetch popular movies to the Home page.

### Week 8 — Core Pages

- **Search page:** query + filters + pagination; URL params reflect filters.
- **Movie details:** poster, meta, cast credits, aggregated rating; show reviews list.
- **Write review** (protected route): form with rating + body; handle pending/approved states.
- **User dashboard:** my reviews, my watchlists (create/remove items).

### Week 9 — Admin, Polish, Testing

- **Admin dashboard:** pending reviews table; approve/hide actions.
- **Protected routes** by role; redirect unauth users to login.
- **Playwright e2e:** login flow, browse, open movie, submit review, admin approves.
- **Performance:** code-splitting routes, memoizing heavy lists; image lazy-loading.
- **UI polish:** consistent spacing/typography, empty states, loading/skeletons.

## Concrete Commands & Files

- **Backend:** `./mvnw spring-boot:run` (dev), `mvn -B verify` (CI).
- **Docker:** `docker compose up -d` to start Postgres + Mailpit + backend.
- **OpenAPI export (example):** expose `/v3/api-docs` and save to `docs/openapi.json`.
- **TS client generation:** `npx @openapitools/openapi-generator-cli generate -g typescript-axios -i docs/openapi.json -o frontend/src/api`.
- **Postman:** export collection to `docs/postman/collection.json`; Newman CI: `newman run docs/postman/collection.json`.

## Acceptance Criteria (Ship Checks)

- **Backend-only demo:** All core flows (auth, list, details, review, admin approve) pass in Postman and in CI via Newman.
- **OpenAPI:** Documented endpoints + examples; `docs/openapi.json` committed and current.
- **Repo:** `docker compose up` works on a fresh machine; seed data makes UI/API non-empty.
- **Frontend:** Home, Search, Details work without refresh loops; protected routes guard properly.
- **Testing:** At least 10 backend integration tests; 5+ Vitest component tests; 2 Playwright e2e flows.

## Common Pitfalls & Fixes

- **CORS errors:** restrict origins properly; in dev allow `http://localhost:5173`.
- **JWT refresh loops:** guard against parallel refresh; queue refresh or rotate token server-side.
- **N+1 queries:** use fetch joins or dedicated projection queries for details endpoints.
- **Rate limit surprises:** return `429` with `Retry-After` header; surface to UI nicely.
- **TMDB quotas:** cache results; back off with exponential retry (Resilience4j).

## What to Put on the CV

- "Designed and delivered a Spring Boot + PostgreSQL REST API with JWT auth and refresh token rotation; integrated TMDB; implemented FTS search, caching, and admin moderation."
- "Generated a type-safe TypeScript client from OpenAPI; built a React SPA (TanStack Query, React Hook Form) with protected routes and end-to-end tests."
- "Set up CI with GitHub Actions, Testcontainers, and Newman; one-command Docker environment."