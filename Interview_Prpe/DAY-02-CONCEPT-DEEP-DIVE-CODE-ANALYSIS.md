# DAY 2 — Deep-Dive Application & Code Analysis

> **Goal:** Learn exactly how a DevOps engineer analyzes an unfamiliar
> repository before Docker, CI/CD, Kubernetes or AWS deployment.
>
> **Core rule:** **Do not automate what you do not understand.**

## 1. What Are We Trying to Produce?

We are not reading every line like an application developer. We are
extracting a **deployment contract** for every service:

``` text
Service → Language/Framework → Dependencies → Build → Test → Start
→ Port → Configuration → Secrets → Database → Downstream Services
→ API Routes → Health → Failure Behavior → Deployment Requirements
```

That contract becomes the input for Docker, Jenkins, Kubernetes,
security and monitoring.

## 2. Why Source Analysis Comes First

A wrong assumption propagates:

``` text
Wrong source assumption
→ wrong Dockerfile
→ wrong pipeline
→ wrong Kubernetes YAML
→ production failure
```

Example: application listens on `5000`, but Kubernetes sends traffic to
`8080`. The pod can be Running while the application is unreachable.

## 3. Step 1 — Inventory the Repository

``` bash
cd ~/dhl
pwd
find . -maxdepth 2 -type f | sort
```

Current high-level structure:

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
├── .github/workflows/
├── docker-compose.yml
├── Jenkinsfile
└── sonar-project.properties
```

**What are we proving?** This is a multi-service repository. Each
independently deployable service needs its own runtime, image,
configuration and deployment analysis.

## 4. Step 2 — Identify the Technology

Common manifest clues:

| File                                 | Ecosystem          |
|--------------------------------------|--------------------|
| `package.json`                       | Node.js/JavaScript |
| `requirements.txt`, `pyproject.toml` | Python             |
| `pom.xml`                            | Java/Maven         |
| `build.gradle`                       | Java/Gradle        |
| `go.mod`                             | Go                 |
| `*.csproj`                           | .NET               |

Start with backend:

``` bash
cat backend/package.json
```

It tells us:

``` text
main  → server.js
start → node server.js
dev   → nodemon server.js
test  → jest ...
```

**Conclusion:**

``` text
Backend ecosystem: Node.js
Entry point: server.js
Production start: node server.js
Testing: Jest
Test command: npm test
```

## 5. Step 3 — Translate Dependencies into Operational Meaning

Backend dependencies include Express, Mongoose, CORS, dotenv,
jsonwebtoken and bcryptjs. Dev/test dependencies include Jest, Nodemon
and Supertest.

| Package      | What DevOps Learns               |
|--------------|----------------------------------|
| Express      | HTTP API process                 |
| Mongoose     | MongoDB dependency               |
| dotenv       | Environment-driven configuration |
| jsonwebtoken | Authentication/token behavior    |
| cors         | Browser/API origin behavior      |
| Jest         | Automated tests                  |
| Supertest    | API tests                        |
| Nodemon      | Development tooling              |

Do not stop at “Mongoose is installed.” Continue: **Where is
`mongoose.connect()` called? Which URI does it use? What happens if
MongoDB is unavailable?**

## 6. Step 4 — Find the Entry Point

``` bash
less backend/server.js
```

Instead of initially reading every line, search operational patterns:

``` bash
grep -n "process.env" backend/server.js
grep -n "listen(" backend/server.js
grep -n "mongoose.connect" backend/server.js
grep -n "fetch(" backend/server.js
grep -n "app.get\|app.post\|app.put\|app.delete" backend/server.js
```

This is the core DevOps code-analysis technique: turn source code into
deployment questions.

## 7. Step 5 — Find the Runtime Port

Backend source uses:

``` text
PORT environment variable
Default: 5000
```

and starts Express using `app.listen(PORT, ...)`.

Therefore:

``` text
server.js → PORT=5000
             ↓
        Docker runtime
             ↓
   Kubernetes containerPort
             ↓
      Service targetPort
             ↓
          Probe port
