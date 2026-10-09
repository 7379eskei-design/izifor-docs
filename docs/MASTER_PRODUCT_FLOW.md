# IziFor Master Product Flow

This document is the detailed product algorithm for IziFor. It is intentionally more complete than the shorter USER_FLOWS.md and is meant to help developers, QA, product, investors and future team members understand how the product is expected to behave end to end.

The diagrams describe the target product logic. Some parts are still simulated in the current prototype and will become authoritative only after Firebase/Google Cloud backend implementation.

---

# 1. Master user journey

~~~mermaid
flowchart TD
    A[App launch] --> B{Authenticated?}
    B -->|No| C[Onboarding / Login / Register]
    C --> D{Auth successful?}
    D -->|No| C
    D -->|Yes| E[Create or load user profile]
    B -->|Yes| E
    E --> F[Home]

    F --> G{What does the user want to do?}

    G -->|Send a package| S1[Create delivery]
    G -->|Find traveler| S2[Explore traveler routes]
    G -->|Publish my trip| T1[Create route]
    G -->|Find a package to carry| T2[Explore packages]
    G -->|Continue active delivery| A1[Active deliveries]
    G -->|Open chat| C1[Chat]
    G -->|Manage account| P1[Profile / Wallet / Verification / Support]

    S1 --> M1[Package published]
    S2 --> M2[Request selected traveler]
    T1 --> M3[Route published]
    T2 --> M4[Traveler sends offer]

    M1 --> N1[Travelers discover package]
    M3 --> N2[Senders discover route]
    M2 --> O1[Request / offer negotiation]
    M4 --> O1
    N1 --> O1
    N2 --> O1

    O1 --> O2{Agreement reached?}
    O2 -->|No| O3[Counter / Reject / Withdraw / Continue searching]
    O3 --> O1
    O2 -->|Yes| PAY1[Secure payment / escrow]

    PAY1 --> PAY2{Payment secured?}
    PAY2 -->|No| PAY3[Payment failure / retry / alternate method]
    PAY3 --> PAY1
    PAY2 -->|Yes| H1[Match confirmed]

    H1 --> H2[Chat + meeting coordination]
    H2 --> H3[Pickup handover]
    H3 --> H4{Pickup verified?}
    H4 -->|No| H3
    H4 -->|Yes| H5[Delivery in transit]

    H5 --> H6[Tracking / chat / notifications]
    H6 --> H7[Delivery handover]
    H7 --> H8{Delivery verified?}
    H8 -->|No| H7
    H8 -->|Yes| H9[Delivery completed]

    H9 --> H10[Release escrow / traveler payout]
    H10 --> H11[Reviews]
    H11 --> H12[History / receipt / support if needed]
~~~

---

# 2. Authentication and account algorithm

IziFor uses one account model. A user is not permanently registered as Sender or Traveler.

~~~mermaid
flowchart TD
    A[Open app] --> B{Existing session?}
    B -->|Yes| C[Validate session]
    C --> D{Session valid?}
    D -->|Yes| H[Enter app]
    D -->|No| E[Login]

    B -->|No| E
    E --> F{Login or Register?}

    F -->|Login| G1[Enter credentials / provider]
    G1 --> G2{Authentication success?}
    G2 -->|No| G3[Show error / reset password / retry]
    G3 --> G1
    G2 -->|Yes| G4[Load user profile]
    G4 --> H

    F -->|Register| R1[Create Auth account]
    R1 --> R2{Auth account created?}
    R2 -->|No| R3[Show error / retry]
    R3 --> R1
    R2 -->|Yes| R4[Create basic user profile]
    R4 --> R5[Optional verification prompt]
    R5 --> H

    H --> I[User can act as Sender or Traveler depending on action]
~~~

Important rules:

- No permanent role selector during registration.
- Authentication identifies the user.
- Authorization decides what the user is allowed to do.
- Verification status is separate from authentication.
- Sensitive verification/KYC state is server controlled.
- A user can publish both deliveries and travel routes from the same account.

