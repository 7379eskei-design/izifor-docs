# State Machine

The state machine is the contract for backend mutations. UI must not arbitrarily skip critical states.

## Delivery/package states

Current TypeScript model contains:

\`draft → published → matched → awaiting_pickup → in_transit → delivered → completed\`

and terminal \`cancelled\`.

Target transition rules:

\`\`\`mermaid
stateDiagram-v2
    [*] --> draft
    draft --> published
    draft --> cancelled
    published --> matched: offer accepted
    published --> cancelled
    matched --> awaiting_pickup: escrow secured + handover ready
    awaiting_pickup --> in_transit: pickup verified
    in_transit --> delivered: delivery verified
    delivered --> completed: settlement finalized
    completed --> [*]
    cancelled --> [*]
\`\`\`

The current prototype may move directly to \`awaiting_pickup\` when an offer is accepted. Backend implementation may keep \`matched\` as a short-lived explicit state or combine the transaction if product rules remain clear.

## Offer states

Current model:
- pending
- countered
- accepted
- rejected
- withdrawn

\`\`\`mermaid
stateDiagram-v2
    [*] --> pending
    pending --> countered
    countered --> countered
    pending --> accepted
    countered --> accepted
    pending --> rejected
    countered --> rejected
    pending --> withdrawn
    countered --> withdrawn
\`\`\`

Only one accepted offer may control one delivery.

## Route states

Current model:
- draft
- published
- in_progress
- completed

Target rules:
- draft can be edited freely;
- published can receive requests;
- in_progress cannot accept incompatible new capacity;
- completed is historical/read-only.

## Handover state

Handover should eventually be modeled explicitly rather than only as package fields.

Suggested:
- created
- ready
- verified
- expired
- cancelled

Each pickup/delivery token must be one-time and server-validated.

## Forbidden transitions

Examples:
- \`published → in_transit\` without accepted offer and pickup verification
- \`in_transit → completed\` without delivery confirmation
- accepting two offers for the same package
- releasing escrow before successful delivery confirmation
- validating the same QR token twice
