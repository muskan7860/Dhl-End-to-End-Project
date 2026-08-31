# DAY 2 — Application Understanding & Source-Code Analysis

> **Goal:** Learn how a DevOps engineer analyzes an unfamiliar
> application before writing Dockerfiles, pipelines, Kubernetes
> manifests, or cloud infrastructure.
>
> **Project:** DHL-inspired Enterprise Logistics & Shipment Tracking
> Platform.

## 1. Golden Rule

> **First understand the application. Then automate it.**

``` text
Repository → Services → Language/Framework → Dependencies → Build/Test/Start
→ Ports → Environment Variables → Service Calls → Database → Health
→ Deployment Requirements
```

## 2. Actual Repository Structure

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

The frontend contains `src`, `Dockerfile`, `index.html`, `nginx.conf`,
`package.json`, and `vite.config.js`. The backend contains `models`,
`tests`, `Dockerfile`, `package.json`, and `server.js`.

## 3. DevOps Repository-Analysis Checklist

| Question               | Why DevOps Needs It          |
|------------------------|------------------------------|
| Language/framework?    | Runtime and tooling          |
| Package manager?       | Dependency installation      |
| Dependencies?          | Build, runtime, scanning     |
| Build command?         | CI                           |
| Test command?          | CI quality gate              |
| Startup command?       | Container runtime            |
| Listening port?        | Docker/Kubernetes networking |
| Environment variables? | Runtime configuration        |
| Service dependencies?  | Networking/service discovery |
| Database?              | Stateful dependency          |
| Health behavior?       | Probes/load-balancer health  |
| Runtime files?         | Container image design       |

## 4. Frontend Analysis

The frontend is a **React application built with Vite**. Its package
metadata includes React, React DOM, React Router, Axios and Vite.

Important scripts:

``` text
Development → vite
Build       → vite build
Preview     → vite preview
```

The repository also contains `nginx.conf`. That is a clue that the
production frontend is intended to be built into static assets and
served by a web server rather than treating the Vite development server
as the production runtime.

``` text
React Source → npm dependencies → npm run build → static output → web server
```

### DevOps information we need

- Node version
- dependency lockfile
- build command
- build output
- Nginx/runtime configuration
- runtime port
- API endpoint configuration

## 5. Backend Analysis

The backend is **Node.js + Express**.

Important dependencies include:

- Express
- Mongoose
- CORS
- dotenv
- JSON Web Token
- bcryptjs

Development/test tooling includes Jest, Supertest and Nodemon.

Scripts:

``` text
Start → node server.js
Dev   → nodemon server.js
Test  → jest
```

This becomes future CI/container input:

``` text
Jenkins → install dependencies → test
Container → start application
```

## 6. Runtime Configuration

Important backend configuration includes:

``` text
PORT
MONGO_URI
JWT_SECRET
PRICE_SERVICE_URL
AIR_CARGO_SERVICE_URL
SEA_CARGO_SERVICE_URL
BANK_SERVICE_URL
```

The backend defaults to port `5000`.

Environment variables let one artifact run with different
environment-specific configuration:

``` text
Same Artifact
├── Development config
├── Test config
├── UAT config
└── Production config
```

## 7. Security Finding

The current project contains a fallback/default JWT secret pattern and
the Compose configuration contains a literal JWT secret.

That is a finding for us to remediate. Production secrets should not be
committed into source code, Compose files, Dockerfiles, images, or
public repositories.

``` text
Git/source/image → ✗ secret
Runtime secret-management path → ✓
```

We will implement the final secret-management design in its dedicated
phase.

## 8. Database

The current application topology uses **MongoDB**.

``` text
MongoDB :27017
├── dhl-clone
└── dhl-bank
```

The backend uses Mongoose to connect to MongoDB. The banking service
also depends on MongoDB using a separate database name in the current
Compose configuration.

## 9. Service Ports

| Component         |  Port |
|-------------------|------:|
| Frontend          |  3000 |
| Backend           |  5000 |
| Language Service  |  5001 |
| Price Service     |  5002 |
| Air Cargo Service |  5003 |
| Sea Cargo Service |  5004 |
| Banking Service   |  5005 |
| MongoDB           | 27017 |

## 10. Service-to-Service Communication

The backend receives downstream service URLs through environment
variables.

``` text
Backend
├── Price Service :5002
├── Air Cargo Service :5003
├── Sea Cargo Service :5004
└── Banking Service :5005
```

Docker Compose uses service names as network DNS names on the shared
Compose network, for example:

``` text
http://price-service:5002
```

Later Kubernetes will provide stable service discovery through
Kubernetes Services and cluster DNS.

## 11. Shipment Booking Workflow

``` mermaid
flowchart TD
    U["Customer"] --> F["Frontend"]
    F --> B["Backend"]
    B --> BANK["Banking Service"]
    BANK --> MDB[("MongoDB")]
    B --> DB[("Shipment Data")]
    B --> C{"Cargo Type"}
    C -->|Air| AIR["Air Cargo Service"]
    C -->|Sea| SEA["Sea Cargo Service"]
```

