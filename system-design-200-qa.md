# System Design + CS Fundamentals — 200 Interview Q&A (High-Quality v2)
---
## How to use (practical)
- Do **15 Q/day** → finish in ~2 weeks.
- For each question:  
  1) say the definition in 1 line,  
  2) give one real example from a project,  
  3) mention one tradeoff/pitfall.
- If you can’t explain in your own words → mark it and revisit.

## Sections
1) Networking & Web (**1–65**)  
2) API Design, Auth, Security, Observability (**66–100**)  
3) OS/CPU/Concurrency (**101–135**)  
4) Databases & Storage (**136–184**)  
5) Caching/Performance + Messaging/Patterns + Mini Designs (**185–200**)  
---


# 1) Networking & Web (1–65)


## 1. Internet vs World Wide Web (WWW) — what’s the difference?

**Definition:** The Internet is the global network of networks; the Web is one application on top of it that uses HTTP/HTTPS and browsers.
**Why it matters:** In design discussions, you must separate transport (Internet) from application protocol (Web) to reason about latency, failures, and security.
**Key points:**
- Internet includes many protocols (TCP/UDP, SMTP email, DNS, etc.)
- Web primarily = HTTP + URLs + HTML/CSS/JS + browsers
- A backend service can use Internet without being ‘the web’ (e.g., gRPC between services)
  
**Interview line:** “Web is an app on the Internet; Internet is the underlying packet network.”

## 2. What is an IP address?

**Definition:** A numeric address (IPv4/IPv6) used by routers to deliver packets to the correct host/network interface.
**Why it matters:** Everything in distributed systems eventually becomes ‘send packets to an IP’; debugging often starts with IP reachability.
**Key points:**
- IPv4 is 32-bit; IPv6 is 128-bit
- A host can have multiple IPs (multiple interfaces)
- Routing is based on prefixes (networks), not individual hosts
**Common pitfalls:**
- Confusing IP (network layer) with port (application endpoint on a host)

## 3. What is a port and why do we need it?

**Definition:** A 16-bit number that identifies a specific service on a host (e.g., 443 for HTTPS).
**Why it matters:** One machine runs many programs; ports let the OS deliver incoming packets to the right process.
**Key points:**
- Connection is identified by 5-tuple: src ip/port + dst ip/port + protocol
- Servers ‘listen’ on ports; clients use ephemeral source ports
**Example:** A single VM can host Nginx on :443 and Postgres on :5432 simultaneously.

## 4. What is a socket?

**Definition:** An OS object (file descriptor) representing a network endpoint used by processes to send/receive bytes.
**Why it matters:** Scaling decisions (connection limits, keep-alive, WebSockets) translate into how many sockets you hold and how you multiplex them.
**Key points:**
- TCP socket = connection-oriented stream
- UDP socket = endpoint for datagrams
- Sockets consume kernel resources: buffers, state, file descriptors
**Common pitfalls:**
- Assuming sockets are ‘free’—large numbers require tuning fd limits, memory, and load balancing.

## 5. TCP vs UDP — when to use which?

**Definition:** TCP provides reliable, ordered byte-stream delivery; UDP provides best-effort datagrams with low overhead.
**Why it matters:** Protocol choice impacts latency, throughput, head-of-line blocking, and how you handle loss.
**Key points:**
- TCP: reliability, ordering, congestion control → great for DB, HTTP/1.1, HTTP/2
- UDP: app decides reliability/ordering → used by QUIC/HTTP/3, real-time media, gaming
- UDP often needs app-level retransmits/forward error correction
**Common pitfalls:**
- Using UDP without thinking about NAT/firewalls and packet loss handling.

**Interview line:** “Use TCP for correctness; UDP when you can tolerate loss or need custom transport like QUIC.”

## 6. Explain TCP 3-way handshake.

**Definition:** TCP establishes a connection using SYN → SYN-ACK → ACK to agree on initial sequence numbers.
**Why it matters:** Connection setup costs RTT; at scale, handshake + TLS dominate latency for small requests.
**How it works (intuition):**
- Client sends SYN with initial seq number
- Server replies SYN-ACK acknowledging client seq and sending its own
- Client ACKs server seq → connection established
**Key points:**
- Handshake helps reliability and ordering via sequence numbers
- SYN backlog can overflow under load or attack (SYN flood)

**Common pitfalls:**
- Ignoring handshake + TLS cost when estimating p95 latency.

## 7. What is TCP flow control vs congestion control?

**Definition:** Flow control prevents a fast sender from overwhelming the receiver; congestion control prevents overwhelming the network.
**Why it matters:** Throughput collapses when buffers overflow; understanding these explains why ‘adding bandwidth’ doesn’t always fix latency.
**Key points:**
- Flow control uses receiver window (rwnd)
- Congestion control uses congestion window (cwnd) based on loss/delay signals
- Effective send rate ≈ min(rwnd, cwnd)
**Interview line:** “Flow control is receiver safety; congestion control is network safety.”

## 8. What is head-of-line blocking in TCP?

**Definition:** When one lost packet delays delivery of subsequent packets because TCP must deliver bytes in order.
**Why it matters:** It can hurt latency for multiplexed traffic; one loss can stall multiple logical streams (HTTP/2 on TCP).
**Key points:**
- HTTP/2 mitigates app-level HOL but TCP HOL still exists
- HTTP/3 over QUIC reduces HOL by using independent streams at transport layer
**Example:** On mobile networks, a single packet loss can spike p99 even if server is fast.

## 9. What does DNS do?

**Definition:** DNS maps human-readable names (example.com) to IP addresses and other records (MX, TXT, etc.).
**Why it matters:** DNS is a hidden dependency for every request; TTL and caching affect failover and rollout speed.
**Key points:**
- Resolvers cache responses for TTL
- DNS can return multiple A/AAAA records for load distribution
- Anycast DNS helps route to nearby resolvers/edges.

**Common pitfalls:**
- Assuming DNS changes apply instantly—clients may cache old answers for TTL.

## 10. Explain DNS caching and TTL like an engineer.

**Definition:** TTL is how long a DNS response may be cached by resolvers/clients before re-querying.
**Why it matters:** It controls the tradeoff between lookup latency and how quickly you can change IPs (failover).
**Key points:**
- Caches exist at: browser, OS, recursive resolver, sometimes ISP middleboxes
- Lower TTL → faster failover but more DNS query load
- Higher TTL → fewer lookups but slower traffic shift

**Interview line:** “TTL is operational leverage: low TTL for agility, high TTL for stability and cost.”

## 11. What is Anycast and where is it used?

**Definition:** A routing technique where multiple servers share the same IP, and BGP routes clients to the ‘nearest’ one.

**Why it matters:** Used by CDNs/DNS to reduce latency and absorb DDoS by spreading load globally.

**Key points:**
- Same IP announced from many locations
- Routing is based on network topology, not exact geography
- Failover can be fast if one site stops advertising

**Example:** Many DNS providers and CDN edges are anycasted.

## 12. What is NAT and why does it exist?

**Definition:** Network Address Translation allows many private hosts to share one public IPv4 address by rewriting IP/port.

**Why it matters:** Impacts inbound connections, WebRTC/WebSocket behavior, and debugging connectivity.

**Key points:**
- Private IP ranges: 10/8, 172.16/12, 192.168/16
- Outbound is easy; inbound needs port forwarding or a relay
- Also provides a basic barrier but is not a security boundary

**Common pitfalls:**
- Assuming clients can accept inbound connections directly from the Internet.

## 13. What is TLS and what problems does it solve?

**Definition:** TLS provides encryption (confidentiality), integrity (tamper detection), and authentication (server identity via certificates).

**Why it matters:** Without TLS, attackers on the network path can read or modify traffic; modern browsers require HTTPS for many features.

**Key points:**
- Handshake authenticates server certificate chain (PKI)
- Uses asymmetric crypto to agree on symmetric session keys
- Session resumption reduces handshake cost

**Common pitfalls:**
- Thinking TLS is only ‘encryption’—authentication is equally critical.

## 14. Explain certificate chain and PKI briefly.

**Definition:** A server certificate is trusted because it’s signed by an intermediate CA, which is signed by a root CA trusted by the OS/browser.

**Why it matters:** If you misunderstand PKI, you can’t reason about MITM risks, mTLS, or cert rotation outages.

**Key points:**

- Root CA public keys are pre-installed in trust stores
- Intermediates allow rotation without changing root
- Hostname validation (CN/SAN) must match the domain

**Common pitfalls:**
- Expired certs, missing intermediates, wrong hostname → outages.

## 15. HTTP is stateless — what does that mean in practice?
**Definition:** Each HTTP request is independent; the server doesn’t have to remember prior requests to respond correctly.

**Why it matters:** Stateless servers scale horizontally more easily; state must live in client tokens, DB, or shared stores.

**Key points:**
- State can be added via cookies/sessions/JWT
- Stateless ≠ no DB; it means no per-connection memory state required
- WebSockets are stateful by nature (long-lived connection)

**Interview line:** “Stateless makes load balancing trivial; state lives in shared stores.”

## 16. HTTP methods: GET/POST/PUT/PATCH/DELETE — semantics interviewers expect.

**Definition:** Standard verbs with expectations about safety and idempotency.
**Why it matters:** Correct semantics enable caching, retries, and predictable APIs.
**Key points:**
- **GET**: safe + idempotent (no state change)
- **PUT**: idempotent replacement (same request repeated → same state)
- **PATCH**: partial update (often not strictly idempotent unless designed)
- **POST**: create/command (not idempotent unless you add idempotency key)
- **DELETE**: idempotent delete (deleting twice still ‘deleted’)
**Common pitfalls:**
- Using POST for everything; breaking caching and retries.

## 17. What is idempotency and why is it huge in distributed systems?

**Definition:** An operation is idempotent if repeating it produces the same final state.

**Why it matters:** Networks fail → clients retry. Without idempotency you get duplicates: double payments, duplicate orders, repeated emails.
**Key points:**
- Idempotent reads: GET
- Idempotent writes: PUT/DELETE by definition; POST needs idempotency key
- Idempotency key is stored server-side with request fingerprint and result
**Example:** Payment API stores `idempotency_key → charge_id` for 24h to prevent double charge.
**Interview line:** “Retries are mandatory; idempotency is how retries don’t break correctness.”

## 18. What are the most important HTTP status codes and what do they signal?

**Definition:** Standard codes communicate success, client errors, or server failures.

**Why it matters:** Clients implement retries/backoff based on codes; SRE dashboards group errors by class.

