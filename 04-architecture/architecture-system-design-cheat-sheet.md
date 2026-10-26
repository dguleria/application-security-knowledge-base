# Architecture & System Design Cheat Sheet
## Traditional enterprise, cloud-native, AI/LLM and agentic systems — with security overlays

> **Purpose:** A field guide for architects, engineering leaders, developers and security teams. Use it to frame decisions, compare options, document trade-offs and challenge designs. It is a decision aid—not a substitute for workload-specific modelling, threat assessment or testing.

## 1. The 60-second architecture method

1. **Clarify outcomes:** users, business capabilities, critical journeys, SLOs, regulatory obligations and measurable value.
2. **Characterise the workload:** read/write ratio, throughput, burstiness, payload size, data sensitivity, consistency, geographic distribution and growth.
3. **Set quality-attribute targets:** availability, latency (p50/p95/p99), durability, recovery time (RTO), recovery point (RPO), security, operability and cost.
4. **Choose the simplest viable shape:** modular monolith before distributed services unless independent scaling, team autonomy, fault isolation or deployment independence justify distribution.
5. **Draw boundaries and flows:** trust boundaries, data stores, identities, external dependencies, synchronous and asynchronous paths, and failure paths.
6. **Compare options explicitly:** benefits, costs, failure modes, reversibility and exit strategy. Record the decision in an ADR.
7. **Threat-model the design:** assets, actors, entry points, trust boundaries, abuse cases, controls and residual risk.
8. **Prove it:** load tests, failure injection, security tests, restore drills, AI evaluations where relevant, and operational runbooks.

### Architecture one-page canvas

| Question | Record |
|---|---|
| Business outcome | What changes for the user or business? |
| Context and actors | Users, systems, partners, operators, agents |
| Critical journeys | Top 3–5 end-to-end flows |
| Constraints | Budget, skills, cloud, latency, data residency, legacy |
| Quality targets | SLOs, p95/p99 latency, RTO/RPO, throughput |
| Data | Classification, owner, source of truth, retention, residency |
| Trust boundaries | Identity, network, tenant, tool and model boundaries |
| Architecture options | At least 2 viable choices and why one wins |
| Risks and controls | Top risks, preventive/detective/recovery controls |
| Operations | Telemetry, ownership, deployment, rollback, runbooks |
| Decision and review | ADR, assumptions, trigger to revisit |

## 2. Architecture patterns: when to use what

