# DAY 2 — Application Understanding Interview Training

> **Target:** DevOps Engineer interview preparation at approximately 4
> years of experience.

## Basic

### Q1. You receive a new repository. What do you check first?

> I identify the deployable components and, for each one, determine
> language, framework, dependency manager, build/test/start commands,
> ports, environment variables, database and downstream dependencies,
> health behavior and runtime files. Only then do I design Docker, CI/CD
> and Kubernetes.

**Cross-question — Why not start with Docker?**

> Docker configuration depends on application runtime facts. Guessing
> them creates fragile automation.

### Q2. How do you identify the technology stack?

> I inspect language-specific manifests and source entry points. In this
> project, package metadata shows React/Vite on the frontend and
> Express/Mongoose on the backend.

### Q3. What does `package.json` tell a DevOps engineer?

> It exposes scripts, dependencies, development dependencies and project
> metadata. I use it to derive dependency-installation, build, test and
> startup stages.

### Q4. dependencies vs devDependencies?

> Runtime dependencies normally support application functionality. Dev
> dependencies support development/build/test tooling. Frontend build
> tools can be required during the build stage even if absent from the
> final runtime image.

### Q5. What are environment variables?

> External runtime configuration. They allow the same artifact to use
> different ports, database endpoints and service URLs without
> source-code changes.

## Normal / Practical

### Q6. Important backend configuration?

> Port, MongoDB URI, JWT secret and downstream URLs for pricing,
> banking, air cargo and sea cargo. I separate non-sensitive
> configuration from secrets.

### Q7. How do you determine an application's port?

> Source code and runtime configuration are primary evidence. Then I
> validate the actual socket using `ss -lntp`. The backend defaults to
> 5000 unless `PORT` overrides it.

### Q8. Why isn't reading the Dockerfile enough?

> A Dockerfile can be stale or incorrect. I compare it against package
> metadata, source behavior and runtime validation.

### Q9. How do you identify dependencies between services?

> I search environment variables, HTTP calls, database connection
> strings, Compose configuration and deployment manifests, then create a
> dependency map.

### Q10. Which database is used?

> MongoDB. The backend uses Mongoose; the current Compose topology uses
> separate database names for core application and banking data.

## Mid-Level

### Q11. Build-time vs runtime dependency?

> Build-time dependencies produce the artifact. Runtime dependencies are
> required while serving traffic. Vite can build React assets, while a
> production web server serves the generated static files.

### Q12. Why promote the same image across environments?

> Rebuilding per environment can produce different artifacts. Building
> once and promoting an immutable artifact improves traceability and
> consistency while runtime configuration changes per environment.

### Q13. Downstream service unavailable—does the calling service always go down?

> No. The process may stay alive while only workflows requiring that
> dependency fail. That is why I distinguish process health, readiness
> and business-transaction health.

### Q14. Why isn't Compose `depends_on` a readiness check?

> Startup ordering does not prove a dependency is ready to accept
> requests. Readiness must reflect actual application availability.

### Q15. Why make service URLs configurable?

> Local, Compose and Kubernetes environments use different
> service-discovery names. External configuration allows the artifact to
> move without code changes.

## Advanced

### Q16. What would you improve before Kubernetes?

> Reproducible builds, externalized secrets, meaningful health
> endpoints, graceful shutdown, standardized configuration, structured
> logging/metrics, dependency timeouts/retries, automated tests and
> database persistence/backup requirements. Kubernetes cannot compensate
> for poor application operability.

### Q17. Risk of `latest` tags?

> They are mutable and reduce deployment determinism. Explicit versions
> or immutable digests improve traceability and rollback confidence.

### Q18. Can Kubernetes fix bad application architecture?

> No. It can schedule, restart, scale and network workloads, but it
> cannot automatically correct unsafe secrets, bad dependency handling,
> missing health semantics or data-consistency problems.

### Q19. How do you design readiness?

> Readiness should answer whether a pod can safely receive traffic. I
> base it on application behavior and critical dependencies, avoiding
> unnecessary coupling to optional dependencies.

## Architecture

### Q20. Explain the application dependency flow.

> The frontend is user-facing. The backend handles core workflows and
> communicates with pricing, banking and cargo services. MongoDB
> provides persistence. Shipment booking requires the backend, banking,
> persistence and the relevant cargo path.

### Q21. Why not expose all services publicly?

> Internal-only services do not need Internet exposure. Keeping them
> private reduces attack surface and preserves clearer trust boundaries.

### Q22. Compose DNS vs Kubernetes DNS?

> Compose resolves service names on its network. Kubernetes provides
> stable service discovery through Kubernetes Services and cluster DNS.
> Both avoid depending directly on ephemeral workload IPs.

## Troubleshooting

### Q23. Process is running but expected port isn't listening.

