# Requirements checklist

Status key: ✅ implemented and verified · ⚠️ partial (what is missing is stated) · ⛔ blocked/deferred (reason stated).
"Source": `FR-xxx`/`NFR-xxx` = signed requirements v0.3 (`docs/requirements`); `P§n` = the production prompt section.
State below was last updated 2026-10-29 after the backend API work (P1–P6). "API ✅" means implemented and covered by integration tests against a real PostgreSQL; the web/mobile UI is the next phase, so every user-facing feature still needs its screen. The final report copies the final state.

## Accounts and access
| ID | Feature | Source | Route/Page | Components | API | DB models | Roles | Status |
|---|---|---|---|---|---|---|---|---|
| A-01 | Google sign-in | FR-001 | /login | LoginCard | POST /v1/auth/google | User, Session | public | ⚠️ API ✅ tested with fake verifier; no UI; real Google token unverified (BL-3) |
| A-02 | One active session (newest wins) | FR-003 | all | SessionBanner | auth guard | Session | student | ⚠️ API ✅; no UI messaging |
| A-03 | Refresh rotation + reuse detection | NFR-008 | – | – | POST /v1/auth/refresh | Session | student | ✅ API |
| A-04 | Profile (name, language, district, target exam, leaderboard privacy) | FR-004 | /profile | ProfileForm | GET/PATCH /v1/me | User | student | ⚠️ API ✅; no UI |
| A-05 | Register/password/forgot/reset | P§4 | – | – | – | – | – | ⛔ Rejected: conflicts with FR-001 (D-001) |
| A-06 | Email verification | P§4 | – | – | – | – | – | ✅ via Google `email_verified` (D-001) |
| A-07 | Phone / OTP | FR-002 | – | – | – | User.phone | – | ⛔ Deferred by owner |
| A-08 | Data export / delete account | FR-006 | /profile | – | GET /v1/me/export, DELETE /v1/me | User | student | ⚠️ API ✅ (export JSON; delete erases personal data, keeps orders/invoices); no UI |
| A-09 | Language preference (hi/en) | FR-005 | all | LanguageToggle | PATCH /v1/me | User.language | student | ⚠️ stored; no UI |
| A-10 | Account suspension takes effect next request | P§6 | /admin/users | – | admin | User.status | admin | ✅ API (session revoked, tested) |

