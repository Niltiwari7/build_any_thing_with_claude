# From Idea to Engineering Blueprint: The Pre-Code AI Playbook

> **The Golden Rule:** Never start a project by telling an AI coding agent *"Build this app."* If you skip planning, the agent silently makes architectural assumptions that lead to broken integrations and expensive rework. Use AI to **think, challenge, and design first**—and write code only after your blueprint is locked.

---

## Quick Overview: The 7-Phase Pipeline

| Phase | Core Deliverable | Key Question Answered |
| --- | --- | --- |
| **1. Product Thinking** | Problem & Risk Analysis | Is this a real problem, and who is it for? |
| **2. PRD** | Product Requirements Document | *What* should the system do (and *not* do)? |
| **3. MVP Prioritization** | MoSCoW Feature Matrix | What is the smallest end-to-end demo we can ship? |
| **4. High-Level Design (HLD)** | System Architecture & Flow | How do our major components talk to each other? |
| **5. Database Design** | Schema & ER Diagram | How is data structured, versioned, and isolated? |
| **6. API Contracts** | REST / OpenAPI Spec | How do frontend and backend integrate without clashes? |
| **7. Architecture Review** | Risk & Trade-off Audit | Where is the system overengineered or fragile? |

---

## Phase 1: Product Thinking & Idea Validation

### Key Notes

* **Understand the problem before the solution:** Never jump into tech stacks or UI screens yet. Define four fundamentals first:
1. **User:** Who specifically experiences this pain?
2. **Problem:** What bottleneck or friction do they face today?
3. **Outcome:** What concrete result do they want?
4. **Differentiation:** Why is this better than existing workarounds?


* **Use Claude as a critic, not a cheerleader:** A good AI lead pushes back on weak assumptions and separates **verified facts** from **untested hypotheses**.

### Customizable Prompt Template

```text
Act as a Senior Product Manager.

I want to build [PROJECT_NAME].

Problem statement:
[DESCRIBE_THE_USER_PROBLEM_IN_2_TO_3_SENTENCES]

Proposed solution:
[DESCRIBE_YOUR_SOLUTION_AND_HOW_IT_HELPS]

Analyze:
1. Target users (primary and secondary).
2. Core user pain points.
3. Existing alternatives and why they fall short.
4. Core user journeys (start to finish).
5. Main product value proposition.
6. Key assumptions that need validation.
7. Potential product and adoption risks.

Constraints:
- Do not generate any code.
- Challenge my idea rather than blindly agreeing with it.
- Clearly distinguish verified facts from hypotheses.

```

---

## Phase 2: Product Requirements Document (PRD)

### Key Notes

* **What vs. How:** A PRD defines *what* the software must do and *why*—never *how* to implement it in code.
* **Functional vs. Non-Functional Requirements (Crucial for real engineering & SDE interviews):**
* **Functional Requirements (FR):** Actions the system performs (e.g., *"User can submit an idea"*, *"System generates and saves a blueprint"*).
* **Non-Functional Requirements (NFR):** Quality, security, and reliability constraints (e.g., *"User data must be strictly isolated"*, *"External AI API timeouts must fail gracefully"*).


* **Hackathon Rule for Numbers:** Never invent fake latency or scale metrics. If you set provisional targets (e.g., `< 10s response time`), explicitly label them as **assumptions**.

### Customizable Prompt Template

```text
Act as a Principal Product Manager.

Create a detailed PRD for [PROJECT_NAME] based on our validated problem analysis.

Include:
1. Product overview.
2. Problem statement.
3. Target users and user personas.
4. Functional requirements (specific and testable).
5. Non-functional requirements (security, reliability, performance, usability).
6. End-to-end user journeys.
7. Success metrics.
8. Out-of-scope features (what we will deliberately NOT build).
9. Assumptions and risks.

Constraints:
- Do not make technical implementation or stack decisions yet.
- Clearly distinguish MVP requirements from future enhancements.
- Label any provisional performance targets as assumptions.

```

---

## Phase 3: MVP Prioritization (MoSCoW Method)

### Key Notes

* In a 24-hour hackathon, your goal is **one working end-to-end user journey**, not a feature-bloated half-working app.
* **MoSCoW Framework:**
* **Must Have:** Without this, the core demo fails completely.
* **Should Have:** High value, but the core flow still works without it.
* **Could Have:** Nice polish if extra hours remain (e.g., Markdown/PDF export).
* **Won't Have:** Explicitly cut from this release (e.g., billing, team roles, OAuth social logins).