> Check startup logs, effective port, bind address and socket state.
> Compare what the application actually listens on with platform
> configuration.

``` bash
ss -lntp
ps -ef | grep node
```

### Q24. Backend cannot connect to MongoDB.

``` text
Exact error → MONGO_URI → DNS → TCP/27017 → MongoDB health → auth/config → retest
```

> I capture the exact error first, verify the effective connection
> string, resolve the hostname, test reachability, verify MongoDB and
> then inspect authentication/configuration.

### Q25. Backend starts but pricing fails.

> Narrow the failure to the pricing path: backend error/logs →
> `PRICE_SERVICE_URL` → DNS → TCP → pricing endpoint → pricing-service
> logs.

### Q26. Frontend loads but API calls fail.

> Static frontend success does not prove API connectivity. Inspect
> browser network errors and the actual API URL, then check routing/DNS,
> CORS where applicable, backend reachability and backend logs.

## Scenarios

### Scenario 1 — Pods Running, rate API returns 500

> Determine whether only pricing is affected. Inspect backend logs and
> effective pricing URL, verify DNS/connectivity from the backend
> runtime, test the pricing endpoint directly, correct the configuration
> source and retest the complete business flow.

### Scenario 2 — MongoDB down, backend pod still Running

> Running is container/process state, not proof of business readiness.
> Inspect logs and database connectivity, identify impacted workflows
> and evaluate whether readiness should remove the pod from traffic when
> critical functions cannot operate.

### Scenario 3 — Works locally, fails in Kubernetes

> Compare hostnames, ports, environment variables, secrets, bind
> addresses and filesystem assumptions. `localhost` is a common issue
> because it refers to the current pod/container context, not another
> Kubernetes Service.

### Scenario 4 — Frontend build passes but page is blank

> Verify generated assets are copied to the web-server document root,
> browser console/network errors, Nginx configuration, SPA routing and
> API configuration. Build success does not prove runtime serving.

## Project-Based

### Q27. How did you analyze this unfamiliar application?

> In my hands-on enterprise-style project, I created a service
> inventory, inspected package manifests and source entry points,
> documented scripts, ports and environment variables, then mapped
> MongoDB and service-to-service dependencies using source code and
> Compose configuration. That became the deployment contract for later
> Docker, CI/CD and Kubernetes work.

### Q28. What security issue did you identify?

> I identified a default/hardcoded JWT-secret pattern in the current
> project configuration. I treat that as a remediation item and will
> externalize production secrets rather than embedding them in source,
> Compose or images.

## Cross-Questions

Be ready for:

- `npm install` vs `npm ci`
- Why use a lockfile?
- What is `NODE_ENV`?
- How do you discover health endpoints?
- What if no health endpoint exists?
- What is graceful shutdown?
- Why can `localhost` fail between services?
- DNS failure vs connection refused?
- ConfigMap vs Secret?
- How should production secrets be delivered?
- How do you test TCP connectivity?
- How do you validate an end-to-end business transaction?

## STAR Answer

**Situation:** A multi-service logistics application needed an
end-to-end DevOps platform.

**Task:** Before creating the pipeline and Kubernetes configuration, I
needed a verified deployment contract.

**Action:** I analyzed repository structure, package manifests, source
entry points and Compose topology; documented runtime, scripts, ports,
environment variables, database and downstream dependencies; and
identified secret-management and failure-path concerns.

**Result:** The next infrastructure and automation phases can be derived
from verified application requirements rather than assumptions, and
failures can be isolated by dependency path.

## Rapid-Fire Revision

1.  First checks in a new repo?
2.  Why inspect `package.json`?
3.  How find startup command?
4.  How find build command?
5.  How find listening port?
6.  What are environment variables?
7.  Why not hardcode secrets?
8.  Which database is used?
9.  What is Mongoose?
10. What is service-to-service communication?
11. Why configurable service URLs?
12. Build time vs runtime?
13. Process health vs readiness?
14. Why isn't `depends_on` readiness?
15. Why can a Running app be unusable?
16. What happens if MongoDB fails?
17. What happens if pricing fails?
18. How troubleshoot DNS failure?
19. Why can `localhost` be wrong?
20. How does application analysis influence Kubernetes?

## Completion Checklist

- [ ] Analyze an unfamiliar repository
- [ ] Identify frontend/backend stack
- [ ] Derive build/test/start commands
- [ ] Find ports and environment variables
- [ ] Map dependencies
- [ ] Explain MongoDB usage
- [ ] Explain configuration vs secrets
- [ ] Explain process health vs readiness
- [ ] Troubleshoot DB/downstream failures
- [ ] Give the project and STAR answers naturally

## Interview Incident Framework

``` text
Symptom → Blast Radius → Recent Change → Evidence → Failing Layer
→ Diagnostic Test → Root Cause → Minimum Fix → Validation → Prevention
```