## Roles, admin auth, audit
| ID | Feature | Source | Route/Page | API | DB | Roles | Status |
|---|---|---|---|---|---|---|---|
| R-01 | RBAC matrix enforced server-side on every route | P§6 | – | guard | AdminUser | all | ✅ fail-closed guard; route-coverage test (docs/PERMISSIONS.md) |
| R-02 | Admin login with password + TOTP 2FA, recovery codes | FR-094 | /admin/login | /admin/auth/* | AdminUser, AdminSession | admin | ⚠️ API ✅ (lockout, CSRF, replay protection); no UI |
| R-03 | Audit log for sensitive actions | FR-094 | /admin/audit | /admin/audit | AuditLog | admin, super | ✅ API (redacting; viewer) |
| R-04 | IDOR protection (student cannot read other attempts) | P§6 | – | attempts | Attempt | student | ✅ tested (404) |
| R-05 | Staff role management | P§5 | /admin/staff | /admin/staff | AdminUser | super | ⚠️ accounts created with `scripts/create-admin`; no staff-management API/UI yet |

## Domain model and content
| ID | Feature | Source | API | DB | Status |
|---|---|---|---|---|---|
| D-01 | Exams, papers, admin-editable patterns (marks, negative fraction, qualifying %, merit flag) | P§2, FR-014 | /admin/exams, /v1/exams | Exam, ExamPaper | ⚠️ API ✅; patterns flagged unverified until an admin confirms (BL-6) |
| D-02 | Taxonomy Exam→Paper→Subject→Topic→Subtopic | P§2 | /admin/taxonomy | Subject, Topic, Subtopic, PaperSubject | ✅ API |
| D-03 | Question types: single, multi-statement, match, assertion-reason, chronology, passage | P§2 | /admin/questions | Question, Passage | ✅ API (per-type zod schemas) |
| D-04 | Bilingual (hi/en) questions/options/explanations/instructions + missing-translation filter | P§2, FR-005 | /admin/questions | Question.body | ✅ API |
| D-05 | Question status workflow, versions, duplicate detection, image support, sanitised rich text | P§5 | /admin/questions | Question, QuestionVersion | ✅ API (images via presigned upload) |
| D-06 | PYQ archive by year/paper/exam | FR-032 | /v1/pyq | Test.pyq* | ✅ API (+ topic frequency) |
| D-07 | Study material (PDF/notes/video links) with access control | FR-030/31 | /v1/materials | Material | ✅ API (private PDFs by 10-minute links) |
| D-08 | Current affairs (daily/monthly, CG/national, linked quiz) | FR-020..22 | /v1/ca | CurrentAffair | ✅ API |
| D-09 | Daily quiz | FR-022 | /v1/daily-quiz | DailyQuiz | ✅ API |
| D-10 | Exam calendar, official updates, FAQ | FR-034 | /v1/updates, /v1/faqs | Announcement, CalendarEvent, Faq | ✅ API |
| D-11 | Syllabus view mapped to notes | FR-033 | /v1/subjects | Subject, Topic | ⚠️ taxonomy + notes by subject ✅; no dedicated syllabus page/API |

## Test engine (P§3)
| ID | Feature | Status |
|---|---|---|
| T-01 | Exam-hall UI: palette (not visited / not answered / answered / marked / answered+marked), legend, section tabs, Save&Next, Mark&Next, Clear, submit summary | ⛔ No UI yet (palette states are served by the API) |
| T-02 | Server-authoritative timer + client resync | ⚠️ server deadlines + 15 s grace ✅; no client |
| T-03 | Server-side auto-submit at expiry even if client is gone | ✅ sweeper (`AttemptFinalizer`, idempotent; D-008 updated: no queue needed at this scale) |
| T-04 | Debounced autosave + IndexedDB offline queue + Saved/Saving/Offline indicator | ⚠️ autosave API ✅ (stale-write protection); no client queue |
| T-05 | Resume exact state (answers, marks, current question, remaining time); multi-tab/device policy | ✅ API (takeover/block policy in settings) |
| T-06 | Idempotent submit, DB-enforced single final result, transactional scoring | ✅ row lock + `AttemptResult` primary key + submit key |
| T-07 | Answer key never sent before submit / result release | ✅ tested, including bookmarks/revision paths |
| T-08 | Results: score, accuracy, counts, negative marks, time/question, section & topic breakdown, rank, percentile, solutions in both languages, topper/average | ✅ API |
| T-09 | Attempt rules: max attempts, practice reattempt (unranked), live window | ✅ API |
| T-10 | Sections with sectional time limits | ✅ API |
| T-11 | Qualifying-only papers (min %) | ✅ API (general and reserved thresholds) |
| T-12 | Test versioning protects existing attempts | ✅ immutable versions (DB triggers), audited key correction + regrade |
| T-13 | Language toggle per test/question | ⚠️ attempt language stored; no UI |

## Student platform (P§4, FR)
| ID | Feature | Status |
|---|---|---|
| S-01 | Dashboard: continue, upcoming/live, series, streak, score trend, weak topics | ⚠️ `GET /v1/home` ✅; no UI |
| S-02 | Discovery with filters (exam, subject, free/paid, language) | ⚠️ API ✅ (catalog filters); no UI |
| S-03 | Bookmarks, personal notes, revision list from wrong + bookmarked, question reports, practice sets | ⚠️ API ✅; no UI |
| S-04 | Analytics: topic heatmap, progress, time management, recommendations | ⚠️ `GET /v1/me/analytics` ✅ (rule-based advice, no AI); no UI |
| S-05 | Announcements, in-app notifications | ⚠️ API ✅ (inbox, result-ready / plan-expiry / live-test jobs, templates); no UI |
| S-06 | Push/email/SMS delivery | ⛔ Blocked BL-4 |
| S-07 | Leaderboards (per test, overall) with privacy control | ⚠️ API ✅; no UI |
| S-08 | Plans, Razorpay purchase, coupons, entitlements | ⚠️ API ✅ tested with test double; no UI; keys BL-2 |
| S-09 | Purchase history + receipts, access expiry display | ⚠️ API ✅; no UI; invoice has no GST breakdown (BL-10) |
| S-10 | Doubts desk (text/photo/voice, quota 3/day, 2 free/month, 48 h SLA, rating) | ⚠️ API ✅ (similar FAQs, attachments, reopen, rating); operator "away" mode (FR-096, COULD) not built; no UI |
| S-11 | Study planner (mentor) | ⚠️ API ✅ (rule-based draft, mentor edits and sends); no UI |
| S-12 | Interview preparation questions | ⚠️ API ✅; no UI |
| S-13 | 1v1 battle (7 questions, bot/replay opponent) | ⛔ |
| S-14 | Live test lobby + scheduled start | ⚠️ window check only |
| S-15 | PWA / offline notes | ⛔ |

## Admin panel (P§5, FR-090..096)
| ID | Feature | Status |
|---|---|---|
| AD-01 | Dashboard with real metrics + alerts | ⚠️ API ✅; no UI |
| AD-02 | Users: search/filter, profile, suspend/activate, grant/revoke access | ⚠️ API ✅; no UI |
| AD-03 | Question bank CRUD, bilingual editor + preview, tagging, duplicates, report queue | ⚠️ API ✅; no UI |
| AD-04 | Bulk CSV/XLSX import: template, row errors, dry-run, commit; export | ⚠️ API ✅ (CSV; XLSX not supported); no UI |
| AD-05 | Test builder (manual + rule-based), sections, timing, marking, instructions, schedule, release time, Draft→Published→Archived | ⚠️ API ✅; no UI |
| AD-06 | Content: material, CA, announcements, banners with draft/publish/schedule | ⚠️ API ✅; no UI |
| AD-07 | Results analytics (difficulty index, most-wrong), exports | ⚠️ API ✅; no UI |
| AD-08 | Commerce: plans, coupons, transactions, refunds, reconciliation | ⚠️ API ✅ (test double; live Razorpay keys BL-2); no UI |
| AD-09 | Audit log viewer | ⚠️ API ✅; no UI |
| AD-10 | Settings: site, exam patterns, notification templates | ⚠️ API ✅; no UI |
| AD-11 | Doubts inbox, mentor drafts | ⚠️ API ✅ (inbox, SLA filters, stats, canned replies, abuse flag; mentor requests); no UI |
| AD-12 | Backups and restore procedure | ⚠️ RDS backups configured in Terraform (not applied); documented at the end |

## Non-functional
| ID | Feature | Status |
|---|---|---|
| N-01 | Server-side input validation, IDOR, rate limits | ✅ API (zod on every body/query, ownership checks, throttling) |
| N-02 | Question HTML sanitisation / XSS | ✅ API (markup stripped on write; CSV formula guard) |
| N-03 | Mobile-first design system, WCAG AA, Hindi+English typography | ⛔ |
| N-04 | Performance on slow networks (Lighthouse mobile) | ⛔ |
| N-05 | Unit + integration + E2E tests | ⚠️ 280 backend tests passing in CI-equivalent run; no browser E2E yet |
| N-06 | Deployed dev environment, deploy workflow | ⛔ BL-1 |
| N-07 | Android app (React Native) | ⛔ Not started (D-012) |
| N-08 | Mains/descriptive practice | ⛔ Deferred (D-004) |
