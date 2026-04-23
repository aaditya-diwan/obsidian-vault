# Technical Interview Preparation
### Aaditya Diwan — Senior Backend Engineer

> Derived from 4+ years at Principal Global Services (Intern → Senior SE) + MediTrack & FocusFlow projects.
> Every question maps directly to real experience. Answers written at the level of a Senior SWE at a top-tier product company.

---

## Section 1: System Design

---

### Q1. Walk me through designing the event-driven insurance policy processing system you built. How did you ensure reliability, idempotency, and scalability?

**Why they're asking:** They want to know if you can translate a real system into a structured design, explain your choices, and identify failure modes before they happen.

**Answer:**

The core constraint was that insurance policy events are financially consequential — processing an event twice or losing one has real business and regulatory impact. So correctness came before throughput in my priority ordering.

**High-level architecture:**

```
Event Source (API Gateway / S3 / upstream service)
        │
    AWS SQS (buffer + backpressure)
        │
    AWS Lambda (Ingestion)
        │
    AWS Step Functions (Orchestration)
    ┌───┴──────────────────────────────────────┐
    │  State 1: Validate Policy Data           │
    │  State 2: Enrich from IBM DB2            │
    │  State 3: Apply Business Rules (Lambda)  │
    │  State 4: Persist to DynamoDB            │
    │  State 5: Publish Downstream Events      │
    └──────────────────────────────────────────┘
        │
    SQS → Downstream consumers
```

**Why Step Functions over a raw SQS → Lambda chain:**

Two reasons. First, auditability — insurance workflows require an execution trail. Step Functions gives you a visual execution history for every policy event: which step ran, how long it took, what it returned. That's compliance evidence. A raw Lambda chain requires you to piece together CloudWatch log entries across multiple functions to reconstruct what happened.

Second, native error handling. Step Functions lets you define retry policies (with backoff), `Catch` blocks per state, and fallback states — all in configuration, not code. In a pure Lambda chain, every function re-implements retry logic and you inevitably get inconsistencies.

**Idempotency:**

SQS delivers at-least-once. The Step Functions execution name was keyed on the policy event ID — so if SQS re-delivered an event, the Step Functions API would reject a duplicate execution name with a conflict error, which we treated as success. For the DynamoDB write step, we used conditional writes (`attribute_not_exists(pk)`) so a duplicate persist attempt failed safely.

**Scaling:**

Lambda scaled horizontally by default. The real concern was IBM DB2 — a legacy relational system with fixed connection limits. We addressed this by rate-limiting Lambda concurrency (reserved concurrency = max safe DB connection count), and using connection reuse via shared connection pool initialized outside the Lambda handler (persisted across warm invocations).

**What I'd change at 10x volume:**

If event volume grew to tens of millions per month, Step Functions Standard Workflow costs ($0.025 per 1,000 state transitions × many steps) would compound significantly. I'd move the high-throughput commodity processing to Apache Kafka + Kafka Streams, retaining Step Functions only for complex, multi-day human-approval workflows where the audit trail is non-negotiable.

**Trade-offs:**
- Step Functions adds ~100ms overhead per state. For latency-critical paths, we collapsed multiple steps into a single Lambda.
- Cold starts (especially JVM Lambdas) were mitigated with provisioned concurrency on the ingestion Lambda, which was on the critical path.
- Express Workflows were evaluated for high-volume, short-lived workflows — cheaper, but no full execution history. We kept Standard for policy processing given the compliance requirement.

---

### Q2. You implemented PII data protection using AWS KMS + DynamoDB, aligned with GDPR Article 25. Walk me through the full encryption architecture.

**Why they're asking:** Security-aware system design is increasingly valued. They want to see if you actually understand encryption mechanics or just configured a checkbox.

**Answer:**

GDPR Article 25 is "data protection by design" — not encryption at rest as a DynamoDB default setting, but deliberate, field-level protection where the application owns the encryption, not the storage layer. The distinction matters: if you rely solely on DynamoDB's server-side encryption and AWS is compromised (or an over-privileged IAM role is exploited), your PII is exposed. Envelope encryption at the application layer means even internal actors can't read PII without explicit KMS permission.

**Architecture — Envelope Encryption:**

```
Application writes PII:

1. Call KMS GenerateDataKey(CMK-ARN)
   → Returns: { PlaintextDEK, EncryptedDEK }

2. Use PlaintextDEK to encrypt PII fields (AES-256-GCM)
   → ssn_encrypted, name_encrypted, etc.

3. Discard PlaintextDEK from memory

4. Write to DynamoDB:
   {
     "policyId": "P-1234",          ← plain identifier
     "ssn":      "<ciphertext>",    ← encrypted
     "dek":      "<EncryptedDEK>",  ← KMS-encrypted key stored alongside
     "status":   "ACTIVE"           ← non-PII, plain
   }

Application reads PII:

1. Fetch DynamoDB item → get EncryptedDEK + ciphertext
2. Call KMS Decrypt(EncryptedDEK) → PlaintextDEK
3. Decrypt ciphertext with PlaintextDEK → plaintext PII
4. Discard PlaintextDEK
```

**Why envelope encryption:**
KMS is not designed to encrypt large data — it has a 4KB limit. The DEK model is how KMS is meant to be used. Critically, key rotation only requires re-encrypting the small DEK, not re-encrypting terabytes of data.

**Access control:**
IAM policies restricted `kms:Decrypt` to specific Lambda execution roles — not to individual engineers, not to the DynamoDB service itself. CloudTrail logged every Decrypt API call with principal identity, timestamp, and key ARN. We set a CloudWatch alarm for anomalous decrypt volume (a spike = potential data exfiltration attempt).

**GDPR alignment:**
- **Article 25 (data protection by design):** Encryption at the application layer, not relying on storage defaults
- **Article 17 (right to erasure):** Crypto-shredding — deleting the CMK or rotating and revoking old key versions permanently renders historical ciphertexts undecipherable without touching every DynamoDB record
- **Article 5 (data minimisation):** Only fields that are legally required were encrypted and stored; others were dropped at ingestion

**Performance trade-off:**
Each DynamoDB read now requires a KMS Decrypt call (~3-5ms added latency). We mitigated this by caching the plaintext DEK in Lambda memory for the lifetime of a warm invocation — a single decrypt per cold start rather than per request. The DEK never touches disk or logs.

**Alternatives considered:**
The AWS Encryption SDK abstracts envelope encryption and handles DEK caching natively. We evaluated it but implemented manually for finer control over the caching strategy and to avoid adding a dependency with its own upgrade burden at the time.

---

### Q3. You migrated legacy SOAP services to REST and improved availability from 97.5% to 99.2%. How did you architect a zero-downtime migration?