| Pattern | Use when | Advantages | Costs / failure modes | Security and design notes |
|---|---|---|---|---|
| Layered / n-tier | Conventional business apps with clear UI, domain and persistence concerns | Familiar, easy onboarding | Cross-layer coupling; business logic leaks into UI/data layers | Enforce validation and authorisation server-side; avoid trusting presentation-layer checks |
| Modular monolith | One product/domain, small-to-medium teams, early product evolution | Simple deployment and transactions; low operational overhead | Module boundaries can erode; one release unit | Enforce module APIs, dependency rules, least privilege for runtime identity |
| Microservices | Independent domain ownership, scaling, deployment or fault isolation are real requirements | Team autonomy, independent releases and scaling | Network failure, distributed data, observability and platform burden | Service identity, mTLS where appropriate, per-service authorisation, secrets, API controls |
| Service-based / macroservices | Several bounded services are enough; microservice granularity would be excessive | Fewer deployments and simpler operations | Larger blast radius than fine-grained services | Strong internal boundaries; limit shared credentials and data access |
| Hexagonal / ports-and-adapters | Domain logic must survive changes to databases, APIs or frameworks | Testable core; infrastructure replaceable | More interfaces and indirection | Put security policy at explicit boundaries; test adapters and authorisation paths |
| Clean architecture | Long-lived domain-heavy systems needing dependency inversion | Clear inward dependencies; domain isolation | Can become ceremony for CRUD applications | Keep policy and domain decisions testable; do not mistake layers for security boundaries |
| Domain-Driven Design (DDD) | Complex business domains and multiple teams | Shared language, bounded contexts, explicit ownership | Modelling overhead; overuse for simple domains | Boundaries expose data ownership; map sensitive data and access rules per context |
| Event-driven architecture (EDA) | Producers and consumers need temporal decoupling or multiple reactions to events | Loose coupling, burst absorption, extensibility | Eventual consistency, duplicate/out-of-order events, harder tracing | Authenticate producers; validate schemas; minimise event data; protect broker and replay paths |
| CQRS | Read and write models have meaningfully different needs | Independent query optimisation; explicit commands | More models, synchronisation and complexity | Authorise commands and queries separately; read models must not leak fields |
| Event sourcing | Auditable state transitions, replay or temporal reconstruction is a genuine requirement | History, auditability, rebuildable projections | Schema evolution, retention, replay complexity, storage | Append-only is not automatically tamper-proof; protect event access and redact via controlled strategies |
| Serverless / functions | Bursty/event-driven workloads with managed runtime acceptable | Low ops, elastic scale, pay-per-use | Cold starts, quotas, vendor constraints, distributed tracing | Short-lived least-privilege roles, secure triggers, dependency controls, egress restrictions |
| Microkernel / plugin | Stable core with independently developed capabilities | Extensible; plugin isolation possible | Versioning and plugin compatibility | Treat plugins as untrusted extensions; sandbox and scope permissions |
| Strangler Fig | Gradually replacing legacy capabilities | Incremental migration and rollback | Temporary dual-running and routing complexity | Secure old and new paths; prevent bypass of the facade; reconcile identities and audit trails |
| Anti-corruption layer | Integrating legacy or third-party domain models | Protects domain from external semantics | Mapping code and operational overhead | Validate and normalise inputs; prevent external trust assumptions leaking inward |
| Sidecar | Cross-cutting capability needs per-workload lifecycle (proxy, telemetry) | Separation of concerns | Resource overhead and multi-container complexity | Sidecar inherits powerful network position; harden, patch and scope its identity |
| Service mesh | Many services need consistent service-to-service traffic policy | Standardised identity, mTLS, telemetry, retries | Operational complexity; not business authorisation | Mesh controls do not replace application-level authorisation or input validation |
| Backend-for-Frontend (BFF) | Different clients need tailored APIs | Reduced client chatter and client-specific shaping | Duplicate logic and extra service | Enforce access controls at BFF and downstream; never rely on hidden UI fields |
| API gateway | Central routing, auth integration, quotas and API lifecycle | Consistent edge policies | Bottleneck, policy sprawl, false sense of security | Validate tokens, schemas, quotas and size; gateway is not the only authorisation layer |
| Multi-tenant architecture | One platform serves multiple customer organisations | Shared efficiency | Isolation mistakes, noisy neighbours | Tenant context must be server-derived and enforced on every query, cache, object and tool call |
| Cell-based architecture | Large-scale isolation and blast-radius containment | Limits impact; easier regional/cell scaling | Fleet management and placement complexity | Cell isolation, independent credentials, quotas and tested failover |
| Active-active multi-region | Very high availability and low geographic latency | Regional continuity | Conflict resolution, data sovereignty and cost | Define data authority, replication lag, regional key strategy and failover abuse cases |

**Rule of thumb:** distribution is a cost you choose to pay. Start with explicit module boundaries; split deployables when independent ownership, scale, resilience or release cadence provides measurable value.

## 3. Distributed systems and integration patterns

### Communication choice

| Option | Best for | Trade-off / guardrail |
|---|---|---|
| Synchronous REST/HTTP | Request-response, broad interoperability | Caller depends on callee availability and latency; use timeouts and bounded retries |
| gRPC / RPC | Typed service-to-service calls and streaming | Contract/tooling coupling; deadline propagation and compatibility matter |
| GraphQL | Client-selected data across related resources | Resolver N+1, complexity and authorisation pitfalls; enforce depth/cost limits and field-level policy |
| Queue / work queue | Background jobs and competing consumers | At-least-once delivery is common; idempotency and dead-letter handling required |
| Pub/sub events | One event consumed by multiple independent subscribers | Ordering often scoped to a key/partition; schema evolution and replay must be designed |
| Webhooks | External systems notify your service | Signature verification, replay defence, retries and allowlisted destinations |
| Batch / file transfer | High-volume, periodic or partner integration | Staleness and reconciliation; encrypt, checksum, validate schema and control retention |
| Streaming log | High-throughput event history and replay | Partition design, consumer lag, retention and operational expertise |

### Reliability patterns (apply deliberately)

- **Timeouts and deadlines:** every network call has a bounded deadline; propagate remaining budget downstream.
- **Retries:** only transient failures; exponential backoff plus jitter; cap attempts; avoid retry multiplication across layers.
- **Idempotency key:** deduplicate externally visible commands and payments; persist key, result and expiry safely.
- **Circuit breaker:** stop repeatedly calling a failing dependency; define half-open recovery and fallback semantics.
- **Bulkhead:** isolate thread pools, queues, tenants or workloads so one failure cannot exhaust everything.
- **Rate limit / load shed:** protect capacity using identity-aware quotas, queues and graceful rejection.
- **Outbox pattern:** commit domain state and an outbox record in one database transaction; publish asynchronously; consumers deduplicate.
- **Saga:** coordinate multi-service business transactions with explicit compensating actions; compensation is not always a true rollback.
- **Inbox / deduplication:** record processed message IDs to prevent duplicate side effects.
- **CQRS/read replica/cache:** reduce read load, but document staleness and invalidation semantics.
- **Leader election / leases:** prevent concurrent ownership where necessary; account for lease expiry and split-brain.
- **Backpressure:** let consumers signal or limit demand; do not buffer without bounds.
- **Graceful degradation:** define which features can be stale, disabled or read-only during dependency failure.

