# ClaimGuard AI — Microsoft Azure Architecture

## Purpose

This document demonstrates how ClaimGuard AI's business, security, reliability, and AI requirements could be implemented using Microsoft Azure.

The Azure architecture is not intended to be a one-to-one service-name translation of the AWS architecture.

The business requirements remain constant while implementation choices are evaluated using Azure-native capabilities.

---

## 1. Customer and Workforce Identity

ClaimGuard separates external customer identity from internal workforce identity.

### Customers

Microsoft Entra External ID provides customer-facing identity capabilities.

Customer authentication may include appropriate multifactor authentication and account-recovery controls.

### Adjusters and Workforce Users

NorthStar Mutual employees use the organization's Microsoft Entra ID environment.

Successful authentication does not automatically authorize an employee to access every claim.

ClaimGuard additionally applies application-level authorization based on factors such as:

- User role
- Claim assignment
- Requested action
- Business permissions
- Security state

**Architecture principle:** Authentication establishes identity. Authorization determines what that identity may access.

---

## 2. Claim Intake API

ClaimGuard uses Azure API Management as the managed API layer for customer-facing claim operations.

Azure Functions provide event-driven/serverless application logic for functions such as:

- Claim creation
- Input validation
- Claim status operations
- Evidence-upload authorization
- Workflow initiation

This separates the public API layer from application compute and allows the intake tier to scale independently.

---

## 3. Direct Evidence Upload

Large claim evidence should not unnecessarily travel through the application-compute layer.

ClaimGuard authorizes customers to upload evidence directly to Azure Blob Storage using appropriately scoped and time-limited access mechanisms.

Examples of evidence include:

- Police reports
- Photographs
- Repair estimates
- Scanned forms
- Supporting PDFs
- Other claim documents

The application records the relationship between each accepted object and its associated claim.

---

## 4. Structured Claim Data

Structured relational claim information is stored using an Azure-managed relational database such as Azure Database for PostgreSQL.

Examples include:

- Claim number
- Customer relationship
- Claim status
- Submission date
- Assigned adjuster
- Priority
- Verification state
- Document metadata
- Workflow state

Large binary evidence is not stored directly inside the relational database.

---

## 5. Original and Processed Evidence

Azure Blob Storage provides object storage for claim evidence.

Original evidence is preserved separately from derived or processed artifacts.

This allows ClaimGuard to maintain an authoritative original while creating optimized or processed representations for downstream workloads.

Original evidence should not be silently modified by AI or document-processing workflows.

---

## 6. Asynchronous Processing

Successful customer intake is separated from downstream document and AI processing.

After required information and evidence have been durably accepted, processing work can be placed onto Azure Service Bus queues or topics as appropriate.

This architecture supports:

- Durable asynchronous work
- Retry handling
- Dead-letter handling
- Processing isolation
- Workload buffering
- Catastrophe traffic absorption
- Independent downstream scaling

A downstream processing failure does not automatically require a customer to resubmit evidence that was already safely accepted.

---

## 7. Document Intelligence

Azure AI Document Intelligence can extract structured and textual information from supported claim documents.

Potential extraction targets include:

- Names
- Dates
- Claim identifiers
- Police report numbers
- Vehicle information
- Locations
- Form fields
- Tables
- Narrative text
- Repair-estimate information

Extracted information remains traceable to the original evidence where practical.

Low-confidence or materially uncertain extraction results should be routed for human verification.

---

## 8. AI and Retrieval-Augmented Generation

ClaimGuard uses a grounded retrieval workflow rather than relying solely on unconstrained model generation.

Processed claim content is divided into retrievable units with relevant source metadata.

An appropriate Azure retrieval layer can locate evidence relevant to an adjuster's question or analysis request.

Azure AI capabilities, including appropriate Azure OpenAI or Azure AI Foundry model services, can then assist with:

- Claim summarization
- Document comparison
- Missing-information detection
- Potential inconsistency detection
- Adjuster question answering
- Evidence organization

Generated responses should retain source traceability where required by the workflow.

---

## 9. Human Decision Authority

ClaimGuard does not independently approve or deny insurance claims.

AI-generated analysis functions as decision support.

When information is:

- Low confidence
- Conflicting
- Materially unsupported
- Missing
- Potentially consequential

the system routes the issue for human verification.

Original evidence remains available to the adjuster during review.

**Architecture principle:** AI accelerates understanding; authorized humans retain consequential claim decision authority.

---

## 10. Incremental Processing

ClaimGuard avoids repeatedly processing unchanged information.

When new evidence arrives, the platform identifies newly added or affected information and selectively performs required document extraction, retrieval updates, and AI analysis.

Broader reprocessing occurs when new evidence materially changes existing information or introduces conflicts requiring additional review.

This reduces unnecessary AI, document-processing, compute, and retrieval costs.

---

## 11. Security and Authorization

ClaimGuard applies defense in depth.

Security controls include:

- Microsoft Entra identity controls
- Application-level claim authorization
- Least-privilege access
- Encryption in transit
- Encryption at rest
- Protected secrets and credentials
- Sensitive-data masking where appropriate
- Audit logging
- Security monitoring
- Controlled administrative access

An authenticated workforce user cannot automatically access every claim.

Authorization may consider role, claim assignment, requested action, and other business rules.

---

## 12. Security Monitoring and Incident Response

Security and operational telemetry should be collected using appropriate Azure monitoring and security capabilities.

Potential services include:

- Azure Monitor
- Log Analytics
- Microsoft Defender for Cloud
- Microsoft Sentinel where justified by the organization's security operations model
- Microsoft Entra security and sign-in telemetry

ClaimGuard should preserve both successful and denied access activity according to approved audit requirements.

