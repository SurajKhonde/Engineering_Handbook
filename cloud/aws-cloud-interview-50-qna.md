# AWS / Cloud Interview — 50 Questions with Answers (3 YOE)  
*(Focus: AWS deployment with EC2 + Lambda. Includes “memory technique” for each.)*  
Last updated: 2026-01-24

---

## How to answer in interviews (the “W-H-T” pattern)
For most questions, answer in this order:

1) **What** it is (1 line)  
2) **Why** it exists (problem it solves)  
3) **How** it works (key moving parts)  
4) **Tradeoff** (when NOT to use)  
5) **Example** (1 real use-case)

I’ll tag a short **Memory hook** for each question so you can recall it quickly.

---

# A) Cloud & AWS Fundamentals (1–8)

## 1) What is “Cloud Computing”?
**Answer:** Renting compute, storage, and networking on-demand with pay-as-you-go, instead of buying/maintaining servers.  
**Memory hook:** *Rent vs Buy.*  
**Interview move:** Add one line about **elasticity** (scale up/down) + **managed services**.

## 2) What is AWS “Region” vs “Availability Zone”?
**Answer:** A **Region** is a geographic area (e.g., Mumbai). An **AZ** is an isolated datacenter group inside a region (e.g., ap-south-1a/1b). Use multiple AZs for HA.  
**Memory hook:** *Region = City, AZ = Buildings.*  
**Tradeoff:** Multi-AZ improves availability but can add complexity/cost.

## 3) What is Shared Responsibility Model?
**Answer:** AWS secures the **cloud** (hardware, data centers). You secure **in the cloud** (IAM, OS patches on EC2, app security, data encryption).  
**Memory hook:** *AWS locks the building; you lock your apartment.*  
**Example:** On EC2, you patch OS; on Lambda, AWS patches runtime infra.

## 4) What is an AWS account and why use multiple accounts?
**Answer:** Account is the top-level security/billing boundary. Multiple accounts isolate environments (dev/stage/prod), reduce blast radius, and simplify access controls.  
**Memory hook:** *One wallet vs many wallets (risk isolation).*  

## 5) What is an AWS “VPC”?
**Answer:** A logically isolated private network in AWS where you define subnets, routing, and security rules.  
**Memory hook:** *VPC = your private office network in AWS.*  

## 6) What is “High Availability” in AWS terms?
**Answer:** Designing so the service continues despite failures—usually via **Multi-AZ**, load balancing, and stateless instances.  
**Memory hook:** *No single box should be a single point of failure.*  

## 7) What is “Scalability” vs “Elasticity”?
**Answer:** **Scalability** = can grow capacity. **Elasticity** = auto-adjust capacity with demand.  
**Memory hook:** *Scalable = bigger gym, elastic = gym adds/removes machines automatically.*  

## 8) What is “Fault Tolerance”?
**Answer:** System keeps running even if components fail, often using redundancy and automated recovery.  
**Memory hook:** *Fail, but service doesn’t.*  
**Example:** Auto Scaling across AZs + health checks.

---

# B) Compute: EC2, Auto Scaling, Load Balancing (9–18)

## 9) What is EC2?
**Answer:** Virtual servers in AWS with control over OS, runtime, networking (like renting a VM).  
**Memory hook:** *EC2 = “my server” in the cloud.*  

## 10) EC2 vs Lambda — when would you use each?
**Answer:**  
- **EC2:** long-running apps, custom OS, steady workloads, WebSockets, background daemons.  
- **Lambda:** event-driven, bursty workloads, pay-per-use, quick scaling, less ops.  
**Memory hook:** *EC2 = own bike; Lambda = Uber.*  
**Tradeoff:** Lambda has time limits + cold starts; EC2 needs patching/ops.

## 11) What is an AMI?
**Answer:** A template image for launching EC2 (OS + preinstalled packages).  
**Memory hook:** *AMI = “golden image” snapshot for servers.*  

## 12) What is an Auto Scaling Group (ASG)?
**Answer:** Manages a fleet of EC2 instances: scales in/out based on policies and replaces unhealthy instances.  
**Memory hook:** *ASG = “server factory + bouncer” (keeps correct count, kicks bad ones).*  

## 13) What is an Application Load Balancer (ALB)?
**Answer:** Layer 7 load balancer for HTTP/HTTPS—routes by path/host, supports health checks, target groups.  
**Memory hook:** *ALB = traffic police for HTTP.*  
**Example:** `/api` to Node service, `/static` to another target.

