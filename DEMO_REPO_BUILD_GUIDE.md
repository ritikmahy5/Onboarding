# EventPulse Demo Repo — Build Guide

> **This file is your complete instruction set. Read it fully before starting.**
> **Goal:** Create a fake but believable Python backend repo with rich git history that DevFlow (our prototype) will ingest and demo against.

---

## What You're Building

A GitHub repo called `eventpulse-backend` — a fictional event ticketing startup's backend. The code does **NOT** need to run. It needs:

1. **Believable Python files** (~20-25 files)
2. **Rich git history** (80-120 commits from 3 different authors)
3. **5-6 Pull Requests** with detailed descriptions
4. **The code itself has comments carrying institutional knowledge**

DevFlow will parse the git log, PR descriptions, and code comments to power its AI features.

---

## Team Roles & Git Authors

Each person commits **as themselves** using their own GitHub account. Assign roles:

| Role | Who | Domain Ownership | What They Commit |
|---|---|---|---|
| **Friend A (Departing Engineer)** | `[assign someone]` | Payments, billing, Stripe, auth (early), timezone hacks | Most commits (40-50). Detailed commit messages explaining WHY. This person's knowledge is what DevFlow captures. |
| **Friend B (Staying Engineer)** | `[assign someone]` | Events, venues, search, ticket allocation, rate limiting | 25-35 commits. Normal commit messages. |
| **Friend C (New Hire)** | `[assign someone]` | Notifications (email, SMS) | 15-20 commits. Smaller scope. During demo, this person uses DevFlow to ask questions. |

**IMPORTANT:** Each person MUST commit from their own GitHub account so `git log --author` works correctly. This is critical — DevFlow uses author attribution.

---

## Repository Structure

Create this exact structure:

```
eventpulse-backend/
├── README.md
├── requirements.txt
├── config/
│   ├── __init__.py
│   ├── settings.py              # Shared config (all contributors)
│   └── stripe_config.py         # Friend A — has magic numbers with commit explanations
├── services/
│   ├── __init__.py
│   ├── payments/
│   │   ├── __init__.py
│   │   ├── payment_service.py   # Friend A — core payment logic
│   │   ├── stripe_webhook.py    # Friend A — webhook handling with workarounds
│   │   ├── refund_processor.py  # Friend A — edge cases for international refunds
│   │   └── invoice_generator.py # Friend A — client-specific formatting hacks
│   ├── events/
│   │   ├── __init__.py
│   │   ├── event_service.py     # Friend B — event CRUD
│   │   ├── venue_search.py      # Friend B — search with geo queries
│   │   └── ticket_allocator.py  # Friend B — seat assignment logic
│   ├── users/
│   │   ├── __init__.py
│   │   ├── auth_service.py      # Friend A (early) + Friend B (later additions)
│   │   ├── user_service.py      # Shared
│   │   └── account_service.py   # Friend A — split from user_service (has a "why" story)
│   └── notifications/
│       ├── __init__.py
│       ├── email_service.py     # Friend C
│       └── sms_service.py       # Friend C
├── utils/
│   ├── __init__.py
│   ├── timezone_helpers.py      # Friend A — has the critical timezone workaround
│   └── rate_limiter.py          # Friend B
├── tests/
│   ├── __init__.py
│   ├── test_payments.py         # Friend A — incomplete, a known gap
│   └── test_events.py           # Friend B
└── docs/
    └── architecture.md          # Started by Friend A, outdated
```

---

## Step-by-Step Build Process

### Step 1: Create the Repo

One person creates the repo on GitHub:
```bash
mkdir eventpulse-backend && cd eventpulse-backend
git init
```

Add all three people as collaborators. Each person clones the repo.

### Step 2: Commit in Phases

**YOU MUST COMMIT IN ORDER. DO NOT BATCH EVERYTHING AT ONCE.**

The commits should look like a real project evolving over time. Each phase below lists what to commit and who commits it.

---

### Phase 1: Foundation (Commits 1-30) — All Contributors

Each person sets up their module. Normal, short commit messages here.

