# ExamGadh – Requirements

Status: DRAFT v0.1 for user confirmation · Date: 2026-10-27 · Step 1 of the SDLC pipeline

Source tags: `[brief]` store listing text · `[sketch]` hand sketch (`inputs/sketch-test-ca-notes.jpg`) ·
`[notebook]` notebook page dated 5 Sep 2026 (`inputs/notebook-5sep2026.jpg`) · `[logo]` ·
`[apk-user]` ExamGadh v4.2.0 (`com.cgpsc.prep`) · `[apk-admin]` ExamGadh Admin v1.2.0 / "CA Studio" (`com.cgpsc.admin`) ·
`[answer]` user's answers in this session. Items marked `(inferred)` are not stated by the user.

## 1. Vision and scope decision

ExamGadh is an exam-preparation platform for Chhattisgarh state exams (CGPSC and CG Vyapam), tagline
"CGPSC & Vyapam Prep" `[logo]`, "Your Personal Guide" `[apk-user]`. It offers mock tests, notes, PYQs, current
affairs, analytics and a doubt desk in Hindi and English `[brief]`.

Two Android prototypes already exist (`[apk-user]`, `[apk-admin]`). **Decision `[answer]`: rebuild from scratch.**
The APKs are a feature and UX reference only, and are not to be extended. The new system is delivered in three repos:
`exam-gadh-frontend` (apps), `exam-gadh-backend` (API + infra), `exam-gadh-docs` (documents and PDFs).

Commercial model: free + paid content; the test series is the paid product, current affairs is the engagement/free
hook `[sketch][notebook]`.

## 2. Users and roles

| Role | Description | Source |
|---|---|---|
| Student (free) | Aspirant for CGPSC / Vyapam exams in all CG districts; Hindi-first | [brief] |
| Student (paid) | Holds a plan/entitlement; unlocks paid tests, notes, doubts | [sketch][apk-user] |
| Admin: owner | Full control, manages other admins | [apk-admin] (`role: owner`) |
| Admin: other roles | Content editor, doubt-answering teacher, read-only (exact roles TBD, OQ-009) | [apk-admin] (`readonly`, `/admins`) |

## 3. Platforms `[answer]`

- Android app (React Native) – primary
- Web app (Next.js) – students
- Admin web panel – replaces the Android-only admin app
- NOT at launch: iOS (inferred: React Native keeps it open for later)

## 4. Functional requirements

Priority: MUST / SHOULD / COULD. Phase: MVP = first release `[answer]`, P2 = after MVP.

### 4.1 Accounts and access
| ID | Requirement | Pri | Phase | Source |
|---|---|---|---|---|
| FR-001 | Sign in with Google | MUST | MVP | [notebook][answer] |
| FR-002 | Phone number captured on the account profile (notebook says "Google login + phone no"); whether it is verified by OTP is OQ-002 | MUST | MVP | [notebook] |
| FR-003 | **One user active on one device at a time**: operators can see and monitor concurrent usage, and a second simultaneous login is blocked or forces the first session out | MUST | MVP | [answer] |
| FR-004 | Profile with avatar (16 stock avatars exist), name, district (33 CG districts), target exam | SHOULD | MVP | [apk-user] avatars, districts |
| FR-005 | Hindi/English language toggle on all content and UI | MUST | MVP | [brief][apk-user] |
| FR-006 | Data export for the user ("me/export") | COULD | P2 | [apk-user] `/v1/me/export` |

