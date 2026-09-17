# NISHCHINT — Autonomous AI Support Teammate
## Hackathon Prototype: Complete 0→100 Build Specification

> **Purpose of this document**
>
> This document is the authoritative implementation specification for building the Nishchint prototype with Claude Code.
>
> Claude Code must treat this document as the source of truth for the prototype scope, architecture, behavior, data model, API contracts, workflows, UI, demo scenarios, safety boundaries, and implementation priorities.
>
> **Do not invent additional product scope. Do not leave core behavior ambiguous. Do not replace the architecture with a simpler chatbot implementation.**
>
> The goal is a **production-grade vertical-slice prototype**: the internal system behavior must be real and functional, while external Paytm/RBI/NPCI integrations are represented through clearly isolated mock/simulated services.

---

# 1. PROJECT CONTEXT

## 1.1 Hackathon

Event: Paytm Build for India AI Hackathon — Bengaluru Edition

Selected track:

**Track 3 — Autonomous AI Teammates**

Track objective:

Build AI teammates that go beyond answering questions and can understand context, make decisions, take actions, and work alongside human teams to deliver measurable outcomes in sales or customer service.

## 1.2 Product

Product name:

**Nishchint**

Tagline:

> **The AI support teammate that doesn't stop at replying.**

Core promise:

> **Complain once. Nishchint keeps working until the issue is resolved or safely handed to a human.**

## 1.3 Existing submitted concept

The original concept is a Paytm customer-support teammate focused first on failed UPI payment resolution.

The submitted concept defines the core loop:

1. Listen
2. Investigate
3. Decide
4. Act
5. Follow up

The prototype must make this loop genuinely functional.

---

# 2. WHAT WE ARE BUILDING

We are NOT building the complete Paytm production infrastructure.

We ARE building a working vertical slice of Nishchint.

The prototype must contain:

- Real frontend
- Real backend
- Real database/state
- Real LLM interaction
- Real agent orchestration
- Real tool calling
- Real context retrieval
- Real deterministic policy/rules engine
- Real action execution against mock external services
- Real n8n workflow automation
- Real follow-up lifecycle
- Real human escalation flow
- Real audit trail
- Real simulated clock for demonstration
- Real safety checks

External systems are mocked:

- Paytm transaction APIs
- Paytm ticketing/dispute APIs
- NPCI/real dispute integration
- Actual money movement
- Production authentication
- Production notifications

The architecture must make these integrations replaceable later without rewriting the agent.

---

# 3. CORE PRODUCT PRINCIPLE

Nishchint is not a chatbot.

A chatbot ends after generating a response.

Nishchint owns a case and continues working after the conversation ends.

The system must demonstrate:

```text
Customer complaint
        ↓
Understand
        ↓
Retrieve context
        ↓
Investigate
        ↓
Apply deterministic policy
        ↓
Decide next action
        ↓
Execute action
        ↓
Schedule follow-up
        ↓
Conversation ends
        ↓
n8n wakes workflow later
        ↓
Re-investigate
        ↓
Decide again
        ↓
Resolve / Dispute / Escalate
```

This autonomous closed loop is the most important capability in the prototype.

---

# 4. KEY TRACK-3 CAPABILITIES AND THEIR IMPLEMENTATION

Do not merely mention these terms in UI copy. Each must correspond to real system behavior.

| Capability | Actual implementation |
|---|---|
| Autonomous AI Teammate | LangGraph agent + n8n continuation workflow |
| Context Understanding | Conversation + customer history + transaction + case context |
| Decision Making | Agent workflow decisions + deterministic policy engine |
| Action Execution | Tools that create tickets, disputes, follow-ups, notifications |
| Task Automation | n8n workflows |
| Outcome Driven | Case resolution, dispute, escalation, compensation, status |
| Customer Service | Failed-payment support workflow |
| Human-AI Collaboration | Human operations dashboard + override |
| Memory | Persistent customer/case/ticket history |
| Indian Language Support | Hindi/Hinglish initially; Sarvam adapter if available |
| Auditability | Persistent audit events |
| Safety | OTP/UPI PIN guard + high-value/unclear escalation |

---

# 5. SCOPE FREEZE

## 5.1 Primary use case

Build and fully demonstrate:

> **Failed UPI merchant payment where the customer's account was debited but the merchant did not receive the payment.**

Example:

> "Mere ₹2,400 kat gaye but payment fail dikha raha hai."

This use case is the primary vertical slice.

## 5.2 Required secondary scenarios

### Scenario A — Contextual follow-up

Customer says:

> "Abhi tak paise nahi aaye."

The system must understand that this refers to the customer's existing open payment case without requiring the customer to repeat the transaction ID.

### Scenario B — High-value human escalation

A high-value transaction must be routed to a human rather than autonomously completed.

### Scenario C — Security guard

If a customer shares a UPI PIN or OTP, Nishchint must warn the customer and must not persist the sensitive value.

## 5.3 Explicitly out of scope for the current prototype

Do NOT implement these unless the core MVP is already stable:

- Real Paytm APIs
- Real NPCI APIs
- Real UPI payment execution
- Real money movement
- Complete loan workflows
- Complete insurance workflows
- Complete shopping workflows
- Complete bill-payment workflows
- Travel booking
- Full fraud detection engine
- Kubernetes
- Microservice decomposition
- Complex RAG system
- Complex vector database
- Production authentication
- Full enterprise observability stack

Travel and other Paytm services remain future expansion areas.

---

# 6. TECHNOLOGY DIRECTION

Use a simple, maintainable architecture.

## Backend

Preferred:

- Python
- FastAPI
- LangGraph
- Pydantic
- SQLAlchemy
- SQLite for fastest local prototype OR PostgreSQL if already available and stable
- LLM provider through an adapter

The architecture must isolate LLM provider-specific code.

## Frontend

Preferred:

- Next.js
- React
- TypeScript
- Tailwind CSS

Build a polished but small UI.

## Automation

- n8n Cloud
- n8n webhook + workflow nodes
- n8n is a core system component, not decorative integration

## Memory

Implement a `MemoryService` abstraction.

Initial implementation may use the application database for persistent customer/case context.

If Cognee integration is already available and can be integrated without destabilizing the MVP, implement a Cognee adapter behind the same abstraction.

Do not make the entire prototype dependent on a difficult external memory setup.

## Voice

Implement a `VoiceService` abstraction.

If Sarvam can be integrated quickly and reliably, support Hindi/Hinglish voice.

Voice must never block the core autonomous workflow.

---

# 7. HIGH-LEVEL ARCHITECTURE