Suspicious activity can trigger event-driven response workflows where appropriate.

Automated containment must be governed by predefined security policies and should preserve evidence for investigation.

---

## 13. Compromised Account Response

If an authenticated adjuster account begins attempting abnormal access to claims outside its authorization scope, ClaimGuard should:

1. Deny unauthorized claim access.
2. Detect the abnormal behavior.
3. Preserve successful and denied access records.
4. Alert security personnel.
5. Contain the affected account or session according to policy.
6. Preserve relevant audit evidence.
7. Keep access restricted during investigation.
8. Require authorized restoration.

ClaimGuard records observed activity without automatically determining malicious intent.

---

## 14. Operational Monitoring

Azure Monitor and related telemetry capabilities can track platform health.

Important signals include:

- API latency
- API error rates
- Function failures
- Service Bus queue depth
- Oldest queued-work age
- Database health
- Document-processing failures
- AI-processing failures
- Storage errors
- Authentication failures
- Abnormal workload behavior
- Cost anomalies

Operational monitoring should distinguish customer-intake health from downstream-processing health.

---

## 15. Catastrophe Demand Handling

During a catastrophe, claim submissions may rise far above normal volume.

ClaimGuard prioritizes:

1. Customer access
2. Claim submission
3. Durable claim-record creation
4. Evidence preservation
5. Critical/emergency claim prioritization

Azure Service Bus buffers downstream processing demand.

Document and AI processing can scale in a controlled manner while lower-priority work temporarily accumulates when necessary.

ClaimGuard does not intentionally discard safely accepted claims simply because downstream processing is delayed.

---

## 16. Cost Management

Azure implementation decisions should consider both operational requirements and cloud cost.

Cost controls should include appropriate use of:

- Azure Cost Management
- Budgets and alerts
- Resource tagging
- Workload-level cost visibility
- Storage lifecycle management
- Incremental AI/document processing
- Right-sizing
- Controlled catastrophe scaling

Cost optimization must not compromise required evidence preservation, security controls, human decision authority, or critical recovery objectives.

---

## 17. Storage Lifecycle

Original evidence is retained according to approved business, legal, and regulatory retention requirements.

Eligible older evidence may transition to lower-cost Azure Blob Storage access tiers through lifecycle-management policies when immediate access is no longer required.

Evidence is not deleted solely because lower cloud cost is desirable.

---

## 18. Multi-Region Disaster Recovery

ClaimGuard maintains a cost-optimized secondary-region recovery strategy for critical capabilities.

Critical data and evidence use appropriate Azure replication and recovery mechanisms based on the required recovery objectives.

The secondary environment does not need to operate at full production scale continuously.

During regional failure, recovery prioritizes:

1. Authentication and access
2. Claim submission
3. Claim-record creation
4. Evidence upload
5. Durable storage

After critical intake is stable, ClaimGuard progressively restores:

- Document processing
- AI analysis
- Retrieval-augmented generation
- Adjuster-support functionality
- Analytics and other noncritical workloads

---

## 19. Recovery Objectives

**Target RTO:** 15 minutes or less for critical claim-intake capabilities.

**Target RPO:** Near-zero data loss for successfully accepted claims.

These are design targets rather than guarantees.

They must be validated through periodic recovery testing using the selected Azure data-replication and failover mechanisms.

---

## 20. Cross-Cloud Architecture Mapping

The following table shows conceptual implementation relationships rather than guaranteed one-to-one service equivalence.

| Architecture Capability | AWS Design | Azure Candidate |
| --- | --- | --- |
| Customer identity | Amazon Cognito | Microsoft Entra External ID |
| Workforce identity | IAM / workforce identity controls | Microsoft Entra ID |
| API management | Amazon API Gateway | Azure API Management |
| Serverless compute | AWS Lambda | Azure Functions |
| Relational claim data | Amazon Aurora PostgreSQL | Azure Database for PostgreSQL |
| Original evidence | Amazon S3 | Azure Blob Storage |
| Asynchronous messaging | Amazon SQS / event workflows | Azure Service Bus |
| Document extraction | Amazon Textract | Azure AI Document Intelligence |
| Generative AI | Amazon Bedrock | Azure OpenAI / Azure AI Foundry capabilities |
| Retrieval / knowledge layer | Bedrock-compatible retrieval architecture | Azure AI retrieval/search capabilities |
| Operational monitoring | Amazon CloudWatch | Azure Monitor |
| Audit/security telemetry | CloudTrail and security services | Azure Monitor / Entra / security telemetry |
| Security posture | AWS security services | Microsoft Defender for Cloud |
| Security operations | Security Hub / organizational tooling | Microsoft Sentinel where appropriate |
| Cost management | AWS Budgets / Cost Explorer / Cost Anomaly Detection | Azure Cost Management / budgets / alerts |
| Object lifecycle | Amazon S3 lifecycle policies | Azure Blob lifecycle management |
| Regional recovery | Secondary AWS region / warm standby | Secondary Azure region / cost-optimized recovery environment |

---

## 21. Architecture Conclusion

The Azure implementation preserves ClaimGuard's core architectural principles:

- Preserve customer claims before downstream processing.
- Separate structured claim data from original evidence.
- Decouple intake from document and AI workloads.
- Ground AI output in retrievable source evidence.
- Preserve human decision authority.
- Apply fine-grained authorization beyond authentication.
- Detect and contain abnormal security behavior.
- Control variable AI and document-processing cost.
- Preserve critical services during catastrophe conditions.
- Recover customer-facing claim intake before nonessential workloads.

The cloud provider changes.

**The business requirements, security principles, resilience objectives, and architectural reasoning remain the foundation of the system.**