```

Validate at runtime:

``` bash
ss -lntp
```

If nothing is listening on the expected port, investigate the
application/runtime before blaming networking.

## 8. Step 6 — Find and Classify Configuration

``` bash
grep -Rni "process.env" backend banking-service language-service price-service air-cargo-service sea-cargo-service
```

Important backend variables:

``` text
PORT
MONGO_URI
JWT_SECRET
PRICE_SERVICE_URL
BANK_SERVICE_URL
AIR_CARGO_SERVICE_URL
SEA_CARGO_SERVICE_URL
NODE_ENV
```

Classify them:

| Variable     | Classification                                  | Future Handling        |
|--------------|-------------------------------------------------|------------------------|
| `PORT`       | Configuration                                   | Deployment/ConfigMap   |
| `MONGO_URI`  | Connection configuration, potentially sensitive | Secret/config design   |
| `JWT_SECRET` | Secret                                          | Secret-management path |
| Service URLs | Configuration                                   | ConfigMap/env          |
| `NODE_ENV`   | Configuration                                   | Deployment/env         |

**Security finding:** the current project contains a default/literal
JWT-secret pattern. We identify it now and remediate it in the security
phase rather than copying it into production.

## 9. Step 7 — Discover the Database

``` bash
grep -Rni "mongoose\|MONGO_URI\|mongodb://" backend banking-service
```

Backend:

``` text
MONGO_URI
  ↓
mongoose.connect()
  ↓
MongoDB
```

Current topology:

``` text
Backend → mongodb:27017/dhl-clone
Banking → mongodb:27017/dhl-bank
```

Always cross-check:

``` text
Source expectation = Runtime configuration
```

If source expects `MONGO_URI` but the platform supplies `DATABASE_URL`,
deployment is misconfigured.

## 10. Step 8 — Discover Downstream Services

``` bash
grep -n "SERVICE_URL" backend/server.js
grep -n "fetch(" backend/server.js
```

Backend dependencies:

``` mermaid
flowchart TD
    FE["Frontend"] --> BE["Backend"]
    BE --> MDB[("MongoDB")]
    BE --> PRICE["Price Service"]
    BE --> BANK["Banking Service"]
    BANK --> BDB[("MongoDB / dhl-bank")]
    BE --> AIR["Air Cargo Service"]
    BE --> SEA["Sea Cargo Service"]
```

Operational lesson:

``` text
Backend Running + Price Service Down
→ authentication may work
→ rate calculation can fail
```

**Process health is not business-transaction health.**

## 11. Step 9 — Discover API Routes

``` bash
grep -n "app.get\|app.post\|app.put\|app.delete" backend/server.js
```

Important routes include:

``` text
POST /api/auth/register
POST /api/auth/login
POST /api/shipments/calculate-rate
POST /api/shipments/book
GET  /api/shipments/user
GET  /api/shipments/track/:consignmentNumber
GET  /api/admin/shipments
PUT  /api/admin/shipments/:id/status
```

Routes tell us what real functionality can be validated. Do not blindly
curl `/health` unless it exists.

## 12. Step 10 — Trace One Business Transaction

### Rate Calculation

``` text
Customer → Frontend → Backend
→ /api/shipments/calculate-rate
→ PRICE_SERVICE_URL
→ Price Service /api/pricing/calculate
→ Backend → Frontend
```

Now a failed rate request has a concrete troubleshooting path.

### Shipment Booking

``` mermaid
flowchart TD
    U["Authenticated User"] --> BE["Backend /api/shipments/book"]
    BE --> BANK["Banking Service"]
    BANK --> PAY{"Payment OK?"}
    PAY -->|No| ERR["Return Error"]
    PAY -->|Yes| DB[("Save Shipment")]
    DB --> TYPE{"Cargo Type"}
    TYPE -->|Air| AIR["Air Cargo"]
    TYPE -->|Sea| SEA["Sea Cargo"]
    AIR --> OK["Response"]
    SEA --> OK
```

The transaction depends on authentication, banking, persistence and the
selected cargo path.

## 13. Step 11 — Understand Failure Semantics

This is deeper analysis.

The current backend treats dependencies differently:

``` text
Banking failure
→ booking is rejected

Air/Sea cargo call failure after persistence
→ error is logged
→ booking can still return success
```

That means dependency **criticality differs**.

This affects:

- readiness design;
- alert severity;
- business monitoring;
- retries/queues;
- incident diagnosis;
- consistency handling.

A 4-year DevOps engineer should notice this rather than treating every
dependency as identical.

## 14. Step 12 — Search for Health Semantics

``` bash
grep -Rni "health\|ready\|readiness\|live\|liveness" backend/
```

If no appropriate endpoint exists:

``` text
Finding:
No verified dedicated health endpoint.

Impact:
Production Kubernetes probe design needs an application health contract.

