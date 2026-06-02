# 11. Troubleshooting

## 11.1 Troubleshooting Mindset

Effective container troubleshooting starts with the right questions:

- Did the container start?
- Is the main process running?
- Is it healthy or merely alive?
- Is networking correct?
- Is storage mounted correctly?
- Are permissions or resource limits causing failure?

## 11.2 First Inspection Commands

```bash
docker ps -a
```

```bash
docker logs <container>
```

```bash
docker inspect <container>
```

```bash
docker stats
```

## 11.3 `docker logs`

Use logs first for quick signal.

Examples:

```bash
docker logs api
```

```bash
docker logs -f --tail=100 api
```

Look for:

- Startup exceptions
- Missing environment variables
- Port binding errors
- Database connection failures
- Permission errors

## 11.4 `docker inspect`

`docker inspect` reveals low-level container configuration and state.
Useful fields include:

- Entrypoint/command
- Environment variables
- Mounts
- Network settings
- Restart count
- State exit code

Example:

```bash
docker inspect api
```

## 11.5 Exit Codes Matter

| Exit Code | Typical Meaning |
|---|---|
| `0` | Success/completed normally |
| `1` | Generic application error |
| `126` | Command invoked cannot execute |
| `127` | Command not found |
| `137` | Killed, often OOM or SIGKILL |
| `143` | Terminated with SIGTERM |

## 11.6 `docker stats`

Monitor live resource usage:

```bash
docker stats
```

This helps detect:

- Memory pressure
- CPU saturation
- Unexpected spikes

## 11.7 Common Failure: App Exits Immediately

Cause:

- Main process completed
- Wrong command/entrypoint
- Missing dependencies
- Config error

Checks:

- `docker ps -a`
- `docker logs`
- `docker inspect` for command

## 11.8 Common Failure: Crash Loop

Possible causes:

- Missing dependency service
- Bad environment variables
- Permission issue
- Health check killing container repeatedly
- OOM kill

## 11.9 Common Failure: Port Not Reachable

Checklist:

- Is port published with `-p`?
- Is app listening on `0.0.0.0`, not `127.0.0.1`?
- Is firewall blocking host port?
- Is correct network mode used?

## 11.10 Common Failure: Database Connection Refused

Checklist:

- Is DB container running?
- Are both services on same network?
- Is hostname correct?
- Has DB finished startup?
- Are credentials valid?

## 11.11 Debugging with `docker exec`

If the container stays running:

```bash
docker exec -it api sh
```

Then inspect:

- Environment
- Config files
- DNS resolution
- Open ports
- File permissions

## 11.12 Debugging Crashed Containers

If the container exits too quickly, override entrypoint/command temporarily.

Example:

```bash
docker run --rm -it --entrypoint sh myapp:latest
```

This lets you inspect the filesystem and environment manually.

## 11.13 Using `nsenter`

`nsenter` lets you enter namespaces of a target process.
This is powerful for advanced debugging.

Example workflow:

1. Find host PID of container process
2. Enter namespaces

```bash
docker inspect --format '{{.State.Pid}}' api
```

```bash
sudo nsenter --target <pid> --mount --uts --ipc --net --pid
```

Use cases:

- Low-level network debugging
- Namespace inspection
- Investigating weird runtime behavior

## 11.14 Inspecting Network Configuration

```bash
docker network inspect app-net
```

```bash
docker exec -it api ip addr
```

```bash
docker exec -it api ip route
```

```bash
docker exec -it api cat /etc/resolv.conf
```

## 11.15 Inspecting Mounts

```bash
docker inspect api --format '{{json .Mounts}}'
```

Inside container:

```bash
mount
```

Check for:

- Wrong path
- Missing volume
- Read-only vs read-write mismatch
- Ownership errors

## 11.16 OOM Kill Diagnosis

Symptoms:

- Exit code 137
- Container restarts unexpectedly
- Kernel messages show OOM kill

Checks:

```bash
docker inspect api --format '{{.State.OOMKilled}}'
```

Mitigations:

- Increase memory limit
- Reduce app memory usage
- Tune JVM/GC/runtime settings
- Fix leaks