---

# 3. Sender flow — create a package from scratch

This is the standard path when the sender starts from Create rather than from a specific traveler.

~~~mermaid
flowchart TD
    A[Create] --> B[Send a Package]
    B --> C[Pickup and destination]
    C --> C1{Required route fields valid?}
    C1 -->|No| C2[Show validation]
    C2 --> C
    C1 -->|Yes| D[Schedule]

    D --> D1[Pickup date]
    D1 --> D2[Available from / until]
    D2 --> D3[Deliver-by deadline]
    D3 --> D4[Flexible dates optional]
    D4 --> E[Package details]

    E --> E1[Title]
    E1 --> E2[Category]
    E2 --> E3[Weight]
    E3 --> E4[Quantity]
    E4 --> E5[Dimensions]
    E5 --> E6[Description]
    E6 --> E7{Required package data valid?}
    E7 -->|No| E
    E7 -->|Yes| F[Photos and documents]

    F --> F1[Add up to allowed photos]
    F1 --> F2[Attach invoice / customs docs optional]
    F2 --> G[Protection]

    G --> G1{Insurance selected?}
    G1 -->|Yes| G2[Declared value]
    G1 -->|No| H[Receiver]
    G2 --> H

    H --> H1[Receiver full name]
    H1 --> H2[Mobile number]
    H2 --> H3[Delivery address]
    H3 --> H4[Note optional]
    H4 --> I[Transport and handling]

    I --> I1[Accepted transport modes]
    I1 --> I2[Hand to hand optional]
    I2 --> I3[Fragile handling optional]
    I3 --> I4[Signature optional]
    I4 --> I5[Photo proof optional]
    I5 --> J[Pricing]

    J --> J1{Pricing model}
    J1 -->|Open to offers| J2[Travelers propose price]
    J1 -->|Fixed price| J3[Sender sets price]
    J2 --> K[Review]
    J3 --> K

    K --> K1[Review route / schedule / package / receiver / handling / price]
    K1 --> K2{Publish or save draft?}
    K2 -->|Draft| L1[Save draft]
    K2 -->|Publish| L2[Create published delivery]

    L2 --> M[Marketplace visibility]
    M --> N[Compatible travelers can discover package]
    N --> O[Offers begin]
~~~

Validation before publish should include at least:

- origin exists;
- destination exists;
- pickup address exists;
- pickup window is coherent;
- deadline is not before pickup;
- title exists;
- weight is positive;
- receiver required fields exist;
- at least one accepted transport mode;
- fixed price is positive when fixed-price mode is selected.

Later production validation should also check restricted goods, impossible values and anti-abuse rules.

---

# 4. Sender flow — find a traveler first

This is one of the most important IziFor flows.

~~~mermaid
flowchart TD
    A[Explore] --> B[Routes tab]
    B --> C[Search / filter routes]
    C --> D[Open traveler route]
    D --> E[View traveler + route details]
    E --> F{Action}

    F -->|Ask a question| G[Open conversation]
    G --> E

    F -->|Save route| H[Save for later]
    H --> E

    F -->|Request this traveler| I[Open Request Traveler sheet]
    I --> J{Compatible existing package available?}

    J -->|Yes| K[Show compatible own published packages]
    K --> L[Sender selects package]
    L --> M[Enter/request price + optional message]
    M --> V[Final compatibility validation]

    J -->|No or create new| N[Create a new package]
    N --> O[Open delivery wizard with route context]
    O --> P[Route fields remain user-controlled]
    P --> Q[Prefill useful route data]
    Q --> Q1[Departure date/time]
    Q1 --> Q2[Transport mode]
    Q2 --> Q3[Suggested price / deadline context]
    Q3 --> R[Complete all package steps]
    R --> S[Review]
    S --> V[Final compatibility validation]

    V --> W{Compatible with selected route?}
    W -->|No| X[Block request]
    X --> X1[Show exact mismatch reason]
    X1 --> Y[User edits package or chooses another route]
    Y --> V

    W -->|Yes| Z[Create request linked to package + exact route + exact traveler]
    Z --> AA[Sent tab]
    AA --> AB[Traveler receives request]