**Friend A commits (10-12 commits):**
```
"Initialize payments module with PaymentService base class"
"Add Stripe SDK integration for payment processing"
"Implement basic auth service with JWT token generation"
"Add user service with registration and login"
"Create payment intent creation flow"
"Add webhook endpoint for Stripe events"
"Implement basic refund processing"
"Add invoice generation with PDF support"
"Create stripe config with API keys and webhook secrets"
"Add shared settings module with database and cache config"
```

**Friend B commits (10-12 commits):**
```
"Create event service with CRUD operations"
"Add PostGIS-based venue search with radius filtering"
"Implement ticket allocator with seat assignment logic"
"Add event validation and date conflict checking"
"Create venue capacity management"
"Add search filters for events by category and date"
"Implement ticket reservation with TTL-based locking"
"Add rate limiter utility for API endpoints"
"Create event test suite with fixtures"
"Add pagination for event listing endpoint"
```

**Friend C commits (8-10 commits):**
```
"Set up email service with SendGrid integration"
"Add SMS notifications via Twilio for ticket confirmations"
"Create notification templates for booking confirmation"
"Add email queue with retry logic"
"Implement SMS rate limiting per user"
"Add notification preferences to user model"
"Create unsubscribe handling for email notifications"
"Add notification logging for debugging"
```

---

### Phase 2: Architectural Decisions (Commits 31-60) — Mostly Friend A

**THIS IS THE MOST CRITICAL PHASE.** These commits MUST have detailed multi-line messages explaining WHY. DevFlow mines these for the demo.

**Friend A MUST make these exact commits with detailed messages:**

#### Commit: Split AccountService from UserService
```bash
git commit -m "Split AccountService from UserService

UserService was handling both authentication and profile management.
During load testing, profile updates were blocking auth token refresh
because they shared the same database connection pool. Splitting into
AccountService (profile, preferences, billing info) and UserService
(auth, sessions, permissions) lets us use separate connection pools.

Note: The split is NOT clean — AccountService still calls UserService
internally for permission checks. This is intentional to avoid
duplicating the RBAC logic. If we ever refactor RBAC into its own
service, this dependency should be removed.

Related: This also fixes the intermittent 503s we saw on the login
endpoint during the Black Friday load test."
```

#### Commit: Manual HMAC webhook verification
```bash
git commit -m "Add manual HMAC webhook verification instead of Stripe SDK

Stripe's SDK webhook verification has a known issue with clock skew
greater than 5 minutes. During testing, our staging server's NTP was
drifting by 3-8 minutes, causing ~15% of webhooks to fail signature
validation. The SDK has a hardcoded 5-minute tolerance.

Implemented manual HMAC-SHA256 verification with a configurable
tolerance window (default: 10 minutes). This is intentionally more
permissive than Stripe's recommendation because:
1. Our event processing is idempotent (duplicate webhooks are safe)
2. Failed webhook verification causes silent payment failures
3. The security risk of a wider window is minimal for our use case

WARNING: If we switch payment providers, this entire module needs
rewriting — it's heavily coupled to Stripe's signature format."
```

#### Commit: Timezone hack
```bash
git commit -m "HACK: Hardcode UTC for all Stripe timestamp comparisons

Stripe's API returns timestamps in UTC but their webhook payloads
include a 'created' field that uses the MERCHANT's timezone setting
in the Stripe dashboard. Our merchant account is set to PST.

This caused a 3-hour window where refund eligibility calculations
were wrong (comparing UTC event time against PST webhook time).

Workaround: Force all Stripe-related timestamps to UTC in
timezone_helpers.py. The Stripe team acknowledged this as a known
inconsistency (support ticket #STK-847291) but no ETA on fix.

TODO: Remove this workaround when Stripe normalizes their timestamp
handling. Check quarterly."
```