## 11.17 Health Check Issues

Common pitfalls:

- Health check command missing from image
- Endpoint not ready during startup
- Timeout too strict
- Probe too expensive

Inspect health state:

```bash
docker inspect api --format '{{json .State.Health}}'
```

## 11.18 Minimal Images and Debugging

Small images often lack tools like:

- `curl`
- `ps`
- `ping`
- `bash`

Options:

- Use temporary debug image/container on same network
- Add diagnostics only to a debug variant, not production image
- Use orchestration debug mechanisms

## 11.19 File Permission Problems

Symptoms:

- `Permission denied`
- Cannot write logs/data
- Cannot read mounted config

Checks:

- `id`
- `ls -l`
- Mount ownership on host
- Container `USER`

## 11.20 DNS Troubleshooting

Checks:

- Service names
- Attached networks
- `/etc/resolv.conf`
- Embedded DNS behavior on user-defined network

Temporary checks:

```bash
docker exec -it api getent hosts db
```

```bash
docker exec -it api nslookup db
```

## 11.21 Signal and Shutdown Debugging

If your app does not stop gracefully:

- Check PID 1 behavior
- Use exec form in Dockerfile
- Ensure the app handles SIGTERM
- Avoid shell wrappers that swallow signals unnecessarily

## 11.22 Log Driver Considerations

Docker supports multiple log drivers.
If logs are missing, inspect logging configuration.

```bash
docker inspect api --format '{{.HostConfig.LogConfig.Type}}'
```

## 11.23 `docker events`

Real-time event stream can help track restarts and state changes.

```bash
docker events
```

## 11.24 Cleaning Broken State

Sometimes stale containers, networks, or volumes cause confusion.
Carefully clean up:

```bash
docker compose down
```

```bash
docker container prune
```

```bash
docker network prune
```

```bash
docker volume ls
```

Do not destroy needed data accidentally.

## 11.25 Troubleshooting Workflow Checklist

1. Check container state
2. Read logs
3. Inspect configuration
4. Validate networking
5. Validate mounts and permissions
6. Check resource constraints
7. Reproduce interactively if needed
8. Use namespace-level tools for advanced cases

## 11.26 Summary

Troubleshooting containers is easiest when you remember that containers are processes with namespaces, cgroups, networks, and mounts.
Start with logs and inspect output; escalate to `exec`, `nsenter`, and runtime internals only when needed.

---

# 13. Command Cheat Sheet

## 13.1 Image Commands

```bash
docker pull nginx:stable
```

```bash
docker image ls
```

```bash
docker image inspect nginx:stable
```

```bash
docker build -t myapp:latest .
```

```bash
docker push registry.example.com/team/myapp:1.0.0
```

## 13.2 Container Commands

```bash
docker run --rm hello-world
```

```bash
docker run -d --name web -p 8080:80 nginx:stable
```

```bash
docker ps
```

```bash
docker ps -a
```

```bash
docker stop web
```

```bash
docker rm web
```

```bash
docker exec -it web sh
```

```bash
docker logs -f web
```

```bash
docker inspect web
```

```bash
docker stats
```

## 13.3 Network Commands

```bash
docker network ls
```

```bash
docker network create app-net
```

```bash
docker network inspect app-net
```

```bash
docker network connect app-net api
```

## 13.4 Volume Commands

```bash
docker volume ls
```

```bash
docker volume create pgdata
```

```bash
docker volume inspect pgdata
```

## 13.5 Compose Commands

```bash
docker compose up -d
```

```bash
docker compose ps
```

```bash
docker compose logs -f
```

```bash
docker compose down
```

## 13.6 Cleanup Commands

```bash
docker container prune
```

```bash
docker image prune
```

```bash
docker volume prune
```

```bash
docker system prune
```

---

## Appendix A.8 Troubleshooting Quick Reference

- Logs first
- Inspect config second
- Check mounts and networks
- Verify health and resource limits
- Use `nsenter` for advanced issues

---

## B.11 Troubleshooting Q&A

### Q201. What command should you try first for many container failures?
A201. `docker logs`.