## 14) ALB vs NLB vs Classic ELB?
**Answer:**  
- **ALB:** L7, HTTP routing, websockets supported, best for web apps.  
- **NLB:** L4, very high performance, TCP/UDP, static IP.  
- **Classic:** legacy.  
**Memory hook:** *ALB=App, NLB=Network.*  

## 15) What are Security Groups?
**Answer:** Stateful firewall attached to ENIs/instances—controls inbound/outbound rules.  
**Memory hook:** *Security Group = “who can knock on my door”.*  
**Tip:** Stateful means response traffic is automatically allowed.

## 16) What are NACLs?
**Answer:** Stateless subnet-level rules (allow/deny) applied to all resources in a subnet.  
**Memory hook:** *NACL = “gate rules for the whole street”.*  
**Tradeoff:** Harder to manage; SGs are preferred for most.

## 17) How do you do a zero-downtime deployment on EC2?
**Answer:** Use ALB + ASG + rolling update: launch new instances (new AMI), register to target group, pass health checks, then terminate old ones.  
**Memory hook:** *Blue/Green or Rolling behind LB.*  

## 18) How do you handle instance bootstrapping on EC2?
**Answer:** User Data scripts or baked AMIs. For fast scale, prefer **baked AMIs** with minimal boot tasks.  
**Memory hook:** *Bake more, bootstrap less.*

---

# C) Serverless: Lambda, API Gateway, Eventing (19–27)

## 19) What is AWS Lambda?
**Answer:** Runs code on-demand in response to events; you pay for execution time + requests.  
**Memory hook:** *Lambda = “functions, not servers”.*  

## 20) What is “cold start” in Lambda?
**Answer:** Extra latency when a new execution environment must be created (init runtime + code).  
**Memory hook:** *Cold start = “first coffee takes time”.*  
**Mitigation:** Provisioned Concurrency, smaller package, reuse connections.

## 21) Lambda concurrency — what does it mean?
**Answer:** Number of parallel executions. Each event can spin up separate concurrent invocations.  
**Memory hook:** *Concurrency = “how many lambdas run at once”.*  
**Interview tip:** Mention throttling + reserved concurrency.

## 22) API Gateway vs ALB for Lambda?
**Answer:** API Gateway adds auth, throttling, request validation, caching, usage plans; ALB is simpler routing and can invoke Lambda too.  
**Memory hook:** *Gateway = API features; ALB = routing.*  

## 23) How do you version and deploy Lambda safely?
**Answer:** Use versions + aliases (e.g., `prod` alias). Shift traffic gradually (canary) using alias routing.  
**Memory hook:** *Version + Alias = safe release knob.*  

## 24) How do retries work for Lambda?
**Answer:** Depends on trigger:  
- Async invokes often have built-in retries + DLQ/On-Failure destination.  
- SQS event source retries until success/visibility timeout; then DLQ if configured.  
**Memory hook:** *Retry belongs to the event source.*  

## 25) What are Lambda Layers?
**Answer:** Shared code/dependencies packaged separately and reused across functions.  
**Memory hook:** *Layer = “shared library zip”.*  

## 26) What is EventBridge?
**Answer:** Event bus for routing events between services with rules/targets (decoupling).  
**Memory hook:** *EventBridge = “event router”.*  

## 27) When would you NOT use Lambda?
**Answer:** Very long jobs, heavy CPU/large binaries, ultra-low latency without cold start, persistent connections, strict runtime needs.  
**Memory hook:** *If it’s “always-on”, Lambda may not fit.*

---

# D) Networking: Subnets, NAT, Routing, DNS (28–35)

## 28) Public vs Private Subnet?
**Answer:** Public subnet has route to Internet Gateway; private subnet does not.  
**Memory hook:** *Public = can reach internet directly; Private = hidden.*  

## 29) What is an Internet Gateway (IGW)?
**Answer:** Enables internet connectivity for resources in public subnets.  
**Memory hook:** *IGW = door to the internet.*  

## 30) What is a NAT Gateway?
**Answer:** Allows private subnet instances to access internet **outbound only** (for updates, downloads) without being publicly reachable.  
**Memory hook:** *NAT = “private machines can browse web, but web can’t reach them”.*  

## 31) How do route tables work?
**Answer:** They decide where traffic goes (local, IGW, NAT, VPC peering, etc.). Subnets are associated with route tables.  
**Memory hook:** *Route table = GPS rules for subnet traffic.*  

## 32) What is VPC Peering?
**Answer:** Private connection between two VPCs to route traffic using private IPs (non-transitive).  
**Memory hook:** *Peering = “direct private lane”.*  

