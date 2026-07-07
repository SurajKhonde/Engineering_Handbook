# AWS for Newbies — Episode 1: IAM, EC2 &amp; "Who's Paying for This?"
### An Uncle–Nephew Chronicle

*A ~1 hour read. No prior AWS experience assumed. Every term is defined the moment it shows up. Grab chai.*

---

👦 **Nephew:** Uncle, I want to actually *build* something on AWS this time. Not just read blog posts with 40 acronyms in the first paragraph.

👨‍🦳 **Uncle:** Good. That's the right instinct. Most people open the AWS console, see a search bar with 200 services in it, panic, and close the tab. We're not doing that today. Today you and I are going to set up exactly two things — an EC2 server, and by the end, you'll understand enough to guess what S3 is before I even tell you. And we'll do it the way I always do it — no term gets used until it's been explained. If I say a word you don't know, stop me. Deal?

👦 **Nephew:** Deal. But please — start from zero. What even *is* AWS?

---

## Part 1 — What "The Cloud" Actually Is

👨‍🦳 **Uncle:** Forget the word "cloud" for a second. It makes people imagine something floating and magical. It's not. It's just — **someone else's computer, in a building, that you rent by the hour.**

Amazon owns giant warehouses full of physical servers — real machines, real hard drives, real electricity bills — spread across the world. **AWS (Amazon Web Services)** is the business that lets you rent tiny (or huge) slices of those machines. You don't buy a computer. You don't manage a generator. You just click a few buttons and a slice of a machine becomes "yours" for as long as you're paying for it.

👦 **Nephew:** So it's like renting a flat instead of buying a house.

👨‍🦳 **Uncle:** Exactly the right metaphor, and I'll use it again and again today. Renting means: no huge upfront cost, you can move out anytime, but if you leave the lights on all month, you still pay for it. Keep that in your head. It explains almost every billing surprise you'll ever have on AWS.

👦 **Nephew:** Okay. So where do I even start?

👨‍🦳 **Uncle:** With an **AWS account**. That's the container for everything you'll ever create — servers, storage, databases, all of it lives inside one account, tied to one email and one billing method. Think of the AWS account as the entire apartment building. Inside that building, you'll rent specific rooms — a server here, some storage there.

👦 **Nephew:** And I create that with my email and a credit card?

👨‍🦳 **Uncle:** Yes. And this is where I need you to slow down and listen carefully, because this next part is the single most important lesson of the day — more important than any button you'll click.

---

## Part 2 — The Root Account: The Landlord Who Should Never Visit Daily

👨‍🦳 **Uncle:** When you sign up for AWS, the login you create — that email and password — is called the **root account** or **root user**. It has absolute, unrestricted power over everything in that AWS account. It can:

- Delete every server you've built
- Delete every database, with every customer's data in it
- Change or cancel the billing method
- Delete literally everything, no confirmation dialog can save you

👦 **Nephew:** That sounds like exactly what I want — the "admin" login.

👨‍🦳 **Uncle:** That's the trap. Having that much power available on your everyday login is like carrying the master key to every flat, the boiler room, and the bank vault, just to go water your own plants. One phishing email, one leaked password, one tired 1 AM mistake — and someone (maybe even you) can nuke the entire company's infrastructure.

So the rule, which I want you to write down: **the root account is only ever used for two things — setting up billing, and account-level emergencies.** Nothing else. Not daily work. Not even by you, the owner.

👦 **Nephew:** Then what do I use for daily work?

👨‍🦳 **Uncle:** This is where **IAM** enters.

---

## Part 3 — IAM: The Reception Desk of AWS

👨‍🦳 **Uncle:** **IAM** stands for **Identity and Access Management**. It's the AWS service whose entire job is answering one question, over and over, for every single action anyone tries to take: *"Is this person allowed to do this?"*

Think of IAM as the reception desk and security guard of your apartment building. The root account is the actual landlord who owns the building outright. Everyone else — you, your teammates, even your own automated scripts — gets an **IAM identity**, which is like an ID badge. The badge says exactly which floors, which rooms, and which switches that person is allowed to touch.

