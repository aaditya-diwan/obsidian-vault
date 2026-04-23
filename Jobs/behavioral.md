# Behavioral Interview Preparation
### Aaditya Diwan — Senior Backend Engineer

> Derived from 4+ years at Principal Global Services (Intern → Senior SE) + MediTrack, FocusFlow, and leadership experience.
> Every answer is in STAR format: **Situation → Task → Action → Result**
> Written at the level of a Senior SWE / Tech Lead candidate at a top-tier product company.

---

## Section 1: Leadership & Ownership

---

### Q1. Tell me about a time you owned a complex project completely end-to-end, from requirements through production.

**Situation:**
During my internship at Principal Global Services, the engineering team managed a category of internal workflows called SBD processes entirely through manual effort — tracking statuses in spreadsheets, generating reports by hand, and following up on incomplete processes via email chains. The Head of Engineering mentioned the overhead in passing, but there was no formal project assigned to address it.

**Task:**
I took it upon myself to scope, design, build, and deploy a solution. There was no senior engineer assigned to mentor me through the technical decisions — I was expected to figure it out and deliver something useful.

**Action:**
I started with stakeholder discovery, not code. I interviewed workflow owners to understand the actual pain points rather than assuming I knew what they needed. The key insight from those conversations: the biggest cost wasn't time to complete individual workflows — it was time lost to *finding* which workflows were stuck and *knowing* when follow-up was required.

From that, I designed "Diligence" — a serverless automation system. Key decisions I made independently:
- **AWS Lambda + DynamoDB** over a server-based stack: I had no operational support as an intern, so zero-maintenance infrastructure was essential. Lambda eliminated server provisioning entirely. DynamoDB's access patterns (get workflow by ID, list by status) were simple and well-suited to a key-value model.
- **TypeScript** for the Lambda functions: type safety reduced runtime errors in a project with no QA process.
- I wrote the system design documentation before writing a line of code, circulated it for feedback, and iterated based on stakeholder input before committing to the implementation.

I handled requirement gathering, system design, implementation, testing, deployment, and operational documentation. I ran the final demo for the Head of Engineering.

**Result:**
Diligence reduced manual SBD workflow effort by approximately 150 hours per month across the team. The Head of Engineering recognized the project specifically for driving measurable improvements in process efficiency. More consequentially for my career: it established a pattern I've followed ever since — start from the user's real problem, design before building, and take full ownership of outcomes.

---

### Q2. Tell me about a time you were the technical decision-maker for an architectural choice that had significant consequences.

**Situation:**
As a Software Engineer at Principal, I was leading the modernization of a set of legacy insurance policy workflows that ran on dedicated on-premise servers. These systems had grown organically over years — undocumented, tightly coupled, and with deployment processes that required planned downtime windows. Business stakeholders were frustrated by the 3-4 hour monthly maintenance windows.

**Task:**
I was responsible for proposing the architecture for the modernized system and getting buy-in from engineering leadership. The decision — serverless Lambda + Step Functions versus containerized services on EC2 or EKS — would shape operational complexity for years.

**Action:**
I ran a structured evaluation instead of going with my instinct. I built a scoring matrix against the criteria that actually mattered for this system: traffic pattern (bursty/event-driven), execution duration (< 5 minutes per workflow), state management requirements (none — workflows were stateless), and operational capacity of the team (no dedicated platform team). I also projected infrastructure costs for both options at 3x current volume.

The analysis strongly favored serverless. But I didn't stop at the technical case — I identified the risks upfront:
- Cold start latency for JVM Lambdas would require provisioned concurrency for user-facing flows
- Step Functions Standard Workflow costs would need monitoring at high event volume
- The team would need CDK training before the migration could begin

I proposed the migration in three phases, with go/no-go checkpoints at each phase based on performance benchmarks. I presented to engineering leadership with the trade-off analysis, not just the recommendation.

**Result:**
Leadership approved the phased approach. The migration reduced infrastructure costs (no more idle server provisioning), eliminated the maintenance window downtime (Lambda deployments are atomic), and improved policy processing time by 30%. The CDK training I ran for the team also became the foundation for the org-wide cloud adoption initiative I led afterward.

---

### Q3. Describe a time you served as a Technical Anchor or lead for a team. What did that mean in practice?

**Situation:**
When I was promoted to Senior Software Engineer at Principal in February 2025, I was designated as the team's Technical Anchor — a role defined as bridging high-level architectural design with operational reality, while simultaneously developing the team's engineering capability.

