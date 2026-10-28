# Engineering architecture (current and target)

The **signed** design is `docs/architecture/ARCHITECTURE.pdf` (v0.1 + amendment A1). This page is the engineering view: what exists today and the structure we are building toward. Decisions that deviate from the signed document go to `DECISIONS.md` and, if they change the signed design, as amendments in the signed document.

## Repositories
| Repo | Purpose | State |
|---|---|---|
| `exam-gadh-backend` | NestJS 11 + Prisma 6 + PostgreSQL API | auth, content, basic attempts, billing, 49 tests, CI green |
| `exam-gadh-frontend` | pnpm monorepo: `apps/web` (student), `apps/admin`, `packages/*` | empty |
| `exam-gadh-terraform` | AWS infra (Mumbai) | bootstrap applied; `envs/dev` planned, not applied |
| `exam-gadh-docs` | requirements, architecture, engineering docs | this folder |

## Current backend (one page)
```
HTTP -> helmet/CORS -> throttler -> ValidationPipe(whitelist) -> controller
     -> JwtAuthGuard (JWT HS256 + sessions row check, 5 s cache) -> service -> PrismaService -> PostgreSQL
errors: ApiError(code) -> AllExceptionsFilter -> { error: { code, message, details } }
modules: health, auth (google, sessions, refresh rotation), users(me), content(exams, subjects, notes),
         mocktests (list, start, autosave, submit, result), billing (plans, Razorpay orders/verify/webhook, coupons), access
```
Single process, in-memory throttler and session cache, no Redis, no scheduler.

## Target backend structure
```
src/
  modules/<feature>/
    <feature>.controller.ts     thin: parse, authorise, delegate
    <feature>.service.ts        business rules
    <feature>.repository.ts     all Prisma access for the feature
    <feature>.schemas.ts        zod schemas shared by controller + import + tests
  common/   api-error, filters, permissions (RBAC), audit, sanitiser, clock, pagination
  platform/ prisma, config, queue (BullMQ), realtime (Socket.IO), storage (S3 presign)
features: auth, admin-auth, rbac, users, taxonomy, exam-config, questions, passages, import,
          tests, test-series, attempts, results, analytics, bookmarks, notes, material, current-affairs,
          pyq, announcements, notifications, leaderboard, doubts, planner, interview, billing,
          audit, settings, live (realtime), battle (realtime), jobs
```
Cross-cutting rules: every route declares a permission; services never trust ids from the client without an ownership check; scoring only on the server; test snapshots are immutable.

## Target data model (summary)
`Exam → ExamPaper (marks, minutes, marking, qualifying) → PaperSubject → Subject → Topic → Subtopic`.
`Question (type, bilingual body, answer, status, version) + Passage`; `Test → TestVersion (immutable once published) → TestSection → TestVersionQuestion (snapshot)`; `TestSeries`; `Attempt (version-bound, idempotent submit) → AttemptAnswer (palette state) → AttemptResult (1:1, unique)`; `UserTopicStat`; admin side `AdminUser, AdminSession, Role->Permission matrix, AuditLog, ImportJob, Setting`.

## Frontend target
pnpm workspace: `apps/web` (Next.js, student, Hindi-first, PWA), `apps/admin` (Next.js, separate hostname), `packages/ui` (tokens + components), `packages/i18n`, `packages/api` (typed client generated from the backend OpenAPI). Mobile (React Native/Expo) follows the same `packages/api`.

## Runtime target (signed)
One EC2 `t4g` per environment running Docker Compose (caddy, api, web, admin, redis) + private RDS PostgreSQL + S3/CloudFront; GitHub OIDC deploys. Dev not applied yet.