👦 **Nephew:** So an "IAM user" is basically a badge for a human?

👨‍🦳 **Uncle:** Precisely. An **IAM user** is an identity — with its own username, its own password (for console login) and/or its own access keys (for programmatic/API login) — that represents one person or one application. It is completely separate from the root login. You, as the account owner, will create an IAM user *for yourself* and use that for daily work instead of root.

👦 **Nephew:** Wait — why would I need a separate login for myself? I already have root.

👨‍🦳 **Uncle:** Because even you shouldn't be walking around with the master key every day. If your laptop gets compromised while you're logged in as an IAM user with limited permissions, the damage is contained. If it happens while you're logged in as root, it's game over. Same reason a bank manager doesn't carry the vault key in his everyday wallet — he has a separate, limited-access card for routine work.

👦 **Nephew:** Okay, I'm convinced. How do I actually create one?

---

## Part 4 — Setting Up Your First IAM User, Step by Step

👨‍🦳 **Uncle:** Let's actually do this, not just talk about it. Log in with root, just this once, to do the setup.

**Step 1 — Turn on 2FA on root first.**
Before anything else — go to your account settings and enable **2FA (Two-Factor Authentication)**. This means logging in needs your password *plus* a rotating code from an app on your phone (like Google Authenticator). This is non-negotiable for the root account. A leaked password alone should never be enough to take over your whole AWS bill.

**Step 2 — Go to IAM.**
In the AWS Console search bar, type "IAM" and open it. This is the control room for every identity in your account.

**Step 3 — Create the user.**
Path: `IAM → Users → Create user`

You'll be asked:

- **Username** — e.g. `suraj-admin`
- **Access type** — this is important, AWS asks *how* this identity will log in:
  - **AWS Management Console access** — lets the person log in through the website, with a username and password. This is for humans clicking around.
  - **Programmatic access** — generates an **Access Key ID** and **Secret Access Key**, a pair of long random strings used by code, CLI tools, or SDKs to call AWS instead of a human clicking buttons.

You can enable one or both, depending on whether this identity is "a person" or "a script."

**Step 4 — Attach permissions.**
This is where you decide what the badge actually unlocks. For your own personal admin user (used only by you, the owner), you'd typically attach the built-in **AdministratorAccess** policy, so you have full functional access — but notice: this is still not root. You can't touch billing settings or close the account, and this identity can be disabled or deleted by root if something goes wrong, without touching the account itself.

**Step 5 — Save the credentials AWS shows you.**
AWS will show you, exactly once:

- The console **login URL** (a special link tied to your account)
- **Username**
- **Password** (or a link to set one)
- If programmatic access was chosen: **Access Key** and **Secret Key**

👦 **Nephew:** "Exactly once"? What happens if I lose it?

👨‍🦳 **Uncle:** You can always reset a password, but a lost **Secret Access Key** cannot be recovered — you'd have to deactivate that key and generate a fresh one. This is why real teams store these in a password manager immediately, never in a text file on the Desktop, and *never* commit them to GitHub. I've personally seen bills spike overnight because someone pushed an access key to a public repo and a bot found it within minutes and started mining cryptocurrency on rented servers.

👦 **Nephew:** Okay, noted, painfully. Now — say I hire a developer next month. How do they get access without me handing them my login?

---

## Part 5 — Giving a Teammate Access, the Right Way

👨‍🦳 **Uncle:** You never, ever share your own login — root or IAM — with another person. Each human gets their own IAM user. But here's the deeper question you should be asking: *how much access does this person actually need?*

This is called the **Principle of Least Privilege** — give the minimum access required to do the job, nothing more. It sounds obvious, but almost every AWS security disaster in the industry traces back to someone ignoring this.

👦 **Nephew:** How do I decide "how much" for each person, though?

👨‍🦳 **Uncle:** You think about it by role. Here's a realistic split for a small startup team:

