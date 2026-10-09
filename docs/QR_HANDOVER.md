# QR Handover

The current prototype simulates QR scanning and uses generated pickup/delivery codes. Production must use one-time server-controlled handover sessions.

## Pickup

\`\`\`mermaid
sequenceDiagram
    participant S as Sender
    participant T as Traveler
    participant App
    participant Backend

    S->>App: Open pickup handover
    T->>App: Scan/enter token
    App->>Backend: verifyPickupToken(deliveryId, token)
    Backend->>Backend: validate participants, state, expiry, one-time use
    Backend-->>App: verified
    Backend->>Backend: delivery -> in_transit
\`\`\`

## Delivery

Receiver can use:
- receiver-facing link/QR/code via SMS, or
- sender/traveler device flow depending on final UX.

After successful validation:
- mark delivery delivered/completed according to settlement design;
- timestamp event;
- store evidence;
- trigger payment release;
- prompt reviews.

## Security rules

Token must be:
- random/high entropy;
- time-limited where appropriate;
- one-time;
- stored hashed or otherwise safely;
- bound to delivery and handover type;
- verified by backend.

Never allow client-only state change from scanning a locally generated QR.

## Evidence

Optional/required by delivery preferences:
- pickup photo
- delivery photo
- signature
- time
- coarse/precise location only with explicit product/privacy decision