```text
                    ┌─────────────────────┐
                    │    CUSTOMER UI      │
                    │      Next.js        │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │       FastAPI       │
                    │      API Layer      │
                    └──────────┬──────────┘
                               │
                               ▼
              ┌────────────────────────────────┐
              │       NISHCHINT AGENT          │
              │          LangGraph             │
              │                                │
              │ Understand → Investigate       │
              │ Decide → Act → Follow-up       │
              └──────┬────────┬────────┬───────┘
                     │        │        │
                     ▼        ▼        ▼
              ┌──────────┐ ┌────────┐ ┌────────────┐
              │ Context  │ │ Rules  │ │   Tools    │
              │ /Memory  │ │ Engine │ │            │
              └────┬─────┘ └────────┘ └─────┬──────┘
                   │                         │
                   ▼                         ▼
              ┌──────────┐          ┌─────────────────┐
              │ Database │          │ Mock Paytm APIs │
              └──────────┘          └─────────────────┘
                                             │
                          ┌──────────────────┼─────────────┐
                          ▼                  ▼             ▼
                     Transactions        Tickets       Disputes


                         ┌──────────────────────┐
                         │         n8n          │
                         │  Autonomous Workflow │
                         └──────────┬───────────┘
                                    │
                             scheduled re-check
                                    │
                                    ▼
                              Mock Paytm API
                                    │
                                    ▼
                              Nishchint Agent
                                    │
                          ┌─────────┴─────────┐
                          ▼                   ▼
                      Resolve             Escalate
                                              │
                                              ▼
                                  ┌────────────────────┐
                                  │ Human Operations   │
                                  │     Dashboard      │
                                  └────────────────────┘
```

---

# 8. ARCHITECTURAL RULES

These rules are mandatory.

## Rule 1 — LLM does not invent financial facts

The LLM must never independently invent:

- transaction status
- refund status
- refund deadline
- compensation
- dispute status
- transaction amount
- merchant receipt status

These must come from tools/database/rules.

## Rule 2 — Rules engine decides financial policy

The LLM may identify the user's intent and select the appropriate workflow.

The deterministic rules engine calculates:

- applicable policy
- T+1/T+5 deadline
- compensation
- whether the SLA is breached
- the prescribed next action
- whether the case requires human escalation based on configured thresholds

## Rule 3 — Agent acts through tools

The LangGraph agent must use explicit tools/services.

Do not let the LLM directly manipulate database tables.

## Rule 4 — External services are behind interfaces

The agent should call services such as:

```python
transaction_service.get_transaction()
ticket_service.create_ticket()
dispute_service.raise_dispute()
notification_service.notify_customer()
followup_service.schedule()
```

The current implementation uses mock services.

## Rule 5 — n8n owns asynchronous follow-up work

The initial agent creates/schedules a follow-up.

n8n continues the workflow after the conversation ends.

## Rule 6 — Every meaningful action is auditable

Create an audit event for:

- complaint received
- context retrieved
- transaction retrieved
- rule evaluated
- decision made
- ticket created
- follow-up scheduled
- follow-up executed
- transaction rechecked
- dispute raised
- compensation calculated
- escalation created
- human override
- notification sent
- sensitive credential detected

## Rule 7 — Safety overrides autonomy

The system must not blindly act autonomously in:

- high-value configured cases
- fraud/suspicious cases
- ambiguous cases
- sensitive credential situations

---

# 9. USER STORY

## Customer: Priya

```text
Customer ID: CUST001
Name: Priya Sharma
Preferred language: Hindi
```

Transaction:

```text
Transaction ID: TXN24001
Amount: ₹2,400
Type: MERCHANT
Merchant: Apollo Medicals
Transaction date: 10 Sep 2026
Status: FAILED
Customer debited: YES
Merchant credited: NO
Refund status: PENDING
```

Customer enters:

> "Mere ₹2,400 kat gaye but payment fail dikha raha hai."

The system must:

1. Understand the complaint.
2. Retrieve Priya's context.
3. Identify the likely transaction.
4. Verify transaction facts.
5. Determine it is a merchant transaction.
6. Apply the configured merchant failed-payment rule.
7. Calculate T+5 deadline.
8. Create a support case.
9. Schedule a follow-up.
10. Tell Priya the exact expected refund date.
11. End the immediate conversation.
12. Later, n8n executes the follow-up.
13. Recheck the transaction.
14. If still unresolved, raise a mock dispute.
15. Calculate configured compensation.
16. Update the case.
17. Notify Priya.
18. Record all major events.

---

# 10. POLICY / RULE ENGINE

The prototype uses a versioned policy configuration.

Important:

> This prototype must not present itself as a live legal/compliance authority. The policy data is a prototype representation based on the rules already described in the submitted hackathon concept. Production would require legal/compliance verification and maintained policy sources.

## 10.1 Merchant failed payment

Configured prototype rule:

```text
Condition:
Money debited
AND merchant not credited/confirmed
AND transaction type = MERCHANT

Refund deadline:
T + 5 calendar days

Compensation:
₹100 per day of delay according to prototype policy configuration

If deadline passes and refund remains unresolved:
Raise dispute
```

## 10.2 Person-to-person failed payment

Configured prototype rule:

```text
Condition:
Money debited
AND receiver not credited
AND transaction type = PERSON

Refund deadline:
T + 1 calendar day

Compensation:
₹100 per day of delay according to prototype policy configuration
```

## 10.3 T definition

`T` is the calendar date of the transaction.

Do not use business-day arithmetic.

## 10.4 Unresolved after 30 days

The original concept states that an unresolved case after 30 days can proceed toward an RBI Ombudsman path and Nishchint can prepare a summary.

For the current prototype:

- Do not implement an actual Ombudsman integration.
- Store/configure the 30-day threshold.
- Optionally display "Ombudsman summary available" after 30 days.
- This is lower priority than the core T+1/T+5 flow.

## 10.5 Policy configuration

Use a configuration file or database table.

Example:

```json
{
  "policy_id": "UPI_FAILED_TRANSACTION_V1",
  "version": "1.0",
  "transaction_rules": {
    "PERSON": {
      "deadline_days": 1,
      "compensation_per_day": 100
    },
    "MERCHANT": {
      "deadline_days": 5,
      "compensation_per_day": 100
    }
  },
  "high_value_threshold": 50000,
  "ombudsman_threshold_days": 30
}
```

The threshold of ₹50,000 is a prototype/demo configuration, not a claim about Paytm policy or RBI regulation.

---

# 11. RULE ENGINE INTERFACE

Implement something equivalent to:

```python
class PolicyEngine:
    def evaluate(transaction, current_time) -> PolicyResult:
        ...
```

Expected output:

```json
{
  "policy_id": "UPI_FAILED_TRANSACTION_V1",
  "rule_id": "MERCHANT_T_PLUS_5",
  "applicable": true,
  "deadline": "2026-09-15",
  "days_until_deadline": 5,
  "compensation_per_day": 100,
  "is_breached": false,
  "recommended_action": "FOLLOW_UP"
}
```

After deadline:

```json
{
  "policy_id": "UPI_FAILED_TRANSACTION_V1",
  "rule_id": "MERCHANT_T_PLUS_5",
  "applicable": true,
  "deadline": "2026-09-15",
  "days_overdue": 1,
  "compensation_per_day": 100,
  "compensation": 100,
  "is_breached": true,
  "recommended_action": "RAISE_DISPUTE"
}
```

---

# 12. AGENT DESIGN

Use LangGraph.

The graph should have explicit state and nodes.

## 12.1 Suggested state

```python
class NishchintState(TypedDict, total=False):
    customer_id: str
    case_id: str | None

    user_message: str

    intent: str | None
    extracted_entities: dict

    customer_context: dict
    transaction: dict | None

    policy_result: dict | None

    decision: str | None
    next_action: str | None

    ticket_id: str | None
    followup_id: str | None
    dispute_id: str | None

    escalation_reason: str | None

    assistant_response: str | None

    audit_events: list
```

## 12.2 Graph

```text
START
 ↓
understand_complaint
 ↓
retrieve_context
 ↓
identify_transaction
 ↓
investigate_transaction
 ↓
evaluate_policy
 ↓
decide_next_action
 ↓
execute_action
 ↓
schedule_followup
 ↓
write_audit
 ↓
generate_response
 ↓
END
```

For follow-up:

```text
n8n
 ↓
recheck_transaction
 ↓
evaluate_policy
 ↓
decide
 ├── RESOLVE
 ├── RAISE_DISPUTE
 └── ESCALATE_HUMAN
```

---

# 13. LLM RESPONSIBILITIES

The LLM is responsible for:

- natural-language understanding
- intent extraction
- entity extraction
- identifying missing information
- contextual interpretation
- selecting appropriate tools/workflow
- explaining verified results
- generating customer-friendly responses
- generating human case summaries

The LLM is NOT authoritative for:

- financial amounts
- transaction state
- deadlines
- compensation
- dispute state
- policy facts

All authoritative values must be tool-derived.

---

# 14. TOOLS

Implement tools/services equivalent to:

```text
get_customer_context
find_relevant_transaction
get_transaction
create_ticket
get_case
schedule_followup
raise_dispute
calculate_compensation
escalate_to_human
notify_customer
write_audit_event
```

Each tool should return structured data.

Do not return uncontrolled prose where structured data is expected.

---

# 15. MOCK PAYTM SERVICES

External Paytm services must be represented by internal mock APIs/services.

## Required mock endpoints

```http
GET /api/customers/{customer_id}
GET /api/customers/{customer_id}/context

GET /api/transactions/{transaction_id}
GET /api/customers/{customer_id}/transactions

POST /api/tickets
GET /api/tickets/{ticket_id}

POST /api/disputes
GET /api/disputes/{dispute_id}

POST /api/notifications

POST /api/followups
GET /api/followups/{followup_id}

POST /api/simulate/advance-time
POST /api/simulate/reset
```

These are prototype APIs, not real Paytm APIs.

---

# 16. DATA MODEL

Use relational storage.

## `customers`

```text
id                  PK
name
phone
preferred_language
created_at
updated_at
```

## `transactions`

```text
id                  PK
customer_id         FK
amount
currency
type                PERSON | MERCHANT
merchant_name
status              SUCCESS | FAILED | PENDING | REVERSED
debited             BOOLEAN
merchant_credited   BOOLEAN | NULL
refund_status       PENDING | RECEIVED | NOT_APPLICABLE
transaction_date
created_at
updated_at
```

## `cases`

```text
id                  PK
customer_id         FK
transaction_id      FK NULLABLE
intent
status
priority
deadline
next_action
escalation_reason
created_at
updated_at
closed_at NULLABLE
```

Recommended status values:

```text
NEW
INVESTIGATING
DECIDED
ACTION_TAKEN
FOLLOW_UP_SCHEDULED
WAITING_FOR_RESOLUTION
RECHECKING
RESOLVED
DISPUTE_RAISED
HUMAN_ESCALATED
```

## `messages`

```text
id                  PK
case_id             FK
sender              CUSTOMER | ASSISTANT | HUMAN
message
language
timestamp
```

## `followups`

```text
id                  PK
case_id             FK
scheduled_for
status              SCHEDULED | RUNNING | COMPLETED | FAILED | CANCELLED
workflow_id NULLABLE
attempt_count
last_run_at NULLABLE
created_at
updated_at
```

## `disputes`

```text
id                  PK
case_id             FK
transaction_id      FK
status              RAISED | PROCESSING | RESOLVED | FAILED
reason
compensation_amount
created_at
updated_at
```

## `audit_logs`

```text
id                  PK
case_id             FK NULLABLE
customer_id         FK NULLABLE
event_type
actor               SYSTEM | AGENT | RULE_ENGINE | N8N | HUMAN | CUSTOMER
metadata_json
timestamp
```

## Optional `notifications`

```text
id
customer_id
case_id
channel
message
status
created_at
```

---

# 17. CUSTOMER CONTEXT / MEMORY

Create a `MemoryService`.

Interface:

```python
class MemoryService:

    def get_customer_context(customer_id) -> dict:
        ...

    def get_open_cases(customer_id) -> list:
        ...

    def get_recent_transactions(customer_id) -> list:
        ...

    def get_previous_messages(customer_id, limit=20) -> list:
        ...

    def save_interaction(...) -> None:
        ...
```

For the prototype, relational database retrieval is sufficient to demonstrate persistent memory.

Cognee can be added through an adapter if integration is stable.

The important behavior:

Customer can say:

> "Abhi tak paise nahi aaye."

and Nishchint can connect that statement to the existing open case.

---

# 18. CONTEXT RESOLUTION

When the user does not provide a transaction ID:

1. Retrieve recent transactions for customer.
2. Retrieve open cases.
3. Use conversation context.
4. Identify the most relevant transaction.
5. If exactly one high-confidence candidate exists, continue.
6. If multiple candidates are ambiguous, ask the customer for clarification.
7. Do not hallucinate a transaction.

Example:

```text
Priya has one open failed merchant transaction:
TXN24001
```

Therefore:

> "Abhi tak paise nahi aaye."

can map to TXN24001.

If there are three unresolved transactions, ask:

> "Aap ₹2,400 wale medical-store payment ki baat kar rahi hain, ya kisi aur transaction ki?"

---

# 19. CASE STATE MACHINE

Cases must have an explicit lifecycle.

```text
NEW
 ↓
INVESTIGATING
 ↓
DECIDED
 ↓
ACTION_TAKEN
 ↓
FOLLOW_UP_SCHEDULED
 ↓
WAITING_FOR_RESOLUTION
 ↓
RECHECKING
 ├── RESOLVED
 ├── DISPUTE_RAISED
 └── HUMAN_ESCALATED
```

Every state transition should generate an audit event.

Invalid transitions should be rejected.

---

# 20. n8n DESIGN

n8n is a first-class component.

Do not integrate it merely as a decorative webhook.

## 20.1 Initial workflow