**Why they're asking:** Legacy migrations are one of the highest-risk, highest-reward projects a backend engineer can own. They want to know if you think in risk-managed increments or just rewrite and pray.

**Answer:**

A big-bang SOAP-to-REST rewrite is how you create a production incident that makes it into the company retrospective. These SOAP services had downstream consumers — other internal services, third-party integrations, reporting pipelines — that had built-in expectations about payload structure and error formats. Any migration needed to be invisible to consumers until we chose to advertise the change.

**Strategy: Strangler Fig + Traffic Shadowing**

```
Phase 0 (Before):
Consumers → Legacy SOAP Service

Phase 1 (Routing layer introduced):
Consumers → API Gateway → Legacy SOAP Service (100% traffic)

Phase 2 (Parallel new service, shadow mode):
Consumers → API Gateway ─┬─→ Legacy SOAP Service (100% live)
                          └─→ New REST Service (shadow, response discarded)

Phase 3 (Gradual cutover):
Consumers → API Gateway → New REST Service (5% → 25% → 50% → 100%)

Phase 4 (Decommission):
Legacy SOAP Service → shutdown
```

**Why shadow mode first:**
Before routing any real user traffic to the REST service, we ran shadow mode for one full business cycle. Every request was mirrored to both services. We compared: response structure equivalence, error rates, latency profiles. Shadow mode surfaced 3 edge cases that our test suite hadn't covered — SOAP-specific payload quirks in multi-line address fields and null handling in legacy business rule outputs. We fixed these before a single user saw the REST service.

**Gradual cutover with feature flags:**
We used AWS AppConfig to control the routing split percentage. Each increment (5% → 25%) was gated on 48 hours of stable metrics: error rate parity between old and new, p99 latency within 10% of baseline. Rollback was a single configuration toggle — no deployment, no code change.

**Why availability improved:**
- The SOAP service had XML parsing overhead that spiked CPU under concurrent load → timeouts → cascading failures → 2.5% downtime
- It also had implicit session state coupling — one service's session affected another's — which caused hard-to-reproduce failures
- The REST replacement was stateless, Lambda-backed (no session state possible), with lighter JSON payloads and explicit timeout budgets at every external call
- The API Gateway layer itself added health check + circuit breaker capabilities that the old SOAP stack lacked

**What I watched for that most people miss:**
Consumer error-code expectations. SOAP has `<faultcode>` and `<faultstring>` in the envelope. REST returns HTTP status codes. Any consumer doing `if (fault == "POLICY_NOT_FOUND")` needed to be updated or we needed an HTTP → SOAP fault translation layer during the transition. We catalogued all downstream consumers before beginning and coordinated upgrade timelines.

---

### Q4. Walk me through the MediTrack microservices architecture. How did you decide service boundaries, communication patterns, and data ownership?

**Why they're asking:** Service boundaries are where most microservices projects go wrong. They want to see if you reason from business capabilities and failure domains, not from entity names.

**Answer:**

The instinct is to split by entity — one service per database table. That's the wrong model. The right question is: what are the independent business capabilities, and which services must stay running if others fail?

**Boundary reasoning:**

| Service | Business Capability | Failure Impact |
|---------|---------------------|----------------|
| Patient Service | Identity & demographics | Everything needs this — so it must be highly available and simple |
| Lab Service | Order lifecycle, results, critical flags | Lab can go down without affecting insurance policy management |
| Insurance Service | Policy CRUD, coverage rules | Insurance can be down without blocking lab operations |

A lab system going down should not prevent patient registration. If they shared a monolithic codebase or a shared database, a lab schema migration would require coordinated downtime across the system. Separate services, separate databases, separate deployment pipelines.

**Data ownership — no cross-service joins:**

Each service owns its data exclusively. The Insurance Service needs minimal patient data (patient ID, name for display). Instead of calling Patient Service synchronously on every policy operation, it subscribes to `patient.created` Kafka events and stores a local projection of the fields it needs. This is deliberate denormalization. The trade-off — slightly stale data — is acceptable. The benefit — Insurance works even if Patient Service is down — is essential.

**Communication decisions:**

*Synchronous REST* for:
- User-initiated operations that need immediate feedback (patient registration form, lab order submission)
- Idempotent reads where staleness is unacceptable
- Failing fast with HTTP 503 is better than blocking indefinitely when downstream is unavailable

*Asynchronous Kafka* for:
- Cross-service state propagation (`patient.created` → Insurance creates a linked record)
- Critical result flags that need reliable at-least-once delivery
- Anything where the producer shouldn't be blocked waiting for the consumer

**Frontend BFF (Backend for Frontend) layer:**

The Next.js App Router frontend never directly calls microservice URLs. All API calls go through Next.js server API routes, which act as an aggregation and auth proxy layer. This:
- Keeps service URLs, auth tokens, and service discovery off the browser
- Lets us aggregate data from multiple services (patient + lab + insurance) into a single response for a dashboard view — no N+1 requests from the browser
- CORS becomes a non-issue since all calls are server-to-server

**Hexagonal architecture per service:**

Domain logic must not know whether it's being invoked via REST, Kafka, or a test. Hexagonal architecture (ports and adapters) achieves this:

```
[REST Controller] ─→ [LabOrderPort interface] ─→ [LabOrderService domain] ─→ [LabOrderRepository port]
[Kafka Listener]  ─→ [LabOrderPort interface] ─→ [LabOrderService domain] ─→ [DB Adapter]
[Unit Test]       ─→ [LabOrderPort interface] ─→ [LabOrderService domain] ─→ [In-Memory Adapter]
```

I can test every business rule in the domain layer with pure Java unit tests — no Spring context boot, no Kafka broker, no database. Tests run in milliseconds and are completely deterministic.

---

### Q5. Design a developer experience (DX) platform — specifically, a shared microservice design library like the one you built for 10+ teams at Principal.

**Why they're asking:** Senior engineers are expected to think about platform leverage — how do you make 10 teams faster simultaneously?

**Answer:**

The problem isn't that each team can't write retry logic. It's that they each write it slightly differently. Team A retries 3 times with fixed 1s interval. Team B retries 5 times but doesn't handle `InterruptedException`. Team C built a custom exception hierarchy that their downstream can't interpret. When a cross-team incident occurs, every team's logging format is different and correlation is manual.

A shared library solves this. But a poorly designed shared library creates a different problem: a bottleneck where 10 teams are blocked on one team's review queue.

**What I put in the library:**

**1. Standardized Logging Foundation:**
Pre-configured SLF4J + Logback with structured JSON output, MDC integration for correlation IDs, and standard log level semantics. Every service emits the same log schema — `{"correlationId": "...", "service": "...", "event": "...", "duration_ms": ...}` — so CloudWatch Insights queries work identically across services.