~~~

Critical atomic rule:

When the user reaches this flow through a selected route and creates a new package, pressing Send request must create the package and the request as one logical operation. It must not depend on React state re-rendering between two separate client calls.

Required link:

Package ID ↔ Route ID ↔ Sender ID ↔ Traveler ID ↔ Request/Offer ID.

The backend must repeat all route/package compatibility checks even if the client already checked them.

---

# 5. Route-package compatibility algorithm

~~~mermaid
flowchart TD
    A[Package + Route] --> B{Route published and active?}
    B -->|No| FAIL1[Reject: route unavailable]
    B -->|Yes| C{Origin compatible?}

    C -->|No| FAIL2[Reject: origin mismatch]
    C -->|Yes| D{Destination compatible?}

    D -->|No| FAIL3[Reject: destination mismatch]
    D -->|Yes| E{Package weight <= available route weight?}

    E -->|No| FAIL4[Reject: weight exceeded]
    E -->|Yes| F{Package category accepted?}

    F -->|No| FAIL5[Reject: category not accepted]
    F -->|Yes| G{Route mode allowed by package?}

    G -->|No| FAIL6[Reject: transport mismatch]
    G -->|Yes| H{Dates and times feasible?}

    H -->|No| FAIL7[Reject: schedule mismatch]
    H -->|Yes| I{Other safety restrictions pass?}

    I -->|No| FAIL8[Reject: restricted / incompatible]
    I -->|Yes| J[Compatible]

    J --> K[Calculate ranking score]
    K --> L[Use score for search/matching order]
~~~

Initial hard matching rules:

1. Origin.
2. Destination.
3. Route status.
4. Weight.
5. Category.
6. Transport mode.
7. Date/time feasibility.

Future corridor matching can allow intermediate stops, nearby airports/cities and partial route segments, but only after explicit algorithm design.

Example that must fail:

- Traveler: London → Amsterdam
- Package: Paris → London

The user may have entered valid package data, but it is not valid for that specific selected traveler route.

---

# 6. Traveler flow — publish a route

~~~mermaid
flowchart TD
    A[Create] --> B[Publish a Route]
    B --> C[Origin]
    C --> D[Destination]
    D --> E[Intermediate stops optional]
    E --> F[Departure]
    F --> G[Arrival]
    G --> H[Transport mode]
    H --> I[Vehicle / carrier info]
    I --> J[Available weight]
    J --> K[Available space]
    K --> L[Accepted categories]
    L --> M[Price per kg / pricing rules]
    M --> N[Meeting preferences / notes]
    N --> O[Review]
    O --> P{Save draft or publish?}
    P -->|Draft| Q[Route draft]
    P -->|Publish| R[Published route]
    R --> S[Visible to compatible senders]
    S --> T[Requests received]
~~~

After requests or accepted packages reduce capacity:

- available capacity must be updated consistently;
- an accepted package must not make capacity negative;
- route status and departure time must prevent late requests;
- completed/departed routes should become historical.

---

# 7. Traveler flow — find packages

~~~mermaid
flowchart TD
    A[Explore] --> B[Packages tab]
    B --> C[Browse / filter packages]
    C --> D[Open package]
    D --> E{Traveler has compatible own route?}

    E -->|No| F[Create route or leave package]
    F --> G[Publish route]
    G --> E

    E -->|Yes| H[Choose route]
    H --> I[Validate compatibility]
    I --> J{Valid?}
    J -->|No| K[Explain mismatch]
    K --> H
    J -->|Yes| L[Enter offer amount]
    L --> M[ETA / message]
    M --> N[Send offer]
    N --> O[Sender receives offer]
    O --> P[Offer negotiation]
~~~