```text
Webhook
 ↓
Validate follow-up payload
 ↓
Load case information
 ↓
Schedule/wait until target time
 ↓
Fetch transaction
 ↓
Determine refund state
 ↓
Call policy/rules endpoint
 ↓
IF resolved
    ↓
Update case → RESOLVED
    ↓
Notify customer
    ↓
Audit
ELSE
    ↓
Raise dispute
    ↓
Calculate compensation
    ↓
Update case → DISPUTE_RAISED
    ↓
Notify customer
    ↓
Audit
```

## 20.2 Why n8n is core

The key product behavior is:

> Work continues after the chat ends.

n8n owns:

- scheduled follow-up
- delayed execution
- transaction recheck
- conditional branching
- dispute workflow
- notification
- case update
- audit trigger

This is the meaningful n8n contribution.

---

# 21. SIMULATED CLOCK

A real five-day wait is unacceptable for a live demo.

Implement a demo clock.

Example:

```text
Current simulated date:
2026-09-10 14:14
```

Controls:

```text
+1 Day
Advance to Deadline
Advance 5 Days
Reset Demo
```

The simulation must update the time used by the application.

Do not modify real system time.

All policy calculations in demo mode must use the simulated application clock.

---

# 22. DEMO CLOCK FLOW

Initial:

```text
10 Sep
₹2,400 failed
Refund pending
Deadline = 15 Sep
Follow-up scheduled
```

Click:

**Advance to Deadline**

System becomes:

```text
15 Sep
```

n8n follow-up executes.

Mock transaction remains:

```text
refund_status = PENDING
```

System:

```text
deadline breached
↓
dispute required
↓
raise dispute
↓
calculate compensation
↓
update case
↓
notify customer
```

This should happen visibly.

---

# 23. MOCK TRANSACTION DATA

At minimum create:

## Transaction 1 — Main demo

```json
{
  "id": "TXN24001",
  "customer_id": "CUST001",
  "amount": 2400,
  "currency": "INR",
  "type": "MERCHANT",
  "merchant_name": "Apollo Medicals",
  "status": "FAILED",
  "debited": true,
  "merchant_credited": false,
  "refund_status": "PENDING",
  "transaction_date": "2026-09-10"
}
```

## Transaction 2 — High-value escalation

```json
{
  "id": "TXN85001",
  "customer_id": "CUST002",
  "amount": 85000,
  "currency": "INR",
  "type": "MERCHANT",
  "merchant_name": "Demo Electronics",
  "status": "FAILED",
  "debited": true,
  "merchant_credited": false,
  "refund_status": "PENDING",
  "transaction_date": "2026-09-10"
}
```

## Transaction 3 — Successful case

```json
{
  "id": "TXN12001",
  "customer_id": "CUST001",
  "amount": 1200,
  "currency": "INR",
  "type": "MERCHANT",
  "merchant_name": "Demo Store",
  "status": "SUCCESS",
  "debited": true,
  "merchant_credited": true,
  "refund_status": "NOT_APPLICABLE",
  "transaction_date": "2026-09-09"
}
```

## Optional Transaction 4 — P2P

```json
{
  "id": "TXN30001",
  "customer_id": "CUST003",
  "amount": 3000,
  "currency": "INR",
  "type": "PERSON",
  "merchant_name": null,
  "status": "FAILED",
  "debited": true,
  "merchant_credited": null,
  "refund_status": "PENDING",
  "transaction_date": "2026-09-10"
}
```

---

# 24. MOCK CUSTOMERS

## Priya

```text
ID: CUST001
Name: Priya Sharma
Language: Hindi
```

## Arjun

```text
ID: CUST002
Name: Arjun Mehta
Language: English
```

## Optional customer

```text
ID: CUST003
Name: Rahul Verma
Language: Hindi
```

---

# 25. PRIMARY CUSTOMER EXPERIENCE

The UI should look like a real support product, not a generic LLM playground.

## Customer screen

```text
NISHCHINT
Your payment support teammate

Hi Priya. How can I help?

[ conversation ]

Message...

[Send] [Voice]
```

When the case is active, show a compact status card:

```text
CASE #TKT-1042

₹2,400 Failed Payment

✓ Transaction checked
✓ Policy evaluated
✓ Ticket created
✓ Follow-up scheduled

Expected refund:
15 Sep

Status:
Waiting for refund
```

---

# 26. CASE TIMELINE SCREEN

Show:

```text
CASE #TKT-1042

₹2,400
FAILED MERCHANT PAYMENT

Customer:
Priya Sharma

Merchant:
Apollo Medicals

Transaction:
TXN24001

--------------------------------

10 Sep · 2:14 PM
Complaint received

10 Sep · 2:14 PM
Customer context retrieved

10 Sep · 2:14 PM
Transaction verified

10 Sep · 2:14 PM
Merchant T+5 rule applied

10 Sep · 2:15 PM
Ticket created

10 Sep · 2:15 PM
Follow-up scheduled

15 Sep · 9:00 AM
Deadline reached

15 Sep · 9:00 AM
Transaction rechecked

15 Sep · 9:01 AM
Dispute raised

15 Sep · 9:01 AM
Compensation calculated
```

This timeline must be populated from actual audit events, not hardcoded frontend text.

---

# 27. HUMAN OPERATIONS DASHBOARD

Dashboard metrics:

```text
Active Cases
Follow-ups
Escalations
Resolved
Disputes
```

Case list:

```text
HIGH PRIORITY

Arjun
₹85,000
Failed Payment

Reason:
High-value transaction

AI Recommendation:
Human review required
```

Case detail:

```text
Customer
Transaction
Conversation
Context
AI decision
Policy result
Actions
Audit trail
```

Actions:

```text
[Approve]
[Override]
```

If human overrides:

1. Save human action.
2. Update case.
3. Create audit event.
4. Show the action in timeline.

---

# 28. HUMAN ESCALATION RULES

Prototype configuration:

```text
If amount >= configured high_value_threshold:
    HUMAN_ESCALATION
```

Default prototype threshold:

```text
₹50,000
```

Also escalate if:

- intent cannot be confidently determined
- multiple transactions are equally likely
- fraud/suspicious language is detected
- tool results are inconsistent
- required information cannot be obtained
- system encounters an action failure that cannot be safely retried

The exact threshold is a demo configuration, not a Paytm/RBI claim.

---

# 29. SECURITY GUARD

If input contains:

- UPI PIN
- OTP
- card PIN
- other authentication secrets

Nishchint must not store the secret.

Response:

> "For your security, please don't share your UPI PIN or OTP. I don't need it to investigate your transaction."

Audit:

```text
event_type = SENSITIVE_CREDENTIAL_DETECTED
```

Metadata must NOT contain the actual credential.

Never log the sensitive value.

---

# 30. NOTIFICATION SERVICE

Create an abstraction:

```python
class NotificationService:
    def notify_customer(customer_id, case_id, message):
        ...
```

Prototype implementation may simply:

