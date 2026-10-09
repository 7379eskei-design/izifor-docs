# Roadmap

The roadmap prioritizes product correctness before full backend migration.

## Phase 0 — Demo stability
- Finish critical flow fixes
- Remove remaining legacy iZiSend branding
- APK usability/safe-area/keyboard/back behavior
- Keep demo usable with mock data

## Phase 1 — Product contract
- Documentation in /docs
- User flows
- State machine
- Matching rules
- Data model
- Smoke scenarios

## Phase 2 — Application architecture
Introduce:
- services
- repository interfaces
- mock repositories
- Firebase repositories later
- domain validation helpers

Goal: UI does not depend directly on Firestore.

## Phase 3 — Automated tests
- Vitest setup
- Playwright setup
- CI test workflow
- Critical Request Traveler flow
- Matching validation tests
- state transition tests

## Phase 4 — Firebase dev foundation
Project: \`izifor-dev\`

Enable progressively:
1. Firebase Authentication
2. Firestore
3. Security Rules
4. Storage
5. App Check
6. FCM
7. Cloud Functions

## Phase 5 — Move domains from mock to Firebase
Recommended order:
1. users
2. routes
3. deliveries
4. offers/requests
5. conversations/messages
6. files
7. notifications

Keep mock path available during early migration where practical.

## Phase 6 — Critical server workflows
- request traveler atomic operation
- accept offer transaction
- matching validation
- QR handover
- payment hold/release
- notification fanout

## Phase 7 — External services
- Maps/Places
- SMS
- Payment provider
- KYC
- Gemini AI Cargo

## Phase 8 — Production readiness
- \`izifor-production\`
- production Firebase rules
- monitoring/logging
- Crashlytics
- analytics
- backups/export policy
- abuse/rate limiting
- privacy/terms
- store signing/release
- QA regression suite

## Phase 9 — Team scale
After financing/team growth:
- formal CI/CD
- separate web/mobile/admin apps if needed
- stronger observability
- load/performance testing
- security review
- dedicated QA regression coverage
- backend services moved to Cloud Run/PostgreSQL only where justified