**Task:**
My responsibilities were dual: deliver complex features as an individual contributor AND ensure the team's collective technical quality was rising. In practice, this meant I was accountable for architectural decisions, code review standards, onboarding, and cross-team technical coordination simultaneously.

**Action:**
I approached the role by separating it into two tracks:

*Track 1 — Technical quality:*
I established a PR review standard that went beyond "does this code work." Every PR I reviewed included: at least one comment on edge cases the author hadn't considered, at least one comment on the failure mode of the approach under load, and one comment on testability. I didn't just leave comments — I made time to walk through the reasoning in a follow-up Slack message. Within a month, other engineers started using the same framework in their own reviews.

I also set up a weekly "architecture office hours" — 30 minutes where anyone on the team could bring a design problem, no matter how early-stage. This created a culture where engineers sought review early (when it's cheap) rather than late (when it's expensive).

*Track 2 — Team capability:*
I created structured onboarding documentation for our high-concurrency distributed systems patterns — documenting not just "how" but "why." New engineers who previously spent weeks getting up to speed on system context could get to productive contribution in significantly less time with this documentation.

**Result:**
The three engineers I onboarded and mentored had a 30% faster ramp-up to productive contribution compared to previous onboarding. Two hackathon prototypes I led or co-led were shipped to production — not a common outcome for hackathon work. And the PR review culture shift was measurable: production defect rates decreased, and we went through multiple release cycles with zero production incidents.

---

## Section 2: Conflict Resolution

---

### Q4. Tell me about a time you disagreed with a technical decision and how you handled it.

**Situation:**
Early in my time as a Software Engineer at Principal, the team was designing a new cross-service integration layer. The initial architectural proposal was to use direct synchronous REST calls between two backend services for what was fundamentally an asynchronous workflow — a policy event that triggered downstream processing that could take minutes.

**Task:**
I believed the synchronous coupling was the wrong approach — it would create availability coupling between the services (if one is slow, the caller times out), make it harder to scale each service independently, and create retry complexity at the call site. But I was relatively new to the team and the proposal came from a more senior engineer.

**Action:**
I didn't push back in the group design review — not because I lacked conviction, but because criticizing a colleague's design publicly without a clear alternative isn't constructive. Instead, I drafted a written comparison of the two approaches:
- Sync REST: simpler to implement initially, caller blocks, availability coupling, scaling ceiling limited by slowest service
- Async SQS/Lambda: more infrastructure upfront, decoupled availability, each service scales independently, cleaner retry semantics with DLQ

I documented a specific failure scenario: if the downstream service was under load and took 30 seconds to respond, the caller would time out, and the caller's Lambda would also time out. The failure would cascade upward. With async messaging, the caller deposits the event and returns immediately — the downstream processes at its own pace.

I shared this privately with the senior engineer first, framing it as "I want to make sure I'm not missing something about why sync is preferred here" rather than "you're wrong." We aligned on the async approach within one conversation.

**Result:**
We implemented the SQS-based async integration. When the downstream service did experience a load spike six months later, the upstream system continued operating without interruption. The DLQ caught the queued events during the spike, which were processed when load normalized. Had we used synchronous calls, that spike would have caused a cascading outage to the upstream service as well.

**What I'd do differently:**
Now I'd raise the concern in the group review, but frame it as a question: "I want to understand the availability behavior when downstream is slow — can we walk through that scenario?" That invites collaborative thinking rather than making it a disagreement between two people.

---

### Q5. Describe a time you had to deliver difficult technical feedback to a peer.

**Situation:**
As a Senior SE at Principal, I was reviewing a PR from a peer engineer — an experienced developer who had been at the company longer than me. The PR implemented a new data enrichment Lambda that pulled sensitive PII from DynamoDB for processing. The code was functionally correct, but it was logging the full PII payload at DEBUG level, and the Lambda had no field-level access restrictions — any service with DynamoDB read access could read any record.

**Task:**
This wasn't a stylistic preference — it was a security risk that needed to be addressed before the code was deployed, and I needed to do it in a way that didn't embarrass or alienate a peer I respected.

**Action:**
I didn't leave a comment in the PR that said "this is a security problem." Instead, I scheduled a 15-minute Zoom call and framed it as: "I was reviewing your PR and I came across a pattern that I want to understand better — can we talk through the data access design?"

