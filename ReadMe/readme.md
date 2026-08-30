Enterprise Logistics & Shipment Tracking Platform --- DevOps Production Project

DHL-inspired enterprise DevOps learning project

This repository documents an enterprise-style logistics platform built
for hands-on DevOps learning. It is not a representation of DHL's
internal infrastructure.

Purpose of This Documentation

This README is the master project notebook, implementation guide,
troubleshooting runbook, and interview-preparation guide for the
complete project.

The project will be developed incrementally using the following
engineering cycle:

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

The objective is not to memorize commands. The objective is to
understand why every component exists, how it interacts with other
components, how it fails, and how to explain it confidently in a
technical interview.

1. Project Scope

The application repository contains a logistics-oriented microservices
application with components such as:

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

The project will evolve into an enterprise-style DevOps platform using:

Layer                               Technology

Source Control                      Git + GitHub

CI                                  Jenkins

Code Quality                        SonarQube

Dependency Security                 OWASP Dependency-Check

Container / IaC Security            Trivy

Containerization                    Docker

Container Registry                  Amazon ECR

Infrastructure as Code              Terraform

Cloud                               AWS

Container Platform                  Amazon EKS

Orchestration                       Kubernetes

External Traffic                    AWS ALB + Ingress

GitOps CD                           Argo CD

Metrics                             Prometheus

Visualization                       Grafana

Alerting                            Alertmanager / Slack or Email

Database                            Existing application database
dependency initially

2. Target End-to-End Architecture

flowchart TD
    DEV[Developer] --> GIT[GitHub Application Repository]
    GIT --> JENKINS[Jenkins CI]

    JENKINS --> BUILD[Compile / Build]
    BUILD --> TEST[Unit Tests]
    TEST --> SONAR[SonarQube]
    SONAR --> QG[Quality Gate]
    QG --> OWASP[OWASP Dependency Check]
    OWASP --> DBUILD[Docker Build]
    DBUILD --> TRIVY[Trivy Scan]
    TRIVY --> ECR[Amazon ECR]

    ECR --> UPDATE[Update Image Tag]
    UPDATE --> GITOPS[GitOps Repository]
    GITOPS --> ARGO[Argo CD]
    ARGO --> EKS[Amazon EKS]

    USER[Customer] --> DNS[DNS]
    DNS --> ALB[AWS Application Load Balancer]
    ALB --> INGRESS[Kubernetes Ingress]
    INGRESS --> SVC[Kubernetes Service]
    SVC --> PODS[Application Pods]

    EKS --> PROM[Prometheus]
    PROM --> GRAF[Grafana]
    PROM --> ALERT[Alertmanager]

3. Infrastructure Architecture

flowchart TD
    INTERNET[Internet]

    subgraph AWS[AWS Account]
        subgraph VPC[VPC]
            subgraph PUBLIC[Public Subnets - Multi AZ]
                ALB[Application Load Balancer]
                NAT[NAT Gateway]
            end

            subgraph PRIVATE[Private Subnets - Multi AZ]
                NODE1[EKS Worker Node]
                NODE2[EKS Worker Node]
                APP1[Application Pods]
                APP2[Application Pods]
            end
        end

        ECR[Amazon ECR]
        S3[S3 Terraform State]
        IAM[IAM Roles / Policies]
    end

    INTERNET --> ALB
    ALB --> APP1
    ALB --> APP2
    NODE1 --> NAT
    NODE2 --> NAT
    NODE1 --> APP1
    NODE2 --> APP2
    NODE1 --> ECR
    NODE2 --> ECR

Why private worker nodes?

Worker nodes host application workloads. They generally should not need
direct inbound Internet exposure.

The preferred flow is:

Internet
   ↓
Public Load Balancer
   ↓
Ingress / Target Routing
   ↓
Private Kubernetes Workloads

This reduces the attack surface while still allowing controlled
application access.

4. Application Request Flow

When a customer opens the application:

Customer Browser
       ↓
DNS Resolution
       ↓
AWS Application Load Balancer
       ↓
Ingress Routing Rule
       ↓
Kubernetes Service
       ↓
Service Endpoint
       ↓
Application Pod
       ↓
Backend / Internal Microservice
       ↓
Database or Downstream Dependency
       ↓
Response travels back to customer

WH-Method Explanation

WHAT is happening?

The customer sends an HTTP/HTTPS request to the application's domain.

WHY do we need multiple layers?

Each layer solves a different problem:

DNS translates a hostname into a reachable endpoint.

ALB accepts and distributes external traffic.

Ingress provides HTTP routing rules.

Service gives pods a stable network endpoint.

Pods execute application containers.

Backend services process business logic.

Databases persist required data.

HOW does traffic reach the correct pod?

The request is routed according to the ALB/Ingress configuration to a
Kubernetes Service. The Service selects eligible pods using
labels/selectors and forwards traffic to one of its endpoints.

WHAT IF a pod dies?

The Deployment controller works toward the declared replica count and
creates a replacement. The Service endpoint set changes so traffic is
sent only to eligible pods.