**Avoid:** unbounded retries, infinite queues, distributed transactions by default, assuming “exactly once” eliminates duplicate business effects, or adding a cache without an invalidation plan.

## 4. Data and storage selection

| Need | Likely fit | Key trade-off |
|---|---|---|
| Relational integrity, joins, transactions | Relational SQL | Schema and write contention; strong default for business records |
| Flexible documents / aggregate reads | Document database | Denormalisation and cross-document transaction constraints |
| Key-value, low-latency lookup | Key-value store | Limited query model; TTL and eviction semantics matter |
| Graph relationships and traversal | Graph database | Specialised modelling and query operations |
| Time-series metrics / telemetry | Time-series store | Cardinality, retention and downsampling |
| Search, relevance, log retrieval | Search index | Usually derived, eventually consistent data—not source of truth |
| Analytical warehouse / lakehouse | OLAP / lakehouse | Freshness, governance and compute cost |
| Object storage | Documents, media, backups, data lake | Lifecycle, access policies, object-level authorisation and public exposure risk |
| Vector index | Semantic retrieval / RAG | Recall-quality trade-offs, stale embeddings, tenant filtering and deletion propagation |
| Cache | Repeated reads and expensive computations | Staleness, stampedes, eviction and sensitive-data leakage |

### Data decision checklist
- Identify system of record and owner; distinguish source data from derived projections.
- Choose consistency semantics per operation: strong, read-your-writes, monotonic or eventual.
- Define keys, partitioning, hot-key strategy, indexing and migration path.
- Define retention, deletion, legal hold, backup, restore, encryption and key ownership.
- Enforce tenant and row/object-level authorisation in the data access path.
- Treat caches, logs, embeddings, analytics extracts and backups as data stores with equivalent classification obligations.
- Test restore and deletion end-to-end, including replicas, search indexes, vector stores and exports.

### CAP and consistency in plain language
During a network partition, a distributed data system cannot guarantee both a single globally consistent view and successful responses from every partition. CAP is not a blanket database ranking; normal operation, latency and specific consistency guarantees still matter. Choose per business operation and document what stale or unavailable behaviour users will see.

## 5. System design: capacity, performance and resilience

### Estimate before choosing technology
- **Request rate:** average RPS = daily requests ÷ 86,400; size for peaks, not just averages.
- **Concurrency:** Little’s Law: average in-flight work ≈ throughput × average time in system.
- **Data growth:** daily ingest × retention × replication/overhead factor.
- **Bandwidth:** requests/sec × average payload bytes; include response, replication and protocol overhead.
- **Queue stability:** sustained consumer throughput must exceed average arrival rate; otherwise lag grows without bound.
- **AI inference:** estimate input/output tokens per request × requests × price per token, plus retries, evaluations, retrieval, tools and idle capacity.

Use measured load tests and production telemetry to replace estimates. Include burst, skew, failover and degraded dependency scenarios.

### Performance levers
1. Remove unnecessary work and network round trips.
2. Profile and measure p50/p95/p99, not averages alone.
3. Optimise query plans, indexes and payload size.
4. Cache only with explicit freshness, invalidation and privacy rules.
5. Batch or stream where latency and semantics allow.
6. Apply bounded concurrency, connection pooling and backpressure.
7. Scale vertically first when it is simpler; scale horizontally when workload and architecture justify it.
8. Use asynchronous processing for work that need not block the user.
9. Load-test cold starts, cache misses, hot keys, large tenants and downstream throttling.

### Availability and disaster recovery
- **SLO:** user-relevant reliability target; **SLI:** measured indicator; **SLA:** external commitment with consequences.
- **Error budget:** tolerated unreliability within a period; use it to balance feature delivery and reliability work.
- **RTO:** maximum target time to restore service. **RPO:** maximum tolerable data loss measured in time.
- Redundancy only helps if failure domains are independent and failover works.
- Test backup restoration, credential recovery, dependency outages, regional loss and rollback.
- Identify common-mode failure: shared identity, DNS, secrets, control plane, deployment pipeline or data store.
- Define degraded modes, incident ownership, communications and operational runbooks.

## 6. Frameworks and reference models: what each is for