The backend receives shipment data, performs payment through the banking
service, persists shipment information and calls the relevant cargo
service.

### Production lesson

``` text
Backend process = Running
Banking service = Down
Result = Some backend APIs can work while shipment booking fails
```

**Process health is not the same as business-transaction health.**

## 12. Docker Compose as an Architecture Document

The current Compose file reveals:

- service names
- build contexts
- ports
- environment variables
- dependencies
- shared network
- MongoDB persistence

The shared network is `dhl-network`, and MongoDB has persistent storage.

`depends_on` can express startup ordering, but startup order does
**not** prove a dependency is ready to serve requests.

## 13. Health Analysis

Before Kubernetes probes, determine:

``` text
Is the process alive?
Is the application ready?
Can it serve requests?
Are critical dependencies usable?
```

Later:

``` text
Liveness  → Should Kubernetes restart this container?
Readiness → Should this pod receive traffic?
Startup   → Has this slow-starting app finished starting?
```

Do **not** invent `/health`. First verify whether the application
exposes a suitable endpoint. If it does not, define an
application-health contract before choosing probes.

## 14. Local Analysis Commands

``` bash
git clone <repository>
cd dhl
find . -maxdepth 2 -type f | sort
```

Inspect manifests:

``` bash
cat backend/package.json
cat frontend/package.json
```

Find environment variables:

``` bash
grep -R "process.env" -n backend banking-service language-service price-service air-cargo-service sea-cargo-service
```

Find listening ports:

``` bash
grep -R "listen(" -n backend banking-service language-service price-service air-cargo-service sea-cargo-service
```

Find service calls:

``` bash
grep -R "SERVICE_URL\|fetch(\|axios" -n backend frontend/src
```

Validate Compose syntax/configuration:

``` bash
docker compose config
```

## 15. Dependency Installation and Tests

For the backend, investigate:

``` bash
cd backend
npm install
npm test
npm start
```

If a valid lockfile exists, controlled CI generally prefers `npm ci` for
deterministic installation. Always verify the repository before choosing
the command.

## 16. Runtime Validation

After startup:

``` bash
ss -lntp
```

Then test a **verified existing endpoint**:

``` bash
curl -i http://localhost:<port>/<known-endpoint>
```

Never assume `/health` exists.

## 17. Troubleshooting Decision Tree

``` text
Application fails
↓
Build succeeded?
├── No → dependency/build problem
└── Yes
    ↓
Process started?
├── No → startup/config/runtime
└── Yes
    ↓
Expected port listening?
├── No → port/bind/process
└── Yes
    ↓
Local endpoint responds?
├── No → application/runtime
└── Yes
    ↓
Dependencies reachable?
├── No → DNS/network/config/dependency
└── Yes
    ↓
Business transaction succeeds?
├── No → downstream/data/workflow
└── Yes → baseline healthy
```

## 18. Failure Lab — Wrong Service URL

Deliberately configure:

``` text
PRICE_SERVICE_URL=http://wrong-service:5002
```

Expected:

``` text
Backend can start
but
rate calculation fails
```

Troubleshoot:

``` text
Exact error → effective environment value → DNS → port → downstream process → endpoint
```

## 19. Failure Lab — MongoDB Unavailable

Stop MongoDB and observe:

- backend logs
- connection behavior
- affected APIs
- whether process remains running
- what readiness should mean

## 20. Failure Lab — Port Mismatch

``` text
Application listens: 5000
Platform expects: 8080
```

Symptom:

``` text
Process Running
but traffic fails
```

Validate with:

``` bash
ss -lntp
```

## 21. How Day 2 Drives Later Phases

| Application Fact         | DevOps Decision                |
|--------------------------|--------------------------------|
| Node.js                  | Build/runtime image            |
| `npm test`               | Jenkins test stage             |
| Vite build               | Frontend build stage           |
| Port 5000                | Container/Service/probe config |
| MongoDB                  | Stateful dependency design     |
| Service URLs             | Runtime configuration          |
| JWT secret               | Secret management              |
| Multiple downstream APIs | Service discovery              |
| Failure behavior         | Readiness/monitoring           |

## 22. Production Considerations Identified

Future phases must address:

- reproducible dependency installation
- pinned/immutable image versions
- non-root containers
- minimal runtime images
- externalized secrets
- health endpoints
- graceful shutdown
- resource requests/limits
- structured logs
- metrics
- timeout/retry behavior
- database persistence/backups
- network restrictions
- TLS
- security scanning

## 23. Day 2 Checklist

- [ ] Explain repository structure
- [ ] Identify frontend/backend technologies
- [ ] Find build/test/start commands
- [ ] Find ports
- [ ] Find environment variables
- [ ] Map service dependencies
- [ ] Explain MongoDB dependency
- [ ] Identify unsafe secret handling
- [ ] Explain Compose service-name DNS
- [ ] Explain process health vs readiness vs business health
- [ ] Troubleshoot wrong downstream URL
- [ ] Troubleshoot port mismatch
- [ ] Explain why source analysis comes before CI/CD

## Day 2 Golden Rule

> **Source code defines application requirements. DevOps automation must
> implement verified requirements, not assumptions.**