### Customizable Prompt Template

```text
Act as a Hackathon Product Strategist.

Review the PRD for [PROJECT_NAME] and prioritize all features using the MoSCoW framework (Must have, Should have, Could have, Won't have).

For each feature, evaluate:
1. User value.
2. Implementation complexity.
3. Technical dependencies.
4. Demo impact (how well it showcases core value).
5. Priority justification.

Constraints:
- Timeline is [TIMELINE, e.g., 24 hours / 1 weekend].
- Optimize for a working end-to-end user journey.
- Aggressively cut non-essential features.

Conclude by defining the smallest possible MVP scope that proves the core value.

```

---

## Phase 4: High-Level Design (HLD)

### Key Notes

* **HLD = Macro Architecture:** Maps out the major building blocks, their responsibilities, and how data flows between them.
* **Strict Separation of Responsibilities:**
* **Frontend:** UI rendering and user input only. **Never** store LLM API keys or call external AI providers directly from the client.
* **Backend API:** Request validation, authentication middleware, business logic, database persistence, and external AI calls.


* **Default to a Modular Monolith:** For MVPs, avoid microservices, message brokers, and complex distributed setups unless hard requirements demand them.

### Customizable Prompt Template

```text
Act as a Principal Software Architect.

Based on the approved PRD and MVP scope for [PROJECT_NAME], design a High-Level Architecture.

Preferred technology stack:
- Frontend: [FRONTEND_STACK, e.g., Vue 3 + Vuetify / React + Tailwind]
- Backend: [BACKEND_STACK, e.g., Node.js + Express / Python + FastAPI]
- Database: [DATABASE, e.g., PostgreSQL / MongoDB]
- External Services: [EXTERNAL_APIS, e.g., OpenAI / Claude API, Stripe]

Include:
1. Major system components and clear responsibility for each.
2. Communication protocols between components.
3. Step-by-step request and response flow for the core feature.
4. Authentication and authorization flow.
5. External API / AI integration flow.
6. Error-handling and fallback strategy (especially if external APIs fail).
7. Security considerations.
8. Simple deployment architecture.
9. A Mermaid architecture diagram.

Constraints:
- Prefer a modular monolith over microservices.
- Explain key architectural trade-offs.
- Do not write application code.

```

---

## Phase 5: Database Design & ERD

### Key Notes

* **Design for History & State:**
* Separate parent records (e.g., `Projects`) from execution records (e.g., `Generations`) so new runs don't silently overwrite previous versions.
* Split distinct outputs (e.g., `GeneratedDocuments` by `document_type`) so individual sections can be fetched or regenerated independently.
* Track lifecycle states (`PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`) to handle external AI timeouts cleanly.


* **Security at the Query Level:** Never look up a resource by ID alone. Always scope queries to the authenticated server-side user ID:
```sql
-- WRONG (Insecure Direct Object Reference vulnerability)
SELECT * FROM projects WHERE id = $1;

-- RIGHT (Enforces ownership from verified auth token)
SELECT * FROM projects WHERE id = $1 AND user_id = $2;

```



### Customizable Prompt Template

```text
Act as a Senior Database Architect.

Design the [DATABASE, e.g., PostgreSQL] schema for [PROJECT_NAME].

Core data requirements:
- [ENTITY_RELATIONSHIP_1, e.g., Users can own multiple projects]
- [ENTITY_RELATIONSHIP_2, e.g., Each project can have multiple AI generations]
- [ENTITY_RELATIONSHIP_3, e.g., History and status of every generation must be preserved]

Provide:
1. Entities, attributes, and data types.
2. Primary keys, foreign keys, and relationship cardinalities.
3. Unique constraints and NOT NULL rules.
4. Index recommendations based on frequent queries.
5. Referential integrity rules (e.g., ON DELETE CASCADE vs RESTRICT).
6. Critical query patterns (including user data isolation).
7. A Mermaid ER diagram.
8. Recommended migration sequence.

Constraints:
- Explain important schema trade-offs.
- Highlight data-isolation and security rules.
- Do not generate migration files or application code yet.

```

### Mandatory Database Review Checklist

Before approving Claude's schema, verify:

* [ ] Every table has an explicit primary key.
* [ ] Foreign keys match parent types and prevent orphaned records.
* [ ] Frequently filtered columns (`user_id`, `project_id`, `status`) are indexed.
* [ ] Sensitive fields (`password_hash`, tokens) are excluded from standard read queries.
* [ ] Every user-owned table enforces row-level ownership via `user_id`.

---