## 33) What is Transit Gateway?
**Answer:** Hub-and-spoke routing for many VPCs/on-prem, solves peering mesh complexity.  
**Memory hook:** *TGW = “network router hub”.*  

## 34) What is Route 53 used for?
**Answer:** DNS service for routing domains to endpoints, health checks, and routing policies (weighted, latency, failover).  
**Memory hook:** *Route 53 = “DNS + smart routing”.*  

## 35) What is a “bastion host”?
**Answer:** A hardened public instance used to SSH into private instances (now often replaced by SSM Session Manager).  
**Memory hook:** *Bastion = “one guarded entry point”.*  

---

# E) Storage: S3, EBS, EFS (36–41)

## 36) What is S3?
**Answer:** Object storage (files as objects) with very high durability, used for static assets, backups, logs.  
**Memory hook:** *S3 = “infinite folder of objects”.*  

## 37) S3 vs EBS vs EFS?
**Answer:**  
- **S3:** object storage, not mounted like disk.  
- **EBS:** block storage attached to EC2 (like a hard disk).  
- **EFS:** managed network file system (shared mount) across instances.  
**Memory hook:** *S3=objects, EBS=disk, EFS=shared folder.*  

## 38) What is an S3 bucket policy?
**Answer:** Resource-based JSON policy that controls who can access the bucket/objects.  
**Memory hook:** *Bucket policy = “rules on the bucket itself”.*  

## 39) What are S3 pre-signed URLs?
**Answer:** Time-limited URL granting access to upload/download without making bucket public.  
**Memory hook:** *Pre-signed URL = “temporary ticket”.*  
**Example:** Direct browser upload to S3.

## 40) How do you serve static assets securely from S3?
**Answer:** Put CloudFront in front of S3, use Origin Access Control/Identity, keep bucket private.  
**Memory hook:** *CloudFront = secure CDN gatekeeper.*  

## 41) What is S3 lifecycle policy?
**Answer:** Automates transitions to cheaper storage classes and expiration.  
**Memory hook:** *Lifecycle = “auto-archive and delete”.*

---

# F) Databases & Caching (42–46)

## 42) What is RDS?
**Answer:** Managed relational databases (MySQL/Postgres/etc.) with backups, patching, Multi-AZ options.  
**Memory hook:** *RDS = “DB without DB ops”.*  

## 43) Multi-AZ vs Read Replica in RDS?
**Answer:**  
- **Multi-AZ:** HA failover (same region, standby).  
- **Read Replica:** scale reads, async replication.  
**Memory hook:** *Multi-AZ = survive failure, Replica = read scale.*  

## 44) What is DynamoDB?
**Answer:** Fully managed NoSQL key-value/document DB with single-digit ms latency, massive scale.  
**Memory hook:** *DynamoDB = “serverless NoSQL”.*  

## 45) What is ElastiCache (Redis) used for?
**Answer:** In-memory caching, sessions, rate limiting, pub/sub, leaderboards.  
**Memory hook:** *Redis = “fast memory near app”.*  
**Interview tip:** Mention TTL + cache invalidation strategy.

## 46) How do you avoid “cache stampede”?
**Answer:** Use TTL jitter, request coalescing (single flight), stale-while-revalidate, or locks.  
**Memory hook:** *Stampede = many users miss cache together → DB dies.*

---

# G) Security & IAM (47–52)

## 47) What is IAM?
**Answer:** Identity and Access Management—users/roles/policies controlling permissions.  
**Memory hook:** *IAM = “who can do what”.*  

## 48) IAM Role vs IAM User?
**Answer:**  
- **User:** for humans (console/CLI).  
- **Role:** for services/apps (temporary credentials via STS).  
**Memory hook:** *User=person, Role=hat your service wears.*  

## 49) What is “least privilege”?
**Answer:** Grant only the minimum permissions needed—reduces blast radius if compromised.  
**Memory hook:** *Small keys, not master keys.*  

## 50) How do you store secrets safely on AWS?
**Answer:** Use Secrets Manager or SSM Parameter Store (SecureString), rotate secrets, avoid hardcoding, restrict IAM access.  
**Memory hook:** *Secrets in vault, not in code.*  

## 51) What is KMS?
**Answer:** Key Management Service—manages encryption keys, integrates with S3, EBS, RDS, etc.  
**Memory hook:** *KMS = “key locker”.*  

## 52) What is VPC Endpoint (Gateway/Interface)?
**Answer:** Private access to AWS services (like S3, DynamoDB) without internet/NAT.  
**Memory hook:** *Endpoint = “private tunnel to AWS services”.*  

---

# H) Observability: Logs, Metrics, Tracing (53–57)

