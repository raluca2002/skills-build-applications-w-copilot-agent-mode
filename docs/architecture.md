# OctoFit Tracker architecture

OctoFit Tracker is a three-tier application for a school fitness program. The starter code includes API contracts and intentionally incomplete implementation points so you can use GitHub Copilot agent mode in a test-driven loop.

![Three-tier OctoFit architecture](../.github/images/octofit-architecture.svg)

## Tier boundaries

- **Presentation tier**: React 19, Vite, React Router, Bootstrap, and a shared API client in `octofit-tracker/frontend/src`.
- **Logic tier**: Express and TypeScript routes/services in `octofit-tracker/backend/src` expose `/api/*` behavior on port `8000`.
- **Data tier**: Mongoose models and seed data in `octofit-tracker/backend/src/models` and `src/scripts/seed.ts` persist to MongoDB on port `27017`.

## Ports and environment rules

- MongoDB: `27017`
- Express API: `8000`
- Vite frontend: `5173`
- The browser calls relative `/api` paths. Vite proxies them to `http://localhost:8000`, so the unauthenticated API port stays private in Codespaces. `VITE_API_BASE_URL` can override the base URL.

## API contract

| Capability | Method and path | Expected behavior |
| --- | --- | --- |
| Activity tracking | `GET /api/activities`, `POST /api/activities` | List and create valid activity records. POST accepts `userId`, `teamId`, `type`, and `minutes`; `userId` and `teamId` must be well-formed IDs for an existing user and team; otherwise respond with a deliberate 4xx (never a 500 from a cast error). A created activity stores the submitted `user`, `team`, `type`, and `minutes` plus computed `points` and `occurredAt`. |
| Teams | `GET /api/teams`, `POST /api/teams` | List and create teams with members. Created and listed teams keep the submitted `name` and `mascot`. Reject a team without a name with `400`. |
| Leaderboard | `GET /api/leaderboard` | Return teams ordered by the sum of their activity points, highest first. Each entry has `teamName`, `totalPoints`, and a 1-based `rank`. |
| Workout suggestions | `GET /api/workouts/suggestions/:userEmail` | Recommend workouts from recent activity patterns for the matching seeded user's email address. The first suggestion's `focus` matches that user's goal or recent activity, with a `reason` explaining why. An email with no matching user returns a deliberate 4xx (for example, `404`). |

## Implementation decisions

Record your plan here before editing code.

- Presentation tier decision: Keep React pages focused on rendering and interaction; use the shared API client for relative `/api` requests, with explicit loading, empty, and error states.
- Logic tier decision: Implement API contracts in Express routes/services, validate request data before database operations, and return deliberate 4xx responses for invalid or unknown resources. Do not add authentication.
- Data tier decision: Keep persistence in Mongoose models and the seed script; require the contract fields, use references for user/team relationships, and keep seeding repeatable.
- Activity tracking approach: Validate `userId`, `teamId`, activity type, and minutes before creating a record; persist the related user/team and submitted values, and compute points and `occurredAt` on the server.
- Team management approach: List and create teams with their name, mascot, and member references; reject a missing name with `400`.
- Leaderboard approach: Sum activity points by team, sort totals descending, and return each team with its name, total, and 1-based rank.
- Workout suggestions approach: Find the requested user by email and use their goal and recent activities to choose relevant workouts with an explanatory reason; return a 4xx response if no user matches.

## Test plan

Run focused contract tests while implementing each tier, then run both builds. A failure indicates:

- Data tier: `npm run test:data --prefix octofit-tracker/backend` — model validation, relationship, or seed behavior does not meet the data contract.
- API: `npm run test:api --prefix octofit-tracker/backend` — route validation/status codes, persistence, ranking, or recommendation behavior does not meet the API contract.
- Frontend: `npm test --prefix octofit-tracker/frontend` — rendered content, API requests, or accessible loading/empty/error states do not meet the presentation contract.
- Backend build: `npm run build --prefix octofit-tracker/backend` — backend TypeScript or build configuration is invalid.
- Frontend build: `npm run build --prefix octofit-tracker/frontend` — frontend compilation or bundling is invalid.

Latest baseline test run: `npm test --prefix octofit-tracker/backend` had 8 failures and 1 pass (4 data, 4 API); `npm test --prefix octofit-tracker/frontend` had 15 failures. These failures identify the contract gaps above; rerun the focused tier tests after each implementation step.
