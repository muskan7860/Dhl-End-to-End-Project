# 🚚 Enterprise Logistics & Shipment Tracking Platform

## AWS DevOps Production Project --- Master README

> **DHL-inspired hands-on learning project**
>
> This repository documents an enterprise-style logistics platform built
> to learn the complete DevOps lifecycle: **source code → CI → security
> → container registry → GitOps → Kubernetes/EKS → traffic →
> observability → troubleshooting**.
>
> **This is a learning project and does not represent DHL's internal
> infrastructure.**

------------------------------------------------------------------------

## 📌 Project Objective

The goal is not to copy commands or memorize YAML. The goal is to
understand how a DevOps engineer designs, builds, secures, deploys,
monitors, scales, and troubleshoots a production-style platform.

``` text
TEACH
  ↓
DESIGN
  ↓
WRITE
  ↓
EXPLAIN
  ↓
EXECUTE
  ↓
VALIDATE
  ↓
BREAK
  ↓
TROUBLESHOOT
  ↓
DOCUMENT
  ↓
INTERVIEW
```

### Final skills

-   Understand an unfamiliar application before deployment.
-   Derive Dockerfiles from source code.
-   Design AWS networking.
-   Write Terraform from scratch.
-   Build Jenkins pipelines from scratch.
-   Write Kubernetes manifests from scratch.
-   Deploy workloads to Amazon EKS.
-   Store images in Amazon ECR.
-   Implement SonarQube, OWASP Dependency-Check, and Trivy.
-   Implement GitOps using Argo CD.
-   Route traffic using AWS ALB and Kubernetes Ingress.
-   Monitor using Prometheus and Grafana.
-   Troubleshoot production-style incidents.
-   Explain every architectural decision in interviews.

------------------------------------------------------------------------

## 🧰 Technology Stack

  -----------------------------------------------------------------------
  Layer                   Technology              Why We Use It
  ----------------------- ----------------------- -----------------------
  Source Control          Git + GitHub            Version control,
                                                  collaboration, PR
                                                  workflow

  CI                      Jenkins                 Automated build, test,
                                                  scan, image publication

  Code Quality            SonarQube               Static analysis and
                                                  Quality Gate

  Dependency Security     OWASP Dependency-Check  Detect known vulnerable
                                                  dependencies

  Container / IaC         Trivy                   Scan filesystem,
  Security                                        images, and
                                                  configuration

  Containerization        Docker                  Package application
                                                  with runtime
                                                  dependencies

  Registry                Amazon ECR              Store approved
                                                  container images

  Infrastructure as Code  Terraform               Reproducible AWS
                                                  infrastructure

  Cloud                   AWS                     Hosting and managed
                                                  services

  Kubernetes              Amazon EKS              Managed Kubernetes
                                                  control plane

  Traffic                 AWS ALB + Ingress       External HTTP/HTTPS
                                                  routing

  GitOps CD               Argo CD                 Reconcile Git desired
                                                  state into Kubernetes

  Metrics                 Prometheus              Collect/query
                                                  time-series metrics

  Dashboards              Grafana                 Visualize and
                                                  investigate metrics

  Alerting                Alertmanager            Route operational
                                                  alerts
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 🏗️ Final End-to-End Architecture

``` mermaid
flowchart TD
    DEV[Developer] --> GH[GitHub Application Repository]
    GH --> JENKINS[Jenkins CI]

    JENKINS --> BUILD[Build]
    BUILD --> TEST[Unit Tests]
    TEST --> SONAR[SonarQube]
    SONAR --> QG{Quality Gate}
    QG -->|Pass| OWASP[OWASP Dependency Check]
    OWASP --> DOCKER[Docker Build]
    DOCKER --> TRIVY[Trivy Image Scan]
    TRIVY --> ECR[Amazon ECR]

    ECR --> UPDATE[Update GitOps Image Version]
    UPDATE --> GITOPS[GitOps Repository]
    GITOPS --> ARGO[Argo CD]
    ARGO --> EKS[Amazon EKS]

    USER[Customer] --> DNS[DNS]
    DNS --> ALB[AWS Application Load Balancer]
    ALB --> ING[Ingress]
    ING --> SVC[Kubernetes Service]
    SVC --> PODS[Application Pods]

    EKS --> PROM[Prometheus]
    PROM --> GRAF[Grafana]
    PROM --> ALERT[Alertmanager]
```

