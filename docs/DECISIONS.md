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

**D-008 · Server-side auto-submit uses a BullMQ delayed job per attempt plus a periodic sweeper** (belt and braces), so expiry works with the client gone. Requires Redis (already in the signed architecture); local dev and tests use an in-process fallback.

**D-009 · Rank/percentile are computed on read** from `AttemptResult` with an index, not stored, for ranked first attempts only. Practice reattempts are never ranked.

**D-010 · Question rich text is a restricted structure, not raw HTML.** Bodies are stored as plain strings plus an allow-listed inline markup (bold, italic, sub/sup, line breaks) and optional tables as structured data. The sanitiser strips everything else on write; the renderer never uses `dangerouslySetInnerHTML` with unsanitised input.

**D-011 · Suspension takes effect on the next request** by clearing the in-process session cache on suspend (single instance) and re-reading user status on cache miss. Multi-instance caveat documented; revisit with Redis.

**D-012 · Mobile.** Until the web apps are complete, the student web app is a PWA that works on mid-range Android. The React Native app is built afterwards on `packages/api` and is tracked separately in the checklist.
