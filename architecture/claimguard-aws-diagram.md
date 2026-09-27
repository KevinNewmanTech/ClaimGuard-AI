# ClaimGuard AI — AWS Architecture Diagram

## Enterprise Insurance Claims Intelligence & Automation Platform

```mermaid
flowchart TB

    subgraph Intake["1. Customer & Claim Intake"]
        Customer[Customer]
        Cognito[Amazon Cognito<br/>Authentication & MFA]
        API[Amazon API Gateway<br/>Claim Intake API]
        Lambda[AWS Lambda<br/>Intake Logic]

        Customer --> Cognito
        Cognito --> API
        API --> Lambda
    end

    subgraph Storage["2. Durable Data Storage"]
        Aurora[(Amazon Aurora PostgreSQL<br/>Structured Claim Data)]
        S3Original[(Amazon S3<br/>Original Evidence)]
    end

    subgraph Processing["3. Asynchronous Document Processing"]
        SQS[Amazon SQS<br/>Processing Queues]
        Textract[Amazon Textract<br/>OCR & Document Extraction]
        Processed[(Amazon S3<br/>Processed Artifacts)]

        SQS --> Textract
        Textract --> Processed
    end

    subgraph Intelligence["4. AI & Retrieval-Augmented Generation"]
        Knowledge[Claim Knowledge Base<br/>Chunk Retrieval & Source Metadata]
        Bedrock[Amazon Bedrock<br/>AI Analysis & Summarization]

        Knowledge --> Bedrock
    end

    subgraph Human["5. Human Review & Decision Authority"]
        Review[Verification Queue<br/>Missing / Conflicting / Low-Confidence Information]
        Adjuster[Adjuster Dashboard<br/>Human Review & Verification]

        Review --> Adjuster
    end

    Lambda --> Aurora
    Lambda -->|Pre-signed upload authorization| Customer
    Customer -->|Direct document upload| S3Original

    S3Original -->|Document accepted| SQS
    Lambda -->|Claim ready for processing| SQS

    Processed --> Knowledge
    Bedrock -->|Grounded summary + source references| Adjuster
    Bedrock -->|Low confidence / conflict / missing information| Review

    Aurora -->|Claim metadata| Adjuster
    S3Original -->|Original evidence| Adjuster
    Adjuster -->|Verified corrections & decisions| Aurora

    subgraph Security["6. Security, Monitoring & Incident Response"]
        CloudTrail[AWS CloudTrail<br/>Audit Activity]
        CloudWatch[Amazon CloudWatch<br/>Operational Health & Alerts]
        GuardDuty[Amazon GuardDuty<br/>Threat Detection]
        SecurityHub[AWS Security Hub<br/>Security Findings]
        EventBridge[Amazon EventBridge<br/>Security Event Routing]
        Containment[AWS Lambda / Workflow<br/>Automated Containment]
        SNS[Amazon SNS<br/>Critical Notifications]

        GuardDuty --> SecurityHub
        SecurityHub --> EventBridge
        CloudWatch --> EventBridge
        EventBridge --> Containment
        EventBridge --> SNS
    end

    Lambda -. Telemetry .-> CloudWatch
    API -. API health .-> CloudWatch
    S3Original -. Audit activity .-> CloudTrail
    Aurora -. Audit activity .-> CloudTrail
    Containment -. Suspend / revoke access .-> Cognito

    subgraph DR["7. Secondary AWS Region — Warm Standby"]
        DRIntake[Critical Claim Intake<br/>Warm Standby]
        DRDatabase[(Cross-Region Database<br/>Recovery Copy)]
        DRStorage[(Replicated S3<br/>Critical Evidence)]
        DRTarget[Recovery Objectives<br/>RTO ≤ 15 minutes<br/>Near-Zero-Loss RPO Target]

        DRIntake --> DRTarget
        DRDatabase --> DRTarget
        DRStorage --> DRTarget
    end

    Aurora -. Cross-region recovery strategy .-> DRDatabase
    S3Original -. Cross-region replication .-> DRStorage
    Lambda -. Critical intake recovery .-> DRIntake
