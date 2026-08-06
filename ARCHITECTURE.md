# TackPath Lotus OS — Architecture

*A side project exploring what it takes to turn Lotus from a managed dispatch tier into a full courier service operating system. Nothing here touches tackpath-app or tackpath-driver.*

---

## What This Is

Lotus is TackPath's fully-managed tier — TackPath's own team runs dispatch end-to-end for a courier company. This project explores the infrastructure stack that makes that genuinely scalable: one TackPath operator managing multiple courier companies simultaneously, with AI agent pods doing the continuous monitoring and humans retaining approval authority.

The thesis: **the shift from alerting to acting, with humans in the loop.**

---

## The Stack

| Layer | Tool | Status | Purpose |
|---|---|---|---|
| Coordination surface | Buzz (Block/Dorsey) | Planned — pre-1.0, watch for stable release | Shared workspace where humans and AI agents work in the same channels |
| Workflow automation | n8n | Planned — production-ready, self-hostable | Orchestrates sequences between pods; visual, auditable, changeable without rewriting code |
| Data layer | Supabase MCP | Ready — official server exists today | Gives agent pods direct natural-language access to TackPath's real Postgres database |
| Communication | Twilio | In progress — being worked on separately | Real SMS/voice notifications to drivers and clients on actual phones |
| Agent model | Claude (Anthropic API) | Live — already used in TackPath today | Powers the reasoning inside each pod |
| Protocol | MCP (Model Context Protocol) | Ready — universal standard, 1000+ servers | Universal connector between pods and external services; no custom glue code per integration |

---

## What Each Layer Actually Does

### Supabase MCP (the data layer — ready now)
The existing TackPath Supabase database (`hofijsiphyjpdvujjzfi.supabase.co`) already holds all real operational data: jobs, drivers, driver_locations, messages, organizations. The official Supabase MCP server (`@supabase/mcp-server`) exposes this to AI agents directly — no custom fetch wrappers, no manual JSON parsing. An agent can ask "which jobs have been in_transit for more than 30 minutes with no GPS update?" and get a real answer from real data. Read-only access scoped to the TackPath project for monitoring agents; write access only for specific agents with human approval gates.

### n8n (the workflow engine — production-ready)
Orchestrates the sequence of actions between pods. When the Exception Intelligence Agent detects a Driver Dark situation, n8n coordinates what happens next: check driver GPS → fetch remaining stops → identify available drivers → draft client messages → post to Buzz channel → wait for human approval → execute approved actions. Visual, auditable, and changeable without touching any code in tackpath-app.

### Buzz (the coordination surface — planned)
The shared workspace where a TackPath Lotus operator and the AI agent pods all work together. Agents post updates, flag exceptions, surface recovery plans, and draft messages as first-class channel members. The human operator reviews and approves. Full context is always shared and logged. Currently pre-1.0 — architecture is right, timing is early.

### Twilio (the communication layer — in progress)
Real SMS/voice notifications that reach drivers and clients on their actual phones, not just messages sitting in a database. This is the layer that makes driver check-ins and client delay notifications genuinely real rather than theoretical.

### MCP (the connector standard)
Rather than building custom integrations between each pod and each external service, TackPath's pods expose their capabilities as MCP servers. Any surface — Buzz, n8n, a future Mission Control — can call them without custom glue code. Already has community servers for Supabase, Stripe, Slack, and 1000+ other tools.

---

## The Pods (from the Architecture Document)

Each pod runs as a narrow, named agent with a specific responsibility. None acts without human approval when stakes are high.

| Pod | Primary Agent | Autonomous Actions | Requires Human Approval |
|---|---|---|---|
| Dispatch | Route Sequencing + Assignment | Build and propose routes | Final driver assignment |
| Fleet Operations | Location Tracking | Flag Driver Dark, send check-in | Any reassignment |
| Customer Experience | Status Update | Draft client notifications | Send any message |
| Finance & Billing | Invoice Generation + OWL | Generate draft invoices, flag margin leakage | Send any invoice |
| Analytics & Intelligence | Exception Intelligence | Detect Late/Unassigned/Dark, generate recovery plan | Execute any recovery action |
| AI Executive | Cross-Pod Health + Executive Query | Daily digest, answer business questions | Any cross-pod action |
| Capacity Planning | Staffing Alerts + Revenue Forecasting | Flag demand/capacity gaps | Any staffing recommendation |

---

## What Gets Built Here (in order)

**Phase 1 — Supabase MCP proof of concept**
Connect Claude directly to the TackPath Supabase database via the official MCP server. Verify that an agent can query real job data, real driver locations, and real exception history using natural language. This is the data foundation everything else depends on. Uses read-only access, scoped to the TackPath project.

**Phase 2 — Exception Intelligence Agent (real data)**
Rebuild the exception detection (Late, Unassigned, Driver Dark) as a real agent using the Supabase MCP connection from Phase 1. Compare against the current dispatcher.html implementation — same logic, but the agent reasons over real data rather than JavaScript reading from a pre-fetched array.

**Phase 3 — n8n workflow for recovery sequence**
Wire the exception detection into an n8n workflow: detect exception → query available drivers → draft recovery messages → surface for human approval. This is the workflow engine that connects detection to action.

**Phase 4 — Twilio integration**
Wire Twilio into the n8n workflow so approved messages actually reach drivers and clients on their phones. This is the step that makes everything above genuinely real rather than theoretical.

**Phase 5 — Buzz integration (when stable)**
Wire the n8n workflow into a Buzz channel so the full recovery sequence — detection, plan, approval, execution — happens in a single shared workspace with full context and audit trail. Timing depends on Buzz reaching a stable 1.0.

---

## What This Is Not

This project deliberately excludes:
- Marketplace features (autonomous bidding on contracts)
- Warehouse or robotics coordination
- Actual HR, recruiting, or hiring functions
- Any code that touches tackpath-app or tackpath-driver

---

## Repo Isolation Guarantee

This repo (`bbrian34/tackpath-lotus`) is verified independent — not a fork, no parent, no shared deployment path with tackpath-app or tackpath-driver. Nothing built here can affect the live TackPath product at tackpath.com.