**Key points:**
- 200/201/204 success, 304 cache revalidation
- 400 validation, 401 auth required, 403 forbidden, 404 not found
- 409 conflict (version mismatch), 429 rate-limited
- 500 server bug, 502/504 upstream issues, 503 overloaded/maintenance

**Common pitfalls:**
- Returning 200 with error message; breaking client logic and observability.

## 19. What is CORS and what problem does it actually solve?

**Definition:** A browser-enforced rule that controls whether a page can read responses from a different origin.
**Why it matters:** Prevents malicious sites from using your browser as a credentialed cross-origin data reader.
**Key points:**
- Not a server security feature; it’s a browser protection
- Controlled via `Access-Control-Allow-Origin`, methods, headers
- Preflight OPTIONS requests happen for non-simple requests
**Common pitfalls:**
- Confusing CORS with CSRF; they’re different threats.

## 20. Cookies vs Sessions vs JWT — compare like an interviewer.

**Definition:** Cookies are browser storage sent with requests; sessions are server-side state keyed by cookie; JWT is a signed token carrying claims.

**Why it matters:** Your choice impacts scaling, revocation, and security posture.

**Key points:**
- Cookie+session: easy revocation, but needs shared session store (Redis) for multi-instance
- JWT: stateless and fast, but revocation is hard (use short TTL + refresh tokens)
- Cookies can be HttpOnly/Secure/SameSite; JWT in localStorage increases XSS risk

**Interview line:** “Sessions are easier to revoke; JWT scales well but requires careful expiry/refresh design.”

## 21. Explain HttpOnly, Secure, SameSite on cookies.

**Definition:** Flags controlling cookie access and cross-site behavior.

**Why it matters:** Directly reduces XSS/CSRF risk.
**Key points:**
- HttpOnly: JS can’t read cookie (protects against token theft via XSS)
- Secure: only sent over HTTPS
- SameSite: controls cross-site sending (Lax/Strict/None)

**Common pitfalls:**
- Setting SameSite=None without Secure; browsers may reject it.

## 22. HTTP caching: Cache-Control, ETag, Last-Modified — what’s the difference?

**Definition:** Headers that control reuse of responses and revalidation.
**Why it matters:** Caching is the cheapest performance win; but incorrect caching causes data leaks and stale UX.
**Key points:**
- Cache-Control: freshness policy (max-age, public/private, no-store)
- ETag: content version; enables conditional requests with 304
- Last-Modified: timestamp-based revalidation (less precise than ETag)

**Common pitfalls:**
- Caching personalized responses at CDN without `private` or proper cache key scoping.

## 23. What is gzip/brotli compression and what is the tradeoff?

**Definition:** Server compresses payload to reduce bytes over the network.
**Why it matters:** For JSON/HTML it can cut size significantly and improve latency on slow links.

**Key points:**
- Brotli often better compression than gzip (especially for text)
- CPU cost increases; can hurt if you compress already-compressed binaries
- Usually enable for text types only

**Interview line:** “Compression trades CPU for network—great for text, avoid double-compress.”

## 24. HTTP/1.1 vs HTTP/2 vs HTTP/3 — practical differences.

**Definition:** HTTP/2 multiplexes streams over one TCP connection; HTTP/3 uses QUIC over UDP with improved loss handling.
**Why it matters:** Impacts tail latency and mobile performance; relevant for CDNs and high-traffic APIs.

**Key points:**
- HTTP/1.1: limited multiplexing, more connections
- HTTP/2: multiplexing + header compression; but TCP HOL can still hurt
- HTTP/3: QUIC streams reduce HOL at transport; faster connection migration

**Common pitfalls:**
- Assuming HTTP/2 always faster; server CPU and mis-tuned settings can negate benefits.

## 25. What is a reverse proxy and why is it everywhere?

**Definition:** A server that accepts client requests and forwards them to backend services.
**Why it matters:** Central place for TLS termination, routing, auth, rate limiting, caching, and observability.
**Key points:**
- Examples: Nginx, Envoy, API gateways, cloud load balancers
- Can do path-based routing, canary releases, header rewriting

**Common pitfalls:**
- Making reverse proxy a single bottleneck; scale it and keep configs safe.

## 26. Load balancer L4 vs L7 — explain with examples.

**Definition:** L4 balances based on TCP/UDP (IP+port). L7 understands HTTP and can route by path/host/headers.
**Why it matters:** L7 enables smarter routing but adds CPU and complexity.
**Key points:**
- L4: fast, low overhead; good for raw TCP services (DB proxies) and WebSockets
- L7: supports auth, rate limiting, path routing, A/B tests

**Interview line:** “Choose L4 for simplicity/perf; L7 when you need HTTP-aware routing/policy.”

## 27. Load balancing algorithms — round robin vs least connections vs hashing.

**Definition:** Strategies to distribute traffic across instances.
**Why it matters:** Different workloads need different algorithms to avoid hotspots.
**Key points:**
- Round robin: simple, assumes equal capacity
- Least connections: good when request time varies
- Hash (IP/userId): sticky routing; good for caches but can create uneven load
- Weighted variants handle different machine sizes
**Common pitfalls:**
- Sticky routing with stateful instances can block scaling and failover.

## 28. What are sticky sessions and why are they usually a smell?

**Definition:** Routing a user to the same backend instance repeatedly.
**Why it matters:** It hides state in memory and makes autoscaling/failover painful.
**Key points:**
- Needed only if session state is in-process
- Better: store session in Redis or use stateless JWT
- Sticky can also be used for cache locality (acceptable)
**Interview line:** “Sticky sessions are a workaround; shared session store is the scalable fix.”

## 29. What is CDN and how does it reduce latency?

**Definition:** A global edge network that caches content close to users.
**Why it matters:** Most latency is distance; CDN reduces RTT and offloads origin bandwidth/CPU.
**Key points:**
- Edge cache hit avoids origin trip
- CDNs also terminate TLS near users
- Can cache static assets and cacheable GET APIs

**Common pitfalls:**
- Wrong cache keys can serve incorrect or private data.

## 30. CDN cache key — what should it include?

**Definition:** The set of request attributes that determine the correct cached response.
**Why it matters:** Bad cache key = data leaks or low hit rate.

**Key points:**
- Usually includes: host + path + query params (selected), maybe headers like Accept-Encoding
- Avoid including volatile headers (User-Agent) unless necessary
- For personalized data, either don’t cache or include user scope (rare).

**Interview line:** “Cache key design is correctness first, then hit rate.”

## 31. Explain WebSockets: why, how, and when not to use them.