5. Application Architecture

A simplified application view:

flowchart TD
    CUSTOMER[Customer] --> FRONTEND[Frontend]
    FRONTEND --> BACKEND[Backend]

    BACKEND --> PRICE[Price Service]
    BACKEND --> AIR[Air Cargo Service]
    BACKEND --> SEA[Sea Cargo Service]
    BACKEND --> BANK[Banking Service]

    FRONTEND --> LANG[Language Service]

    BACKEND --> DB[(Database)]
    BANK --> DB

Why microservices?

Microservices separate business capabilities so services can be
developed, deployed, scaled, and troubleshot with clearer boundaries.

Benefits

Independent deployment where architecture permits it.

Independent scaling.

Smaller failure domains.

Clear ownership boundaries.

Technology flexibility.

Easier targeted observability.

Trade-offs

Network calls introduce latency and failure modes.

Distributed tracing becomes more important.

Configuration and secrets increase.

Deployment complexity increases.

Version compatibility must be managed.

Troubleshooting requires understanding service-to-service
dependencies.

6. CI vs CD vs GitOps

Continuous Integration

Developer
   ↓
Git Push
   ↓
Jenkins
   ↓
Checkout
   ↓
Build
   ↓
Unit Test
   ↓
SonarQube
   ↓
OWASP
   ↓
Docker Build
   ↓
Trivy
   ↓
ECR

CI answers:

Can we reliably build, test, scan, and publish this application
artifact?

GitOps Continuous Deployment

ECR Image
   ↓
GitOps Repository Updated
   ↓
Argo CD Detects Change
   ↓
Desired State Compared With Actual State
   ↓
Argo CD Reconciles Kubernetes

CD answers:

How does the approved artifact reach the runtime environment safely
and reproducibly?

Why Jenkins + Argo CD?

We intentionally separate responsibilities:

Jenkins → CI
Argo CD → Kubernetes CD / reconciliation

Jenkins builds and validates artifacts. Argo CD continuously compares
Kubernetes with the desired state stored in Git.

7. Security Architecture

Security is implemented in layers.

Source Code
   ↓
SonarQube

Dependencies
   ↓
OWASP Dependency-Check

Filesystem / Container Image / IaC
   ↓
Trivy

Artifact Registry
   ↓
Amazon ECR

AWS Access
   ↓
IAM Least Privilege

Kubernetes Access
   ↓
RBAC / Service Accounts

Network
   ↓
Security Groups / NetworkPolicy

Runtime
   ↓
Non-root Containers
Resource Limits
Read-only Filesystem where practical
Security Context

Secrets
   ↓
Jenkins Credentials
AWS Secrets Manager
Pod Identity / approved Kubernetes integration

8. Monitoring and Observability Architecture

flowchart LR
    APP[Application] --> PROM[Prometheus]
    PODS[Kubernetes Pods] --> PROM
    NODES[EKS Nodes] --> PROM

    PROM --> GRAF[Grafana]
    PROM --> ALERT[Alertmanager]
    ALERT --> NOTIFY[Slack / Email]

We will monitor:

CPU usage

Memory usage

Node health

Pod status

Pod restart count

Request rate

HTTP error rate

Request latency

Application availability

Deployment health

Prometheus scrape health

A dashboard is not the final objective. We must learn how to interpret
metrics during incidents.

9. Repository Strategy

The project should eventually separate three concerns.

Application Repository

dhl/
├── frontend/
├── backend/
├── banking-service/
├── language-service/
├── price-service/
├── air-cargo-service/
├── sea-cargo-service/
├── tests/
├── Dockerfiles
├── Jenkinsfile
└── README.md

Infrastructure Repository

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

GitOps Repository

dhl-gitops/
├── base/
├── environments/
│   ├── dev/
│   ├── stage/
│   └── prod/
├── argocd/
└── README.md

10. Full Training Roadmap

Phase 0 --- Project Requirements & Architecture

Learn:

Business requirements

Functional requirements

Non-functional requirements

Application architecture

DevOps architecture

CI/CD architecture

GitOps architecture

AWS architecture

Security architecture

Monitoring architecture

Networking

End-to-end traffic flow

Phase 1 --- Application Understanding

Learn how a DevOps engineer analyzes unfamiliar source code:

Programming language

Framework

Dependency files

Build files

Environment variables

Ports

Startup command

Database

APIs

Health endpoints

Logging

Goal: derive deployment requirements from source code instead of
memorizing Dockerfiles.

Phase 2 --- Git & GitHub

Cover:

clone / fetch / pull

add / commit / push

branches

merge

rebase

tags

pull requests

merge conflicts

branch protection

release/hotfix workflow

Phase 3 --- Docker

Cover:

Containers vs VMs

Docker architecture

Images and layers

Registries

Networking

Volumes

Dockerfile

Multi-stage builds

ENTRYPOINT vs CMD

Environment configuration

Non-root containers

Image optimization

Failure labs:

Container exits immediately

Wrong port

Missing dependency