| Framework / method | Primary use | How to apply without turning it into paperwork |
|---|---|---|
| TOGAF | Enterprise architecture method, governance and views | Connect business capabilities to target architecture, transition states and architecture decisions |
| ArchiMate | Enterprise architecture modelling language | Show business, application, data and technology relationships consistently |
| C4 model | Software architecture communication | Context → containers → components → code only as needed; supplement with dynamic and deployment views |
| 4+1 views | Architecture views for different stakeholders | Logical, development, process, physical and scenarios; tailor views to concerns |
| ISO/IEC/IEEE 42010 | Architecture description and stakeholder concerns | Identify stakeholders, concerns, viewpoints, views and rationale |
| ADRs | Decision history | One decision, context, options, outcome, consequences and revisit trigger |
| DDD | Domain complexity and boundaries | Ubiquitous language, bounded contexts, aggregates and explicit context maps |
| Well-Architected frameworks | Quality review across operational excellence, security, reliability, performance, cost and sustainability | Use as a review checklist, not a substitute for workload-specific decisions |
| NIST CSF | Cybersecurity outcomes and risk communication | Govern, identify, protect, detect, respond and recover mapped to measurable outcomes |
| NIST SSDF | Secure software development practices | Integrate secure practices into the SDLC and supplier lifecycle |
| OWASP ASVS | Verifiable application security requirements | Select requirements by risk and verify evidence; map requirements to tests |
| OWASP SAMM | AppSec programme maturity | Baseline, prioritise, measure improvement and assign owners |
| STRIDE | Threat modelling by threat category | Examine spoofing, tampering, repudiation, information disclosure, denial of service and elevation of privilege |
| PASTA | Risk-centric threat modelling | Connect business impact, technical scope, attack paths and mitigations |
| MITRE ATT&CK | Adversary tactics and techniques | Use for detection coverage and adversary-informed validation, not as a complete architecture standard |
| Zero Trust (NIST SP 800-207) | Access decisions based on identity, context and resource | Verify explicitly, minimise access, assume breach; do not equate it with a product |
| FAIR | Quantitative cyber risk analysis | Estimate loss event frequency and magnitude where data supports it |
| SRE practices | Reliability engineering | SLOs, error budgets, toil reduction, incident learning and safe releases |
| FinOps | Cloud financial accountability | Unit costs, allocation, forecasting and engineering optimisation |
| ISO/IEC 42001 | AI management system | Organisational AI governance, roles, lifecycle processes and continual improvement |
| NIST AI RMF | AI risk management | Govern, map, measure and manage AI risks throughout the lifecycle |
| ISO/IEC 42005 | AI system impact assessment | Assess impacts of a specific AI system on people and society, proportionate to context |
| OWASP Top 10 for LLM Applications | Common LLM application risk awareness | Use as a threat prompt, alongside current versioned guidance and system-specific testing |
| MITRE ATLAS | AI-enabled adversary tactics and techniques | Inform AI threat scenarios, red-team exercises and detection planning |
| MAESTRO | Layered threat modelling for AI systems | Trace threats across AI/agent layers, dependencies and cross-layer interactions; complement with STRIDE and abuse cases |

Frameworks are complementary, not interchangeable. Record the actual control, owner, evidence, test and residual risk—not only the framework label.

## 7. AI application architecture: model, retrieval and orchestration

### Common reference flow
**Client → API/authentication → policy and input validation → orchestration → retrieval/tools/model → output validation → response**, with telemetry, evaluation, audit, cost controls and human escalation across the lifecycle.

Separate the following concerns:
- **Model:** generates predictions or tokens; may be hosted or third-party.
- **Application/harness:** prompts, orchestration, context assembly, tool permissions, state, retries and output handling.
- **Data/retrieval:** sources, chunking, embeddings, indexes, access filtering and provenance.
- **Evaluation:** offline test sets, online quality signals, safety and regression tests.
- **Operations:** latency, cost, token usage, rate limits, model versions and fallback.
- **Governance:** approved use, data handling, impact assessment, ownership and incident response.

### AI pattern selection

| Pattern | Use when | Benefits | Risks / trade-offs |
|---|---|---|---|
| Direct model call | Small, bounded task with no retrieval or tools | Simple and low latency | Limited grounding; prompt and output variability |
| Structured output / constrained decoding | Output must fit a schema or downstream contract | Easier validation and integration | Schema-valid does not mean true or safe |
| RAG (retrieval-augmented generation) | Answers must use changing or private knowledge | Grounding, source attribution, no model retraining required for each update | Retrieval misses, stale content, prompt injection in documents, access-filtering bugs |
| Tool/function calling | Model needs selected capabilities or fresh data | Explicit integration boundary | Tool abuse, excessive permissions, invalid arguments and side effects |
| Workflow / deterministic chain | Steps and transitions are known in advance | Predictability, testability and easier cost control | Less flexible when tasks vary |
| Router / model cascade | Requests differ by complexity, cost or sensitivity | Cost/latency optimisation | Routing errors, fallback complexity and inconsistent quality |
| Human-in-the-loop | High-impact, ambiguous or irreversible decisions | Oversight and accountable approval | Latency, workload and automation bias |
| Fine-tuning | Stable task behaviour, style or domain-specific response patterns are needed | Can improve consistency for a target task | Dataset cost, drift and limited factual freshness; not a replacement for access control |
| Small model / local inference | Privacy, offline operation, predictable narrow tasks or cost warrant it | Data locality and control | Hardware, quality, patching and operational burden |
| Multi-model ensemble | Independent signals add measurable value | Potential quality or robustness improvement | Cost, latency and correlated errors |
| Agentic loop | Goal requires dynamic planning, tool selection or iterative feedback | Handles variable multi-step work | Nondeterminism, loops, prompt injection, escalating tool risk and hard-to-bound cost |

