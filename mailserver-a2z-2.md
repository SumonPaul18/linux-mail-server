নিচে আপনার জন্য একটি **প্রফেশনাল, কমপ্লিট, রিয়েল-ওয়ার্ল্ড রেডি ও Production-Grade Mail Server ট্রেনিং কোর্স আউটলাইন** এবং সম্পূর্ণ কনটেন্ট দেওয়া হলো। এই কোর্সটি আপনি ধারাবাহিকভাবে ট্রেনিং দেবার জন্য ব্যবহার করতে পারবেন। এখানে **Day-1** থেকে শুরু করে প্রতিটি দিনের বিস্তারিত থিওরি, প্র্যাকটিক্যাল, ক্যালকুলেশন, প্রশ্নোত্তর, বাস্তব উদাহরণ, সিকিউরিটি লেয়ার, প্রোটোকল, এবং সেটআপ গাইডলাইন অন্তর্ভুক্ত করা হয়েছে।

---

## ✅ কোর্সের নাম:  
**“Production-Grade Open Source Mail Server Mastery: From Zero to Hero”**  
*ট্রেনার: Sumon Paul — DevOps & Cloud Engineer*

---

## 📄 README.md (Markdown Format)

```markdown
# 📧 Production-Grade Open Source Mail Server Mastery  
## From Zero to Hero — Designed for DevOps & Cloud Engineers  
**Trainer: Sumon Paul**  
*Last Updated: April 2025*

---

## 🎯 Course Objective

This comprehensive, hands-on training course is designed to equip you with **real-world, production-ready skills** to architect, deploy, secure, and manage **open-source mail servers** — both **on-premise** and **cloud-hosted**. You will gain deep theoretical understanding followed by step-by-step practical implementation, troubleshooting, and optimization — exactly as required in enterprise environments.

By the end of this course, you will be able to:
- Explain every layer of email infrastructure with real-world analogies.
- Calculate system requirements for any scale of deployment.
- Deploy multiple types of mail servers (Postfix, Dovecot, iRedMail, Mail-in-a-Box, etc.).
- Configure DNS records (MX, SPF, DKIM, DMARC, PTR).
- Implement TLS, SASL, MTA-STS, DANE, and other security layers.
- Monitor, log, backup, and scale mail servers.
- Troubleshoot delivery, spam, authentication, and performance issues.
- Train others confidently using this curriculum.

---

## 🧭 Course Outline (7 Days)

### 📅 Day 1: Theoretical Deep Dive & Foundation
- What is a Mail Server? Core Components & Architecture
- Email Flow: From Sender to Recipient (with Real Diagrams)
- Protocols: SMTP, IMAP, POP3, Submission, LMTP — Deep Comparison
- Security Layers: TLS, SASL, SPF, DKIM, DMARC, MTA-STS, DANE, BIMI
- Pre-Requisites: DNS, Reverse DNS, FQDN, SSL Certificates, Ports & Firewalls
- System Configuration Calculation: RAM, CPU, Disk, Bandwidth per User
- Mail Server Types: MTA, MDA, MUA, MSA — Roles & Responsibilities
- Common Misconceptions & Real-World Pitfalls

### 📅 Day 2: On-Premise Self-Hosted Mail Server (Postfix + Dovecot)
- Environment Setup: Ubuntu/Debian, Minimal Install
- Install & Configure Postfix (SMTP MTA)
- Install & Configure Dovecot (IMAP/POP3 MDA)
- Integrate SASL for SMTP Authentication
- Configure Virtual Domains & Users (MySQL/PostgreSQL backend)
- Generate & Configure SSL/TLS Certificates (Let’s Encrypt)
- Test SMTP/IMAP with Telnet, OpenSSL, Thunderbird
- Enable Logging & Basic Monitoring

### 📅 Day 3: Advanced Security & DNS Hardening
- Configure SPF, DKIM (OpenDKIM), DMARC Records
- Set up Reverse DNS (PTR) — Why it’s Critical
- Implement MTA-STS & TLS Reporting (TLS-RPT)
- Harden Postfix: Disable VRFY, restrict relay, rate limiting
- Anti-Spam: Configure SpamAssassin + Amavis
- Anti-Virus: ClamAV Integration
- Greylisting with Postgrey
- Test deliverability with GlockApps, Mail-Tester.com

### 📅 Day 4: Webmail, Backup & Automation
- Install Roundcube or RainLoop Webmail
- Configure Nginx/Apache as Reverse Proxy with SSL
- Automate Let’s Encrypt Certificate Renewal
- Backup Strategy: MySQL, Maildir, Configs (BorgBackup/Restic)
- Restore Procedure from Backup
- Log Rotation & Monitoring with Logwatch
- Fail2Ban Setup for Brute Force Protection

### 📅 Day 5: All-in-One Solutions (iRedMail, Mail-in-a-Box)
- Deploy iRedMail (All-in-One: Postfix, Dovecot, Roundcube, SOGo, Fail2Ban, etc.)
- Deploy Mail-in-a-Box (Ubuntu-based, opinionated, easy setup)
- Compare: Pros, Cons, Scalability, Maintenance
- Customize iRedMail: Add Domains, Users, Aliases via Admin Panel
- Migrate from Manual Setup to iRedMail
- Backup & Restore in iRedMail

### 📅 Day 6: Cloud-Hosted Open Source Mail Server
- Deploy on AWS EC2 / DigitalOcean / Linode
- Configure Elastic IP, Security Groups, Firewall
- Use Cloudflare Tunnel for Secure Access (optional)
- Set up S3 for Backup Storage
- Monitor with CloudWatch / NetData
- Automate Deployment with Ansible Playbook
- Cost Calculation & Optimization for Cloud

### 📅 Day 7: Production Readiness, Scaling & Teaching Others
- Load Testing with smtp-source / swaks
- High Availability: Keepalived + Floating IP
- Horizontal Scaling: Multiple MTAs with Load Balancer
- Monitoring: Prometheus + Grafana Dashboards
- Alerting: Slack/Telegram via Alertmanager
- Create Your Own Training Material (Slides, Cheatsheets, Labs)
- Anticipate Trainee Questions & Prepare Answers (Q&A Bank)
- Final Project: Deploy, Secure, Monitor, Document a Full Mail Server

---

## 🛠️ Tools & Technologies Covered

- **MTA**: Postfix, Exim (brief)
- **MDA**: Dovecot
- **Webmail**: Roundcube, RainLoop, SOGo
- **Security**: OpenDKIM, OpenDMARC, SpamAssassin, ClamAV, Fail2Ban
- **DNS**: BIND, Cloudflare, AWS Route53
- **Certificates**: Let’s Encrypt (Certbot), ACME.sh
- **Automation**: Ansible, Bash Scripts
- **Monitoring**: NetData, Prometheus, Grafana, Logwatch
- **Backup**: BorgBackup, Restic, Rsync
- **Cloud**: AWS EC2, DigitalOcean, Linode
- **All-in-One**: iRedMail, Mail-in-a-Box

---

## 📚 Pre-Requisites for Trainees

- Basic Linux Command Line Skills
- Understanding of DNS (A, MX, CNAME, TXT records)
- Familiarity with Networking (Ports, Firewalls, TCP/IP)
- SSH Access & Text Editor (nano/vim)
- Domain Name (for practical labs)

---

## 🎓 Target Audience

- DevOps Engineers
- System Administrators
- Cloud Engineers
- IT Trainers
- Tech Enthusiasts aiming for Production Deployment

---

## 🧑‍🏫 Trainer’s Note (Sumon Paul)

> “This course is born from real production pain points. Every module is battle-tested. I’ve structured it so you not only learn — you become confident enough to teach others and deploy in real enterprises. Anticipate questions before they’re asked. Master the ‘why’ behind every config. That’s the key to professional mastery.”

---

## 📂 Repository Structure (Suggested)

```
mail-server-mastery/
├── README.md
├── day-1-theory/
│   ├── slides.md
│   ├── cheatsheet.md
│   └── qna-bank.md
├── day-2-postfix-dovecot/
│   ├── step-by-step-guide.md
│   ├── config-files/
│   └── test-commands.txt
├── day-3-security/
│   ├── spf-dkim-dmarc-configs.md
│   └── hardening-checklist.md
├── day-4-webmail-backup/
├── day-5-iredmail/
├── day-6-cloud/
├── day-7-production/
└── final-project-template.md
```

---

## 📜 License

This course material is open for educational use. Attribution required.  
© 2025 Sumon Paul — DevOps & Cloud Engineer
```