| Role | What they actually need |
|---|---|
| DevOps engineer | Full infrastructure access — servers, networking, deployments |
| Backend developer | Access to EC2 (servers) and CloudWatch (logs) — not billing, not IAM itself |
| Frontend developer | Access to S3 (for uploading built assets) — nothing else |
| Intern | Read-only access — can look, cannot touch |

👦 **Nephew:** So do I create these permissions one by one for every single new person?

👨‍🦳 **Uncle:** No — and this is the second big lesson. You use **IAM Groups**. A group is a bundle of permissions with a label, like "backend-developers." You attach the permissions to the *group* once. Then when a new backend developer joins, you just add their IAM user to that group, and they instantly inherit every permission the group has. When they leave, you remove them from the group, and access is gone instantly — no hunting through a dozen individual permission settings.

This is exactly like giving someone a badge that says "Level 2 Access" instead of programming the door lock individually for every single employee.

👦 **Nephew:** That's much cleaner. Okay — what about when a server itself, not a person, needs to reach into another AWS service? Like if my EC2 server needs to read a file from storage?

👨‍🦳 **Uncle:** Great question, and it's exactly where people mess up most often. You do **not** put an Access Key and Secret Key inside your server's code for that. Instead, you use an **IAM Role**.

An IAM Role is like a temporary, revocable badge that you *attach to a service itself* — not a person. When your EC2 server has a role attached, AWS quietly hands it short-lived, automatically-rotating credentials behind the scenes. Your code never touches a static secret key. Nothing to leak, nothing to accidentally commit to GitHub. We'll actually attach one to our server today, so you'll see it in action, not just hear about it.

👦 **Nephew:** Alright, I think I finally get the identity side. Landlord (root), badges (IAM users), teams of badges (groups), and machine badges (roles). Can we finally rent the actual "flat" now? The server?

👨‍🦳 **Uncle:** Yes. Let's talk EC2.

---

## Part 6 — EC2: Renting a Computer by the Hour

👨‍🦳 **Uncle:** **EC2** stands for **Elastic Compute Cloud**. Strip away the fancy name and it is simply: *a virtual computer, that lives in Amazon's data center, that you rent.*

"Elastic" here means it can be resized or multiplied easily — you can go from one small server to ten large ones without buying new hardware, just by clicking a button (or calling an API).

Each individual rented computer is called an **instance**.

👦 **Nephew:** So when people say "spin up an EC2 instance," they mean —

👨‍🦳 **Uncle:** They mean "turn on one of these rented virtual computers." That instance behaves just like a real computer — it has a CPU, memory (RAM), storage, an operating system, and a network connection. You can SSH into it, install software, run your Node.js backend, whatever you'd do on a real machine.

👦 **Nephew:** What decides how powerful — and how expensive — that computer is?

👨‍🦳 **Uncle:** Two choices, made when you launch it: the **AMI** and the **instance type**.

An **AMI (Amazon Machine Image)** is the operating system template your server boots from — think of it as choosing which "starting photo" of a computer you want: Ubuntu, Amazon Linux, Windows Server, and so on, sometimes with software pre-installed. For most Node.js projects, people pick Ubuntu — it's stable, has a massive community, and package management (`apt`) is easy.

The **instance type** decides the hardware specs — how many virtual CPUs, how much RAM. AWS names these in a pattern like `t3.small` or `t3.medium`. The letter-family (`t3`, `m5`, `c5`...) hints at what the machine is optimized for — `t`-family is "general purpose, burstable," `c`-family is "compute-optimized," `r`-family is "memory-optimized," and so on. The size word (`small`, `medium`, `large`) scales the actual CPU/RAM up.

👦 **Nephew:** How do I know which one to pick?

👨‍🦳 **Uncle:** You size for your actual traffic, not for imaginary future scale. If you're expecting 200–300 users and maybe 10–100 requests per second, a `t3.small` (2 vCPU, 2 GB RAM) is genuinely enough to run a Node.js app comfortably. Don't rent a mansion because you might have guests someday — rent the flat that fits today, and move later. That instinct alone will save you a lot of money.

