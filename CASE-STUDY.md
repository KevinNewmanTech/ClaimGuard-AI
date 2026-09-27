# ClaimGuard AI — Enterprise Solutions Architecture Case Study

## Secure Cloud-Native Insurance Claims Intelligence & Automation Platform

**Project Type:** Independent Enterprise Solutions Architecture Case Study  
**Industry:** Property & Casualty Insurance  
**Primary Cloud:** AWS  
**Secondary Architecture:** Microsoft Azure  
**Architecture Focus:** Cloud • Generative AI • Security • Reliability • Disaster Recovery • FinOps • Human-in-the-Loop AI

---

## Executive Summary

ClaimGuard AI is an independent enterprise architecture case study designed around a fictional mid-sized property and casualty insurer, NorthStar Mutual Insurance.

NorthStar processes approximately 50,000 claims per month and faces several challenges common to large insurance claims operations:

- Manual document review slows claim processing.
- Claim information is distributed across structured data and supporting documents.
- Catastrophe events can create sudden increases in claim volume.
- Sensitive customer information requires strong access controls and auditing.
- Generative AI introduces hallucination, privacy, security, reliability, and cost concerns.
- Critical claim-intake capabilities must remain available even when downstream AI services are degraded.

The objective was not simply to add AI to the claims process.

The architecture was designed around a more important question:

> **How can an insurer safely use cloud and generative AI to accelerate claims operations without sacrificing customer-data protection, human decision authority, operational resilience, or financial control?**

ClaimGuard addresses this through asynchronous cloud architecture, durable evidence storage, document intelligence, retrieval-augmented generation, human verification, fine-grained authorization, security monitoring, controlled AI processing, disaster recovery, and FinOps guardrails.

---

## 1. Business Problem

Traditional claims workflows may require adjusters to manually review police reports, repair estimates, photographs, forms, scanned documents, and customer-submitted evidence.

At normal claim volumes this creates operational overhead.

During catastrophe events, the same workflow can become a significant bottleneck.

ClaimGuard therefore separates two responsibilities:

### Critical Customer Operations

The platform prioritizes:

1. Customer authentication
2. Claim submission
3. Claim-record creation
4. Evidence upload
5. Durable storage
6. Submission confirmation

### Downstream Intelligence

More expensive or delay-tolerant workloads operate asynchronously:

- Document extraction
- Document classification
- AI summarization
- Retrieval-augmented generation
- Conflict detection
- Missing-information detection
- Adjuster decision support

This separation allows customer claims and evidence to be preserved even when downstream processing is delayed or experiencing unusually high demand.

---

## 2. Architecture Principles

Several principles guided the design.

### Preserve Evidence First

Original customer evidence is stored before expensive downstream processing occurs.

Original evidence is preserved separately from processed artifacts so AI or document-processing workflows do not modify the canonical source material.

### Separate Structured Data From Documents

Structured claim information is maintained in a relational data store.

Large binary evidence such as PDFs, photographs, police reports, and repair estimates is maintained in object storage, with the claim record referencing the associated objects.

### Decouple Intake From Processing

Claim intake does not depend on completion of OCR or generative AI processing.

Asynchronous queues absorb downstream workloads and allow processing capacity to scale independently.

### Keep Humans in Authority

ClaimGuard provides decision support rather than autonomous claim adjudication.

AI may summarize documents, retrieve relevant evidence, identify inconsistencies, or flag missing information.

Authorized insurance personnel retain responsibility for consequential claim decisions.

### Treat Security as Part of the Architecture

Authentication alone is not sufficient.

Authorization considers the user's role, assigned claim, requested action, and access context.

Sensitive information is masked by default where appropriate, and security-relevant activity is preserved for investigation.

### Design for Failure

The architecture assumes that cloud services, AI systems, accounts, integrations, and regions can fail.

Recovery, containment, monitoring, and evidence preservation are therefore architectural requirements rather than afterthoughts.

### Optimize Cost Without Sacrificing Protection

FinOps controls expensive variable workloads while preserving critical customer-facing services, security controls, evidence retention, human authority, and recovery objectives.

---

## 3. AWS Architecture

The primary design uses AWS services to support the ClaimGuard requirements.

Major architectural capabilities include:

- **Amazon Cognito** — customer authentication and MFA
- **Amazon API Gateway** — claim-intake API
- **AWS Lambda** — serverless intake and workflow logic
- **Amazon Aurora PostgreSQL** — structured claim information
- **Amazon S3** — original evidence and processed artifacts
- **Amazon SQS** — asynchronous processing
- **Amazon Textract** — OCR and document extraction
- **Amazon Bedrock** — generative AI analysis and summarization
- **Amazon CloudWatch** — operational monitoring and alerting
- **AWS CloudTrail** — audit activity
- **AWS Security Hub / Amazon GuardDuty** — security findings and threat detection
- **AWS KMS** — encryption-key management