**Definition:** A persistent full-duplex channel created by HTTP Upgrade (wss:// uses TLS).
**Why it matters:** Great for real-time; expensive for servers at scale if you keep millions of open connections.
**Key points:**
- One connection supports many messages; avoids polling overhead
- Scaling needs many WS servers + shared pub/sub for fanout
- Use for chat, live dashboards, multiplayer; avoid when updates are rare (use polling/SSE)

**Common pitfalls:**
- No backpressure → memory blowups; also idle connections require heartbeat/timeout handling.

## 32. SSE vs WebSocket vs long polling — which should you choose?

**Definition:** SSE is server→client stream over HTTP; WS is two-way; long polling is repeated HTTP requests held open.

**Why it matters:** Choosing simplest viable option reduces cost and bugs.

**Key points:**
- SSE: simpler than WS, works well for notifications/feeds
- WS: needed for client→server realtime (chat typing, collaborative editing)
- Long polling: fallback, higher overhead but easiest to deploy.

**Interview line:** “Prefer SSE/polling if one-way or low frequency; WebSocket when you truly need bi-directional realtime.”

## 33. What is a WAF and what can it protect against?

**Definition:** A Web Application Firewall that filters HTTP traffic using rules/signatures/behavior models.
**Why it matters:** Stops common web attacks at the edge and reduces load on your app.
**Key points:**
- Helps mitigate SQLi/XSS patterns, bot traffic, some layer-7 DDoS
- Not a substitute for secure coding
- Requires tuning to avoid false positives
**Common pitfalls:**
- Overblocking legitimate traffic or relying on WAF as the only security control.

## 34. DDoS at a high level: what types and defenses?

**Definition:** Distributed Denial of Service overwhelms bandwidth, protocol state, or application resources.
**Why it matters:** Designing for public APIs means planning for abuse, not just normal users.
**Key points:**
- Volumetric (Gbps): mitigate with CDN/Anycast scrubbing
- Protocol (SYN flood): SYN cookies, L4 protection
- App-layer: rate limiting, caching, WAF, bot management

**Interview line:** “Use multi-layer defense: edge absorption + rate limiting + efficient app.”

## 35. What is MTU and why do you care?

**Definition:** Maximum Transmission Unit: largest packet size on a link without fragmentation (often ~1500 bytes Ethernet).
**Why it matters:** Fragmentation/loss can hurt performance; important for VPNs, QUIC, and tuning payload sizes.
**Key points:**
- PMTUD (Path MTU Discovery) tries to find safe size
- Large UDP packets can be dropped; keep UDP payloads conservative
**Common pitfalls:**
- Assuming you can send arbitrarily large UDP datagrams reliably.

## 36. What is traceroute and how does it help debugging?

**Definition:** A tool that reveals the network hops between client and server using TTL-expired messages.
**Why it matters:** Helps locate where latency/loss occurs (ISP, region, datacenter edge).
**Key points:**
- High latency at one hop can indicate congestion
- Some hops deprioritize ICMP so interpret carefully
**Interview line:** “Use traceroute/ping to separate network issues from server issues.”

## 37. What is an API gateway and how is it different from a load balancer?

**Definition:** An API gateway is an L7 proxy specialized for APIs: auth, rate limiting, routing, transformations.
**Why it matters:** Centralizes cross-cutting concerns so services stay simple.
**Key points:**
- LB focuses on distributing traffic; gateway adds policy & developer experience
- Gateways can enforce quotas, auth, versioning, logging
**Common pitfalls:**
- Gateway becomes a bottleneck or single point of failure if not scaled/HA.

## 38. What is service discovery and why does it exist?

**Definition:** How services find each other’s network locations dynamically (instead of hardcoded IPs).
**Why it matters:** In autoscaled systems, instances come/go; discovery keeps routing correct.
**Key points:**
- DNS-based (Consul), registry-based (Eureka), sidecar/mesh-based
- Clients or proxies query registry for healthy endpoints
**Common pitfalls:**
- Stale registries or slow propagation can cause traffic to dead instances.

## 39. What is mTLS and when do you use it?

**Definition:** Mutual TLS: both client and server authenticate via certificates.
**Why it matters:** Strong service-to-service identity; reduces risk of lateral movement and spoofing inside your network.
**Key points:**
- Common in service meshes (Istio/Linkerd) and zero-trust architectures
- Requires cert issuance/rotation automation
**Common pitfalls:**
- Operational complexity: cert rotation failures can cause outages.

## 40. What is an internal vs external load balancer?

**Definition:** External LB faces the Internet; internal LB routes traffic within private network/VPC.
**Why it matters:** Keeps internal services private and reduces attack surface.
**Key points:**
- Public APIs behind external LB + WAF/CDN
- Microservices behind internal LB/service mesh
**Interview line:** “Expose minimal surface externally; everything else stays private.”

## 41. What is connection keep-alive and why does it improve performance?

**Definition:** Reusing existing TCP/TLS connections for multiple requests.
**Why it matters:** Avoids repeated handshakes (RTT + CPU), improves p95 and reduces server load.
**Key points:**
- HTTP keep-alive is default in modern stacks
- Connection pools reuse upstream connections (DB, microservices)
**Common pitfalls:**
- Too many idle connections waste memory; tune timeouts and pool sizes.

## 42. What is a health check and why have liveness vs readiness?

**Definition:** Health check is an endpoint used by orchestrators/LBs to decide if instance should receive traffic.
**Why it matters:** Bad health checks cause cascading failures or traffic to broken instances.
**Key points:**
- Liveness: process alive; restart if failed
- Readiness: able to serve (DB reachable, warmed caches); remove from rotation if failed
**Common pitfalls:**
- Doing heavy DB queries in health checks → self-inflicted load.

## 43. What is ‘tail latency’ and why does it dominate user experience?

**Definition:** High-percentile latency (p95/p99) experienced by the slowest requests.
**Why it matters:** Users remember slow requests; microservice fanout amplifies tails (max of many calls).
**Key points:**
- Measure p50/p95/p99
- Reduce long tails via caching, timeouts, bulkheads, queue isolation, and removing slow dependencies from request path
**Interview line:** “In distributed systems, p99 is where reliability and performance meet.”

## 44. What is RTT and why is it a first-class design constraint?

**Definition:** Round-trip time is the time for a message to go client→server→client.
**Why it matters:** Many protocols pay RTT costs (TCP, TLS, request/response). RTT dominates small payload latency.
**Key points:**
- Minimize number of round trips (batching, keep-alive, co-locating services)
- Use CDN/edge to reduce geographic RTT
**Interview line:** “Latency is mostly RTT count × RTT size.”

## 45. What is a proxy (forward vs reverse)?

**Definition:** Forward proxy acts for clients; reverse proxy acts for servers.
**Why it matters:** Helps reason about corporate proxies, caching, and edge routing.
**Key points:**
- Forward proxies: outbound control, anonymity, caching
- Reverse proxies: TLS termination, routing, security policies
**Interview line:** “Reverse proxy is server-side gateway; forward proxy is client-side gateway.”

## 46. What is HTTP request smuggling (high-level) and why proxies matter?

**Definition:** An attack where inconsistent parsing between proxies/backends lets attackers sneak extra requests.
**Why it matters:** Edge proxies and backends must agree on parsing; misconfig can cause severe security issues.
**Key points:**
- Typically involves `Content-Length` vs `Transfer-Encoding` parsing mismatches
- Mitigation: keep stacks updated, enforce strict parsing, normalize headers
**Common pitfalls:**
- Assuming L7 components are ‘transparent’—they parse and can be attacked.

## 47. What is ‘origin’ vs ‘edge’ in CDN terminology?

**Definition:** Origin is your source server (where content is generated). Edge is CDN PoP that caches/serves content near users.
**Why it matters:** Explains where to debug: cache hit issues happen at edge; correctness and auth often happen at origin.
**Key points:**
- Edge can cache based on rules (TTL, cache-control), perform TLS termination, WAF, and sometimes compute (edge functions)
- Origin must set correct cache headers and vary keys
**Common pitfalls:**
- Assuming edge always has the latest content; invalidation and TTL control freshness.

## 48. What is ‘origin shield’ / ‘tiered caching’?

**Definition:** A CDN feature where edges fetch from an intermediate ‘shield’ cache before hitting origin.
**Why it matters:** Reduces origin load during cache misses and helps handle thundering herds globally.
**Key points:**
- Improves hit ratio at the origin layer
- Useful for large global traffic patterns
**Interview line:** “Tiered caching reduces origin load by adding a second cache layer.”

## 49. What is HSTS?

**Definition:** HTTP Strict Transport Security tells browsers to use HTTPS only for a domain for a period of time.
**Why it matters:** Prevents SSL-stripping attacks and accidental HTTP usage.
**Key points:**
- Set via `Strict-Transport-Security` header
- Can include subdomains and preload list (careful!)
**Common pitfalls:**
- Enabling HSTS without valid HTTPS everywhere can lock users out.

## 50. What is ALPN and why relevant for HTTP/2?

**Definition:** Application-Layer Protocol Negotiation lets TLS negotiate which application protocol to use (h2 vs http/1.1).
**Why it matters:** Explains how clients/servers upgrade to HTTP/2 without extra round trips.
**Key points:**
- H2 often enabled via ALPN during TLS handshake
- Misconfigured ALPN can force fallback to HTTP/1.1

## 51. What is SNI?

**Definition:** Server Name Indication is a TLS extension that indicates the hostname so servers can choose the correct certificate.
**Why it matters:** Allows multiple HTTPS domains to share one IP (virtual hosting).
**Key points:**
- Modern CDNs/LBs rely on SNI
- Some old clients without SNI have issues

## 52. What is a connection backlog (SYN backlog / accept queue)?

**Definition:** Kernel queues for pending TCP handshakes and accepted-but-not-yet-handled connections.
**Why it matters:** Under load or attack, backlog overflow causes connection drops/timeouts.
**Key points:**
- Tune backlog sizes and enable SYN cookies
- Ensure app accepts connections fast enough
**Common pitfalls:**
- Thinking ‘CPU is low’ means you can’t drop connections—kernel queues can overflow first.

## 53. What is HTTP keep-alive timeout vs idle timeout on load balancers?

**Definition:** Limits how long an idle connection is kept open before being closed by server/LB.
**Why it matters:** Mismatched timeouts cause random disconnects and retry storms.
**Key points:**
- Client, proxy, LB, and server all have independent idle timeouts
- Set them consistently (LB slightly higher than server is common)
**Common pitfalls:**
- WebSockets need longer idle timeouts or heartbeats to keep connection alive.

## 54. What is ‘connection draining’ on load balancers?

**Definition:** Stopping new requests to an instance while allowing in-flight requests to complete before termination.
**Why it matters:** Prevents user-facing errors during deploys/autoscaling.
**Key points:**
- Works with readiness + graceful shutdown
- Critical for long-lived requests and WebSockets

## 55. What is ‘zero-downtime deploy’ for HTTP services at a high level?

**Definition:** Deploying new versions without dropping requests using health checks, draining, and rolling strategies.
**Why it matters:** Production reliability depends more on deploy safety than just code correctness.
**Key points:**
- Start new instances → mark ready → shift traffic → drain old → terminate
- Use canary/blue-green when risk is high
**Common pitfalls:**
- Killing processes without draining causes spikes in 5xx and client retries.

## 56. What is WebRTC (very high level) and why is it harder than WebSockets?

**Definition:** A real-time peer-to-peer media/data protocol that often needs NAT traversal (STUN/TURN).
**Why it matters:** If asked to design video calling, WebSockets alone won’t solve media transport and NAT traversal.
**Key points:**
- Uses ICE (STUN/TURN) to find paths through NAT
- Often uses UDP for media; relays via TURN when direct fails
**Interview line:** “WebSockets are great for signaling; WebRTC handles the media plane.”

## 57. What is HTTP request/response size limit and why does it matter?

**Definition:** Limits at proxies/servers (headers/body) beyond which requests are rejected or slowed.
**Why it matters:** Large headers (huge cookies/JWTs) hurt latency and can break at proxies.
**Key points:**
- Watch header size limits on CDNs/LBs
- Prefer short tokens; avoid storing lots of data in cookies

## 58. What is ‘hop-by-hop’ vs ‘end-to-end’ header?

**Definition:** Hop-by-hop headers apply to a single proxy connection; end-to-end headers are forwarded to the origin.
**Why it matters:** Avoids bugs in caching/proxying and explains why some headers disappear.
**Key points:**
- `Connection`, `Keep-Alive`, `Transfer-Encoding` are hop-by-hop
- Proxies may add `X-Forwarded-For` / `Forwarded` (be careful trusting them)

## 59. What is `X-Forwarded-For` and how can it be abused?

**Definition:** A header added by proxies to convey the original client IP.
**Why it matters:** Rate limiting and auditing rely on client IP—spoofing can bypass controls if you trust it blindly.
**Key points:**
- Trust XFF only from your known proxy/LB
- Prefer standardized `Forwarded` header if available
**Common pitfalls:**
- Using XFF directly without proxy trust boundary.

## 60. What is ‘TLS termination’ and what is the tradeoff?

**Definition:** Terminating TLS at the LB/proxy and forwarding plaintext HTTP internally (or re-encrypting).
**Why it matters:** Simplifies cert management and improves performance; but impacts internal trust model.
**Key points:**
- Common: TLS at edge + optional mTLS internally
- End-to-end TLS is used in zero-trust environments
**Interview line:** “TLS termination is fine if your internal network is trusted; otherwise use mTLS.”

## 61. What is HTTP/2 connection coalescing (concept)?

**Definition:** Using one HTTP/2 connection for multiple origins when certificates and DNS allow it (mostly for same CDN).
**Why it matters:** Reduces connection count and handshake overhead for many subdomains.
**Key points:**
- Works when cert covers multiple hosts and IP is shared
- Often leveraged by CDNs

## 62. What is QUIC connection migration and why helpful on mobile?

**Definition:** QUIC can keep a connection alive when client IP changes (Wi‑Fi ↔ LTE) without reconnecting.
**Why it matters:** Improves reliability and reduces reconnect latency for mobile clients.
**Key points:**
- Uses connection IDs instead of strict 4-tuple identity
- Still requires security checks

## 63. What is ‘N+1 RTTs’ problem in microservices?

**Definition:** A request triggers many sequential network calls; each adds at least one RTT, inflating p95/p99.
**Why it matters:** Explains why seemingly ‘small’ service decomposition can destroy latency.
**Key points:**
- Prefer batching, parallelism, and reducing sync hops
- Move non-critical work async via queues/events
**Interview line:** “Latency is additive across serial hops; minimize synchronous dependencies.”

## 64. What is ‘north-south’ vs ‘east-west’ traffic?

**Definition:** North-south is client↔service traffic; east-west is service↔service traffic inside your network.
**Why it matters:** Designing security, observability, and load balancing differs for public vs internal traffic.
**Key points:**
- North-south needs WAF, DDoS protection, API gateway
- East-west benefits from service mesh, mTLS, retries/circuit breakers carefully tuned

## 65. What is ‘TLS handshake latency’ and how do you reduce it?

**Definition:** Time spent establishing a secure session before app data flows.
**Why it matters:** For small responses, handshake can dominate total latency.
**Key points:**
- Use keep-alive and connection pooling
- Enable TLS session resumption / 0-RTT where safe
- Terminate TLS at nearby edge/CDN
**Common pitfalls:**
- 0-RTT can allow replay; only safe for idempotent requests.

# 2) API Design, Auth, Security, Observability (66–100)


## 66. REST — what does it mean beyond ‘JSON over HTTP’?

**Definition:** REST is an architectural style: resources identified by URLs, manipulated with standard HTTP semantics, stateless interactions, and cacheable responses.
**Why it matters:** Interviewers want you to design predictable APIs that leverage HTTP (caching, idempotency) instead of reinventing RPC badly.
**Key points:**
- Resource modeling: nouns not verbs (`/users/123`)
- Use correct methods/status codes
- Hypermedia is part of pure REST but most practical APIs use a subset
**Interview line:** “REST is about using HTTP semantics well: resources, statelessness, caching, and standard verbs.”

## 67. How do you model resources and endpoints cleanly?

**Definition:** Design URLs around entities and collections with consistent patterns.
**Why it matters:** Clean modeling reduces client complexity and makes versioning and permissions easier.
**Key points:**
- Collections: `GET /orders`, create: `POST /orders`
- Entity: `GET/PUT/PATCH/DELETE /orders/{id}`
- Sub-resources: `GET /orders/{id}/items`
- Avoid verbs in paths unless truly an action (`POST /orders/{id}:cancel` style in some APIs)
**Common pitfalls:**
- Over-nesting (deep paths) and leaking internal DB tables directly.

## 68. Pagination — offset vs cursor (keyset). When and why?

**Definition:** Pagination is returning results in pages. Offset uses `LIMIT/OFFSET`; cursor uses a stable sort key (e.g., id/time) to fetch next page.
**Why it matters:** Offset becomes slow and inconsistent with inserts/deletes; cursor is faster and stable at scale.
**Key points:**
- Offset: simple, good for small datasets/admin UI
- Cursor: scalable, consistent ordering, supports infinite scroll
- Always define deterministic sort order
**Interview line:** “At scale, use cursor pagination; offset is fine for small datasets.”

## 69. Design a good error response format for APIs.

**Definition:** A consistent JSON error schema that clients can parse reliably.
**Why it matters:** Makes debugging and client logic simpler; improves observability.
**Key points:**
- Include: machine code, human message, request id, field errors
- Use correct HTTP status code
- Do not leak internal stack traces to clients
**Example:** {"error":{"code":"VALIDATION_ERROR","message":"Invalid input","fields":{"email":"invalid"}},"request_id":"..."}

## 70. API versioning strategies — what are the tradeoffs?

**Definition:** Methods to evolve APIs without breaking clients.
**Why it matters:** Breaking changes are costly; versioning is an operational necessity.
**Key points:**
- URL versioning: `/v1/...` (simple, common)
- Header versioning: `Accept: application/vnd...` (cleaner but harder)
- Prefer backward-compatible changes: add fields, never change meaning
**Common pitfalls:**
- Versioning every tiny change; better to keep stable contracts and deprecate carefully.

## 71. What is authentication vs authorization?

**Definition:** Authentication proves identity; authorization checks permissions.
**Why it matters:** Mixing them causes security holes (e.g., ‘logged in’ ≠ ‘allowed’).
**Key points:**
- AuthN: passwords, OAuth login, API keys
- AuthZ: RBAC/ABAC, ownership checks, scopes
**Interview line:** “AuthN is who you are; AuthZ is what you can do.”

## 72. OAuth 2.0 — explain it at a high level without getting lost.

**Definition:** A delegation protocol that lets an app access a resource server on behalf of a user without sharing the user’s password.
**Why it matters:** Used for ‘Login with Google’ and token-based access; common interview topic for APIs.
**Key points:**
- Actors: Resource Owner (user), Client (app), Authorization Server, Resource Server
- Client gets access token (and optionally refresh token) after user consents
- Scopes limit what token can do
**Common pitfalls:**
- Confusing OAuth (delegation) with authentication; OIDC adds identity layer.

## 73. OpenID Connect (OIDC) — how is it different from OAuth?

**Definition:** OIDC is an identity layer on top of OAuth 2.0; it provides an ID Token that represents the user identity.
**Why it matters:** If you implement ‘login’, OIDC is the standard, not raw OAuth.
**Key points:**
- ID Token is usually a JWT with user claims
- Access token is for APIs; ID token is for client identity
**Interview line:** “OAuth is authorization; OIDC is authentication built on OAuth.”

## 74. JWT — what are the real pros/cons for backend systems?

**Definition:** A signed token containing claims, verified by services without DB lookup.
**Why it matters:** Common in microservices; impacts revocation, token size, and XSS risk.
**Key points:**
- Pros: stateless verification, easy for distributed services
- Cons: revocation hard, tokens can get large, leakage risk
- Use short TTL + refresh tokens; rotate signing keys (kid)
**Common pitfalls:**
- Storing JWT in localStorage (XSS); prefer HttpOnly cookies when possible.

## 75. Refresh tokens — why needed and how to secure them?

**Definition:** Long-lived tokens used to obtain new short-lived access tokens.
**Why it matters:** Short-lived access tokens reduce blast radius; refresh tokens preserve UX.
**Key points:**
- Store refresh tokens securely (HttpOnly cookie or secure device storage)
- Rotate refresh tokens on use; revoke on suspicion
- Bind to device/client and track token family
**Common pitfalls:**
- Long-lived access tokens in clients; impossible to revoke quickly.

## 76. API keys vs OAuth tokens — when to use each?

**Definition:** API keys identify a client/app; OAuth tokens represent delegated user access or service identity.
**Why it matters:** Choosing wrong mechanism leads to weak security and poor auditing.
**Key points:**
- API keys: server-to-server, simple integrations, rate limiting per key
- OAuth: user consent/scopes, third-party apps, fine-grained access
- Both need rotation and revocation strategy
**Interview line:** “API keys identify the caller; OAuth tokens encode permissions and identity context.”

## 77. RBAC vs ABAC — compare with examples.

**Definition:** RBAC grants permissions by role; ABAC uses attributes (user, resource, environment) to decide.
**Why it matters:** Authorization models must scale as product complexity grows.
**Key points:**
- RBAC: simple (admin/editor/viewer), easy to reason about
- ABAC: flexible (tenantId, ownership, region, time, device trust)
- Often hybrid: roles + attribute checks
**Example:** ABAC: allow edit if user.tenant==doc.tenant AND (user.id==doc.owner OR user.role==admin).

## 78. Multi-tenant system: key design decisions you must mention.

**Definition:** A system serving multiple customers (tenants) with isolation.
**Why it matters:** Most SaaS interview designs are multi-tenant; isolation mistakes are catastrophic.
**Key points:**
- Tenant isolation model: shared DB with tenant_id, schema-per-tenant, DB-per-tenant
- AuthZ must include tenant checks everywhere
- Noisy neighbor controls: rate limit/quotas per tenant
**Common pitfalls:**
- Missing tenant filter in queries → data leak across customers.

## 79. Input validation — what belongs in API layer vs domain layer?

**Definition:** API layer validates shape/type; domain layer validates business rules/invariants.
**Why it matters:** Prevents security issues and preserves correctness across all entry points.
**Key points:**
- API: required fields, types, limits, formats
- Domain: uniqueness, state transitions, cross-field rules
- Return structured field errors
**Common pitfalls:**
- Relying only on frontend validation.

## 80. SQL injection — what is it and how do you prevent it?

**Definition:** Attacker injects SQL via untrusted input to change query meaning.
**Why it matters:** Still one of the most common severe vulnerabilities.
**Key points:**
- Use parameterized queries/prepared statements
- Least-privilege DB users
- Validate/escape identifiers if dynamic (table/column names)
**Interview line:** “Never build SQL with string concatenation; always bind parameters.”

## 81. XSS — what is it and why does backend care?

**Definition:** Cross-site scripting: attacker causes victim browser to execute malicious JS.
**Why it matters:** Token/session theft and account takeover; backend sets headers and output that affect XSS risk.
**Key points:**
- Escape output, sanitize rich text
- Use Content Security Policy (CSP)
- Use HttpOnly cookies for session tokens
**Common pitfalls:**
- Storing untrusted HTML and rendering it without sanitization.

## 82. CSRF — what is it and how do you mitigate it?

**Definition:** Cross-site request forgery: attacker tricks a logged-in browser into sending unwanted requests.
**Why it matters:** Cookies are automatically sent; without protection, attackers can perform actions as the user.
**Key points:**
- SameSite cookies (Lax/Strict) reduce risk
- CSRF tokens for state-changing requests
- Require custom headers (not sent by forms)
**Interview line:** “CSRF exists because cookies are ambient authority; mitigate with SameSite + tokens.”

## 83. CORS vs CSRF — clarify the confusion.

**Definition:** CORS controls which origins can read responses; CSRF is about forged requests being accepted.
**Why it matters:** Common interview trap; mixing them leads to wrong fixes.
**Key points:**
- CORS is enforced by browsers when reading cross-origin responses
- CSRF succeeds even if attacker cannot read response
- Mitigate CSRF with SameSite/CSRF token, not CORS alone

## 84. Rate limiting algorithms: fixed window vs sliding window vs token bucket.

**Definition:** Techniques to cap request rate.
**Why it matters:** Protects against abuse and prevents overload cascades.
**Key points:**
- Fixed window: simple but bursty at boundaries
- Sliding window: smoother but more state
- Token bucket: allows bursts up to bucket size; widely used
- Leaky bucket: smooth constant outflow (good for shaping)
**Interview line:** “Token bucket is a solid default: supports bursts while enforcing average rate.”

## 85. Where should rate limiting live: client, gateway, service, or DB?

**Definition:** Rate limiting can be applied at multiple layers; best place depends on goal.
**Why it matters:** Stopping abuse early is cheaper and safer.
**Key points:**
- Edge/gateway for DDoS and broad limits
- Service-level limits for per-user/tenant fairness
- Avoid doing limits in DB; too expensive and late
**Common pitfalls:**
- Only limiting at the app after expensive auth/DB work is already done.

## 86. Webhooks: how to design them reliably?

**Definition:** Server-to-server callbacks triggered by events (e.g., payment succeeded).
**Why it matters:** Webhooks fail in the real world; reliability depends on retries, signing, and idempotency.
**Key points:**
- Sign payloads (HMAC) so receiver can verify authenticity
- Retry with backoff; store delivery attempts
- Use idempotency (event id) to prevent duplicates on retries
**Common pitfalls:**
- Assuming receiver is always up; not handling retries and ordering.

## 87. How do you secure webhooks against spoofing?

**Definition:** Ensure the receiver can verify the sender and integrity.
**Why it matters:** Fake webhooks can create fake orders/payments and cause fraud.
**Key points:**
- HMAC signature with shared secret; include timestamp to prevent replay
- Use TLS and verify certificate
- Optional: allowlist sender IPs (but CDNs can complicate)
**Interview line:** “Treat webhook endpoints like public APIs: verify signature, handle retries, make processing idempotent.”

## 88. What is ‘least privilege’ and how it shows up in system design?

**Definition:** Grant only the permissions required for a component to do its job.
**Why it matters:** Limits blast radius if credentials are compromised.
**Key points:**
- Separate DB users by service
- Scopes/roles for tokens
- Restrict network access via security groups
**Interview line:** “Assume breach; least privilege limits damage.”

## 89. Secrets management — what’s wrong with putting secrets in env vars or code?

**Definition:** Secrets are sensitive values like DB passwords, API keys, signing keys.
**Why it matters:** Leaks happen via logs, screenshots, git history, crash dumps, and misconfig.
**Key points:**
- Use secret manager (Vault/Cloud secrets) with rotation and access policies
- Avoid long-lived static secrets; rotate regularly
- Never log secrets; scrub them
**Common pitfalls:**
- Checking `.env` into git or exposing secrets via debug endpoints.

## 90. Encryption at rest vs in transit — define both.

**Definition:** In transit: TLS on network links. At rest: encrypt data on disk/storage (DB, backups).
**Why it matters:** Different threat models: network attackers vs disk/bucket compromise.
**Key points:**
- In transit protects over-the-wire
- At rest protects storage media and backups
- Key management is the real hard part (KMS, rotation)

## 91. What is observability for APIs (metrics you should mention)?

**Definition:** The ability to understand system behavior from outputs: metrics, logs, traces.
**Why it matters:** In production, you debug with observability, not with ‘reproduce locally’.
**Key points:**
- Golden signals: latency, traffic, errors, saturation
- Track p50/p95/p99, 4xx vs 5xx
- Add request IDs and distributed tracing context
**Interview line:** “If you can’t measure it, you can’t operate it.”

## 92. SLI/SLO/SLA — explain with a real example.

**Definition:** SLI is a measurement (e.g., % of requests under 300ms). SLO is a target. SLA is a contract with penalties.
**Why it matters:** Guides tradeoffs (ship features vs reliability) using error budgets.
**Key points:**
- Example SLI: 99% of GET /feed < 500ms
- SLO: 99% monthly target
- SLA: 99.9% uptime with credits if violated

## 93. Correlation ID / request ID — why add it and where?

**Definition:** A unique ID attached to a request and propagated through all services/logs.
**Why it matters:** Makes debugging multi-service issues possible.
**Key points:**
- Generate at edge, pass via header
- Log it in every service and include in error responses
- Use tracing IDs (W3C Trace Context) where possible

## 94. Distributed tracing — what problem does it solve?

**Definition:** Tracks a request across services with spans and timings.
**Why it matters:** Helps pinpoint where latency happens (which dependency, which hop) and identify fanout patterns.
**Key points:**
- Propagate trace context headers
- Instrument DB calls, RPC calls, queues
- Sample intelligently to control cost

## 95. GraphQL vs REST — choose with tradeoffs (not ideology).

**Definition:** GraphQL lets clients request exactly the fields they need; REST uses predefined endpoints/resources.
**Why it matters:** Frontend teams like GraphQL; backend teams worry about complexity and cost controls.
**Key points:**
- Pros: avoid over/under-fetching, single endpoint, typed schema
- Cons: N+1 resolver issues, caching harder, query complexity abuse
- Add query cost limits, persisted queries, and server-side batching
**Interview line:** “GraphQL is great for complex UI data needs if you enforce query cost and caching strategy.”

## 96. gRPC — what is it and when is it better than REST?

**Definition:** A high-performance RPC framework using HTTP/2 and Protocol Buffers with strongly typed contracts.
**Why it matters:** Common in microservices for efficiency and strong tooling.
**Key points:**
- Pros: binary, fast, streaming support, codegen
- Cons: not browser-friendly without proxies, harder debugging
- Great for internal service-to-service APIs

## 97. Protobuf/Avro schema evolution — what rules keep compatibility?

**Definition:** Evolving message schemas without breaking old/new consumers.
**Why it matters:** Event-driven systems live or die by compatibility discipline.
**Key points:**
- Only add optional fields with new tags/ids
- Never reuse field numbers/ids
- Deprecate instead of delete; maintain default behavior

## 98. What is ‘idempotent consumer’ in messaging and why does it matter?

**Definition:** A consumer that can process the same message multiple times safely.
**Why it matters:** At-least-once delivery is common; duplicates are normal, not exceptional.
**Key points:**
- Use dedup keys (event_id) stored in DB/Redis
- Make handlers upsert-based
- Avoid side effects without guard (emails, payments)
**Interview line:** “Assume duplicates; build idempotent handlers.”

## 99. Timeouts in distributed systems — what types and how to set them?

**Definition:** Limits on how long you wait for a dependency before failing fast.
**Why it matters:** Without timeouts, slow dependencies consume all workers and cause cascading failures.
**Key points:**
- Use separate connect timeout and read/response timeout
- Set budgets: total request SLO minus internal work
- Use hedged requests carefully for tail latency (advanced)
**Common pitfalls:**
- Setting timeouts too high (hang) or too low (false failures) without measuring p99.

## 100. Retries: when are they safe and how to do them correctly?

**Definition:** Retrying failed requests to handle transient errors (timeouts, 503).
**Why it matters:** Retries can both improve success rate and also cause outages if uncontrolled.
**Key points:**
- Retry only idempotent operations or use idempotency keys
- Use exponential backoff + jitter
- Cap retries; respect server `Retry-After`
- Combine with circuit breaker to avoid retry storms
**Interview line:** “Retries must be bounded and idempotent; otherwise they amplify failure.”

# 3) OS/CPU/Concurrency (101–135)


## 101. What does an Operating System do for a server process?

**Definition:** The OS schedules CPU time, manages memory, handles I/O (network/files), and enforces isolation/permissions.
**Why it matters:** Performance and scalability often hit OS limits first (fds, sockets, scheduling, page cache).
**Key points:**
- Syscalls are boundary between app and kernel
- Kernel manages TCP state, buffers, timers
- Resource limits (ulimits) can crash otherwise healthy apps

## 102. Process vs thread — what’s the difference?

**Definition:** A process has its own virtual address space; threads are execution units within a process sharing memory.
**Why it matters:** Scaling strategies depend on this: multi-process for isolation, threads for shared memory parallelism.
**Key points:**
- Threads share heap; each thread has its own stack
- Process crash isolation is stronger than threads
- IPC between processes is slower than shared memory

## 103. CPU core vs OS thread vs hardware thread (SMT) — clarify.

**Definition:** Core is physical execution unit; OS thread is schedulable context; SMT/hyperthreading exposes 2+ hardware threads per core.
**Why it matters:** Explains why ‘4-core CPU’ can run hundreds of threads but only a few truly execute simultaneously.
**Key points:**
- OS time-slices runnable threads onto cores
- SMT improves throughput when workloads stall (cache misses), not 2× always

## 104. What is context switching and why can it hurt performance?

**Definition:** Switching CPU execution from one thread to another (saving/restoring registers and scheduling state).
**Why it matters:** Too many threads → more context switches → cache misses → higher latency.
**Key points:**
- High context switch rate is a symptom of oversubscription
- Thread-per-connection models can suffer at high concurrency

## 105. Memory hierarchy (registers, L1/L2/L3, RAM, SSD) — why it matters for backend performance.

**Definition:** Different storage levels have drastically different latency and size.
**Why it matters:** Hot data in CPU cache/RAM is fast; random disk access is slow—designs rely on caching to keep hot sets in memory.
**Key points:**
- L1/L2/L3 caches reduce RAM access
- RAM is ~orders faster than SSD; SSD faster than HDD
- Access pattern (sequential vs random) matters

## 106. What is virtual memory and paging?

**Definition:** OS gives each process a virtual address space mapped to physical memory pages; pages can be swapped/loaded on demand.
**Why it matters:** Page faults can cause huge latency spikes; memory pressure affects tail latency.
**Key points:**
- Page size typically 4KB (varies)
- Major page fault hits disk (very slow)
**Common pitfalls:**
- Assuming ‘free memory’ is required—OS uses memory as file cache; focus on working set and page faults.

## 107. What is the OS page cache and why does it matter for databases/files?

**Definition:** OS caches file contents in RAM to speed reads/writes.
**Why it matters:** DB performance depends on caching layers (DB buffer pool + OS cache).
**Key points:**
- Repeated file reads often served from RAM via page cache
- Flushing and fsync affect durability and latency

## 108. Blocking vs non-blocking I/O — define with server example.

**Definition:** Blocking I/O makes a thread wait until operation completes; non-blocking allows a thread to continue and be notified later.
**Why it matters:** Explains event-loop servers vs thread-per-request servers.
**Key points:**
- Non-blocking I/O enables high concurrency with fewer threads
- But CPU-bound work still blocks the event loop unless offloaded

## 109. What is epoll/kqueue and what problem does it solve?

**Definition:** Kernel mechanisms to efficiently wait for readiness events on many file descriptors (sockets).
**Why it matters:** Allows one thread to manage thousands of idle connections without spinning or thousands of threads.
**Key points:**
- Replaces naive polling loops
- Used under the hood by Node.js, Nginx, many servers

## 110. Event loop — explain it in one paragraph (backend perspective).

**Definition:** A loop that waits for I/O events, runs callbacks, and keeps the process responsive without blocking.
**Why it matters:** Node’s scalability comes from non-blocking I/O + event loop, but CPU-heavy tasks can stall it.
**Key points:**
- One main thread runs JS callbacks
- I/O completion events queue callbacks
- Timers and microtasks influence ordering

## 111. Node.js is ‘single-threaded’ — what exactly is single?

**Definition:** The JavaScript execution (call stack) runs on one thread; Node also uses background threads for some tasks.
**Why it matters:** Interviewers test whether you understand CPU-bound vs I/O-bound behavior in Node services.
**Key points:**
- libuv thread pool handles some fs/crypto/zlib
- Networking uses OS async I/O; not one thread per socket
- Use multiple processes/worker threads for CPU-heavy work

## 112. CPU-bound vs I/O-bound — how does it affect scaling approach?

**Definition:** CPU-bound workloads spend time computing; I/O-bound spend time waiting on network/disk.
**Why it matters:** CPU-bound needs parallelism across cores; I/O-bound needs concurrency and fewer round trips.
**Key points:**
- CPU-bound: scale via multi-process, worker threads, or separate compute service
- I/O-bound: optimize with caching, batching, async I/O, connection pooling

## 113. What is the libuv thread pool in Node and when does it matter?

**Definition:** A small pool of threads used by Node for certain blocking operations (fs, crypto, compression, DNS in some cases).
**Why it matters:** Saturation causes surprising latency spikes even if event loop looks fine.
**Key points:**
- Default size often small; tasks queue up
- Offload heavy crypto/compression carefully
- Monitor event loop delay and threadpool queue

## 114. What is backpressure in systems and why do you need it?

**Definition:** A mechanism that prevents producers from overwhelming consumers by limiting in-flight work.
**Why it matters:** Without backpressure, queues grow unbounded → memory blowups and latency collapse.
**Key points:**
- Bounded queues, rate limits, stream flow control
- Load shedding when overload is inevitable

## 115. What is load shedding and why is it sometimes the correct choice?

**Definition:** Intentionally rejecting or degrading some requests under extreme load to protect the system.
**Why it matters:** Better to fail fast for some than time out for everyone and crash.
**Key points:**
- Return 429/503 quickly
- Prioritize critical traffic (auth, payments) over non-critical (analytics)

## 116. Thread pools — why not just increase threads a lot?

**Definition:** More threads increase concurrency but also add context switching and memory overhead.
**Why it matters:** Oversubscription makes latency worse; correct sizing improves throughput and stability.
**Key points:**
- Size based on cores and blocking ratio
- Separate pools for different dependencies (bulkheads)

## 117. What is a deadlock (not just in DB—also in code)?

**Definition:** Two or more threads wait forever for each other’s locks/resources.
**Why it matters:** Can freeze services under load; hard to debug.
**Key points:**
- Avoid lock ordering cycles
- Keep lock scope small
- Use timeouts and try-lock where possible

## 118. Race condition — what is it and how do you prevent it?

**Definition:** Outcome depends on timing of concurrent operations.
**Why it matters:** Causes inconsistent state and intermittent bugs.
**Key points:**
- Use atomic operations/locks
- Prefer immutability and message passing
- In distributed systems: use DB transactions or compare-and-set (CAS)

## 119. Graceful shutdown — what must a production server do before exiting?

**Definition:** Stop accepting new traffic, finish in-flight requests, flush logs/metrics, close connections cleanly.
**Why it matters:** Prevents deploy-induced errors and partial writes.
**Key points:**
- Handle SIGTERM: mark not-ready, drain, then exit
- Stop background workers safely (checkpoint offsets)

## 120. File descriptor limits — why do servers hit them?

**Definition:** Each socket/file uses a file descriptor; OS limits per process.
**Why it matters:** High-concurrency servers can fail to accept connections if fd limit is too low.
**Key points:**
- Increase ulimit, tune kernel settings
- Close idle connections; use pooling

## 121. Garbage collection (GC) and tail latency — what’s the connection?

**Definition:** GC pauses can stop application threads briefly to reclaim memory.
**Why it matters:** Even small pauses show up as p99 spikes; important for low-latency systems.
**Key points:**
- Reduce allocations, avoid huge heaps, tune GC if needed
- Use memory profiling to find leaks and churn

## 122. What is ‘thundering herd’ at the OS level?

**Definition:** Many threads/processes wake up for the same event causing contention and CPU spikes.
**Why it matters:** Shows up in naive lock/wakeup designs and can create latency storms.
**Key points:**
- Use condition variables carefully, prefer single-flight mechanisms

## 123. What is a mutex vs semaphore (quick)?

**Definition:** Mutex is a lock for mutual exclusion; semaphore counts permits for limited resources.
**Why it matters:** Helps in designing bounded concurrency and shared resource protection.
**Key points:**
- Semaphore is useful for limiting concurrent DB calls
- Mutex protects critical sections

## 124. Why do long-lived connections (WebSockets) stress memory more than short HTTP requests?

**Definition:** They keep per-connection state in kernel and app for long time (buffers, maps, heartbeats).
**Why it matters:** At 1M connections, small per-conn memory becomes huge.
**Key points:**
- Estimate memory per connection
- Use efficient connection bookkeeping and backpressure

## 125. What is NUMA and when does it matter?

**Definition:** Non-Uniform Memory Access: memory latency depends on which CPU socket accesses which RAM region.
**Why it matters:** At high scale (big servers), memory locality affects throughput; less common but good to mention for performance deep dives.
**Key points:**
- Pinning and locality can matter in very high-performance services

## 126. How do you detect an event-loop stall in Node in production?

**Definition:** Measure event loop delay/lag and correlate with CPU and GC metrics.
**Why it matters:** Stalls cause request timeouts even if network and DB are healthy.
**Key points:**
- Track event loop delay histogram
- Look for synchronous CPU work or heavy JSON parsing
- Use profiling (pprof/clinic)

## 127. What is ‘bulkhead isolation’ in a server process?

**Definition:** Separating resources (thread pools, connection pools, queues) so one dependency can’t starve everything.
**Why it matters:** Prevents cascading failures.
**Key points:**
- Separate pool for DB vs external API calls
- Cap concurrency per dependency

## 128. What is priority queueing and why use it in APIs?

**Definition:** Serving high-priority requests ahead of low-priority under contention.
**Why it matters:** Protects critical paths (login/payments) during load.
**Key points:**
- Separate queues/worker pools by priority
- Enforce fairness to avoid starvation

## 129. What is ‘saturation’ (one of the golden signals)?

**Definition:** How ‘full’ a resource is (CPU, memory, thread pool, DB connections).
**Why it matters:** High saturation predicts latency spikes before errors.
**Key points:**
- Monitor queue length, pool usage, CPU steal, disk I/O, DB connections

## 130. When do you use worker threads vs separate processes in Node?

**Definition:** Worker threads for CPU parallelism with shared memory; processes for isolation and independent heaps.
**Why it matters:** Helps scale CPU-heavy tasks and reduce impact of leaks.
**Key points:**
- Workers: faster messaging, but shared-memory hazards
- Processes: safer isolation; use cluster or orchestrator to manage

## 131. What is ‘accept queue overflow’ symptom and fix?

**Definition:** Incoming connections are dropped/timeout because server isn’t accepting fast enough or backlog too small.
**Why it matters:** Shows up under spikes; looks like random connection timeouts.
**Key points:**
- Increase backlog, optimize accept loop, scale out, use LB with retries
- Reduce expensive work at accept/handshake time

## 132. What is back-of-envelope throughput for a worker pool?

**Definition:** Approximate capacity using service time and concurrency: throughput ≈ workers / avg_time.
**Why it matters:** Helps decide worker count and whether to move work async.
**Key points:**
- Combine with Little’s Law: concurrency ≈ throughput × latency
- Account for p95 not just average

## 133. What is ‘request fanout’ and why can it kill latency?

**Definition:** One request triggers many downstream calls.
**Why it matters:** Tail latency becomes the max of many calls; failure probability increases.
**Key points:**
- Batch, cache, parallelize, or redesign data flow
- Set budgets and timeouts per dependency

## 134. Memory leak — how does it show up in production and how do you mitigate?

**Definition:** Memory usage grows over time because objects aren’t freed (references kept).
**Why it matters:** Causes GC pressure, latency spikes, and eventually OOM kills/restarts.
**Key points:**
- Monitor RSS/heap usage and GC time
- Use heap snapshots and allocation profiling
- Avoid unbounded caches/maps; add limits/TTL

## 135. What is ‘CPU steal’ in cloud VMs and why it matters?

**Definition:** Time when your VM wants CPU but the hypervisor schedules other tenants instead.
**Why it matters:** Can cause unexplained latency even when your app isn’t busy.
**Key points:**
- Monitor steal time metric
- Scale up instance class or use dedicated cores for latency-sensitive systems

# 4) Databases & Storage (136–184)


## 136. Why do we use a database instead of just writing files?

**Definition:** A DB provides efficient querying, indexing, concurrency control, durability, and operational tooling (backup/restore).
**Why it matters:** At scale, correctness and performance under concurrency are hard to build from scratch.
**Key points:**
- Handles concurrent reads/writes safely
- Provides indexes and query optimizer
- Durability via logs, checkpoints, replication

## 137. ACID — define each with a simple example.

**Definition:** Atomicity: all-or-nothing; Consistency: invariants preserved; Isolation: concurrent txns behave safely; Durability: committed data survives crashes.
**Why it matters:** Interviewers use ACID to test your correctness thinking.
**Key points:**
- Atomicity: debit+credit both happen or neither
- Isolation prevents lost updates/dirty reads
- Durability via WAL + fsync + replication

## 138. What is a transaction and why do we need it?

**Definition:** A group of operations committed together as a unit with isolation guarantees.
**Why it matters:** Without transactions, partial updates create inconsistent state (money debited but not credited).
**Key points:**
- Begin → read/write → commit/rollback
- DB enforces isolation via locks or MVCC

## 139. Isolation levels — what anomalies do they prevent?

**Definition:** Rules controlling visibility of concurrent changes.
**Why it matters:** Choosing too-weak isolation causes subtle bugs; too-strong reduces throughput.
**Key points:**
- Read Uncommitted: allows dirty reads (rarely used)
- Read Committed: prevents dirty reads; allows non-repeatable reads
- Repeatable Read: stable reads; can still have write skew (depends)
- Serializable: strongest; behaves like single-threaded execution
**Interview line:** “Pick isolation based on invariants; use SERIALIZABLE only when necessary due to cost.”

## 140. MVCC — what is it and why is it popular?

**Definition:** Multi-Version Concurrency Control stores multiple row versions so readers don’t block writers (and vice versa).
**Why it matters:** Improves concurrency for read-heavy workloads.
**Key points:**
- Readers see a snapshot at txn start (or statement start)
- Writers create new versions
- Old versions must be vacuumed/garbage collected

## 141. Locks vs MVCC — what’s the difference?

**Definition:** Locks block concurrent access; MVCC uses versions to avoid blocking reads.
**Why it matters:** Helps diagnose slow queries and contention.
**Key points:**
- Write-write conflicts still need locks or conflict detection
- Long-running transactions can bloat MVCC storage and slow vacuum

## 142. What is a primary key and why should it be stable?

**Definition:** A unique identifier for a row used for indexing and relations.
**Why it matters:** Changing primary keys is expensive and breaks references; stable keys simplify sharding and caching.
**Key points:**
- Prefer immutable IDs (UUID/ULID/Snowflake) or auto-increment where safe
- Avoid using mutable fields like email as PK

## 143. B-tree index — explain why it makes lookups fast on disk.

**Definition:** A balanced tree with high branching factor, optimized so each node fits a disk page.
**Why it matters:** Reduces the number of disk page reads needed for search (log N with small constant).
**Key points:**
- Each node contains many keys → shallow tree
- Supports range queries efficiently (ORDER BY, BETWEEN)

## 144. Hash index — when is it useful and when not?

**Definition:** An index that hashes keys for O(1) equality lookups.
**Why it matters:** Great for exact-match, bad for range scans and ordering.
**Key points:**
- Fast for `WHERE key = ...`
- Not good for `WHERE key > ...` or ORDER BY
- Some DBs use hash internally but expose B-tree commonly

## 145. Composite index (A,B) — why does column order matter?

**Definition:** An index on multiple columns where the leftmost prefix determines usability.
**Why it matters:** Wrong order leads to unused indexes and slow queries.
**Key points:**
- Index (A,B) helps queries filtering by A or (A and B)
- Does not help filtering only by B (usually)
- Choose order by selectivity + query patterns

## 146. Covering index — what is it?

**Definition:** An index that contains all columns needed by a query so the DB can answer without reading the table row.
**Why it matters:** Saves random I/O and improves latency drastically.
**Key points:**
- Sometimes called ‘index-only scan’
- Works best when selected columns are in the index

## 147. Why do too many indexes hurt write performance?

**Definition:** Each insert/update/delete must update every relevant index, increasing CPU, locks, and I/O.
**Why it matters:** Write amplification can dominate under high write loads.
**Key points:**
- More indexes → slower writes and bigger storage
- Keep only indexes that serve real queries (measure!)

## 148. What is a query planner/optimizer and what is `EXPLAIN` used for?

**Definition:** Planner chooses execution strategy (which index, join order, scan type). `EXPLAIN` shows that plan.
**Why it matters:** Performance tuning starts with understanding the plan, not guessing.
**Key points:**
- Look for full table scans, wrong index, bad join order
- Update statistics; avoid functions on indexed columns if possible

## 149. Join types — nested loop vs hash join vs merge join (intuition).

**Definition:** Different algorithms to combine rows from two sets.
**Why it matters:** Explains why some joins explode in cost and how indexes help.
**Key points:**
- Nested loop: good when one side small + index on other
- Hash join: good for large unsorted sets (build hash table)
- Merge join: good when both sides sorted by join key

## 150. Normalization vs denormalization — when do you denormalize?

**Definition:** Normalization reduces duplication; denormalization duplicates data to speed reads.
**Why it matters:** Read-heavy products (feeds, catalogs) often denormalize to meet latency and cost goals.
**Key points:**
- Denormalize when joins are too slow/expensive at scale
- Use events/ETL to keep duplicates in sync
- Accept eventual consistency for derived views

## 151. What is a ‘materialized view’ and why use it?

**Definition:** A precomputed query result stored for fast reads.
**Why it matters:** Turns expensive joins/aggregations into cheap reads (especially analytics-ish endpoints).
**Key points:**
- Needs refresh strategy (incremental or periodic)
- Good for dashboards, reporting, ‘top N’ lists

## 152. Connection pooling to DB — why necessary?

**Definition:** Reusing DB connections instead of creating a new connection per request.
**Why it matters:** Connection setup is expensive and DB has limited connection capacity.
**Key points:**
- Pool size should be tuned to DB capacity
- Too many app instances × big pools can overload DB
- Use PgBouncer-like proxies when needed

## 153. Read replicas — what problem do they solve and what new problems appear?

**Definition:** Replicas copy data from primary to scale reads and improve availability.
**Why it matters:** Often the first scaling step for SQL databases.
**Key points:**
- Scale reads by routing read-only queries to replicas
- Replication lag causes stale reads
- Critical reads may need primary (read-your-writes)

## 154. Sync vs async replication — tradeoff?

**Definition:** Sync waits for replicas to acknowledge before commit; async commits on primary then ships to replicas.
**Why it matters:** Affects durability and write latency.
**Key points:**
- Sync: stronger durability, higher latency, lower availability
- Async: fast writes, possible data loss on primary crash
- Many systems use semi-sync (at least 1 replica)

## 155. Failover — what happens when primary DB dies?

**Definition:** A replica is promoted to primary and clients must reconnect and route writes to new leader.
**Why it matters:** Failover behavior determines real availability; it’s where many systems break.
**Key points:**
- Need detection + election + fencing to avoid split-brain
- Apps must handle reconnect and retry safely

## 156. Split-brain in databases — what is it and how to avoid it?

**Definition:** Two nodes both think they are primary and accept writes, causing divergence.
**Why it matters:** Catastrophic data corruption risk.
**Key points:**
- Use consensus/quorum for leader election (Raft-like)
- Use fencing tokens/leases to ensure only one writer
- Prefer managed DB solutions when possible

## 157. Read-your-writes consistency — what is it and how to implement?

**Definition:** After a user writes, they must see their write on subsequent reads.
**Why it matters:** Users hate writing something and not seeing it due to replica lag.
**Key points:**
- Route user’s reads to primary for a short window
- Session consistency: track last write LSN and read from replica that caught up
- Or accept eventual consistency for non-critical views

## 158. Sharding — what is it and why do we do it?

**Definition:** Splitting data across multiple DB nodes by a shard key to scale writes/storage.
**Why it matters:** Single-node DB eventually hits limits; sharding is how you scale beyond.
**Key points:**
- Each shard holds a subset of keys
- Router or app chooses shard based on key
- Cross-shard joins/transactions become hard

## 159. Shard key selection — what makes a good shard key?

**Definition:** A key that distributes load evenly and matches common access patterns.
**Why it matters:** Bad shard keys create hot shards and constant rebalancing pain.
**Key points:**
- High cardinality, evenly distributed
- Most queries include the shard key
- Avoid monotonic keys (time) without bucketing

## 160. Hot shard / hot partition — how do you detect and fix it?

**Definition:** One shard receives disproportionate traffic.
**Why it matters:** Becomes bottleneck and causes p99 spikes.
**Key points:**
- Detect via per-shard QPS/CPU/latency
- Fix via better shard key, add bucketing/salting, split shard, cache hot data

## 161. Resharding — why is it painful and how do systems handle it?

**Definition:** Moving data between shards when adding/removing nodes or changing shard key.
**Why it matters:** Live data migration risks downtime and correctness issues.
**Key points:**
- Use consistent hashing or shard maps to minimize movement
- Do dual writes/reads during migration
- Throttle migration to avoid overload

## 162. Consistent hashing — why does it reduce rebalancing cost?

**Definition:** A mapping where adding/removing a node changes only a small fraction of key→node assignments.
**Why it matters:** Makes scaling caches and sharded stores operationally manageable.
**Key points:**
- Uses a hash ring with virtual nodes
- Keys map to nearest node clockwise

## 163. Partitioning (table partitioning) vs sharding — difference?

**Definition:** Partitioning splits a table inside one DB (or cluster) by range/list/hash; sharding splits across independent DB nodes.
**Why it matters:** Partitioning helps manage big tables; sharding helps scale beyond single primary.
**Key points:**
- Partitioning improves maintenance (drop old partitions), some query pruning
- Sharding changes topology and query model

## 164. Time-series / time-based partitioning — why common?

**Definition:** Splitting data by time ranges (daily/monthly partitions).
**Why it matters:** Makes retention (delete old data) cheap and keeps indexes small/hot.
**Key points:**
- Drop partition to delete old data instantly
- Queries prune partitions by time filter

## 165. WAL (write-ahead log) — what is it and why does it exist?

**Definition:** A sequential log where changes are recorded before data pages are updated.
**Why it matters:** Enables crash recovery and durable commits without random disk writes first.
**Key points:**
- Sequential writes are fast
- On crash, DB replays WAL to recover consistent state

## 166. What is `fsync` and why can it spike latency?

**Definition:** A call that forces OS to flush buffers to durable storage.
**Why it matters:** Durability has a real cost; sync commits can create p99 spikes.
**Key points:**
- Group commit can batch fsyncs
- Fast disks and WAL tuning help

## 167. LSM-tree — why do many NoSQL databases use it?

**Definition:** A write-optimized structure: write to memtable, flush to immutable SSTables, periodically compact.
**Why it matters:** Great for high write throughput and sequential I/O.
**Key points:**
- Writes are fast; reads may need to consult multiple files
- Compaction trades background I/O for read efficiency

## 168. Compaction — what is it and why can it cause latency storms?

**Definition:** Background merging of LSM files to reduce read amplification.
**Why it matters:** Compaction consumes CPU and disk I/O; if it falls behind, reads/writes degrade.
**Key points:**
- Tune compaction concurrency and disk bandwidth
- Use monitoring to detect compaction debt

## 169. Bloom filter in LSM stores — what problem does it solve?

**Definition:** A probabilistic structure to quickly determine a key is definitely not in an SSTable.
**Why it matters:** Reduces unnecessary disk reads and improves read latency.
**Key points:**
- False positives possible; false negatives not
- Small memory cost for big read savings

## 170. Secondary indexes in NoSQL — what’s tricky about them?

**Definition:** Indexes on non-primary keys in distributed NoSQL systems.
**Why it matters:** They can create hotspots and require extra coordination; often avoided or handled via search systems.
**Key points:**
- Global secondary indexes need careful partitioning
- Index updates add write amplification
- Sometimes maintain manually with denormalized tables

## 171. Optimistic locking — how does it work and when do you use it?

**Definition:** Use a version/timestamp column; update succeeds only if version matches.
**Why it matters:** Prevents lost updates without heavy locking, good for high-concurrency apps.
**Key points:**
- Read version, update with `WHERE id=? AND version=?`
- On conflict, retry or return 409 to client

## 172. Pessimistic locking — when is it necessary?

**Definition:** Lock rows before updating to prevent concurrent modification.
**Why it matters:** Needed when conflicts are common or business rules require strict serialization.
**Key points:**
- `SELECT ... FOR UPDATE` patterns
- Keep transactions short to avoid contention

## 173. Deadlocks in databases — how do they happen and what do you do?

**Definition:** Two transactions hold locks the other needs; DB detects cycle and aborts one.
**Why it matters:** Common under concurrent updates.
**Key points:**
- Consistent lock ordering reduces risk
- Retry aborted transaction
- Keep transactions small

## 174. N+1 query problem — what is it and how do you fix it?

**Definition:** Fetching a list then issuing one query per item (N extra queries).
**Why it matters:** Kills latency and DB load at scale.
**Key points:**
- Use joins or batch queries (IN clause), or prefetch patterns
- Use data loaders/batching in GraphQL

## 175. Full-text search: why not use SQL LIKE for everything?

**Definition:** Text search needs tokenization, ranking, and efficient inverted indexes; LIKE scans are slow for large text.
**Why it matters:** Search is a distinct problem; using the right tool matters.
**Key points:**
- Use dedicated search engine or DB FTS features
- Design for relevance, stemming, typo tolerance

## 176. OLTP vs OLAP — what design difference should you mention?

**Definition:** OLTP handles many small transactions; OLAP handles large scans/aggregations.
**Why it matters:** Mixing them on one DB causes performance interference.
**Key points:**
- Use separate analytics store/warehouse for OLAP
- Use CDC/ETL to feed OLAP from OLTP

## 177. Backups vs replication — why you still need backups?

**Definition:** Replication copies both good and bad changes; backups enable point-in-time restore.
**Why it matters:** Human error is the #1 cause of data loss scenarios.
**Key points:**
- Use PITR (point-in-time recovery) with WAL archiving
- Test restore regularly (backup you can’t restore is not a backup)

## 178. RPO vs RTO — define and how they influence architecture.

**Definition:** RPO: max tolerable data loss time. RTO: max tolerable downtime.
**Why it matters:** These determine whether you need multi-region replication, sync writes, or simpler backups.
**Key points:**
- Low RPO → synchronous replication or frequent log shipping
- Low RTO → automated failover and hot standby

## 179. Multi-region databases: active-passive vs active-active tradeoff.

**Definition:** Active-passive has one write region; active-active allows writes in multiple regions.
**Why it matters:** Latency vs consistency conflict is central in global systems.
**Key points:**
- Active-passive: simpler consistency, higher write latency for distant users
- Active-active: lower local write latency but needs conflict resolution/consensus (hard)

## 180. CAP theorem — say it correctly in interviews.

**Definition:** In presence of network partitions, a distributed system must trade off consistency vs availability for some operations.
**Why it matters:** Partitions are inevitable; your design must specify behavior during partitions.
**Key points:**
- Most systems choose different points for different operations
- Often: CP for money, AP for feeds/metrics

## 181. Quorum reads/writes (N, R, W) — what do they mean?

**Definition:** In replicated systems, write requires W acknowledgements, read consults R replicas; if R+W > N you can get strong read of latest write.
**Why it matters:** Shows you understand consistency knobs in distributed storage.
**Key points:**
- Example N=3, W=2, R=2 gives overlap
- Higher R/W increases latency but improves consistency

## 182. Two-phase commit (2PC) — what is it and why do people avoid it?

**Definition:** A coordinator asks participants to prepare, then commit; ensures atomic commit across resources.
**Why it matters:** It’s a classic distributed transaction approach with availability and blocking issues.
**Key points:**
- Coordinator failure can block progress
- Increases latency and coupling
- Often replaced with sagas/outbox + idempotency

## 183. Schema migrations — how do you do them safely with zero downtime?

**Definition:** Change schema in backward-compatible steps.
**Why it matters:** Bad migrations cause outages and data corruption.
**Key points:**
- Add nullable column → deploy code that writes both → backfill → switch reads → enforce constraints → remove old
- Avoid long locks; use online migration techniques

## 184. What is CDC (change data capture) and why useful?

**Definition:** Capturing DB changes (from WAL/binlog) as a stream of events.
**Why it matters:** Feeds search indexes, caches, analytics, and event-driven pipelines reliably.
**Key points:**
- Ensures ordering and completeness of changes
- Consumers can rebuild derived views

# 5) Caching/Performance + Messaging/Patterns + Mini Designs (185–200)


## 185. Cache-aside pattern — explain read and write paths.

**Definition:** Application reads from cache first; on miss reads DB and fills cache. Writes go to DB then invalidate/update cache.
**Why it matters:** Most common caching pattern; correctness depends on invalidation discipline.
**Key points:**
- Read: cache hit → fast; miss → DB → set cache with TTL
- Write: update DB → invalidate cache key (or update cache)
- Use TTL to limit staleness
**Common pitfalls:**
- Race: concurrent misses cause stampede; fix with single-flight locks.

## 186. Write-through vs write-back cache — when would you use each?

**Definition:** Write-through writes cache and DB synchronously; write-back writes cache first and persists later asynchronously.
**Why it matters:** Tradeoff between latency and durability/complexity.
**Key points:**
- Write-through: simpler correctness, higher write latency
- Write-back: fast writes, but risk data loss and complex recovery
- Most user-facing critical data uses write-through or DB-first + invalidate

## 187. Cache stampede — what is it and how do you prevent it?

**Definition:** Many requests hit a key right after expiry → all miss and hammer DB.
**Why it matters:** Creates sudden DB overload and outage.
**Key points:**
- Single-flight (mutex per key) so only one rebuilds
- Soft TTL + background refresh
- Add random jitter to TTL to avoid synchronized expiry

## 188. Cache penetration — what is it and defenses?

**Definition:** Requests for non-existent keys repeatedly miss cache and hit DB.
**Why it matters:** Attackers or bugs can overload DB with misses.
**Key points:**
- Negative caching for ‘not found’ with short TTL
- Bloom filter to reject impossible keys
- Rate limit suspicious patterns

## 189. Hot keys in Redis — what are they and how to fix?

**Definition:** A small number of keys get extremely high QPS.
**Why it matters:** Becomes CPU bottleneck and causes latency for unrelated keys.
**Key points:**
- Shard the key (e.g., counter:{bucket}) and aggregate
- Add local in-memory cache for hot reads
- Use pipelining/batching to reduce round trips

## 190. Redis as cache: what should you never cache (or cache carefully)?

**Definition:** Highly personalized or sensitive data unless keying and permissions are correct.
**Why it matters:** Cache mistakes can leak data across users/tenants.
**Key points:**
- Avoid caching responses that depend on auth unless cache key includes user/tenant scope
- Be careful with caching ‘role-based’ data; role changes require invalidation
- Prefer caching derived, non-sensitive, or properly scoped data

## 191. Cache invalidation strategies — list the common ones with pros/cons.

**Definition:** Ways to keep cached data correct when source changes.
**Why it matters:** ‘Cache invalidation is hard’ is true because it’s a consistency problem.
**Key points:**
- TTL-only: simple, stale possible
- Write invalidate: accurate but must cover all write paths
- Write update: keep cache warm but more write complexity
- Versioned keys: great for static assets (content hash)

## 192. When do you choose CDN caching vs Redis caching vs in-process caching?

**Definition:** Different cache layers serve different needs.
**Why it matters:** Correct placement gives big wins; wrong placement gives bugs or no benefit.
**Key points:**
- CDN: global edge for cacheable GET + static assets
- Redis: shared cache across app instances
- In-process: fastest but per-instance, best for extremely hot small items with short TTL

## 193. Little’s Law — how do you use it in capacity planning?

**Definition:** In steady state: Concurrency ≈ Throughput × Latency.
**Why it matters:** Quickly estimates required workers or queue sizes.
**Key points:**
- If you serve 1k rps with 100ms avg latency, concurrency ≈ 100
- Helps size thread pools, DB connections, and in-flight limits

## 194. Percentiles p50/p95/p99 — why averages lie.

**Definition:** Percentiles show distribution; average hides tail spikes.
**Why it matters:** Users feel p99; timeouts/retries depend on tail latency.
**Key points:**
- Report histograms, not just average
- Tail is amplified by fanout and GC pauses

## 195. Message queue vs stream log (Kafka) — what’s the difference?

**Definition:** Queue: messages consumed once and removed; log: append-only with offsets; consumers track position.
**Why it matters:** Choosing wrong tool complicates ordering, replay, and scalability.
**Key points:**
- Kafka allows replay, multiple consumer groups, ordering per partition
- Classic queues (RabbitMQ/SQS) focus on work distribution and ack/retry semantics

## 196. At-least-once vs at-most-once vs exactly-once — what do you implement in real systems?

**Definition:** Delivery guarantees for messaging.
**Why it matters:** Exactly-once is hard; most systems do at-least-once + idempotent consumers.
**Key points:**
- At-most-once: may drop; simplest
- At-least-once: duplicates possible; default in many MQs
- Exactly-once: usually approximated with dedup + transactional outbox
**Interview line:** “Assume duplicates and build idempotency; that’s the practical ‘exactly once’.”

## 197. DLQ (dead-letter queue) — what should go there and what’s the workflow?

**Definition:** A separate queue for messages that repeatedly fail processing.
**Why it matters:** Prevents poison messages from blocking the main queue and enables debugging/reprocessing.
**Key points:**
- Send after N retries with metadata (error, stack, attempts)
- Alert on DLQ growth
- Provide replay tooling after fixing bug/data

## 198. Outbox pattern — explain it with steps.

**Definition:** Write business update and an outbox event in the same DB transaction; a relay publishes events reliably.
**Why it matters:** Prevents ‘DB commit succeeded but event publish failed’ inconsistency.
**Key points:**
- Within txn: update tables + insert outbox row
- Publisher reads outbox and sends to broker
- Mark outbox sent (idempotent)

## 199. System design mini: Design a rate limiter for an API gateway (high-level).

**Definition:** A component that enforces per-user/per-IP/per-tenant request budgets.
**Why it matters:** Rate limiting is asked frequently and tests caching + consistency thinking.
**Key points:**
- Algorithm: token bucket stored in Redis (key per user+window) or local with periodic sync
- Use atomic operations (Lua script) to avoid race conditions
- Return 429 with Retry-After
- Use multi-layer: edge coarse limits + service fine limits

## 200. System design mini: Design a URL shortener — key decisions interviewers expect.

**Definition:** A service that maps a short code to a long URL and redirects quickly.
**Why it matters:** Tests storage model, key generation, caching, and scale estimation.
**Key points:**
- Key generation: base62 of auto-increment, Snowflake, or random with collision check
- Store mapping in DB; cache hot codes in Redis/CDN
- Redirect path must be ultra-fast; write path can be slower
- Handle abuse/spam, expiration, custom aliases, analytics asynchronously
