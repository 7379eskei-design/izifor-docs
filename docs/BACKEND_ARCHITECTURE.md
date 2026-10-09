# Backend Architecture

## Target stack

### Client
Current:
- React
- TypeScript
- Vite
- Tailwind CSS
- Capacitor Android wrapper

Later:
- React Native for primary iOS/Android app
- Kotlin only for Android-native modules where necessary

### Backend
- Firebase Authentication
- Cloud Firestore
- Firebase Storage
- Firebase Cloud Messaging
- Firebase App Check
- Cloud Functions
- Google Cloud Run for heavier services when needed

External integrations later:
- payment provider
- SMS provider
- KYC/identity provider
- Google Maps/Places
- Gemini AI

## Layering rule

UI should not import Firestore logic everywhere.

\`\`\`mermaid
flowchart TD
    UI[React / React Native UI] --> S[Services]
    S --> R[Repositories]
    R --> M[Mock repository]
    R --> F[Firebase repository]
    F --> A[Firebase Auth]
    F --> DB[Firestore]
    F --> ST[Storage]
    S --> FN[Cloud Functions / API]
    FN --> DB
    FN --> PAY[Payments]
    FN --> SMS[SMS / KYC / Gemini]
\`\`\`

This allows mock and Firebase implementations to coexist during migration.

## Environment strategy

### izifor-dev
Used for:
- development
- destructive schema/rule changes
- test users
- test data
- Firebase integration
- automated test environment where practical

### izifor-production
Create later, after core flows are stable and security rules are reviewed.

Never share service-account private keys in chat, source control or client apps.

## What may be client-side

Reasonable client operations after rules are implemented:
- read public routes/packages
- read own private data
- subscribe to chat
- update low-risk profile/preferences
- upload permitted files through constrained paths

## What must be server-authoritative

- accept/counter critical offer state
- route/package compatibility at final request
- escrow/payment operations
- QR creation/verification
- delivery state transitions
- payment release
- KYC status changes
- trust score changes
- privileged moderation/admin actions

## Example — Accept offer

\`\`\`mermaid
sequenceDiagram
    participant App
    participant Function
    participant Firestore
    participant PSP as Payment Provider
    participant FCM

    App->>Function: acceptOffer(offerId)
    Function->>Firestore: validate offer/package/users
    Function->>PSP: authorize/hold funds
    PSP-->>Function: success
    Function->>Firestore: transaction: accept + reject alternatives + update delivery
    Function->>FCM: notify traveler/sender
    Function-->>App: accepted
\`\`\`

## Example — AI Cargo

Client → Cloud Function/Cloud Run → Gemini → validated structured JSON → client confirmation.

AI must not silently overwrite package data and must not be trusted for exact weight/dimensions from an image.