------------------------------------------------------------------------

## ☁️ AWS Infrastructure Architecture

``` mermaid
flowchart TD
    INTERNET[Internet]

    subgraph AWS[AWS Account]
        subgraph VPC[VPC]
            subgraph PUB[Public Subnets - Multiple AZs]
                ALB[Application Load Balancer]
                NAT[NAT Gateway]
            end

            subgraph PRIV[Private Subnets - Multiple AZs]
                N1[EKS Worker Node]
                N2[EKS Worker Node]
                P1[Application Pods]
                P2[Application Pods]
            end
        end

        ECR[Amazon ECR]
        S3[S3 Terraform State]
        IAM[IAM Roles and Policies]
    end

    INTERNET --> ALB
    ALB --> P1
    ALB --> P2
    N1 --> P1
    N2 --> P2
    N1 --> NAT
    N2 --> NAT
    N1 --> ECR
    N2 --> ECR
```

### Why private worker nodes?

Worker nodes host application workloads and normally do not require
arbitrary direct inbound Internet access.

``` text
Internet
   ↓
AWS ALB
   ↓
Ingress / Routing
   ↓
Private Kubernetes Workloads
```

This reduces the attack surface.

------------------------------------------------------------------------

## 🌐 Complete Customer Request Flow

``` text
Customer Browser
       ↓
DNS
       ↓
AWS ALB
       ↓
Listener / Routing Rule
       ↓
Kubernetes Ingress
       ↓
Kubernetes Service
       ↓
Ready Pod Endpoint
       ↓
Application
       ↓
Internal Microservice
       ↓
Database / Dependency
       ↓
Response
```

A major project goal is to understand **every arrow** in this flow.

------------------------------------------------------------------------

## 🧩 Application Architecture

The DHL-style repository contains multiple components:

``` text
dhl/
├── frontend/
├── backend/
├── banking-service/
├── language-service/
├── price-service/
├── air-cargo-service/
├── sea-cargo-service/
├── k8s/
├── Jenkinsfile
├── docker-compose.yml
└── sonar-project.properties
```

``` mermaid
flowchart TD
    CUSTOMER[Customer] --> FE[Frontend]
    FE --> BE[Backend]
    FE --> LANG[Language Service]
    BE --> PRICE[Price Service]
    BE --> AIR[Air Cargo Service]
    BE --> SEA[Sea Cargo Service]
    BE --> BANK[Banking Service]
    BE --> DB[(Database)]
    BANK --> DB
```

------------------------------------------------------------------------

## 🔄 CI vs CD vs GitOps

### Continuous Integration

``` text
Git Push
 ↓
Jenkins
 ↓
Checkout
 ↓
Build
 ↓
Unit Tests
 ↓
SonarQube
 ↓
Quality Gate
 ↓
OWASP
 ↓
Docker Build
 ↓
Trivy
 ↓
Amazon ECR
```

### GitOps Continuous Deployment

``` text
Approved ECR Image
 ↓
Update GitOps Repository
 ↓
Argo CD Detects Change
 ↓
Compare Desired vs Actual State
 ↓
Reconcile Amazon EKS
```

### Responsibility Separation

  Jenkins                  Argo CD
  ------------------------ ------------------------------
  CI orchestration         Kubernetes CD
  Build and test           Desired-state reconciliation
  Quality/security gates   Drift detection
  Publish image            Synchronize Kubernetes

------------------------------------------------------------------------

## 🔐 Security Architecture