In the call, I walked through the risk concretely: "If CloudWatch Logs for this Lambda are ever exported to a third-party SIEM or accessed by an auditor, the PII fields will be plaintext in the log stream. From a GDPR audit perspective, that's a data breach." I wasn't citing a rule — I was showing the actual downstream consequence.

I came prepared with a proposed fix: structured logging with masked fields, DEK-based field-level encryption for the PII attributes, and an IAM policy scoped to only the DynamoDB attributes this Lambda legitimately needed.

**Result:**
The engineer received the feedback well — precisely because I treated it as a shared problem to solve rather than a failure to call out. The PR was updated with the security controls before merging. The engineer later became a champion for the PII handling standards I proposed for the team, because they understood the reasoning rather than just following a rule.

**What I learned:**
Peer security feedback works best when you help them see the risk, not just the violation. Engineers who understand why a pattern is dangerous will catch future instances themselves.

---

## Section 3: Decision-Making Under Ambiguity

---

### Q6. Tell me about a time you had to make a significant technical decision with incomplete information or unclear requirements.

**Situation:**
During the SOAP-to-REST migration at Principal, we reached a point where we needed to decide whether to continue running both the legacy SOAP service and the new REST service in parallel (shadow mode) for an additional 3 weeks, or proceed with the cutover on the original timeline.

**Task:**
The business stakeholders wanted the migration complete on schedule — the SOAP service was generating support overhead they were eager to eliminate. But my shadow mode data showed a 0.8% discrepancy rate between SOAP and REST responses for a specific category of edge-case policies. I couldn't fully characterize whether these were actual bugs in the REST service, acceptable differences in rounding/formatting, or test data artifacts.

**Action:**
I didn't resolve this by waiting for perfect information — that's not how production systems work. Instead, I made the decision explicitly:

I classified the 0.8% discrepancy cases by downstream impact. For 90% of them, the difference was formatting — trailing zeros in currency fields, date format variations. These were cosmetic and downstream consumers handled them fine. For the remaining 10% (0.08% of total), there was a substantive difference in calculated premium values.

I presented this analysis to stakeholders: "We can proceed with cutover today for 99.92% of cases. For the 0.08% edge-case premium calculations, I recommend routing those to the legacy SOAP service via a feature flag while we investigate. We can eliminate the flag within two weeks."

This gave stakeholders the timeline they needed while protecting the critical edge cases. I was explicit about what I knew, what I didn't know, and how I was managing the risk.

**Result:**
Stakeholders approved the approach. The cutover proceeded on schedule. The 0.08% edge cases were isolated to the legacy path via a feature flag, and we resolved the root cause (a rounding mode difference between SOAP and REST calculation libraries) within 10 days. The flag was removed shortly after. The migration completed without a production incident.

**The principle I operate by:**
In ambiguous situations, the most valuable thing I can do is be explicit about my confidence level. "I'm 99% sure this is safe to proceed and here's what I'm not sure about" is infinitely more useful than "I need more time to be certain." Certainty is rarely available in production.

---

### Q7. Tell me about a time you had to re-scope or change direction mid-project because your initial assumptions were wrong.

**Situation:**
During the design of the MediTrack insurance service, my initial implementation consumed `patient.created` Kafka events and made a synchronous REST call to the Patient Service during event processing to fetch additional patient details before creating the insurance policy record.

**Task:**
The design looked clean — event-driven trigger, synchronous data fetch, persist. But as I load-tested the consumer under simulated production volume, I realized the design had a hidden availability coupling: if the Patient Service was slow or down, the insurance consumer would back up on Kafka, retries would accumulate, and the consumer lag would grow until the system was effectively unavailable.

**Action:**
I had already invested a week in the initial implementation. Rather than defending the design, I treated the load test discovery as information and re-evaluated the architecture.

The correct approach was to consume the `patient.created` event and project the fields I needed directly from the event payload into the insurance service's own database — eliminating the synchronous call entirely. This required:
1. Ensuring the `patient.created` event contained all the fields the insurance service needed (required a contract discussion with the patient service team)
2. Re-designing the insurance service's data model to include a local patient projection table
3. Adding a consumer for `patient.updated` events to keep the projection current

This added a week to the implementation timeline. I communicated the delay transparently, with the reason: "The synchronous dependency created an availability coupling that would make the system unreliable under load. The extra week buys independent availability." No stakeholder pushed back when the trade-off was explained clearly.

**Result:**
The refactored design performed correctly under load testing — insurance consumer maintained normal lag even when a simulated Patient Service outage was introduced. The data projection pattern also became a deliberate design standard for all cross-service data dependencies in MediTrack.