### Q202. What exit code often indicates OOM kill or SIGKILL?
A202. 137.

### Q203. What exit code indicates command not found?
A203. 127.

### Q204. What command shows detailed container config and state?
A204. `docker inspect`.

### Q205. What command shows real-time usage?
A205. `docker stats`.

### Q206. How do you inspect inside a running container?
A206. `docker exec -it <name> sh`.

### Q207. How do you debug a container that exits immediately?
A207. Override entrypoint or command to start a shell.

### Q208. What command reveals the host PID of a container process?
A208. `docker inspect --format '{{.State.Pid}}' <name>`.

### Q209. What tool enters target namespaces?
A209. `nsenter`.

### Q210. What is a common reason a published port is unreachable?
A210. The app binds only to container loopback.

### Q211. Why inspect mounts during troubleshooting?
A211. Wrong paths or read-only settings can break apps.

### Q212. Why inspect environment variables?
A212. Missing or incorrect config is a frequent cause of startup failure.

### Q213. How can health checks cause trouble?
A213. They can mark recovering apps unhealthy or fail if required tools are absent.

### Q214. What command shows Docker events in real time?
A214. `docker events`.

### Q215. Why are minimal images harder to debug?
A215. They lack common diagnostic tools.

### Q216. What does `.State.OOMKilled` show?
A216. Whether Docker recorded an OOM kill.

### Q217. Why do permissions matter more with non-root images?
A217. The process cannot simply bypass ownership constraints.

### Q218. Why should cleanup commands be used carefully?
A218. They can destroy needed state or data.

### Q219. Why are network inspection tools useful?
A219. They reveal connectivity, DNS, and route issues.

### Q220. What is the best general troubleshooting order?
A220. Logs, inspect, runtime checks, network/storage checks, then low-level tools.

---

# Appendix D: 200 Common Commands with Brief Explanations