**2. Exception Hierarchy:**
```
ServiceException (HTTP 500 — base)
├── ValidationException (400)
├── ResourceNotFoundException (404)
├── ExternalServiceException (502)
└── RetryableException (triggers retry at caller)
```
Every exception maps to a consistent HTTP response body. Consumers can write a single error handler. Before this, each service had ad-hoc exception types with inconsistent HTTP status mappings.

**3. Resilience Primitives:**
Pre-configured Resilience4j `CircuitBreaker` and `Retry` beans with sensible production defaults. Teams could override for specific use cases, but 80% used the defaults without modification — reducing the cognitive load of "what are the right retry settings for calling DB2?"

**4. AWS Client Configurations:**
Pre-configured SQS, DynamoDB, and Lambda SDK clients with appropriate SDK retry policies, connection pool sizes, and regional endpoint resolution. The service-specific SDK configuration mistakes I'd seen (default retry causing thundering herd, wrong timeout values causing Lambda OOM) were encoded as correct defaults.

**5. Health Check Contract:**
Standard `GET /actuator/health` response structure. Enables Dynatrace and CloudWatch monitoring dashboards to be built once and applied to all services.

**How I prevented it from becoming a bottleneck:**

**SemVer with a 2-major-version support window:** Teams aren't forced to upgrade. They pin to a minor version range and upgrade on their schedule. Breaking changes only in major versions, with a documented migration guide and a 6-month deprecation window.

**Contribution from any team via PR:** The Technical Pillar owns the library, but any team can submit a PR. We had a clear contribution guide and a 2-business-day review SLA. A library only one team can change is a single point of failure for developer productivity.

**Documentation as a release gate:** No feature merged without Javadoc, a usage example in the README, and a changelog entry. Without this, adoption requires Slack messages to us for every question — which doesn't scale.

**What I'd do differently:** At scale, I'd consider separating the library into narrower modules (logging, resilience, AWS clients) rather than one monolithic jar. Smaller modules = smaller upgrade surface = fewer adoption blockers per team.

---

## Section 2: Backend Engineering & Patterns

---

### Q6. Explain the Transactional Outbox Pattern. Why did you implement it in MediTrack's lab service specifically?

**Why they're asking:** The dual-write problem is one of the most common correctness bugs in event-driven systems. They want to know if you understand it mechanically or just read about it.

**Answer:**

**The problem — the dual-write dilemma:**

When a lab order completes, the service needs to:
1. Update the order status in PostgreSQL (`status = COMPLETED`)
2. Publish a `lab.order.completed` event to Kafka

These are two separate systems. There is no atomic "write to DB AND publish to Kafka" operation. If you do (1) then (2) and Kafka is down, the order is marked complete but no downstream notification is sent. The Insurance service never knows. If you do (2) then (1) fails, you've published a lie — an event for an order that never completed.

This is the dual-write problem. Traditional distributed transactions (2PC) solve it but at the cost of availability and complexity. For most microservices, 2PC is too heavy.

**The Outbox solution:**

```java
@Transactional  // ← Single database transaction
public void completeLabOrder(String orderId, LabResult result) {
    // Step 1: Update order status
    LabOrder order = labOrderRepo.findById(orderId);
    order.complete(result);
    labOrderRepo.save(order);

    // Step 2: Write to outbox table — IN THE SAME TRANSACTION
    OutboxEvent event = OutboxEvent.builder()
        .topic("lab.order.completed")
        .aggregateId(orderId)
        .payload(serialize(order))
        .published(false)
        .build();
    outboxRepo.save(event);
    
    // Both writes commit atomically, or both roll back
}
```

A separate relay process polls the outbox table for unpublished events and publishes to Kafka:

```java
@Scheduled(fixedDelay = 1000)
@Transactional
public void relay() {
    List<OutboxEvent> pending = outboxRepo.findByPublishedFalse();
    for (OutboxEvent event : pending) {
        kafkaTemplate.send(event.getTopic(), event.getAggregateId(), event.getPayload());
        event.setPublished(true);
        outboxRepo.save(event);
    }
}
```

**Idempotency at the consumer:**
The relay guarantees at-least-once delivery (if it publishes but crashes before marking as published, it re-publishes on restart). Kafka consumers must handle duplicates — we used the `orderId` as the Kafka message key (same key → same partition → ordered delivery) and consumers checked for duplicate event IDs in their own DB before processing.

**Alternatives I evaluated:**
- **Kafka Transactions:** Kafka has transactional producers, but they don't span a Kafka transaction and a DB transaction — you'd still have the dual-write problem.
- **Change Data Capture (Debezium):** Debezium watches PostgreSQL's WAL and emits events. More operationally complex (separate Kafka Connect cluster), but eliminates the polling overhead. Better choice for high-throughput scenarios.
- **Saga Pattern:** For compensating transactions across multiple services. Not the right fit here — we needed reliable event delivery within a single service.

**Trade-offs I accepted:**
- ~1 second latency between order completion and event publication (polling interval)
- Outbox table grows until events are published — needs a cleanup job
- At high write rates, the relay loop can become a bottleneck. Mitigated by partitioned polling (relay workers own subsets of outbox rows by ID range)

---

### Q7. Walk me through the RBAC implementation in MediTrack using Spring Security and JWT. What are the security design decisions you made?

**Why they're asking:** Security-conscious backend engineers design auth systems that are correct by construction, not correct by convention.

**Answer:**

**Full request lifecycle:**

```
POST /auth/login { username, password }
        │
AuthController → UserDetailsService.loadByUsername()
        │                    │
        │           PasswordEncoder.matches()
        │
JwtTokenProvider.generateToken(user)
        │  → Signs with RS256 (RSA private key)
        │  → Payload: { sub, roles: ["ROLE_DOCTOR"], iat, exp: +1h }
        │
Response: { accessToken, refreshToken }

...

GET /labs/orders/456
Authorization: Bearer <token>
        │
JwtAuthenticationFilter extends OncePerRequestFilter
        │  → Extract Bearer token
        │  → Validate signature (RSA public key)
        │  → Check expiry
        │  → Extract roles from claims
        │  → Build UsernamePasswordAuthenticationToken(principal, null, authorities)
        │  → SecurityContextHolder.setAuthentication(...)
        │
Spring Security FilterChain
        │  → .requestMatchers("/labs/**").hasRole("LAB_TECH")
        │  → .requestMatchers("/admin/**").hasRole("ADMIN")
        │
LabOrderController
        @PreAuthorize("hasRole('LAB_TECH') or hasRole('ADMIN')")
        public LabOrder getOrder(@PathVariable String id) { ... }
```

