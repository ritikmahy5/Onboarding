# DevFlow — Hackathon Build Guide

> **"Every engineer departure makes your next hire's first week faster, not slower."**

**Hackathon:** From Prototype to Product: Hands-on AI Workshop and Hackathon
**Team Size:** 3 members
**Build Window:** 24–48 hours

---

## Table of Contents

1. [Product Overview](#1-product-overview)
2. [Demo Scenario & Storyline](#2-demo-scenario--storyline)
3. [Demo Repo Setup Guide](#3-demo-repo-setup-guide)
4. [Technical Architecture](#4-technical-architecture)
5. [Agent Design & Prompts](#5-agent-design--prompts)
6. [Data Models & Vector DB Schema](#6-data-models--vector-db-schema)
7. [Frontend Specification](#7-frontend-specification)
8. [MVP Build Plan (Hour-by-Hour)](#8-mvp-build-plan-hour-by-hour)
9. [Demo Day Script](#9-demo-day-script)
10. [Pre-Demo Checklist](#10-pre-demo-checklist)

---

## 1. Product Overview

### The Problem

When an engineer leaves a team, critical knowledge vanishes — workarounds, architectural decisions, client-specific quirks, undocumented edge cases. The onboarding process for their replacement involves weeks of Slack archaeology, stale wiki pages, and interrupting senior engineers with repeated questions.

Current tools (Greptile, Sourcegraph Cody, Augment Code) can answer **"what does this code do?"** but none of them answer **"why does this exist?"** — the institutional memory that lives only in people's heads.

### The Solution

DevFlow is an engineering team lifecycle intelligence platform that connects offboarding knowledge capture with onboarding knowledge delivery. It has three modes:

- **Departure Mode:** When an engineer gives notice, DevFlow analyzes their footprint (git history, code ownership) and conducts a structured, AI-guided exit interview with targeted questions only their work history could generate.
- **Onboarding Mode:** When a new engineer joins, they can query a knowledge base that combines captured exit interview answers, git history context, PR descriptions, and code — getting grounded "why" answers with citations.
- **Knowledge Map:** A visual dashboard showing which areas of the codebase have documented institutional knowledge vs. red zones with bus-factor risk.

### Why This Wins

| Judging Criteria | How DevFlow Delivers |
|---|---|
| High Utility | Every engineering manager has lost critical knowledge when someone left. Universal pain. |
| Technical Feasibility | Multi-agent RAG pipeline over vector DB. Proven patterns, novel combination. |
| Marketability | Three distinct UI modes (departure dashboard, onboarding chat, knowledge map). Not a chatbot. |
| Trend-Alignment | Multi-agent system, RAG over heterogeneous sources, local-first AI potential. |

---

## 2. Demo Scenario & Storyline

### The Fictional Company

**EventPulse** — A startup building an event ticketing platform. The backend has 4 modules: payments, events/venues, users/auth, and notifications.

### The Cast

| Role | Person | GitHub Username | Domain Ownership | Notes |
|---|---|---|---|---|
| **The Departing Engineer** | Friend A | `[their-username]` | Payments, billing, Stripe integration, auth (early decisions) | Has the most tribal knowledge. Their departure creates the crisis. |
| **The Staying Engineer** | Friend B | `[their-username]` | Events, venues, search, some shared config | Overlaps with Friend A on a few files but doesn't deeply understand payments. |
| **The New Hire** | Friend C | `[their-username]` | Notifications (built during the shared phase) | During the demo, plays the role of someone who just joined after Friend A left. |

### The Story Arc

1. **Setup:** Three engineers built EventPulse together over several months (simulated via commit history).
2. **Crisis:** Friend A (payments expert) announces they're leaving.
3. **Capture:** DevFlow analyzes Friend A's footprint and conducts a targeted exit interview.
4. **Resolution:** Friend C joins as replacement. Uses DevFlow to get answers about Friend A's domain — and gets grounded, cited responses sourced from the exit interview + git history.
5. **Insight:** The knowledge map reveals remaining red zones where no knowledge was captured.

---

## 3. Demo Repo Setup Guide

### 3.1 Repository Structure

Create a GitHub repo called `eventpulse-backend` (Python). The code does NOT need to run — it needs believable structure and rich git history.

```
eventpulse-backend/
├── README.md
├── requirements.txt
├── config/
│   ├── settings.py              # Shared config (all contributors)
│   └── stripe_config.py         # Friend A — has magic numbers with commit explanations
├── services/
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
│   ├── timezone_helpers.py      # Friend A — has the critical timezone workaround
│   └── rate_limiter.py          # Friend B
├── tests/
│   ├── test_payments.py         # Friend A — incomplete, a known gap
│   └── test_events.py           # Friend B
└── docs/
    └── architecture.md          # Started by Friend A, outdated
```

Total: ~20-25 files, aim for 80-120 commits across all three contributors.

### 3.2 Commit Strategy

This is the most critical part. Your product mines git history, so the commits ARE the demo data.

#### Phase 1: Foundation (Commits 1-30) — All Contributors

Start with repo setup, README, basic structure. Each person creates their module scaffolding.

```
# Example commits (Friend A)
git commit -m "Initialize payments module with PaymentService base class"
git commit -m "Add Stripe SDK integration for payment processing"
git commit -m "Implement basic auth service with JWT token generation"
```

```
# Example commits (Friend B)
git commit -m "Create event service with CRUD operations"
git commit -m "Add PostGIS-based venue search with radius filtering"
```

```
# Example commits (Friend C)
git commit -m "Set up email service with SendGrid integration"
git commit -m "Add SMS notifications via Twilio for ticket confirmations"
```

#### Phase 2: Architectural Decisions (Commits 31-60) — Mostly Friend A

These commits must have DETAILED messages explaining WHY, not just what. This is the content DevFlow will surface during onboarding.

```
# CRITICAL — These are your demo gold
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

```
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

```
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

```
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

#### Phase 3: Workarounds & Hacks (Commits 61-80) — Mostly Friend A

```
git commit -m "Skip email validation for @eventpulse.com SSO accounts

Our corporate SSO (Okta) returns email addresses in a non-standard
format — 'user@EVENTPULSE.COM' with uppercase domain. The email
validation regex rejects uppercase domains per RFC 5321.

Rather than making the regex case-insensitive (which could mask
other issues), we skip validation entirely for @eventpulse.com
domains since SSO already guarantees the email is valid.

If we switch SSO providers, re-enable validation for internal accounts."
```

```
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

#### Phase 4: Cross-cutting Changes (Commits 81-100) — Multiple Contributors

```
# Friend A + Friend B collaboration
git commit -m "Fix race condition in event checkout flow

Payment confirmation webhook was arriving before the ticket
reservation lock expired, causing double-bookings for popular events.

Root cause: The reservation lock TTL was 30 seconds, but Stripe's
average webhook delivery time is 5-45 seconds. For high-demand events,
the payment intent was created, the lock expired while waiting for
Stripe, and another user grabbed the same seats.

Fix: Extended lock TTL to 5 minutes and added a 'pending_payment'
state that prevents seat reallocation even after lock expiry.
Co-authored with Friend B who owns the ticket_allocator.

Co-authored-by: [Friend B] <[email]>"
```

#### Phase 5: Recent Changes (Commits 100+) — Final state before departure

Add a few maintenance commits, minor bug fixes, and leave some things explicitly incomplete:

```
git commit -m "TODO: Add test coverage for international refund edge cases

International refunds have different processing windows depending on
the card network (Visa: 5-10 days, Mastercard: 3-5 days, Amex: up to
30 days). Currently no tests for the timeout handling.

Marking as tech debt — will address next sprint."
```

### 3.3 Pull Requests (Create at Least 5-6 Real PRs)

Even if you merge them immediately, the PR descriptions are indexed by DevFlow.

**PR #1: "Implement Stripe webhook signature verification"**
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
- Unit tests for valid/invalid signatures ✅
- Integration test with Stripe CLI webhook forwarding ✅
- Load test: 1000 webhooks/minute with 8-minute simulated skew ✅
```

**PR #2: "Split AccountService from UserService"**
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

**PR #3: "Add Acme Corp custom invoice formatting"**
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

**PR #4: "Fix checkout race condition with extended reservation locks"**
```
## What
Extend ticket reservation lock TTL from 30s to 5min. Add 'pending_payment'
state to prevent seat reallocation during payment processing.

## Root Cause
Stripe webhook delivery: 5-45 seconds average.
Previous lock TTL: 30 seconds.
Result: For high-demand events, seats were released while payment was
still processing, allowing double-bookings.

## Co-authors
- [Friend A]: Payment service changes, idempotency handling
- [Friend B]: Ticket allocator lock extension, new pending state

## Risk
Longer locks mean seats are held longer for abandoned checkouts.
Added a cleanup job that releases pending_payment reservations after
5 minutes if no webhook confirmation arrives.
```

**PR #5: "Implement idempotency key caching for payment retries"**
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
they get a new key. This is acceptable because:
1. The 2-second window is shorter than minimum retry interval (5s)
2. Even if it happens, Stripe's own dedup catches most cases
3. We have a reconciliation job that flags duplicates daily
```

### 3.4 In-Code Comments (Seed These for the AI to Find)

Add comments in the code that carry institutional knowledge:

```python
# stripe_webhook.py

# WARNING: Do not use Stripe SDK's verify_webhook_signature()
# Clock skew issue causes ~15% failure rate. See PR #1.
# Manual HMAC verification with 10-min tolerance below.
def verify_webhook(payload, sig_header, webhook_secret):
    ...

# IMPORTANT: Tolerance window is intentionally wider than Stripe's
# recommendation (5 min). Our event processing is idempotent,
# so the risk of accepting a replayed webhook is minimal compared
# to the risk of silently dropping a legitimate payment confirmation.
WEBHOOK_TOLERANCE_SECONDS = 600  # 10 minutes
```

```python
# timezone_helpers.py

# HACK: Force UTC for all Stripe timestamp comparisons.
# Stripe dashboard is set to PST but API returns UTC.
# Webhook 'created' field uses merchant timezone (PST).
# This caused a 3-hour refund eligibility window bug.
# Stripe support ticket: STK-847291 — no fix ETA.
# Check quarterly if Stripe has resolved this.
# Last checked: [current month/year]
def normalize_stripe_timestamp(ts):
    ...
```

```python
# account_service.py

# NOTE: AccountService was split from UserService during the
# Black Friday incident. See PR #2 for full context.
#
# This service INTENTIONALLY depends on UserService for RBAC checks.
# Do NOT duplicate permission logic here. If RBAC becomes its own
# service, remove this dependency.
from services.users.user_service import check_permission
```

```python
# invoice_generator.py

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

---

## 4. Technical Architecture

### 4.1 System Overview

```
┌─────────────────────────────────────────────────────────┐
│                     FRONTEND (React)                     │
│  ┌──────────────┐ ┌───────────────┐ ┌────────────────┐  │
│  │  Departure    │ │  Onboarding   │ │  Knowledge     │  │
│  │  Dashboard    │ │  Assistant    │ │  Map           │  │
│  └──────┬───────┘ └──────┬────────┘ └───────┬────────┘  │
└─────────┼────────────────┼──────────────────┼────────────┘
          │                │                  │
          ▼                ▼                  ▼
┌─────────────────────────────────────────────────────────┐
│                  BACKEND (Python / FastAPI)               │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │              LangGraph Orchestrator               │   │
│  │                                                    │   │
│  │  ┌─────────────┐ ┌──────────────┐ ┌────────────┐ │   │
│  │  │ Footprint   │ │  Interview   │ │  Knowledge  │ │   │
│  │  │ Analyzer    │ │  Generator   │ │  Retriever  │ │   │
│  │  │ Agent       │ │  Agent       │ │  Agent      │ │   │
│  │  └─────────────┘ └──────────────┘ └────────────┘ │   │
│  └──────────────────────────────────────────────────┘   │
│                          │                               │
│              ┌───────────┼───────────┐                   │
│              ▼           ▼           ▼                   │
│       ┌──────────┐ ┌─────────┐ ┌──────────┐            │
│       │ Git Log  │ │ Vector  │ │ Claude   │            │
│       │ Parser   │ │ DB      │ │ API      │            │
│       │          │ │(Chroma/ │ │          │            │
│       │          │ │ Qdrant) │ │          │            │
│       └──────────┘ └─────────┘ └──────────┘            │
└─────────────────────────────────────────────────────────┘
```

### 4.2 Tech Stack

| Component | Technology | Why |
|---|---|---|
| Backend Framework | Python + FastAPI | Fast to build, great async support |
| Agent Orchestration | LangGraph | Multi-agent workflows with state management |
| LLM | Claude (Anthropic API) | Strong at structured analysis, long context window |
| Vector Database | Chroma (simpler) or Qdrant (more features) | Chroma for speed of setup, Qdrant if you want metadata filtering |
| Embeddings | Voyage AI or OpenAI `text-embedding-3-small` | Fast, cheap, good quality for code + natural language |
| Git Parsing | Python `subprocess` + `gitpython` | Parse git log, blame, show |
| Frontend | React + Tailwind CSS | Fast UI development |
| Charts/Visualization | Recharts or D3.js | Knowledge map treemap/heatmap |

### 4.3 API Endpoints

```
POST   /api/departure/analyze          # Analyze departing engineer's footprint
POST   /api/departure/generate-interview # Generate exit interview questions
POST   /api/departure/save-answers     # Store exit interview answers
POST   /api/onboarding/query           # New hire asks a question (RAG)
GET    /api/knowledge/map              # Get knowledge coverage data
GET    /api/knowledge/gaps             # Get undocumented areas
```

---

## 5. Agent Design & Prompts

### 5.1 Agent 1: Footprint Analyzer

**Input:** GitHub repo URL + departing engineer's username.
**Output:** Structured footprint (owned files, commit frequency, co-authors, key PRs).

**Process:**
1. Clone repo / parse pre-exported git log
2. Run `git log --author="[username]" --format="%H|%s|%ad|%b" --date=short`
3. Run `git log --author="[username]" --name-only` to get files touched
4. Calculate ownership: files where this person authored >50% of commits
5. Identify co-author relationships from `Co-authored-by` lines
6. Extract PR descriptions (via GitHub API or pre-exported JSON)

**System Prompt for Analysis:**
```
You are an engineering team analyst. Given the following git history data
for an engineer who is departing, generate a structured footprint report.

For each file/module they own, assess:
- Ownership level (sole owner, primary, contributor)
- Complexity signals (number of commits, frequency of changes)
- Bus factor risk (are they the ONLY person who has touched this?)
- Key decisions (commits with detailed messages explaining "why")

Output as JSON:
{
  "engineer": "name",
  "owned_modules": [...],
  "bus_factor_risks": [...],
  "key_decisions": [...],
  "co_author_relationships": [...],
  "total_commits": N,
  "files_touched": N,
  "sole_owner_files": [...]
}
```

### 5.2 Agent 2: Interview Generator

**Input:** Footprint analysis + commit messages + PR descriptions + code comments.
**Output:** 8-10 targeted exit interview questions.

**System Prompt:**
```
You are a senior engineering manager conducting a knowledge transfer
interview with a departing engineer. You have analyzed their code
footprint and git history.

Your job is to generate targeted interview questions that extract
TRIBAL KNOWLEDGE — the undocumented context, workarounds, edge cases,
and relationship information that will vanish when this person leaves.

Rules:
- Do NOT ask generic questions ("What does this module do?")
- DO ask specific questions based on their actual commits and code
- Focus on: WHY decisions were made, known workarounds/hacks,
  client-specific context, edge cases, things that broke before,
  things that WILL break if someone changes them without context
- Reference specific files, commits, or PRs in your questions
- Ask about human relationships (who to contact, who knows what)

For each question, include:
- The question text
- WHY you're asking (what signal from their history triggered this)
- Which files/modules this relates to
- Priority (critical / important / nice-to-have)
```

**Example output the agent should generate:**
```json
{
  "questions": [
    {
      "id": "Q1",
      "question": "Your webhook verification uses a 10-minute tolerance window instead of Stripe's SDK. The commit mentions clock skew issues. Can you explain what happens if we ever need to tighten this tolerance? Are there monitoring gaps we should be aware of?",
      "reason": "Commit abc123 mentions a specific Stripe support ticket and a manual HMAC implementation. No monitoring for clock skew was mentioned.",
      "related_files": ["services/payments/stripe_webhook.py", "config/stripe_config.py"],
      "priority": "critical"
    },
    {
      "id": "Q2",
      "question": "You're the only contributor to the refund processing logic. The TODO in your last commit mentions missing test coverage for international refund timing. What are the specific edge cases by card network that a new developer should know about?",
      "reason": "refund_processor.py has 0 test coverage for international cases. Friend A is sole author.",
      "related_files": ["services/payments/refund_processor.py"],
      "priority": "critical"
    },
    {
      "id": "Q3",
      "question": "The Acme Corp invoice formatting is hardcoded by client_id. You mention their AP system crashes on UTF-8. Have there been other encoding issues with Acme, and is Karen Chen still the right contact for billing escalations?",
      "reason": "PR #3 mentions specific client contacts and a legacy system. Contact info may be outdated.",
      "related_files": ["services/payments/invoice_generator.py"],
      "priority": "important"
    }
  ]
}
```

### 5.3 Agent 3: Knowledge Retriever (RAG)

**Input:** New hire's natural language question.
**Output:** Grounded answer with citations from exit interview + git history + code.

**System Prompt:**
```
You are a codebase knowledge assistant for a new engineer onboarding to
the EventPulse backend. You have access to:

1. Exit interview answers from a previous engineer (high-trust source)
2. Git commit messages and PR descriptions (medium-trust source)
3. Code comments (medium-trust source)

When answering questions:
- ALWAYS cite your sources: [Exit Interview], [Commit: abc123], [PR #2], [Code: filename.py]
- Prioritize exit interview answers — they contain the richest context
- If you combine information from multiple sources, show how they connect
- If the question has NO good answer in your knowledge base, say so
  explicitly and flag it as a "knowledge gap"
- Never hallucinate or guess at context you don't have
- Structure your answer as: Brief answer → Detailed context → Sources → Related files

Tone: Friendly, like a knowledgeable colleague explaining things over coffee.
```

---

## 6. Data Models & Vector DB Schema

### 6.1 Vector DB Collections

You need three collections in your vector DB, each storing different types of knowledge:

**Collection 1: `commit_history`**
```json
{
  "id": "commit_abc123",
  "text": "Split AccountService from UserService. UserService was handling both authentication and profile management...",
  "metadata": {
    "type": "commit",
    "hash": "abc123",
    "author": "friend-a",
    "date": "2024-06-15",
    "files_changed": ["services/users/account_service.py", "services/users/user_service.py"],
    "module": "users"
  }
}
```

**Collection 2: `exit_interviews`**
```json
{
  "id": "interview_q1_answer",
  "text": "The webhook tolerance is set to 10 minutes because our staging servers had NTP drift of 3-8 minutes. The Stripe SDK only allows 5 minutes. If you tighten it, you need to first fix NTP sync across all servers...",
  "metadata": {
    "type": "exit_interview",
    "engineer": "friend-a",
    "question_id": "Q1",
    "related_files": ["services/payments/stripe_webhook.py"],
    "module": "payments",
    "priority": "critical"
  }
}
```

**Collection 3: `pr_descriptions`**
```json
{
  "id": "pr_1",
  "text": "Implement Stripe webhook signature verification. Manual HMAC-SHA256 replacing Stripe SDK...",
  "metadata": {
    "type": "pr",
    "pr_number": 1,
    "author": "friend-a",
    "date": "2024-05-20",
    "files_changed": ["services/payments/stripe_webhook.py"],
    "module": "payments"
  }
}
```

### 6.2 Chunking Strategy

- Commit messages: One chunk per commit (include full body, not just subject line)
- PR descriptions: One chunk per PR
- Exit interview answers: One chunk per question-answer pair
- Code comments: Extract multi-line comments (3+ lines) as separate chunks with file path metadata

### 6.3 Retrieval Strategy

For each user query:
1. Embed the question
2. Search across ALL three collections (top 5 from each)
3. Re-rank results by relevance
4. Pass top 8-10 chunks as context to the Knowledge Retriever agent
5. Agent synthesizes answer with citations

---

## 7. Frontend Specification

### 7.1 Three Main Views

#### View 1: Departure Dashboard

```
┌─────────────────────────────────────────────────────────┐
│  DevFlow — Departure Mode                    [Friend A] │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ENGINEER FOOTPRINT                                      │
│  ┌──────────────────────────────────────────────────┐   │
│  │  👤 [Friend A]                                    │   │
│  │  Commits: 67  │  Files: 18  │  PRs: 5            │   │
│  │  Tenure: 8 months  │  Bus Factor Risks: 4 files  │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  MODULE OWNERSHIP                                        │
│  ┌───────────────┐ ┌───────────┐ ┌──────────────────┐  │
│  │ payments/     │ │ users/    │ │ config/          │  │
│  │ ████████ 85%  │ │ ███░ 40% │ │ ██░░ 30%         │  │
│  │ SOLE OWNER    │ │ SHARED   │ │ SHARED           │  │
│  └───────────────┘ └───────────┘ └──────────────────┘  │
│                                                          │
│  KEY DECISIONS DETECTED                                  │
│  ┌──────────────────────────────────────────────────┐   │
│  │ ⚠️  Webhook verification: manual HMAC vs SDK     │   │
│  │ ⚠️  AccountService/UserService split             │   │
│  │ ⚠️  Stripe timezone hack in timezone_helpers.py  │   │
│  │ ⚠️  Acme Corp custom invoice formatting          │   │
│  │ 🔴 Idempotency key edge case (no test coverage)  │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  EXIT INTERVIEW (8 questions generated)                  │
│  ┌──────────────────────────────────────────────────┐   │
│  │ Q1: [Critical] Your webhook verification uses... │   │
│  │ ┌─────────────────────────────────────────┐      │   │
│  │ │ [Text area for answer]                   │      │   │
│  │ └─────────────────────────────────────────┘      │   │
│  │ Related: stripe_webhook.py, stripe_config.py     │   │
│  │                                            [Save] │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

#### View 2: Onboarding Assistant

```
┌─────────────────────────────────────────────────────────┐
│  DevFlow — Onboarding Mode                  [Friend C]  │
├────────────────────────┬────────────────────────────────┤
│                        │                                │
│  FILE EXPLORER         │  KNOWLEDGE CHAT                │
│  ┌──────────────────┐  │                                │
│  │ 📁 services/     │  │  🧑 Why doesn't the webhook   │
│  │   📁 payments/   │  │     use Stripe's SDK?         │
│  │     📄 stripe_   │  │                                │
│  │       webhook.py │  │  🤖 The team chose manual     │
│  │       ★ cited    │  │  HMAC verification because    │
│  │     📄 refund_   │  │  of a clock skew issue...     │
│  │       processor  │  │                                │
│  │   📁 events/     │  │  Sources:                     │
│  │   📁 users/      │  │  📎 [Exit Interview: Q1]      │
│  │   📁 notifications│ │  📎 [PR #1: Webhook verify]   │
│  │ 📁 config/       │  │  📎 [Commit: abc123]          │
│  │ 📁 utils/        │  │                                │
│  └──────────────────┘  │  Related files:                │
│                        │  → stripe_webhook.py (line 42) │
│  CONTEXT PANEL         │  → stripe_config.py (line 8)   │
│  ┌──────────────────┐  │                                │
│  │ Currently viewing │  │  ──────────────────────────── │
│  │ sources for last  │  │                                │
│  │ answer:           │  │  🧑 What should I know about  │
│  │                   │  │     the Acme Corp account?    │
│  │ Exit Interview    │  │                                │
│  │ Answer from       │  │  🤖 Acme Corp is your largest │
│  │ [Friend A]:       │  │  enterprise client (~30% of   │
│  │ "The Stripe SDK   │  │  revenue). They use a legacy  │
│  │ has a hardcoded   │  │  AP system called LegacyFin...│
│  │ 5-minute..."      │  │                                │
│  └──────────────────┘  │  ┌──────────────────────────┐  │
│                        │  │ Ask a question...     [↵] │  │
│                        │  └──────────────────────────┘  │
└────────────────────────┴────────────────────────────────┘
```

#### View 3: Knowledge Map

```
┌─────────────────────────────────────────────────────────┐
│  DevFlow — Knowledge Map                                │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  CODEBASE KNOWLEDGE COVERAGE                             │
│                                                          │
│  ┌────────────────────────────────────────────────────┐ │
│  │                                                     │ │
│  │  ┌─────────────────────────┐  ┌────────────────┐   │ │
│  │  │     payments/           │  │   events/      │   │ │
│  │  │  ┌───────┐ ┌─────────┐ │  │  ┌──────────┐  │   │ │
│  │  │  │webhook│ │ refund  │ │  │  │ event_   │  │   │ │
│  │  │  │  🟢   │ │  🟡    │ │  │  │ service  │  │   │ │
│  │  │  │       │ │        │ │  │  │   🟢     │  │   │ │
│  │  │  └───────┘ └─────────┘ │  │  └──────────┘  │   │ │
│  │  │  ┌───────┐ ┌─────────┐ │  │  ┌──────────┐  │   │ │
│  │  │  │invoice│ │payment_ │ │  │  │ ticket_  │  │   │ │
│  │  │  │  🟢   │ │service  │ │  │  │ alloc    │  │   │ │
│  │  │  │       │ │  🟢    │ │  │  │   🟢     │  │   │ │
│  │  │  └───────┘ └─────────┘ │  │  └──────────┘  │   │ │
│  │  └─────────────────────────┘  └────────────────┘   │ │
│  │                                                     │ │
│  │  ┌─────────────────────────┐  ┌────────────────┐   │ │
│  │  │     users/              │  │ notifications/ │   │ │
│  │  │  ┌───────┐ ┌─────────┐ │  │  ┌──────────┐  │   │ │
│  │  │  │auth_  │ │account_ │ │  │  │ email_   │  │   │ │
│  │  │  │service│ │service  │ │  │  │ service  │  │   │ │
│  │  │  │  🟡   │ │  🟢    │ │  │  │   🔴     │  │   │ │
│  │  │  └───────┘ └─────────┘ │  │  └──────────┘  │   │ │
│  │  └─────────────────────────┘  └────────────────┘   │ │
│  │                                                     │ │
│  └────────────────────────────────────────────────────┘ │
│                                                          │
│  LEGEND                                                  │
│  🟢 Well-documented (exit interview + git context)       │
│  🟡 Partial coverage (git context only, no interview)    │
│  🔴 Knowledge gap (sole owner left, no context captured) │
│                                                          │
│  BUS FACTOR ALERTS                                       │
│  ⚠️  refund_processor.py — sole owner departed,          │
│     international refund edge cases undocumented          │
│  ⚠️  auth_service.py — early decisions by [Friend A],    │
│     no exit interview coverage for auth module            │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 7.2 Color & Design Notes

- Keep the UI clean and professional. Dark sidebar, light main panel.
- Use green/yellow/red consistently for knowledge coverage status.
- The onboarding chat should feel like a Slack conversation, not a terminal.
- Citations should be clickable — expanding to show the source text.
- The knowledge map treemap should be interactive — click a module to see detail.

---

## 8. MVP Build Plan (Hour-by-Hour)

### Pre-Hackathon (Night Before or Week Before)

| Task | Owner | Time |
|---|---|---|
| Create `eventpulse-backend` repo on GitHub | Everyone | 1 hour |
| Phase 1 commits: foundation and scaffolding | Everyone | 1 hour |
| Phase 2-5 commits: architectural decisions, workarounds, cross-cutting | Mostly Friend A | 2 hours |
| Create 5-6 PRs with detailed descriptions | Friend A | 30 min |
| Write exit interview answers for Friend A (Google Doc first) | Friend A | 30 min |
| Total pre-work | | ~5 hours |

### Hackathon Day 1 (Hours 1-12)

| Hour | Task | Owner |
|---|---|---|
| 1-2 | Set up Python backend (FastAPI scaffold, project structure) | Backend Dev |
| 1-2 | Set up React frontend (create-react-app/Vite, Tailwind, routing) | Frontend Dev |
| 1-2 | Set up vector DB (Chroma), write ingestion scripts for git log parsing | Data/AI Dev |
| 3-4 | Build git log parser: extract commits, PR descriptions, code comments | Data/AI Dev |
| 3-4 | Build Footprint Analyzer agent (Agent 1) | Backend Dev |
| 3-4 | Build Departure Dashboard UI (View 1) | Frontend Dev |
| 5-6 | Build Interview Generator agent (Agent 2) | Backend Dev |
| 5-6 | Ingest demo repo data into vector DB (all three collections) | Data/AI Dev |
| 5-6 | Build exit interview UI (question display + answer input) | Frontend Dev |
| 7-8 | Build Knowledge Retriever agent (Agent 3 — RAG pipeline) | Backend Dev |
| 7-8 | Test retrieval quality with sample onboarding questions | Data/AI Dev |
| 7-8 | Build Onboarding Chat UI (View 2) | Frontend Dev |
| 9-10 | Connect frontend to backend APIs end-to-end | Everyone |
| 11-12 | Bug fixes, test full departure → onboarding flow | Everyone |

### Hackathon Day 2 (Hours 13-24)

| Hour | Task | Owner |
|---|---|---|
| 13-14 | Build Knowledge Map visualization (View 3) | Frontend Dev |
| 13-14 | Fine-tune agent prompts based on testing | Backend Dev |
| 13-14 | Add citation rendering (clickable source links) | Data/AI Dev |
| 15-16 | Polish UI: loading states, transitions, error handling | Frontend Dev |
| 15-16 | Add "knowledge gap" detection (flag unanswered queries) | Backend Dev |
| 17-18 | End-to-end demo rehearsal #1 | Everyone |
| 19-20 | Fix issues found in rehearsal, polish edge cases | Everyone |
| 21-22 | Final demo rehearsal #2 with timing | Everyone |
| 23-24 | Prepare slides (if needed), buffer for emergencies | Everyone |

---

## 9. Demo Day Script

**Total demo time: 3-4 minutes**

### Act 1 — The Problem (30 seconds)

*[Show the EventPulse repo on GitHub]*

> "This is EventPulse — an event ticketing platform built by a team of three engineers over several months. [Friend A] owns the payment system — the most critical and complex part of the codebase. They've just given their two weeks' notice. What happens to all the knowledge in their head?"

*[Show a quick montage: the timezone hack comment, the Acme Corp workaround, the webhook decision — all things a new hire would never understand without context]*

> "These decisions, workarounds, and client-specific hacks? They're about to walk out the door."

### Act 2 — Departure Capture (60 seconds)

*[Switch to DevFlow — Departure Dashboard]*

> "DevFlow analyzed [Friend A]'s git history and built their engineering footprint. They authored 67 commits across 18 files. They're the sole owner of 4 critical files. And the system detected 5 key architectural decisions buried in their commit history."

*[Show the footprint visualization]*

> "Based on this footprint, DevFlow generated a targeted exit interview. These aren't generic questions — they're specific to what [Friend A] actually built."

*[Show Question 1 about webhook verification]*

> "Watch this question: 'Your webhook verification uses a 10-minute tolerance window instead of Stripe's SDK. Can you explain what happens if we need to tighten this?' — DevFlow found the commit that introduced this workaround and knows to ask about it."

*[Show [Friend A]'s pre-filled answer]*

> "The answers are now captured, embedded, and searchable forever."

### Act 3 — Onboarding Magic (90 seconds) ⭐ THE WOW MOMENT

*[Switch to Onboarding Mode. Friend C sits at the laptop.]*

> "Three weeks later, [Friend C] joins as [Friend A]'s replacement. Day one. They open DevFlow."

*[Friend C types: "Why doesn't the webhook verification use Stripe's SDK?"]*

> "Watch what happens."

*[DevFlow returns an answer citing the exit interview, the PR description, AND the original commit]*

> "Three sources, synthesized into one answer. The exit interview explains the clock skew issue. The PR describes the alternatives that were considered. The commit shows the actual implementation with the 10-minute tolerance. [Friend C] just got in 10 seconds what would have taken two weeks of Slack archaeology."

*[Friend C types: "What should I know about the Acme Corp account?"]*

> "This is the question that kills you when someone leaves."

*[DevFlow returns the answer about LegacyFin, the invoice formatting, Karen Chen's contact, the encoding crash]*

> "Client-specific knowledge that exists in exactly one person's head — now permanently accessible."

### Act 4 — The Map (30 seconds)

*[Switch to Knowledge Map view]*

> "And here's the strategic layer. Green means knowledge has been captured. Yellow means we have git context but no exit interview. Red means the sole contributor left and there's a knowledge gap."

*[Point to red zone on refund_processor.py]*

> "This red zone? That's the international refund logic. [Friend A] mentioned it was untested but we didn't capture the edge cases in the interview. DevFlow flags this as a bus-factor risk that needs attention NOW."

### Closing (15 seconds)

> "DevFlow turns every engineer departure into onboarding material for their replacement. The more people leave, the smarter the system gets. The knowledge stays with the team, not the individual."

---

## 10. Pre-Demo Checklist

### Data Preparation (Do This the Night Before)

- [ ] Demo repo has 80+ commits with rich messages from all 3 contributors
- [ ] At least 5 PRs with detailed descriptions are merged
- [ ] Exit interview answers for Friend A are pre-written and stored in vector DB
- [ ] Vector DB is pre-indexed with all commits, PRs, code comments
- [ ] Test all 5 demo queries and confirm good retrieval quality
- [ ] Pre-generate the footprint analysis (don't rely on live computation)

### Technical Checks

- [ ] Backend API is running and responding
- [ ] Frontend is built and connecting to backend
- [ ] Claude API key is set and working (check rate limits)
- [ ] Vector DB is loaded and retrieval is fast (<3 seconds per query)
- [ ] All three views are accessible and rendering correctly
- [ ] Knowledge map data is pre-computed

### Demo Environment

- [ ] Laptop is charged and plugged in
- [ ] Browser has DevFlow open in one tab, GitHub repo in another
- [ ] Font size is large enough for projection (Ctrl+= to zoom)
- [ ] Close all notifications, Slack, email
- [ ] Have a backup plan if the API is slow (pre-recorded screenshots)
- [ ] Internet connection is verified

### Rehearsal Checkpoints

- [ ] Full demo rehearsal #1 completed (Day 2, Hour 17)
- [ ] Timing confirmed: under 4 minutes total
- [ ] All three acts tested with live queries
- [ ] Friend C has practiced typing the demo queries quickly
- [ ] Transitions between views are smooth (no loading spinners during demo)
- [ ] Team has agreed on who speaks during each act

### Emergency Fallbacks

- If Claude API is down: Have pre-generated responses ready as static JSON
- If vector DB is slow: Pre-cache the 5 demo query results
- If WiFi fails: Run everything locally (backend + frontend + Chroma)
- If a demo query returns bad results: Have 2 backup questions tested and ready

---

## Appendix A: Sample Exit Interview Answers (Pre-write These)

### Q1: Webhook Verification Tolerance

> "The 10-minute tolerance is there because our servers had NTP drift of 3-8 minutes. The Stripe SDK's verify_webhook_signature function has a hardcoded 5-minute tolerance that you can't configure. We were getting about 15% webhook failures in staging because of this.
>
> If you ever need to tighten the tolerance, first make sure NTP is properly synced across all servers. Talk to the infra team — when I asked them to fix it, they said it was a 2-week timeline because of how our Kubernetes nodes handle time sync.
>
> The current implementation is safe because our event processing is idempotent. A replayed webhook just processes the same payment confirmation twice, which is a no-op. The real danger is the other direction — rejecting a legitimate webhook means a customer pays but we don't confirm their ticket.
>
> There's no monitoring for clock skew right now. That's something you should add — an alert if the skew exceeds 5 minutes would give you early warning."

### Q2: International Refund Edge Cases

> "The refund processing times are completely different by card network, and Stripe doesn't normalize them:
> - Visa: 5-10 business days
> - Mastercard: 3-5 business days
> - Amex: up to 30 business days (seriously)
> - Discover: 5-10 business days
>
> The biggest edge case: if a customer requests a refund on day 25 with an Amex card, we can initiate it, but if Stripe hasn't processed it within their 30-day window, it silently fails. There's no callback or webhook for this — you have to poll the refund status.
>
> We have a daily reconciliation job (refund_reconciler.py) that checks for stuck refunds, but it doesn't handle the Amex timeout specifically. I was going to add that but ran out of time.
>
> Also, international transactions have currency conversion in the refund. If the exchange rate changed between the original charge and the refund, the customer might get a slightly different amount. We eat the difference — it's not worth building logic for a few cents."

### Q3: Acme Corp Account Context

> "Acme is our biggest enterprise client — about 30% of revenue. Their AP system is called LegacyFin, and it's genuinely from 2003. It can only handle ASCII, fixed date formats (DD/MM/YYYY), and commas as decimal separators.
>
> Karen Chen is the AP manager and your main contact. She's responsive on email (karen@acmecorp.com) but the real person who can fix things on their end is David Liu (VP Finance, david.liu@acmecorp.com). Escalate to him if invoices are rejected — Karen doesn't have authority to override LegacyFin settings.
>
> One thing that bit us: they changed their tax ID format last year and didn't tell us. Our invoices were getting silently rejected for a month. Now I check with Karen quarterly to confirm their requirements haven't changed. Set a calendar reminder for this.
>
> If you ever need to add another client with custom invoice formatting, the current approach of hardcoding by client_id won't scale. I was planning to build a template system but didn't get to it. The CUSTOM_INVOICE_CLIENTS dict in invoice_generator.py is the place to start."

### Q4: AccountService / UserService Split

> "The split happened because of the Black Friday load test. Profile updates and auth token refreshes shared the same database connection pool. When traffic spiked, profile update queries (which are slow — they fetch preferences, billing info, avatar URLs) were starving the auth queries (which need to be fast — just checking a JWT against the session store).
>
> The split is intentional but not clean. AccountService imports check_permission from UserService. This is ON PURPOSE — do not duplicate the RBAC logic. If anyone ever suggests 'let's give AccountService its own permission checking,' push back hard. The RBAC logic has edge cases around role inheritance that took me two weeks to get right.
>
> If RBAC ever becomes its own microservice, then yes, remove the cross-dependency. But until then, the import is the right call."

### Q5: Stripe Timezone Hack

> "This is embarrassing but important. The Stripe dashboard is configured to PST (Pacific time) because our CEO set it up from San Francisco. The API returns timestamps in UTC. But the webhook 'created' field uses the merchant's dashboard timezone — which is PST.
>
> This means there's an 8-hour mismatch (or 7 during daylight saving) between API timestamps and webhook timestamps. I discovered this when refund eligibility was being calculated wrong for a 3-hour window each day.
>
> The fix in timezone_helpers.py forces everything to UTC. It's ugly but it works. I filed Stripe support ticket STK-847291 about this. Last I checked (two months ago), they acknowledged it as a known issue but gave no ETA.
>
> Check quarterly whether Stripe has fixed this. If they have, you can remove normalize_stripe_timestamp() and all its call sites. There's about 8 places in the codebase that call it — grep for 'normalize_stripe_timestamp' to find them all."

---

## Appendix B: Sample Onboarding Questions to Pre-test

Test these queries against your RAG pipeline and make sure the answers are good BEFORE the demo:

1. "Why doesn't the webhook verification use Stripe's SDK?" → Should cite Exit Interview Q1, PR #1, commit message
2. "What should I know about the Acme Corp account?" → Should cite Exit Interview Q3, PR #3, code comments
3. "Why are AccountService and UserService separate?" → Should cite Exit Interview Q4, PR #2, commit message
4. "What are the timezone issues with Stripe?" → Should cite Exit Interview Q5, commit message, code comments
5. "What's the risk with international refunds?" → Should cite Exit Interview Q2, TODO commit
6. "Who should I contact if there's a billing issue with Acme?" → Should cite Exit Interview Q3 (Karen Chen, David Liu)
7. "What happens if I modify the RBAC permission checks?" → Should cite Exit Interview Q4 (warning about not duplicating)

For each question, verify:
- The answer is grounded (not hallucinated)
- At least 2 sources are cited
- The answer would actually help a new engineer
- Response time is under 5 seconds

---

## Appendix C: Quick Reference Commands

### Git Log Export (for ingestion)

```bash
# Full commit log with body
git log --format="COMMIT_START%nHASH:%H%nAUTHOR:%an%nDATE:%ad%nSUBJECT:%s%nBODY:%b%nCOMMIT_END" --date=iso > git_log_export.txt

# Files changed per commit
git log --name-only --format="HASH:%H" > git_files_export.txt

# Commits by specific author
git log --author="friend-a" --format="%H|%s|%ad" --date=short > friend_a_commits.txt

# File ownership (who wrote the most lines per file)
git ls-files | while read f; do
  echo "$f $(git log --format='%an' -- "$f" | sort | uniq -c | sort -rn | head -1)"
done > file_ownership.txt
```

### Vector DB Quick Setup (Chroma)

```bash
pip install chromadb langchain-community sentence-transformers
```

```python
import chromadb

client = chromadb.PersistentClient(path="./devflow_db")

# Create collections
commits = client.get_or_create_collection("commit_history")
interviews = client.get_or_create_collection("exit_interviews")
prs = client.get_or_create_collection("pr_descriptions")
```

### FastAPI Scaffold

```bash
pip install fastapi uvicorn langchain langgraph anthropic
uvicorn main:app --reload --port 8000
```