**What I learned:**
Sunk cost thinking is the enemy of correct engineering decisions. One week of rework discovered early is better than a production architecture flaw discovered at 3 AM.

---

## Section 4: Handling Failures

---

### Q8. Tell me about a time something you built failed in production. How did you handle it?

**Situation:**
During my time as a Software Engineer at Principal, I led the implementation of a new Step Functions-based policy processing workflow. The implementation went through code review, QA, and a staging environment validation before being promoted to production.

Within the first 24 hours of production deployment, we saw a spike in Step Functions `ExecutionsFailed` metrics — approximately 3% of policy executions were failing at the "Enrich from DB2" state. The failures hadn't appeared in staging.

**Task:**
I was the engineer who designed and built this workflow. I needed to: identify the root cause, implement a fix, and ensure no policy events were permanently lost.

**Action:**

*Immediate response (first 30 minutes):*
I activated the Step Functions console and pulled execution history for 10 failed executions. The error messages were consistent: `Read timeout after 30000ms` on the DB2 integration call. Staging didn't reproduce this because it used a smaller, less contended DB2 environment with faster query times.

Critically, I verified that Step Functions was correctly retrying failed executions — our state machine had a `Retry` block configured on this state, so all 3% of failed executions were in retry state, not permanently failed. No policy events were lost.

*Root cause analysis:*
Production DB2 had significantly higher query latency under concurrent load than the staging DB2. The timeout I had configured (30 seconds) was based on staging benchmarks. Production p99 latency at peak was 45 seconds for the enrichment query.

*Fix:*
Two parallel changes:
1. Immediate: Increased the Lambda timeout and HTTP client timeout to 60 seconds. Redeployed Lambda without touching the Step Functions definition (Lambda version aliasing meant no state machine update required).
2. Root cause: The DB2 enrichment query was doing a full table scan for a specific edge-case policy type. I added an index on the query predicate, which reduced p99 latency from 45 seconds to 3 seconds.

*Communication:*
I sent a written incident summary to my manager and the team the same day: what failed, why staging missed it (environment difference), what I changed, and what I was doing to prevent recurrence (load testing against production-representative data volumes going forward).

**Result:**
The failed executions retried successfully within 2 hours of the Lambda timeout increase. The DB2 index was deployed the following morning and reduced enrichment query latency by 15x. No policy events were permanently lost.

**What I changed afterward:**
We added a synthetic load test against a production-scale data snapshot as a mandatory gate before any production deployment. Staging with small data volumes is not a reliable proxy for production behavior under query load.

---

### Q9. Tell me about a time you prevented a failure before it happened — something you caught in review or testing that would have been a production incident.

**Situation:**
While reviewing a PR from a colleague during my Senior SE tenure at Principal, the change added a new Lambda function that needed to read from a DynamoDB table that was shared across multiple services. The function was implemented correctly, but I noticed that the IAM execution role attached to it had `dynamodb:*` permissions on the table ARN — a wildcard that allowed it to read, write, delete, and modify any record.

**Task:**
The code itself was correct. The test coverage was reasonable. But the IAM permissions were wildly overprivileged for a Lambda whose sole purpose was to read a specific set of records.

**Action:**
I added a PR comment explaining the risk: "This Lambda only needs to read policy status records. The `dynamodb:*` wildcard grants it write and delete permissions on every record in the shared table. If this Lambda has a bug, or if its execution role is accidentally used by another resource, it can corrupt or delete data it should never touch."

I included a corrected IAM policy scoped to:
- `dynamodb:GetItem` and `dynamodb:Query` only (no write permissions)
- The specific table ARN
- A Condition block restricting attribute access to only the fields the Lambda legitimately needed (using `dynamodb:Attributes` condition key)

I also flagged that wildcard permissions on shared tables should trigger a security review comment in our PR template — a process gap I volunteered to address.

**Result:**
The PR was updated with the least-privilege IAM policy before merging. I worked with the team to add "check IAM permission scope" as an explicit item on our PR review checklist, and updated our CDK Lambda construct to default to an empty execution role with permissions added explicitly. This made the default behavior secure: you opt into permissions rather than inheriting wildcards.

---

## Section 5: Mentorship & Team Impact

---

### Q10. Tell me about a time you mentored engineers and the impact it had.