Traveler should never be able to make an offer from a route that cannot physically carry the package according to hard compatibility rules.

---

# 8. Requests, offers and negotiation

A request from a sender to a traveler and an offer from a traveler to a sender share one negotiation domain.

~~~mermaid
flowchart TD
    A[Request / Offer created] --> B[pending]
    B --> C{Recipient action}

    C -->|Accept| D[Attempt agreement]
    C -->|Counter| E[countered]
    C -->|Reject| F[rejected]
    C -->|Ignore| B

    E --> G{Other party action}
    G -->|Accept counter| D
    G -->|Counter again| E
    G -->|Reject| F
    G -->|Withdraw| H[withdrawn]

    B -->|Sender withdraws before agreement| H

    D --> I{Server validation still passes?}
    I -->|No| J[Agreement rejected / refresh state]
    I -->|Yes| K[Close competing offers]
    K --> L[Attach traveler to delivery]
    L --> M[Reserve route capacity]
    M --> N[Start payment / escrow]
~~~

Server checks at acceptance time:

- offer is still active;
- package is still available;
- route is still valid;
- sender and traveler are allowed to transact;
- amount/currency are valid;
- capacity is still sufficient;
- no other offer has already won;
- package is not cancelled or already in transit.

Acceptance must be atomic. Two offers must not be accepted simultaneously.

---

# 9. Payment and escrow algorithm

~~~mermaid
flowchart TD
    A[Agreement reached] --> B[Calculate authoritative amount]
    B --> C[Service fee + protection premium]
    C --> D[Create payment authorization / intent]
    D --> E{Provider result}

    E -->|Failed| F[Payment failed]
    F --> G[Retry / alternate method / cancel]
    G --> D

    E -->|Pending| H[Wait for provider/webhook]
    H --> E

    E -->|Secured| I[Record held/authorized state]
    I --> J[Delivery ready for pickup]
    J --> K[Pickup]
    K --> L[In transit]
    L --> M[Delivery verified]
    M --> N[Release funds]
    N --> O{Release result}

    O -->|Success| P[Traveler payout/receivable]
    O -->|Pending| Q[Settlement pending]
    Q --> O
    O -->|Failure| R[Operational retry / support]
~~~

Rules:

- Client UI never declares payment success.
- Provider response/webhook is authoritative.
- Hold/release/refund operations are idempotent.
- Repeated webhook delivery must not duplicate money movement.
- Payment records should be auditable.

---

# 10. Chat and negotiation communication

~~~mermaid
flowchart TD
    A[Conversation created] --> B[Participants loaded]
    B --> C[Realtime message stream]
    C --> D{Message type}
    D -->|Text| E[Send text]
    D -->|Photo| F[Upload image then send reference]
    D -->|Document| G[Upload document then send reference]
    D -->|Location| H[Send permitted location]
    D -->|Voice| I[Upload audio then send reference]
    D -->|System event| J[Server-generated event]
    D -->|Offer event| K[Negotiation event]

    E --> L[Persist message]
    F --> L
    G --> L
    H --> L
    I --> L
    J --> L
    K --> L

    L --> M[Update conversation preview/unread]
    M --> N[FCM push to other participant]
    N --> C
~~~

Rules:

- Only participants can read/write the conversation.
- Sensitive files use protected Storage paths.
- Push notifications are not authoritative state.
- Opening the app should refresh actual backend state.

---

# 11. Pickup handover algorithm

~~~mermaid
flowchart TD
    A[Matched delivery + payment secured] --> B[Create pickup handover session]
    B --> C[Generate one-time token / QR]
    C --> D[Sender and traveler meet]
    D --> E[Check package condition/details]
    E --> F{Evidence required?}
    F -->|Yes| G[Capture pickup photo / evidence]
    F -->|No| H[Scan / enter pickup token]
    G --> H
    H --> I[Backend verifies token]
    I --> J{Valid?}

    J -->|No| K[Reject scan]
    K --> K1[Reason: wrong / expired / already used / wrong delivery]
    K1 --> H

    J -->|Yes| L[Mark token used]
    L --> M[Timestamp pickup]
    M --> N[Delivery -> in_transit]
    N --> O[Notify sender / traveler / receiver as required]
    O --> P[Tracking begins]