#### Commit: Idempotency key caching
```bash
git commit -m "Add retry logic with exponential backoff for idempotency keys

After the March outage, we discovered that Stripe was processing
duplicate charges when our payment service retried on network timeouts.
The root cause: we were generating new idempotency keys on each retry
instead of reusing the original.

Fix: Generate the idempotency key ONCE from a hash of (user_id,
event_id, amount, timestamp_minute) and cache it for 24 hours.
The timestamp_minute granularity means a user can legitimately
buy the same ticket at the same price if they come back later,
but rapid retries within the same minute are deduplicated.

Known edge case: If a user tries to buy at 11:59:59 and the retry
happens at 12:00:01, they get a new idempotency key. This is
acceptable because the 2-second window is shorter than our
retry interval (5 seconds minimum)."
```

#### Commit: SSO email validation skip
```bash
git commit -m "Skip email validation for @eventpulse.com SSO accounts

Our corporate SSO (Okta) returns email addresses in a non-standard
format — 'user@EVENTPULSE.COM' with uppercase domain. The email
validation regex rejects uppercase domains per RFC 5321.

Rather than making the regex case-insensitive (which could mask
other issues), we skip validation entirely for @eventpulse.com
domains since SSO already guarantees the email is valid.

If we switch SSO providers, re-enable validation for internal accounts."
```

#### Commit: Acme Corp invoice hack
```bash
git commit -m "Add Acme Corp invoice format exception in invoice_generator

Acme Corp (our largest enterprise client, ~30% of revenue) uses a
legacy AP system from 2003 that can only parse invoices with:
- Date format: DD/MM/YYYY (not ISO 8601)
- Amount: no currency symbol, comma as decimal separator
- Tax: must be a separate line item, not included in total

This is hardcoded by client_id for now. If we get more clients
with custom invoice requirements, refactor into a template system.

Contact for Acme billing issues: Karen Chen (karen@acmecorp.com)
Their AP system is called 'LegacyFin' and it crashes on UTF-8."
```

**Add 5-8 more Friend A commits with shorter but still meaningful messages for other payment/auth changes.**

**Friend B adds 5-8 commits during this phase too** (normal messages for events/venues improvements).

---

### Phase 3: Cross-cutting & Bug Fixes (Commits 61-90)

#### Co-authored commit (Friend A + Friend B):
```bash
git commit -m "Fix race condition in event checkout flow

Payment confirmation webhook was arriving before the ticket
reservation lock expired, causing double-bookings for popular events.

Root cause: The reservation lock TTL was 30 seconds, but Stripe's
average webhook delivery time is 5-45 seconds. For high-demand events,
the payment intent was created, the lock expired while waiting for
Stripe, and another user grabbed the same seats.

Fix: Extended lock TTL to 5 minutes and added a 'pending_payment'
state that prevents seat reallocation even after lock expiry.

Co-authored-by: [Friend B Name] <[Friend B Email]>"
```

Add 15-20 more commits here from all three contributors — bug fixes, improvements, minor changes. Keep messages shorter but still informative.

---

### Phase 4: Final State (Commits 90+)

Leave some things explicitly incomplete:

#### Friend A's last commits:
```bash
git commit -m "TODO: Add test coverage for international refund edge cases

International refunds have different processing windows depending on
the card network (Visa: 5-10 days, Mastercard: 3-5 days, Amex: up to
30 days). Currently no tests for the timeout handling.

Marking as tech debt — will address next sprint."
```

Add a few more small commits from all contributors to fill out the history.

---

## Step 3: Create Pull Requests

Create these PRs on GitHub. You can merge them immediately — the PR descriptions are what matter.

### PR #1: "Implement Stripe webhook signature verification"
```
## What
Manual HMAC-SHA256 webhook verification replacing Stripe SDK's built-in method.

## Why
Stripe SDK has hardcoded 5-minute clock skew tolerance. Our staging server
drifts 3-8 minutes via NTP. ~15% webhook failure rate in staging.

## Alternatives Considered
1. Fix NTP sync on all servers — infra team said 2-week timeline, we needed this now
2. Use Stripe SDK with monkey-patched tolerance — fragile, breaks on SDK updates
3. Manual HMAC verification — chosen approach, full control over tolerance window

## Known Limitations
- Heavily Stripe-specific. Full rewrite needed if we switch providers.
- 10-minute tolerance is more permissive than Stripe recommends.
- No alerting if clock skew exceeds tolerance — should add monitoring.

## Testing
- Unit tests for valid/invalid signatures
- Integration test with Stripe CLI webhook forwarding
- Load test: 1000 webhooks/minute with 8-minute simulated skew
```