## 53) What is CloudWatch?
**Answer:** Metrics + logs + alarms. You monitor CPU, latency, errors, custom metrics.  
**Memory hook:** *CloudWatch = “AWS monitoring dashboard”.*  

## 54) How do you set up alarms for an API?
**Answer:** Alarm on p95 latency, 5xx rate, error count, saturation (CPU/memory), and dependency failures.  
**Memory hook:** *RED + USE methods.*  
- RED: Rate, Errors, Duration  
- USE: Utilization, Saturation, Errors

## 55) What is distributed tracing and why X-Ray?
**Answer:** Tracing links a request across services to find bottlenecks. AWS X-Ray helps see service map + latency breakdown.  
**Memory hook:** *Tracing = “request GPS”.*  

## 56) How do you debug a production incident quickly?
**Answer:** Check alarms → dashboards → logs with correlation IDs → trace → rollback/mitigate.  
**Memory hook:** *Signal → Slice → Fix.*  

## 57) What’s the difference between logs and metrics?
**Answer:** Metrics are numbers over time (fast alerting), logs are detailed events (deep debugging).  
**Memory hook:** *Metrics tell “something broke”, logs tell “why”.*

---

# I) Deployment & CI/CD (58–63)

## 58) What is Infrastructure as Code (IaC)?
**Answer:** Manage infra via code (CloudFormation/Terraform/CDK) for repeatability and review.  
**Memory hook:** *Infra = code, not clicks.*  

## 59) How do you do CI/CD for EC2 services?
**Answer:** Build artifact/AMI → deploy via ASG rolling/blue-green → health checks → rollback on failure.  
**Memory hook:** *Build → Bake → Shift → Verify → Rollback.*  

## 60) What is CodeDeploy used for?
**Answer:** Automated deployments to EC2/ASG/Lambda with strategies (rolling, blue/green).  
**Memory hook:** *CodeDeploy = “release manager”.*  

## 61) What is blue/green deployment?
**Answer:** Two identical environments. Route traffic from blue (old) to green (new) after validation. Rollback by switching back.  
**Memory hook:** *Two lanes, move traffic.*  

## 62) What is canary release?
**Answer:** Gradually send small % of traffic to new version, watch metrics, then increase.  
**Memory hook:** *Small bite first.*  

## 63) How do you rollback safely?
**Answer:** Keep previous AMI/version, automate rollback triggers via alarms, and keep DB migrations backward compatible.  
**Memory hook:** *Rollback needs “undo button” + compatible DB.*

---

# J) Architecture & System Design on AWS (64–70)

## 64) How would you host a Node API on AWS (simple, scalable)?
**Answer:** ALB → ASG (EC2) in private subnets, NAT for outbound, RDS in private subnets, CloudWatch for monitoring.  
**Memory hook:** *LB front, private app, private DB.*

## 65) How do you make a system stateless for scaling?
**Answer:** Store session state in Redis or JWT, store files in S3, keep instances replaceable.  
**Memory hook:** *No user data on instance.*  

## 66) How do you handle file uploads at scale?
**Answer:** Client uploads directly to S3 via pre-signed URL, backend stores metadata in DB, CloudFront serves assets.  
**Memory hook:** *Browser → S3, not Browser → Server.*  

## 67) How do you reduce cost on AWS?
**Answer:** Right-size instances, use Auto Scaling, Spot for non-critical, caching, S3 lifecycle, reserved/savings plans.  
**Memory hook:** *Pay less by matching demand.*  

## 68) How do you design for DR (Disaster Recovery)?
**Answer:** Backups, multi-region replication for critical data, infra as code, tested runbooks. Choose RTO/RPO.  
**Memory hook:** *RTO=how fast, RPO=how much data loss.*  

## 69) What is SQS and where does it fit?
**Answer:** Managed message queue for decoupling and buffering. Producers push; consumers pull.  
**Memory hook:** *SQS = shock absorber between services.*  

## 70) How do you secure a public API on AWS?
**Answer:** TLS everywhere, WAF, rate limiting, IAM/Cognito/JWT auth, least privilege, private subnets, secrets manager.  
**Memory hook:** *Encrypt, authenticate, limit, isolate.*

---

# Quick Practice Drill (How to memorize fast)
Pick 10 questions/day. For each one, practice a 20–30 second answer using:

**Definition → Use-case → Tradeoff → Example**

If you can do that without notes, you’re ready.

---

## If you want, I can also generate:
- 20 “scenario-based” AWS questions (real interview style)
- A 1-page cheat sheet (most asked AWS topics)
