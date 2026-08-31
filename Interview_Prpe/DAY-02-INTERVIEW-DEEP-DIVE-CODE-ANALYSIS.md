# DAY 2 — Deep-Dive Code Analysis Interview Training

> **Target:** DevOps Engineer — approximately 4 years experience.

## Basic

### Q1. What do you check first in a new repository?

> I inventory deployable services, then determine each service's
> language, manifest, entry point, dependencies, build/test/start
> commands, port, configuration, secrets, database, downstream
> dependencies and health behavior. This produces a deployment contract.

### Q2. Why inspect `package.json`?

> It exposes Node.js scripts and dependencies. I use it to derive
> dependency installation, testing, build and startup requirements.

### Q3. How do you find the entry point?

> I correlate package metadata with source structure. Here
> `package.json` identifies `server.js` and `npm start` runs
> `node server.js`.

### Q4. How do you find the port?

> I inspect the entry point for `process.env` and `listen()`, then
> validate the actual runtime socket using `ss -lntp`.

### Q5. What is a deployment contract?

> The verified requirements for running a service: runtime, artifact,
> startup command, port, configuration, secrets, dependencies and health
> behavior.

## Practical / Mid-Level

### Q6. How do you find environment variables?

``` bash
grep -Rni "process.env" <service>
```

> I then classify ordinary configuration separately from secrets.

### Q7. How do you find downstream dependencies?

> I search HTTP calls such as `fetch`/Axios, service URL variables, DB
> connection code and deployment configuration, then build a dependency
> map.

### Q8. Why compare source with Compose?

> Source defines what the app expects; Compose shows what runtime
> supplies. Comparing them detects missing variables, bad hostnames,
> port mismatches and secret issues.

### Q9. Why can `localhost` break microservices?

> Inside a container or pod, localhost refers to that local network
> context, not automatically another service. Inter-service traffic
> needs the appropriate service-discovery hostname.

### Q10. Why doesn't Running mean healthy?

> Running can describe process/container state while the database,
> downstream service or business transaction is failing.

## Advanced

### Q11. How do you determine dependency criticality?

> Trace business workflows and failure handling. In this project,
> banking failure blocks booking, while a later cargo-service call can
> be logged without necessarily preventing the success response. That
> means dependency criticality differs.

### Q12. What belongs in readiness?

> Whatever is required for that workload to safely receive traffic. I do
> not blindly include every optional dependency because that can
> unnecessarily remove healthy capacity.

### Q13. What if there is no health endpoint?

> Document the gap. Coordinate/design meaningful health semantics rather
> than inventing `/health`.

### Q14. Why inspect error handling?

> It tells me whether failures crash, return errors, get swallowed,
> retry, hang or leave partial state. That drives observability and
> incident response.

### Q15. Can Kubernetes fix a bad application?

> No. Kubernetes cannot infer correct ports, fix hardcoded secrets,
> create meaningful health semantics or repair transaction consistency.

## Architecture

### Q16. Explain backend dependencies.

> The Node/Express backend uses MongoDB through Mongoose and calls
> pricing, banking, air-cargo and sea-cargo services using configurable
> URLs.

### Q17. Explain shipment booking.

``` text
User → Frontend → Backend → Authentication → Banking
→ MongoDB → Air/Sea Cargo → Response
```

> That dependency chain becomes my troubleshooting map.

### Q18. How does code analysis influence Kubernetes?

> Source ports drive container/Service ports; environment variables
> drive ConfigMaps/Secrets; downstream URLs drive service discovery;
> health behavior drives probes; failure behavior drives monitoring.

## Troubleshooting

### Q19. Backend works but pricing fails.

> Scope the incident to pricing, inspect backend logs, verify
> `PRICE_SERVICE_URL`, test DNS/TCP, call the price endpoint directly,
> inspect price-service logs, fix the failing layer and retest
> end-to-end.

### Q20. Backend cannot reach MongoDB.