### PR #2: "Split AccountService from UserService"
```
## What
Separate user profile/billing management from auth/session management.

## Why
Black Friday load test showed 503s on /login when profile update traffic spiked.
Both operations shared a DB connection pool. Auth was starved of connections.

## Architecture Decision
AccountService handles: profile data, billing info, preferences
UserService handles: authentication, sessions, RBAC permissions

IMPORTANT: AccountService depends on UserService for permission checks.
This is intentional — do NOT duplicate RBAC logic. If RBAC becomes its
own service later, remove this cross-dependency.

## Migration Notes
- All existing /api/user/profile endpoints now route to AccountService
- /api/user/auth endpoints unchanged
- Database: same tables, separate connection pools
- Session tokens still managed exclusively by UserService
```

### PR #3: "Add Acme Corp custom invoice formatting"
```
## What
Client-specific invoice format for Acme Corp (client_id: ACME_001).

## Why
Acme Corp uses a legacy AP system ("LegacyFin", circa 2003) that crashes
on standard invoice formats. They represent ~30% of our enterprise revenue.

## Acme-Specific Requirements
- Date: DD/MM/YYYY (not ISO 8601)
- Amount: no currency symbol, comma as decimal (1.234,56 not $1,234.56)
- Tax: separate line item, excluded from total line
- Encoding: ASCII only — their system crashes on UTF-8

## Client Contact
Karen Chen (karen@acmecorp.com) — AP manager
Escalation: David Liu (david.liu@acmecorp.com) — VP Finance

## Future
If more clients need custom formats, refactor into a template system.
For now, hardcoded switch by client_id is simplest.
```

### PR #4: "Fix checkout race condition with extended reservation locks"
```
## What
Extend ticket reservation lock TTL from 30s to 5min. Add 'pending_payment'
state to prevent seat reallocation during payment processing.

## Root Cause
Stripe webhook delivery: 5-45 seconds average.
Previous lock TTL: 30 seconds.
Result: For high-demand events, seats were released while payment was
still processing, allowing double-bookings.

## Risk
Longer locks mean seats are held longer for abandoned checkouts.
Added a cleanup job that releases pending_payment reservations after
5 minutes if no webhook confirmation arrives.
```

### PR #5: "Implement idempotency key caching for payment retries"
```
## What
Cache Stripe idempotency keys for 24h to prevent duplicate charges on retries.

## Incident Background
March 2024 — Three users were double-charged during a network blip.
Root cause: Each retry generated a new idempotency key instead of
reusing the original. Stripe treated each as a unique payment.

## Design Decision
Key = hash(user_id, event_id, amount, timestamp_minute)

Why timestamp_minute (not timestamp_second)?
- Too granular = legitimate separate purchases get deduplicated
- Too broad = user can't buy again if first attempt failed
- Minute granularity balances both risks

## Known Edge Case
If a user initiates payment at 11:59:59 and retry happens at 12:00:01,
they get a new key. Acceptable because minimum retry interval is 5s.
```

### PR #6: "Add timezone normalization for Stripe webhook timestamps"
```
## What
Force UTC conversion for all Stripe-related timestamps.

## Why
Stripe dashboard is set to PST. API returns UTC. Webhook 'created' field
uses merchant timezone (PST). This caused a 3-hour refund eligibility bug.

## Stripe Support
Ticket #STK-847291 — acknowledged as known issue, no ETA.
Check quarterly if resolved.

## Affected Code
~8 call sites use normalize_stripe_timestamp(). Grep to find them all.
```

---

## Step 4: Code Comments (SEED THESE IN THE FILES)

These comments carry institutional knowledge. DevFlow will extract and index them.