## Phase 6: API Contract Design

### Key Notes

* **Why Contracts Come Before Code:** Locking JSON request/response shapes allows frontend and backend development (or parallel AI agents) to work simultaneously without integration mismatches.
* **Every endpoint spec must define 8 items:** HTTP Method & Path, Auth requirement, Request Schema, Success Response (`200`/`201`), Error Responses (`400`, `401`, `403`, `404`, `502`), Validation rules, and Authorization rules.

### Customizable Prompt Template

```text
Act as a Senior API Architect.

Design REST API contracts for [PROJECT_NAME] to support all MVP user journeys.

Required endpoints/operations:
- [OPERATION_1, e.g., Auth: Register & Login]
- [OPERATION_2, e.g., Create & list projects]
- [OPERATION_3, e.g., Trigger AI blueprint generation]
- [OPERATION_4, e.g., Fetch generation history and individual documents]

For each endpoint, specify:
1. HTTP method and URL path.
2. Authentication requirement (Public vs Bearer/Session).
3. Request parameters and JSON body schema (with concrete examples).
4. Success response schema and HTTP status code.
5. Error response format and status codes (400, 401, 403, 404, 500/502).
6. Input validation rules.
7. Ownership/authorization checks.

Constraints:
- Use consistent naming conventions (e.g., camelCase JSON keys, plural resource nouns).
- Ensure contracts align 1:1 with the database schema and PRD user journeys.
- Do not generate backend or frontend implementation code.

```

---

## Phase 7: Tech Stack Audit & Risk Review

### Key Notes

* **Match tools to requirements, not hype:** Skip Redis, message queues, Docker Swarm, or GraphQL if a synchronous REST API + PostgreSQL handles your MVP cleanly.
* **Audit for failure modes:** External LLM APIs are slow and occasionally fail. Ensure your architecture handles rate limits, malformed JSON responses from the LLM, and timeouts without crashing the server.

### Customizable Prompt Template

```text
Act as a Principal Engineer conducting a pre-code architecture review for [PROJECT_NAME].

Critically evaluate our PRD, HLD, Database Schema, and API Contracts across:
1. Unnecessary complexity or overengineering.
2. Single points of failure.
3. Security and data-isolation gaps.
4. External API / LLM failure modes (timeouts, rate limits, bad outputs).
5. Development speed and deployment bottlenecks within [TIMELINE, e.g., 24 hours].

For every issue found:
- Explain the real-world impact.
- Propose a simpler, MVP-friendly alternative.
- Identify any contradictions across our PRD, DB schema, and API specs.

Do not recommend microservices. Optimize strictly for shipping a reliable, demo-ready MVP.

```

---

## Master Capstone Prompt: Run the Full Blueprint Pipeline

Use this single prompt when starting a new project with an agentic coding tool (such as Claude Code) to generate, verify, and save all six engineering artifacts into a `docs/` folder before writing application code:

```text
Act as a Principal Software Architect and Product Engineering Lead.

I am building [PROJECT_NAME]: [ONE_LINE_DESCRIPTION_OF_PROJECT].

Preferred stack:
- Frontend: [FRONTEND_STACK]
- Backend: [BACKEND_STACK]
- Database: [DATABASE]
- External Integrations: [EXTERNAL_APIS]
- Timeline: [TIMELINE, e.g., 24-hour hackathon]

Produce a complete, implementation-ready engineering blueprint containing these 6 artifacts:
1. Product Requirements Document (PRD) — functional & non-functional requirements, user journeys, out-of-scope.
2. MVP Feature Matrix — prioritized using MoSCoW.
3. High-Level Design (HLD) — components, request flows, error handling, Mermaid diagram.
4. Database Design — PostgreSQL/DB schema, constraints, indexes, Mermaid ERD.
5. REST API Specification — endpoints, JSON request/response schemas, status codes, auth rules.
6. Architecture Decision Records (ADRs) — key stack choices, trade-offs, and risk mitigations.

Execution rules:
- Work sequentially from Phase 1 (Problem Analysis) through Phase 7 (Architecture Risk Review).
- Optimize strictly for a working modular monolith MVP; avoid unnecessary infrastructure.
- Preserve generation/audit history and enforce strict user-level data isolation.
- Address external API/LLM failure handling explicitly.
- Clearly label all assumptions and verify zero contradictions between the PRD, ERD, and API contracts.
- Do NOT write application source code yet.

First, present the proposed structure and key architectural decisions for my approval. Only after I approve should you save the finalized markdown documents inside a `docs/` directory.

```