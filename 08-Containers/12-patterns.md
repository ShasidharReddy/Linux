# 12. Real-World Patterns

## 12.1 Why Patterns Matter

Real production systems rarely consist of a single isolated container.
Patterns help structure supporting behavior cleanly.

## 12.2 Sidecar Pattern

A sidecar is a helper container running alongside a primary application container, usually sharing a pod or tight deployment context.

Common sidecar uses:

- Log shipping
- Metrics export
- Service mesh proxying
- File sync or config reload

Benefits:

- Separation of concerns
- Reusable helper capabilities
- Independent lifecycle in some orchestrators

Trade-offs:

- More moving parts
- Resource overhead
- Operational complexity

## 12.3 Ambassador Pattern

An ambassador container acts as a local proxy to external services.
It hides complexity such as:

- TLS setup
- Service discovery
- Connection retry logic
- Protocol translation

Example use case:

- App talks to `localhost:5432`
- Ambassador proxies to external managed PostgreSQL

## 12.4 Adapter Pattern

An adapter container transforms one interface into another.
Examples:

- Convert application logs into a standard format
- Translate metrics protocol
- Normalize output for centralized systems

## 12.5 Init Container Pattern

An init container runs before the main application starts.
Common jobs:

- Database schema migrations
- Waiting for dependency readiness
- Fetching config artifacts
- Preparing filesystem permissions

In Kubernetes this is explicit.
In plain Docker/Compose, similar behavior often requires startup scripts or dedicated one-shot jobs.

## 12.6 Health Check Pattern

Health checks should answer the right question.
There are multiple kinds of health:

- Is the process running?
- Is the app ready to serve traffic?
- Is the app making progress?

A weak health check may say “healthy” while the app is effectively broken.

## 12.7 Graceful Shutdown Pattern

A containerized app should:

1. Receive SIGTERM
2. Stop accepting new work
3. Finish or safely abort in-flight work
4. Flush necessary state/logs
5. Exit before timeout

This is critical for rolling updates and autoscaling events.

## 12.8 Reverse Proxy Frontend Pattern

Common deployment:

- NGINX/Traefik/Envoy container in front
- App containers behind it

Benefits:

- TLS termination
- Routing
- Compression
- Rate limiting
- Centralized access logs

## 12.9 Worker Pattern

Separate async/background tasks into worker containers rather than embedding them inside the web container.

Benefits:

- Independent scaling
- Clearer resource allocation
- Better failure isolation

## 12.10 One-Off Job Pattern

Use ephemeral containers for:

- Migrations
- Backfills
- Admin tasks
- Data imports

These should usually:

- Have clear logging
- Be idempotent where possible
- Run with limited privileges

## 12.11 Batch Processing Pattern

For large jobs:

- Use queue-driven workers
- Externalize progress/state
- Handle retries explicitly
- Make jobs restart-safe

## 12.12 Config Injection Pattern

Keep images generic.
Inject environment-specific config at runtime via:

- Environment variables
- Config files via mounts
- Orchestrator config objects
- Secret managers

## 12.13 Sidecar Example in Compose Style

```yaml
services:
  app:
    image: myapp:latest
    networks:
      - appnet

  metrics-exporter:
    image: prom/statsd-exporter:latest
    networks:
      - appnet

networks:
  appnet:
```

## 12.14 Graceful Shutdown Example: Node.js

```javascript
process.on('SIGTERM', async () => {
  server.close(() => {
    process.exit(0);
  });
});
```

## 12.15 Graceful Shutdown Example: Python

```python
import signal
import sys

def handle_sigterm(signum, frame):
    # stop accepting traffic, flush work, close connections
    sys.exit(0)

signal.signal(signal.SIGTERM, handle_sigterm)
```

## 12.16 Health Endpoint Guidance

A `/health` endpoint should usually verify only what is needed for liveness.
A readiness endpoint may check dependencies required for serving traffic.
Do not overload one probe for every purpose.

## 12.17 Logging Pattern

Recommended approach:

- Application logs to stdout/stderr
- Platform collects/ships logs
- Avoid writing app logs only to local files inside container

## 12.18 Metrics Pattern

Expose metrics on a dedicated endpoint when possible.
Examples:

- `/metrics` for Prometheus
- StatsD sidecar/agent
- OpenTelemetry exporter

## 12.19 Twelve-Factor Influence

Many container best practices align with twelve-factor app ideas:

- Config in environment
- Logs as event streams
- Disposable processes
- Backing services as attached resources

## 12.20 Anti-Patterns to Avoid