### In `stripe_webhook.py`:
```python
# WARNING: Do not use Stripe SDK's verify_webhook_signature()
# Clock skew issue causes ~15% failure rate. See PR #1.
# Manual HMAC verification with 10-min tolerance below.
WEBHOOK_TOLERANCE_SECONDS = 600  # 10 minutes

# IMPORTANT: Tolerance window is intentionally wider than Stripe's
# recommendation (5 min). Our event processing is idempotent,
# so the risk of accepting a replayed webhook is minimal compared
# to the risk of silently dropping a legitimate payment confirmation.
```

### In `timezone_helpers.py`:
```python
# HACK: Force UTC for all Stripe timestamp comparisons.
# Stripe dashboard is set to PST but API returns UTC.
# Webhook 'created' field uses merchant timezone (PST).
# This caused a 3-hour refund eligibility window bug.
# Stripe support ticket: STK-847291 — no fix ETA.
# Check quarterly if Stripe has resolved this.
# Last checked: Feb 2024
```

### In `account_service.py`:
```python
# NOTE: AccountService was split from UserService during the
# Black Friday incident. See PR #2 for full context.
#
# This service INTENTIONALLY depends on UserService for RBAC checks.
# Do NOT duplicate permission logic here. If RBAC becomes its own
# service, remove this dependency.
from services.users.user_service import check_permission
```

### In `invoice_generator.py`:
```python
# Client-specific invoice formatting.
# Currently only Acme Corp (ACME_001) has custom requirements.
# Their AP system ("LegacyFin") is from 2003 and crashes on:
# - ISO 8601 dates
# - Currency symbols
# - UTF-8 encoding
# Contact: Karen Chen (karen@acmecorp.com) for billing issues.
# See PR #3 for full requirements.
CUSTOM_INVOICE_CLIENTS = {
    "ACME_001": {
        "date_format": "DD/MM/YYYY",
        "decimal_separator": ",",
        "include_currency_symbol": False,
        "tax_as_separate_line": True,
        "encoding": "ascii"
    }
}
```

### In `refund_processor.py`:
```python
# TODO: No test coverage for international refund timing edge cases.
# Refund processing windows by card network:
# - Visa: 5-10 business days
# - Mastercard: 3-5 business days
# - Amex: up to 30 business days
# - Discover: 5-10 business days
# The Amex 30-day timeout can cause silent failures — no webhook for this.
# Need to add specific handling. See last commit by [Friend A].
```

---

## Step 5: Requirements File

```
# requirements.txt
fastapi==0.104.1
uvicorn==0.24.0
stripe==7.0.0
sqlalchemy==2.0.23
psycopg2-binary==2.9.9
redis==5.0.1
pyjwt==2.8.0
sendgrid==6.11.0
twilio==8.10.0
geoalchemy2==0.14.2
celery==5.3.4
httpx==0.25.2
pydantic==2.5.2
python-dotenv==1.0.0
```

---

## Checklist Before You're Done

- [ ] Repo has 80+ commits total
- [ ] Friend A has 40-50 commits (most with detailed messages)
- [ ] Friend B has 25-35 commits
- [ ] Friend C has 15-20 commits
- [ ] At least 10 commits have multi-line messages explaining WHY
- [ ] 5-6 PRs created with detailed descriptions (can be merged immediately)
- [ ] Code comments with institutional knowledge are in the files listed above
- [ ] All commits are attributed to the correct GitHub user
- [ ] `git log --author="[Friend A username]"` returns their commits correctly
- [ ] The repo looks believable at a glance

---

## Timeline

You have ~2 hours. Suggested split:

| Time | Task |
|---|---|
| 0-20 min | Create repo, set up file structure, all `__init__.py` files |
| 20-50 min | Phase 1 commits — everyone commits their module foundations |
| 50-80 min | Phase 2 commits — Friend A does detailed architectural commits |
| 80-100 min | Phase 3+4 — cross-cutting changes, bug fixes, final state |
| 100-110 min | Create 5-6 PRs on GitHub |
| 110-120 min | Verify: check git log, author attribution, PR descriptions |

**The code quality matters less than the git history quality. Focus on making commits rich and realistic.**