Wrong ENTRYPOINT

Permission denied

Missing environment variable

Database connectivity failure

Phase 4 --- AWS Foundation

Cover:

Region / Availability Zone

VPC

CIDR

Public/private subnet

Route table

IGW

NAT Gateway

Security Group

NACL

Elastic IP

IAM

STS

Route 53

Packet flow

Phase 5 --- Terraform

Cover:

IaC

Provider

Resource

Data source

Variable

Output

Locals

State

Remote state

Locking

Modules

lifecycle

depends_on

count

for_each

import

plan/apply/destroy

Build:

VPC

Subnets

Route tables

IGW

NAT

Security Groups

IAM

ECR

EKS

Node groups

Terraform backend

Phase 6 --- Amazon ECR

Cover:

Container registry

Authentication

Tags

Immutable tags

Lifecycle policy

Scanning

Image push/pull

Phase 7 --- Kubernetes Fundamentals

Cover:

API Server

etcd

Scheduler

Controller Manager

kubelet

kube-proxy

Container runtime

Pods

ReplicaSets

Deployments

Services

ConfigMaps

Secrets

RBAC

StatefulSets

DaemonSets

Jobs

PV/PVC

HPA

PDB

NetworkPolicy

Ingress

Phase 8 --- Kubernetes YAML From Scratch

Write and understand:

Namespace

Deployment

Service

ConfigMap

Secret

ServiceAccount

HPA

PDB

Ingress

NetworkPolicy

Phase 9 --- Amazon EKS

Cover:

Managed control plane

Worker nodes

Managed node groups

VPC CNI

CoreDNS

kube-proxy

EBS CSI

AWS Load Balancer Controller

IAM integration

Scaling

Phase 10 --- ALB, Ingress & Traffic

Deep-dive:

Browser
↓
DNS
↓
ALB
↓
Listener
↓
Rule
↓
Target Group
↓
Ingress
↓
Service
↓
Endpoint
↓
Pod

Phase 11 --- Jenkins

Cover:

Controller

Agent

Executor

Workspace

Credentials

Webhooks

Declarative Pipeline

Shared Library

Jenkinsfile syntax

Phase 12 --- Complete CI

Checkout
↓
Build
↓
Unit Test
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
ECR Push
↓
GitOps Update

Phase 13 --- SonarQube

Cover:

Bugs

Vulnerabilities

Code smells

Duplication

Coverage

Technical debt

Quality profile

Quality gate

Phase 14 --- OWASP Dependency-Check

Cover:

SCA

CVE

CVSS

Direct dependencies

Transitive dependencies

False positives

Vulnerability policy

Phase 15 --- Trivy

Scan:

Filesystem

Container images

IaC/configuration where useful

Policy example:

CRITICAL → normally block pipeline
HIGH     → evaluate according to policy
MEDIUM   → track/remediate
LOW      → track according to risk

Phase 16 --- Argo CD / GitOps

Cover:

Desired state

Actual state

Sync

Health

Auto-sync

Self-heal

Prune

Drift

Rollback

Phase 17 --- Prometheus

Cover:

Metrics

Time series

Labels

Scraping

Exporters

Targets

PromQL

Alert rules

Phase 18 --- Grafana

Cover:

Data sources

Dashboards

Panels

Variables

Alerts

Interpretation

Phase 19 --- Alerting

Simulate:

Pod crash

CPU saturation

Memory pressure

Node failure

Application unavailable

HTTP 5xx spike

Latency spike

Deployment failure

Phase 20 --- Production Hardening

Implement:

requests / limits

readiness

liveness

startup probes

HPA

PDB

affinity / anti-affinity

rolling update

RBAC

NetworkPolicy

non-root containers

secret management

TLS

immutable images

Phase 21 --- HA & Scaling

Test:

Pod failure

Node failure

AZ considerations

Replica distribution

HPA

Cluster scaling

Zero-downtime rollout

Phase 22 --- Deployment Strategies

Learn:

Recreate

Rolling

Blue/Green

Canary

Phase 23 --- Secrets Management

Use:

Jenkins Credentials

AWS Secrets Manager

IAM least privilege

EKS workload identity mechanisms

Secret rotation

Phase 24 --- Observability Troubleshooting

Troubleshoot:

Load Balancer
↓
Ingress
↓
Service
↓
Endpoints
↓
Pod
↓
Resources
↓
Logs
↓
Application
↓
Database
↓
Network

Phase 25 --- Incident Simulation

Practice:

CrashLoopBackOff

ImagePullBackOff

OOMKilled

Pending

DNS failures

selector mismatch

Ingress 404

ALB unhealthy targets

IAM AccessDenied

Terraform state issues

Jenkins credential failures

Argo CD OutOfSync

Prometheus target down

Grafana no data

Phase 26 --- Production Incidents

For every incident:

SYMPTOM
↓
BLAST RADIUS
↓
RECENT CHANGES
↓
MONITORING
↓
INVESTIGATION
↓
ROOT CAUSE
↓
REMEDIATION
↓
VALIDATION
↓
PREVENTION