---

## 🗓️ Day-1: Theoretical Deep Dive & Foundation (Full Content)

### ✅ Module 1: What is a Mail Server?

> **Real-World Analogy**: Think of a Mail Server as a **Digital Post Office**.  
> - **MTA (Mail Transfer Agent)** = Sorting Facility + Delivery Trucks (Postfix, Sendmail)  
> - **MDA (Mail Delivery Agent)** = Mailbox + Inbox Organizer (Dovecot, Cyrus)  
> - **MUA (Mail User Agent)** = Your Letter + Envelope (Thunderbird, Outlook, Webmail)  
> - **MSA (Mail Submission Agent)** = Post Office Counter (Submission Port 587 with Auth)

---

### ✅ Module 2: Email Flow — Step by Step (with Diagram)

```
[User Composes Email in MUA]  
       ↓ (Port 587 w/ TLS & Auth)  
[MSA → MTA (Outgoing Queue)]  
       ↓ (DNS MX Lookup)  
[MTA → Remote MTA via Port 25]  
       ↓ (Spam/Virus Check)  
[MDA Stores in Maildir/mbox]  
       ↓ (Port 143/993 or 110/995)  
[MUA Fetches via IMAP/POP3]
```

> **Real Example**:  
> You send `hello@yourcompany.com` → Gmail  
> 1. Your MUA → Your MSA (port 587, authenticated)  
> 2. Your MTA looks up `gmail.com MX` → finds `aspmx.l.google.com`  
> 3. Your server connects to Google’s MTA on port 25  
> 4. Google checks SPF/DKIM → accepts → delivers to recipient’s MDA  
> 5. Recipient opens Gmail (MUA) → fetches via IMAP

