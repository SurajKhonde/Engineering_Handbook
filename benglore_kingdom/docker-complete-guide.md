# Docker — Complete Guide (Concepts → Your Dockerfile → 50 Interview Questions)

A single-file reference covering what Docker is, how it differs from a VM, images vs containers, your exact Dockerfile explained line-by-line, `Dockerfile` vs `docker-compose.yml`, services, volumes, networking, and 50 well-explained interview questions.

---

## Table of Contents

1. [What is Docker?](#1-what-is-docker)
2. [The problem Docker solves](#2-the-problem-docker-solves)
3. [Docker vs Virtual Machines](#3-docker-vs-virtual-machines)
4. [Why Docker is "better" (and when it isn't)](#4-why-docker-is-better-and-when-it-isnt)
5. [Core concepts: Image, Container, Layer, Registry](#5-core-concepts)
6. [The Dockerfile (your file explained line-by-line)](#6-the-dockerfile)
7. [Improving your Dockerfile (multi-stage build)](#7-improving-your-dockerfile)
8. [`Dockerfile` vs `docker-compose.yml`](#8-dockerfile-vs-docker-composeyml)
9. [What is a "service"? (with examples)](#9-what-is-a-service)
10. [Volumes & data persistence](#10-volumes--data-persistence)
11. [Networking: how one container talks to another](#11-networking)
12. [Essential command cheat sheet](#12-command-cheat-sheet)
13. [Best practices](#13-best-practices)
14. [50 interview questions with answers](#14-50-interview-questions)

---

## 1. What is Docker?

**Docker is a tool for packaging an application together with everything it needs to run — code, runtime, system libraries, dependencies, and config — into a single, portable unit called a *container*.**

A container runs the same way on your laptop, your teammate's laptop, a CI runner, and a production server. You stop caring about "what's installed on the host" because the container carries its own environment.

Mental model:

- An **image** is a frozen, read-only snapshot of a filesystem + metadata (like a class in OOP, or a `.iso`).
- A **container** is a running instance of that image (like an object instantiated from a class).
- You build images **from a `Dockerfile`**, store/share them in a **registry** (Docker Hub, GHCR, ECR), and run them as containers.

```
Dockerfile  ──build──▶  Image  ──run──▶  Container(s)
 (recipe)              (snapshot)        (running process)
```

---

## 2. The problem Docker solves

The classic pain: **"It works on my machine."**

You build something with Node 20, a specific OpenSSL version, a particular locale, a Redis client built against a certain glibc. It runs on your machine. You ship it. Production has Node 18, a different libc, missing system packages — and it breaks.

Docker fixes this by shipping the *whole environment* in the image, not just your code. The image **is** the environment. If it runs in the container locally, it runs in the container in prod, because it's literally the same filesystem and the same processes.

Secondary wins:
- **Fast onboarding** — `docker compose up` instead of a 3-page README of installs.
- **Consistency across stages** — dev = staging = prod.
- **Isolation** — two apps needing different Node versions coexist without conflict.
- **Density** — many containers per host (lighter than VMs).

---

## 3. Docker vs Virtual Machines

Both give isolation, but at very different layers.

### Virtual Machine
A VM virtualizes **hardware**. A hypervisor (VMware, VirtualBox, KVM, Hyper-V) runs a **full guest operating system** — its own kernel, init system, drivers — on top of the host.

```
┌─────────────────────────────────────┐
│  App A      │  App B     │  App C    │
│  Bins/Libs  │  Bins/Libs │ Bins/Libs │
│  Guest OS   │  Guest OS  │  Guest OS │   ← each VM ships a full OS + kernel
├─────────────────────────────────────┤
│            Hypervisor                │
├─────────────────────────────────────┤
│            Host OS                   │
├─────────────────────────────────────┤
│            Physical Hardware         │
└─────────────────────────────────────┘
```

### Container
A container virtualizes the **operating system**. All containers **share the host kernel** and are isolated using Linux primitives: **namespaces** (isolation of process IDs, network, mounts, users) and **cgroups** (resource limits on CPU/memory). There is no guest OS and no guest kernel.

```
┌─────────────────────────────────────┐
│  App A      │  App B     │  App C    │
│  Bins/Libs  │  Bins/Libs │ Bins/Libs │   ← only what each app needs
├─────────────────────────────────────┤
│        Docker Engine                 │
├─────────────────────────────────────┤
│   Host OS (shared kernel)            │   ← ONE kernel, shared
├─────────────────────────────────────┤
│        Physical Hardware             │
└─────────────────────────────────────┘
```

### Side-by-side

| Aspect | Virtual Machine | Container |
|---|---|---|
| Isolation level | Hardware | OS / process |
| Guest OS | Full OS per VM | None — shares host kernel |
| Size | GBs | MBs (often tens to a few hundred) |
| Startup time | Seconds to minutes | Milliseconds to seconds |
| Resource overhead | High | Low |
| Density per host | Tens | Hundreds |
| Isolation strength | Stronger (separate kernels) | Weaker (shared kernel) |
| Run different OS kernels | Yes (Linux VM on Windows host, etc.) | No (Linux containers need a Linux kernel) |

> On Windows/macOS, Docker actually runs a tiny Linux VM under the hood (WSL2 / a lightweight VM), and your "Linux containers" run inside it. That's why "containers share the host kernel" is precisely true on Linux, and true *via that helper VM* on Win/Mac.

---

## 4. Why Docker is "better" (and when it isn't)

**Better when you want:**
- Fast startup and shutdown (scaling, CI, ephemeral jobs).
- High density — pack many services per machine.
- Identical environments across dev/staging/prod.
- Immutable, versioned deploys (`myapp:1.4.2`) with trivial rollback (`myapp:1.4.1`).
- A reproducible build recipe checked into git.

**A VM is still better when you need:**
- **Strong security isolation** between untrusted tenants — separate kernels are a harder boundary than namespaces.
- A **different OS/kernel** than the host.
- To run things that need deep kernel access or custom kernel modules.

In practice many teams combine them: VMs as the host fleet, containers running inside them. "Better" is contextual — Docker wins on speed, portability, and density; VMs win on isolation strength and kernel flexibility.

---

## 5. Core concepts

### Image
A read-only template built from a stack of **layers**. Each instruction in a Dockerfile (`FROM`, `RUN`, `COPY`, …) usually creates a layer. Layers are cached and shared between images, which is why builds and pulls are fast.

### Container
A running (or stopped) instance of an image. It adds a thin **writable layer** on top of the image's read-only layers. Anything the running process writes goes into that writable layer — and **disappears when the container is removed** unless you use a volume.

### Layer
An incremental filesystem diff. Because layers are content-addressed and cached, Docker can reuse unchanged layers across builds. This is the heart of build-cache optimization (see §6/§13).

### Registry
A store for images. Public: Docker Hub, GitHub Container Registry (GHCR). Private/cloud: AWS ECR, GCP Artifact Registry. You `docker push` to publish and `docker pull` to fetch. `node:20` means image `node`, tag `20`, pulled from Docker Hub by default.

### Docker Engine / daemon / CLI
The **daemon** (`dockerd`) does the actual work — builds images, runs containers. The **CLI** (`docker`) is what you type; it talks to the daemon over an API.

---

## 6. The Dockerfile

A **Dockerfile** is the recipe to build an image — a sequence of instructions executed top-to-bottom. Here's **your** Dockerfile, explained line by line.

```dockerfile
FROM node:20

WORKDIR /app

RUN corepack enable && corepack prepare pnpm@10.33.0 --activate

COPY package.json pnpm-lock.yaml ./

RUN pnpm install

COPY . .

RUN pnpm build

EXPOSE 5000

CMD ["pnpm", "start"]
```

| Line | What it does |
|---|---|
| `FROM node:20` | Starts from the official Node 20 image (Debian-based). This sets the base filesystem: Node + npm + a Linux userland are already present. Every image must start with a `FROM`. |
| `WORKDIR /app` | Creates `/app` (if absent) and makes it the working directory. Every later `COPY`, `RUN`, `CMD` runs relative to `/app`. Cleaner than `RUN cd /app`. |
| `RUN corepack enable && corepack prepare pnpm@10.33.0 --activate` | Corepack ships with Node and manages package-manager versions. This turns on `pnpm` and **pins it to 10.33.0** so the build is reproducible regardless of what's globally installed. |
| `COPY package.json pnpm-lock.yaml ./` | Copies **only** the manifest and lockfile first. This is the key **layer-caching trick** — see note below. |
| `RUN pnpm install` | Installs dependencies into `node_modules`. Because only the lockfile/manifest were copied above, this layer is **cached** and only re-runs when those files change. |
| `COPY . .` | Now copies the rest of your source code into `/app`. Changing source invalidates *this* layer (and below) but **not** the install layer above. |
| `RUN pnpm build` | Runs your build script (e.g. Next.js build / TypeScript compile) to produce production output. |
| `EXPOSE 5000` | **Documentation only.** It declares the app listens on 5000. It does **not** publish the port — you still need `-p 5000:5000` (or `ports:` in Compose) to reach it from the host. |
| `CMD ["pnpm", "start"]` | The default command run when a container starts. Exec form (JSON array) is preferred — it runs the process directly (PID 1) instead of via a shell, so signals like `SIGTERM` reach your app for graceful shutdown. |

### The layer-caching trick (why COPY is split)

Dependencies change far less often than source code. By copying `package.json` + lockfile *first*, installing, *then* copying the rest:

- Edit a `.ts` file → only `COPY . .`, `pnpm build` re-run. Install stays cached. **Fast.**
- Change a dependency → lockfile changes → install re-runs. **Correct.**

If you did `COPY . .` before `pnpm install`, **every** source edit would bust the install cache and re-download everything. Slow.

### `RUN` vs `CMD` vs `ENTRYPOINT`
- `RUN` — executes **at build time**, creating a layer (install stuff, compile).
- `CMD` — the **default command at runtime**; easily overridden by `docker run myimg <other-cmd>`.
- `ENTRYPOINT` — the fixed executable at runtime; `CMD` becomes its default arguments. Common pattern: `ENTRYPOINT ["node"]` + `CMD ["server.js"]`.

---

## 7. Improving your Dockerfile

Your Dockerfile works, but for production it has three issues: the final image is **huge** (full Node 20 + build toolchain + dev deps + source), it runs as **root**, and `pnpm install` isn't reproducible. A **multi-stage build** fixes all three.

```dockerfile
# ---- Stage 1: build ----
FROM node:20-slim AS build
WORKDIR /app

RUN corepack enable && corepack prepare pnpm@10.33.0 --activate

COPY package.json pnpm-lock.yaml ./
# --frozen-lockfile = fail if lockfile is out of date (reproducible CI builds)
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build

# Drop dev dependencies before copying to the runtime image
RUN pnpm prune --prod

# ---- Stage 2: runtime ----
FROM node:20-slim AS runtime
WORKDIR /app
ENV NODE_ENV=production

# Copy only what's needed to run, from the build stage
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY --from=build /app/package.json ./package.json

# Run as a non-root user (security)
USER node

EXPOSE 5000
CMD ["pnpm", "start"]
```

What changed and why:

- **`node:20-slim`** (or `node:20-alpine`) → much smaller base than `node:20`.
- **Multi-stage** → the build toolchain and source never reach the final image; only the built output + production `node_modules` ship. Smaller image = faster pulls, smaller attack surface.
- **`--frozen-lockfile`** → build fails if the lockfile and manifest disagree, guaranteeing reproducible installs.
- **`pnpm prune --prod`** → strips dev dependencies.
- **`USER node`** → container process isn't root, limiting blast radius if compromised.
- **`.dockerignore`** (separate file) → keep `node_modules`, `.git`, `.env`, logs out of the build context so `COPY . .` is fast and you never leak secrets.

Example `.dockerignore`:
```
node_modules
.git
.env
.env.*
dist
*.log
Dockerfile
docker-compose.yml
```

---

## 8. `Dockerfile` vs `docker-compose.yml`

These solve **different problems** and are used together.

### `Dockerfile`
Describes **how to build ONE image**. It's a build recipe. Output: an image. It knows nothing about other services, databases, networks, or volumes between services.

### `docker-compose.yml`
Describes **how to RUN a set of containers together** — your whole app as a system. It's an orchestration/config file. It defines multiple **services**, how they network, what volumes they mount, environment variables, ports, restart policies, dependencies, etc. It can *build* images (pointing at Dockerfiles) **or** use prebuilt images from a registry.

| | `Dockerfile` | `docker-compose.yml` |
|---|---|---|
| Purpose | Build one image | Run/orchestrate many containers |
| Scope | A single service's image | The whole multi-service app |
| Format | Dockerfile instructions | YAML |
| Produces | An image | Running containers + networks + volumes |
| Command | `docker build` | `docker compose up` |
| Knows about networking between services? | No | Yes |

> Note: a single `docker-compose.yml` can reference **many** Dockerfiles (one per service). You don't have multiple compose files for one app normally — you have one compose file describing several services, each possibly with its own Dockerfile.

### How they fit together

```
Dockerfile (api)  ─┐
                   ├──▶ docker-compose.yml ──▶ docker compose up ──▶ api + db + redis running, networked
Dockerfile (worker)┘        (also pulls postgres:16, redis:7 from Docker Hub)
```

---

## 9. What is a "service"?

In Compose, a **service** = the definition of one container (or several identical replicas of it) plus its config. Roughly: "one component of your app." A typical backend has services like `api`, `worker`, `db`, `redis`.

A service definition says: which image to use (or which Dockerfile to build), what ports to expose, what env vars to set, what volumes to mount, which network to join, and what it depends on.

### Minimal example — your app + Postgres + Redis

```yaml
# docker-compose.yml
services:
  api:
    build: .                 # builds using the Dockerfile in this folder
    ports:
      - "5000:5000"          # host:container — now reachable at localhost:5000
    environment:
      DATABASE_URL: postgres://user:pass@db:5432/appdb
      REDIS_URL: redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:16       # prebuilt image from Docker Hub
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: appdb
    volumes:
      - pgdata:/var/lib/postgresql/data   # persist DB data

  redis:
    image: redis:7

volumes:
  pgdata:                    # named volume, survives container removal
```

How it works:
- `docker compose up` starts all three services on a shared default network.
- The `api` reaches Postgres at the hostname `db` and Redis at `redis` — **by their service names** (Compose gives each service a DNS name). No IP addresses needed.
- `depends_on` controls **start order** (note: it waits for the container to start, not for the DB to be *ready* — for readiness use a healthcheck).
- `volumes: pgdata` keeps your database data even if you `docker compose down` and recreate the container.

### Scaling a service
A service can run multiple identical replicas:
```bash
docker compose up --scale worker=4
```
Now four `worker` containers run behind the same service name — useful for your BullMQ-style job processing where you want N workers pulling jobs from Redis.

---

## 10. Volumes & data persistence

**Problem:** a container's writable layer is destroyed when the container is removed. Databases, uploads, logs — anything you want to keep — must live **outside** that layer. That's what volumes are for.

### Three kinds of mounts

**1. Named volume (recommended for app data)**
Managed by Docker, stored in Docker's area on the host. Best for databases.
```yaml
volumes:
  - pgdata:/var/lib/postgresql/data
```
```bash
docker volume create pgdata
docker volume ls
docker volume inspect pgdata
```

**2. Bind mount (great for local dev)**
Maps a **host folder** straight into the container. Edit code on your machine, the container sees it instantly (hot reload).
```yaml
volumes:
  - ./src:/app/src        # host path : container path
```

**3. tmpfs mount**
Stored in host RAM, never written to disk — for ephemeral secrets/scratch. Gone when the container stops.

### How isolation works
- Each container has its own filesystem view (mount namespace) — by default container A can't see container B's files.
- A **named volume can be shared** between containers by mounting the same volume in each. That's the controlled way to share data.
- A bind mount punches a hole to a specific host directory — so it's not isolated from the host by design (that's the point in dev).

| Type | Lives where | Survives container removal | Typical use |
|---|---|---|---|
| Named volume | Docker-managed area | Yes | DB data, persistent app state |
| Bind mount | A host directory you choose | Yes (it's your folder) | Local dev hot-reload |
| tmpfs | Host RAM | No | Secrets, scratch data |

> Key gotcha: `docker compose down` keeps named volumes; `docker compose down -v` **deletes** them. Don't run `-v` against a DB you care about.

---

## 11. Networking

### The big idea
When you launch containers (especially via Compose), they join a **virtual network**. Docker runs an embedded **DNS server** so containers can reach each other **by service/container name** instead of brittle IPs. In the §9 example, `api` connects to Postgres at host `db` because `db` resolves to that container on the shared network.

### Network drivers
- **bridge** (default): a private virtual network on a single host. Containers on the same bridge talk to each other; to reach them from outside you **publish ports** (`-p`/`ports:`). The *default* bridge requires linking by IP, but a **user-defined bridge** (what Compose creates) gives automatic DNS by name — always prefer it.
- **host**: container shares the host's network stack directly (no isolation, no port mapping needed). Lower overhead, less isolation. Linux only.
- **none**: no networking at all — fully isolated.
- **overlay**: spans multiple hosts (Docker Swarm / multi-node) so containers on different machines talk as if on one network.

### Ports: publish vs expose
- `EXPOSE 5000` (Dockerfile) / `expose:` (Compose) → **documentation**; makes the port reachable to *other containers on the same network*, not the host.
- `-p 5000:5000` / `ports: ["5000:5000"]` → **publishes** the port to the host so you can hit `localhost:5000`. Format is `HOST:CONTAINER`.

### How one container talks to another — concretely
```yaml
services:
  api:
    build: .
    environment:
      REDIS_URL: redis://redis:6379   # 'redis' is the service name = the hostname
  redis:
    image: redis:7
```
Inside `api`, `redis` resolves to the Redis container's IP automatically. No hardcoded IPs, no `--link`. If you scale or recreate `redis`, the name still resolves.

### Custom networks (segmentation)
You can put services on separate networks so they can't all reach each other — e.g. only `api` can reach `db`, while a public-facing `proxy` can't:
```yaml
services:
  proxy:   { image: nginx, networks: [frontend] }
  api:     { build: ., networks: [frontend, backend] }
  db:      { image: postgres:16, networks: [backend] }
networks:
  frontend:
  backend:
```
Here `proxy` ↔ `api` (frontend) and `api` ↔ `db` (backend), but `proxy` can't reach `db`. That's network-level isolation as a security boundary.

---

## 12. Command cheat sheet

```bash
# Images
docker build -t myapp:1.0 .         # build image from Dockerfile in current dir
docker images                       # list images
docker rmi myapp:1.0                # remove image
docker pull node:20                 # fetch image from registry
docker push ghcr.io/me/myapp:1.0    # publish image

# Containers
docker run -d -p 5000:5000 --name api myapp:1.0   # run detached, publish port
docker ps                           # running containers
docker ps -a                        # all containers (incl. stopped)
docker logs -f api                  # stream logs
docker exec -it api sh              # shell inside a running container
docker stop api && docker rm api    # stop + remove

# Volumes & networks
docker volume ls
docker network ls
docker network inspect bridge

# Compose
docker compose up -d                # build/start all services in background
docker compose ps                   # status of services
docker compose logs -f api          # logs for one service
docker compose exec api sh          # shell into the api service
docker compose down                 # stop + remove containers/networks
docker compose down -v              # ...and DELETE named volumes (careful!)
docker compose up --build           # rebuild images before starting

# Cleanup
docker system df                    # disk usage
docker system prune                 # remove dangling stuff
docker system prune -a --volumes    # aggressive cleanup (careful!)
```

---

## 13. Best practices

- **Use a `.dockerignore`** — keep `node_modules`, `.git`, `.env` out of the build context. Faster builds, no secret leaks.
- **Order Dockerfile from least- to most-frequently-changing** — copy lockfiles + install before copying source (cache wins).
- **Multi-stage builds** — ship only runtime artifacts; keep compilers/dev deps out of the final image.
- **Pin versions** — `node:20.11.1-slim`, not `node:latest`. Reproducibility and predictable rebuilds.
- **Small base images** — `slim`/`alpine` where compatible (watch out: alpine uses musl libc, which can break some native modules).
- **Run as non-root** (`USER node`).
- **One main process per container** — let the orchestrator manage lifecycle; don't cram nginx + node + cron into one container.
- **Healthchecks** — `HEALTHCHECK` / Compose `healthcheck:` so orchestrators know when a service is truly ready.
- **Never bake secrets into images** — use env vars, secret stores, or runtime injection. Image layers are inspectable; a secret in a layer is leaked forever even if "deleted" later.
- **Tag immutably for deploys** — `myapp:1.4.2`, and keep `latest` only as a convenience.

---

## 14. 50 Interview Questions

### Fundamentals

**1. What is Docker?**
A platform for building, shipping, and running applications in containers — lightweight, isolated units that package an app with all its dependencies so it runs identically across environments.

**2. What is a container?**
A running instance of an image: an isolated process (or group of processes) with its own filesystem, network, and process space, sharing the host kernel. Isolation comes from Linux namespaces and cgroups.

**3. What is an image?**
A read-only template — a stack of filesystem layers plus metadata — used to create containers. Built from a Dockerfile, stored in registries.

**4. Image vs container?**
An image is the static blueprint (like a class); a container is a running instance of it (like an object). One image → many containers.

**5. What is a Dockerfile?**
A text file of instructions describing how to build an image, step by step (`FROM`, `RUN`, `COPY`, `CMD`, …).

**6. What is Docker Hub / a registry?**
A registry stores and distributes images. Docker Hub is the default public one; ECR/GHCR/Artifact Registry are common private/cloud options. `push` to publish, `pull` to fetch.

**7. What is a layer?**
An incremental filesystem change produced by a Dockerfile instruction. Layers are cached and shared across images, making builds and pulls efficient.

**8. What is the Docker daemon?**
`dockerd` — the background service that builds images and runs containers. The `docker` CLI talks to it via an API.

**9. What is the difference between the Docker client and daemon?**
The client is the CLI you type commands into; the daemon does the actual work. They can even run on different machines.

**10. How does Docker achieve isolation?**
Via Linux **namespaces** (isolate PID, network, mounts, users, hostname) and **cgroups** (limit and account CPU, memory, I/O).

### Containers vs VMs

**11. Docker vs VM — core difference?**
A VM virtualizes hardware and runs a full guest OS with its own kernel; a container virtualizes the OS and shares the host kernel. Containers are far lighter and faster.

**12. Why are containers faster to start than VMs?**
No guest OS to boot — a container just starts a process. VMs must boot an entire operating system.

**13. Are containers as secure as VMs?**
Generally not as strongly isolated — containers share the host kernel, so a kernel exploit can cross the boundary. VMs have separate kernels, a harder boundary. Use VMs (or gVisor/Kata) for strong multi-tenant isolation.

**14. Can you run a Windows container on a Linux host (or vice versa)?**
No — containers share the host kernel, so a Linux container needs a Linux kernel. On Windows/macOS, Docker runs Linux containers inside a lightweight Linux VM.

**15. When would you choose a VM over a container?**
When you need strong security isolation between untrusted workloads, a different OS/kernel, or deep kernel/hardware access.

### Dockerfile

**16. `RUN` vs `CMD` vs `ENTRYPOINT`?**
`RUN` executes at build time (creates a layer). `CMD` sets the default runtime command (easily overridden). `ENTRYPOINT` sets the fixed executable; `CMD` becomes its default arguments.

**17. Shell form vs exec form of CMD?**
Shell form `CMD pnpm start` runs via `/bin/sh -c` (your process isn't PID 1, signals may not propagate). Exec form `CMD ["pnpm","start"]` runs the binary directly as PID 1 — preferred for proper signal handling and graceful shutdown.

**18. What does `EXPOSE` do?**
It's documentation — declares which port the app listens on. It does **not** publish the port; you still need `-p`/`ports:` to reach it from the host.

**19. `COPY` vs `ADD`?**
Both copy files in. `ADD` additionally can auto-extract local tarballs and fetch URLs. Prefer `COPY` for clarity; use `ADD` only when you specifically need its extra behavior.

**20. What is `WORKDIR`?**
Sets the working directory for subsequent instructions (and the running container). Creates it if missing. Cleaner than chained `cd`.

**21. What is build cache and how do you optimize it?**
Docker caches each layer; if an instruction and its inputs are unchanged, it reuses the cached layer. Optimize by ordering instructions least- to most-frequently-changing — e.g. copy lockfiles and install dependencies before copying source.

**22. What is a multi-stage build and why use it?**
Multiple `FROM` stages in one Dockerfile: build in a heavy stage, then `COPY --from=build` only the artifacts into a slim runtime stage. Result: small, secure final images without build toolchains or dev deps.

**23. What is `.dockerignore`?**
Like `.gitignore` for the build context — excludes files (e.g. `node_modules`, `.git`, `.env`) from being sent to the daemon. Speeds up builds and prevents leaking secrets into images.

**24. Why pin base image versions?**
`latest` drifts; rebuilds become non-reproducible and can break unexpectedly. Pin (e.g. `node:20.11.1-slim`) for predictable, reproducible builds.

**25. What is the build context?**
The set of files sent to the daemon when you run `docker build` — typically the directory you point at. `.dockerignore` trims it.

**26. How do you reduce image size?**
Slim/alpine base images, multi-stage builds, removing dev dependencies, combining `RUN` steps to reduce layers, cleaning package caches, and a tight `.dockerignore`.

**27. What does `ARG` vs `ENV` do?**
`ARG` is a build-time variable (available only during build, not in the running container). `ENV` sets an environment variable that persists into the running container.

**28. Explain the Dockerfile `FROM node:20 ... CMD ["pnpm","start"]`.**
`FROM node:20` picks the base; `WORKDIR /app` sets the dir; corepack enables/pins pnpm; copying lockfiles then installing leverages cache; `COPY . .` adds source; `pnpm build` compiles; `EXPOSE 5000` documents the port; `CMD ["pnpm","start"]` is the default run command.

### Compose & services

**29. What is Docker Compose?**
A tool to define and run multi-container apps with a single YAML file, starting them all (with networks and volumes) via `docker compose up`.

**30. `Dockerfile` vs `docker-compose.yml`?**
A Dockerfile builds one image; a compose file orchestrates multiple containers (networking, volumes, env, dependencies) as one app. They're used together.

**31. What is a "service" in Compose?**
The definition of a container (or set of identical replicas) plus its config — image/build, ports, env, volumes, network, dependencies.

**32. What does `depends_on` guarantee?**
Start order only — it waits for the dependency container to *start*, not to be *ready*. For readiness, add a `healthcheck` and use `condition: service_healthy`.

**33. How do you scale a service in Compose?**
`docker compose up --scale worker=4` runs four replicas of the `worker` service behind one service name.

**34. How do containers in Compose talk to each other?**
They share a user-defined network with built-in DNS, so they reach each other by **service name** (e.g. `redis://redis:6379`) — no IPs needed.

**35. `docker compose down` vs `down -v`?**
`down` removes containers and networks but keeps named volumes; `down -v` also deletes named volumes (data loss risk).

### Volumes & data

**36. Why do you need volumes?**
A container's writable layer is lost when the container is removed. Volumes persist data (databases, uploads) outside that layer.

**37. Named volume vs bind mount?**
A named volume is Docker-managed storage (best for app/DB data). A bind mount maps a specific host directory into the container (best for dev hot-reload). 

**38. What is a tmpfs mount?**
An in-memory mount that never touches disk and vanishes when the container stops — for ephemeral secrets/scratch data.

**39. Can multiple containers share a volume?**
Yes — mount the same named volume in each container to share data in a controlled way.

**40. Where does container data go without a volume?**
Into the container's thin writable layer, which is deleted when the container is removed.

### Networking

**41. What are Docker's main network drivers?**
`bridge` (default, single-host private network), `host` (shares host network stack), `none` (no networking), and `overlay` (multi-host).

**42. Default bridge vs user-defined bridge?**
The default bridge requires IP-based communication; a user-defined bridge (what Compose creates) provides automatic DNS resolution by container/service name. Prefer user-defined.

**43. Publish (`-p`) vs expose?**
`-p HOST:CONTAINER` publishes a port to the host (reachable from outside). `EXPOSE`/`expose:` only documents/opens it to other containers on the same network.

**44. How does service discovery work in Docker?**
Docker's embedded DNS resolves container/service names to their current IPs on the shared network, so apps connect by name even as containers are recreated.

**45. What is an overlay network used for?**
Connecting containers across multiple hosts (Swarm or multi-node setups) so they communicate as if on a single network.

### Operations & best practices

**46. How do you debug a running container?**
`docker logs -f <name>` for output; `docker exec -it <name> sh` to get a shell; `docker inspect` for config; `docker stats` for live resource usage.

**47. How do you limit a container's resources?**
With flags like `--memory` and `--cpus` (or Compose `deploy.resources` / `mem_limit`), which map to cgroup limits.

**48. What is a HEALTHCHECK?**
A command Docker runs periodically to report container health (healthy/unhealthy), letting orchestrators restart or gate traffic based on real readiness.

**49. How do you handle secrets in Docker?**
Never bake them into images (layers are inspectable). Inject at runtime via env vars, Docker/Compose secrets, or an external secret manager.

**50. What happens to data in a container when it's removed, and how do you keep it?**
The writable layer (and its data) is destroyed. Use a named volume (or bind mount) to persist data beyond the container's lifecycle.

---

*Reference doc — Docker fundamentals through interview prep. Pair the multi-stage Dockerfile in §7 with a Compose file like §9 for a realistic api + db + redis + worker setup.*
