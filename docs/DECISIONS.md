# Decisions log

Format: ID · date · decision · why · alternatives · who can reverse.

**D-001 · 2026-10-28 · Students sign in with Google only; no password registration, no forgot/reset, no OTP.**
The signed requirements (FR-001, FR-002) say Google sign-in, one active device, phone deferred. The production prompt lists register/password/reset/OTP; per Ground Rule 1 the original wins. Email is verified by Google (`email_verified` is required). Phone verification returns when the owner asks. Reversible: yes (add providers behind `AuthService`).

**D-002 · Admin accounts use email + password + mandatory TOTP, httpOnly cookie session, CSRF header.**
Admins are not Google users; the prototype admin used TOTP. Password hashing: Node `crypto.scrypt` (no native dependency). Reversible: yes.

**D-003 · Keep the stack:** NestJS 11, Prisma 6, PostgreSQL 16, Next.js, React Native (later). No change.

**D-004 · Mains/descriptive practice (answer upload + teacher evaluation) is deferred.** The owner did not select "answer writing" in the MVP scope question (2026-10-27); the prompt says "if in the original scope". Tracked as ⛔ Deferred in the checklist, not stubbed.

**D-005 · Razorpay is the only payment gateway** (owner decision; Play Billing risk accepted, see BLOCKERS).

**D-006 · Exam patterns are data, not code.** `ExamPaper` rows hold questions, marks, minutes, correct marks, negative fraction, qualifying % (general and reserved), merit flag. Defaults are seeded and marked `unverified`; an admin must confirm them against the current official notification before launch (no network access to psc.cg.gov.in from this environment to verify).

**D-007 · Tests are versioned.** Publishing freezes a `TestVersion` with a snapshot of every question (body + answer key). Attempts bind to a version. Editing a published test creates a new draft version; old attempts keep their snapshot. Answer-key corrections go through an explicit "regrade version" admin action that is audit-logged.

**D-008 · Server-side auto-submit is a periodic sweeper (every 15 s) over running attempts past deadline + 15 s grace.** Finalising is idempotent (row lock + one `AttemptResult` per attempt), so several instances may sweep at once and a lost run costs only a delay. *Changed 2026-10-29 from "BullMQ delayed job per attempt":* at the signed scale (small, single server) a queue adds Redis and a failure mode without improving correctness. Redis arrives with realtime (live lobby, battle) and the sweeper can then be replaced by delayed jobs without changing `finalize`.

**D-009 · Rank/percentile are computed on read** from `AttemptResult` with an index, not stored, for ranked first attempts only. Practice reattempts are never ranked.

**D-010 · Question rich text is a restricted structure, not raw HTML.** Bodies are stored as plain strings plus an allow-listed inline markup (bold, italic, sub/sup, line breaks) and optional tables as structured data. The sanitiser strips everything else on write; the renderer never uses `dangerouslySetInnerHTML` with unsanitised input.

**D-011 · Suspension takes effect on the next request** by clearing the in-process session cache on suspend (single instance) and re-reading user status on cache miss. Multi-instance caveat documented; revisit with Redis.

**D-012 · Mobile.** Until the web apps are complete, the student web app is a PWA that works on mid-range Android. The React Native app is built afterwards on `packages/api` and is tracked separately in the checklist.

**D-013 · Notifications are an in-app inbox only for now.** One service (`NotificationsService.notify`) writes bilingual notifications from admin-editable templates, de-duplicated per (student, key), so periodic jobs may run twice or on two servers. Reminders (live test, plan expiry, announcements) count against the daily cap (default 2); answers to the student's own actions (doubt answered, result ready, plan ready) never do. Push, email and SMS are blocked on provider choice (BL-4); quiet hours (22:00–07:00) apply when push exists. Jobs run in-process every minute (disabled in tests).

**D-014 · Doubts desk rules.** Quota is 3 a day for everyone and 2 a month for students without an active plan (both settings). SLA is 48 h from asking; a follow-up on an answered doubt reopens it and restarts the SLA. Attachments are uploaded straight to private S3 under `doubts/<studentId>/`; the API refuses keys outside the student's own folder and serves them by 10-minute links. Operator "away" mode (FR-096, COULD) is not built.

**D-015 · Study plans are rule-based drafts that a mentor must send.** The generator (`study-plan.ts`) is deterministic: weakest topics first from the student's answers, last ~20% of the time for revision and mocks, Saturday mock, Sunday rest, never more than the hours the student gave. The student sees nothing until a teacher/admin reviews, edits and sends it. No AI is used; none is claimed.

**D-016 · Analytics advice is rule-based and says when it has too little data.** With fewer than 20 answered questions the only advice is to take a diagnostic test; topics need 5 answers before they are judged.

**D-017 · Account deletion erases personal data but keeps orders and invoices** (tax record-keeping) and test results as anonymous history. The same Google account can sign up again as a new student.