---

### ✅ Module 3: Protocols — Deep Comparison

| Protocol | Port | Use Case | Encryption | Notes |
|----------|------|----------|------------|-------|
| **SMTP** | 25 | Server-to-Server Transfer | Optional (STARTTLS) | **Never use for submission** |
| **SMTP Submission** | 587 | Client-to-Server (MUA→MSA) | **Mandatory TLS** | Requires AUTH (SASL) |
| **SMTPS (Legacy)** | 465 | Encrypted SMTP | SSL/TLS Wrapper | Deprecated, but still used |
| **IMAP** | 143 | Sync Mailbox (Folders, Flags) | STARTTLS | Stateful, server-side storage |
| **IMAPS** | 993 | Encrypted IMAP | SSL/TLS | **Production Standard** |
| **POP3** | 110 | Download & Delete | STARTTLS | Stateless, client-side storage |
| **POP3S** | 995 | Encrypted POP3 | SSL/TLS | Avoid in enterprise |

> **Why 587 over 25?**  
> Port 25 is often blocked by ISPs to prevent spam. Port 587 requires authentication — prevents open relay.

---

### ✅ Module 4: Security Layers — Beyond Passwords

#### 🔐 TLS (Transport Layer Security)
- Encrypts traffic between MUA↔MSA and MTA↔MTA.
- Use Let’s Encrypt: `certbot certonly --standalone -d mail.yourdomain.com`

#### 🔐 SASL (Simple Authentication and Security Layer)
- Authenticates users before allowing email submission.
- Dovecot provides SASL to Postfix: `smtpd_sasl_type = dovecot`

#### 🛡️ SPF (Sender Policy Framework)
- TXT Record: `v=spf1 mx a:mail.yourdomain.com -all`
- Tells receivers: “Only these IPs can send email for my domain.”

#### 🖋️ DKIM (DomainKeys Identified Mail)
- Adds digital signature to headers. Verified via DNS public key.
- Prevents spoofing. Uses `opendkim`.

#### 📊 DMARC (Domain-based Message Authentication, Reporting & Conformance)
- Policy: `v=DMARC1; p=quarantine; rua=mailto:admin@yourdomain.com`
- Tells receivers what to do if SPF/DKIM fail + sends aggregate reports.