- create a notification record
- expose it in the customer UI
- optionally log a mock delivery event

Do not claim real SMS/WhatsApp delivery unless actually integrated.

---

# 31. RESPONSE GENERATION

The customer-facing response should be based only on verified system state.

For the initial Priya case:

> "Maine aapka transaction check kiya. ₹2,400 debit hua hai, lekin merchant ko payment receive nahi hua. Is transaction ke liye expected refund date 15 September hai. Main us date par automatically dobara check karunga. Aapko dobara contact karne ki zarurat nahi hai."

After missed deadline:

> "Refund abhi tak receive nahi hua, isliye maine aapke case ke liye dispute raise kar diya hai. Configured policy ke hisaab se delay compensation ₹100 per day hai. Maine case update kar diya hai."

Do not generate exact dates/amounts from model memory. Inject verified values into the response generation context.

---

# 32. API CONTRACTS

## POST `/chat`

Request:

```json
{
  "customer_id": "CUST001",
  "message": "Mere 2400 kat gaye but payment fail dikha raha hai.",
  "case_id": null
}
```

Response:

```json
{
  "case_id": "CASE-1042",
  "message": "Maine aapka transaction check kiya...",
  "intent": "FAILED_PAYMENT",
  "status": "FOLLOW_UP_SCHEDULED",
  "actions": [
    "TRANSACTION_CHECKED",
    "TICKET_CREATED",
    "FOLLOWUP_SCHEDULED"
  ]
}
```

## GET `/cases/{case_id}`

Returns complete case state:

```json
{
  "id": "CASE-1042",
  "customer": {},
  "transaction": {},
  "status": "FOLLOW_UP_SCHEDULED",
  "policy_result": {},
  "timeline": [],
  "followup": {}
}
```

## GET `/customers/{customer_id}/context`

Returns:

```json
{
  "customer": {},
  "recent_transactions": [],
  "open_cases": [],
  "recent_messages": []
}
```

## GET `/transactions/{transaction_id}`

Returns authoritative transaction state.

## POST `/tickets`

Creates a ticket.

## POST `/disputes`

Creates a mock dispute.

Request:

```json
{
  "case_id": "CASE-1042",
  "transaction_id": "TXN24001",
  "reason": "REFUND_DEADLINE_BREACHED"
}
```

## POST `/followups`

Request:

```json
{
  "case_id": "CASE-1042",
  "scheduled_for": "2026-09-15T09:00:00"
}
```

## POST `/simulate/advance-time`

Request:

```json
{
  "days": 5
}
```

## POST `/simulate/reset`

Reset all demo state.

---

# 33. SERVICE LAYER

Use clear service boundaries.

Suggested:

```text
TransactionService
CaseService
TicketService
DisputeService
PolicyService
MemoryService
FollowupService
NotificationService
AuditService
SimulationClockService
```

The agent should depend on service/tool interfaces, not directly on ORM internals.

---

# 34. PROJECT STRUCTURE

Use a clean monorepo.

```text
nishchint/
│
├── README.md
├── .env.example
├── docker-compose.yml
├── .gitignore
│
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   │
│   │   ├── api/
│   │   │   ├── chat.py
│   │   │   ├── customers.py
│   │   │   ├── transactions.py
│   │   │   ├── cases.py
│   │   │   ├── tickets.py
│   │   │   ├── disputes.py
│   │   │   ├── followups.py
│   │   │   └── simulation.py
│   │   │
│   │   ├── agent/
│   │   │   ├── graph.py
│   │   │   ├── state.py
│   │   │   ├── nodes.py
│   │   │   ├── prompts.py
│   │   │   └── tools.py
│   │   │
│   │   ├── rules/
│   │   │   ├── engine.py
│   │   │   └── policies.json
│   │   │
│   │   ├── services/
│   │   │   ├── transaction_service.py
│   │   │   ├── case_service.py
│   │   │   ├── ticket_service.py
│   │   │   ├── dispute_service.py
│   │   │   ├── memory_service.py
│   │   │   ├── followup_service.py
│   │   │   ├── notification_service.py
│   │   │   ├── audit_service.py
│   │   │   └── simulation_clock.py
│   │   │
│   │   ├── models/
│   │   │   ├── customer.py
│   │   │   ├── transaction.py
│   │   │   ├── case.py
│   │   │   ├── message.py
│   │   │   ├── followup.py
│   │   │   ├── dispute.py
│   │   │   └── audit.py
│   │   │
│   │   ├── schemas/
│   │   │   ├── chat.py
│   │   │   ├── case.py
│   │   │   └── ...
│   │   │
│   │   ├── integrations/
│   │   │   ├── llm/
│   │   │   ├── sarvam/
│   │   │   └── cognee/
│   │   │
│   │   ├── database/
│   │   │   ├── session.py
│   │   │   └── seed.py
│   │   │
│   │   └── config.py
│   │
│   ├── tests/
│   ├── requirements.txt
│   └── Dockerfile
│
├── frontend/
│   ├── app/
│   │   ├── page.tsx
│   │   ├── customer/
│   │   ├── cases/
│   │   └── operations/
│   │
│   ├── components/
│   ├── lib/
│   ├── types/
│   ├── public/
│   ├── package.json
│   └── Dockerfile
│
├── n8n/
│   ├── README.md
│   └── workflows/
│
└── docs/
    ├── architecture.md
    ├── api.md
    └── demo-script.md
```

Claude may simplify file count if necessary, but must preserve these logical boundaries.

---

# 35. FRONTEND ROUTES

Required:

```text
/
```

Customer experience.

```text
/customer
```

Customer support experience.

```text
/cases/[caseId]
```

Case timeline.

```text
/operations
```

Human operations dashboard.

No need for additional screens unless required by core functionality.

---

# 36. UI PRINCIPLES

The UI must communicate:

- this is a support product
- the agent is working
- the case has state
- actions actually occurred
- future work is scheduled
- humans can intervene

Avoid:

- generic ChatGPT clone UI
- giant "AI generated" labels
- unnecessary dashboards
- fake metrics
- decorative AI animations that don't represent real state

The timeline and case state should be data-driven.

---

# 37. INTERACTIVE DEMO

The demo must be controllable from the UI.

Required controls:

```text
Start Demo
Reset Demo
Advance 1 Day
Advance to Deadline
```

Optional:

```text
Simulate Refund Received
Simulate Refund Pending
```

The demo must never require manually editing database records.

---

# 38. DEMO SCENARIO 1 — PRIYA

### Step 1

User:

> "Mere ₹2,400 kat gaye but payment fail dikha raha hai."

### Step 2

Agent:

- identifies failed payment
- retrieves transaction
- checks merchant credit state
- evaluates T+5 policy

### Step 3

System creates:

```text
CASE-1042
```

### Step 4

System calculates:

```text
Transaction date: 10 Sep
Deadline: 15 Sep
```

### Step 5

