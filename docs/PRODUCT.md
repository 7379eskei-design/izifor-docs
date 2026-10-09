# Product Definition

## Product

**IziFor** is a peer-to-peer logistics marketplace that connects:

- **Sender** — needs to send a package.
- **Traveler** — is already traveling on a compatible route and can carry a package.
- **Receiver** — receives the package; normally does not need an IziFor account.

A single IziFor user can act as sender on one transaction and traveler on another. Sender and traveler are **contextual roles, not account types**.

## Product promise

IziFor should make this sequence safe and understandable:

\`\`\`mermaid
flowchart LR
    A[Create delivery or route] --> B[Find compatible person]
    B --> C[Negotiate terms]
    C --> D[Agree and secure payment]
    D --> E[Pickup handover]
    E --> F[In transit]
    F --> G[Delivery handover]
    G --> H[Payment release]
    H --> I[Review]
\`\`\`

## Core principles

### 1. Human-to-human marketplace
The traveler is already taking the route. The product should not behave like a taxi/courier dispatch app.

### 2. One account, multiple actions
Do not ask a user to choose a permanent Sender/Traveler role during registration.

### 3. Trust before convenience
Identity verification, route/package compatibility, QR handover, evidence and dispute history are more important than reducing one extra tap.

### 4. Server-authoritative critical actions
Payments, offer acceptance, handover state changes, QR validation and permissions must eventually be validated server-side.

### 5. Receiver can remain lightweight
The receiver can receive SMS/link/QR information without being forced to create a full account.

## Main navigation

Target mobile navigation:

- Home
- Explore
- Create
- Chat
- Profile

**Create** opens:
- Send a Package
- Publish a Route

## Current prototype

The repository currently includes screens for:
- Auth/onboarding
- Home/search
- Create delivery
- Create route
- Delivery and route details
- Offers and negotiations
- Chat
- QR handover
- Tracking
- Wallet/transactions
- Reviews
- Verification
- Support/disputes

These screens do not all represent production-ready backend behavior yet.
