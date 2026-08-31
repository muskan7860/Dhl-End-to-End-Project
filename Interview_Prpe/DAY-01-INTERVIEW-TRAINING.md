# DAY 1 — Project Kickoff & Architecture Interview Training

> [!IMPORTANT]
> **Target:** DevOps Engineer interview preparation at approximately 4
> years of experience.  
> **Project context:** Enterprise-style DHL-inspired Logistics &
> Shipment Tracking Platform.  
> These answers are written so they can be **spoken naturally in an
> interview**, not memorized as textbook definitions.

------------------------------------------------------------------------

## Table of Contents

- [Day 1 Learning Scope](#day-1-learning-scope)
- [Architecture](#architecture)
- [Section 1 — Basic Interview
  Questions](#section-1--basic-interview-questions)
- [Section 2 — Normal / Practical
  Questions](#section-2--normal--practical-questions)
- [Section 3 — Mid-Level Questions](#section-3--mid-level-questions)
- [Section 4 — Advanced Questions](#section-4--advanced-questions)
- [Section 5 — Architecture
  Questions](#section-5--architecture-questions)
- [Section 6 — Troubleshooting
  Questions](#section-6--troubleshooting-questions)
- [Section 7 — Scenario-Based
  Questions](#section-7--scenario-based-questions)
- [Section 8 — Project-Based
  Questions](#section-8--project-based-questions)
- [Section 9 — Cross-Questions](#section-9--cross-questions)
- [Section 10 — STAR Answers](#section-10--star-answers)
- [Section 11 — Rapid-Fire Revision](#section-11--rapid-fire-revision)
- [Day 1 Completion Checklist](#day-1-completion-checklist)

------------------------------------------------------------------------

# Day 1 Learning Scope

Today focuses on understanding the **complete system before
implementation**.

You should be able to explain:

1.  What business problem the application solves.
2.  Why the application contains multiple services.
3.  Application architecture vs DevOps architecture.
4.  Why services are containerized.
5.  Why a container registry such as Amazon ECR is required.
6.  Why Terraform is used instead of relying only on the AWS Console.
7.  Continuous Integration.
8.  Continuous Delivery/Deployment.
9.  Jenkins vs Argo CD responsibilities.
10. SonarQube vs OWASP Dependency-Check vs Trivy.
11. Why EKS worker nodes are normally private.
12. Purpose of AWS Application Load Balancer.
13. ALB vs Ingress vs Kubernetes Service.
14. Why Kubernetes Services sit in front of pods.
15. Complete customer request flow.
16. How to troubleshoot failures across this architecture.

------------------------------------------------------------------------

# Architecture

## Application Architecture

``` mermaid
flowchart TD
    USER["Customer"] --> FE["Frontend"]
    FE --> BE["Backend"]
    FE --> LANG["Language Service"]

    BE --> PRICE["Price Service"]
    BE --> AIR["Air Cargo Service"]
    BE --> SEA["Sea Cargo Service"]
    BE --> BANK["Banking Service"]

    BE --> DB[("Database")]
    BANK --> DB
```

## Complete DevOps Architecture

``` mermaid
flowchart TD
    DEV["Developer"] --> GH["GitHub"]
    GH --> JENKINS["Jenkins CI"]

    JENKINS --> BUILD["Build"]
    BUILD --> TEST["Unit Tests"]
    TEST --> SONAR["SonarQube"]
    SONAR --> QG{"Quality Gate"}
    QG -->|Pass| OWASP["OWASP Dependency-Check"]
    OWASP --> DOCKER["Docker Build"]
    DOCKER --> TRIVY["Trivy Image Scan"]
    TRIVY --> ECR["Amazon ECR"]

    ECR --> UPDATE["Update GitOps Image Version"]
    UPDATE --> GITOPS["GitOps Repository"]
    GITOPS --> ARGO["Argo CD"]
    ARGO --> EKS["Amazon EKS"]

    CUSTOMER["Customer"] --> DNS["DNS"]
    DNS --> ALB["AWS Application Load Balancer"]
    ALB --> ING["Kubernetes Ingress"]
    ING --> SVC["Kubernetes Service"]
    SVC --> POD["Application Pod"]

    EKS --> PROM["Prometheus"]
    PROM --> GRAF["Grafana"]
    PROM --> ALERT["Alertmanager"]
```

## Customer Request Flow

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
Frontend Pod
       ↓
Backend / Internal Service
       ↓
Database / Dependency
       ↓
Response
```

------------------------------------------------------------------------

# Section 1 — Basic Interview Questions

## Q1. What business problem does this DHL-style application solve?

### Interview Answer

> This is an enterprise-style logistics and shipment platform. At a high
> level, it represents capabilities such as customer-facing shipment
> operations, pricing, air cargo, sea cargo, banking-related operations
> and language functionality. The application is useful for learning how
> multiple business capabilities can be delivered as cooperating
> services and then operated through a complete DevOps platform.

### Remember

``` text
Business Problem
    ↓
Logistics Operations
    ↓
Multiple Business Capabilities
    ↓
Multiple Services
    ↓
DevOps Platform
```

### Cross-question: Is this DHL's actual architecture?

> No. It is a DHL-inspired hands-on enterprise-style project. I use it
> to demonstrate architecture, automation, Kubernetes, AWS, CI/CD,
> security and troubleshooting concepts.

------------------------------------------------------------------------

## Q2. Why does the application contain multiple services instead of one application?

### Interview Answer

> Different business capabilities are separated into services so they
> can have clearer boundaries. For example, pricing, banking and cargo
> functionality do not necessarily need to be implemented as one tightly
> coupled codebase. Service separation can support independent
> development, deployment and scaling where appropriate. The trade-off
> is increased distributed-system complexity, including networking,
> service discovery, observability and failure handling.

### Do not say

> Microservices are always better than monoliths.

### Better answer

> The architecture should be selected based on business and operational
> requirements. Microservices provide useful independence, but they also
> introduce operational complexity.

------------------------------------------------------------------------

## Q3. What is application architecture?

> Application architecture explains how application components are
> structured and how they communicate. In this project that includes the
> frontend, backend, price service, cargo services, banking service,
> language service and database dependencies.

------------------------------------------------------------------------

## Q4. What is DevOps architecture?

> DevOps architecture explains how the application moves from source
> code to a secure, deployed and observable runtime environment. It
> includes GitHub, Jenkins, security gates, Docker, ECR, Terraform, EKS,
> Argo CD, ALB/Ingress and Prometheus/Grafana.

------------------------------------------------------------------------

## Q5. What is the difference between application architecture and DevOps architecture?

| Application Architecture        | DevOps Architecture                    |
|---------------------------------|----------------------------------------|
| Business/application components | Software delivery and runtime platform |
| Frontend/backend/services       | Git/Jenkins/ECR/EKS/Argo CD            |
| Service communication           | CI/CD and deployment                   |
| Business dependencies           | Infrastructure/security/monitoring     |

### Interview Answer

> Application architecture answers **how the software itself is
> structured**. DevOps architecture answers **how that software is
> built, secured, packaged, deployed, operated and monitored**.

------------------------------------------------------------------------

## Q6. Why containerize each service?

### Interview Answer

> Containerization packages the application together with its required
> runtime and dependencies into a consistent image. This gives CI and
> Kubernetes a predictable artifact and reduces environment differences
> between development, testing and deployment environments. Each
> independently deployable service can have its own image and version.

------------------------------------------------------------------------

## Q7. What is Amazon ECR?

> Amazon ECR is AWS's managed container registry. We use it to store
> versioned container images produced by the CI pipeline so EKS can
> later pull the approved image for deployment.

------------------------------------------------------------------------

## Q8. Why do we need ECR if Jenkins already built the image?

> Jenkins is the build/orchestration system, not the long-term container
> artifact store. Kubernetes nodes need a registry from which they can
> pull the required image. ECR becomes the controlled source for
> deployable container artifacts.

------------------------------------------------------------------------

## Q9. What is Terraform?

> Terraform is an Infrastructure as Code tool. It allows us to describe
> infrastructure such as VPCs, subnets, IAM resources, ECR and EKS in
> code and manage that infrastructure through a repeatable workflow.

------------------------------------------------------------------------

## Q10. What is CI?

> Continuous Integration is the practice of frequently integrating code
> changes and automatically validating them through steps such as build,
> unit tests, quality analysis and security checks.

``` text
Code
 ↓
Build
 ↓
Test
 ↓
Quality
 ↓
Security
 ↓
Artifact
```

------------------------------------------------------------------------

## Q11. What is CD?

> CD refers to the process of moving validated application changes
> toward deployment. In our target GitOps architecture, the desired
> Kubernetes deployment state is stored in Git and Argo CD reconciles
> that state into EKS.

------------------------------------------------------------------------

# Section 2 — Normal / Practical Questions

## Q12. Why Terraform when AWS Console can create the same infrastructure?

### Interview Answer

> The AWS Console is useful for learning, inspection and certain
> operational tasks, but manual infrastructure creation is difficult to
> reproduce consistently. Terraform allows infrastructure to be
> version-controlled, reviewed, reused and recreated. Before applying
> changes I can inspect a plan, and modules can standardize
> infrastructure patterns across environments.

### Cross-question: Do you never use AWS Console?

> I still use the Console for visibility, verification and
> troubleshooting. Infrastructure provisioning should preferably follow
> the controlled IaC workflow instead of relying on undocumented manual
> changes.

------------------------------------------------------------------------

## Q13. Why Jenkins?

> Jenkins orchestrates our CI workflow. It checks out code, executes
> build and tests, invokes security/quality tools, builds the Docker
> image and publishes the approved artifact to ECR.

------------------------------------------------------------------------

## Q14. Why Argo CD?

> Argo CD implements GitOps for Kubernetes. It compares the desired
> state stored in Git with the live state in EKS and reconciles
> differences according to the configured synchronization policy.

------------------------------------------------------------------------

## Q15. Why Jenkins and Argo CD together instead of Jenkins doing everything?

### Interview Answer

> Jenkins handles artifact creation and validation, while Argo CD
> handles Kubernetes reconciliation. Jenkins can technically run
> `kubectl` or Helm directly, but separating CI and CD means Git remains
> the desired-state source of truth. This improves auditability and
> makes configuration drift visible.

``` text
Jenkins
  ↓
Build / Test / Scan
  ↓
ECR
  ↓
Update GitOps
  ↓
Git
  ↓
Argo CD
  ↓
EKS
```

------------------------------------------------------------------------

## Q16. SonarQube vs OWASP Dependency-Check vs Trivy?

| Tool                   | Primary Purpose                                                       |
|------------------------|-----------------------------------------------------------------------|
| SonarQube              | Source-code quality and static analysis                               |
| OWASP Dependency-Check | Known vulnerable third-party dependencies                             |
| Trivy                  | Container images, packages, filesystem and configuration/IaC scanning |

### Interview Answer

> I use them at different layers. SonarQube analyzes source-code quality
> and relevant static-analysis issues. OWASP Dependency-Check focuses on
> known vulnerabilities in dependencies. Trivy gives additional coverage
> for container images, packages, filesystems and
> infrastructure/configuration. One scanner should not be treated as
> complete application security.

------------------------------------------------------------------------

## Q17. Why should EKS worker nodes normally remain private?

> Worker nodes host application workloads and normally do not need
> arbitrary direct inbound Internet access. Customer traffic should
> enter through controlled entry points such as an ALB. Keeping workers
> private reduces attack surface. Approved outbound connectivity can be
> provided through NAT or appropriate VPC endpoints depending on the
> architecture.

------------------------------------------------------------------------

## Q18. What is the purpose of an AWS ALB?

> An Application Load Balancer provides Layer-7 HTTP/HTTPS load
> balancing. It can accept external application traffic, perform
> listener-based routing and health checks, and distribute traffic
> toward healthy application targets.

------------------------------------------------------------------------

# Section 3 — Mid-Level Questions

## Q19. What is the difference between ALB, Ingress and Kubernetes Service?

### ALB

AWS infrastructure that receives and distributes Layer-7 HTTP/HTTPS
traffic.

### Ingress

Kubernetes API object representing HTTP/HTTPS routing rules.

### Service

Stable Kubernetes networking abstraction for a dynamic set of pods.

### Interview Answer

> They operate at different layers. ALB is the AWS load-balancing
> resource. Ingress expresses Kubernetes HTTP routing rules. A Service
> provides stable routing/discovery for the backend pods. The exact AWS
> implementation depends on the controller and target configuration, but
> conceptually the request moves from the external entry point toward
> the Kubernetes workload through these routing layers.

------------------------------------------------------------------------

## Q20. Why does a Kubernetes Service sit between clients and pods?

> Pods are ephemeral. They can be replaced during failures, scaling or
> deployments, which means individual pod identities and IPs should not
> be treated as a permanent client endpoint. A Service provides a stable
> abstraction and routes to eligible backend endpoints.

------------------------------------------------------------------------

## Q21. What happens if one frontend pod dies?

> A controller such as a Deployment/ReplicaSet works toward the desired
> replica count and creates a replacement. Readiness determines whether
> a pod should receive traffic. The Service abstraction prevents clients
> from depending on one fixed pod IP.

------------------------------------------------------------------------

## Q22. What is desired state?

> Desired state is the configuration we declare—for example, three
> replicas of a particular application image. Kubernetes controllers
> continuously compare that desired state with actual state and take
> actions to reduce differences.

------------------------------------------------------------------------

## Q23. What is reconciliation?

> Reconciliation is the control-loop process of comparing desired and
> actual state and acting when they differ. Kubernetes controllers
> reconcile cluster resources, and Argo CD performs a similar comparison
> between Git desired state and the live Kubernetes environment.

------------------------------------------------------------------------

# Section 4 — Advanced Questions

## Q24. Why EKS instead of ECS?

### Interview Answer

> Both are valid AWS container platforms. ECS is simpler for many
> AWS-native workloads. EKS is selected here because Kubernetes itself
> is a core requirement: we want to implement and troubleshoot
> Deployments, Services, Ingress, RBAC, HPA, PDB, GitOps and the
> Kubernetes ecosystem. In a real architecture decision, I would compare
> requirements, operational complexity, team skills and cost rather than
> automatically choosing Kubernetes.

------------------------------------------------------------------------

## Q25. Why not expose every microservice through an ALB?

> Most internal microservices do not need direct Internet exposure.
> Exposing unnecessary services increases attack surface and routing
> complexity. Public traffic should normally enter through explicitly
> required application entry points while internal services communicate
> through private service networking.

------------------------------------------------------------------------

## Q26. Why use immutable image versions?

> Immutable image tags or digests make a deployment traceable. If the
> same mutable tag such as `latest` is reused for different builds, it
> becomes harder to know exactly what code is running and rollback
> behavior becomes less deterministic.

------------------------------------------------------------------------

## Q27. Why multi-AZ?

> Multiple Availability Zones reduce dependency on a single AWS failure
> domain. But HA requires more than creating subnets in two AZs. Worker
> capacity, application replicas, pod placement, load balancing and data
> dependencies must also tolerate an AZ failure.

------------------------------------------------------------------------

## Q28. Why is a successful Jenkins pipeline not proof that production is healthy?

> CI proves that the configured build, test and validation stages
> succeeded. Runtime health depends on deployment state, Kubernetes
> scheduling, configuration, secrets, networking, dependencies, traffic
> routing and application behavior. CI success is one checkpoint, not
> end-to-end production validation.

------------------------------------------------------------------------

# Section 5 — Architecture Questions

## Q29. Explain your complete architecture.

### Interview Answer

> A developer pushes code to GitHub, which triggers Jenkins CI. Jenkins
> performs build and unit tests, then invokes quality and security gates
> using SonarQube, OWASP Dependency-Check and Trivy. After validation,
> Jenkins publishes the container image to Amazon ECR.
>
> The deployment version is represented in the GitOps repository. Argo
> CD monitors that repository and reconciles the desired Kubernetes
> state into Amazon EKS.
>
> For application traffic, the customer reaches the public application
> endpoint through DNS and AWS ALB. Kubernetes routing then directs
> traffic toward the appropriate Service and ready application pods.
> Prometheus collects runtime metrics and Grafana provides dashboards,
> while alerts can be routed through Alertmanager.
>
> Terraform is responsible for reproducibly provisioning the required
> AWS infrastructure.

------------------------------------------------------------------------

## Q30. What happens from entering the DHL URL until the frontend pod receives the request?

### Interview Answer

> First, DNS resolves the application hostname. The browser establishes
> the required network/TLS connection to the external application
> endpoint. The AWS Application Load Balancer receives the HTTP/HTTPS
> request and evaluates its configured listener and routing rules.
> Kubernetes ingress-related configuration identifies the intended
> backend. The request is directed toward the appropriate Kubernetes
> backend, and the Service abstraction represents the frontend workload.
> An eligible ready frontend pod ultimately processes the request.
>
> If this flow fails, I troubleshoot it hop by hop instead of starting
> directly at the pod.

------------------------------------------------------------------------

## Q31. Where is the source of truth?

> We have different sources of truth for different concerns. Application
> source is stored in the application repository. Terraform code
> represents the intended infrastructure configuration. The GitOps
> repository represents the intended Kubernetes deployment state. ECR
> stores the immutable application artifact.

------------------------------------------------------------------------

# Section 6 — Troubleshooting Questions

## Q32. Pods are Running, but users cannot access the application. What do you check?

``` text
DNS
 ↓
ALB
 ↓
Target Health
 ↓
Ingress
 ↓
Service
 ↓
Endpoints
 ↓
Readiness
 ↓
Pod
 ↓
Application
```

### Interview Answer

> `Running` only tells me that the pod has been scheduled and its
> container runtime state; it does not prove application readiness or
> reachability. I check readiness, Service selectors and endpoints,
> application logs, Ingress routing, ALB target health and dependencies.
> I use the observed symptom to identify the failing layer.

------------------------------------------------------------------------

## Q33. Service exists but has no endpoints. What could be wrong?

Possible causes:

- Service selector does not match pod labels.
- Eligible pods are not Ready.
- Workload is missing.
- Labels changed.
- Wrong configuration assumptions.

### Commands

``` bash
kubectl get svc -n <namespace>
kubectl describe svc <service-name> -n <namespace>
kubectl get pods -n <namespace> --show-labels
kubectl get endpoints -n <namespace>
```

### Interview Answer

> I compare the Service selector with actual pod labels and inspect pod
> readiness. A Service can exist successfully while having no usable
> backend endpoints.

------------------------------------------------------------------------

## Q34. Jenkins built the image but EKS cannot pull it. How do you troubleshoot?

``` text
Pod Event
 ↓
Exact Image URI
 ↓
Tag/Digest Exists?
 ↓
ECR Access
 ↓
IAM
 ↓
Network Connectivity
```

### Commands

``` bash
kubectl describe pod <pod-name> -n <namespace>
kubectl get pod <pod-name> -n <namespace> -o yaml
```

> I begin with pod events to capture the exact registry error. Then I
> verify the image URI/tag, confirm the image exists, validate the
> relevant authorization and verify that the node/workload path can
> reach the required registry endpoints.

------------------------------------------------------------------------

## Q35. Why should you not restart everything immediately?

> A restart may temporarily remove the symptom while destroying useful
> evidence. Unless incident severity requires immediate recovery, I
> first capture logs, events, metrics and state, identify the failing
> layer and then apply the minimum remediation.

------------------------------------------------------------------------

# Section 7 — Scenario-Based Questions

## Scenario 1 — HTTP 503

**Interviewer:** Customers suddenly receive `503 Service Unavailable`.
What will you do?

### Strong Answer

> First I determine the blast radius: all users, one hostname, one path,
> one service or one AZ. I correlate the start time with deployments,
> infrastructure changes and monitoring.
>
> Because 503 often means a proxy/load-balancing layer cannot route to a
> usable backend, I inspect ALB target health, Ingress/controller state,
> Service configuration, endpoints and pod readiness. If endpoints are
> empty, I inspect selectors and readiness. If endpoints exist, I test
> connectivity and inspect application and dependency health.
>
> Once I have evidence for the root cause, I apply the minimum
> remediation, validate customer traffic and record a prevention action.

------------------------------------------------------------------------

## Scenario 2 — Application works through Service but not externally

### Evidence

``` text
Pod             ✓
Service         ✓
Internal curl   ✓
External URL    ✗
```

### Answer

> Because the internal path works, I narrow the investigation to the
> external routing layer. I inspect Ingress configuration/controller
> state, ALB listener/rules, target group health, health-check
> configuration, security groups and DNS. I would not start by modifying
> application code because the evidence already shows that the internal
> application path works.

------------------------------------------------------------------------

## Scenario 3 — Argo CD is Synced but old application version is visible

> `Synced` only proves that live Kubernetes state matches Git's desired
> manifests. I verify the image version stored in Git, Deployment image,
> rollout status and actual running pod image ID/digest. I also verify
> whether a mutable tag was reused. Using immutable versions makes this
> easier to diagnose.

------------------------------------------------------------------------

## Scenario 4 — One AZ fails

### Answer

> I expect the load-balancing and compute design to continue serving
> traffic from healthy capacity in another AZ if sufficient replicas and
> capacity exist there. I verify ALB target health, node availability,
> pod rescheduling, topology/placement and application dependencies.
> Multi-AZ subnets alone do not guarantee HA.

------------------------------------------------------------------------

## Scenario 5 — CPU is normal but application latency is high

> Low CPU means the workload is not obviously CPU-saturated. Latency can
> come from a database, downstream service, DNS, network, connection
> pool, lock, storage or application logic. I correlate latency with
> dependency metrics/logs and trace the slow request path before
> deciding to scale.

------------------------------------------------------------------------

# Section 8 — Project-Based Questions

## Q36. What exactly did you implement in this project?

### Safe Interview Wording

> In my hands-on enterprise-style project, I am implementing the
> complete delivery platform around an existing multi-service logistics
> application. My scope includes understanding the source,
> containerization, AWS infrastructure through Terraform, Jenkins CI,
> security scanning, ECR, EKS, Kubernetes configuration, ALB/Ingress
> routing, Argo CD GitOps and Prometheus/Grafana observability. I am
> also documenting and reproducing failure scenarios to strengthen
> troubleshooting skills.

------------------------------------------------------------------------

## Q37. What is the most important thing you learned from the architecture phase?

> The biggest lesson is to understand responsibilities and traffic flow
> before implementing tools. If I know which component owns each
> responsibility and how requests/artifacts move through the system,
> troubleshooting becomes systematic rather than command-driven.

------------------------------------------------------------------------

# Section 9 — Cross-Questions

These are the follow-up questions an interviewer may ask after your
initial answer.

### Terraform

- What is Terraform state?
- Why use remote state?
- How do you prevent concurrent state changes?
- What happens if someone changes AWS resources manually?

### EKS

- Who manages the control plane?
- Who manages worker nodes?
- How does a private node pull from ECR?
- How do pods get IP addresses?
- What happens when a node fails?

### Kubernetes Networking

- Service vs Ingress?
- ClusterIP vs NodePort vs LoadBalancer?
- What are endpoints/EndpointSlices?
- What happens if Service selectors are wrong?
- What does readiness have to do with traffic?

### CI/CD

- What happens if SonarQube fails?
- Should a CRITICAL vulnerability stop the pipeline?
- Who updates the GitOps repository?
- Why not deploy `latest`?
- How do you roll back?

### Security

- IAM vs RBAC?
- Where do secrets live?
- How do you prevent credentials from entering Git?
- Why are private subnets not enough by themselves?

------------------------------------------------------------------------

# Section 10 — STAR Answers

## STAR — Designing the DevOps Architecture

### Situation

I wanted to convert a multi-service logistics application into a
complete enterprise-style DevOps learning project instead of only
running the application locally.

### Task

I needed to design an end-to-end platform covering CI, security,
container artifacts, AWS infrastructure, Kubernetes deployment, GitOps,
traffic management and observability.

### Action

I separated responsibilities across the architecture. Terraform handles
AWS infrastructure, Jenkins handles CI, SonarQube/OWASP/Trivy provide
delivery gates, ECR stores approved container images, EKS runs
Kubernetes workloads, Argo CD handles GitOps reconciliation, ALB/Ingress
handles external routing, and Prometheus/Grafana provides monitoring. I
also defined the complete request path and planned failure simulations
for each layer.

### Result

I created a structured implementation roadmap where each tool has a
specific responsibility. This also gives me a systematic troubleshooting
model because I can isolate failures by layer rather than randomly
executing commands.

------------------------------------------------------------------------

# Section 11 — Rapid-Fire Revision

Try answering each in **20–30 seconds**.

1.  What problem does the application solve?
2.  Why microservices?
3.  Application architecture vs DevOps architecture?
4.  Why Docker?
5.  Why ECR?
6.  Why Terraform?
7.  What is CI?
8.  What is CD?
9.  Jenkins vs Argo CD?
10. SonarQube vs OWASP vs Trivy?
11. Why private worker nodes?
12. Why ALB?
13. ALB vs Ingress vs Service?
14. Why Service in front of pods?
15. What is desired state?
16. What is reconciliation?
17. What happens if a pod dies?
18. What happens if Service selectors are wrong?
19. How would you investigate HTTP 503?
20. How does Browser → Pod traffic work?

------------------------------------------------------------------------

# Day 1 Completion Checklist

Before moving to the next day, I should be able to:

- [ ] Draw the application architecture without looking.
- [ ] Draw the complete DevOps architecture without looking.
- [ ] Explain every major component in one sentence.
- [ ] Explain the business problem.
- [ ] Explain why multiple services exist.
- [ ] Explain Docker and ECR.
- [ ] Explain Terraform vs manual AWS provisioning.
- [ ] Explain CI and CD.
- [ ] Explain Jenkins vs Argo CD.
- [ ] Explain SonarQube vs OWASP vs Trivy.
- [ ] Explain private EKS worker nodes.
- [ ] Explain ALB vs Ingress vs Service.
- [ ] Trace Browser → ALB → Ingress → Service → Pod.
- [ ] Troubleshoot a Service with no endpoints.
- [ ] Explain how I would investigate HTTP 503.
- [ ] Answer interviewer cross-questions without memorizing scripts.
- [ ] Give the architecture STAR answer naturally.

------------------------------------------------------------------------

# Day 1 Golden Rule

> **Do not memorize tools. Understand responsibilities, dependencies and
> failure paths.**

For every component, be ready to answer:

``` text
WHAT is it?
WHY do we need it?
HOW does it work?
WHERE is it used?
WHAT happens if it fails?
HOW will I troubleshoot it?
WHY did I choose it instead of an alternative?
```

------------------------------------------------------------------------

## Next

**DAY 2 — Application Understanding**

The next step is to inspect the actual application repository and
determine:

``` text
Source Code
    ↓
Language / Framework
    ↓
Dependencies
    ↓
Build Command
    ↓
Startup Command
    ↓
Ports
    ↓
Environment Variables
    ↓
Service Dependencies
    ↓
Database
    ↓
Health Endpoints
    ↓
Docker Requirements
    ↓
Kubernetes Requirements
```