**Default:** prefer a deterministic workflow with a model at the uncertain step. Add agent autonomy only when evaluations show that dynamic planning outperforms a simpler workflow enough to justify the added risk and operating cost.

### RAG design checklist
1. Ingest only approved sources; track owner, classification, version, timestamp and provenance.
2. Parse and chunk by document structure; test chunk size and overlap rather than assuming one optimum.
3. Choose embedding model and index based on language, domain, latency and retrieval evaluation.
4. Apply document-level access filtering **before content reaches the model**; do not rely on prompt instructions to enforce permissions.
5. Retrieve, rerank and bound context; deduplicate and preserve source references.
6. Treat retrieved content as untrusted data, not instructions. Detect or constrain embedded prompt injection.
7. Validate citations and abstention behaviour; distinguish “no evidence” from a confident answer.
8. Propagate deletion and updates through source stores, caches, embeddings and indexes.
9. Evaluate retrieval recall/precision, groundedness, answer correctness, leakage and cross-tenant isolation.
10. Monitor freshness, drift, failed retrievals and cost.

## 8. Agentic AI: architecture and autonomy controls

### Common agent building blocks
- **Goal/task contract:** objective, scope, success criteria, deadline, allowed actions and stop conditions.
- **Planner:** decomposes work; plans are proposals, not trusted commands.
- **Orchestrator/state machine:** controls transitions, retries, budgets and termination.
- **Model(s):** reasoning, classification or generation.
- **Tools:** narrow APIs for read/search/write/execute actions.
- **Memory/state:** working context, conversation state, durable task state and optional knowledge memory—each with separate retention and access rules.
- **Policy engine:** independent authorisation and risk checks before every tool action.
- **Verifier/evaluator:** checks outputs, invariants, evidence and completion.
- **Human approval:** required for defined high-impact, external or irreversible actions.
- **Audit/observability:** trace goal, model/version, retrieved evidence, tool arguments/results, policy outcomes and approvals with secrets/PII minimised.

### Autonomy ladder

| Level | Behaviour | Typical controls |
|---|---|---|
| 0 — Assist | Suggests or explains; no external action | Output validation, clear limitations |
| 1 — Read-only | Searches approved sources and summarises | Identity-scoped retrieval, tenant filtering, citations |
| 2 — Draft | Prepares emails, code, tickets or changes but does not submit | Preview, diff, validation and explicit user confirmation |
| 3 — Bounded action | Performs reversible actions in a narrow scope | Tool allowlist, scoped credentials, quotas, idempotency, audit and rollback |
| 4 — Conditional autonomy | Executes multi-step work within policy and risk limits | Runtime policy enforcement, independent verification, approval gates, kill switch and monitoring |
| 5 — Broad autonomy | Chooses and executes varied actions across systems | Exceptional risk; require strong justification, segmentation and continuous assurance; avoid for high-impact unsupervised actions |

Choose the **lowest autonomy level that meets the business outcome**. Autonomy is not a binary feature flag; it is a set of permissions, limits and verifiable conditions.

### Single agent vs multi-agent

| Choose | When | Trade-off |
|---|---|---|
| Single agent with tools | One coherent task and shared context | Simpler debugging, permissions and cost |
| Deterministic workflow | Known steps and critical correctness | Less flexible; often best for regulated transactions |
| Supervisor + specialists | Distinct specialist tasks with clear contracts | Handoffs, orchestration and aggregate failure risk |
| Parallel agents | Independent research or analysis can run concurrently | Higher cost; duplicate work and conflicting results |
| Debate/critic-verifier | Independent review measurably improves outcomes | Agreement can be correlated; extra tokens and latency |
| Hierarchical agent teams | Large tasks need decomposition and delegated subtasks | Harder tracing, privilege propagation and termination control |

Multi-agent systems do not automatically become more intelligent or reliable. Use them only when task separation, parallelism or independent verification produces measurable benefit.