n8n receives follow-up request.

### Step 6

Customer sees exact deadline.

### Step 7

Click:

**Advance to Deadline**

### Step 8

n8n workflow rechecks transaction.

### Step 9

Refund is still pending.

### Step 10

System:

```text
RAISE DISPUTE
CALCULATE COMPENSATION
UPDATE CASE
NOTIFY CUSTOMER
AUDIT
```

This is the primary demo.

---

# 39. DEMO SCENARIO 2 — CONTEXT

Customer says:

> "Abhi tak paise nahi aaye."

No transaction ID.

System retrieves the open case and understands the reference.

The response must reference the correct case/transaction.

If ambiguity exists, ask for clarification instead of guessing.

---

# 40. DEMO SCENARIO 3 — HIGH VALUE

Customer:

> "Mere ₹85,000 kat gaye aur payment fail ho gaya."

System:

```text
Transaction identified
 ↓
High-value threshold detected
 ↓
Human escalation
```

Operations dashboard displays:

```text
HIGH PRIORITY
₹85,000
Human review required
```

Human chooses:

```text
Approve
OR
Override
```

The decision is persisted and audited.

---

# 41. DEMO SCENARIO 4 — SECURITY

Customer:

> "My UPI PIN is 1234."

System:

- warns customer
- does not store PIN
- does not send PIN to LLM if preventable
- does not include PIN in logs
- writes only a security event

---

# 42. TESTING REQUIREMENTS

At minimum create tests for:

## Policy

```text
merchant → T+5
person → T+1
deadline calculation
calendar-day calculation
compensation
deadline breach
```

## Context

```text
one open case → automatic association
multiple open cases → clarification
```

## Safety

```text
OTP detected → warning
PIN detected → warning
secret absent from audit logs
```

## Escalation

```text
amount >= threshold → human escalation
```

## Case state

```text
valid transitions succeed
invalid transitions fail
```

## Follow-up

```text
scheduled follow-up exists
recheck executes
pending refund → dispute
received refund → resolve
```

## API

At least basic integration tests for:

```text
/chat
/cases
/transactions
/disputes
/followups
/simulation
```

---

# 43. FAILURE HANDLING

The prototype must not crash or falsely claim success.

If transaction API fails:

> "I'm unable to verify the transaction right now. I've kept the case open and routed it for review."

If dispute API fails:

```text
case remains open
action marked failed
retry/error state recorded
human escalation if required
```

If LLM fails:

- do not invent an answer
- return a safe fallback
- preserve case state

If n8n callback fails:

- follow-up remains pending/failed
- audit event created
- case remains recoverable

---

# 44. IDEMPOTENCY

Critical actions must be safe against duplicate execution.

Especially:

```text
raise_dispute
create_ticket
schedule_followup
notify_customer
```

Example:

If n8n executes the same follow-up twice, it must not create two disputes.

Use:

```text
case_id + action_type
```

or another deterministic idempotency key.

---

# 45. OBSERVABILITY

For prototype:

Log:

```text
request ID
case ID
customer ID
transaction ID
agent node
tool call
rule ID
decision
action
n8n workflow ID
error
```

Never log:

- UPI PIN
- OTP
- authentication secrets

---

# 46. ENVIRONMENT VARIABLES

Create `.env.example`.

Suggested:

```text
LLM_API_KEY=
LLM_MODEL=

SARVAM_API_KEY=

COGNEE_API_KEY=

DATABASE_URL=

N8N_BASE_URL=
N8N_WEBHOOK_URL=
N8N_API_KEY=

APP_ENV=development
SIMULATION_MODE=true
HIGH_VALUE_THRESHOLD=50000
```

Secrets must never be committed.

---

# 47. LLM ABSTRACTION

Do not couple application logic to one provider.

Create:

```python
class LLMService:
    def understand_complaint(...)
    def generate_response(...)
    def summarize_case(...)
```

The implementation may use the configured provider.

Use structured output wherever possible.

---

# 48. VOICE ABSTRACTION

If implementing Sarvam:

```python
class VoiceService:
    def transcribe(audio) -> str:
        ...

    def synthesize(text, language) -> bytes:
        ...
```

Supported target languages:

- Hindi
- Hinglish where supported through the speech pipeline
- English
- Kannada if time permits

Voice is P1, not P0.

---

# 49. COGNEE ABSTRACTION

If using Cognee:

```python
class KnowledgeMemoryService:
    def add_customer_event(...)
    def retrieve_customer_context(...)
    def retrieve_related_cases(...)
```

Do not make core case resolution fail because Cognee is unavailable.

Database-backed context must remain available.

---

# 50. n8n WEBHOOK CONTRACT

The backend should send something similar to:

```json
{
  "case_id": "CASE-1042",
  "customer_id": "CUST001",
  "transaction_id": "TXN24001",
  "scheduled_for": "2026-09-15T09:00:00",
  "action": "RECHECK_FAILED_PAYMENT"
}
```

n8n should return an execution acknowledgment.

The actual asynchronous execution should update the backend through an API call.

---

# 51. n8n CALLBACK CONTRACT

n8n can call:

```http
POST /api/workflows/followup-result
```

Payload:

```json
{
  "case_id": "CASE-1042",
  "workflow_id": "n8n-followup-001",
  "result": "DISPUTE_RAISED",
  "transaction_status": "FAILED",
  "refund_status": "PENDING",
  "dispute_id": "DSP-8821"
}
```

Backend validates the case and updates state.

---

# 52. IMPORTANT n8n DESIGN DECISION

Do not put the entire AI brain inside n8n.

n8n is the workflow/orchestration layer.

LangGraph is the agent reasoning/orchestration layer.

Rules engine is the deterministic policy layer.

Backend is the system-of-record/API layer.

This separation must remain clear.

---

# 53. RESPONSIBILITY MATRIX

| Component | Responsibility |
|---|---|
| Next.js | User experience |
| FastAPI | API + application boundary |
| LangGraph | Agent workflow |
| LLM | Language understanding + communication |
| Rules engine | Policy/financial calculations |
| Database | Persistent state |
| MemoryService | Customer/context retrieval |
| Mock Paytm services | External system simulation |
| n8n | Delayed/asynchronous automation |
| Operations dashboard | Human intervention |
| Audit service | Traceability |

---

# 54. TWO-DEVELOPER PARALLEL PLAN

## Developer A — Brain / Backend

Own:

- FastAPI
- database
- models
- mock APIs
- LangGraph
- tools
- rules engine
- memory
- audit
- case state
- LLM integration

Primary deliverable:

```text
POST /chat
```

must execute the actual core loop.

## Developer B — Experience / Automation

Own:

- Next.js
- customer UI
- case timeline
- operations dashboard
- n8n
- simulation controls
- notifications
- UI state

Both must agree on API contracts before parallel work.

---

# 55. BUILD TIMELINE — 6 HOURS

## 0:00–0:20

Both:

- freeze architecture
- freeze API contract
- create repo
- create branches
- seed data
- create `.env.example`

## 0:20–1:30

Developer A:

- FastAPI
- database
- models
- mock APIs
- rules engine

Developer B:

- Next.js
- customer screen
- case screen
- operations screen

## 1:30–2:30

Developer A:

- LangGraph
- tools
- context
- chat endpoint

Developer B:

- connect frontend to backend
- display case state
- display timeline

## 2:30–3:30

Core integration.

Required working path:

```text
complaint
→ transaction lookup
→ policy
→ decision
→ ticket
→ follow-up
```

## 3:30–4:30

n8n:

```text
follow-up
→ wait/simulation
→ recheck
→ dispute/resolve
→ notification
→ audit
```

## 4:30–5:15

- human escalation
- override
- security guard
- audit UI
- idempotency

## 5:15–6:00

Only after core functionality works:

- UI polish
- voice
- Sarvam
- error states
- demo reset
- demo rehearsal

---

# 56. PRIORITY SYSTEM

## P0 — MUST WORK

- customer chat
- context understanding
- transaction lookup
- rules engine
- decision
- ticket creation
- follow-up
- n8n workflow
- simulated deadline
- transaction recheck
- dispute
- compensation
- case state
- audit
- human escalation

## P1

- memory improvements
- Sarvam voice
- Kannada
- richer operations UI
- Cognee adapter
- notification polish

## P2

- merchant incident clustering
- advanced analytics
- travel
- multi-service Paytm workflows

---

# 57. DEFINITION OF DONE

The prototype is complete only when this can happen without manually editing database records:

```text
[ ] User sends complaint
[ ] AI understands complaint
[ ] System retrieves customer context
[ ] System identifies transaction
[ ] System retrieves authoritative transaction state
[ ] Rules engine evaluates policy
[ ] Agent determines next workflow
[ ] Ticket is created
[ ] Follow-up is sent to n8n
[ ] Customer receives exact deadline
[ ] Demo clock advances
[ ] n8n executes follow-up
[ ] Transaction is rechecked
[ ] System detects unresolved refund
[ ] Dispute is created
[ ] Compensation is calculated
[ ] Case is updated
[ ] Customer is notified
[ ] Audit trail is complete
[ ] High-value case escalates
[ ] Human can override
[ ] Contextual follow-up works
[ ] PIN/OTP guard works
[ ] Reset Demo restores initial state
```

---

# 58. DEMO NARRATIVE

The demo should be approximately 4 minutes.

## Part 1 — Complaint

Say:

> "Mere ₹2,400 kat gaye but payment fail dikha raha hai."

Show the agent:

```text
Understand
→ Context
→ Transaction
→ Merchant
→ T+5
→ Deadline
```

## Part 2 — Autonomous continuation

Show:

```text
Follow-up scheduled
```

Click:

```text
Advance to Deadline
```

Show n8n execution.

Then:

```text
Refund still pending
→ dispute raised
→ compensation calculated
→ customer notified
```

## Part 3 — Context

Say:

> "Abhi tak paise nahi aaye."

Nishchint remembers the case.

## Part 4 — Human

Use ₹85,000 case.

Show:

```text
High value
→ Human escalation
→ Dashboard
→ Human decision
→ Audit
```

---

# 59. WHAT MAKES THIS DIFFERENT FROM A CHATBOT

The system must visibly demonstrate:

```text
Chatbot:
Answer → End

Nishchint:
Answer
→ Case ownership
→ Action
→ Future commitment
→ Scheduled execution
→ Re-investigation
→ Action again
→ Resolution
```

The judge should be able to see that the AI continues working without the customer remaining in the conversation.

---

# 60. PRODUCTION-GRADE PROTOTYPE PRINCIPLES

Even though external systems are mocked:

## Use real interfaces

```text
MockTransactionService
```

rather than:

```text
if demo:
    pretend transaction exists
```

## Use real state

Case state must be stored.

## Use real actions

Dispute creation must create a database record.

## Use real workflows

n8n must actually execute.

## Use real audit

Timeline must be generated from events.

## Use deterministic rules

Financial calculations must not be generated by the LLM.

## Use recoverable failures

Failures must result in explicit states.

## Use idempotency

Repeated workflow execution must not duplicate irreversible actions.

---

# 61. DO NOT FAKE THE FOLLOW-UP

The following is NOT acceptable:

```text
Frontend waits 3 seconds
↓
Frontend displays "Dispute Raised"
```

The correct architecture is:

```text
Backend
↓
n8n
↓
scheduled workflow
↓
backend transaction check
↓
policy evaluation
↓
dispute API
↓
database
↓
frontend reflects new state
```

The UI must reflect backend state.

---

# 62. DO NOT FAKE AI DECISION MAKING

The following is NOT acceptable:

```python
if "2400" in message:
    deadline = 5
```

Instead:

```text
LLM
→ intent/entity extraction
→ transaction service
→ transaction type
→ rules engine
→ policy result
→ action decision
```

The agent should select the workflow based on structured context.

---

# 63. DO NOT FAKE MEMORY

The following is NOT acceptable:

```python
if message == "abhi tak paise nahi aaye":
    return predefined_response
```

Instead retrieve actual case/customer state.

---

# 64. DO NOT FAKE AUDIT

Do not hardcode timeline entries.

Timeline must be generated from `audit_logs`.

---

# 65. DO NOT FAKE HUMAN-IN-THE-LOOP

The human dashboard must modify actual case state.

Human override must create an audit event.

---

# 66. DO NOT FAKE n8n

n8n must actually receive the follow-up payload and execute the workflow.

The system should be able to demonstrate an n8n execution.

---

# 67. FUTURE PRODUCTION PATH

The prototype architecture should allow:

```text
Mock Paytm API
       ↓
Real Paytm API adapter
```

and:

```text
Prototype policy JSON
       ↓
Verified policy service
```

and:

```text
Database memory
       ↓
Cognee / production memory layer
```

and:

```text
Mock notification
       ↓
Real notification channels
```

The agent/tool interfaces should remain stable.

---

# 68. FUTURE PRODUCT EXPANSION

The original concept proposes extending the same loop to:

- Travel
- Loans
- Insurance
- Shopping
- Bills

These are not part of the current MVP.

The abstraction should nevertheless support:

```text
ResolveIssueWorkflow
```

rather than hardcoding every agent node specifically to one merchant-payment case.

Future:

```text
PaymentSupportWorkflow
TravelSupportWorkflow
LoanSupportWorkflow
InsuranceSupportWorkflow
```

But implement only payment support now.

---

# 69. DOCUMENTATION REQUIRED

Claude Code must create:

```text
README.md
docs/architecture.md
docs/api.md
docs/demo-script.md
```

README must explain:

- what Nishchint is
- architecture
- setup
- environment variables
- backend startup
- frontend startup
- n8n setup
- seed data
- demo flow
- limitations