**Situation:**
When I was promoted to Senior Software Engineer at Principal, three engineers joined the team within two months of each other — one junior, two mid-level. All three needed to get productive on a high-concurrency distributed system involving Step Functions, Lambda, DynamoDB, and downstream IBM DB2 integrations. The previous onboarding approach was "read the code and ask questions."

**Task:**
Reducing ramp-up time was a direct part of my responsibilities as Technical Anchor. But I also had feature delivery commitments simultaneously — I couldn't spend all my time in 1:1s without my own work slipping.

**Action:**
I approached mentoring as a systems problem, not a scheduling problem.

*Asynchronous onboarding documentation:*
I wrote comprehensive onboarding documentation covering not just "what the system does" but "why the system is designed this way." Each major design decision (why Step Functions, why DynamoDB over RDS, why this retry configuration) was documented with the trade-off reasoning. Engineers who understand the "why" don't need to ask me every time they encounter a related question.

*Structured code review as teaching tool:*
Instead of just leaving corrective comments, I started adding "why this matters" annotations to every significant review comment: "I'd suggest using a conditional write here because [explanation of race condition it prevents] — this isn't obvious until you've seen it fail." The goal was that engineers could apply the lesson to the next problem, not just fix the current one.

*Architecture office hours:*
30-minute weekly session, open to all three engineers. They could bring designs in progress, not just finished work. Catching a design issue at the whiteboard stage is 10x cheaper than catching it in a PR.

*Pair programming on the first complex task:*
For each engineer, I pair-programmed their first non-trivial feature — not to do it for them, but to narrate my thought process: "Before I write code I want to understand the failure mode — what happens if this Lambda invocation is retried? Let's trace through that." The goal was to transfer a debugging and design mindset, not just syntax.

**Result:**
The three engineers reached productive contribution in approximately 30% less time than the previous onboarding cohort. By month two, all three were submitting PRs that passed review without significant architectural feedback — meaning the design instincts were taking hold. Two of them went on to lead features independently within 4 months. The documentation I wrote is still used for new team members.

---

### Q11. Tell me about a time your work had a positive impact beyond your immediate team.

**Situation:**
During my time as a Software Engineer at Principal, I noticed that while my team had successfully migrated to AWS CDK and serverless, our patterns and learnings were entirely siloed. Other product teams were still dealing with on-premise provisioning overhead, scheduled maintenance windows, and the same infrastructure pain points we had solved 12 months earlier. The knowledge transfer wasn't happening organically.

**Task:**
There was no assigned mandate to drive cross-team adoption. This was self-initiated. But I believed the impact of scaling our practices to other teams was significantly higher than any feature I could have built in the same time.

**Action:**
I requested time from my manager to run a BU-wide technical session. My framing to him: "I can spend the next month building one more Lambda function, or I can spend two weeks preparing a presentation that accelerates five teams' cloud migration. The ROI is asymmetric."

The presentation I prepared wasn't a generic "AWS is great" talk. It was a structured knowledge transfer covering:
- The specific failure modes of on-premise workflows (availability coupling, deployment downtime, capacity planning overhead)
- Our CDK architecture patterns, with reusable code templates teams could use directly
- The measured outcomes from our migration (cost reduction, deployment time, on-call reduction)
- A live Q&A on architectural questions teams were wrestling with

I presented to 150+ engineers across the business unit. I specifically addressed the adoption skeptics in the room by acknowledging the migration risk and offering structured support (architecture review sessions for any team considering migration).

Following the presentation, I established a working group for teams actively planning migrations, providing architecture review and template-sharing on an ongoing basis.

**Result:**
Multiple product teams initiated cloud migration projects in the months after the presentation. The innovation award I received reflected both the technical impact and the cross-team scale of the influence. Beyond the award — the shared CDK templates I created reduced the time for a new team to bootstrap a serverless stack from weeks of infrastructure setup to roughly a day.

---

## Section 6: Initiative & Innovation

---

### Q12. Tell me about a time you took a creative approach to a problem that resulted in outsized impact.

**Situation:**
During a hackathon at Principal, the challenge was to propose solutions for improving insurance agent workflow efficiency. Most teams focused on incremental optimizations to existing tools — faster search, better sorting. I saw a different framing of the problem: agents weren't slow because the tools were slow, they were slow because they had to manually track where they were in multi-step policy workflows.

**Task:**
My goal was to build a prototype that would demonstrate a fundamentally different approach, not an incremental improvement to the existing one. And critically — build something production-viable in the hackathon timeframe, not a demo with fake data.

