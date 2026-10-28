# Blockers and needed inputs

| ID | Blocker | Needed from | Impact | Workaround meanwhile |
|---|---|---|---|---|
| BL-1 | **AWS dev apply** is paused: needs the alert email address and an explicit yes to `terraform apply` of the 60-resource plan. Bootstrap (state bucket, lock table, OIDC, CI roles) is applied. | Owner | Nothing is deployed; no public URL | All work runs locally and in CI |
| BL-2 | **Razorpay test keys + webhook secret** | Owner (Razorpay dashboard) | Real payments cannot be exercised end to end; flows are tested with a gateway test double and HMAC-signed fixtures | – |
| BL-3 | **Google OAuth client IDs** (web + Android) | Owner (Google Cloud) | Real Google sign-in cannot be exercised | Verifier is behind a port; tests use a fake |
| BL-4 | **Push (FCM), email and SMS providers** | Owner | Notifications are in-app only | In-app notification centre built; delivery adapters are interfaces |
| BL-5 | **Domain name and TLS** | Owner | No HTTPS hostname for web/admin/api | Caddy config ready for a hostname |
| BL-6 | **Official CGPSC pattern verification**: this environment cannot reach psc.cg.gov.in and I must not guess marks | Owner/admin | Seeded exam patterns are flagged `unverified` | Admin edits patterns in the admin panel before launch |
| BL-7 | **Real content** (questions, notes, PYQs) | Owner/content team | Only SAMPLE content exists | Bulk import + template ready |
| BL-8 | **Google Play policy** with Razorpay-only (OQ-001a) | Owner decision | Possible store rejection | Web purchase + direct APK |
| BL-9 | Graviton (`t4g`) vCPU quota and CloudFront account verification in the new AWS account are unknown until apply | AWS | Apply may stop on one resource | Fails cleanly, re-run |