Phase 27 --- Final Interview Training

Cover the complete project from Linux/networking through AWS, Terraform,
Jenkins, Kubernetes, GitOps, observability and incident response.

11. Interview Answer Frameworks

Different questions require different frameworks. Do not force STAR into
every technical definition.

WH Framework --- Concept Questions

Use:

WHAT
WHY
HOW
WHERE / WHEN
WHAT IF

Example:

Question: What is Kubernetes readiness probe?

WHAT: A readiness probe determines whether a container is currently
ready to receive traffic.

WHY: A Running pod does not guarantee the application is ready. The
application may still be starting, waiting for a dependency, or
temporarily unhealthy.

HOW: kubelet executes the configured HTTP, TCP, gRPC, or command
check. When the probe fails, the pod remains running but is removed from
eligible Service endpoints.

WHERE: We use it for application workloads behind Kubernetes
Services.

WHAT IF: Without a correct readiness probe, traffic can reach an
application that is not ready, causing failed requests during startup or
degradation.

STAR Framework --- Experience / Behavioral Questions

S → Situation
T → Task
A → Action
R → Result

Use STAR when asked:

Tell me about a production issue.

Describe a difficult deployment.

Tell me about an automation you implemented.

Describe a troubleshooting situation.

Never invent employment experience. For this project say:

"In my hands-on enterprise-style DHL project..."

SPIR Framework --- Incident Questions

S → Symptom / Situation
P → Problem / Investigation
I → Intervention
R → Result / Prevention

This is useful when an interviewer wants a concise incident narrative.

12-Step Production Troubleshooting Framework

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

12. Interview Project Introduction --- Strong Answer

Question

Tell me about your DevOps project.

What the interviewer is testing

Can you explain architecture clearly?

Do you understand the tools or merely list them?

Can you explain ownership boundaries?

Do you understand CI/CD?

Do you understand networking and production operations?

Strong Answer

I built a hands-on enterprise-style logistics microservices project
inspired by a shipment platform. The application consists of a
frontend, backend and supporting services such as pricing, air-cargo,
sea-cargo, banking and language services.

I designed the infrastructure on AWS using Terraform so the
environment could be reproducible instead of manually provisioned. The
target infrastructure includes a multi-AZ VPC, public and private
subnets, routing, IAM, Amazon ECR and Amazon EKS.

For CI, I use Jenkins. A source-code change goes through checkout,
build and testing, followed by SonarQube quality analysis, dependency
scanning with OWASP Dependency-Check, container build and Trivy
security scanning. Approved images are stored in Amazon ECR.

For deployment, I separate CI from Kubernetes CD. Jenkins updates the
GitOps desired state and Argo CD reconciles that state into EKS.
Kubernetes Deployments manage application replicas, Services provide
stable service discovery, and external HTTP traffic reaches the
application through an AWS Application Load Balancer and Ingress
routing.

For observability, I use Prometheus for metrics and Grafana for
visualization, with alerts for conditions such as pod failures,
resource pressure, application errors and latency.

I also use the project to simulate production incidents such as
ImagePullBackOff, CrashLoopBackOff, service-selector problems, Ingress
routing failures, IAM AccessDenied, Argo CD drift and monitoring
failures. The objective is not only deployment; it is understanding
the complete lifecycle from infrastructure provisioning through CI,
security, GitOps, runtime traffic and troubleshooting.

Follow-up questions to expect

Why Jenkins and Argo CD together?

Why EKS instead of ECS?

Why private worker nodes?

Why ECR instead of Docker Hub?

Why Terraform modules?

How does ALB traffic reach a pod?

What happens if a pod dies?

How do you rollback?

How do you manage secrets?

How do you investigate latency?

13. Architecture Interview Questions With Answers

Q1. Why did you choose a microservices architecture?

Strong Answer

Microservices separate business capabilities into independently
manageable services. In this project, pricing, cargo, banking and
language responsibilities are separated rather than implemented as one
large process.

The benefit is clearer ownership and the ability to independently deploy
or scale appropriate components. The trade-off is distributed-system
complexity: network latency, service discovery, configuration,
observability and failure handling become more important.

Weak Answer

Microservices are modern and Kubernetes is good for microservices.

Why weak: It does not discuss the engineering trade-off.

Q2. Why Kubernetes?

WH Answer

WHAT: Kubernetes is the container orchestration layer.

WHY: Running containers manually does not solve scheduling,
self-healing, service discovery, rolling updates or declarative scaling.

HOW: We declare workloads through Kubernetes objects such as
Deployments and Services. Controllers continuously reconcile actual
state toward desired state.

WHERE: Our containerized DHL services run on EKS.

WHAT IF Kubernetes did not exist? We would need to build or operate
equivalent mechanisms for scheduling, recovery, networking, rollout and
scaling ourselves.

Q3. Why EKS instead of self-managed Kubernetes?

Strong Answer