~~~

The production system must not use a reusable static local QR as authoritative proof.

---

# 12. In-transit and tracking algorithm

~~~mermaid
flowchart TD
    A[in_transit] --> B[Delivery timeline active]
    B --> C{Tracking mode enabled?}
    C -->|No| D[Milestone/status tracking only]
    C -->|Yes| E[Traveler device sends permitted location updates]
    E --> F[Backend validates/throttles updates]
    F --> G[Authorized viewers receive updates]

    D --> H[Chat and notifications remain active]
    G --> H

    H --> I{Issue reported?}
    I -->|No| J[Continue toward destination]
    I -->|Yes| K[Support / dispute flow]
    K --> L{Can delivery continue?}
    L -->|Yes| J
    L -->|No| M[Freeze/exception state and payment handling]

    J --> N[Ready for delivery handover]
~~~

Tracking/privacy rules must be explicitly defined before production:
- who can see exact location;
- when tracking starts/stops;
- background location consent;
- retention period;
- approximate vs precise location.

---

# 13. Delivery handover algorithm

~~~mermaid
flowchart TD
    A[Traveler reaches destination] --> B[Receiver delivery handover]
    B --> C[Receiver gets SMS/link/QR/code]
    C --> D{Evidence required?}
    D -->|Photo| E[Capture photo]
    D -->|Signature| F[Capture signature]
    D -->|Both| G[Capture both]
    D -->|No| H[Continue]
    E --> H
    F --> H
    G --> H

    H --> I[Scan / enter delivery token]
    I --> J[Backend verifies token]
    J --> K{Valid?}
    K -->|No| L[Reject and explain]
    L --> I

    K -->|Yes| M[Mark handover token used]
    M --> N[Timestamp delivery]
    N --> O[Delivery -> delivered/completed]
    O --> P[Trigger escrow release]
    P --> Q[Notify participants]
    Q --> R[Ask for review]
~~~

Receiver account is optional. A secure receiver-facing flow can work through SMS/link/code.

---

# 14. Review and reputation algorithm

~~~mermaid
flowchart TD
    A[Completed delivery] --> B[Review becomes available]
    B --> C{User submits review?}
    C -->|No| D[Optional reminder]
    D --> C
    C -->|Yes| E[Rating]
    E --> F[Text feedback optional]
    F --> G[Safety/report flags if needed]
    G --> H[Persist review]
    H --> I[Recalculate public rating]
    I --> J[Update reputation metrics]
~~~

Trust score should not be a simple user-editable value. It can later combine:

- identity verification;
- successful deliveries;
- ratings;
- cancellations;
- disputes;
- response behavior;
- account age;
- fraud/safety signals.

---

# 15. Cancellation, failure and dispute branches

Not every delivery finishes normally.

~~~mermaid
flowchart TD
    A[Active transaction] --> B{Problem type}

    B -->|Before offer accepted| C[Sender cancels request]
    C --> C1[Close active offers]
    C1 --> C2[No delivery settlement]

    B -->|Traveler cancels route/request| D[Notify sender]
    D --> D1[Release reserved capacity / rematch]

    B -->|Payment failed| E[Keep delivery unconfirmed]
    E --> E1[Retry or cancel]

    B -->|Pickup disagreement| F[Do not mark in transit]
    F --> F1[Open support / dispute]

    B -->|Damage / loss in transit| G[Open claim/dispute]
    G --> G1[Freeze relevant escrow state]
    G1 --> G2[Collect evidence]
    G2 --> G3[Support decision]

    B -->|Delivery cannot be verified| H[Do not release funds automatically]
    H --> H1[Support / alternative verification]

    G3 --> I{Resolution}
    I -->|Release traveler| J[Settlement]
    I -->|Refund sender| K[Refund]
    I -->|Partial/manual| L[Manual resolution]