**Action:**
I focused the design on one insight: in a multi-step workflow, the most expensive cognitive task is remembering your state and knowing what comes next. My prototype automated state management — the system tracked which step a policy was at, surfaced the required next action to the agent, and sent automated follow-up triggers for time-sensitive steps.

I built it as a serverless Lambda + Step Functions workflow (applying the architecture patterns I knew well) so it was architecturally sound, not just a UI demo. I used real policy workflow data (anonymized) to validate the prototype, which made the demo credible.

The presentation focused on the measured time savings per policy completion (estimated, but based on real workflow timing data I had collected beforehand) rather than on technical features.

**Result:**
The prototype won a hackathon award — but more importantly, it was one of two hackathon prototypes I built at Principal that were fully productionized. Shipping a hackathon prototype to production is rare — it requires both a good idea and a technically viable implementation. The fact that I used production-quality architecture patterns rather than a demo-only approach was directly responsible for the production path being feasible.

---

### Q13. Tell me about a time you identified a problem no one else was tracking and proactively addressed it.

**Situation:**
As a member of the Technical Pillar at Principal (Trainee Engineer level), I observed that while we had standardized microservice design patterns documented centrally, new teams joining or splitting were frequently reimplementing the same foundational components from scratch — retry logic, exception hierarchies, logging configuration, health check endpoints. The implementations were all slightly different. In cross-team incidents, log formats were inconsistent and correlation was manual.

**Task:**
This wasn't assigned work. It was a pattern I noticed from being involved in cross-team incident reviews and code reviews for teams that were setting up new services.

**Action:**
I proposed a shared microservice design library to the Technical Pillar team — framing the problem in terms the leadership would care about: "Every new service team spends approximately X days reinventing infrastructure plumbing that should be a 30-minute dependency import. That's engineering time that's not building product features. And the variation in implementations is a reliability risk in cross-team incidents."

I built the first version of the library to prove the concept before asking for broader investment. Starting with the three highest-leverage components: standardized structured logging with MDC correlation ID support, a common exception hierarchy with consistent HTTP mapping, and a pre-configured Resilience4j retry/circuit-breaker bean.

I published the library to our internal Nexus repository, wrote documentation with usage examples, and offered to migrate one team's service to use it as a reference. That reference migration reduced their boilerplate by roughly 200 lines of infrastructure code and made their log format consistent with the rest of the platform.

**Result:**
The library was adopted by 10+ engineering teams. The reduction in new service setup time was significant — teams went from spending days on infrastructure plumbing to spending hours. The standardized log format made cross-team incident correlation in ElasticSearch queries dramatically faster. This initiative was one of the contributions recognized when I was later promoted to Software Engineer.

---

## Section 7: Stakeholder Communication

---

### Q14. Tell me about a time you had to communicate a complex technical concept to a non-technical audience and make a decision land.

**Situation:**
As a Software Engineer at Principal, I needed business stakeholder approval to extend the SOAP-to-REST migration timeline by 3 weeks. The reason was technical: shadow mode had surfaced a 0.8% response discrepancy that I needed time to investigate. But business stakeholders were eager to complete the migration — they had planned communications around the go-live date and had already told downstream teams about the timeline.

**Task:**
I needed to explain a technical risk (response discrepancy, possible edge-case bug) to people who didn't understand SOAP, shadow mode, or what a 0.8% discrepancy actually meant for policy processing — and get a timeline extension approved.

**Action:**
I deliberately avoided technical jargon in the stakeholder meeting. Instead, I reframed the technical situation in business terms:

"We found that in 8 out of every 1,000 policy transactions, the new system gives a different answer than the old system. We're not sure yet whether those differences matter to policyholders or not. If we go live without resolving this, we might process a small number of policies incorrectly — and in insurance, processing a policy incorrectly can mean an incorrect premium, a coverage gap, or a regulatory issue."

I then presented three options with honest trade-offs:
1. **Proceed on schedule:** accept the 0.8% discrepancy risk, with a fallback plan to detect and remediate incorrect policies post-launch (highest risk, meets the timeline)
2. **3-week extension:** resolve the discrepancy, proceed with confidence (moderate risk, minor timeline impact)
3. **Hybrid approach:** proceed with cutover for 99.2% of cases, route the 0.8% edge cases through the legacy system via a feature flag (minimal risk, meets the timeline for most of the system)

I recommended Option 3. Stakeholders chose Option 3. I had come in with a recommendation, not just a problem.