EKS moves responsibility for operating the Kubernetes control plane to
AWS. AWS manages control-plane availability and the managed Kubernetes
API service, while we remain responsible for workload design, worker
capacity, networking, IAM integration, security policies, upgrades of
components under our control and application operations.

The trade-off is additional AWS cost and platform dependency, but it
removes a significant amount of control-plane operational burden.

Q4. Why Terraform?

Strong Answer

Terraform makes infrastructure declarative, reviewable and reproducible.
Instead of creating VPCs, subnets, IAM roles and EKS manually, the
infrastructure definition is stored as code.

We can review terraform plan before changing infrastructure, reuse
modules across environments, and maintain remote state so the team works
against a controlled infrastructure state.

Q5. Why Jenkins?

Strong Answer

Jenkins acts as the CI orchestrator. It provides a repeatable pipeline
for source checkout, build, test, quality analysis, security scanning,
image creation and publication.

The key value is not simply automation. It creates a consistent software
delivery gate so the same validation sequence is applied for every
eligible change.

Q6. Why Argo CD when Jenkins can deploy Kubernetes?

Strong Answer

Jenkins can run kubectl apply, but that creates a push-based
deployment model where the CI system needs deployment credentials and
Git may no longer represent the exact runtime state.

With GitOps, Jenkins builds the artifact and updates the desired
deployment state in Git. Argo CD continuously reconciles Git with
Kubernetes.

This gives us clearer separation:

Jenkins → artifact creation and validation
Argo CD → deployment reconciliation

It also improves auditability, drift detection and rollback workflows.

Q7. Why Amazon ECR?

ECR is the container artifact registry for the AWS environment. Jenkins
pushes approved images to ECR and EKS pulls those images during pod
creation.

Using ECR provides AWS IAM integration and keeps the registry close to
the AWS deployment platform.

Q8. Why ALB + Ingress instead of exposing every service with NodePort?

NodePort exposes a port on nodes and is not a desirable public routing
architecture for many HTTP microservices.

An ALB with Ingress allows centralized Layer-7 routing based on hostname
or URL path, TLS termination, health checking and controlled external
exposure.

Internal services can remain ClusterIP services.

Q9. Why are worker nodes private?

Worker nodes run workloads and generally do not require direct inbound
Internet exposure.

External users should enter through a controlled public endpoint such as
an ALB. Private workers reduce attack surface while outbound access,
when needed, can be provided through appropriate NAT/VPC endpoints and
routing.

14. CI/CD Interview Questions

Q10. Explain your pipeline end to end.

Git Push
↓
GitHub
↓
Jenkins Webhook Trigger
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
OWASP Dependency Check
↓
Docker Build
↓
Trivy
↓
Push to ECR
↓
Update GitOps Repository
↓
Argo CD
↓
EKS

Strong Explanation

Each stage is a gate. We try to fail as early as practical. There is no
reason to deploy an image if tests fail, and there is no reason to
publish an image that violates the agreed vulnerability policy.

Q11. SonarQube vs OWASP vs Trivy?

Tool                                Primary Role

SonarQube                           Static code quality/security
analysis

OWASP Dependency-Check              Known vulnerable dependency
analysis

Strong Answer

They overlap in some security areas but address different layers.
Security should be layered rather than relying on a single scanner.

15. Kubernetes Interview Questions

Q12. Deployment vs ReplicaSet vs Pod?

Deployment
   ↓ manages
ReplicaSet
   ↓ maintains
Pods

A Pod is the runtime unit. ReplicaSet maintains a desired number of pod
replicas. Deployment manages ReplicaSets and provides declarative
rollout/rollback behavior.

Q13. Why Service if pods already have IP addresses?

Pod IPs are ephemeral and pods are replaced routinely.

A Service provides stable discovery and selects eligible pods through
labels/selectors.

Client
↓
Service
↓
Endpoint set
↓
Pods

Q14. Running pod but application unavailable --- why?

Running describes pod/container runtime state; it does not prove the
application is usable.

Check:

Readiness
Service selector
Endpoints
Container port
Application logs
Dependencies
Network policy
Ingress

This is why readiness probes matter.

Q15. Readiness vs liveness vs startup probe?

Readiness: Should this pod receive traffic?

Liveness: Should Kubernetes restart this container because it is
unhealthy?

Startup: Has a slow-starting application completed startup before
liveness behavior should apply?

Incorrect probes can create outages, so they must reflect real
application behavior.

16. Scenario-Based Interview Questions

Scenario 1 --- HTTP 503

Question

Customers suddenly receive HTTP 503. No application deployment occurred.
What do you do?

Strong Troubleshooting Flow

1. Confirm blast radius.
2. Check monitoring and alert timeline.
3. Validate DNS/ALB reachability.
4. Check ALB target health.
5. Inspect Ingress.
6. Inspect Service.
7. Check Service endpoints.
8. Inspect pod readiness.
9. Inspect pod logs/events.
10. Check downstream dependencies.
11. Check node/resource/network conditions.
12. Remediate and validate.

Interview Answer