``` text
Source Code
   ↓
SonarQube

Dependencies
   ↓
OWASP Dependency-Check

Filesystem / Container / IaC
   ↓
Trivy

AWS Authorization
   ↓
IAM Least Privilege

Kubernetes Authorization
   ↓
RBAC + Service Accounts

Network
   ↓
Security Groups + NetworkPolicy

Runtime
   ↓
Non-root Containers
Resource Requests/Limits
Security Context

Secrets
   ↓
Jenkins Credentials
AWS Secrets Manager
Approved EKS Workload Identity
```

------------------------------------------------------------------------

## 📊 Monitoring Architecture

``` mermaid
flowchart LR
    APP[Application] --> PROM[Prometheus]
    K8S[Kubernetes] --> PROM
    NODE[EKS Nodes] --> PROM
    PROM --> GRAF[Grafana]
    PROM --> ALERT[Alertmanager]
    ALERT --> TEAM[Slack / Email]
```

We will monitor CPU, memory, pod restarts, node health, request rate,
error rate, latency, application availability, deployment health, and
Prometheus target health.

------------------------------------------------------------------------

## 📁 Repository Strategy

### Application Repository

``` text
dhl/
├── frontend/
├── backend/
├── services/
├── tests/
├── Dockerfiles
├── Jenkinsfile
└── README.md
```

### Infrastructure Repository

``` text
dhl-infrastructure/
├── modules/
│   ├── vpc/
│   ├── security-group/
│   ├── iam/
│   ├── ecr/
│   └── eks/
├── environments/
│   ├── dev/
│   ├── stage/
│   └── prod/
├── backend/
└── README.md
```

### GitOps Repository

``` text
dhl-gitops/
├── base/
├── environments/
│   ├── dev/
│   ├── stage/
│   └── prod/
├── argocd/
└── README.md
```

------------------------------------------------------------------------

## 🗺️ Phase-by-Phase Roadmap

  ------------------------------------------------------------------------
                         Phase Topic                 Main Outcome
  ---------------------------- --------------------- ---------------------
                             1 Project Kickoff &     Understand the
                               Architecture          complete system

                             2 Application Analysis  Learn how DevOps
                                                     studies unknown
                                                     source code

                             3 Git & GitHub          Enterprise Git
                                                     workflow

                             4 Docker                Derive, build,
                                                     optimize and debug
                                                     images

                             5 AWS Foundation        VPC, IAM, routing and
                                                     packet flow

                             6 Terraform             Build AWS
                                                     infrastructure as
                                                     code

                             7 Amazon ECR            Registry,
                                                     authentication and
                                                     lifecycle

                             8 Kubernetes            Understand
                               Fundamentals          architecture and
                                                     objects

                             9 Kubernetes YAML       Write manifests from
                                                     scratch

                            10 Amazon EKS            Deploy Kubernetes on
                                                     AWS

                            11 ALB & Ingress         Trace Internet-to-pod
                                                     traffic

                            12 Jenkins               Build Jenkinsfile
                                                     from scratch

                            13 Complete CI Pipeline  Build, test, scan and
                                                     publish

                            14 SonarQube             Code-quality gate

                            15 OWASP                 Dependency security

                            16 Trivy                 Container/IaC
                                                     security

                            17 Argo CD & GitOps      Kubernetes CD

                            18 Prometheus            Metrics and PromQL

                            19 Grafana               Dashboards and
                                                     analysis

                            20 Alerting              Operational
                                                     notifications

                            21 Production Hardening  Probes, RBAC, HPA,
                                                     PDB, security

                            22 HA & Scaling          Pod/node/AZ
                                                     resilience

                            23 Deployment Strategies Rolling, Blue/Green,
                                                     Canary

                            24 Secrets Management    Secure secret
                                                     delivery

                            25 Observability         Layer-by-layer
                               Troubleshooting       diagnosis

                            26 Incident Simulations  Break, diagnose,
                                                     recover

                            27 Final Mock Interview  Defend the entire
                                                     architecture
  ------------------------------------------------------------------------

------------------------------------------------------------------------

## 🎤 Interview Answer Frameworks

### WH Framework --- Technical Questions