**Key design decisions:**

**RS256 over HS256:**
HS256 is symmetric — every service that validates tokens needs the signing secret. If any service is compromised, the signing key is exposed. RS256 is asymmetric — I sign with a private key, every service validates with the public key. Validation-only services never hold signing capability.

**Method-level security as defense-in-depth:**
`@PreAuthorize` at the service or controller method provides a second gate. If a route misconfiguration in `SecurityFilterChain` accidentally allows a request through, the method annotation still rejects it. Two independent checks means a single misconfiguration isn't catastrophic.

**Short expiry + refresh token rotation:**
Access tokens expire in 1 hour. Refresh tokens expire in 7 days but are single-use — each use issues a new refresh token and invalidates the old one (stored in DB). This limits the blast radius of a stolen access token (max 1 hour exposure) while keeping UX smooth.

**No PHI in JWT payload:**
JWT payloads are base64-encoded, not encrypted. Anyone with the token can decode the payload (though not forge it). We put only roles and the user ID in the payload — never patient data, diagnoses, or any PHI. Application code fetches sensitive data using the user ID.

**Trade-offs:**
- Stateless JWT means no server-side session to invalidate. A token issued before a role change is valid until expiry. Short expiry windows are the mitigation — we accepted this trade-off to avoid maintaining a server-side token blacklist (which reintroduces statefulness and a cache invalidation problem).
- `@PreAuthorize` with roles is coarse-grained. For per-resource ACLs ("Dr. Smith can only view their own patients"), you'd need a more complex attribute-based access control (ABAC) model. We didn't need that granularity, so we didn't build it.

---

### Q8. How does Kafka consumer reliability work with Dead Letter Topics and exponential backoff? Walk through the configuration you built.

**Why they're asking:** Anyone can write a Kafka consumer. Building a reliable one that doesn't lose messages or block the partition under failure is a different skill.

**Answer:**

**The problem:**
A Kafka consumer is processing a `patient.created` event. The Insurance service's database is briefly unavailable. What should happen?

- **Crash and restart?** → Kafka will re-deliver from the last committed offset, but you've lost partition progress and may reprocess hundreds of messages.
- **Block indefinitely?** → The partition queue backs up. Consumer lag grows. Downstream is effectively unavailable.
- **Retry with backoff, then park the message safely?** ✓ This is the correct approach.

**Configuration:**

```java
@Bean
public DefaultErrorHandler kafkaErrorHandler(
        KafkaTemplate<String, Object> kafkaTemplate) {
    
    // Where failed messages go after all retries are exhausted
    DeadLetterPublishingRecoverer recoverer =
        new DeadLetterPublishingRecoverer(kafkaTemplate,
            (record, ex) -> new TopicPartitionOffset(
                record.topic() + ".DLT",  // patient.created.DLT
                record.partition()
            ));

    // 5 attempts: 1s → 2s → 4s → 8s → 16s (capped at 30s)
    ExponentialBackOffWithMaxRetries backOff =
        new ExponentialBackOffWithMaxRetries(5);
    backOff.setInitialInterval(1_000L);
    backOff.setMultiplier(2.0);
    backOff.setMaxInterval(30_000L);

    DefaultErrorHandler handler = new DefaultErrorHandler(recoverer, backOff);

    // Don't retry business logic failures — go straight to DLT
    handler.addNotRetryableExceptions(
        PatientAlreadyExistsException.class,
        ValidationException.class
    );
    
    return handler;
}
```

**Retry sequence:**
```
Attempt 1: immediate → fails (DB unavailable)
Attempt 2: +1s      → fails
Attempt 3: +2s      → fails
Attempt 4: +4s      → fails
Attempt 5: +8s      → fails
→ Publish to patient.created.DLT (with failure metadata in Kafka headers)
→ Commit offset → move on to the next message
```

