# User Flows

This file defines the intended product flows. UI labels may evolve, but these business sequences should remain explicit.

## Flow A — Sender publishes a package and waits for offers

\`\`\`mermaid
flowchart TD
    A[Home / Create] --> B[Send a Package]
    B --> C[Route details]
    C --> D[Pickup schedule]
    D --> E[Package details]
    E --> F[Photos & documents]
    F --> G[Protection]
    G --> H[Receiver]
    H --> I[Transport & handling]
    I --> J[Pricing]
    J --> K[Review]
    K --> L[Publish]
    L --> M[Travelers see request]
    M --> N[Offer received]
    N --> O{Sender decision}
    O -->|Accept| P[Escrow hold]
    O -->|Counter| N
    O -->|Reject| N
    P --> Q[Chat / meeting]
    Q --> R[Pickup QR]
    R --> S[In transit]
    S --> T[Delivery QR]
    T --> U[Payment release]
    U --> V[Review]
\`\`\`

### Required delivery creation data

**Route**
- Pickup city
- Pickup address
- Destination city
- Destination address optional
- Intermediate stops optional

**Schedule**
- Pickup date
- Available from
- Available until
- Deliver-by deadline
- Flexible dates flag

**Package**
- Title
- Category
- Weight
- Quantity
- Dimensions
- Description

**Evidence**
- Photos
- Documents

**Protection**
- Insurance selection
- Declared value

**Receiver**
- Full name
- Phone
- Delivery address
- Note optional

**Handling**
- Accepted transport modes
- Hand-to-hand
- Fragile handling
- Signature
- Photo proof

**Pricing**
- Open to offers OR fixed price

## Flow B — Sender finds a traveler first

This flow is critical and must be tested after every relevant change.

\`\`\`mermaid
flowchart TD
    A[Explore Routes] --> B[Open Route]
    B --> C[Request this traveler]
    C --> D{Use existing package?}
    D -->|Yes| E[Choose compatible published package]
    D -->|No| F[Create a new package]
    F --> G[Complete package wizard]
    E --> H[Validate route/package compatibility]
    G --> H
    H -->|Invalid| I[Block request and explain mismatch]
    H -->|Valid| J[Create request/offer linked to route + package]
    J --> K[Sent tab]
    K --> L[Traveler receives request]
    L --> M{Traveler response}
    M -->|Accept| N[Agreement / escrow]
    M -->|Counter| O[Negotiation]
    M -->|Reject| P[Request closed]
    O --> N
    N --> Q[Chat / meeting]
    Q --> R[Pickup QR]
    R --> S[In transit]
    S --> T[Delivery QR]
    T --> U[Payment release]
\`\`\`

### Important atomic rule

When a user creates a new package from an existing route and presses **Send request**, the system must create both:
1. the package, and
2. the request linked to that exact route/traveler.

The operation must not depend on UI state having already re-rendered.

## Flow C — Traveler publishes a route

\`\`\`mermaid
flowchart TD
    A[Create] --> B[Publish a Route]
    B --> C[From / To / stops]
    C --> D[Departure / arrival]
    D --> E[Transport]
    E --> F[Available weight / space]
    F --> G[Accepted categories]
    G --> H[Price / notes]
    H --> I[Review]
    I --> J[Publish]
    J --> K[Receive compatible requests]
\`\`\`

## Flow D — Traveler finds a package

\`\`\`mermaid
flowchart TD
    A[Explore Packages] --> B[Open Package]
    B --> C[Choose own route]
    C --> D[Validate compatibility]
    D --> E[Send offer]
    E --> F[Sender receives offer]
    F --> G[Negotiate / accept]
    G --> H[Escrow]
    H --> I[Pickup]
\`\`\`

## Flow E — Offer negotiation

Possible actions:
- Send
- Accept
- Counter
- Reject
- Withdraw

When one offer is accepted for a package:
- that offer becomes accepted;
- alternative active offers for the same package should be closed/rejected;
- package becomes matched/awaiting pickup;
- traveler is attached to the package;
- handover credentials are created;
- payment/escrow process begins;
- conversation is available.

## Flow F — Pickup and delivery

**Pickup**
1. Sender and traveler meet.
2. Package identity/details are checked.
3. QR/code is validated.
4. Optional proof photo is captured.
5. Delivery becomes \`in_transit\`.

**Delivery**
1. Traveler meets receiver.
2. Receiver validates delivery QR/code.
3. Optional signature/photo proof is captured.
4. Delivery becomes delivered/completed.
5. Escrow is released.
6. Sender/traveler are prompted for review.
