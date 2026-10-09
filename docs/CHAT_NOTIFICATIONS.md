# Chat & Notifications

## Chat

Conversation is associated with a delivery/package context and counterpart.

Supported prototype message types:
- text
- image
- document
- location
- voice
- system
- offer

Target implementation:
- Firestore realtime subscription for messages;
- Storage for file attachments;
- security rules restrict reads/writes to participants;
- message creation timestamps from server;
- moderation/report references available.

## Conversation creation

A conversation should be unique enough to avoid accidental duplicate threads for the same delivery/counterparty context.

Conversation may be created:
- when a user asks a traveler a question;
- when a request/offer starts;
- when an offer is accepted.

## Push notifications

Target provider: Firebase Cloud Messaging.

Important events:
- new request to traveler
- new offer to sender
- counter-offer
- accepted/rejected/withdrawn
- new chat message
- pickup ready/confirmed
- delivery confirmed
- payment held/released/refunded
- verification status
- support/dispute updates

Push notification is not the authoritative record. The app should refresh authoritative state from backend when opened.

## Unread counts

Unread counts should eventually be server-consistent or derived in a robust way; do not trust a single local-only counter across multiple devices.