### 4.2 Tests (test series)
| ID | Requirement | Pri | Phase | Source |
|---|---|---|---|---|
| FR-010 | Test types: **static** and **live** `[sketch]`; static tests are topic-wise, subject-wise or complete (full-length) `[sketch]` | MUST | MVP | [sketch] |
| FR-011 | Each test is free or paid `[sketch]`; access rules by plan, per test / subject / series / general bucket | MUST | MVP | [sketch][apk-admin] `/access/*` |
| FR-012 | Question model: subject, topic, subtopic, question (English), options 1-4, explanation in Hindi and English `[sketch]`; plus difficulty level `[notebook]`, answer key, PYQ metadata (year/paper/set), source (original/pyq), image/passage layouts | MUST | MVP | [sketch][notebook][apk-user] |
| FR-013 | Question bank holding N questions, organised by topic and difficulty; tests can be built by manual pick or auto-pick (random + personalisation) | MUST | MVP | [notebook][apk-admin] `/tests/pick`, `/tests/autopick` |
| FR-014 | Real-time exam simulation: timer, section/paper structure, negative marking per exam (e.g. CGPSC Prelims 100 Q, 200 marks, 120 min, -1/3 negative; CSAT qualifying 33% / 23% reserved) | MUST | MVP | [brief][apk-user] exams |
| FR-015 | Test kinds seen in prototype: diagnostic, mock, full-length prelims, CSAT, topic, subject, PYQ, weekly live | MUST | MVP | [apk-user] tests |
| FR-016 | Attempt controls: attempt limit, shuffle, integrity logging, entry close for live tests | SHOULD | MVP | [apk-user] test fields |
| FR-017 | **Pre-test** and **live test**: scheduled live tests with a lobby/pre-test step, live monitor for operators | MUST | MVP | [notebook][apk-admin] `/tests/{id}/monitor` |
| FR-018 | Answer key with bilingual explanations after submission; answer-key status (official/model) | MUST | MVP | [brief][apk-user] |
| FR-019 | Question error reports from students, triaged by admins | SHOULD | MVP | [apk-user] `/v1/reports`, [apk-admin] `/error-reports` |

### 4.3 Current affairs (engagement)
| ID | Requirement | Pri | Phase | Source |
|---|---|---|---|---|
| FR-020 | Daily current-affairs capsules, browsable **date-wise** and **topic-wise**, split Chhattisgarh / India (national) | MUST | MVP | [sketch][notebook][apk-user] ca_categories |
| FR-021 | Each item: title, summary, exam-angle, tags, source, image/cards, link to related notes and quiz questions, special flag | SHOULD | MVP | [apk-user] ca fields |
| FR-022 | Daily current-affairs quiz | SHOULD | MVP | [apk-user] `ca_quiz` |
| FR-023 | Admin "CA Studio": collect articles from configured portals/YouTube/social feeds the admin is logged in to, draft capsules (translation, optional AI summary), review inbox, publish with schedule | SHOULD | MVP (inferred) | [apk-admin] `/ca`, `/ca-inbox`, `portals.json`, js extractors |
| FR-024 | Monthly current-affairs compilation | COULD | P2 | [apk-user] `ca_monthly` |

### 4.4 Notes and PYQs
| ID | Requirement | Pri | Phase | Source |
|---|---|---|---|---|
| FR-030 | Notes organised by subject; CG GS and India (GS) sections, subjects → topics `[sketch]`; structured bilingual sections with anchors | MUST | MVP | [sketch][brief][apk-user] |
| FR-031 | Notes access paid/free, with optional free preview | MUST | MVP | [apk-user] access_rules |
| FR-032 | PYQ archive by exam/year/paper; solved with explanations; "asked in" links from notes to PYQs; PYQ frequency by topic | MUST | MVP | [brief][apk-user] pyq_papers, asked_in, pyq_freq |
| FR-033 | Exam syllabus view mapped to notes | SHOULD | MVP | [apk-user] syllabi |
| FR-034 | Exam calendar and official updates/notifications, FAQ | SHOULD | MVP | [brief][apk-user] calendar, updates, faqs |

### 4.5 Analytics and ranking
| ID | Requirement | Pri | Phase | Source |
|---|---|---|---|---|
| FR-040 | Student performance analytics on profile: accuracy, time per question, strong/weak topics, unattempted vs incorrect | MUST | MVP | [notebook][brief] |
| FR-041 | Percentile/rank among aspirants; leaderboard (per test and app-wide) | MUST | MVP | [brief][apk-user] `/v1/leaderboard/app` |
| FR-042 | Admin analytics: signups, DAU/MAU, attempts, revenue, conversion, topic accuracy, anomalies, question flags | MUST | MVP | [apk-admin] `/analytics`, `/dashboard` |

