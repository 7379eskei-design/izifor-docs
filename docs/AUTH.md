# Authentication & Account Model

## Product rule

There is one human account model. Do **not** create separate permanent Sender and Traveler account types.

The same authenticated user can:
- create delivery requests;
- publish travel routes;
- send/receive offers;
- chat;
- receive/send reviews.

## Planned Firebase Authentication

Initial recommended methods:
1. Email/password
2. Google

Later:
- Apple Sign-In
- Phone OTP if product/business need justifies SMS cost and anti-abuse complexity

## Registration flow

Target:

\`\`\`mermaid
flowchart TD
    A[Register] --> B[Firebase Auth account]
    B --> C[Create users/{uid}]
    C --> D[Basic profile]
    D --> E[Optional verification]
    E --> F[App]
\`\`\`

Do not create a user profile before Auth succeeds.

## Authorization

Authentication answers **who the user is**. Firestore Rules / backend authorization answer **what the user may do**.

Examples:
- only sender can edit an unmatched delivery;
- only route owner can edit their route;
- only participants can read a conversation;
- only server can mark KYC as verified;
- only server can finalize payment/QR state.

## Verification

Verification is separate from Auth.

Possible checks:
- phone
- email
- government ID
- face/liveness
- address

Verification level/trust score should be computed from server-controlled facts rather than user-editable fields.
