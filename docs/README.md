# IziFor Product & Engineering Documentation

This folder is the source of truth for the IziFor product flow, domain rules, target backend architecture, data model, and test scenarios.

## How to use these docs

When product behavior changes, update the documentation first, then update tests, then update implementation.

**Priority order**
1. Product rules and user flows
2. State machine and matching rules
3. Data model and backend contracts
4. Automated tests
5. UI implementation

## Documents

- [MASTER_PRODUCT_FLOW.md](./MASTER_PRODUCT_FLOW.md) — detailed almost-complete product algorithm with visual flowcharts
- [PRODUCT.md](./PRODUCT.md) — product definition, roles, scope, principles
- [USER_FLOWS.md](./USER_FLOWS.md) — end-to-end sender and traveler flows
- [STATE_MACHINE.md](./STATE_MACHINE.md) — delivery, route, offer and handover states
- [MATCHING_LOGIC.md](./MATCHING_LOGIC.md) — route/package compatibility rules
- [DATA_MODEL.md](./DATA_MODEL.md) — target domain entities and Firestore collections
- [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md) — Firebase/Google Cloud architecture
- [AUTH.md](./AUTH.md) — authentication and account model
- [PAYMENTS_ESCROW.md](./PAYMENTS_ESCROW.md) — target payment and escrow workflow
- [QR_HANDOVER.md](./QR_HANDOVER.md) — pickup/delivery QR workflow
- [CHAT_NOTIFICATIONS.md](./CHAT_NOTIFICATIONS.md) — messaging and push behavior
- [SECURITY.md](./SECURITY.md) — security boundaries and rules
- [TEST_SCENARIOS.md](./TEST_SCENARIOS.md) — smoke and E2E scenarios
- [ROADMAP.md](./ROADMAP.md) — staged implementation plan

## Current implementation status

The current app is a React + TypeScript + Vite prototype packaged for Android with Capacitor. The current state is mostly in-memory mock data through \`AppContext\`. Firebase is the planned backend and must be integrated behind service/repository interfaces rather than directly throughout UI components.

Some current screens simulate production behavior such as escrow, QR scanning, attachments and notifications. These are product prototypes until backed by real server-side validation.

## Core product rule

IziFor is a **peer-to-peer logistics marketplace**. A traveler is already going from A to B and can carry a package for a sender. IziFor is not a dedicated courier-driver dispatch product.
