# Data Model

This document describes the target domain model for Firebase/Firestore. It is not yet fully implemented.

## Collections

Recommended initial collections:

- \`users\`
- \`routes\`
- \`deliveries\`
- \`offers\`
- \`conversations\`
- \`conversations/{id}/messages\`
- \`handoverSessions\`
- \`notifications\`
- \`reviews\`
- \`transactions\`
- \`verifications\`
- \`reports\`
- \`savedAddresses\`

Optional later:
- \`supportTickets\`
- \`claims\`
- \`invoices\`
- \`matchingEvents\`
- \`auditLogs\`

## users

Key fields:
- id
- firstName
- lastName
- avatarUrl
- city/country
- rating
- reviewCount
- trustScore
- verification flags/status
- joinedAt
- status

Sensitive identity/KYC details should not live in broadly readable user documents.

## routes

Key fields:
- id
- travelerId
- from
- to
- stops
- departAt
- arriveAt
- transportMode
- vehicle
- weightAvailableKg
- initialWeightAvailableKg
- spaceAvailable
- acceptedCategories
- pricePerKg
- notes
- status
- requestCount
- createdAt
- updatedAt

## deliveries

Key fields:
- id
- senderId
- travelerId nullable until match
- title
- from/to/stops
- pickupWindow
- deliverBy
- category
- weightKg
- dimensions
- quantity
- description
- media/document references
- protection/declared value
- receiver data
- transport preferences
- pricing
- status
- matchedOfferId
- progress
- createdAt
- updatedAt

Receiver phone/address must have stricter access than public marketplace fields.

## offers

Key fields:
- id
- packageId/deliveryId
- routeId
- fromUserId
- toUserId
- amount
- currency
- message
- status
- etaDays
- history
- createdAt
- updatedAt

Production negotiation history should preferably be append-only/auditable.

## conversations/messages

Conversation:
- id
- deliveryId/packageId
- participantIds
- context label
- lastMessage preview
- lastAt
- unread counters

Message:
- id
- authorId
- type: text/image/document/location/voice/system/offer
- body/reference
- metadata
- createdAt
- moderation flags if needed

## handoverSessions

- id
- deliveryId
- kind: pickup/delivery
- token hash
- status
- expiresAt
- verifiedAt
- verifiedBy
- proofPhotoRef
- signatureRef
- location evidence optional
- createdAt

Never store a reusable plaintext secret as the authoritative verification mechanism.

## transactions

- id
- deliveryId
- userId
- provider
- providerReference
- type: hold/release/refund/fee/payout
- amount
- currency
- status
- createdAt
- updatedAt

Client code must never create a successful payment transaction directly.

## Firestore design rules

- Avoid deeply coupling UI to document shapes.
- Use service/repository mapping.
- Denormalize only intentionally for read performance.
- Keep authoritative financial/security state server-written.
- Add \`createdAt\`, \`updatedAt\`, and status to major entities.
- Plan composite indexes from actual query patterns.