The architecture also includes human-verification workflows, security-event routing, automated containment concepts, catastrophe-demand handling, and cross-region disaster-recovery planning.

See the detailed AWS architecture documentation and architecture diagram in this repository.

---

## 4. AI and Human Review

ClaimGuard uses AI as an assistant to claims professionals rather than as the final decision-maker.

Documents can be extracted and divided into retrievable information that maintains source metadata.

Relevant information can then be retrieved for AI analysis so responses remain connected to claim evidence.

The adjuster can receive:

- Claim summaries
- Relevant document information
- Source references
- Missing-information warnings
- Conflicting-information warnings
- Low-confidence extraction warnings

When confidence is insufficient or information conflicts, the claim is routed for human verification.

Human-reviewed corrections take precedence over unverified automated extraction.

---

## 5. Security Architecture

ClaimGuard applies defense-in-depth principles across identity, authorization, data protection, monitoring, and incident response.

The design includes:

- Multi-factor authentication
- Least-privilege access
- Fine-grained claim authorization
- Sensitive-data masking
- Encryption at rest
- Encryption in transit
- Successful and denied access logging
- Protected audit records
- Behavioral and rule-based monitoring
- Security-event severity classification
- Automated containment workflows
- Controlled account restoration

A compromised account can therefore be contained while security evidence remains available for investigation.

---

## 6. Reliability and Disaster Recovery

ClaimGuard distinguishes critical claim-intake services from downstream intelligence workloads.

During a major regional failure, recovery prioritizes:

1. Authentication
2. Claim submission
3. Claim-record creation
4. Evidence upload
5. Durable storage

AI analysis, RAG, analytics, and other noncritical processing can recover progressively afterward.

The architecture uses a warm-standby disaster-recovery strategy with cross-region data protection.

Design targets include:

- **Critical-service RTO target:** 15 minutes or less
- **RPO objective:** Near-zero data loss for successfully accepted claims

These are architectural recovery objectives and must be validated through periodic disaster-recovery testing rather than assumed from the existence of backups.

---

## 7. FinOps Strategy

ClaimGuard does not define cost optimization as simply producing the lowest possible cloud bill.

The objective is to eliminate unnecessary spending while protecting:

- Customer claims
- Original evidence
- Security controls
- Critical customer services
- Human decision authority
- Disaster-recovery requirements

Variable-cost workloads such as document processing and generative AI can therefore be controlled more aggressively than critical claim-intake capabilities.

Cost monitoring distinguishes normal operations from catastrophe-related scaling, AI processing, document processing, storage, security and observability, and disaster recovery.

This provides management with visibility into both **how much the platform costs and why the cost changed.**

---

## 8. Failure and Incident Testing

The architecture is evaluated against controlled failure scenarios rather than assuming the design will always operate normally.

Scenarios include:

1. Compromised adjuster account
2. Unsupported or hallucinated AI response
3. Unauthorized cross-user claim access
4. Downstream service failure
5. Catastrophe-related traffic surge
6. Primary AWS Region failure

Each scenario evaluates detection, containment, evidence preservation, customer impact, recovery behavior, and continued operation of critical insurance services.

---

## 9. Microsoft Azure Architecture

A secondary Azure architecture demonstrates that the business requirements are not tied to AWS service names.

Azure-native implementation candidates include:

- Microsoft Entra External ID
- Microsoft Entra ID
- Azure API Management
- Azure Functions
- Azure Database for PostgreSQL
- Azure Blob Storage
- Azure Service Bus
- Azure AI Document Intelligence
- Azure OpenAI / Azure AI capabilities
- Azure Monitor
- Azure Key Vault

The Azure architecture is intentionally not presented as a guaranteed one-to-one translation of AWS services.

Instead, the same business, security, reliability, AI, and operational requirements are evaluated using Azure-native capabilities.

---

## 10. Architecture Outcome

ClaimGuard demonstrates an enterprise architecture in which AI is only one component of a larger business system.

The design connects:

**Business Requirements → Architecture Decisions → Cloud Services → Security Controls → AI Guardrails → Human Review → Reliability → Disaster Recovery → Cost Governance**

The resulting architecture prioritizes customer protection and operational continuity while allowing AI-assisted claims processing to scale independently.

---

## Scope and Disclosure

ClaimGuard AI is an **independent enterprise solutions architecture case study** using a fictional insurance organization and synthetic scenarios.

It is not represented as a production deployment for an actual insurance carrier.

The project demonstrates architecture analysis, requirements discovery, technical decision-making, cloud-service selection, security design, AI governance, reliability engineering, disaster-recovery planning, FinOps reasoning, and cross-cloud architecture mapping.