### 4.6 1v1 live battle `[notebook]`
| ID | Requirement | Pri | Phase | Source |
|---|---|---|---|---|
| FR-050 | Live Battle mode: matchmaking queue, 7 quick questions, beat a rival | MUST | MVP | [notebook][apk-user] `battle_*` |
| FR-051 | Opponent is a **previous user's recorded test (async)** or a **bot** when no live opponent is available | MUST | MVP | [notebook] "previous user base test + bot" |

### 4.7 Doubts desk
| ID | Requirement | Pri | Phase | Source |
|---|---|---|---|---|
| FR-060 | Student raises a doubt on a question/note (text, photo, voice), sees similar answered FAQs first | MUST | MVP | [brief][apk-user] `/v1/doubts`, `RECORD_AUDIO` |
| FR-061 | Quota: 3/day, free users 2/month (configurable); SLA 48 h with alerts when due; rating; reopen; abuse flag | MUST | MVP | [apk-user] settings, [apk-admin] doubts |
| FR-062 | Admin inbox for doubts with canned replies, first-response/answered timestamps and median-response stats | MUST | MVP | [apk-admin] |

### 4.8 Study planner and interview prep `[answer]`
| ID | Requirement | Pri | Phase | Source |
|---|---|---|---|---|
| FR-070 | Study planner ("mentor"): student gives hours/day and exam; weekly plan generated to the exam date; admin can review/draft and send | MUST | MVP | [answer][apk-user] `/v1/mentor`, [apk-admin] `/mentor` |
| FR-071 | Interview preparation question bank by exam and category with answer points | MUST | MVP | [answer][apk-user] `interview` |

### 4.9 Plans, payments, notifications
| ID | Requirement | Pri | Phase | Source |
|---|---|---|---|---|
| FR-080 | Subscription plans by duration (3/6/12 months), per exam or all, with trial days; sample prices Rs 399–1499 | MUST | MVP | [apk-user] plans |
| FR-081 | Razorpay as payment gateway. User wording: "razorpay would also be there but inside the in app purchase as payment gateway"; exact flow is OQ-001 | MUST | MVP | [answer] |
| FR-082 | Coupon codes (plan + days) | SHOULD | MVP | [apk-user] `/v1/coupons/redeem` |
| FR-083 | Entitlements synced to the client; offline licence validity (30 days) for downloaded content | SHOULD | MVP (inferred) | [apk-user] settings |
| FR-084 | Push notifications and reminders with quiet hours (22:00–07:00) and a cap of 2 nudges/day | SHOULD | MVP | [apk-user] settings, `POST_NOTIFICATIONS`, `ReminderReceiver` |

### 4.10 Admin panel (web)
| ID | Requirement | Pri | Phase | Source |
|---|---|---|---|---|
| FR-090 | CRUD for exams, topics, subjects, syllabi, notes, questions (single and bulk import via template, preview then commit), tests, test sections, plans, coupons, FAQs, updates, calendar, avatars, app texts | MUST | MVP | [apk-admin] `/r/*`, `/import/*` |
| FR-091 | Question workflow: draft → approved → published, schedule (`publish_after`), checklist, versions, bulk status change, machine-translation flag | MUST | MVP | [apk-admin] question fields |
| FR-092 | Access-control matrix for free/paid per test/series/subject | MUST | MVP | [apk-admin] `/access` |
| FR-093 | User directory with per-user activity, attempts, doubts, entitlements | MUST | MVP | [apk-admin] `/users` |
| FR-094 | Admin accounts and roles, TOTP (Google Authenticator) 2FA, recovery codes, audit log, OTP log | MUST | MVP | [apk-admin] `/admins`, `/audit`, `/otp-log`, `/me/recovery` |
| FR-095 | Backups and restore | SHOULD | MVP | [apk-admin] `/backups` |
| FR-096 | Operator "away" mode that pauses doubt SLA | COULD | P2 | [apk-user] `away` |

## 5. Non-functional requirements