### Agent execution guardrails
- Set per-task limits for wall-clock time, model calls, tokens, tool calls, retries, spend and recursion depth.
- Give each tool a narrow schema, explicit side-effect class and least-privilege identity.
- Enforce authorisation outside the model; check resource, tenant, user, action and context on every call.
- Separate read and write tools. Require confirmation for sending, publishing, deleting, paying, granting access or deploying.
- Make actions idempotent where possible; use dry-run, preview, transaction boundaries and compensating actions.
- Treat user text, emails, web pages, documents, tool output and memory as untrusted input.
- Prevent indirect prompt injection from overriding system policy or tool permissions.
- Never expose secrets in prompts, tool output, traces or error messages; use secret brokers and short-lived credentials.
- Keep durable memory scoped, inspectable, correctable and subject to retention/deletion policies.
- Detect loops, repeated failed actions, policy denials, unusual tool sequences and budget exhaustion.
- Add a kill switch, task cancellation, quarantine path and human escalation.
- Verify completion from authoritative system state, not the agent's own claim that it succeeded.

## 9. Security overlay: apply to every architecture

### Identity and access
- Human SSO/MFA, workload identity, short-lived credentials and lifecycle automation.
- Least privilege at user, service, database, queue, cloud, model and tool layers.
- Server-side authorisation on every request; deny by default; protect object-level and tenant-level access.
- Separate developer, CI/CD, runtime and break-glass identities; monitor privileged access.
- Use explicit delegation/actor context when an agent acts for a user; preserve attribution and do not silently elevate permissions.

### Application and API security
- Threat-model business flows and trust boundaries; validate inputs and canonicalise consistently.
- Strong authentication, object/function-level authorisation, quotas, payload limits and abuse protection.
- Safe output encoding, injection prevention, SSRF controls and secure file handling.
- Secure session/cookie configuration, CSRF protection where relevant, CORS allowlists and security headers.
- Schema validation, versioning, backward compatibility and safe error responses.
- Protect webhooks against forgery and replay; restrict outbound destinations to reduce SSRF/data exfiltration.

### Data security and privacy
- Classify data; minimise collection; define purpose, retention, residency and deletion.
- Encrypt in transit and at rest; manage keys separately with rotation and audited access.
- Mask/tokenise sensitive fields; prevent sensitive content from entering telemetry by default.
- Apply row-, object-, document- and tenant-level access checks in data and retrieval layers.
- Include logs, caches, backups, embeddings, vector stores, model inputs/outputs and evaluation datasets in data governance.

### Cloud, infrastructure and platform
- Segmented networks, controlled egress, private endpoints where warranted and hardened baseline images.
- Infrastructure as code, policy as code, drift detection and reviewed privileged changes.
- Patch operating systems, runtimes, frameworks, containers, models and dependencies.
- Container/Kubernetes controls: non-root, minimal capabilities, resource limits, network policy, secret mounts, admission policy and image provenance.
- Separate environments and accounts/subscriptions; avoid production data in development without approved protection.
- Protect CI/CD identities, runners, artifacts, registries, signing keys and deployment approvals.

### Supply chain and secure delivery
- Threat-model third-party services and AI/model providers; review permissions, data use, retention and subprocessors.
- Pin dependencies where appropriate; scan source, dependencies, IaC, containers and secrets.
- Produce and verify SBOMs; assess provenance and artifact integrity; sign and verify release artifacts.
- Use protected branches, review, isolated builds and short-lived pipeline credentials.
- Treat AI-generated code like all other code: review, test, scan, license-check and verify behaviour before merge.
- Record exceptions with owner, rationale, expiry, compensating controls and revalidation date.

### AI/LLM-specific security
- **Prompt injection:** instruction/data separation, trusted templates, content isolation, tool policy enforcement and adversarial tests.
- **Sensitive data disclosure:** minimisation, access-filtered retrieval, output checks and provider/data-processing review.
- **Unsafe tool use:** allowlisted tools, typed arguments, per-action authorisation, bounded side effects and human approval.
- **RAG poisoning:** source provenance, ingestion controls, integrity monitoring and retrieval evaluation.
- **Cross-tenant leakage:** enforce tenant filters in retrieval, cache keys, memory, traces and tool calls; test adversarially.
- **Model/output risk:** validate schema and semantics, constrain downstream execution, encode outputs and require verification for high-impact results.
- **Denial of service/cost abuse:** quotas, token and context limits, rate limits, timeouts, caching where safe and spend alerts.
- **Model/supply-chain risk:** inventory models and versions, verify source/licence, assess provider changes and maintain fallback plans.
- **Agent memory:** classify and scope memory, avoid treating stored text as trusted policy, protect against poisoning and provide correction/deletion workflows.
- **Evaluation:** maintain versioned safety, quality, robustness and security test sets; test regressions before model, prompt, retrieval or tool changes.