1. `docker version` — Show client and server version details.
2. `docker info` — Show engine configuration and environment data.
3. `docker login` — Authenticate to a registry.
4. `docker logout` — Remove registry credentials.
5. `docker search nginx` — Search Docker Hub images.
6. `docker pull alpine:3.20` — Download an image.
7. `docker push repo/app:1.0.0` — Upload an image.
8. `docker image ls` — List local images.
9. `docker image rm image:tag` — Remove a local image.
10. `docker image inspect image:tag` — Inspect image metadata.
11. `docker image history image:tag` — Show image layer history.
12. `docker build -t app:dev .` — Build image from Dockerfile.
13. `docker build --no-cache -t app:dev .` — Build without cache.
14. `docker build --target runtime -t app:runtime .` — Build a specific stage.
15. `docker build --build-arg VERSION=1.0.0 -t app:1.0.0 .` — Pass build arg.
16. `DOCKER_BUILDKIT=1 docker build .` — Enable BuildKit for build.
17. `docker run --rm hello-world` — Run test container.
18. `docker run -it ubuntu:24.04 bash` — Run interactive shell.
19. `docker run -d nginx:stable` — Run detached container.
20. `docker run --name web nginx:stable` — Name the container.
21. `docker run -p 8080:80 nginx:stable` — Publish a port.
22. `docker run -e APP_ENV=prod app:latest` — Set env var.
23. `docker run --env-file .env app:latest` — Load env file.
24. `docker run -v myvol:/data app:latest` — Mount named volume.
25. `docker run -v $(pwd):/app app:latest` — Bind mount current directory.
26. `docker run --mount type=tmpfs,dst=/tmp app:latest` — Mount tmpfs.
27. `docker run --network app-net app:latest` — Attach to network.
28. `docker run --cpus=1 app:latest` — Limit CPU.
29. `docker run --memory=512m app:latest` — Limit memory.
30. `docker run --pids-limit=200 app:latest` — Limit process count.
31. `docker run --read-only app:latest` — Read-only root filesystem.
32. `docker run --cap-drop=ALL app:latest` — Drop all capabilities.
33. `docker run --cap-add=NET_BIND_SERVICE app:latest` — Add one capability.
34. `docker run --security-opt no-new-privileges:true app:latest` — Block new privilege gain.
35. `docker run --user 10001:10001 app:latest` — Run as specific UID/GID.
36. `docker run --restart unless-stopped app:latest` — Set restart policy.
37. `docker ps` — List running containers.
38. `docker ps -a` — List all containers.
39. `docker stop web` — Stop container gracefully.
40. `docker kill web` — Send SIGKILL.
41. `docker start web` — Start stopped container.
42. `docker restart web` — Restart container.
43. `docker rm web` — Remove stopped container.
44. `docker rm -f web` — Force-remove container.
45. `docker rename old new` — Rename container.
46. `docker logs web` — Show container logs.
47. `docker logs -f web` — Follow logs.
48. `docker logs --tail=100 web` — Show last 100 lines.
49. `docker exec -it web sh` — Open shell in running container.
50. `docker exec web env` — Print env vars inside container.
51. `docker cp web:/app/file ./file` — Copy file from container.
52. `docker cp ./file web:/app/file` — Copy file into container.
53. `docker inspect web` — Inspect container JSON.
54. `docker inspect --format '{{.State.Pid}}' web` — Show host PID.
55. `docker top web` — Show container processes.
56. `docker stats` — Show live usage stats.
57. `docker attach web` — Attach local terminal to container streams.
58. `docker wait web` — Wait for container exit.
59. `docker commit web debug-image:latest` — Snapshot a container into image.
60. `docker diff web` — Show container filesystem changes.
61. `docker network ls` — List networks.
62. `docker network create app-net` — Create network.
63. `docker network inspect app-net` — Inspect network.
64. `docker network connect app-net web` — Connect container to network.
65. `docker network disconnect app-net web` — Disconnect container.
66. `docker network rm app-net` — Remove network.
67. `docker volume ls` — List volumes.
68. `docker volume create pgdata` — Create volume.
69. `docker volume inspect pgdata` — Inspect volume.
70. `docker volume rm pgdata` — Remove volume.
71. `docker container prune` — Remove stopped containers.
72. `docker image prune` — Remove dangling images.
73. `docker image prune -a` — Remove unused images.
74. `docker volume prune` — Remove unused volumes.
75. `docker network prune` — Remove unused networks.
76. `docker system prune` — Remove unused resources.
77. `docker system df` — Show Docker disk usage.
78. `docker context ls` — List Docker contexts.
79. `docker context use default` — Switch context.
80. `docker compose up` — Start Compose app.
81. `docker compose up -d` — Start Compose app detached.
82. `docker compose down` — Stop/remove Compose app.
83. `docker compose down -v` — Stop/remove app and volumes.
84. `docker compose ps` — Show Compose service state.
85. `docker compose logs -f` — Follow Compose logs.
86. `docker compose build` — Build Compose services.
87. `docker compose pull` — Pull service images.
88. `docker compose exec api sh` — Exec into a service container.
89. `docker compose restart api` — Restart one service.
90. `docker compose config` — Render resolved Compose config.
91. `docker compose top` — Show service processes.
92. `docker compose images` — Show images used by services.
93. `docker compose up --scale api=3` — Scale service replicas.
94. `docker swarm init` — Initialize swarm.
95. `docker node ls` — List swarm nodes.
96. `docker service ls` — List swarm services.
97. `docker service ps web` — Show service tasks.
98. `docker service inspect web` — Inspect service.
99. `docker service scale web=5` — Scale swarm service.
100. `docker service rm web` — Remove swarm service.
101. `podman version` — Show Podman version.
102. `podman run --rm alpine:3.20 echo hi` — Run with Podman.
103. `podman ps` — List Podman containers.
104. `podman images` — List Podman images.
105. `buildah bud -t app:latest .` — Build image with Buildah.
106. `skopeo inspect docker://docker.io/library/nginx:stable` — Inspect remote image.
107. `ctr images ls` — List images in containerd.
108. `ctr containers ls` — List containers in containerd.
109. `nerdctl ps` — Docker-like CLI for containerd environments.
110. `trivy image app:latest` — Scan image for vulnerabilities.
111. `grype app:latest` — Scan image with Grype.
112. `cosign sign repo/app:1.0.0` — Sign image.
113. `cosign verify repo/app:1.0.0` — Verify image signature.
114. `lsns` — List namespaces.
115. `readlink /proc/<pid>/ns/net` — Inspect a namespace reference.
116. `cat /proc/<pid>/cgroup` — Show process cgroup membership.
117. `mount | grep overlay` — Inspect overlay mounts.
118. `sudo nsenter --target <pid> --net` — Enter network namespace.
119. `sudo nsenter --target <pid> --mount --pid --ipc --uts --net` — Enter multiple namespaces.
120. `ip addr` — Show interfaces.
121. `ip route` — Show routing table.
122. `ss -lntp` — Show listening TCP sockets.
123. `ps -ef` — Show processes.
124. `pstree -a` — Show process tree.
125. `free -m` — Show memory usage.
126. `df -h` — Show disk space.
127. `du -sh .` — Show directory size.
128. `curl http://localhost:8080/health` — Test HTTP endpoint.
129. `wget -qO- http://localhost:8080` — Fetch page content.
130. `nc -zv db 5432` — Test TCP connectivity.
131. `ping db` — Test basic connectivity.
132. `getent hosts db` — Resolve hostname.
133. `cat /etc/resolv.conf` — Inspect DNS resolver config.
134. `env | sort` — Review environment variables.
135. `id` — Show current user identity.
136. `ls -l /path` — Inspect permissions.
137. `stat /path` — Show file metadata.
138. `find . -maxdepth 2 -type f` — List files for context checks.
139. `tar czf backup.tar.gz data/` — Make archive backup.
140. `tar xzf backup.tar.gz` — Restore archive.
141. `journalctl -u docker` — Show Docker service logs on systemd systems.
142. `systemctl status docker` — Show Docker daemon status.
143. `systemctl restart docker` — Restart Docker daemon.
144. `docker buildx ls` — Show buildx builders.
145. `docker buildx create --use` — Create and use a builder.
146. `docker buildx build --platform linux/amd64,linux/arm64 .` — Multi-platform build.
147. `docker buildx build --push -t repo/app:1.0.0 .` — Build and push.
148. `docker save image:tag -o image.tar` — Save image archive.
149. `docker load -i image.tar` — Load image archive.
150. `docker tag app:latest repo/app:1.0.0` — Tag image.
151. `docker export web -o web.tar` — Export container filesystem.
152. `docker import web.tar imported:latest` — Import image from tar.
153. `docker inspect --format '{{json .Mounts}}' web` — Show mounts JSON.
154. `docker inspect --format '{{.HostConfig.NetworkMode}}' web` — Show network mode.
155. `docker inspect --format '{{.State.OOMKilled}}' web` — Show OOM state.
156. `docker inspect --format '{{.HostConfig.RestartPolicy.Name}}' web` — Show restart policy.
157. `docker events` — Stream Docker events.
158. `docker system events` — Same general event stream.
159. `docker builder prune` — Remove build cache.
160. `docker image history --no-trunc app:latest` — Detailed image history.
161. `docker compose config --services` — List service names.
162. `docker compose config --volumes` — List volume names.
163. `docker compose pull --ignore-buildable` — Pull pullable images only.
164. `docker compose rm` — Remove stopped service containers.
165. `docker compose stop` — Stop services without removing.
166. `docker compose start` — Start existing services.
167. `docker compose pause` — Pause services.
168. `docker compose unpause` — Unpause services.
169. `docker compose events` — Stream Compose events.
170. `docker compose port web 80` — Show published port mapping.
171. `docker compose run --rm app sh` — Run one-off service command.
172. `docker compose build --no-cache` — Build Compose images without cache.
173. `docker compose up --build` — Rebuild before starting.
174. `docker compose down --remove-orphans` — Remove extra service containers.
175. `docker network create --driver bridge custom-net` — Create custom bridge.
176. `docker network create --subnet 172.28.0.0/16 custom-net` — Create bridge with subnet.
177. `docker run --network none app:latest` — Disable networking.
178. `docker run --network host app:latest` — Use host networking.
179. `docker run --add-host=host.docker.internal:host-gateway app:latest` — Add host gateway mapping.
180. `docker run --health-cmd='curl -f http://localhost:8080/health || exit 1' app:latest` — Set runtime health check.
181. `docker run --entrypoint sh app:latest` — Override entrypoint.
182. `docker run -w /app app:latest` — Set runtime working directory.
183. `docker run --hostname app1 app:latest` — Set hostname.
184. `docker run --dns 1.1.1.1 app:latest` — Override DNS server.
185. `docker run --ipc=host app:latest` — Share host IPC namespace.
186. `docker run --pid=host app:latest` — Share host PID namespace.
187. `docker run --uts=host app:latest` — Share host UTS namespace.
188. `docker run --cgroupns=host app:latest` — Share host cgroup namespace.
189. `docker run --device /dev/fuse app:latest` — Pass through device.
190. `docker run --ulimit nofile=65535:65535 app:latest` — Set file descriptor limit.
191. `docker run --init app:latest` — Use tiny init process to reap zombies.
192. `docker run --sig-proxy=true app:latest` — Proxy signals to process.
193. `docker run --stop-timeout 30 app:latest` — Set stop timeout.
194. `docker run --stop-signal SIGTERM app:latest` — Set stop signal.
195. `docker run --label com.example.role=api app:latest` — Apply metadata label.
196. `docker inspect --format '{{json .Config.Labels}}' app` — Show labels.
197. `docker plugin ls` — List Docker plugins.
198. `docker volume prune -f` — Force prune unused volumes.
199. `docker image prune -af` — Remove all unused images.
200. `docker system prune -af --volumes` — Aggressively prune unused resources and volumes.