**Why the partition doesn't block:**
The DefaultErrorHandler retries the message in-place (doesn't commit the offset until retries are exhausted), then publishes to DLT and commits. This means the partition continues making progress regardless of individual message failures.

**DLT metadata:**
Spring's `DeadLetterPublishingRecoverer` automatically adds failure context as Kafka headers:
- Exception class + message
- Original topic, partition, offset
- Stack trace (first 2KB)

This makes debugging DLT entries trivial — you see exactly what failed and where without hunting through logs.

**DLT monitoring:**
CloudWatch alarm on DLT consumer group lag > 0. If any message hits the DLT, on-call is paged. DLT messages are not silently discarded — they're either replayed (after fixing the root cause) or manually acknowledged with explanation.

**Distinguishing transient vs permanent failures:**
This is critical. A database being temporarily unavailable is transient — retry makes sense. A message with an invalid patient ID (business rule violation) will never succeed regardless of retries. `addNotRetryableExceptions` routes permanent failures directly to DLT without burning retry budget. Without this distinction, permanent failures chew through your backoff sequence and delay other messages unnecessarily.

---

### Q9. You architected a shared microservice design library used by 10+ teams. How do you version, distribute, and govern a shared platform library without it becoming a source of cross-team friction?

**Why they're asking:** This is a senior engineer question — it tests whether you can think about organizational dynamics, not just code quality.

**Answer:**

The naive approach is: one team owns the library, all changes go through them, all teams must upgrade together. This creates a chokepoint. Teams block each other on upgrades. The owning team becomes a bottleneck. Teams fork the library to avoid the upgrade tax. You end up with N forks, which is worse than the original problem.

**The model I used:**

**SemVer with a two-major-version support window:**
- `PATCH` (1.0.0 → 1.0.1): Bug fixes. Backward compatible. Teams auto-upgrade via range dependency.
- `MINOR` (1.0.0 → 1.1.0): New capabilities. Backward compatible. Teams opt-in on their schedule.
- `MAJOR` (1.0.0 → 2.0.0): Breaking changes. Full migration guide required. v1 maintained for 6 months after v2 release.

Teams pin to `~1.x` — they get patches and minor upgrades automatically, but major upgrades are deliberate. No team is forced to upgrade on your timeline.

**Distribution:**
Internal Nexus/Artifactory repository. The library is versioned, published, and consumed like any other dependency. Not a git submodule (avoid coupling repos), not a shared folder (avoid coupling deployments).

**Governance — contribution from any team:**
The Technical Pillar owns the vision and final approval, but any engineer in the org can submit a PR. We maintained a `CONTRIBUTING.md` with:
- Design principles the library follows (what goes in vs what stays in the service)
- How to write a proposal for a new addition (prevents wasted PR effort)
- Review SLA: 2 business days
- What automated checks are required (tests, Javadoc, changelog entry)

**What goes in vs what stays out:**
- IN: Cross-cutting concerns applicable to every service (logging, exception hierarchy, resilience primitives, AWS client configuration, health check contracts)
- OUT: Business logic, service-specific utilities, anything that only 2-3 teams need (those go in a separate team-scoped library)

**The test that matters:**
Could a brand-new engineer join any team, clone their service, add the library, and get correct behavior without reading the library source? If yes, the library is well-designed. If they need to understand internal implementation details to use it correctly, it's leaking complexity and will be misused.

---

## Section 3: Cloud & AWS

---

### Q10. You've used AWS CDK, CloudFormation, and Terraform across your career. How do you decide which to use for a new project?

**Why they're asking:** IaC tool proliferation is a real problem at many companies. They want to see principled decision-making, not tool preference masquerading as strategy.

**Answer:**

These three tools exist on a spectrum from raw AWS-native to provider-agnostic programmatic, and the right choice depends on the infrastructure's scope, the team's existing expertise, and the composition requirements.

**CloudFormation:**
Native AWS. JSON/YAML templates. The trade-off is verbosity — a non-trivial stack (Lambda + Step Functions + IAM + SQS + alarms) becomes hundreds of lines of YAML that's hard to modularize. Nested stacks help but add complexity. I use CloudFormation when:
- The team already has a large CFN investment and migration cost isn't justified
- The infrastructure is simple, static, and won't grow significantly
- You need CFN-native features like StackSets for multi-account deployment

**AWS CDK:**
A programmatic abstraction over CloudFormation. You write infrastructure in TypeScript/Python/Java, CDK synthesizes CFN templates at deploy time. The real value:
- Type safety: a misconfigured IAM policy is a compile error, not a deployment failure
- Constructs as reusable components (think: npm packages for infrastructure)
- Loops, conditionals, and functions that YAML can't do elegantly

At Principal, we adopted CDK for all new serverless infrastructure. A `PolicyProcessingStack` class in TypeScript, parameterized by environment (dev/staging/prod), could deploy the complete Lambda + Step Functions + SQS + CloudWatch topology with one command. New environments went from half a day of YAML wrangling to 5 minutes.

I reach for CDK when:
- The team writes TypeScript or Python
- The infrastructure is primarily AWS services
- You want reusable, composable infrastructure components

**Terraform:**
Provider-agnostic HCL. The killer feature is `terraform plan` — shows you the exact diff of what will change before it changes. The ecosystem is massive (AWS, Kubernetes, Helm, DNS, monitoring). State management (remote state in S3 + DynamoDB locking) is mature.

I reach for Terraform when:
- Infrastructure spans AWS + Kubernetes + on-premise or multi-cloud
- The team already has Terraform expertise
- You need the `plan` safety net for high-stakes infrastructure (production EKS clusters, networking)
- At Principal (Senior role), we used Terraform for EKS cluster management because the Kubernetes provider for Terraform was more mature than CDK's K8s support

**What I avoid:**
Mixing CDK and Terraform for the same resource. State drift between the two tools creates incidents. Pick a primary tool per boundary (CDK for application-level AWS resources, Terraform for platform-level infrastructure) and enforce the boundary.

**Drift detection:**
Both CDK (`cdk diff`) and Terraform (`terraform plan`) detect drift from the defined state. We ran these weekly in CI against production to catch manual console changes — a common source of "works on my machine" infrastructure bugs.

---

### Q11. Design an observability strategy for a distributed system — specifically, what did you do at Principal to reduce Mean Time to Detection (MTTD)?

**Why they're asking:** Any engineer can add some logging. Building observability that makes incidents faster to detect and diagnose is a distinct engineering skill.

**Answer:**

Before we had proper observability, the investigation workflow looked like this: customer reports a policy hasn't been processed → engineer receives alert → spends 20 minutes grep-ing CloudWatch log groups across 4 Lambda functions, looking for the policy ID → finds the error in Lambda C, but now needs to understand what Lambda A and B did → another 20 minutes reconstructing the execution. Total time to detection: 40+ minutes.

The fix wasn't just "add more logs." It was restructuring how we emit, store, and query observability data.

**The four signals (Google SRE framework):**
1. **Latency:** p50, p95, p99 — never average (average hides tail latency problems)
2. **Traffic:** events/second, policies/hour (know your baseline, detect anomalies)
3. **Errors:** Lambda error rate, Step Functions failure rate, DynamoDB throttle events
4. **Saturation:** Lambda concurrent execution utilization, DB2 connection pool depth

**What we built:**

**Correlation IDs end-to-end:**
Every event entering the system received a UUID correlation ID, set as an HTTP header and included in every SQS message attribute. Each Lambda extracted this ID into MDC (Mapped Diagnostic Context) at invocation start. Every log line emitted by that invocation automatically included the correlation ID.

Result: `filter @logStream, @message | filter correlationId = "abc-123"` in CloudWatch Insights returned the complete execution trace across all Lambdas in one query.

**Structured logging (JSON):**
Before: `logger.info("Processing policy " + policyId + " for user " + userId)`
After: `log.info("event=policy_processing policyId={} userId={} duration_ms={}", policyId, userId, elapsed)`

JSON-structured logs are parseable by CloudWatch Insights with field-level queries. You can aggregate `avg(duration_ms)` across a time window, not just search for strings.

**Dynatrace distributed tracing:**
Dynatrace auto-instrumented Lambda invocations and Step Functions executions, producing a service map with latency at each hop: API Gateway → Lambda A → Step Functions → Lambda B → DB2. A p99 spike in the system was immediately attributable to a specific service segment, not just "the system is slow."

**Alert on SLOs, not just errors:**
Alerting when `errorRate > 0` generates alert fatigue — minor transient errors page on-call unnecessarily. We defined SLOs (99% of policy workflows complete within 5 minutes, p99 API latency < 2 seconds) and alerted on SLO budget burn rate. An alert fires when the error rate is high enough to exhaust the error budget within 1 hour, not on every individual error.

**Trade-offs:**
- Dynatrace is expensive. For teams with tighter budgets, AWS X-Ray + CloudWatch Insights achieves 80% of the observability benefit at a fraction of the cost.
- 100% trace sampling for business-critical workflows vs. 10% sampling for high-volume commodity events — sampling reduces cost but means some incidents leave no trace. We classified workflows by criticality and sampled accordingly.

---

### Q12. When would you choose serverless (Lambda) over containerized workloads (Kubernetes/EKS)? How did you make this decision at Principal?

**Why they're asking:** Senior engineers are expected to own infrastructure decisions with clear rationale, not just follow whatever the current trend is.

**Answer:**

**Decision framework:**

| Factor | Lambda (Serverless) | Kubernetes (EKS) |
|--------|---------------------|------------------|
| Traffic pattern | Spiky, unpredictable | Steady, predictable |
| Execution duration | < 15 minutes | Long-running or indefinite |
| Cold start tolerance | Acceptable (>1s OK) | Intolerable (< 100ms required) |
| Operational complexity | Low (no cluster management) | High (nodes, networking, upgrades) |
| Cost model | Pay per invocation | Pay for provisioned capacity |
| State | Stateless | Can be stateful (PVCs) |
| Customization | Limited (AWS runtimes) | Full (custom images, OS) |

**At Principal — where we chose Lambda:**
Policy processing workflows were event-triggered (a policy document arriving triggers processing), had unpredictable traffic (spikes when large batches of policies arrived), and were complete in under 5 minutes. Lambda was a natural fit: no idle cost, automatic horizontal scaling, and no cluster management overhead. A team of backend engineers without deep Kubernetes expertise could operate it confidently.

**At Principal (Senior) — where we chose Kubernetes (EKS):**
Services that needed long-running processes (polling an external API every 30 seconds indefinitely), required specific runtime environments (custom JVM flags, specific glibc versions), or needed guaranteed low latency (provisioned concurrency costs can exceed a small EKS node for high-traffic services). Also, services with persistent sidecar requirements (e.g., a service mesh proxy) don't fit Lambda's model.

**The hybrid model:**
SQS queues buffer incoming events, Lambda handles event-triggered processing, and EKS hosts long-running services that manage state or require constant presence. The boundaries are infrastructure-level — business logic stays in pure Java independent of the runtime.

**What I tell teams when they're choosing:**
If you don't have ops expertise and your workloads are event-driven and short-lived — Lambda. If you're building something that needs to run continuously, needs custom runtime environment, or your team already has Kubernetes experience — EKS. Never choose Kubernetes because it seems "more serious." Operational complexity is a cost you pay forever.

---

## Section 4: Distributed Systems & Data

---

### Q13. How do you handle data consistency across microservices without distributed transactions?

**Why they're asking:** This is a core distributed systems question. They want to see if you understand the CAP theorem implications and know real patterns, not just textbook definitions.

**Answer:**

The fundamental problem with microservices and consistency: each service owns its database. Cross-service ACID transactions require 2-Phase Commit (2PC), which means:
- A coordinator holds locks across all participants until all agree
- If the coordinator fails, the system is in an uncertain state (blocking protocol)
- Throughput degrades under coordination overhead

The practical answer is: accept eventual consistency by design, and build the guarantees you actually need at the application layer.

**Patterns I've used:**

**1. Transactional Outbox (Lab Service in MediTrack):**
Atomic DB write + outbox event write in one transaction. Relay publishes to Kafka. Consumer uses idempotency keys to handle at-least-once delivery. Guarantees: event is eventually delivered iff the DB transaction commits. Gap: ~1s relay polling latency.

**2. Saga Pattern (for multi-step workflows with compensation):**
Each service executes its step and publishes an event. If a later step fails, a compensating transaction is published for earlier steps to undo.

```
Step 1: Reserve policy slot → publishes PolicyReserved
Step 2: Charge premium → if fails → publishes PaymentFailed
PolicyReserved listener → releases slot on PaymentFailed
```

No global rollback — each service implements its own compensating logic. The system eventually reaches a consistent state, but may be temporarily inconsistent.

**3. Read-your-own-writes via event sourcing:**
If a user creates a patient and immediately reads their patient record, they might miss their own write if the read hits a replica with replication lag. We solved this with version tokens — the create response included a version number, and reads included that version as a "consistency marker," retrying on replicas until their version matched.

**4. Data projection / local cache:**
The Insurance Service in MediTrack needs patient name for display but doesn't need real-time accuracy. It subscribes to `patient.updated` events and maintains a local projection. If the projection is 5 minutes stale — acceptable. If patient payment info is stale — not acceptable, that would require synchronous lookup. Know your staleness tolerance per data class.

**The question I always anchor decisions to:**
"What is the acceptable consistency window for this specific data?" If it's "user must see their own write immediately" — you need synchronous coordination or routing. If it's "analytics can be 5 minutes stale" — async events with eventual consistency is fine. One-size-fits-all consistency requirements usually mean someone hasn't thought carefully about the actual user experience requirement.

---

### Q14. When would you choose DynamoDB over PostgreSQL, and vice versa? Walk me through a real decision you made.

**Why they're asking:** Database selection is a load-bearing architectural decision. Poor choices are expensive to correct years later.

**Answer:**

**The decisive question isn't "which database is better." It's "what are my access patterns, and are they likely to change?"**

DynamoDB forces you to design around your access patterns upfront — your key schema is your query capability. If your access patterns are clear and stable, DynamoDB is exceptional. If they're exploratory or evolving, you'll regret a DynamoDB choice.

**DynamoDB is the right choice when:**
- Primary access is by a known key (get policy by policyId, get user by userId)
- Throughput requirements are high and traffic is unpredictable (DynamoDB auto-scales storage and throughput with no pre-provisioning)
- Single-digit millisecond latency is required at any scale
- Operational simplicity matters (no schema migrations, no connection pool management, no cluster upgrades)

**Real example — Diligence System (intern project):**
The SBD workflow tracker needed to look up a workflow by ID, update its status, and list all workflows by status. Access patterns were fixed and simple. DynamoDB was the obvious choice — zero ops burden, scales to any volume, and the single-table design with a GSI on `status` covered all queries. PostgreSQL would have required provisioning an RDS instance, managing schema, and handling connection limits — all overhead for a workflow tracker.

**PostgreSQL is the right choice when:**
- You need joins, aggregations, or ad-hoc queries during development
- Multiple attributes form complex filter conditions (`WHERE status = 'active' AND premium > 1000 AND region = 'IE'`)
- Strong ACID guarantees across multiple rows are required (e.g., when "complete the order AND create the result record" must be atomic)
- The data model will evolve significantly during development (Flyway migrations on PostgreSQL are mature)

**Real example — MediTrack Lab Service:**
Lab orders have relationships with patients, results with orders, critical flags with results. Report generation needed joins across tables. During development, the schema evolved 8 times in 2 weeks. PostgreSQL with Flyway migrations handled this without pain. Attempting the same in DynamoDB would have required re-designing the single-table schema multiple times — the friction would have slowed development significantly.

**The anti-pattern I actively avoid:**
Using DynamoDB for everything because it's "AWS-native" or "NoSQL is modern." Single-table DynamoDB design is a real engineering skill. Getting the key schema wrong means a migration that requires exporting your entire table, transforming, and re-importing. I've seen teams spend weeks undoing this. If access patterns aren't clear by the time you're building, use PostgreSQL and migrate to DynamoDB later if the access patterns solidify and throughput demands it.

---

## Section 5: Debugging & Failure Handling

---

### Q15. Walk me through how you discovered and fixed the PHI log-leak in MediTrack. What systemic changes did you put in place?

**Why they're asking:** Security incidents in production are far more expensive than security-conscious engineering practices. They want to see if you think defensively.

**Answer:**

**Discovery:**
During local debugging with detailed logging enabled, I noticed that Spring Data JPA's Hibernate emits parameter binding logs at `TRACE` level via `org.hibernate.type.descriptor.sql.BasicBinder`. With `TRACE` enabled for that package (common during debugging), the logs showed:

```
TRACE BasicBinder - binding parameter [1] as [VARCHAR] - [John Smith]
TRACE BasicBinder - binding parameter [2] as [VARCHAR] - [123-45-6789]   ← SSN
TRACE BasicBinder - binding parameter [3] as [VARCHAR] - [HIV-positive]  ← Diagnosis
```

This is Hibernate printing every bound SQL parameter value — which includes every field being persisted or queried. For a healthcare application, this is a critical PHI exposure: if `TRACE` logging is ever enabled in a production environment (common during incident investigation), every SSN and diagnosis written to the database would appear in plaintext in log aggregation systems.

**Immediate fix:**
```properties
logging.level.org.hibernate.type.descriptor.sql=WARN
logging.level.org.hibernate.SQL=WARN
```

This stops parameter binding from appearing in logs at any normal operational log level.

**Root cause fix — structured audit logging:**
The underlying need (tracing data flow for debugging) was legitimate. The approach (dumping raw SQL parameters) was wrong. I replaced ad-hoc debug logging with structured audit events:

```java
log.info("patient_registered id={} name=REDACTED ssn=***{}", 
    patient.getId(), 
    patient.getSsn().substring(7));  // last 4 digits only
```

This gives you what you need for debugging (confirm the record was created, which record) without exposing the sensitive fields.

**Input validation at the boundary:**
Added `@Valid` with constraint annotations to all request DTOs, so malformed PHI is rejected at the HTTP layer before reaching JPA:

```java
public class PatientRequest {
    @NotNull @Pattern(regexp = "\\d{3}-\\d{2}-\\d{4}")
    private String ssn;
    
    @NotNull @Size(min = 1, max = 100)
    private String firstName;
}
```

**Systemic prevention — CI enforcement:**
Added a GitHub Actions check that scans `application.properties` and `application.yml` for `TRACE` or `DEBUG` log configurations on the Hibernate parameter binder package. If this configuration is ever re-introduced, the build fails before it's merged. Defense in depth means the fix survives team turnover.

**Why this matters beyond MediTrack:**
This is one of the most common PHI/PII exposure vectors in JVM applications. It's not malicious — it's an engineer enabling verbose logging during a debugging session and forgetting to revert it. The fix is not just a configuration change; it's encoding the rule into automated enforcement so it doesn't depend on every engineer remembering.

---

### Q16. Walk me through how you'd debug a production Lambda that works in development but intermittently times out under real load.

**Why they're asking:** Production debugging requires methodical thinking under pressure. They want to see your diagnostic process, not just the answer.

**Answer:**

**Step 1: Characterize before guessing.**
I don't touch the code until I understand the pattern. Pull CloudWatch metrics for the Lambda:
- Is the timeout happening at a specific time of day? → Likely load-correlated
- Is `ConcurrentExecutions` near the reserved limit? → Throttling
- Is `INIT_DURATION` high on failing invocations? → Cold start issue
- Does the timeout correlate with downstream service deploys? → Cascading failure

**Step 2: Identify cold start signatures.**
CloudWatch Logs includes `INIT_DURATION` in the REPORT line for cold-start invocations. For JVM Lambdas (Spring Boot), cold starts can be 5-10 seconds. If the function timeout is 15 seconds and cold start eats 10, you have 5 seconds for actual work — fine under normal conditions but not under load when spikes cause many simultaneous cold starts.

Fix options: provisioned concurrency (warm instances, eliminates cold starts, adds cost), or GraalVM native compilation (eliminates JVM startup overhead but requires build pipeline changes).

**Step 3: Add timing instrumentation to the hot path.**
Without touching the timeout value:

```java
public void handleRequest(SQSEvent event) {
    long start = System.currentTimeMillis();
    
    PolicyData policy = parseEvent(event);
    log.info("event=parse_complete duration_ms={}", elapsed(start));
    
    PolicyData enriched = db2Client.enrich(policy);
    log.info("event=db2_enrich_complete duration_ms={}", elapsed(start));
    
    dynamoDb.putItem(enriched);
    log.info("event=dynamo_write_complete duration_ms={}", elapsed(start));
}
```

Now CloudWatch Insights shows exactly which step is slow, rather than "the Lambda timed out" with no further context.

**Step 4: Check external call timeout configuration.**
Lambda timeout is 30 seconds. Is the HTTP client to DB2 configured with a 35-second socket timeout? It will wait past the Lambda deadline, then the Lambda times out with a cryptic error. External call timeouts must always be less than Lambda timeout, with meaningful budget remaining.

```java
// Bad: DB2 client with 60-second socket timeout inside a 30-second Lambda
// Good:
HttpClientConfig config = HttpClientConfig.builder()
    .connectTimeout(Duration.ofSeconds(5))
    .socketTimeout(Duration.ofSeconds(20))  // < Lambda timeout
    .build();
```

**Step 5: Check for database connection exhaustion.**
Lambda scales horizontally — 100 concurrent Lambda invocations each attempting to open a PostgreSQL connection = 100 simultaneous connections. RDS has connection limits (usually 100-200 per instance). New connections queue until timeout.

Fix: RDS Proxy. It multiplexes connections — 100 Lambda instances share a pool of 20-30 actual DB connections. The proxy handles the queuing, not the Lambda timeout clock.

**Step 6: AWS X-Ray if all else fails.**
Enable X-Ray tracing on the Lambda. It produces a service map showing latency at each segment: Lambda initialization → DB2 call → DynamoDB write. The waterfall view immediately shows which segment is consuming the timeout budget.

---

## Section 6: Trade-offs & Decision-Making

---

### Q17. You influenced multiple product teams to migrate to AWS serverless without formal authority. How do you drive technical adoption across teams you don't manage?

**Why they're asking:** This is a tech lead / staff engineer question — technical influence without hierarchy is the key leverage mechanism at scale.

**Answer:**

The instinct is to send a "you should migrate" email. That fails. Teams have their own backlogs, their own technical debt, their own risk tolerances. Mandating change without addressing their actual concerns is how you get grudging compliance and resentment, not genuine adoption.

**The model I used:**

**1. Demonstrate, don't advocate.**
I migrated my own team's services first. When we reduced infrastructure costs, decreased deploy time from 45 minutes to 8 minutes, and eliminated three on-call pages related to provisioning — those outcomes were visible and credible. Other teams asked questions. I gave them access to our implementation as a reference.

**2. Technical presentations with ROI framing.**
The 150+ engineer BU-wide presentation wasn't "CDK is great, here's how it works." It was: "Here's the workflow we had, here's what changed, here are the numbers: cost down X%, deploy time down Y%, on-call incidents down Z." Engineers care about making their systems better. Managers care about cost and reliability. Both audiences got what they needed from the same evidence.

**3. Lower the activation energy.**
The shared microservice library (Technical Pillar work) and internal CDK starter templates meant "migrating to serverless" no longer required understanding Lambda, CDK, and IAM from scratch. Teams could bootstrap a compliant service in a day using our templates. Reducing the upfront cost of adoption from weeks to days changed the calculus for backlog prioritization.

**4. Be available, not prescriptive.**
I explicitly offered architecture review time to any team considering migration. Not "here's how you should do it" — "tell me your constraints and let's figure out the right approach." Teams that felt heard adopted faster than teams that felt pushed.

**5. Let early adopters do the selling.**
After 2-3 teams successfully migrated, I connected them with teams that were considering it. Peer-to-peer "here's what we learned, here's the gotcha we hit" is more persuasive than anything I could say. Social proof within an organization is underrated.

**What I learned about why teams resist adoption:**
The most common blocker was not technical skepticism — it was "we have existing systems that work, migration risk is real, and we won't get credit for a migration that goes smoothly (only for one that fails)." Framing migrations as risk reduction (fewer on-call pages, lower infrastructure exposure) rather than technical improvement addressed the actual concern.

---

### Q18. You built FocusFlow's heat-based task prioritization algorithm from scratch. Walk me through the design decisions.

**Why they're asking:** Algorithmic problem-solving and product engineering thinking — can you design and justify a non-trivial algorithm?

**Answer:**

**The problem:**
Standard task managers rank by due date or user-assigned priority. Both have failure modes: due date ranking punishes tasks with no deadline (they never surface), and user-assigned priority requires the user to constantly re-rank — which is cognitive work that defeats the purpose of a task manager.

**What "heat" captures:**
Heat is a composite signal designed to surface what actually deserves attention right now, dynamically:

```
heat = (urgency_score × w1) + (importance_score × w2) + (time_decay × w3)
```

**Urgency score:**
Derived from time-to-deadline. Not linear — a task due in 2 hours is not twice as urgent as one due in 4 hours, it's exponentially more urgent. I used an inverse exponential:

```
urgency = 1 / (1 + e^(-k × (deadline_proximity - threshold)))
```

Where `deadline_proximity` is hours until due and `k` controls the steepness of the urgency curve. This produces a sigmoid: tasks far from their deadline have near-zero urgency; tasks within hours have near-maximum urgency; the transition is sharp and feels correct to users.

**Importance score:**
User-defined (1-5), but normalized and weighted against the user's historical importance distribution. If a user marks everything as "5 important," the signal degrades. I applied z-score normalization per user so importance is relative to their own patterns.

**Time decay:**
Tasks that have sat in the backlog without progress slowly accumulate heat — ensuring they don't get permanently buried. This is a slow linear accumulation (not exponential, to prevent backlog items from overtaking genuinely urgent tasks).

**"Big Three" surfacing:**
The algorithm computes heat for every task and surfaces the top 3 at the start of each session. The constraint: the Big Three must include at least one task from each importance band (if tasks exist in all bands), preventing the algorithm from filling all three slots with low-importance tasks that happen to be almost overdue.

**Design trade-offs:**
- The weights (`w1`, `w2`, `w3`) are tunable. I exposed them as configuration rather than hardcoding, so the algorithm can be calibrated based on user behavior data.
- Purely algorithmic prioritization without user override is condescending. The Big Three is a suggestion, not a lock. Users can pin tasks above the algorithm's output.
- No ML model — the algorithm is deterministic and inspectable. A user can understand why a task surfaced in their Big Three. Black-box prioritization erodes user trust.

---

### Q19. You improved team availability from 97.5% to 99.2%. How did you measure that, and what specifically drove the improvement?

**Why they're asking:** Availability numbers without explanation are marketing. They want to understand if you know what actually drove the change.

**Answer:**

**How we measured it:**
Availability = (total minutes in period - minutes with degraded service) / total minutes × 100

We tracked this via a Dynatrace SLO configured on the API endpoint response code distribution. We defined "degraded" as error rate > 5% for more than 3 consecutive minutes. This threshold was deliberately calibrated — transient single-request failures don't count as downtime, but sustained error rates do.

The 97.5% baseline meant approximately 5.5 hours of degraded service per month. At 99.2%, that dropped to approximately 3.5 hours — a 36% reduction in downtime.

**Root causes of the 2.5% downtime (pre-migration):**

**1. XML parsing under concurrent load:**
SOAP uses XML. Our SOAP service was synchronous and CPU-bound on XML deserialization. Under load spikes (policy batch processing arriving in bulk), CPU saturated, threads queued, requests timed out. The service didn't have auto-scaling — it was provisioned for average load, not peak.

**2. Session state coupling:**
The SOAP service maintained in-memory session state. Multiple downstream consumers sharing a session context caused race conditions under concurrent access — hard-to-reproduce failures that appeared as intermittent errors in monitoring.

**3. Deployment downtime:**
Updating the SOAP service required a rolling restart of the application server. During restart windows (typically 2-3 per month for patches and deployments), there was unavoidable downtime.

**What the REST + Lambda migration fixed:**

1. **Stateless by construction:** Lambda functions have no shared memory state between invocations. The session coupling bugs became structurally impossible.

2. **Auto-scaling:** Lambda scales to hundreds of concurrent invocations in seconds. CPU saturation under batch processing load ceased to be a factor.

3. **Zero-downtime deployments:** Lambda deployments are atomic version switches with traffic shifting. No restart window, no in-flight request interruption.

4. **JSON over XML:** Roughly 40% payload size reduction, proportionally less deserialization overhead per request.

The improvement wasn't magic — it was specific structural changes that eliminated specific failure modes. When an interviewer asks about availability improvements, they should always be able to trace the number back to causal mechanisms.

---

*End of Technical Interview Preparation*

---

> **Revision note:** Replace any `[FILL:]` placeholders with real numbers where you have them. Every metric you can verify is worth including. Every metric you're unsure about should be presented as "approximately" or "estimated."
