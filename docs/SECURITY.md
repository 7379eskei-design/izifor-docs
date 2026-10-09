# Security

## Principles

1. Client is untrusted.
2. Critical state transitions are server-controlled.
3. Least privilege for reads/writes.
4. Sensitive receiver/KYC/payment data is not public.
5. Secrets never ship in frontend/mobile bundles.

## Firebase

Use:
- Firebase Auth
- Firestore Security Rules
- Storage Security Rules
- App Check
- Cloud Functions for privileged mutations

## Examples of server-only operations

- KYC verification result
- trust score mutation
- payment hold/release/refund
- accepting an offer atomically
- QR verification and state transition
- moderation/admin changes

## Firestore rule concepts

Users:
- public profile subset readable as needed;
- private fields readable only by owner/server.

Routes:
- public published route fields readable;
- only owner can edit allowed fields;
- server may lock fields after departure/match.

Deliveries:
- marketplace-safe summary readable where needed;
- receiver phone/address not publicly readable;
- sender owns editable fields before match;
- traveler gets sensitive details only after agreement according to policy.

Conversations:
- participant-only.

Transactions:
- owner can read;
- server writes authoritative status.

## Storage

Separate logical paths, e.g.:
- package evidence
- chat attachments
- KYC
- dispute evidence

KYC should have the strictest access.

## Secrets

Never commit:
- service account JSON
- private API keys
- PSP secret keys
- webhook secrets

Web Firebase config is not treated like a server secret, but access still relies on Auth, Security Rules and App Check.