~~~

---

# 16. Notifications algorithm

Important product events should create an in-app notification and, when appropriate, an FCM push.

~~~mermaid
flowchart LR
    A[Domain event] --> B[Create notification record]
    B --> C{User currently active?}
    C -->|Yes| D[Realtime/in-app update]
    C -->|No| E[Send FCM push]
    D --> F[Open target screen]
    E --> F
    F --> G[Refresh authoritative backend state]
~~~

Core events:

- new traveler request;
- new package offer;
- counter offer;
- offer accepted/rejected/withdrawn;
- new chat message;
- payment secured/failed/released/refunded;
- pickup ready/confirmed;
- in-transit update;
- delivery confirmed;
- review reminder;
- verification status;
- support/dispute update.

---

# 17. Profile, verification and safety

~~~mermaid
flowchart TD
    A[Profile] --> B{Section}
    B -->|Personal info| C[Edit allowed profile fields]
    B -->|Verification| D[Start verification]
    B -->|Reviews| E[View ratings/reviews]
    B -->|Saved routes| F[View saved]
    B -->|Addresses| G[Manage saved addresses]
    B -->|Wallet| H[Transactions / payout methods]
    B -->|Support| I[Tickets / insurance / report user]
    B -->|Settings| J[Notifications / privacy / app settings]

    D --> D1[Submit required data]
    D1 --> D2[Provider/server verification]
    D2 --> D3{Result}
    D3 -->|Verified| D4[Update verified status server-side]
    D3 -->|Failed| D5[Explain/retry/support]
~~~

Safety tools:

- block/report user;
- dispute transaction;
- verification badges;
- transaction evidence;
- chat history;
- handover timestamps;
- protected receiver data;
- support ticket trail.

---

# 18. AI Cargo Assist future algorithm

~~~mermaid
flowchart TD
    A[User enters title/category or adds photo] --> B[Request AI Assist]
    B --> C[Cloud Function / Cloud Run]
    C --> D[Send permitted structured input to Gemini]
    D --> E[Receive structured suggestion]
    E --> F[Deterministic validation]
    F --> G[Show suggestions to user]

    G --> H{User accepts?}
    H -->|Yes| I[Apply selected fields]
    H -->|No| J[Keep original user data]

    I --> K[User confirms final values]
    J --> K
~~~

AI may suggest:
- item title;
- category/subcategory;
- fragile flag;
- battery/restricted-goods risk;
- packaging;
- insurance recommendation.

AI must not silently overwrite data and should not be trusted to infer exact weight/dimensions from a photo.

---

# 19. Backend responsibility map

~~~mermaid
flowchart TD
    UI[React / React Native UI] --> SV[Services]
    SV --> RP[Repository interfaces]

    RP --> MOCK[Mock repositories during migration]
    RP --> FIRE[Firebase repositories]

    FIRE --> AUTH[Firebase Auth]
    FIRE --> FS[Firestore]
    FIRE --> STORE[Storage]

    SV --> API[Cloud Functions / Cloud Run]
    API --> FS
    API --> FCM[Firebase Cloud Messaging]
    API --> PAY[Payment provider]
    API --> SMS[SMS provider]
    API --> KYC[KYC provider]
    API --> MAPS[Maps / Places]
    API --> AI[Gemini]
~~~

Client can request actions, but server should own critical truth.

Server-authoritative:
- final route/package validation;
- accept offer;
- route capacity reservation;
- payment state;
- QR token generation and validation;
- delivery state transitions;
- payment release;
- KYC result;
- trust score;
- moderation/admin actions.

---

# 20. Data relationship map