| ID | Requirement | Source |
|---|---|---|
| NFR-001 | Bilingual (Hindi/English) content with proper Devanagari rendering; Krutidev legacy font conversion needed for imported PDFs/docs | [brief][apk-admin] `krutidev010.map` |
| NFR-002 | Cost-conscious infrastructure: "server cost" is a stated concern; team of 2 | [notebook] |
| NFR-003 | Launch aligned with Diwali (notebook header) — date and year unconfirmed, see OQ-008 | [notebook] |
| NFR-004 | Single-active-session enforcement and tamper-resistant tests (integrity log, server time `/v1/time`) | [answer][apk-user] |
| NFR-005 | Live tests and battles need low-latency realtime for synchronous start and ranking | [notebook] (inferred) |
| NFR-006 | Peak concurrency at live tests / exam days must be sized in architecture step (no figures given) | (inferred) |
| NFR-007 | Mobile-network tolerance: Chhattisgarh users on weak 3G/4G; resume/queue attempt submissions | (inferred) |
| NFR-008 | Admin security: 2FA, audit trail, CSRF, HTTPS only (prototype warns about cleartext) | [apk-admin] |
| NFR-009 | Student data privacy (phone, Google identity); India DPDP Act compliance; hosting in India region | (inferred) |
| NFR-010 | Disclaimer shown: independent app, not affiliated with CGPSC / CG Vyapam | [brief] |
| NFR-011 | Content integrity: authenticated PYQ solutions citing government sources | [brief] |

## 6. Integrations

| Integration | Use | Source |
|---|---|---|
| Google Sign-In | Student auth | [answer] |
| Razorpay | Paid plans | [answer] |
| Google Play Billing | Existing app verifies Play purchases; whether to keep it alongside Razorpay is OQ-001 | [apk-user] `/v1/billing/play/verify` |
| Push (FCM, inferred) | Notifications | [apk-user] |
| Google Cloud Translation | Admin machine translation EN↔HI | [apk-admin] |
| LLM (Claude / Gemini) | Optional admin AI assist for CA summaries and answer evaluation | [apk-admin] `api.anthropic.com`, `generativelanguage.googleapis.com` |
| OCR (Tesseract eng + hin) | Admin extraction of questions from scanned PDFs/images | [apk-admin] `tessdata` |
| Content portals | CG DPR, DES, finance, PIB, The Hindu, Drishti etc. for CA collection | [apk-admin] `portals.json` |
| Google Play service account | Play purchase verification | [apk-admin] settings |

## 7. Exams and content taxonomy

Exams in prototype: CGPSC State Service (Prelims, has Mains), Police Sub-Inspector (SI), Patwari, Mandi/food inspector,
Food, Other `[apk-user]`. Brief lists a wider set (Forest Service ACF/Ranger, Assistant Professor, Hostel Warden, RI, ADEO, Labour
Inspector, Rural Agriculture Officer, Platoon Commander) `[brief]`.
Subjects (17): History, Geography, Polity, Economy, Science, Environment, Culture, CG GK, Agriculture, Food, Computer, English,
Hindi, Maths, Reasoning, Aptitude, Current Affairs `[apk-user]`. 42 topics, 12 notes, ~104 sample questions, 41 sample tests exist as demo data.
Hierarchy: Exam → Test series section → Test → Section → Question; Subject → Topic → Subtopic → Question `[sketch]`.

## 8. Constraints
- Team: 2 people `[skill default]`, cost-sensitive `[notebook]`
- Three repos: frontend, backend, docs/PDF push `[user]`
- Next.js (web), React Native (mobile), Terraform AWS, GitHub Actions per pipeline defaults `[skill]`
- Content (questions, notes, PYQs) is owned/authored by the operator; copyright of PYQs and portal-scraped CA must be handled (OQ-007)

## 9. Out of scope for MVP `[answer]`
- iOS app
- Mains answer-writing evaluation and answer sets (exists in prototype; not selected for MVP) → P2
- Rewrite-in-place of the existing APKs

## 10. Assumptions
- A1 (inferred): tests default to Hindi UI with an English toggle.
- A2 (inferred): battle opponents may be bots with accuracy modelled on real user data.
- A3 (inferred): existing demo content (`demo.json`) is sample data, not production content; real content migration is not required.
- A4 (inferred): "Test series ÷ Paid audience" in the notebook means the test series is what paid users buy.