- SSH daemon inside app container
- Multiple unrelated long-lived services in one container
- Writing all state only inside container root filesystem
- Relying on manual hot-fixes in running containers
- Hardcoding environment-specific IPs

## 12.21 Pattern Selection Guidance

Choose patterns based on needs:

| Need | Pattern |
|---|---|
| Extra observability helper | Sidecar |
| Proxy to external dependency | Ambassador |
| Format/protocol conversion | Adapter |
| Pre-start initialization | Init container |
| Controlled shutdown | Graceful shutdown |

## 12.22 Summary

Real-world container architecture succeeds when responsibilities are separated clearly, lifecycle is explicit, and helpers are used intentionally rather than ad hoc.

---

# 15. Further Reading & Practice Plan

## 15.1 Suggested Learning Sequence

1. Run simple containers
2. Build your own image
3. Learn Dockerfile optimization
4. Practice networking with custom bridge networks
5. Practice named volumes and bind mounts
6. Create a multi-service Compose stack
7. Harden a container with non-root, read-only FS, and limits
8. Explore rootless runtimes and Podman
9. Learn orchestration basics with Swarm or Kubernetes

## 15.2 Hands-On Labs

### Lab 1: Run NGINX

```bash
docker run -d --name web -p 8080:80 nginx:stable
curl http://localhost:8080
```

### Lab 2: Build a Simple App Image

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY app.py .
CMD ["python", "app.py"]
```

Build and run:

```bash
docker build -t hello-python .
docker run --rm hello-python
```

### Lab 3: Multi-Container App with Compose

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: secret
  app:
    image: myapp:latest
    depends_on:
      - db
```

### Lab 4: Harden a Container

Try adding:

- `USER`
- `--read-only`
- `--cap-drop=ALL`
- `--memory`
- `--pids-limit`

### Lab 5: Debug a Failing Container

Practice with:

```bash
docker logs <name>
docker inspect <name>
docker exec -it <name> sh
```

## 15.3 Final Recommendations

- Learn the kernel primitives, not only the Docker commands
- Keep images small and explicit
- Treat runtime config and secrets carefully
- Prefer immutable rebuilds over in-place patching
- Make health, logging, and graceful shutdown part of the design
- Think about supply chain trust early

## 15.4 Closing Summary

Containers are one of the most important operational abstractions in modern software delivery.
Mastering them means understanding both the developer workflow and the Linux primitives beneath it.
With Docker, good Dockerfiles, disciplined networking/storage practices, and layered security, you can build portable and production-ready containerized systems.

---

## B.12 Pattern Q&A

### Q221. What is a sidecar?
A221. A helper container running alongside the main app.

### Q222. What is an ambassador container?
A222. A proxy helper that mediates access to external services.

### Q223. What is an adapter container?
A223. A helper that transforms output or protocols.

### Q224. What is an init container?
A224. A pre-start helper that runs before the app.

### Q225. Why is graceful shutdown important?
A225. To avoid dropped requests and corrupted in-flight work during stop or rollout.

### Q226. What signal is commonly used for graceful stop?
A226. SIGTERM.

### Q227. Why should apps log to stdout/stderr?
A227. So the platform can collect logs consistently.

### Q228. Why separate web and worker containers?
A228. For independent scaling and clearer responsibility boundaries.

### Q229. What is a one-off job container used for?
A229. Migrations, imports, or admin tasks.

### Q230. Why should one-off jobs be idempotent when possible?
A230. So retries do not cause inconsistent side effects.

### Q231. Why avoid SSH daemons inside app containers?
A231. They add complexity, attack surface, and conflict with container-native access patterns.

### Q232. Why expose metrics separately?
A232. To support monitoring systems cleanly.

### Q233. Why should readiness and liveness checks differ?
A233. They answer different operational questions.

### Q234. Why use reverse proxy containers?
A234. For TLS termination, routing, and edge concerns.

### Q235. Why use queues for worker patterns?
A235. To decouple async work and support retry/backpressure.

### Q236. Why keep images generic across environments?
A236. So the same artifact can move through environments with config injected at runtime.

### Q237. Why do sidecars increase complexity?
A237. More containers mean more lifecycle, monitoring, and resource considerations.

### Q238. What pattern helps prepare filesystem permissions before app start?
A238. Init container pattern.

### Q239. What pattern helps standardize log formats?
A239. Adapter pattern.

### Q240. What pattern helps hide remote DB connection details from the app?
A240. Ambassador pattern.

---

## B.13 Advanced Q&A

### Q241. Why are digests better than tags for SRE runbooks?
A241. They identify exact immutable content.

