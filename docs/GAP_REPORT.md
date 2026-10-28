# Gap report

Date: 2026-10-28 · Scope: all four repos at the start of the "production quality" pass.
Legend for severity: **H** blocks launch · **M** must fix before real students · **L** polish.

## 1. Missing features (against the signed requirements v0.3 + the production prompt)
| # | Gap | Sev | Plan |
|---|---|---|---|
| G1 | **No frontend at all.** `exam-gadh-frontend` only holds the installed skills. No student web, admin web, or mobile app. | H | P7–P9 |
| G2 | **No admin API.** No admin auth, roles, question bank, import, test builder, users, dashboard. | H | P3–P4 |
| G3 | Question model is a flat JSON blob with one `answer` int; no question types (multi-statement, match, assertion-reason, chronology, passage), no status workflow, no versioning, no duplicate detection, no translation-missing tracking. | H | P2 |
| G4 | Taxonomy is `Subject → Topic` only; the spec needs `Exam → Paper → Subject → Topic → Subtopic` and admin-editable exam patterns (marks, negative fraction, qualifying papers). | H | P2 |
| G5 | Tests have no sections, no sectional timers, no versioning, no result-release time, no series. Editing a test's question list after attempts exist would corrupt those attempts. | H | P2/P4 |
| G6 | Test engine lacks: palette states (visited / marked / answered & marked), persisted current question, per-question language, idempotent submit key, server-side auto-submit when the client is gone (expiry is only handled lazily on next request), multi-tab/device policy, offline queue, rank/percentile, topper/average comparison, section and topic breakdowns, result-release gating. | H | P5 |
| G7 | Student features absent: dashboard, analytics/weak topics, bookmarks, notes, revision list, current affairs, PYQ archive, study material, daily quiz, announcements, notifications, leaderboard, enrollment, purchase history/receipts. | H | P6 |
| G8 | Original-spec features not built: doubts desk with SLA/quota, study planner (mentor), interview prep, 1v1 battle, live-test lobby. | H | P6, P10 |
| G9 | No suspension model: `User` has no status. Session cache (5 s) would delay a suspension. | M | P3 |
| G10 | No RBAC. `AuditLog` table exists but only billing writes to it. | H | P3 |
| G11 | Push/email/SMS delivery: no provider credentials exist. | M | Blocked, see BLOCKERS |
| G12 | Mains/descriptive practice (answer upload + teacher evaluation) was **excluded from the MVP by the owner's own scope choice**; not built. | – | Deferred (D-004) |

## 2. Broken flows / correctness
| # | Issue | Sev |
|---|---|---|
| B1 | `finalize()` runs scoring then `updateMany`; there is no DB constraint guaranteeing a single result row, and scoring is not inside one transaction with answer reads. Two concurrent submits could both compute (only one wins the update, but the logic is racy by design). | M |
| B2 | `result()` returns the full review (answers + explanations) immediately on submit; there is no scheduled release. | H |
| B3 | Attempt `order` is stored as a string array of question ids: it points at **live** question rows. If an admin edits a question or answer key later, past attempts silently change. | H |
| B4 | Autosave endpoint accepts up to 200 answers but has no per-question visited/marked state, so a refresh loses palette state. | M |
| B5 | Dev environment is not applied; there is no deploy workflow or server compose file, so nothing is reachable yet. | H |

## 3. Security risks
| # | Risk | Sev |
|---|---|---|
| S1 | Question HTML is not sanitised anywhere (there is no admin input path yet, but there will be). Needs a sanitiser on write and a safe renderer. | H |
| S2 | JWT is HS256 with a single secret and no rotation procedure. Acceptable for one service; document rotation. | L |
| S3 | Rate limits are in-process (`@nestjs/throttler` memory store). Correct for one instance; breaks if scaled to two. Move to Redis with the realtime tier. | M |
| S4 | Session revocation cache is per-process (5 s). Suspension must clear it explicitly (single instance) and the docs must state the multi-instance caveat. | M |
| S5 | No admin 2FA, no CSRF protection (needed once admin uses cookies), no account lockout. | H |
| S6 | `RAZORPAY_*` are required only in production; webhook is protected by HMAC. Reconciliation job against Razorpay does not exist yet. | M |
| S7 | Uploads (doubt images/voice, material) have no pre-signed upload path, type or size checks yet. | M |

## 4. Performance
| # | Issue | Sev |
|---|---|---|
| P1 | Result review and attempt payload load every question body in one query; fine for 100 questions, but must exclude explanations before release and avoid reloading per request. | L |
| P2 | Leaderboard/rank queries need an index on `(testVersionId, score desc, timeSpentSec)`; none exists. | M |
| P3 | Catalog endpoints (exams, subjects, notes list) are uncached. | L |
| P4 | Nothing is measured on a slow network yet (no frontend). | M |

## 5. Structure, naming, duplication, dead code
- Services call Prisma directly. The target is `controller → service → repository` per feature, with shared validation schemas. New modules follow the target; old ones (`auth`, `billing`, `mocktests`) are refactored as they are touched.
- Folder `mocktests` should be `attempts` + `tests` (two concerns). Seed content is labelled SAMPLE and must not be mistaken for real content.
- No dead code found by lint (`eslint` clean, `tsc --noEmit` clean). One unused stub: `Test.shuffle`/`attemptLimit` are used; no dead models.
- No `CLAUDE.md` in any repo (added in this pass).

## 6. Test gaps
- 49 integration tests + 3 scoring unit tests exist for the current API only.
- Missing: permission tests, import tests, version-lock tests, auto-submit tests, concurrency tests, any frontend test, any E2E.
