# Requirements checklist

Status key: ✅ implemented and verified · ⚠️ partial (what is missing is stated) · ⛔ blocked/deferred (reason stated).
"Source": `FR-xxx`/`NFR-xxx` = signed requirements v0.3 (`docs/requirements`); `P§n` = the production prompt section.
State below is the **start of this pass** (2026-10-28) and is updated as work lands; the final report copies the final state.

## Accounts and access
| ID | Feature | Source | Route/Page | Components | API | DB models | Roles | Status |
|---|---|---|---|---|---|---|---|---|
| A-01 | Google sign-in | FR-001 | /login | LoginCard | POST /v1/auth/google | User, Session | public | ⚠️ API ✅ tested with fake verifier; no UI; real Google token unverified (BL-3) |
| A-02 | One active session (newest wins) | FR-003 | all | SessionBanner | auth guard | Session | student | ⚠️ API ✅; no UI messaging |
| A-03 | Refresh rotation + reuse detection | NFR-008 | – | – | POST /v1/auth/refresh | Session | student | ✅ API |
| A-04 | Profile (name, language, district, target exam) | FR-004 | /profile | ProfileForm | GET/PATCH /v1/me | User | student | ⚠️ API only |
| A-05 | Register/password/forgot/reset | P§4 | – | – | – | – | – | ⛔ Rejected: conflicts with FR-001 (D-001) |
| A-06 | Email verification | P§4 | – | – | – | – | – | ✅ via Google `email_verified` (D-001) |
| A-07 | Phone / OTP | FR-002 | – | – | – | User.phone | – | ⛔ Deferred by owner |
| A-08 | Data export / delete account | FR-006 | /profile | – | GET /v1/me/export, DELETE /v1/me | User | student | ⛔ Not started (P2 phase) |
| A-09 | Language preference (hi/en) | FR-005 | all | LanguageToggle | PATCH /v1/me | User.language | student | ⚠️ stored; no UI |
| A-10 | Account suspension takes effect next request | P§6 | /admin/users | – | admin | User.status | admin | ⛔ Not started |