---

# 70. SETUP REQUIREMENT

The final prototype should be startable with minimal commands.

Prefer:

```bash
docker compose up
```

if practical.

Otherwise document:

```bash
cd backend
pip install -r requirements.txt
uvicorn app.main:app --reload
```

and:

```bash
cd frontend
npm install
npm run dev
```

n8n cloud configuration must be documented separately.

Do not let Docker complexity consume the core build time.

---

# 71. CODE QUALITY REQUIREMENTS

Claude Code must:

- use type hints
- use Pydantic models
- validate API inputs
- avoid giant files
- separate business logic from routes
- separate agent logic from services
- use structured logging
- handle errors explicitly
- keep secrets out of source code
- write meaningful comments only where needed
- avoid unnecessary abstractions

Do not over-engineer.

---

# 72. CLAUDE CODE EXECUTION INSTRUCTIONS

Claude Code should work incrementally.

## Phase 1

Inspect repository.

If an existing project exists:

- do not delete working code blindly
- understand current architecture
- preserve useful existing components
- identify conflicts

## Phase 2

Create/finalize:

- backend
- database
- models
- mock APIs
- rules engine

## Phase 3

Implement LangGraph agent.

## Phase 4

Implement frontend.

## Phase 5

Integrate n8n.

## Phase 6

Add safety/human/audit.

## Phase 7

Test complete demo.

After each major phase:

- run tests
- start application
- verify endpoints
- fix errors before proceeding

---

# 73. CLAUDE MUST NOT MAKE THESE ASSUMPTIONS

Do not assume:

- a real Paytm API exists
- a real dispute can be raised
- a real refund can be initiated
- a customer will provide transaction ID
- LLM output is always correct
- n8n execution always succeeds
- only one transaction exists
- every case is safe for autonomy
- external APIs are always available
- a policy never changes
- memory retrieval is always unambiguous

Build explicit handling for these conditions where relevant.

---

# 74. AMBIGUITY POLICY

If information is missing:

```text
If safely inferable from context:
    retrieve context and continue

If not safely inferable:
    ask customer

If high-risk:
    escalate

Never hallucinate.
```

---

# 75. SYSTEM OF RECORD

For prototype:

**Database is the source of truth for application state.**

The LLM is not the source of truth.

n8n is not the source of truth.

Frontend is not the source of truth.

Audit logs record what happened but do not replace the primary case state.

---

# 76. AGENT AUTONOMY BOUNDARY

The agent may autonomously:

- investigate
- retrieve context
- calculate through rules engine
- create normal support cases
- schedule follow-ups
- recheck status
- execute configured low-risk actions
- notify customers

The agent must escalate:

- high-value configured cases
- fraud/suspicious cases
- ambiguous cases
- unsafe cases
- failed critical actions

---

# 77. CORE DESIGN MANTRA

Keep this visible during implementation:

> **LLM understands.**
>
> **Rules decide policy.**
>
> **Tools act.**
>
> **Database remembers.**
>
> **n8n continues the work.**
>
> **Humans handle risk.**
>
> **Audit records everything important.**

---

# 78. FINAL ACCEPTANCE TEST

Run this exact test before considering the prototype complete.

### Reset

```text
POST /simulate/reset
```

### Customer

```text
CUST001
```

### Message

```text
"Mere ₹2,400 kat gaye but payment fail dikha raha hai."
```

Expected:

```text
intent = FAILED_PAYMENT
transaction = TXN24001
type = MERCHANT
rule = MERCHANT_T_PLUS_5
deadline = 2026-09-15
case created
ticket created
follow-up scheduled
```

### Customer follow-up

```text
"Abhi tak paise nahi aaye."
```

Expected:

```text
same open case identified
no unnecessary transaction-ID question
```

### Advance time

```text
+5 days
```

Expected:

```text
n8n follow-up runs
transaction rechecked
refund still pending
policy breach detected
dispute created
compensation calculated
case = DISPUTE_RAISED
customer notification created
audit events recorded
```

### High-value case

```text
CUST002
₹85,000 failed payment
```

Expected:

```text
HUMAN_ESCALATED
```

### Security

```text
"My OTP is 1234"
```

Expected:

```text
security warning
secret not stored
secret not logged
audit security event
```

### Reset

```text
POST /simulate/reset
```

Expected:

```text
demo returns to clean initial state
```

---

# 79. FINAL PRODUCT DEFINITION

The final prototype should communicate one simple idea:

> **Nishchint transforms customer support from a conversation into an autonomous resolution loop.**

The customer says:

> "My money was deducted and the payment failed."

Nishchint:

```text
understands
→ remembers
→ investigates
→ applies policy
→ decides
→ acts
→ schedules
→ waits
→ rechecks
→ acts again
→ resolves or escalates
```

That is the product.

The prototype does not need to implement all of Paytm.

It needs to make this one loop real.

---

# 80. BUILD ORDER — ABSOLUTE PRIORITY

If time becomes limited, implement in this exact order:

```text
1. Database
2. Mock transaction API
3. Policy/rules engine
4. Case state machine
5. LangGraph agent
6. Tool layer
7. POST /chat
8. Customer UI
9. Follow-up persistence
10. n8n workflow
11. Simulated clock
12. Automatic recheck
13. Dispute action
14. Audit trail
15. Human escalation
16. Human override
17. Security guard
18. Voice/Sarvam
19. Cognee
20. UI polish
```

Never move items 18–20 ahead of items 1–17.

---

# 81. SUCCESS CRITERIA

The prototype succeeds if a judge can interact with it and observe:

```text
"This AI understood my problem."
        ↓
"It found the relevant transaction."
        ↓
"It used actual context."
        ↓
"It applied a deterministic rule."
        ↓
"It made a workflow decision."
        ↓
"It actually performed an action."
        ↓
"It scheduled work for later."
        ↓
"It continued working after the conversation."
        ↓
"It detected that the issue was still unresolved."
        ↓
"It took the next action automatically."
        ↓
"It knew when to involve a human."
```

If those behaviors are real, Nishchint is a valid Track 3 autonomous AI teammate prototype.

---

# 82. FINAL INSTRUCTION TO CLAUDE CODE

Build the system described in this document.

Do not reduce it to a chatbot.

Do not replace real workflows with frontend simulations.

Do not hardcode the final timeline into the UI.

Do not hardcode the decision based on keywords.

Do not allow the LLM to invent authoritative financial facts.

Do not treat n8n as a decorative integration.

Do not implement unnecessary product scope before the core autonomous loop works.

When a technical choice is ambiguous, prefer:

1. simplest production-sensible implementation,
2. explicit interfaces,
3. deterministic behavior for financial/policy logic,
4. persistent state,
5. observable actions,
6. recoverable failures,
7. minimal dependencies,
8. fast local development.

The target is:

> **A working, production-grade vertical slice of an autonomous AI support teammate — not a fake demo and not the entire Paytm platform.**
