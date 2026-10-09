# Test Scenarios

These scenarios are the minimum smoke suite before deeper QA exists.

## Priority P0 — must never break

### TEST-001 — Create published package
**Steps**
1. Create → Send a Package.
2. Complete required fields.
3. Publish.

**Expected**
- package exists;
- appears in My Deliveries/Sending;
- status is published;
- entered details are preserved.

### TEST-002 — Existing package → Request Traveler
**Steps**
1. Explore a route.
2. Request this traveler.
3. Choose a compatible existing package.
4. Send request.

**Expected**
- request/offer appears in Sent;
- correct traveler;
- correct route;
- correct package;
- correct amount;
- route request count updates.

### TEST-003 — Route → Create New Package → Request Traveler
**Steps**
1. Open traveler route.
2. Request this traveler.
3. Create a new package.
4. Complete wizard.
5. Send request.

**Expected**
- new package created;
- request created in same user action;
- request links the newly created package to the original route;
- Sent count increments;
- correct traveler receives it.

This test protects against stale React state where \`createPackage()\` succeeds but immediate \`sendOffer()\` cannot find the newly created package.

### TEST-004 — Route mismatch is blocked
**Given**
Traveler: London → Amsterdam

**When**
Package: Paris → London

**Expected**
- request is blocked;
- user sees route mismatch reason;
- no offer/request document is created.

### TEST-005 — Accept offer
**Expected**
- one offer accepted;
- competing active offers closed;
- delivery gets matched traveler;
- status advances to awaiting pickup/matched;
- handover credentials/session exists;
- escrow/payment hold starts.

### TEST-006 — Pickup handover
**Expected**
- one valid pickup token works once;
- delivery becomes in transit;
- invalid/expired/reused token is rejected.

### TEST-007 — Delivery handover
**Expected**
- valid delivery verification succeeds;
- delivery reaches delivered/completed;
- payment release is triggered once;
- review becomes available.

### TEST-008 — Chat
**Expected**
- sender and traveler see same conversation;
- sent text appears realtime;
- unauthorized user cannot read conversation.

## P1 — important

- Create/publish route
- Save/unsave route
- Counter offer
- Reject/withdraw offer
- Attach package photo/document
- Receiver data privacy
- Notifications/unread
- Cancel unmatched delivery
- Review after completion
- Verification display

## Mobile APK smoke

After important mobile changes:
- cold launch/splash
- app icon/name
- safe areas
- bottom navigation
- keyboard does not cover active controls
- Android Back behaves correctly
- external/network failure gives recoverable UI

## Automation target

Add:
- Vitest for domain/services
- Playwright for web E2E
- GitHub Actions for build + smoke tests

First automated E2E should cover TEST-003 because it already exposed a real product bug.