## Roles, admin auth, audit
| ID | Feature | Source | Route/Page | API | DB | Roles | Status |
|---|---|---|---|---|---|---|---|
| R-01 | RBAC matrix enforced server-side on every route | P§6 | – | guard | AdminUser, Role | all | ⛔ Not started |
| R-02 | Admin login with password + TOTP 2FA, recovery codes | FR-094 | /admin/login | /admin/auth/* | AdminUser, AdminSession | admin | ⛔ Not started |
| R-03 | Audit log for sensitive actions | FR-094 | /admin/audit | /admin/audit | AuditLog | admin, super | ⚠️ table + billing writes only |
| R-04 | IDOR protection (student cannot read other attempts) | P§6 | – | attempts | Attempt | student | ✅ tested (404) |
| R-05 | Staff role management | P§5 | /admin/staff | /admin/staff | AdminUser | super | ⛔ Not started |

## Domain model and content
| ID | Feature | Source | API | DB | Status |
|---|---|---|---|---|---|
| D-01 | Exams, papers, admin-editable patterns (marks, negative fraction, qualifying %, merit flag) | P§2, FR-014 | /admin/exams | Exam, ExamPaper | ⚠️ Exam-level marks only |
| D-02 | Taxonomy Exam→Paper→Subject→Topic→Subtopic | P§2 | /admin/taxonomy | Subject, Topic, Subtopic | ⚠️ Subject→Topic only |
| D-03 | Question types: single, multi-statement, match, assertion-reason, chronology, passage | P§2 | /admin/questions | Question, Passage | ⛔ single only, flat |
| D-04 | Bilingual (hi/en) questions/options/explanations/instructions + missing-translation filter | P§2, FR-005 | /admin/questions | Question.body | ⚠️ stored bilingual; no types, no filter |
| D-05 | Question status workflow, versions, duplicate detection, image support, sanitised rich text | P§5 | /admin/questions | Question, QuestionVersion | ⛔ Not started |
| D-06 | PYQ archive by year/paper/exam | FR-032 | /v1/pyq | Test.pyq*, PyqPaper | ⛔ Not started |
| D-07 | Study material (PDF/notes/video links) with access control | FR-030/31 | /v1/material | Material | ⚠️ Notes only |
| D-08 | Current affairs (daily/monthly, CG/national, linked quiz) | FR-020..22 | /v1/ca | CurrentAffair | ⛔ Not started |
| D-09 | Daily quiz | FR-022 | /v1/daily-quiz | DailyQuiz | ⛔ Not started |
| D-10 | Exam calendar, official updates, FAQ | FR-034 | /v1/updates | Announcement, CalendarEvent, Faq | ⛔ Not started |
| D-11 | Syllabus view mapped to notes | FR-033 | /v1/exams/:id/syllabus | Syllabus | ⛔ Not started |

## Test engine (P§3)
| ID | Feature | Status |
|---|---|---|
| T-01 | Exam-hall UI: palette (not visited / not answered / answered / marked / answered+marked), legend, section tabs, Save&Next, Mark&Next, Clear, submit summary | ⛔ No UI |
| T-02 | Server-authoritative timer + client resync | ⚠️ server deadline ✅; no client |
| T-03 | Server-side auto-submit at expiry even if client is gone | ⚠️ lazy only; no scheduled job |
| T-04 | Debounced autosave + IndexedDB offline queue + Saved/Saving/Offline indicator | ⚠️ autosave API ✅; no client queue |
| T-05 | Resume exact state (answers, marks, current question, remaining time); multi-tab/device policy | ⚠️ answers + deadline resume ✅; no marks/current question/policy |
| T-06 | Idempotent submit, DB-enforced single final result, transactional scoring | ⚠️ conditional update; no constraint/idempotency key |
| T-07 | Answer key never sent before submit / result release | ⚠️ before submit ✅ tested; no release time |
| T-08 | Results: score, accuracy, counts, negative marks, time/question, section & topic breakdown, rank, percentile, solutions in both languages, topper/average | ⚠️ score+counts+review only |
| T-09 | Attempt rules: max attempts ✅, practice reattempt (unranked), live window ✅ | ⚠️ practice mode missing |
| T-10 | Sections with sectional time limits | ⛔ Not started |
| T-11 | Qualifying-only papers (min %) | ⛔ Not started |
| T-12 | Test versioning protects existing attempts | ⛔ Not started |
| T-13 | Language toggle per test/question | ⛔ No UI |

## Student platform (P§4, FR)
| ID | Feature | Status |
|---|---|---|
| S-01 | Dashboard: continue, upcoming/live, series, streak, score trend, weak topics | ⛔ |
| S-02 | Discovery with filters (exam, subject, free/paid, language) | ⚠️ API list only |
| S-03 | Bookmarks, personal notes, revision list from wrong + bookmarked | ⛔ |
| S-04 | Analytics: topic heatmap, progress, time management, recommendations | ⛔ |
| S-05 | Announcements, in-app notifications | ⛔ |
| S-06 | Push/email/SMS delivery | ⛔ Blocked BL-4 |
| S-07 | Leaderboards (per test, overall) with privacy control | ⛔ |
| S-08 | Plans, Razorpay purchase, coupons, entitlements | ⚠️ API ✅ tested with test double; no UI; keys BL-2 |
| S-09 | Purchase history + receipts, access expiry display | ⛔ |
| S-10 | Doubts desk (text/photo/voice, quota 3/day, 2 free/month, 48 h SLA, rating) | ⛔ |
| S-11 | Study planner (mentor) | ⛔ |
| S-12 | Interview preparation questions | ⛔ |
| S-13 | 1v1 battle (7 questions, bot/replay opponent) | ⛔ |
| S-14 | Live test lobby + scheduled start | ⚠️ window check only |
| S-15 | PWA / offline notes | ⛔ |

## Admin panel (P§5, FR-090..096)
| ID | Feature | Status |
|---|---|---|
| AD-01 | Dashboard with real metrics + alerts | ⛔ |
| AD-02 | Users: search/filter, profile, suspend/activate, grant/revoke access | ⛔ |
| AD-03 | Question bank CRUD, bilingual editor + preview, tagging, duplicates, report queue | ⛔ |
| AD-04 | Bulk CSV/XLSX import: template, row errors, dry-run, commit; export | ⛔ |
| AD-05 | Test builder (manual + rule-based), sections, timing, marking, instructions, schedule, release time, Draft→Published→Archived | ⛔ |
| AD-06 | Content: material, CA, announcements, banners with draft/publish/schedule | ⛔ |
| AD-07 | Results analytics (difficulty index, most-wrong), exports | ⛔ |
| AD-08 | Commerce: plans, coupons, transactions, refunds, reconciliation | ⛔ |
| AD-09 | Audit log viewer | ⛔ |
| AD-10 | Settings: site, exam patterns, notification templates | ⛔ |
| AD-11 | Doubts inbox, mentor drafts | ⛔ |
| AD-12 | Backups and restore procedure | ⚠️ RDS backups configured in Terraform (not applied); documented at the end |

## Non-functional
| ID | Feature | Status |
|---|---|---|
| N-01 | Server-side input validation, IDOR, rate limits | ⚠️ done for current API |
| N-02 | Question HTML sanitisation / XSS | ⛔ |
| N-03 | Mobile-first design system, WCAG AA, Hindi+English typography | ⛔ |
| N-04 | Performance on slow networks (Lighthouse mobile) | ⛔ |
| N-05 | Unit + integration + E2E tests | ⚠️ 52 backend tests; no E2E |
| N-06 | Deployed dev environment, deploy workflow | ⛔ BL-1 |
| N-07 | Android app (React Native) | ⛔ Not started (D-012) |
| N-08 | Mains/descriptive practice | ⛔ Deferred (D-004) |