**Result:**
The migration proceeded on schedule for 99.2% of cases. Stakeholders felt heard and in control of the decision. The technical risk was managed correctly. And I learned that framing technical risk in terms of business consequences (incorrect premiums, regulatory exposure) is far more effective than framing it in terms of technical uncertainty.

---

### Q15. Tell me about a time you had to manage expectations when a project wasn't going to deliver on time.

**Situation:**
During MediTrack development, I had communicated an implementation plan for the Lab Service consumer and the Transactional Outbox pattern to the project's stakeholder (in this case, myself — it was a personal project, but I was building it to a self-imposed timeline for portfolio purposes). During implementation, I discovered the outbox relay design had a subtle ordering bug under high concurrent write load: if multiple relay thread iterations ran simultaneously, they could each pick up the same unpublished events, causing duplicate publishes to Kafka.

**Task:**
The fix required implementing optimistic locking on the outbox relay (using `@Version` on the OutboxEvent entity) and rethinking the relay threading model. This added roughly 5 days to the implementation timeline.

**Action:**
I did two things: documented the problem and the fix clearly in a commit message (for my own future reference and for anyone reviewing the code) and updated the project README to accurately reflect the implementation status.

More importantly, I reflected on what I would have done if this were a team project with external stakeholders:
- Communicate early: "I found a correctness issue in the relay design. Here's what it means, here's how I'm fixing it, here's the revised timeline."
- Never sacrifice correctness for timeline: a system that processes events incorrectly under load is worse than a delayed but correct system.
- Provide a concrete revised estimate with the fix approach, not just "we need more time."

**Result:**
The corrected implementation used `SELECT ... FOR UPDATE` pessimistic locking on the relay query to ensure at-most-one relay thread picks up any given batch of outbox events. The added complexity was documented in the code with a comment explaining the concurrency invariant it protects. The system now handles concurrent relay invocations correctly under load.

**The pattern I apply:**
Timeline pressure should never travel faster than technical reality. Communicating a problem early (with a proposed solution) is a sign of engineering maturity. Hiding it until the deadline is not.

---

## Section 8: Time I Disagreed with a Decision

---

### Q16. Tell me about a time you disagreed with a decision made above you and what you did.

**Situation:**
During a project at Principal, the team was deciding on the approach for a new cross-service data sharing requirement. The directive from above was to add a shared database table that both services would read from — a pattern that was faster to implement but violated the service ownership model we had been building toward.

**Task:**
I disagreed with the decision. A shared database table would mean two services were coupled to the same schema — any schema change required coordinating across teams, any write from one service could break the other's reads, and the services could no longer be deployed independently. It was also a violation of the "each service owns its data" principle I had been using as the basis for our architecture decisions.

**Action:**
I first made sure I fully understood the reasoning behind the decision. The driving constraint was timeline — the shared table could be implemented in 3 days, while the correct event-driven approach (publish events from the source service, consume them in the target service) would take 2 weeks.

I didn't refuse or escalate. I wrote a short document — 1 page — that:
1. Acknowledged the timeline constraint explicitly
2. Described the long-term operational risks of the shared table (schema coupling, deployment coupling, inability to scale services independently)
3. Proposed an intermediate option: use the shared table for the initial release but commit to migrating to event-driven within the next sprint, with a specific migration plan and tests to ensure data integrity during the transition

I shared this with my manager privately before raising it more broadly. I asked for his honest read: "Am I overweighting the architectural risk relative to the timeline pressure?"

**Result:**
My manager agreed that the intermediate approach was the right call. We shipped with the shared table and migrated to event-driven 3 weeks later as committed. The migration was cleaner because I had designed the shared table access in a way that made replacement easier (a single DAO class abstracted it, so swapping the implementation required changing one class).

**What I learned:**
Disagreeing with a decision is most effective when you come with a concrete alternative, not just a critique. "Here's why this is wrong" loses. "Here's what the risk is, here's an option that manages the risk, here's my recommendation" wins. And checking your own reasoning with someone you trust before escalating prevents you from being the engineer who cries wolf on every architectural choice.

---

### Q17. Tell me about a time you made a wrong technical decision and how you recovered from it.

**Situation:**
Early in my career at Principal as a Software Engineer, I designed a Lambda function that communicated with the IBM DB2 database directly using a JDBC connection pool initialized at container startup. The reasoning was sound: reuse the connection across warm Lambda invocations to reduce connection overhead.