## 11. Open questions

| ID | Question | Why it matters | Proposed default |
|---|---|---|---|
| OQ-001 | Razorpay "inside the in-app purchase": does this mean Razorpay checkout opened inside the app (webview/SDK), or Google Play Billing with Razorpay on web only? | Google Play policy requires Play Billing for digital content consumed in-app; wrong choice risks store rejection and fees | Razorpay on web + Android app, with Play Billing added if Play review demands it |
| OQ-002 | Is phone number verified by OTP (SMS cost) or only collected? | SMS cost, DLT registration, fraud | Collect and verify once via OTP at first login |
| OQ-003 | Single-device rule: block the new login or log out the old one? Limit per account (1)? Allow web and app simultaneously? | UX and support load | Newest login wins; old session ends; web and app count as one |
| OQ-004 | Live test volumes: number of concurrent students at peak? Free vs paid? | Drives realtime and DB sizing | Ask in architecture step |
| OQ-005 | What does "Diwali" in the notebook mean: target launch date or festive offer? | Deadline | Treat as launch target and confirm date |
| OQ-006 | Which exams are in the launch catalogue (prototype has 6, brief lists more)? | Content workload | CGPSC SSE Prelims, SI, Patwari first |
| OQ-007 | PYQ and portal-sourced CA: licence/permission to republish? | Legal | Own-written summaries only, link out to sources |
| OQ-008 | Is "5 Sep 2026" the current date of the notebook note and Diwali 2026 the target? Today is 27 Oct 2026, after Diwali 2026 (8 Nov 2026 is Diwali; confirm) | Timeline | Confirm |
| OQ-009 | Admin roles: owner, editor, teacher, read-only? | Permission model | Those four |
| OQ-010 | Is AI answer evaluation / AI assist wanted at all (prototype has an optional toggle, currently off)? | Cost and policy | Out of MVP |
| OQ-011 | Data from existing prototype users (40 demo users) - migrate? | Migration effort | No, clean start |
| OQ-012 | Content download for offline use (prototype has 30-day offline licence): required in MVP? | Scope | SHOULD, not MUST |

## 12. Glossary
CGPSC – Chhattisgarh Public Service Commission · Vyapam – Chhattisgarh Professional Examination Board · PYQ – previous years' questions ·
CA – current affairs · CSAT – Civil Services Aptitude Test (Paper 2) · SSE – State Service Exam · RI – Revenue Inspector · ADEO – Assistant Development Extension Officer ·
Doubt – a student question answered by a teacher · Mentor – study planner · Battle – 1v1 quick quiz.

## 13. Unclassified input
- `ExamGadh-v4.2.0.apk`: 16 avatars, 8 CA card sample images (`assets/ca_cards`), `logo.png` – reusable assets, not yet imported.
- `ExamGadh-Admin-v1.2.0.apk`: `kb.tsv` static-background knowledge base (trigger phrases → fact blurbs for CA articles), FILE `questions-template`.
- Circled pie/dial doodle at the lower right of the sketch (may be an analytics/score visual) and a "Paid / Free" marker at the left margin; pie meaning is unconfirmed.
- APK binaries are not committed to this repo.

## 14. Hand-off to architecture
- **Scale hints:** none given. Prototype has 40 demo users. Architecture must ask: monthly active users, peak concurrent users on live tests / battles, content size, monthly budget.
- **Hardest requirements:** FR-003 (one active user), FR-017/FR-050 (live tests and battles with realtime sync), FR-081 (payments under Play policy), FR-013 (question-bank personalisation), NFR-002 (low server cost).
- **Questions architecture must still ask:** traffic, MAU, budget (INR/month), hosting region, DB preference, notification channel, SMS provider, CDN for media/PDFs.
- **Reuse candidates from the prototype API:** `/v1/*` endpoints (auth, content sync, attempts, battle, doubts, mentor, answers, leaderboard, billing, coupons) can serve as a contract reference, not a mandate.