### Q242. Why can distroless images complicate incident response?
A242. They often lack a shell and debug tools.

### Q243. Why are rootless containers not a complete security solution?
A243. They reduce privilege but do not replace broader security controls.

### Q244. What does it mean that a container is “just a process”?
A244. The kernel ultimately sees processes with namespaces, cgroups, and mounts.

### Q245. Why can deleting files in a later Dockerfile layer fail to save space?
A245. Earlier immutable layers still retain the bytes.

### Q246. Why do multi-stage builds improve security?
A246. They omit build tools and reduce attack surface in the final image.

### Q247. Why is service discovery by DNS name preferred over IPs in Compose and Kubernetes?
A247. Because container/pod addresses can change.

### Q248. Why does BuildKit matter for secure builds?
A248. It supports safer secret and SSH mount mechanisms.

### Q249. What is the biggest operational mistake beginners make with containers?
A249. Treating them like mutable VMs instead of replaceable artifacts.

### Q250. What is the core principle tying all advanced container practice together?
A250. Explicitness: explicit images, explicit config, explicit permissions, explicit networking, explicit lifecycle.

---

# Appendix C: 300 Line-by-Line Best Practices Checklist

1. Pin image versions.
2. Prefer digests for critical deployments.
3. Avoid `latest` in production.
4. Keep images small.
5. Prefer multi-stage builds.
6. Use `.dockerignore`.
7. Keep build context small.
8. Copy dependency manifests before source code.
9. Use deterministic package installs.
10. Remove package manager caches.
11. Avoid unnecessary packages.
12. Run as non-root.
13. Use explicit `USER`.
14. Keep writable paths minimal.
15. Consider `--read-only`.
16. Add `tmpfs` for temporary paths.
17. Drop capabilities.
18. Avoid `--privileged`.
19. Keep seccomp enabled.
20. Use AppArmor/SELinux where supported.
21. Set CPU limits.
22. Set memory limits.
23. Set PID limits.
24. Log to stdout/stderr.
25. Avoid SSH in containers.
26. Handle SIGTERM.
27. Reap child processes properly.
28. Prefer exec form for `CMD`.
29. Prefer exec form for `ENTRYPOINT`.
30. Use health checks carefully.
31. Separate liveness from readiness concepts.
32. Use named volumes for persistent data.
33. Use bind mounts mostly for development.
34. Use read-only mounts when possible.
35. Back up volumes explicitly.
36. Test restore procedures.
37. Document data locations.
38. Use user-defined networks.
39. Publish only necessary ports.
40. Bind local-only ports to `127.0.0.1` when appropriate.
41. Use service names instead of IPs.
42. Segment frontend and backend networks.
43. Avoid hardcoding credentials.
44. Do not bake secrets into images.
45. Use secret managers when possible.
46. Use BuildKit secrets for build-time credentials.
47. Scan images regularly.
48. Rebuild images for security patches.
49. Sign release images.
50. Track SBOMs.
51. Use trusted base images.
52. Review high-severity CVEs.
53. Minimize runtime packages.
54. Prefer slim or distroless images where compatible.
55. Keep debug tooling out of runtime image when possible.
56. Use debug variants for troubleshooting.
57. Prefer immutable deployments.
58. Replace containers instead of patching them.
59. Keep Compose files organized.
60. Use profiles for optional services.
61. Use health-based dependencies when available.
62. Make apps tolerate dependency startup delays.
63. Validate configuration on startup.
64. Fail fast for missing required config.
65. Keep default configs safe.
66. Use environment variables carefully.
67. Avoid leaking secrets in logs.
68. Review port exposure after every deployment change.
69. Inspect images for unnecessary files.
70. Remove `.git` and local artifacts from contexts.
71. Keep CI pipelines reproducible.
72. Build in clean environments when possible.
73. Use lockfiles.
74. Avoid manual builds on pet servers.
75. Promote the same artifact across environments.
76. Test images locally before pushing.
77. Check image history when investigating size.
78. Use `docker inspect` often.
79. Use `docker logs` first in incidents.
80. Learn `nsenter` for advanced debugging.
81. Monitor container restarts.
82. Monitor OOM events.
83. Monitor resource saturation.
84. Monitor storage usage.
85. Monitor volume growth.
86. Audit registry permissions.
87. Rotate registry tokens.
88. Limit CI credentials.
89. Use short-lived credentials where possible.
90. Review daemon access carefully.
91. Treat `docker` group membership as sensitive.
92. Consider rootless for developer environments.
93. Verify signal handling in apps.
94. Use one main concern per container.
95. Split workers from web services.
96. Externalize state.
97. Keep containers disposable.
98. Do not assume startup order equals readiness.
99. Add retry logic for dependencies.
100. Keep health checks lightweight.
101. Document exposed ports.
102. Document required environment variables.
103. Document mount points.
104. Document run commands.
105. Keep images architecture-aware.
106. Know your target CPU architecture.
107. Use multi-arch builds when needed.
108. Test on the same OS family when possible.
109. Understand libc differences.
110. Validate Alpine compatibility before adopting it.
111. Do not over-optimize size at the cost of operability blindly.
112. Use distroless intentionally.
113. Keep shell access available in debug workflows.
114. Avoid cron inside app containers when a scheduler exists elsewhere.
115. Prefer one-off job containers for migrations.
116. Make migrations safe to rerun.
117. Use reverse proxies for edge concerns.
118. Keep TLS termination strategy explicit.
119. Separate public and private services.
120. Restrict egress where feasible.
121. Validate DNS resolution during debugging.
122. Validate bind addresses during debugging.
123. Inspect `/etc/resolv.conf` when needed.
124. Use network aliases deliberately.
125. Avoid relying on container IP ordering.
126. Review Docker daemon defaults.
127. Understand storage driver behavior.
128. Learn overlay copy-on-write costs.
129. Avoid heavy write workloads in container layer.
130. Use volumes for persistent write-heavy data.
131. Match host and container UIDs where needed.
132. Avoid blanket `chmod 777` fixes.
133. Apply precise file permissions.
134. Keep runtime users unprivileged.
135. Do not grant `CAP_SYS_ADMIN` casually.
136. Add only required capabilities.
137. Use `no-new-privileges`.
138. Check whether `HEALTHCHECK` binaries exist in the image.
139. Tune health check intervals thoughtfully.
140. Distinguish startup probes from runtime checks conceptually.
141. Review Compose overrides carefully.
142. Keep production and local dev configurations separate.
143. Avoid gigantic monolithic Compose files when modularity helps.
144. Name volumes clearly.
145. Name networks clearly.
146. Name containers only when it aids operations.
147. Avoid name collisions in shared hosts.
148. Use labels for metadata.
149. Standardize OCI labels.
150. Record source repository metadata.
151. Record version metadata.
152. Record build timestamps when useful.
153. Preserve reproducibility when adding metadata.
154. Avoid nondeterministic build steps when possible.
155. Cache dependency layers strategically.
156. Keep frequently changing files late in the Dockerfile.
157. Group related package installs.
158. Verify apt cleanup in same layer.
159. Use `--no-install-recommends` on Debian/Ubuntu.
160. Prefer `pip --no-cache-dir`.
161. Prefer `npm ci` over `npm install` in CI images.
162. Keep lockfiles committed.
163. Validate `.dockerignore` regularly.
164. Exclude secrets from context.
165. Exclude test artifacts if not needed.
166. Exclude local virtual environments.
167. Exclude dependency caches that will be reinstalled.
168. Do not copy SSH keys into images.
169. Do not copy cloud credentials into images.
170. Prefer runtime IAM integration when possible.
171. Separate build-time and runtime credentials.
172. Treat build logs as potentially sensitive.
173. Review image layers with history tools.
174. Prune stale resources carefully.
175. Avoid destructive cleanup on shared hosts.
176. Check restart policy behavior.
177. Use `unless-stopped` intentionally.
178. Use `on-failure` for crash-sensitive jobs when appropriate.
179. Avoid infinite restart loops hiding root causes.
180. Inspect exit codes.
181. Inspect health state.
182. Inspect OOMKilled state.
183. Use `docker events` during incident timelines.
184. Capture logs before cleanup.
185. Keep operational runbooks current.
186. Document dependency ports.
187. Document service names.
188. Prefer declarative infrastructure.
189. Version-control Dockerfiles and Compose files.
190. Review changes with security lens.
191. Scan base images before adoption.
192. Reassess old images periodically.
193. Decommission unused registries and repos.
194. Remove stale tags when appropriate.
195. Keep retention policies sane.
196. Align container tags with release processes.
197. Use semantic versioning where it fits.
198. Use immutable deployment references where possible.
199. Understand orchestration readiness semantics.
200. Understand graceful termination timeouts.
201. Test stop behavior under load.
202. Test failover behavior.
203. Test cold-start latency.
204. Test image pull time in real environments.
205. Watch for large image regressions.
206. Benchmark startup and memory patterns.
207. Profile app behavior under cgroup limits.
208. Tune JVM/.NET/Node settings for containers.
209. Make logs structured when possible.
210. Keep timestamps consistent.
211. Include request correlation IDs.
212. Send logs to platform collectors, not local files only.
213. Expose metrics endpoints.
214. Monitor container health separately from app SLA.
215. Define SLO-aware probes.
216. Use sidecars sparingly and intentionally.
217. Prefer simpler architectures when sidecars are unnecessary.
218. Separate protocol adaptation concerns cleanly.
219. Use init containers or one-off jobs for prep tasks.
220. Avoid giant entrypoint scripts when better patterns exist.
221. Keep entrypoint scripts explicit if you need them.
222. Use `exec` in shell scripts to hand off PID 1.
223. Avoid signal-swallowing wrappers.
224. Ensure child processes are reaped.
225. Validate locale/timezone assumptions.
226. Keep time synchronization on hosts healthy.
227. Understand if the app needs writable home directories.
228. Mount writable state explicitly.
229. Audit container device access.
230. Avoid mounting Docker socket unless absolutely necessary.
231. Treat Docker socket access as highly privileged.
232. Isolate build systems from runtime systems when possible.
233. Keep registries near deployment targets when performance matters.
234. Use content trust and signing where possible.
235. Enforce policy on allowed images.
236. Track provenance from source to image to deployment.
237. Make rollback artifacts readily available.
238. Keep previous known-good digests documented.
239. Test disaster recovery for stateful services.
240. Know whether data backups are application-consistent.
241. For databases, understand WAL/binlog behavior.
242. Monitor disk latency for storage-heavy containers.
243. Use dedicated storage classes in orchestration platforms when relevant.
244. Review kernel compatibility for specialized workloads.
245. Know when VM isolation is more appropriate.
246. Avoid overloading containers with unrelated services.
247. Keep operational ownership clear.
248. Teach teams image hygiene early.
249. Teach teams runtime hardening early.
250. Teach teams troubleshooting fundamentals early.
251. Teach teams difference between build-time and runtime.
252. Teach teams difference between tag and digest.
253. Teach teams difference between liveness and readiness.
254. Teach teams difference between bind mounts and volumes.
255. Teach teams why `localhost` inside a container is not the host.
256. Teach teams why `EXPOSE` is documentation, not publishing.
257. Teach teams why a stopped main process stops the container.
258. Teach teams why interactive fixes are temporary and dangerous.
259. Teach teams how to gather incident data before deleting containers.
260. Teach teams how to reproduce failures with same image digest.
261. Teach teams to design for replacement, not repair.
262. Teach teams to externalize state and config.
263. Teach teams to build once and deploy many times.
264. Teach teams to prefer simplicity over fashionable complexity.
265. Teach teams to document required capabilities and permissions.
266. Teach teams to question every privileged setting.
267. Teach teams to review networking assumptions.
268. Teach teams to review mount assumptions.
269. Teach teams to practice rootless workflows.
270. Teach teams to scan and sign continuously.
271. Teach teams to keep base images current.
272. Teach teams to reduce container startup dependencies when possible.
273. Teach teams to use queues and retries thoughtfully.
274. Teach teams to avoid coupling boot success to slow optional services.
275. Teach teams to use structured, queryable logs.
276. Teach teams to use reproducible build tools.
277. Teach teams to keep supply chain visibility.
278. Teach teams to adopt SBOMs pragmatically.
279. Teach teams to align app shutdown with orchestrator expectations.
280. Teach teams to reserve capacity for spikes.
281. Teach teams to tune GC for memory-limited containers.
282. Teach teams to observe file descriptor limits when relevant.
283. Teach teams to check DNS early when connectivity fails.
284. Teach teams to check health state early when restarts happen.
285. Teach teams to check exit codes before speculating.
286. Teach teams to check `docker inspect` before rebuilding blindly.
287. Teach teams to avoid chasing every CVE without context.
288. Teach teams to prioritize exploitable risk.
289. Teach teams to record operational decisions near the code.
290. Teach teams to review Compose and Dockerfile changes together.
291. Teach teams to keep host patching disciplined.
292. Teach teams to keep container runtimes updated.
293. Teach teams to validate runtime behavior in staging.
294. Teach teams to rehearse incident response with containers.
295. Teach teams to measure deployment rollback time.
296. Teach teams to keep image promotion pipelines auditable.
297. Teach teams to minimize manual production access.
298. Teach teams to rely on automation and policy.
299. Teach teams to verify assumptions with tests.
300. Teach teams that container excellence is mostly disciplined systems engineering.

---

# End of Guide
