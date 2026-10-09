# Matching Logic

Matching decides whether a package can be requested on a traveler's route and how search results are ranked.

## Hard compatibility checks

A request must be blocked if any required rule fails.

### Geography
At minimum for the current product:
- package pickup city must match route origin city;
- package destination city must match route destination city.

Future versions can support corridor/intermediate-stop matching, but this must be explicit and route-aware.

Example:

**Traveler route:** London → Amsterdam  
**Package:** Paris → London  
**Result:** invalid for this traveler.

### Weight
\`package.weightKg <= route.weightAvailableKg\`

### Category
\`route.accepts\` must contain \`package.category\`.

### Transport
At least one package-accepted transport mode must be compatible with the route's mode.

### Date/time
The package pickup window must be feasible relative to route departure and delivery deadline relative to route arrival.

The exact tolerance policy should be configurable later.

### Route status
Only routes that are active/published and not departed/closed can receive new requests.

## Existing package picker

When a sender opens **Request this traveler**, only the sender's compatible published packages should be shown.

Current prototype already filters own packages by:
- published status
- sender ownership
- accepted category
- available weight
- exact origin city
- exact destination city

Server-side matching must repeat these checks. Client filtering alone is not security.

## Creating a new package from a selected route

The selected route context may prefill:
- departure date/time
- arrival/deadline guidance
- transport mode
- suggested price

Origin/destination fields may remain editable/blank, but final submission must validate against the selected route.

## Match score — future ranking

After hard rules pass, rank by score. Suggested factors:

- route fit
- date fit
- available weight margin
- traveler rating
- trust/verification level
- successful delivery count
- response time
- price
- cancellation/dispute history

A future score can be represented as 0–100, but ranking weights must be versioned and measurable.

## Target server contract

\`validateRoutePackageCompatibility(package, route)\`

Returns conceptually:

\`\`\`json
{
  "compatible": true,
  "reasons": [],
  "score": 92
}
\`\`\`

If invalid:

\`\`\`json
{
  "compatible": false,
  "reasons": ["ORIGIN_MISMATCH", "WEIGHT_EXCEEDED"],
  "score": 0
}
\`\`\`