I would avoid restarting components immediately because that can
destroy evidence. First I would determine whether the failure affects
all users, one route or one service. Since 503 usually indicates that
the request reached a proxy/load-balancing layer but no healthy
backend could serve it, I would check ALB target health and Kubernetes
Ingress/Service endpoints. If endpoints are empty, I would inspect
selectors and readiness. If endpoints exist, I would test service
connectivity and inspect pod logs and dependencies. After identifying
the failing layer, I would apply the smallest remediation, validate
customer traffic and then add prevention based on the root cause.

Scenario 2 --- ImagePullBackOff

Investigation

kubectl get pods
kubectl describe pod <pod-name>

Inspect:

image name/tag

registry authentication

image existence

ECR permissions

network reachability

Strong Interview Answer

ImagePullBackOff means Kubernetes attempted to pull the container
image and failed, then entered backoff. I start with
kubectl describe pod because events usually show the direct registry
error. I verify the exact image URI and tag, confirm that the image
exists in ECR, and check the IAM permissions used by the node/runtime
for image pulls. I also consider network reachability to ECR. I fix
the actual pull failure rather than deleting the pod repeatedly.

Scenario 3 --- CrashLoopBackOff

Possible causes:

bad startup command

missing environment variable

dependency unavailable

permission problem

application crash

probe configuration

Investigation:

kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous

--previous is particularly useful when the container already
restarted.

Scenario 4 --- Service exists but cannot reach pods

Check:

kubectl get svc
kubectl get endpoints
kubectl get pods --show-labels
kubectl describe svc <service>

Typical root cause:

Service selector != Pod labels

This is a classic Kubernetes interview scenario.

Scenario 5 --- ALB works but application returns 404

Investigate:

ALB listener/rules
↓
Ingress host/path
↓
Ingress backend service
↓
Service port
↓
Application route

A 404 can originate from different layers. Identify which component
generated the 404 before changing configuration.

Scenario 6 --- CPU 30%, latency 8 seconds

Do not conclude that the application is healthy because CPU is low.

Investigate:

database latency

external APIs

connection pools

DNS

network latency

lock contention

thread/event-loop blocking

storage latency

request queues

downstream microservices

Strong Answer

CPU is only one saturation signal. Low CPU with high latency often
points toward waiting rather than computation. I would correlate
latency with dependency metrics, database response time, network
behavior, connection-pool usage and application traces/logs.

17. Terraform Troubleshooting Questions

Scenario --- Terraform wants to recreate resources unexpectedly

Investigate:

Did configuration change?

Did provider/version behavior change?

Was infrastructure modified manually?

Is Terraform reading the correct state/backend?

Did resource addressing change?

Was a resource renamed or moved?

Is an attribute ForceNew?

Did module structure change?

Never apply a destructive plan until the reason is understood.

Scenario --- Terraform state lock problem

Approach:

Confirm another operation is not running
↓
Identify lock owner/context
↓
Do not force-unlock blindly
↓
Resolve stale operation
↓
Use force-unlock only when confirmed safe
↓
Run plan again

The state lock protects against concurrent state modification.

18. Jenkins Troubleshooting Questions

Jenkins cannot push to ECR

Investigate:

AWS credential / role
↓
Region
↓
Account ID
↓
ECR repository
↓
Authentication
↓
IAM permissions
↓
Image tag
↓
Network

Look for errors such as:

AccessDenied
no basic auth credentials
repository does not exist
network timeout

Do not rotate credentials before understanding the actual error.

Jenkins agent offline

Check:

controller connectivity

agent pod/process

executor availability

agent logs

Kubernetes scheduling

resource pressure

DNS/network

certificate/authentication issues

19. Argo CD Troubleshooting

Synced but customers still see old version

Possible causes:

Git has wrong image tag
Image tag reused
imagePullPolicy behavior
Deployment did not roll
Ingress points elsewhere
Service selects old workload
Browser/CDN cache
Multiple environments/clusters

Strong Answer

Synced only tells me the live Kubernetes objects match the desired
Git state. It does not prove Git contains the intended application
artifact. I verify the GitOps image reference first, then inspect the
Deployment and running pod image IDs. I avoid relying only on mutable
tags such as latest; immutable version or digest references make
this much easier to reason about.

Argo CD OutOfSync

Ask:

What differs?
Who changed it?
Was it expected?
Is Git wrong or cluster state wrong?

Do not immediately press Sync without understanding the drift.

20. Monitoring Troubleshooting

Prometheus target down

Investigate:

Target URL
↓
Service discovery
↓
Service
↓
Endpoints
↓
Port
↓
Metrics path
↓
NetworkPolicy
↓
Application exporter

Grafana shows no data

Check:

Grafana datasource
↓
Prometheus availability
↓
Query
↓
Labels
↓
Time range
↓
Target scrape status

Grafana may be functioning perfectly while Prometheus has no usable
series.

21. Failure Simulation Matrix

Failure                  First Investigation

