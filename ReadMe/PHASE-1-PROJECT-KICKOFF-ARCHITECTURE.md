# 🚚 Phase 1 --- Project Kickoff & Architecture

## Enterprise Logistics & Shipment Tracking Platform

> **Goal of Phase 1:** Understand the business application, application
> architecture, DevOps architecture, CI/CD, AWS design, security,
> observability, traffic flow, repository strategy, and the reason each
> major technology exists **before writing Terraform, Kubernetes YAML,
> or Jenkinsfile**.

------------------------------------------------------------------------

## 📚 Table of Contents

-   [1. Business Problem](#1-business-problem)
-   [2. Application Components](#2-application-components)
-   [3. Application Architecture](#3-application-architecture)
-   [4. DevOps Architecture](#4-devops-architecture)
-   [5. CI/CD Architecture](#5-cicd-architecture)
-   [6. AWS Architecture](#6-aws-architecture)
-   [7. Traffic Flow](#7-traffic-flow)
-   [8. Security Architecture](#8-security-architecture)
-   [9. Monitoring Architecture](#9-monitoring-architecture)
-   [10. Repository Strategy](#10-repository-strategy)
-   [11. Tool-by-Tool WHY](#11-tool-by-tool-why)
-   [12. Interview Answer Frameworks](#12-interview-answer-frameworks)
-   [13. Basic Interview Questions](#13-basic-interview-questions)
-   [14. Intermediate Interview
    Questions](#14-intermediate-interview-questions)
-   [15. Advanced & Architecture
    Questions](#15-advanced--architecture-questions)
-   [16. Scenario Questions](#16-scenario-questions)
-   [17. Troubleshooting Questions](#17-troubleshooting-questions)
-   [18. STAR Project Answers](#18-star-project-answers)
-   [19. Phase 1 Interview Story](#19-phase-1-interview-story)
-   [20. Key Takeaways](#20-key-takeaways)

------------------------------------------------------------------------

# 1. Business Problem

We are building an enterprise-style logistics platform inspired by a
shipment/cargo system.

The application is split into multiple components so different business
capabilities can be handled independently.

### Business capabilities represented in the project

-   User-facing frontend
-   Main backend/API
-   Price-related functionality
-   Air-cargo functionality
-   Sea-cargo functionality
-   Banking/payment-related functionality
-   Language-related functionality
-   Persistent data where required

### Functional requirements

The system should allow the application to serve logistics-related
business functions through multiple cooperating services.

### Non-functional requirements

The DevOps platform should target:

-   Availability
-   Scalability
-   Security
-   Repeatable deployments
-   Observability
-   Recoverability
-   Controlled change
-   Infrastructure reproducibility
-   Auditability
-   Production-style troubleshooting

------------------------------------------------------------------------

# 2. Application Components

Current logical repository structure:

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

> **Important:** We will study the real source before deciding Docker
> runtime, build commands, ports, health checks, environment variables,
> or deployment configuration.

------------------------------------------------------------------------

# 3. Application Architecture

``` mermaid
flowchart TD
    CUSTOMER[Customer Browser] --> FE[Frontend]
    FE --> BE[Backend]
    FE --> LANG[Language Service]

    BE --> PRICE[Price Service]
    BE --> AIR[Air Cargo Service]
    BE --> SEA[Sea Cargo Service]
    BE --> BANK[Banking Service]

    BE --> DB[(Database)]
    BANK --> DB
```

## WHAT is this architecture?

It is a multi-service application where business capabilities are
separated into different application components.

## WHY separate services?

Potential benefits:

-   Independent service ownership
-   Independent scaling
-   Smaller change/failure boundaries
-   Clearer separation of business responsibilities
-   Independent deployment where dependencies permit it

## WHAT are the trade-offs?

-   More network communication
-   More failure modes
-   Service discovery requirements
-   Distributed configuration
-   More observability requirements
-   More deployment complexity
-   Cross-service compatibility concerns

## Interview answer

> In this project, the application is organized into multiple services
> representing different logistics capabilities. The benefit is clearer
> service boundaries and the ability to scale or deploy appropriate
> components independently. The trade-off is distributed-system
> complexity, so networking, service discovery, observability, security
> and failure handling become more important.

------------------------------------------------------------------------

# 4. DevOps Architecture

``` mermaid
flowchart TD
    DEV[Developer] --> GH[GitHub]
    GH --> JENKINS[Jenkins]

    JENKINS --> TEST[Build and Tests]
    TEST --> SONAR[SonarQube]
    SONAR --> OWASP[OWASP Dependency Check]
    OWASP --> DOCKER[Docker Build]
    DOCKER --> TRIVY[Trivy]
    TRIVY --> ECR[Amazon ECR]

    ECR --> GITOPS[Update GitOps Repository]
    GITOPS --> ARGO[Argo CD]
    ARGO --> EKS[Amazon EKS]

    USER[Customer] --> ALB[AWS ALB]
    ALB --> ING[Ingress]
    ING --> SVC[Service]
    SVC --> POD[Pod]

    EKS --> PROM[Prometheus]
    PROM --> GRAF[Grafana]
```

## DevOps lifecycle

``` text
CODE
 ↓
BUILD
 ↓
TEST
 ↓
QUALITY
 ↓
SECURITY
 ↓
PACKAGE
 ↓
STORE
 ↓
DEPLOY
 ↓
RUN
 ↓
ROUTE
 ↓
OBSERVE
 ↓
TROUBLESHOOT
```

------------------------------------------------------------------------

# 5. CI/CD Architecture

## CI

``` text
Developer
 ↓
GitHub
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

CI answers:

> **Can we reliably produce a tested and policy-compliant artifact?**

## CD using GitOps

``` text
Amazon ECR Image
 ↓
GitOps Repository Updated
 ↓
Argo CD Detects Change
 ↓
Desired State vs Actual State
 ↓
Reconciliation
 ↓
Amazon EKS
```

CD answers:

> **How does the approved artifact reach Kubernetes in a controlled and
> reproducible way?**

## Why Jenkins and Argo CD together?

``` text
Jenkins = CI
Argo CD = Kubernetes CD
```

### Strong interview answer

> Jenkins handles the CI lifecycle: checkout, build, tests,
> quality/security gates and container publication. Argo CD handles
> Kubernetes deployment through GitOps. Jenkins updates the desired
> deployment state in Git, and Argo CD reconciles that state into EKS.
> This separates artifact creation from runtime reconciliation and gives
> better auditability and drift detection than using Jenkins as the only
> deployment authority.

------------------------------------------------------------------------

# 6. AWS Architecture

``` mermaid
flowchart TD
    INTERNET[Internet]

    subgraph VPC[AWS VPC]
        subgraph PUB[Public Subnets - AZ A / AZ B]
            ALB[AWS ALB]
            NAT[NAT Gateway]
        end

        subgraph PRIV[Private Subnets - AZ A / AZ B]
            N1[EKS Worker Node]
            N2[EKS Worker Node]
            P1[Application Pods]
            P2[Application Pods]
        end
    end

    INTERNET --> ALB
    ALB --> P1
    ALB --> P2
    N1 --> P1
    N2 --> P2
    N1 --> NAT
    N2 --> NAT
```

## Why multiple Availability Zones?

A production design should avoid making one AZ the only failure domain.

## Why public subnets?

Internet-facing resources such as an external ALB require routing
appropriate for public access.

## Why private worker nodes?

Worker nodes host application workloads. Direct Internet exposure is
generally unnecessary and increases attack surface.

## Why NAT?

Private workloads may require outbound Internet access for approved use
cases. NAT provides outbound translation without making the private node
directly Internet-addressable.

> **Cost note:** NAT Gateway is a paid AWS resource. During the
> Terraform phase we will compare production design with lower-cost lab
> alternatives.

------------------------------------------------------------------------

# 7. Traffic Flow

The final customer path is:

``` text
Customer
 ↓
DNS
 ↓
AWS ALB
 ↓
Listener / Rule
 ↓
Ingress
 ↓
Kubernetes Service
 ↓
Ready Endpoint
 ↓
Pod
 ↓
Application
 ↓
Internal Service / Database
```

## Every hop explained

### 1. Customer

The browser creates an HTTP/HTTPS request.

### 2. DNS

DNS resolves the application hostname to the external entry point.

### 3. AWS ALB

ALB accepts Layer-7 HTTP/HTTPS traffic, performs configured routing and
health checking.

### 4. Ingress

Ingress represents Kubernetes HTTP routing intent. On AWS, the AWS Load
Balancer Controller can translate appropriate Kubernetes resources into
AWS load-balancing configuration.

### 5. Kubernetes Service

The Service provides stable discovery/routing for a dynamic group of
pods.

### 6. Endpoint / EndpointSlice

Eligible pods are represented as backend endpoints.

### 7. Pod

The selected pod runs the application container.

### 8. Internal dependency

The application may call another service or database before returning a
response.

------------------------------------------------------------------------

# 8. Security Architecture

``` mermaid
flowchart TD
    CODE[Source Code] --> SONAR[SonarQube]
    DEPS[Dependencies] --> OWASP[OWASP Dependency Check]
    IMAGE[Container Image] --> TRIVY[Trivy]
    AWS[AWS Access] --> IAM[IAM Least Privilege]
    K8S[Kubernetes Access] --> RBAC[RBAC]
    NET[Network] --> SG[Security Groups / NetworkPolicy]
    SECRET[Secrets] --> SM[Credentials / Secrets Manager]
```

## Security layers

  Layer          Control
  -------------- -------------------------------------------
  Source         SonarQube
  Dependencies   OWASP Dependency-Check
  Image/IaC      Trivy
  AWS            IAM
  Kubernetes     RBAC
  Network        Security Groups / NetworkPolicy
  Runtime        Security Context / non-root / limits
  Secrets        Jenkins Credentials / AWS Secrets Manager

### Interview principle

Do not say:

> "We use Trivy, therefore the pipeline is DevSecOps."

Better:

> Security is implemented across source code, dependencies, container
> artifacts, cloud authorization, Kubernetes authorization, network
> boundaries, runtime controls and secret management.

------------------------------------------------------------------------

# 9. Monitoring Architecture

``` mermaid
flowchart LR
    APP[Application] --> PROM[Prometheus]
    K8S[Kubernetes] --> PROM
    NODE[Nodes] --> PROM

    PROM --> GRAF[Grafana]
    PROM --> ALERT[Alertmanager]
    ALERT --> NOTIFY[Slack / Email]
```

## What will we observe?

-   CPU
-   Memory
-   Pod status
-   Restarts
-   Node condition
-   Request rate
-   Error rate
-   Latency
-   Availability
-   Deployment health
-   Scrape target health

## Why Prometheus?

Prometheus collects and queries time-series metrics.

## Why Grafana?

Grafana visualizes metrics and supports investigation through
dashboards.

### Important interview point

A dashboard alone is not observability. The engineer must know what a
graph means and what diagnostic action to take when it changes.

------------------------------------------------------------------------

# 10. Repository Strategy

We will eventually separate concerns logically.

## Application

``` text
dhl/
└── application source + CI entry point
```

## Infrastructure

``` text
dhl-infrastructure/
└── Terraform for AWS
```

## GitOps

``` text
dhl-gitops/
└── desired Kubernetes deployment state
```

### Why separate?

-   Different change lifecycles
-   Cleaner permissions
-   Clear ownership
-   Better GitOps auditability
-   Infrastructure and application releases do not have to be coupled

------------------------------------------------------------------------

# 11. Tool-by-Tool WHY

  Tool                     What Problem Does It Solve?
  ------------------------ --------------------------------------------
  Git                      Tracks source changes
  GitHub                   Shared remote repository and collaboration
  Docker                   Packages app + runtime dependencies
  Jenkins                  Automates CI
  SonarQube                Detects code-quality/security issues
  OWASP Dependency-Check   Detects known vulnerable dependencies
  Trivy                    Scans images/files/configuration
  ECR                      Stores container artifacts
  Terraform                Reproducible infrastructure
  EKS                      Managed Kubernetes platform
  Kubernetes               Orchestrates containers
  ALB                      External Layer-7 load balancing
  Ingress                  Kubernetes HTTP routing intent
  Argo CD                  GitOps reconciliation
  Prometheus               Metrics collection/query
  Grafana                  Metrics visualization
  Alertmanager             Alert routing

------------------------------------------------------------------------

# 12. Interview Answer Frameworks

## WH Method

Use for technical concepts:

``` text
WHAT
WHY
HOW
WHERE/WHEN
WHAT IF
```

### Example --- Kubernetes Service

**WHAT:** A Kubernetes Service provides a stable network abstraction for
a set of pods.

**WHY:** Pod IPs are dynamic and workloads are recreated.

**HOW:** The Service uses selectors to identify eligible pod backends
represented through endpoint data.

**WHERE:** We use Services between application components and behind
external routing.

**WHAT IF:** If selectors do not match pod labels, the Service can exist
but have no usable endpoints.

------------------------------------------------------------------------

## STAR Method

Use for project/behavioral questions:

``` text
S → Situation
T → Task
A → Action
R → Result
```

Never fabricate production employment. Say:

> "In my hands-on enterprise-style DHL project..."

------------------------------------------------------------------------

## Incident Method

``` text
Symptom
↓
Blast Radius
↓
Recent Changes
↓
Monitoring
↓
Layer-by-Layer Investigation
↓
Root Cause
↓
Minimum Remediation
↓
Validation
↓
Prevention
```

------------------------------------------------------------------------

# 13. Basic Interview Questions

## Q1. What are you building?

### Strong Answer

> I am building a hands-on enterprise-style logistics microservices
> platform and its complete DevOps lifecycle. The application contains a
> frontend, backend and supporting services such as pricing, cargo,
> banking and language services. I am implementing AWS infrastructure
> with Terraform, CI with Jenkins, security gates with
> SonarQube/OWASP/Trivy, container storage in ECR, Kubernetes on EKS,
> GitOps deployment with Argo CD, external routing with ALB/Ingress, and
> observability with Prometheus/Grafana.

### Cross-question

**Why did you choose so many tools?**

> Each tool owns a distinct responsibility. The design is not about tool
> count; it is about separating infrastructure provisioning, CI,
> security, artifact storage, runtime orchestration, deployment
> reconciliation and observability.

------------------------------------------------------------------------

## Q2. What is the difference between application architecture and DevOps architecture?

### Strong Answer

> Application architecture describes how business components such as
> frontend, backend, pricing and cargo services communicate. DevOps
> architecture describes how those components move from source code to a
> secure running environment through CI, artifact management,
> infrastructure, deployment, networking and observability.

------------------------------------------------------------------------

## Q3. Why Docker?

### WH Answer

**WHAT:** Docker packages the application and runtime dependencies into
an image.

**WHY:** It reduces environment inconsistency and gives CI/Kubernetes a
portable artifact.

**HOW:** A Dockerfile defines image layers and runtime behavior.

**WHERE:** Each deployable service will be containerized.

**WHAT IF:** Without standardized images, runtime dependencies would
need to be prepared separately on each host/environment.

------------------------------------------------------------------------

## Q4. Why Kubernetes?

### Strong Answer

> Kubernetes provides declarative orchestration for containerized
> workloads. In this project it will manage replicas, self-healing,
> service discovery, rollout behavior, configuration, health checks and
> scaling. I am not using Kubernetes simply because it is popular; I am
> using it because multiple containerized services require consistent
> runtime orchestration.

------------------------------------------------------------------------

## Q5. Why Terraform?

### Strong Answer

> Terraform lets me describe AWS infrastructure as version-controlled
> code. It makes VPC, subnets, IAM, ECR and EKS reproducible and allows
> me to review a plan before changing infrastructure. Modules will help
> reuse common patterns across environments.

------------------------------------------------------------------------

# 14. Intermediate Interview Questions

## Q6. Why Jenkins and Argo CD together?

### Strong Answer

> Jenkins handles CI and artifact production. Argo CD handles Kubernetes
> CD through GitOps reconciliation. Jenkins should not need to be the
> long-term source of truth for cluster state. Git contains the desired
> deployment state and Argo CD compares it with the live cluster. This
> gives clearer auditability, drift visibility and rollback behavior.

### Follow-up

**Can Jenkins deploy directly?**

> Yes. Jenkins can run `kubectl` or Helm. We will understand that model
> too. The GitOps design is selected because it separates CI from
> runtime reconciliation.

------------------------------------------------------------------------

## Q7. Why ECR instead of Docker Hub?

### Strong Answer

> For an AWS/EKS-focused platform, ECR provides a private AWS-native
> registry with IAM integration and straightforward access from AWS
> workloads. Docker Hub can also work, especially for public/lab images,
> but ECR aligns better with the target AWS architecture.

------------------------------------------------------------------------

## Q8. Why private worker nodes?

### Strong Answer

> Worker nodes host application workloads and generally do not need
> arbitrary direct inbound Internet access. External traffic should
> enter through a controlled public endpoint such as ALB. Private
> workers reduce attack surface while approved outbound access can be
> provided through appropriate routing, NAT or VPC endpoints.

------------------------------------------------------------------------

## Q9. Why Service if pods already have IP addresses?

### Strong Answer

> Pod IPs are ephemeral. A Deployment can replace pods at any time. A
> Kubernetes Service provides stable discovery and forwards traffic to
> eligible pod endpoints. This decouples clients from individual pod
> lifecycle.

------------------------------------------------------------------------

## Q10. SonarQube vs OWASP vs Trivy?

  Tool                     Primary Focus
  ------------------------ ----------------------------------------------------
  SonarQube                Source-code quality/security analysis
  OWASP Dependency-Check   Known vulnerable dependencies
  Trivy                    Images, filesystem, packages and configuration/IaC

### Strong Answer

> They operate at different security layers. I would not treat any one
> scanner as complete security coverage.

------------------------------------------------------------------------

# 15. Advanced & Architecture Questions

## Q11. Why EKS instead of ECS?

### Strong Answer

> EKS is appropriate when Kubernetes APIs/ecosystem and portability are
> important, and when the team is prepared to operate Kubernetes
> workloads. ECS is operationally simpler for many AWS-only container
> workloads. For this project, EKS is deliberately selected because
> Kubernetes, GitOps, Ingress, HPA, PDB, RBAC and cluster
> troubleshooting are core learning objectives. In a real architecture
> review, I would evaluate operational complexity and requirements
> before choosing.

------------------------------------------------------------------------

## Q12. ALB vs NLB?

### Strong Answer

> ALB operates at Layer 7 and is suited to HTTP/HTTPS features such as
> host/path routing. NLB operates primarily at Layer 4 and is
> appropriate for high-performance TCP/UDP/TLS use cases where Layer-7
> routing is not required. Our web application needs HTTP routing, so
> ALB is the natural starting choice.

------------------------------------------------------------------------

## Q13. Why multi-AZ?

### Strong Answer

> A single-AZ design creates an availability dependency on one failure
> domain. Distributing appropriate infrastructure and workloads across
> multiple AZs improves resilience. However, true HA also depends on
> application replicas, load balancing, storage/database design, pod
> placement and capacity---not merely creating two subnets.

------------------------------------------------------------------------

## Q14. Why not expose every microservice publicly?

### Strong Answer

> Most backend services are internal implementation details. Public
> exposure increases attack surface and operational complexity.
> Typically only required entry points are externally exposed, while
> internal services use private Kubernetes networking and authorization
> controls.

------------------------------------------------------------------------

## Q15. What is reconciliation?

### Strong Answer

> Reconciliation is the control-loop pattern where a controller compares
> desired state with actual state and takes actions to reduce the
> difference. Kubernetes controllers do this for resources such as
> Deployments, and Argo CD applies the same principle between Git
> desired state and live Kubernetes state.

------------------------------------------------------------------------

# 16. Scenario Questions

## Scenario 1 --- Customers receive HTTP 503

### Question

Production users suddenly receive HTTP 503, and there was no application
deployment. How would you investigate?

### Strong Answer

> First I determine the blast radius: all users, one route, one service,
> or one AZ. Then I check monitoring and infrastructure events for the
> failure start time. Because 503 commonly indicates that a
> proxy/load-balancing layer cannot use a healthy backend, I trace the
> request path from ALB target health to Ingress, Service, endpoints and
> pod readiness. If endpoints are empty, I inspect selectors and
> readiness. If endpoints exist, I test backend connectivity and inspect
> application/dependency health. I avoid restarting everything because
> that can remove useful evidence. After identifying the root cause, I
> apply the minimum remediation, validate user traffic and document
> prevention.

### Interviewer may ask

-   Why check endpoints?
-   What can make endpoints empty?
-   Could a NetworkPolicy cause this?
-   What if ALB targets are unhealthy but pods are Ready?

------------------------------------------------------------------------

## Scenario 2 --- Application works internally but not through ALB

### Investigation

``` text
Application Pod
   ✓
Service
   ✓
Internal Connectivity
   ✓
External ALB
   ✗
```

Check:

1.  Ingress host/path
2.  Ingress class/controller
3.  ALB listener/rules
4.  Target group
5.  Health-check path/port
6.  Security groups
7.  Service/backend mapping
8.  DNS

### Strong Answer

> Because internal Service access already works, I would not begin by
> changing the application. I would focus on the external path:
> Ingress/controller state, ALB rules, target health, health-check
> configuration, security groups and DNS. This uses the evidence to
> narrow the failing layer.

------------------------------------------------------------------------

## Scenario 3 --- Argo CD says Synced but users see old code

### Strong Answer

> `Synced` means the live manifests match Git desired state; it does not
> prove Git points to the intended artifact. I would verify the image
> reference in GitOps, the Deployment image, the actual pod image
> ID/digest and rollout status. I would also check whether a mutable tag
> was reused. Immutable version tags or digests make this easier to
> diagnose.

------------------------------------------------------------------------

## Scenario 4 --- CPU is low but latency is high

### Strong Answer

> Low CPU only tells me the service is not CPU-saturated. Requests may
> be waiting on the database, downstream APIs, DNS, network, storage,
> locks, connection pools or queues. I would correlate latency with
> dependency metrics/logs and trace the slow request path instead of
> scaling CPU immediately.

------------------------------------------------------------------------

# 17. Troubleshooting Questions

## Q16. Pod is Running but application is unavailable. What do you check?

### Answer

``` text
Pod Running
 ↓
Readiness?
 ↓
Application Port?
 ↓
Service Selector?
 ↓
Endpoints?
 ↓
Application Logs?
 ↓
Dependencies?
 ↓
NetworkPolicy?
 ↓
Ingress?
```

> `Running` is a runtime state, not proof that the application is ready
> to serve requests.

------------------------------------------------------------------------

## Q17. Service exists but has no endpoints. Why?

### Possible causes

-   Selector does not match pod labels
-   Pods are not Ready
-   Wrong namespace/design assumptions
-   Workload not created
-   Label changed during deployment

### Strong Answer

> I compare the Service selector with actual pod labels and inspect
> readiness. A Service object can exist successfully even when it has no
> usable backend endpoints.

------------------------------------------------------------------------

## Q18. Why not restart immediately during an incident?

### Strong Answer

> Restarting may temporarily mask the symptom and destroy evidence such
> as logs or runtime state. I first capture enough evidence to identify
> the failing layer unless immediate recovery is required by incident
> severity. Remediation should be deliberate and followed by root-cause
> analysis.

------------------------------------------------------------------------

# 18. STAR Project Answers

## STAR --- Architecture Design

**Situation:**\
I wanted to convert a logistics microservices application into an
enterprise-style DevOps project rather than only running it locally.

**Task:**\
I needed to design a complete delivery architecture covering
infrastructure, CI, security, Kubernetes deployment, traffic routing,
GitOps and observability.

**Action:**\
I separated the platform into clear responsibilities: Terraform for AWS
infrastructure, Jenkins for CI, SonarQube/OWASP/Trivy for security and
quality gates, ECR for container artifacts, EKS for Kubernetes, Argo CD
for GitOps deployment, ALB/Ingress for external traffic, and
Prometheus/Grafana for observability. I also defined the request path
and planned failure simulations so each layer can be troubleshot
independently.

**Result:**\
The project now has a structured architecture and implementation roadmap
where every technology has a defined responsibility and can be defended
in an interview instead of being listed as disconnected tools.

------------------------------------------------------------------------

## STAR --- Kubernetes Service Failure Example

**Situation:**\
In a production-style failure simulation, pods were Running but
application traffic through a Kubernetes Service failed.

**Task:**\
I needed to determine whether the issue was the application, workload
health, or Service routing.

**Action:**\
I inspected pod state, Service configuration and endpoints. The Service
had no endpoints. I compared Service selectors with pod labels,
identified the mismatch, corrected the configuration, reapplied it and
validated that endpoints were populated before testing traffic.

**Result:**\
Service connectivity was restored. The exercise demonstrated that
`Running` pods do not guarantee application reachability and that
Service endpoints are a key Kubernetes networking checkpoint.

------------------------------------------------------------------------

# 19. Phase 1 Interview Story

## 2-Minute Version

> I am building a hands-on enterprise-style DHL-inspired logistics
> project using a multi-service application. Before implementing
> infrastructure, I designed the complete delivery architecture.
> Terraform will provision AWS networking, IAM, ECR and EKS. Jenkins
> will handle CI with build, tests and security/quality gates using
> SonarQube, OWASP Dependency-Check and Trivy. Approved container images
> will be stored in ECR.
>
> For CD, I am separating Jenkins from Kubernetes deployment by using a
> GitOps repository and Argo CD. Argo CD will reconcile the desired
> state into EKS. External traffic will enter through AWS ALB and
> Ingress, then reach Kubernetes Services and ready pods. Prometheus and
> Grafana will provide monitoring, and the project includes intentional
> failure scenarios so I can practice production-style troubleshooting.
>
> The main objective is to understand why every layer exists and how to
> trace failures across the complete path rather than only learning
> deployment commands.

------------------------------------------------------------------------

# 20. Key Takeaways

By the end of Phase 1, I should be able to explain:

-   What the application does at a high level.
-   Why it has multiple services.
-   Difference between application and DevOps architecture.
-   Difference between CI and CD.
-   Why Jenkins and Argo CD have different responsibilities.
-   Why Terraform is used.
-   Why EKS is used.
-   Why worker nodes are private.
-   Why ECR is required.
-   Why ALB, Ingress, Service and Pod are different layers.
-   How customer traffic reaches a pod.
-   Why security requires multiple layers.
-   Why Prometheus and Grafana have different roles.
-   How to approach incidents layer by layer.
-   How to describe project work truthfully using WH and STAR.

------------------------------------------------------------------------

## ✅ Phase 1 Completion Check

-   [ ] I can draw the complete architecture without looking.
-   [ ] I can explain every major tool in one sentence.
-   [ ] I can explain CI vs CD.
-   [ ] I can explain Jenkins vs Argo CD.
-   [ ] I can trace Browser → ALB → Ingress → Service → Pod.
-   [ ] I can explain why worker nodes are private.
-   [ ] I can explain SonarQube vs OWASP vs Trivy.
-   [ ] I can answer the Phase 1 scenario questions without memorizing.
-   [ ] I can give the 2-minute project introduction.

> **Next Phase:** Application Analysis --- how a DevOps engineer
> receives an unfamiliar repository and derives runtime, build,
> dependency, port, configuration, database, health-check and Docker
> requirements from the source code.