### Detection, response and recovery
- Centralise security-relevant events with reliable timestamps and correlation/trace IDs.
- Log actor, resource, action, policy decision and outcome; minimise secrets and sensitive content.
- Alert on privilege changes, anomalous access, exfiltration indicators, agent tool abuse, cost spikes and policy bypass.
- Define incident playbooks for identity compromise, data exposure, model/provider outage, poisoned retrieval and unsafe agent actions.
- Exercise containment, credential revocation, rollback, restore, evidence preservation and customer communication.

## 10. Security threat-modelling quick start

For each component and flow:
1. **Assets:** credentials, personal/financial/health data, business decisions, model artefacts, prompts, tools, audit evidence.
2. **Actors:** legitimate users, tenant admins, insiders, compromised services, malicious suppliers and external attackers.
3. **Entry points:** UI/API, webhooks, uploads, queues, plugins, MCP/tool endpoints, retrieval sources, model providers and CI/CD.
4. **Trust boundaries:** user-to-service, tenant-to-tenant, service-to-data, application-to-model, agent-to-tool, cloud-to-partner.
5. **Abuse cases:** spoof, tamper, repudiate, disclose, deny service, elevate privilege; add prompt injection, poisoning, tool misuse, exfiltration and unsafe autonomy for AI.
6. **Controls:** preventive, detective, responsive and recovery controls with named owners.
7. **Validation:** abuse-case tests, negative authorisation tests, red-team scenarios, telemetry checks and recovery exercises.
8. **Residual risk:** likelihood/impact, risk owner, treatment, expiry and accepted evidence.

**Useful output:** architecture/data-flow diagram, threat register, control mapping, test plan, residual-risk decisions and operational playbooks. A threat-model workshop without verified actions is not completion.

## 11. Trade-off decision matrix

Score each candidate from 1 (poor) to 5 (strong), weight the criteria for the business context, and record evidence and uncertainty. Do not treat the arithmetic as objective truth.

| Criterion | Questions |
|---|---|
| Business fit | Does it solve the actual user and business problem? |
| Complexity | How much new operational and conceptual complexity is added? |
| Delivery speed | Time to first value and to safe production? |
| Reliability | What fails, how is it detected, and how does it recover? |
| Security/privacy | Can controls be enforced, tested and audited? |
| Performance | Does it meet p95/p99 latency and throughput targets? |
| Scale | Can it handle expected growth, bursts and skew? |
| Cost / unit economics | Infrastructure, licences, support, people and AI inference per outcome |
| Operability | Can teams deploy, debug, observe, patch and restore it? |
| Skills / ecosystem | Can the organisation operate it safely over time? |
| Portability / reversibility | Can data and workloads be moved; what is the exit cost? |
| Governance | Can decisions, access, data lineage and evidence be demonstrated? |

### Common decisions and defaults
- **Monolith vs microservices:** modular monolith until independent deployment, scaling or ownership is worth distributed-systems cost.
- **Sync vs async:** sync when the user needs an immediate answer; async for long-running work, bursts and decoupling—with status, retries and idempotency.
- **SQL vs NoSQL:** start from integrity, query and transaction needs, not fashion; relational is a strong default for transactional business data.
- **Cache vs source read:** cache when measured latency/load justifies it and staleness can be specified.
- **Build vs buy:** build differentiating capability; buy commodity capability when risk, integration, data handling, lock-in and exit terms are acceptable.
- **Multi-region vs single-region HA:** use multi-region only when business RTO/RPO, latency or regulatory needs justify complexity and cost.
- **RAG vs fine-tuning:** RAG for changing, permissioned facts; fine-tuning for repeatable behaviour/style/task adaptation; combine only with a clear reason.
- **Workflow vs agent:** deterministic workflow for known steps; agent for uncertain branching where autonomy demonstrably improves outcomes.
- **Single vs multi-agent:** single agent first; split only for independent specialisation, parallelism or verified review benefit.
- **Model size vs routing:** use the smallest/cheapest model that meets evaluated quality and safety thresholds; escalate selectively.
- **Managed service vs self-hosted:** compare control and residency with patching, availability, operational skill, cost and provider dependency.
- **Prevent vs detect vs recover:** prevention is not perfect; design detection, containment and recovery as first-class controls.

## 12. Architecture decision records (ADR)

Copy this template for every material decision.

```markdown
# ADR-NNN: <decision title>
- Status: Proposed | Accepted | Superseded
- Date:
- Owners:
- Review trigger/date:

## Context
Business outcome, constraints, quality targets, assumptions and evidence.

## Decision
What is being selected and the scope of the decision.

## Options considered
1. Option A — benefits, costs, risks.
2. Option B — benefits, costs, risks.
3. Option C — benefits, costs, risks.

## Rationale and trade-offs
Why this option wins; what is intentionally not optimised.

## Security, privacy and AI impact
Threats, data classification, identities, trust boundaries, safeguards,
human oversight, model/provider dependencies and residual risks.

## Operations and validation
SLOs, telemetry, deployment/rollback, failure tests, security tests,
cost measures, owner and runbook.

## Consequences
Positive, negative, migration and exit implications.
```