What I didn't account for: Lambda scales horizontally to many concurrent instances. Each instance had its own connection pool. Under peak load (200 concurrent Lambda invocations), we had 200 connection pools each trying to maintain 2-3 connections — potentially 600 simultaneous DB2 connections. DB2's connection limit was 200. New Lambda invocations at peak load began failing with connection timeout errors.

**Task:**
The system was in production, actively failing at peak load. I needed to triage the impact, fix the root cause, and prevent recurrence.

**Action:**

*Immediate mitigation:*
I implemented a connection limit via Lambda reserved concurrency — capping the maximum simultaneous invocations to a number that kept total connections under DB2's limit. This was a band-aid (it limited throughput) but it stopped the failures within 30 minutes of deployment.

*Root cause fix:*
The correct architectural solution was an RDS Proxy equivalent for DB2 — a connection pooling middleware that multiplexed all Lambda connections through a managed pool. We implemented a separate, always-running connection proxy service (a small ECS container) that Lambda functions connected to via a lightweight protocol. This allowed 200 concurrent Lambda invocations to share 20 actual DB2 connections.

*Process change:*
I added a connection exhaustion simulation to our load testing suite: spin up concurrent Lambda invocations equal to 150% of the reserved concurrency limit and verify the connection-handling behavior. This test would have caught the bug in staging.

**Result:**
The connection proxy reduced peak DB2 connection counts by 85% while maintaining throughput. The reserved concurrency band-aid was removed once the proxy was validated. More importantly, I documented the lesson in our team's architecture decision records: "Lambda scales horizontally; any resource with hard connection limits must be accessed through a pooling layer, never directly from Lambda."

**What I tell engineers about this:**
The failure wasn't in making a wrong assumption. It was in not testing the assumption at production-representative scale. Staging environments that are too small give false confidence. Load test against production-scale parameters.

---

## Section 9: Cross-functional Influence & Leadership

---

### Q18. Tell me about a time you drove a cultural or process change on a team.

**Situation:**
When I joined the Technical Anchor role at Principal, one pattern I observed quickly: pull request reviews were focused almost exclusively on "does this code work." Reviewers checked logic, syntax, and test coverage. But two categories of issues consistently slipped through: edge cases under concurrent load (race conditions, connection exhaustion) and security properties (overprivileged IAM, unmasked PII in logs).

These weren't individual failures — they were systemic gaps in what the team's review culture considered "complete."

**Task:**
I wanted to raise the floor of what "a good PR review" meant, without making reviews longer or more burdensome. More checkbox items would just create more box-checking. The goal was a genuine shift in how engineers thought about code they were reviewing.

**Action:**
I started by demonstrating the standard rather than announcing it. For six consecutive weeks, every PR I reviewed included:
- One comment on the failure mode under concurrent load: "What happens if this Lambda runs 50 instances simultaneously?"
- One comment on the security property: "This IAM role has write permission — does it need it?"
- One comment on the observability: "How would we detect if this is failing silently in production?"

These weren't blocking comments — they were teaching comments. Most of them were phrased as questions rather than directives.

After four weeks, two things happened: engineers started preemptively answering these questions in their PR descriptions ("I considered the concurrent invocation case — here's how I handled it"). And one engineer started asking similar questions in their own reviews without being asked to.

I then formalized it: I created a one-page PR review checklist that included the three categories above, framed as questions not requirements. I introduced it as "a prompt list to make sure we've thought about the things that most often cause production issues" rather than "a new mandatory process."

**Result:**
Over the following quarter, we saw a measurable reduction in production defects — zero incidents from the categories the checklist addressed (concurrent access, security misconfigurations). Engineers reported in retrospective that the checklist made reviews feel more purposeful rather than more bureaucratic. The most gratifying outcome: new engineers onboarding to the team used the checklist as a learning guide for what senior engineers care about, compressing the time it took to internalize the review mindset.

---

*End of Behavioral Interview Preparation*

---

> **Preparation tips:**
> - Practice saying each answer aloud. STAR answers that read well often run long when spoken — aim for 2-3 minutes per answer in actual interviews.
> - Have a "so what did you learn?" closing for every behavioral answer — interviewers often probe this.
> - Where metrics are presented as approximate ("approximately 30%"), be honest in the interview: "we measured it as roughly X" is more credible than false precision.
> - For technical interviewers who ask behavioral questions: anchor to the technical decision inside the story. They'll follow up on the technical detail.