Action:
Coordinate/design meaningful liveness and readiness behavior.
```

Never invent `/health` just because Kubernetes supports probes.

## 15. Step 13 — Analyze Error Handling

``` bash
grep -Rni "catch\|console.error\|status(500)" backend
```

Ask:

``` text
Does failure crash the process?
Does it return 500?
Is it swallowed?
Is there a timeout?
Is there retry behavior?
Can partial state remain?
Are logs useful?
```

These answers drive monitoring, alerts, probes and incident runbooks.

## 16. Step 14 — Analyze Frontend

``` bash
cat frontend/package.json
```

Findings:

``` text
Framework: React
Build tool: Vite
Build: npm run build
Development: vite
HTTP client: Axios
Routing: React Router
```

Then inspect source:

``` bash
find frontend/src -type f | sort
grep -Rni "axios\|fetch\|http://\|https://" frontend/src
```

Ask:

``` text
Which backend URL is called?
Is it hardcoded?
Is it runtime or build-time configuration?
Will browser CORS matter?
```

## 17. Step 15 — Understand Frontend Build vs Runtime

``` text
React source
    ↓
Node/npm + Vite
    ↓
npm run build
    ↓
Static HTML/CSS/JS
    ↓
Nginx
    ↓
Browser
```

Because the repository contains `nginx.conf`, inspect:

``` bash
cat frontend/nginx.conf
```

Look for:

- Nginx listening port;
- document root;
- SPA fallback;
- API proxying;
- cache behavior.

These findings drive the production frontend image.

## 18. Step 16 — Compare Source with Docker Compose

Only after source analysis:

``` bash
cat docker-compose.yml
docker compose config
```

Current Compose ports:

| Component |  Port |
|-----------|------:|
| MongoDB   | 27017 |
| Backend   |  5000 |
| Language  |  5001 |
| Price     |  5002 |
| Air Cargo |  5003 |
| Sea Cargo |  5004 |
| Banking   |  5005 |
| Frontend  |  3000 |

Create a configuration matrix:

| Source Expects          | Compose Supplies    | Result           |
|-------------------------|---------------------|------------------|
| `PORT`                  | `5000`              | Match            |
| `MONGO_URI`             | MongoDB service URI | Match            |
| `PRICE_SERVICE_URL`     | price-service       | Match            |
| `BANK_SERVICE_URL`      | banking-service     | Match            |
| `AIR_CARGO_SERVICE_URL` | air-cargo-service   | Match            |
| `SEA_CARGO_SERVICE_URL` | sea-cargo-service   | Match            |
| `JWT_SECRET`            | Literal value       | Security finding |

## 19. Step 17 — Understand Service Discovery

Inside Docker Compose, service names provide stable discovery:

``` text
http://price-service:5002
```

Do not depend on container IPs.

Later Kubernetes provides the same general abstraction through
Kubernetes Services and cluster DNS.

## 20. Step 18 — Create the Deployment Contract

### Backend

| Field       | Verified Finding                           |
|-------------|--------------------------------------------|
| Runtime     | Node.js                                    |
| Framework   | Express                                    |
| Entry point | `server.js`                                |
| Start       | `node server.js`                           |
| Test        | `npm test`                                 |
| Port        | 5000 default                               |
| Database    | MongoDB                                    |
| DB config   | `MONGO_URI`                                |
| Secret      | `JWT_SECRET`                               |
| Downstreams | Price, Banking, Air, Sea                   |
| Config      | Service URLs, PORT, NODE_ENV               |
| Health      | Must verify/design                         |
| Main risks  | DB, DNS, service URLs, downstream failures |

### Frontend

| Field             | Finding                    |
|-------------------|----------------------------|
| Framework         | React                      |
| Build tool        | Vite                       |
| Build             | `npm run build`            |
| Dev command       | `vite`                     |
| HTTP client       | Axios                      |
| Production model  | Static assets + web server |
| Web server clue   | `nginx.conf`               |
| API configuration | Inspect source             |
| Health            | Runtime strategy required  |

## 21. Step 19 — Repeat for Every Service

``` bash
cat <service>/package.json
grep -Rni "process.env" <service>
grep -Rni "listen(" <service>
grep -Rni "app.get\|app.post\|app.put\|app.delete" <service>
grep -Rni "fetch(\|axios" <service>
```

Do not guess unknowns. Mark them `VERIFY` and inspect.

## 22. Step 20 — Local Runtime Validation

For backend:

``` bash
cd backend
npm install
npm test
npm start
```

If a valid lockfile exists, CI generally prefers:

``` bash
npm ci
```

because installation is driven by the lockfile.

Then:

``` bash
ss -lntp
```

Test only a **verified existing endpoint**.

## 23. Failure Lab — Wrong Price Service URL

Set:

``` text
PRICE_SERVICE_URL=http://wrong-service:5002
```

Expected:

``` text
Backend process: Running
Some APIs: Working
Rate calculation: Failing
```

Troubleshoot:

``` text
Exact error
→ backend logs
→ effective PRICE_SERVICE_URL
→ DNS
→ TCP port
→ direct pricing endpoint
→ price-service logs
→ fix
→ end-to-end retest
```

## 24. Failure Lab — MongoDB Down

Investigate:

``` text
Logs → MONGO_URI → DNS → TCP 27017
→ MongoDB health → auth/config
→ affected business routes
```

Remember:

``` text
Running process ≠ Ready application ≠ Healthy business transaction
```

## 25. Failure Lab — Port Mismatch

``` text
Application: 5000
Platform: 8080
```

Compare:

``` text
Source PORT
→ Docker runtime
→ Kubernetes containerPort
→ Service targetPort
→ Ingress backend
```

Use:

``` bash
ss -lntp
```

Fix the layer containing the mismatch.

## 26. Complete Code-Analysis Decision Tree

``` text
New Repository
↓
Inventory
↓
Manifest/package files
↓
Entry point
↓
Build/Test/Start
↓
Port
↓
Configuration + Secrets
↓
Database
↓
Downstream HTTP calls
↓
API routes
↓
Health semantics
↓
Error/failure handling
↓
Compose/Docker comparison
↓
Deployment Contract
↓
Runtime validation
↓
Ready for Docker/CI/Kubernetes
```

## 27. How Findings Drive Future DevOps Work

| Code Finding            | Future DevOps Work          |
|-------------------------|-----------------------------|
| Node.js                 | Docker build/runtime image  |
| Vite                    | Frontend build stage        |
| `npm test`              | Jenkins test stage          |
| Port 5000               | Container/Service/probes    |
| MongoDB                 | Stateful dependency design  |
| `MONGO_URI`             | Runtime configuration       |
| `JWT_SECRET`            | Secret management           |
| Service URLs            | ConfigMap/service discovery |
| API routes              | Functional tests            |
| Missing health endpoint | Probe design                |
| Failure behavior        | Monitoring/alerts/runbooks  |

## 28. Interview-Ready Explanation

> When I receive a new application repository, I don't immediately write
> the pipeline. I create a deployment contract for each service. I
> inspect the repository structure and package manifest to identify the
> runtime, dependencies, build, test and startup commands. Then I
> inspect the entry point for listening ports and environment variables.
> I trace database connections and downstream HTTP calls to build a
> dependency map, inspect API and health behavior, and understand
> failure handling. Finally, I compare those requirements against
> existing Docker Compose or deployment configuration and validate the
> service at runtime. Only then do I design Docker, CI/CD and
> Kubernetes.

## 29. Hands-On Assignment

Run:

``` bash
cd ~/dhl
find . -maxdepth 2 -type f | sort
cat backend/package.json
grep -n "process.env" backend/server.js
grep -n "listen(" backend/server.js
grep -n "mongoose.connect" backend/server.js
grep -n "fetch(" backend/server.js
grep -n "app.get\|app.post\|app.put\|app.delete" backend/server.js
grep -Rni "health\|ready\|readiness\|live\|liveness" backend/
cat frontend/package.json
find frontend/src -type f | sort
grep -Rni "axios\|fetch\|http://\|https://" frontend/src
cat frontend/nginx.conf
docker compose config
```

For **every command**, answer:

``` text
1. What am I searching?
2. Why am I searching it?
3. What did the output prove?
4. Which future DevOps configuration depends on it?
5. What failure happens if this assumption is wrong?
```

## 30. Completion Checklist

- [ ] Inventory an unfamiliar repository
- [ ] Identify language/framework from manifests
- [ ] Find the entry point
- [ ] Derive build/test/start commands
- [ ] Find runtime ports
- [ ] Find and classify environment variables
- [ ] Identify secrets
- [ ] Trace database connections
- [ ] Trace downstream HTTP calls
- [ ] List API routes
- [ ] Search for health semantics
- [ ] Analyze error handling
- [ ] Trace a business transaction
- [ ] Compare source with Compose
- [ ] Create a deployment contract
- [ ] Explain how findings drive Docker/Jenkins/Kubernetes
- [ ] Troubleshoot dependency and port mismatches

# Golden Rule

> **Source code defines the application requirements. DevOps automation
> implements verified requirements—not assumptions.**