---

# Appendix E: Scenario Walkthroughs

## Scenario E.1: My container exits immediately

Symptoms:

- `docker ps` does not show it running
- `docker ps -a` shows it exited

Steps:

1. `docker logs <name>`
2. `docker inspect <name>`
3. Check exit code
4. Verify entrypoint and command
5. Run with `--entrypoint sh` for manual inspection

## Scenario E.2: Port mapping exists but app is unreachable

Symptoms:

- `docker ps` shows `0.0.0.0:8080->8080/tcp`
- `curl localhost:8080` fails

Likely causes:

- App bound to `127.0.0.1`
- App crashed after startup
- Firewall issue
- Wrong container port in mapping

## Scenario E.3: Database container starts, app still fails

Symptoms:

- DB container is running
- App logs show connection refused

Likely causes:

- Startup race
- Wrong hostname
- Wrong network
- Wrong credentials
- DB not ready yet

## Scenario E.4: Non-root container cannot write files

Symptoms:

- Permission denied on startup

Likely causes:

- Bind mount ownership mismatch
- Image creates directories as root
- Runtime `USER` not aligned with data path ownership

Fixes:

- Pre-create/chown directory
- Use matching UID/GID
- Use named volumes if cleaner

## Scenario E.5: Image is too large

Checklist:

- Use multi-stage build
- Choose slimmer base image
- Review `.dockerignore`
- Reorder layers for cache and minimal content
- Remove build tools from final stage
- Inspect image history

## Scenario E.6: Container OOMKilled

Checklist:

- Inspect `.State.OOMKilled`
- Check memory limit
- Profile memory behavior
- Tune runtime memory settings
- Reduce concurrency if needed

## Scenario E.7: Rootless mode behaves differently

Checklist:

- Confirm user namespace mapping
- Check file permission behavior
- Review networking assumptions
- Verify unsupported privileged features are not required

## Scenario E.8: Health check always fails

Checklist:

- Confirm tool like `curl` exists
- Verify endpoint path and port
- Relax timing during slow startup
- Separate readiness from liveness logic

## Scenario E.9: Compose stack works locally but not in CI

Checklist:

- Check bind mount assumptions
- Check architecture differences
- Check missing environment variables
- Check race conditions masked locally
- Check service health timing

## Scenario E.10: Production deploy uses wrong image unexpectedly

Checklist:

- Review tag mutability
- Check registry repository and namespace
- Use digest-based deployment
- Audit CI promotion pipeline

---