``` text
Error → MONGO_URI → DNS → TCP 27017 → MongoDB
→ auth/config → application behavior → retest
```

### Q21. Pod Running but Service cannot reach application.

> Compare actual listening port with container/Service target ports,
> then verify readiness and endpoints.

### Q22. Frontend loads but APIs fail.

> Inspect browser network errors/API URL, routing/DNS, CORS where
> relevant, backend reachability and backend logs. Static assets loading
> does not prove backend health.

## Scenarios

### Scenario 1 — Rate calculation fails after Kubernetes migration

> Verify the backend's effective price-service URL, Kubernetes DNS,
> Service endpoints and target port. Do not assume a Compose hostname
> automatically maps to Kubernetes.

### Scenario 2 — Booking fails but tracking works

> The whole backend is probably not down. Compare dependency paths:
> booking requires banking and additional dependencies; tracking
> primarily needs persisted shipment data. Investigate booking-specific
> dependencies first.

### Scenario 3 — Cargo service down but booking says success

> Inspect transaction ordering and exception handling. If persistence
> occurs first and cargo errors are only logged, the API can return
> success while downstream consistency is degraded. Flag this for
> monitoring and application reliability design.

### Scenario 4 — `npm start` works locally but container fails

> Compare environment variables, working directory, copied files,
> runtime user, bind address, port and service hostnames. Use container
> logs and socket state to isolate the environment difference.

## Project Answer

### Q23. How did you analyze your application?

> In my hands-on enterprise-style DHL-inspired project, I inventoried
> the services, used `package.json` to identify runtime/scripts,
> inspected `server.js` for ports and environment variables, traced
> MongoDB and downstream `fetch` calls, mapped pricing and shipment
> workflows, reviewed failure behavior, and compared those findings
> against Docker Compose. That produced the deployment contract for
> Docker, Jenkins and Kubernetes.

## Cross-Questions

- `npm install` vs `npm ci`?
- Why lockfiles?
- What if `PORT` and `targetPort` differ?
- DNS failure vs connection refused?
- What does localhost mean in a container?
- How test DNS/TCP connectivity?
- Should every dependency be in readiness?
- What if an HTTP call has no timeout?
- What is graceful shutdown?
- ConfigMap vs Secret?
- How do source findings influence monitoring?

## STAR

**Situation:** A multi-service logistics app needed an end-to-end DevOps
platform.

**Task:** Establish verified runtime/dependency requirements before
containerization.

**Action:** I inspected repository structure, manifests, entry points,
scripts, ports, environment variables, database and HTTP dependencies,
business workflows and failure handling, then compared source with
Compose.

**Result:** I produced a deployment contract and dependency map,
identified configuration/security gaps, and reduced guesswork for
Docker, CI/CD and Kubernetes.

## Rapid Fire

1.  Deployment contract?
2.  First Node.js file?
3.  Find entry point?
4.  Find env vars?
5.  Find port?
6.  Find DB dependency?
7.  Find HTTP dependencies?
8.  Why trace transactions?
9.  Running vs ready?
10. Why compare source and Compose?
11. localhost inside container?
12. What if no health endpoint?
13. Why inspect error handling?
14. How source drives Docker?
15. How source drives Jenkins?
16. How source drives Kubernetes?
17. How source drives monitoring?
18. Debug wrong service URL?
19. Debug DB failure?
20. Debug port mismatch?

## Completion Checklist

- [ ] Explain deployment contracts
- [ ] Analyze `package.json`
- [ ] Analyze an entry point efficiently
- [ ] Find ports/config/secrets
- [ ] Trace DB and HTTP dependencies
- [ ] Trace business transactions
- [ ] Explain dependency criticality
- [ ] Identify health gaps
- [ ] Compare source and Compose
- [ ] Troubleshoot source/config/platform mismatches
- [ ] Give the project answer naturally

# Incident Framework

``` text
Symptom → Blast Radius → Recent Change → Evidence → Failing Layer
→ Diagnostic Test → Root Cause → Minimum Fix → Validation → Prevention
```
