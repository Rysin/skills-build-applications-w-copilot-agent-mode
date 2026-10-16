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

- Presentation tier decision: Keep the React app as a thin client that renders the four capability sections from API data, using accessible Bootstrap markup and a shared API client to isolate fetch logic from view code.
- Logic tier decision: Implement Express routes and service logic in TypeScript to validate input, normalize IDs, compute derived values, and return contract-shaped JSON responses without leaking database errors to the browser.
- Data tier decision: Use Mongoose schemas and seed data to define the canonical contract for users, teams, activities, and workout suggestions, with required fields and relationship references enforced at the model layer.
- Activity tracking approach: Accept `userId`, `teamId`, `type`, and `minutes`; validate that both IDs resolve to real records; convert invalid casts into deliberate `4xx` responses; and persist `user`, `team`, `type`, `minutes`, computed `points`, and `occurredAt`.
- Team management approach: Store team `name`, `mascot`, and a `members` array of user references; reject missing team names with `400` and expose member data through the API in a consistent shape.
- Leaderboard approach: Aggregate each team's total activity points, sort descending by total, assign a 1-based rank, and return entries with `teamName`, `totalPoints`, and `rank`.
- Workout suggestions approach: Match a seeded user's email to their recent activity pattern or fitness goal, then recommend workouts whose `focus`, `minutes`, and `difficulty` align with that pattern and include a clear reason explaining the recommendation.

## Test plan

Record the commands you run and what each failure means.

- Data-tier command: `npm run test:data --prefix octofit-tracker/backend` — validates the Mongoose contracts. Failures here mean the seeded schema is incomplete: required fields like `name`, `email`, `fitnessGoal`, `team`, `type`, and `focus` are not enforced, so the data layer is not yet implementing the app contract.
- API command: `npm run test:api --prefix octofit-tracker/backend` — validates the route behavior against the documented endpoints. Failures here indicate that request validation, database lookup errors, leaderboard aggregation, or workout suggestion logic are not yet implemented or are returning the wrong status/body.
- Frontend command: `npm test --prefix octofit-tracker/frontend` — validates that the React UI renders the expected capability sections and data from the API. Failures mean the browser view is still using placeholder content rather than the real contract-driven UI.
- Backend build command: `npm run build --prefix octofit-tracker/backend` — checks whether the TypeScript API compiles cleanly after the route and schema fixes. A build failure signals contract implementation changes are type-invalid or missing imports.
- Frontend build command: `npm run build --prefix octofit-tracker/frontend` — confirms the Vite/React presentation layer still compiles after API integration work. A build failure usually points to invalid props, JSX, or data-shape assumptions in the frontend components.