~~~mermaid
erDiagram
    USER ||--o{ ROUTE : publishes
    USER ||--o{ DELIVERY : sends
    USER ||--o{ OFFER : creates
    ROUTE ||--o{ OFFER : receives_or_uses
    DELIVERY ||--o{ OFFER : negotiates
    DELIVERY ||--o| OFFER : matched_by
    DELIVERY ||--o{ HANDOVER_SESSION : has
    DELIVERY ||--o| CONVERSATION : has
    CONVERSATION ||--o{ MESSAGE : contains
    DELIVERY ||--o{ TRANSACTION : settles
    USER ||--o{ REVIEW : writes
    DELIVERY ||--o{ REVIEW : generates
    USER ||--o{ NOTIFICATION : receives
~~~

Core relationship rule for a matched delivery:

- one sender;
- one winning traveler;
- one winning offer/request;
- one travel route;
- zero or more negotiation attempts;
- one pickup handover;
- one delivery handover;
- one financial settlement chain.

---

# 21. End-to-end happy path in one sequence

~~~mermaid
sequenceDiagram
    participant S as Sender
    participant App as IziFor App
    participant B as Backend
    participant T as Traveler
    participant PSP as Payment Provider
    participant R as Receiver

    S->>App: Create package
    App->>B: Publish delivery
    B-->>T: Matching package available
    T->>App: Send offer
    App->>B: Create offer
    B-->>S: Offer notification
    S->>App: Accept offer
    App->>B: acceptOffer
    B->>PSP: Secure payment
    PSP-->>B: Payment held
    B->>B: Accept winner + reserve route + close alternatives
    B-->>S: Match confirmed
    B-->>T: Match confirmed

    S->>App: Coordinate in chat
    T->>App: Coordinate in chat

    S->>App: Open pickup QR
    T->>App: Scan pickup
    App->>B: Verify pickup
    B->>B: Delivery -> in_transit
    B-->>S: Pickup confirmed
    B-->>T: Pickup confirmed

    T->>R: Deliver package
    R->>App: Confirm delivery code/QR
    App->>B: Verify delivery
    B->>B: Delivery -> completed
    B->>PSP: Release escrow
    PSP-->>B: Released
    B-->>S: Delivered
    B-->>T: Payment released
    B-->>R: Delivery confirmed

    S->>App: Review traveler
    T->>App: Review sender
~~~

---

# 22. Minimum critical flows to automate

The first E2E automation should protect business logic rather than visual details.

1. Register/login.
2. Publish package.
3. Publish route.
4. Request traveler with existing compatible package.
5. Route → Create New Package → Send Request.
6. Reject mismatched route/package.
7. Traveler sends offer.
8. Counter offer.
9. Accept offer atomically.
10. Prevent second accepted offer.
11. Pickup QR.
12. Reject reused pickup token.
13. Delivery QR.
14. Payment release only once.
15. Participant chat access.
16. Unauthorized chat access blocked.
17. Receiver private data not exposed publicly.
18. Cancellation before agreement.
19. Dispute/payment-freeze branch.
20. Completed delivery review.

---

# 23. Product states visible to users

Sender-facing delivery stages:

1. Draft
2. Published / Collecting offers
3. Negotiating
4. Matched
5. Waiting for pickup
6. In transit
7. Delivered
8. Completed
9. Cancelled / disputed where applicable

Traveler-facing route stages:

1. Draft
2. Published
3. Receiving requests
4. Capacity partially reserved
5. In progress
6. Completed

Offer/request stages:

1. Pending
2. Countered
3. Accepted
4. Rejected
5. Withdrawn

Payment stages:

1. None
2. Pending
3. Held / Authorized
4. Failed
5. Release pending
6. Released
7. Refund pending
8. Refunded
9. Disputed

---

# 24. Development rule

When implementation changes one of these flows:

1. Update this document if product behavior changes.
2. Update the relevant domain/state documentation.
3. Add or update automated tests.
4. Change service/repository/backend logic.
5. Change UI.
6. Re-run critical E2E flow.

The documentation describes intended behavior; the backend must enforce the critical rules even when the client UI already validates them.