#### 🚫 MTA-STS (Mail Transfer Agent Strict Transport Security)
- Enforces TLS between MTAs. Prevents downgrade attacks.
- Requires `.well-known/mta-sts.txt` and DNS TXT `_mta-sts.yourdomain.com`

#### 🔐 DANE (DNS-based Authentication of Named Entities)
- Uses DNSSEC to pin TLS certificates. Very secure, but complex.

#### 🎨 BIMI (Brand Indicators for Message Identification)
- Shows your logo in supported email clients (Gmail, Yahoo).
- Requires DMARC + Verified Mark Certificate (VMC).

---

### ✅ Module 5: Pre-Requisites — Non-Negotiables

#### 1. **FQDN (Fully Qualified Domain Name)**
- Must resolve publicly: `mail.yourdomain.com → 203.0.113.5`
- Avoid using bare domain (e.g., `yourdomain.com`) for mail server.

#### 2. **Reverse DNS (PTR Record)**
- IP `203.0.113.5` → must resolve to `mail.yourdomain.com`
- **Why?** Major providers (Gmail, Outlook) reject mail without valid PTR.

#### 3. **Firewall Rules**
```bash
# Allow:
25/tcp   # SMTP (inbound from other MTAs)
587/tcp  # Submission (MUA → MSA)
993/tcp  # IMAPS
995/tcp  # POP3S
80/tcp   # HTTP (for Let’s Encrypt)
443/tcp  # HTTPS (Webmail)
```

#### 4. **SSL Certificate**
- Must cover: `mail.yourdomain.com`, `yourdomain.com`, `autodiscover.yourdomain.com`
- Use wildcard if managing multiple subdomains.

---

### ✅ Module 6: System Configuration Calculation

> **Formula: Per 1000 Active Users**

| Resource | Low Volume (1-5 emails/day) | Medium (10-20) | High (50+) |
|----------|------------------------------|----------------|------------|
| **RAM**  | 2 GB                         | 4 GB           | 8 GB+      |
| **vCPU** | 1 Core                       | 2 Cores        | 4 Cores    |
| **Disk** | 50 GB (Maildir)              | 100 GB         | 500 GB+    |
| **Bandwidth** | 5 Mbps                 | 10 Mbps        | 50 Mbps    |

> **Maildir vs mbox**:  
> - **Maildir**: One file per email → better performance, no locking → **Recommended**  
> - **mbox**: Single file per mailbox → risk of corruption under load

> **Disk Calculation Example**:  
> Avg email size = 100 KB  
> 1000 users × 50 emails/day × 100 KB = 50,000 KB = **50 MB/day**  
> 30 days = **1.5 GB/month** → Add logs, spam, attachments → **50 GB safe**

---

### ✅ Module 7: Mail Server Types — Beyond Postfix

| Type | Description | Best For | Open Source |
|------|-------------|----------|-------------|
| **Postfix + Dovecot** | Modular, Highly Customizable | Enterprises, Learning | ✅ |
| **iRedMail** | All-in-One (Postfix, Dovecot, Roundcube, SOGo, Fail2Ban) | Quick Production | ✅ |
| **Mail-in-a-Box** | Opinionated, Easy Setup, LetsEncrypt Built-in | Small Biz, Beginners | ✅ |
| **Modoboa** | Modern Admin Panel, Django-based | Mid-size Companies | ✅ |
| **Citadel** | Groupware (Calendar, Contacts) | Teams | ✅ |
| **Zimbra (Open Source)** | Full Collaboration Suite | Large Orgs | ✅ (Network Edition Paid) |

> **Avoid**: Sendmail (complex), Qmail (outdated), Exim (powerful but steep learning)

---

### ✅ Module 8: Anticipated Trainee Questions (Q&A Bank)

**Q1: Can I use a dynamic IP for my mail server?**  
> ❌ No. Dynamic IPs are blacklisted. Use static IP or cloud VPS.

**Q2: Why is my email going to spam?**  
> Check: PTR, SPF, DKIM, DMARC, content (avoid spammy words), IP reputation.

**Q3: Can I host mail server on shared hosting?**  
> ❌ No. Shared IPs, no root access, port 25 blocked.

**Q4: What’s the difference between IMAP and POP3?**  
> IMAP syncs state (read/unread, folders) across devices. POP3 downloads and deletes (usually).