CrashLoopBackOff         describe + current/previous logs
ImagePullBackOff         pod events + image/ECR
Pending Pod              scheduler events/resources/constraints
OOMKilled                previous state + memory limits/usage
Service unavailable      Service + endpoints + selectors
Ingress 404              host/path/backend
ALB unhealthy            target health + readiness
IAM AccessDenied         exact denied action/resource/principal
Terraform drift          plan + state + manual changes
Jenkins failure          failing stage + exact log
Argo OutOfSync           desired vs live diff
Prometheus target down   scrape target/service/endpoints
Grafana no data          datasource/query/time range
High latency             RED metrics + dependencies
Node failure             node condition + pod rescheduling

22. Production Incident Answer Template

Use this template when explaining a project incident:

Situation

What system/component was affected?

Symptom

What did users or monitoring report?

Blast Radius

Was it one service, one AZ, one endpoint or the entire platform?

Investigation

What evidence did you check and in what order?

Root Cause

What actually caused the problem?

Resolution

What minimal change restored service?

Validation

How did you prove recovery?

Prevention

What was changed to reduce recurrence?

Result

What measurable or observable outcome followed?

23. STAR Example --- Kubernetes Incident

Question

Tell me about a Kubernetes issue you troubleshot.

Answer

Situation:
In my hands-on enterprise-style logistics project, I simulated an
incident where the application pods were running but requests through
the Kubernetes Service were failing.

Task:
My goal was to identify whether the problem was in the application,
Service networking, or workload configuration without restarting
components blindly.

Action:
I checked pod status first, then inspected the Service and its
endpoints. The Service had no endpoints even though the pods were
Running. I compared the Service selector with the pod labels and found
they did not match. I corrected the selector, reapplied the manifest and
verified that endpoints were populated. I then tested connectivity
through the Service.

Result:
Traffic was restored without changing the application. The exercise
demonstrated that a Running pod does not guarantee service availability
and that Service endpoints are a critical checkpoint when debugging
Kubernetes networking.

Follow-up interviewer questions

Why didn't Kubernetes report the Service as failed?

What creates EndpointSlices?

What if endpoints existed but traffic still failed?

What would NetworkPolicy change?

How would readiness affect endpoints?

24. STAR Example --- CI Security Failure

Question

Tell me about a pipeline failure you handled.

Answer

Situation:
During my enterprise-style DHL project, I configured security gates in
Jenkins and intentionally tested a build containing a dependency/image
vulnerability.

Task:
The objective was to ensure insecure artifacts could not silently
continue toward deployment and to understand how to investigate scanner
findings.

Action:
I identified the exact stage that failed, reviewed the vulnerability
report, checked the CVE severity and affected package, and determined
whether an upgraded dependency/base image was available. I rebuilt the
artifact after remediation and reran the scan instead of bypassing the
gate.

Result:
The corrected image passed the defined policy before being published.
The exercise reinforced the principle that security scanning should act
as an enforceable delivery control rather than a report that engineers
ignore.

25. Architecture Challenge Questions

Practice answering these without memorizing paragraphs:

Why EKS instead of ECS?

Why ALB instead of NLB?

Why ClusterIP for internal services?

Why separate public/private subnets?

Why NAT Gateway?

Can we avoid NAT cost?

Why Terraform modules?

Why remote Terraform state?

Why should state be protected?

Why Jenkins rather than GitHub Actions?

Why Argo CD if Jenkins exists?

Why ECR rather than Docker Hub?

Why immutable image tags?

Why readiness probes?

Why liveness probes?

Why resource requests?

Why resource limits?

Why HPA?

Why PDB?

Why anti-affinity?

Why NetworkPolicy?

Why Prometheus if CloudWatch exists?

Why Grafana?

Why Alertmanager?

Why multi-AZ infrastructure?

What happens when one node dies?

What happens when one pod dies?

What happens when an AZ fails?

What happens if Argo CD is temporarily unavailable?

What happens if Jenkins is unavailable after an image was already
published?

26. Interviewer Cross-Question Strategy

When you make a statement, expect the interviewer to attack the next
layer.

If you say:

"We used HPA."

Expect:

What metric?

What threshold?

How are resource requests related to CPU HPA?

What happens before a new pod is Ready?

What if the cluster has no node capacity?

HPA vs Cluster Autoscaler?

What if the bottleneck is the database?

If you say:

"We used ALB."

Expect:

Why ALB?

Which OSI layer?

ALB vs NLB?

What is a target group?

How are health checks configured?

How does the AWS Load Balancer Controller work?

Where does TLS terminate?

If you say:

"We used Terraform remote state."

Expect:

Why remote?

Who can access it?

How do you prevent concurrent writes?

What if state is lost?

Why is state sensitive?

How do you recover from drift?

27. What NOT to Say in Interviews

Avoid:

"We used Kubernetes because it is industry standard."

Better:

"We needed declarative workload management, service discovery,
self-healing, controlled rollouts and horizontal scaling for multiple
containerized services."

Avoid:

"Argo CD deploys automatically."

Better:

"Argo CD continuously compares desired state in Git with live
Kubernetes state and reconciles approved differences according to the
configured sync policy."