## 13. Architecture review checklist

### Business and boundaries
- [ ] Business capability, users, critical journeys and success metrics are explicit.
- [ ] Context/container/data-flow views match the deployed reality.
- [ ] Ownership, interfaces and bounded contexts are clear.
- [ ] Assumptions, constraints and decision records are visible.

### Quality and resilience
- [ ] SLOs, p95/p99 latency, throughput, RTO/RPO and error budgets are defined.
- [ ] Timeouts, bounded retries, idempotency, backpressure and graceful degradation are designed.
- [ ] Failure domains and common-mode failures are understood.
- [ ] Load, failover, backup restore and rollback are tested.

### Data and integration
- [ ] Source of truth, consistency, schema evolution and reconciliation are defined.
- [ ] Data classification, residency, retention and deletion include derived stores and telemetry.
- [ ] External dependencies, contracts, quotas and failure behaviour are documented.
- [ ] Tenant isolation and authorisation are tested at every access path.

### Security and operations
- [ ] Threat model and abuse cases have owners and evidence-backed mitigations.
- [ ] Least privilege, secrets, key management, egress and audit are designed.
- [ ] Secure SDLC, dependency controls, SBOM/provenance and deployment protections are present.
- [ ] Telemetry, alerting, incident response, runbooks and service ownership are ready.

### AI and agents (when applicable)
- [ ] Intended use, impact assessment, model/provider inventory and data-use terms are documented.
- [ ] Evaluation set covers quality, grounding, refusal, privacy, prompt injection and misuse.
- [ ] Retrieval enforces permissions before context reaches the model.
- [ ] Tools have narrow schemas, least privilege, independent policy checks and side-effect controls.
- [ ] Autonomy, budgets, stop conditions, human approval, audit and kill switch are explicit.
- [ ] Model/prompt/index/tool changes trigger regression evaluation and controlled rollout.

## 14. Architecture smells and anti-patterns

- **Premature microservices:** distributed deployment without independent ownership or scaling need.
- **Distributed monolith:** services cannot deploy or fail independently because of synchronous chains and shared data.
- **Shared database without ownership:** teams change tables without stable contracts or migration discipline.
- **God service / god agent:** one component has excessive responsibilities, permissions or context.
- **Chatty synchronous chain:** user latency and availability depend on too many downstream calls.
- **Retry storm:** retries amplify the original outage.
- **Cache as source of truth:** invalidation and recovery become business correctness problems.
- **Security gateway myth:** edge controls are assumed to replace resource-level authorisation.
- **Log everything:** secrets and personal data leak into observability systems.
- **AI as policy engine:** natural-language instructions are trusted to enforce permissions.
- **Agent autonomy by default:** model can write, send, delete or deploy without independent controls.
- **RAG equals truth:** retrieved documents can be stale, malicious, incomplete or unauthorised.
- **Evaluation by demos only:** happy-path examples conceal regressions and unsafe edge cases.
- **Framework compliance without evidence:** labels and checklists exist but controls are not tested.
- **Multi-region without tested failover:** replicas exist, but recovery, identity, DNS or data authority is unproven.

## 15. Reference resources

Prefer primary sources and verify the current edition before using requirements in a formal assessment.

- [C4 model](https://c4model.com/)
- [ISO/IEC/IEEE 42010 overview](https://www.iso.org/standard/74393.html)
- [TOGAF](https://www.opengroup.org/togaf)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/)
- [Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [NIST Secure Software Development Framework (SP 800-218)](https://csrc.nist.gov/pubs/sp/800/218/final)
- [NIST Zero Trust Architecture (SP 800-207)](https://csrc.nist.gov/pubs/sp/800/207/final)
- [OWASP Application Security Verification Standard](https://owasp.org/www-project-application-security-verification-standard/)
- [OWASP Software Assurance Maturity Model](https://owaspsamm.org/)
- [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
- [NIST AI RMF Playbook](https://airc.nist.gov/AI_RMF_Knowledge_Base/Playbook)
- [MITRE ATLAS](https://atlas.mitre.org/)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [Google SRE book](https://sre.google/sre-book/table-of-contents/)
- [FinOps Framework](https://www.finops.org/framework/)
- [Cloud Native Computing Foundation](https://www.cncf.io/)

---
**Maintenance note:** Review this guide at least every six months and after material changes to architecture standards, cloud services, AI threat guidance or regulatory obligations. Frameworks evolve; validate version and jurisdiction before treating any reference as a mandatory requirement.