👦 **Nephew:** Speaking of money — Uncle, this is the part that actually scares me. How does AWS decide what to charge me?

---

## Part 7 — How EC2 Billing Actually Works (No More Mystery)

👨‍🦳 **Uncle:** This is the section I want you to read twice, because "surprise AWS bill" is a genuine rite of passage for every beginner, and it's almost always avoidable once you understand the pricing model.

### 7.1 — You pay for time the machine is *running*, by the second

The core EC2 pricing model, called **On-Demand**, charges you for every second the instance is in a "running" state, at a fixed hourly rate for that instance type — with no long-term commitment. A `t3.small` in Mumbai (`ap-south-1`) region costs roughly $0.02–$0.025 per hour at the time of writing — a handful of dollars a month if it runs constantly. The moment you **stop** the instance (not delete — just stop, like putting the computer to sleep), the compute charge stops too.

👦 **Nephew:** So if I forget an instance running for a month doing nothing —

👨‍🦳 **Uncle:** You get billed for that month, doing nothing. This is the single most common beginner mistake — spinning up a test server, forgetting about it, and finding a $40 charge three weeks later for a machine nobody used after the first hour. Renting a flat and leaving the lights on, remember?

### 7.2 — Storage is billed separately, even when the machine is off

Every EC2 instance needs a virtual hard drive, called **EBS (Elastic Block Store)**. This is billed **per GB, per month**, and — this is the trap — it keeps charging even while the instance itself is *stopped*, because the disk (with all your files on it) still exists and AWS is still storing it somewhere. To stop paying for storage entirely, you have to **terminate** (fully delete) the instance, not just stop it.

| Action | Compute charge? | Storage (EBS) charge? |
|---|---|---|
| Running | Yes | Yes |
| Stopped | No | Yes — disk still exists |
| Terminated | No | No — disk is deleted |

### 7.3 — Data transfer costs money too, but mostly one direction

Data going **into** AWS (uploads) is generally free. Data going **out** of AWS to the internet (downloads, API responses, images served to users) is billed per GB, after a small free monthly allowance. This is usually a minor cost for small apps, but it becomes very real once you're serving lots of video or images — which is one reason people eventually put a CDN in front of things (a future topic, don't worry about it today).

### 7.4 — Free Tier, for practicing without fear

AWS has a **Free Tier** — a set of resources you can use at no cost, within limits, generally for your account's first 12 months (some services have an "always free" allowance too). For EC2, this typically includes 750 hours per month of a `t2.micro` or `t3.micro` instance — which is basically "one small instance running 24/7, free," enough to practice everything we're doing today without spending a rupee. Always double-check current Free Tier limits on the AWS pricing page before assuming, because these terms do get updated.

### 7.5 — Beyond On-Demand: the other pricing models, briefly

You don't need these today, but you should recognize the names:

- **Reserved Instances** — you commit to running an instance for 1 or 3 years upfront, in exchange for a heavy discount (up to ~70%). Good for predictable, always-on production workloads.
- **Spot Instances** — you bid for AWS's *spare* unused capacity, at massive discounts (up to ~90%), but AWS can reclaim the instance with only a short warning if it needs the capacity back. Good for batch jobs, not for your live database.
- **Savings Plans** — similar idea to Reserved Instances, but more flexible about which instance types the discount applies to.

👦 **Nephew:** Okay, so in short: pay for uptime by the second, pay for storage by the month regardless of power state, and don't leave things running by accident.

👨‍🦳 **Uncle:** That's the whole billing model in one sentence. Now, let's rent the flat.

---

## Part 8 — The Neighborhood First: VPC, Subnet, Security Group (Quick Primer)

👨‍🦳 **Uncle:** Before we launch the instance, three quick definitions, because the launch screen will ask about all three, and I don't want you clicking blind.

**VPC (Virtual Private Cloud)** — your own private, isolated network inside AWS. Every serious AWS project starts by creating one. Without it, your resources default to a shared, more exposed network setup. Think of the VPC as the entire gated housing society your flat sits inside.