**Q5: How to check if my server is an open relay?**  
```bash
telnet your-server-ip 25
> HELO test.com
> MAIL FROM: <test@test.com>
> RCPT TO: <external@gmail.com>
> DATA
> Test
> .
```
> If it accepts → **you’re an open relay!** Fix with `smtpd_relay_restrictions`.

**Q6: Do I need a separate server for webmail?**  
> Not mandatory. Can run Nginx + PHP on same server. But for scale, separate.

---

## ➡️ Next Steps

Day-2 এ আমরা শুরু করব **Postfix + Dovecot** ম্যানুয়ালি সেটআপ করা — ধাপে ধাপে, কনফিগ ফাইলসহ, টেস্টিং কমান্ডসহ, ট্রাবলশুটিং গাইডসহ।

চাইলে এখনই Day-2 এর পূর্ণ কনটেন্ট চাইতে পারেন — অথবা পরের দিনগুলো ধারাবাহিকভাবে নিতে পারেন।

---

## 🎁 Bonus: Day-1 Cheatsheet (Markdown)

```markdown
## 🚀 Day-1 Cheatsheet: Mail Server Foundation

### 🔑 Core Components
- MTA: Postfix (sends/receives between servers)
- MDA: Dovecot (stores/fetches emails for users)
- MUA: Thunderbird, Outlook, Roundcube
- MSA: Postfix on port 587 (authenticated submission)

### 📬 Protocols & Ports
- SMTP (Server Relay): 25
- Submission (Client Auth): 587 (STARTTLS) or 465 (SMTPS)
- IMAP (Sync): 143 (STARTTLS) or 993 (IMAPS)
- POP3 (Download): 110 or 995

### 🛡️ Security Must-Haves
1. Valid FQDN: `mail.domain.com`
2. PTR Record matching FQDN
3. SPF: `v=spf1 mx -all`
4. DKIM: Generated via `opendkim-genkey`
5. DMARC: `v=DMARC1; p=quarantine; rua=mailto:you@domain.com`
6. TLS Certificate (Let’s Encrypt)
7. Firewall: Allow 25,587,993,995,80,443

### 💻 System Sizing (Per 1000 Users)
- RAM: 4GB
- CPU: 2 Cores
- Disk: 100GB (Maildir)
- Bandwidth: 10 Mbps

### ❓ Top 3 Trainee Questions
1. “Why PTR record?” → Without it, Gmail/Yahoo reject you.
2. “Port 25 vs 587?” → 25 for server-server, 587 for client-auth.
3. “IMAP vs POP3?” → IMAP for multi-device sync, POP3 for single device download.
```

---

## ✅ পরবর্তী পদক্ষেপ

আপনি চাইলে এখনই বলতে পারেন:  
👉 **“Day-2 এর পূর্ণ কনটেন্ট দিন”** — আমি সম্পূর্ণ প্র্যাকটিক্যাল গাইড, কনফিগ ফাইল, স্ক্রিনশট, ট্রাবলশুটিং সহ দিয়ে দেব।

অথবা চাইলে Day-3, Day-4... ধারাবাহিকভাবে নিতে পারেন।

---

> **আপনার ট্রেনিং সফল হোক — আপনার শিক্ষার্থীরা যেন Production-Ready Mail Server ডিপ্লয় করতে পারে, না শুধু থিওরি জানে।**  
> — Sumon Paul, DevOps & Cloud Engineer

--- 

✅ আপনার জন্য প্রস্তুত করা হয়েছে:  
- পূর্ণাঙ্গ কোর্স আউটলাইন (7 Days)  
- Day-1 এর সম্পূর্ণ থিওরিটিক্যাল কনটেন্ট (বাস্তব উদাহরণ, ক্যালকুলেশন, Q&A ব্যাংক সহ)  
- README.md (Markdown)  
- Cheatsheet  

📌 **পরবর্তী রিকোয়েস্ট**: “Day-2 দিন” — আমি সাথে সাথে Postfix + Dovecot সেটআপের পূর্ণ প্র্যাকটিক্যাল গাইড দিয়ে দেব।

--- 

**শুরু করুন আজই — আপনার শিক্ষার্থীরা Production Mail Server নিয়ে Confident হয়ে উঠুক।** 🚀
