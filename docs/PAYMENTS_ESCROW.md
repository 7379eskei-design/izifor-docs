# Payments & Escrow

This is the target flow. The current prototype only simulates escrow values.

## Core rule

Money should not be treated as secured because the client says so.

All payment status comes from:
- payment provider API response, and/or
- verified provider webhook.

## Target flow

\`\`\`mermaid
flowchart TD
    A[Offer accepted] --> B[Create payment intent / authorization]
    B --> C{Payment secured?}
    C -->|No| D[Keep offer unconfirmed / show failure]
    C -->|Yes| E[Escrow/hold recorded]
    E --> F[Pickup verified]
    F --> G[In transit]
    G --> H[Delivery verified]
    H --> I[Release funds]
    I --> J[Traveler payout]
\`\`\`

## Required backend guarantees

- idempotency for payment commands/webhooks;
- never double-hold or double-release;
- transaction references stored;
- amount/currency validated server-side;
- accepted offer amount is the source for settlement;
- refund/dispute path available;
- audit history retained.

## States

Suggested payment state:
- none
- pending
- authorized/held
- failed
- release_pending
- released
- refund_pending
- refunded
- disputed

## Fees

Service fee and protection premium should be calculated server-side from versioned rules.

UI can show estimates, but final amount must be validated before charge/hold.