Avoid:

"Pod was down, so I restarted it."

Better:

"I first checked pod status, events and previous container logs to
identify why the workload failed before deciding on remediation."

Avoid claiming production employment experience that came only from this
project.

Use:

"In my hands-on enterprise-style project, I implemented..."

or:

"I simulated a production scenario where..."

28. Command Explanation Standard

Every implementation phase should document commands in this format:

COMMAND

kubectl get pods -n <namespace>

WHAT IT DOES

Queries the Kubernetes API for Pod objects in the specified namespace.

WHAT TO INSPECT

STATUS

READY

RESTARTS

AGE

EXPECTED OUTPUT

Healthy workloads should normally show expected readiness such as:

READY   STATUS    RESTARTS
1/1     Running   0

IF IT FAILS

If STATUS shows:

CrashLoopBackOff → inspect logs and previous logs.

ImagePullBackOff → inspect events/image registry.

Pending → inspect scheduler events.

Running but 0/1 → investigate readiness/application health.

NEXT CHECK

kubectl describe pod <pod-name> -n <namespace>

29. Final Project Validation Checklist

Application architecture understood

Services run locally

Dockerfiles understood and validated

Images built successfully

AWS networking understood

Terraform remote backend configured

VPC provisioned

ECR provisioned

EKS provisioned

Kubernetes manifests written and understood

Application deployed to EKS

ALB/Ingress routing validated

Jenkins CI implemented

Unit tests integrated

SonarQube integrated

Quality Gate enforced

OWASP scanning integrated

Trivy scanning integrated

Images pushed to ECR

GitOps repository configured

Argo CD configured

GitOps deployment validated

Prometheus installed/configured

Grafana dashboards created

Alerts validated

Resource requests/limits configured

Health probes configured

HPA tested

PDB understood/tested

RBAC reviewed

Secrets handling reviewed

Network security reviewed

Rollback tested

Failure scenarios simulated

Troubleshooting runbook completed

Final architecture explanation practiced

Mock interview completed

30. Required Failure Labs

Before declaring the project complete, intentionally reproduce and
troubleshoot:

Jenkins pipeline failure

SonarQube Quality Gate failure

Vulnerable dependency

Trivy CRITICAL finding

ECR push failure

ImagePullBackOff

CrashLoopBackOff

OOMKilled

Service selector mismatch

Ingress 404

ALB health-check failure

Argo CD OutOfSync

Terraform state/locking issue

IAM AccessDenied

Prometheus target down

Grafana no data

High application latency

Node failure

Rolling deployment problem

Production-style rollback

Every lab must document:

SYMPTOM
POSSIBLE CAUSES
INVESTIGATION
COMMANDS
EVIDENCE
ROOT CAUSE
FIX
VALIDATION
PREVENTION
INTERVIEW ANSWER

31. Final 5--10 Minute Interview Story Structure

Use this order:

1. Business Problem
2. Application Architecture
3. AWS Infrastructure
4. Terraform
5. Docker
6. CI
7. DevSecOps
8. ECR
9. Kubernetes / EKS
10. GitOps / Argo CD
11. ALB / Ingress Traffic
12. Monitoring
13. Security
14. Scaling / HA
15. Troubleshooting
16. Challenges
17. Lessons Learned

Do not list tools without explaining relationships.

The core story is:

Terraform creates the platform.
Docker packages the services.
Jenkins validates and builds artifacts.
SonarQube/OWASP/Trivy provide delivery security gates.
ECR stores approved images.
Git stores desired deployment state.
Argo CD reconciles that state.
EKS runs the workloads.
ALB/Ingress expose the application.
Prometheus observes it.
Grafana makes the signals usable.
Alerting surfaces abnormal conditions.
Troubleshooting connects all of these layers.

32. Final Engineering Principle

The project is considered successful only when the architecture can be:

Designed
↓
Built
↓
Explained
↓
Operated
↓
Broken
↓
Diagnosed
↓
Recovered
↓
Improved

A DevOps engineer should be able to explain not only:

"What command did you run?"

but also:

"Why did you run that command, which layer were you testing, what
evidence did you expect, what did the result eliminate, and what was
your next diagnostic step?"

That is the standard this project will follow.

33. Documentation Update Rule for Every Future Phase

For every new phase, append these sections to this README:

# Phase / Day

## Introduction
## Business / Technical Requirement
## What
## Why
## How
## Architecture
## Workflow
## Prerequisites
## Repository Structure
## Implementation
## File-by-File Explanation
## Commands
## Expected Output
## Validation
## Failure Lab
## Troubleshooting Decision Tree
## Production Considerations
## Security Considerations
## Cost Considerations
## Common Mistakes
## Basic Interview Questions
## Intermediate Interview Questions
## Advanced Interview Questions
## Scenario Questions
## Troubleshooting Questions
## Architecture Questions
## Strong Answers
## Follow-up / Cross Questions
## STAR Project Example
## Key Takeaways

This file is therefore a living project document and should grow
with the implementation rather than being replaced by disconnected
notes.