**Subnet** — a subdivision of that VPC's network. A **public subnet** has a path to the internet; a **private subnet** does not. Your server, at least for today's simple setup, will sit in a public subnet so you (the developer) can reach it directly to configure it.

**Security Group** — a virtual firewall attached to your instance. It's a list of rules saying exactly which ports, and from which source IP addresses, are allowed to reach this machine. Nothing gets in unless a rule explicitly allows it. Think of it as the actual lock on your specific flat's door — separate from the housing society's outer gate.

👦 **Nephew:** Got it — society (VPC), street inside the society (subnet), and my door lock (security group). Now can we launch this thing?

---

## Part 9 — Launching the EC2 Instance, Step by Step

👨‍🦳 **Uncle:** Let's walk through it exactly as you'll see it in the console.

**Step 1 — Name it.**
`EC2 → Launch instance`. Give it a name, e.g. `node-app-server`. This is just a label to help you find it later among your other instances — it costs nothing and saves you a headache in month three when you have twelve running instances and no idea what half of them do.

**Step 2 — Choose the AMI (operating system).**
Pick **Ubuntu 22.04 LTS**. "LTS" means Long Term Support — it gets security patches for years, which matters for anything you plan to keep running.

**Step 3 — Choose the instance type.**
Pick **t3.small** — 2 vCPU, 2 GB RAM. Enough for a small Node.js app comfortably.

**Step 4 — Create (or choose) a key pair.**
This is your SSH credential — the digital key that proves it's really you connecting to the server, instead of a password. AWS generates a public/private key pair; you download the **private key file** (e.g. `node-server-key.pem`) and it's shown to you exactly once. Guard this file the way you'd guard your house keys — anyone with it can log into your server.

👦 **Nephew:** What's SSH again, in plain words?

👨‍🦳 **Uncle:** **SSH (Secure Shell)** is just the standard, encrypted way to open a command-line session into a remote computer over the internet. It's how you'll "log in" to your EC2 instance from your own laptop's terminal.

**Step 5 — Network settings.**
Choose the VPC we discussed (e.g. `production-vpc`) and the **public subnet**. Set **Auto-assign Public IP** to **ON** — this gives the instance an internet-reachable address the moment it boots. Without this, the instance only has a private IP, reachable only from inside the VPC — invisible to you sitting outside on your home Wi-Fi.

**Step 6 — Configure the security group (the door lock).**
This is where you decide who's allowed to knock. For a fresh server you're about to configure yourself, a sensible starting rule set is:

| Port | Purpose | Source (who's allowed) |
|---|---|---|
| 22 | SSH — your remote terminal access | **Your IP only** (never `0.0.0.0/0` for SSH!) |
| 80 | HTTP — normal web traffic | `0.0.0.0/0` (anyone on the internet) |
| 3000 | Your Node.js app (while testing) | `0.0.0.0/0`, or restrict later once behind a proper setup |

👦 **Nephew:** What does `0.0.0.0/0` actually mean? I keep seeing it everywhere.

👨‍🦳 **Uncle:** It means "any IPv4 address in existence" — i.e., the entire internet, unrestricted. It's the right choice for a public website's HTTP/HTTPS ports, because you genuinely want any visitor to reach it. It is a **dangerous mistake** on a database port or your SSH port, because it means literally any bot scanning the internet can attempt to knock on that door. Restrict SSH to your own IP address specifically — AWS even offers a "My IP" auto-fill option in the console for exactly this reason.

**Step 7 — Storage.**
20 GB of **gp3** SSD storage is a reasonable, cheap default for a small Node.js server. Remember from the billing section — this keeps charging monthly even if you stop the instance later.

**Step 8 — Attach the IAM role (remember Part 5?).**
If this server will ever need to talk to another AWS service (say, reading a file from storage), attach an IAM Role here instead of hardcoding any access keys into your code. We're planting the seed for next episode already.

**Step 9 — Launch.**

Click **Launch Instance**. In about a minute, AWS finishes provisioning, and you'll see a **public IPv4 address** assigned to your instance in the console.

---

## Part 10 — Connecting to Your Server and Making It "Live"

👨‍🦳 **Uncle:** From your own laptop's terminal, first lock down the key file's permissions (Linux/Mac):

```
chmod 400 node-server-key.pem
```

Then connect:

```
ssh -i node-server-key.pem ubuntu@<your-ec2-public-ip>
```

You're now inside the server — a genuinely remote Ubuntu machine, sitting in Amazon's data center, that you're commanding from your living room.

From here, a typical setup for a Node.js app looks like:

```
sudo apt update
sudo apt install -y nodejs npm git
git clone <your-repo-url>
cd your-app
npm install
```

Set your environment variables (things like database connection strings) in a `.env` file rather than hardcoding them, and install **PM2** — a process manager that keeps your app running in the background, and automatically restarts it if it crashes:

```
sudo npm install -g pm2
pm2 start server.js
pm2 list
```

👦 **Nephew:** And now anyone visiting `http://<my-ip>:3000` can see my app?

👨‍🦳 **Uncle:** Exactly — because port 3000 is open in the security group, and Auto-assign Public IP gave the instance an internet-reachable address. That's the whole loop closed: someone types your IP into a browser, their request reaches AWS's network, hits your security group's rule check, passes, arrives at your Node.js process, and PM2's kept it alive to answer.

👦 **Nephew:** One thing bugs me — will that public IP stay the same forever?

👨‍🦳 **Uncle:** Good catch, and this trips people up. By default, the auto-assigned public IP is **dynamic** — if you stop and start the instance again (not just reboot, but stop/start), you'll often get a *new* IP address. If you need a fixed, unchanging address — say, because you're pointing a domain name at it — you'd reserve an **Elastic IP**, a static public IP address you own and can attach to any instance, and it stays constant even across stop/starts. It's free while attached to a running instance, but AWS charges a small hourly fee if you reserve one and leave it *unattached* — a subtle billing trap worth knowing about, but you don't need it for today's practice run.

---

## Part 11 — Quick Recap Before We Stop

👨‍🦳 **Uncle:** Let's replay what actually happened today, in order:

1. We understood AWS as "rented computers in Amazon's buildings," billed like a rental — pay for what's on, keep paying for storage regardless.
2. We separated the **root account** (emergency-only, all-powerful) from **IAM users** (day-to-day, limited-permission badges).
3. We created an IAM user for ourselves, and learned how a teammate would get their *own* badge, bundled into **IAM Groups** by role, following the **Principle of Least Privilege**.
4. We learned **IAM Roles** are how *machines* (like our EC2 server) get credentials — never hardcoded keys.
5. We defined **EC2** — a rented virtual computer — and its two core choices: **AMI** (operating system) and **instance type** (hardware size).
6. We broke down billing: **On-Demand** hourly compute charges, **EBS** storage charges that persist even when stopped, data transfer costs, and the **Free Tier** safety net for practicing.
7. We met the neighborhood: **VPC** (private network), **subnet** (public vs private), **security group** (the actual firewall/door lock).
8. We launched a real instance — chose the AMI, instance type, key pair, network settings, security group rules, storage, and an IAM role.
9. We connected over **SSH**, installed Node.js, and used **PM2** to keep the app alive.
10. We opened the door — understood exactly why `0.0.0.0/0` on port 3000 made the app reachable by anyone, and why the same rule on SSH or a database port would be dangerous.
11. We touched on **Elastic IP** for when you need a public address that never changes.

---

## Part 12 — A Glossary, Because You Asked for Every Term Defined

| Term | Plain-English definition |
|---|---|
| **AWS** | Amazon's business that rents you computing resources — servers, storage, databases — by usage. |
| **AWS Account** | The container for everything you create on AWS, tied to one email and billing method. |
| **Root Account** | The all-powerful login created at sign-up. Emergency/billing use only — never daily work. |
| **IAM** | Identity and Access Management — AWS's system for deciding who/what can do what. |
| **IAM User** | An identity (with its own login) representing one person or app, with limited, assigned permissions. |
| **IAM Group** | A labeled bundle of permissions; users are added to groups to inherit access in bulk. |
| **IAM Role** | A temporary, revocable set of credentials attached to a *service* (like a server), not a person — avoids hardcoded secret keys. |
| **Access Key / Secret Key** | A credential pair used by code/CLI tools to authenticate to AWS programmatically. |
| **2FA** | Two-Factor Authentication — a second proof of identity (like a phone code) beyond just a password. |
| **Principle of Least Privilege** | Give the minimum access needed for the job — nothing more. |
| **EC2** | Elastic Compute Cloud — AWS's rentable virtual computer service. |
| **Instance** | One individual rented virtual computer running on EC2. |
| **AMI** | Amazon Machine Image — the operating system template an instance boots from. |
| **Instance Type** | The hardware spec (CPU/RAM) of an instance, e.g. `t3.small`. |
| **On-Demand Pricing** | Pay-by-the-second billing for a running instance, no long-term commitment. |
| **Reserved Instance** | A 1–3 year commitment to an instance type in exchange for a large discount. |
| **Spot Instance** | Renting AWS's spare capacity at a steep discount, with the risk it can be reclaimed. |
| **EBS** | Elastic Block Store — the virtual hard disk attached to an EC2 instance, billed monthly regardless of instance power state. |
| **Free Tier** | A set of AWS resources usable at no cost within limits, generally for the first 12 months of an account. |
| **VPC** | Virtual Private Cloud — your own isolated private network inside AWS. |
| **Subnet** | A subdivision of a VPC's network; public (internet-reachable) or private (internal-only). |
| **Security Group** | A virtual firewall on a resource, defining allowed inbound/outbound ports and sources. |
| **0.0.0.0/0** | Notation meaning "every possible IPv4 address" — i.e., the entire internet. |
| **SSH** | Secure Shell — the standard encrypted protocol for remotely logging into a server's command line. |
| **Key Pair** | The public/private cryptographic key combo used to securely SSH into an EC2 instance instead of a password. |
| **Public IP** | An internet-reachable address assigned to an instance; dynamic by default (can change on stop/start). |
| **Elastic IP** | A static, reserved public IP you own, which stays fixed even across an instance's stop/start cycles. |
| **PM2** | A Node.js process manager that keeps an app running in the background and restarts it on crash. |

---

## Part 13 — Where We're Headed Next (S3)

👦 **Nephew:** Okay, I have a server running, I can reach it, I understand the bill. What's next?

👨‍🦳 **Uncle:** Think about it for a second before I answer — you tell me. Right now, if a user uploads a profile picture to your app, where does that file physically end up?

👦 **Nephew:** ...On the server itself, I guess? In some folder?

👨‍🦳 **Uncle:** Right. And what happens to that file the day your server crashes, or you need to run *two* servers behind a load balancer for reliability, and the user's next request happens to land on the *other* server that never saw that file?

👦 **Nephew:** ...It's just gone. Or missing, depending which server they hit.

👨‍🦳 **Uncle:** Exactly the problem. A server should be treated as disposable and replaceable — what AWS people call "stateless." Files need a permanent home *outside* any individual server. That permanent home is called **S3 — Simple Storage Service** — and it's our next episode. We'll cover: what an "object" and a "bucket" actually are, how billing works there (very different model from EC2), how to keep uploads private by default, and the elegant trick called a **presigned URL** that lets a user upload a file straight to S3 without your server ever touching the raw bytes.

👦 **Nephew:** I actually can't wait for that one.

👨‍🦳 **Uncle:** That's the goal — by the time we're done with this series, you won't just know 200 AWS terms. You'll know *why* each piece exists, because you'll have hit the exact problem that piece was invented to solve, right before I hand you the solution. That's how this sticks.

---

*End of Episode 1. Next up — Episode 2: S3, Object Storage &amp; the Presigned URL Trick.*