``` text
WHAT → What is it?
WHY → What problem does it solve?
HOW → How does it work?
WHERE/WHEN → Where did we use it?
WHAT IF → What happens when it fails?
```

### STAR --- Behavioral / Project Experience

``` text
S → Situation
T → Task
A → Action
R → Result
```

Use truthful wording:

> **"In my hands-on enterprise-style DHL project, I implemented..."**

### Production Troubleshooting Framework

``` text
1. Understand symptom
2. Determine blast radius
3. Check recent changes
4. Check monitoring
5. Validate external entry point
6. Validate infrastructure
7. Validate Kubernetes
8. Validate application
9. Validate dependencies
10. Identify root cause
11. Recover and validate
12. Prevent recurrence
```

------------------------------------------------------------------------

## 🧪 Required Failure Labs

Before completion we will intentionally reproduce:

1.  Jenkins pipeline failure
2.  SonarQube Quality Gate failure
3.  Vulnerable dependency
4.  Trivy CRITICAL finding
5.  ECR push failure
6.  ImagePullBackOff
7.  CrashLoopBackOff
8.  OOMKilled
9.  Service selector mismatch
10. Ingress 404
11. ALB unhealthy target
12. Argo CD OutOfSync
13. Terraform state/locking issue
14. IAM AccessDenied
15. Prometheus target down
16. Grafana no data
17. Application latency
18. Node failure
19. Rolling deployment failure
20. Rollback

Every failure will use:

``` text
SYMPTOM
↓
POSSIBLE CAUSES
↓
INVESTIGATION
↓
COMMANDS
↓
EVIDENCE
↓
ROOT CAUSE
↓
FIX
↓
VALIDATION
↓
PREVENTION
↓
INTERVIEW ANSWER
```

------------------------------------------------------------------------

## ✅ Final Project Validation

-   [ ] Application architecture understood
-   [ ] Application runs locally
-   [ ] Dockerfiles derived and understood
-   [ ] AWS networking understood
-   [ ] Terraform infrastructure built
-   [ ] ECR configured
-   [ ] EKS configured
-   [ ] Kubernetes YAML written from scratch
-   [ ] ALB/Ingress traffic validated
-   [ ] Jenkins CI implemented
-   [ ] SonarQube Quality Gate implemented
-   [ ] OWASP scanning implemented
-   [ ] Trivy scanning implemented
-   [ ] GitOps repository configured
-   [ ] Argo CD deployment validated
-   [ ] Prometheus configured
-   [ ] Grafana dashboards created
-   [ ] Alerts tested
-   [ ] Probes/resources/HPA/PDB configured
-   [ ] Security controls reviewed
-   [ ] Failure scenarios completed
-   [ ] Rollback tested
-   [ ] Troubleshooting runbook completed
-   [ ] Final project explanation practiced
-   [ ] Mock interview completed

------------------------------------------------------------------------

## 📘 Documentation Rule for Every Phase

Every phase will contain:

``` text
Introduction
Business Requirement
Technical Requirement
WHAT / WHY / HOW
Architecture
Workflow
Implementation
File-by-File Explanation
Commands
Expected Output
Validation
Failure Lab
Troubleshooting Decision Tree
Production Considerations
Security Considerations
Cost Considerations
Common Mistakes
Basic Interview Q&A
Intermediate Interview Q&A
Advanced Interview Q&A
Scenario Q&A
Troubleshooting Q&A
Architecture Q&A
Interviewer Cross-Questions
STAR Project Answer
Key Takeaways
```

------------------------------------------------------------------------

## 🏁 Engineering Standard

The project is complete only when it can be:

``` text
DESIGNED
   ↓
BUILT
   ↓
EXPLAINED
   ↓
OPERATED
   ↓
BROKEN
   ↓
DIAGNOSED
   ↓
RECOVERED
   ↓
IMPROVED
```

The goal is not to say:

> "I ran `kubectl get pods`."

The goal is to say:

> "I queried pod state to establish workload health. Based on the
> result, I used events, logs, readiness, Service endpoints and
> dependency checks to progressively isolate the failing layer before
> applying the minimum remediation."
