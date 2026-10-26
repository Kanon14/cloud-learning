# HCCDA–Tech Essentials — Complete Study Notes

**Huawei Cloud fundamentals, architecture, hands-on labs, and exam review**  
**Study edition:** 26 October 2026  
**Audience:** AI developers, application engineers, solution architects in training, and enterprise/banking technology teams.

> **Status and scope.** These are independently written study notes, **not Huawei's official courseware, leaked exam material, or a guaranteed item-by-item exam syllabus**. They follow the six chapter themes Huawei publishes for HCCDA–Tech Essentials. Huawei may change services, availability, UI, exam rules, and course content. Verify any deployment and certification details against the official documentation for your region.

## How to use this guide

1. Read Chapters 1–6 in order, drawing each architecture before checking the explanation.
2. Complete the exercises in Chapter 7 using an authorized sandbox or company-provided lab account.
3. Explain the design choices in the scenarios in Chapter 8, rather than memorizing product names.
4. Test yourself using Chapter 9, then check the answers in Chapter 10.
5. Use the service matrix, checklist, and official links as quick references.

**Suggested pacing:** 6 weeks, ~5–7 study hours each week. Adjust based on hands-on access.

## Official course-theme alignment

Huawei's published six themes are:

| Chapter | Published theme | This guide |
|---|---|---|
| 1 | Cloud Infrastructure and Computing Capabilities | Cloud models, regions/AZs, ECS, compute choices |
| 2 | Storage and Networking | OBS, EVS, SFS, VPC, subnets, EIP, VPN, ELB |
| 3 | Security and Deployment | Shared responsibility, IAM, firewalls, deployment, encryption |
| 4 | Database and Data Governance | RDS, GaussDB, DDS, data lifecycle and backups |
| 5 | Distributed Deployment and Scalability | HA, AS, ELB, cache, messaging, DR |
| 6 | Cloud Native and Digital Transformation | Docker, CCE/Kubernetes, FunctionGraph, CI/CD, modernization |

Huawei's 2026 developer certification catalog also lists **HCCDA–Tech Essentials** at the associate level in cloud architecture and **HCCDP–Solution Architectures** as a further path.

---

# Chapter 1 — Cloud Infrastructure and Computing Capabilities

## 1.1 What is cloud computing?

Cloud computing provides on-demand compute, networking, storage, databases, and other IT capabilities delivered through a provider platform. Rather than owning all physical servers, an organization rents provisioned resources and can adjust capacity when demand changes.

**Key characteristics**

- **On-demand provisioning:** create/modify resources using a console or API.
- **Elasticity:** add/remove capacity to match demand, subject to limits and design.
- **Metered billing:** usage is measured; a provisioned idle resource may still cost money.
- **Resource pooling:** physical infrastructure supports logically isolated tenants.
- **Automation:** infrastructure can be recreated through APIs or infrastructure-as-code.
- **Global/regional choice:** deployment regions affect latency, sovereignty, service availability, and cost.

### Cloud service models

| Model | You typically manage | Provider typically manages | Example |
|---|---|---|---|
| IaaS | Guest OS, patches, app, data, many networking settings | Physical data center, hardware, virtualization | ECS virtual machine |
| PaaS | App code, data, configuration | Much of runtime/platform operations | Managed application or database service |
| SaaS | Users, data/configuration, organizational access | Application and its platform | Hosted business application |

**Exam trap:** cloud-hosted does not mean fully managed. With an ECS, the customer is still responsible for securing and updating the guest operating system and application.

### Public, private, hybrid, multi-cloud

- **Public cloud:** shared provider platform with tenant separation.
- **Private cloud:** dedicated cloud infrastructure for one organization, whether on-premises or hosted.
- **Hybrid cloud:** integrated on-premises/private environment and public cloud.
- **Multi-cloud:** using services from more than one cloud provider.

A bank can retain selected core systems on-premises while connecting privately to a cloud-hosted document-processing application. That is a hybrid architecture, but the security, legal, and risk requirements depend on the actual deployment.

## 1.2 Regions and availability zones

- **Region:** a geographic location in which Huawei Cloud offers services. Select based on residency requirements, latency, service coverage, connectivity, and cost.
- **Availability Zone (AZ):** a distinct failure domain within a region, intended to help isolate infrastructure faults.
- **Multi-AZ architecture:** places components across AZs to reduce impact of an AZ outage.
- **Multi-region architecture:** used for broader fault tolerance, resilience or geographical requirements; it adds complexity and replication costs.

```text
                   Huawei Cloud Region
             +-----------------------------+
             | AZ-A            AZ-B        |
             |  ECS-1           ECS-2      |
             |    \              /         |
             |          ELB                |
             |      Multi-AZ Database*     |
             +-----------------------------+
                  *where supported/configured
```

**Distinction:** region ≠ AZ ≠ VPC. An AZ is a provider failure domain; a VPC is a user-defined logical network.

## 1.3 Compute choices

### Elastic Cloud Server (ECS)

An ECS is a virtual machine. Choose a flavor (CPU/memory), image (operating system), disks, VPC/subnet, security group, and billing model.

Typical applications: web servers, API backends, enterprise middleware, worker processes, development machines.

**ECS provisioning checklist**

1. Choose region and an appropriate AZ.
2. Select an ECS flavor based on compute/memory requirements.
3. Select a supported image and securely administer credentials/keys.
4. Attach EVS disks, considering capacity and performance.
5. Place ECS in the right VPC and subnet.
6. Apply a restrictive security group; expose only necessary ports.
7. Decide whether a public IP is required (often not for private app servers).
8. Configure backup, monitoring, OS updates, logging, and cost alerts.

### Bare Metal Server (BMS)

Physical-server-style compute for workloads requiring hardware characteristics, specialized performance, or compatibility. Not the default choice for ordinary web APIs.

### Auto Scaling (AS)

Policies can adjust the number of compute instances in response to metrics or schedules. Scaling is **not the same as high availability**; a single-AZ scale group can still share an AZ failure mode.

### GPU/NPU or accelerator resources

AI training and some inference workloads may require specialized compute. Availability, supported frameworks, and pricing vary by region. Avoid selecting accelerators merely because an application uses an LLM API: a client calling an external inference API may run on ordinary CPU compute.

## 1.4 ECS versus containers versus serverless

| Requirement | ECS | CCE (Kubernetes) | FunctionGraph (functions) |
|---|---|---|---|
| Full OS control | Strong | Limited to cluster/node approach | Little |
| Operational overhead | OS and app operations | Cluster, workloads, networking and configuration | Focus on function and integration |
| Best fit | General server applications | Multiple containerized services | Event-driven, short-lived tasks |
| Scaling | Manual/AS | Pod/node scaling based on configuration | Service-specific event/concurrency scaling |

**Architecture principle:** pick the smallest operationally appropriate platform, not the most complicated.

## 1.5 Cost fundamentals

Common cost dimensions include compute instance runtime, block storage, object storage/calls, public egress, load balancers, managed database instances, backups, observability data, and dedicated connectivity.

**Practical cost hygiene:** assign project tags; configure budgets/alerts; stop and delete unused lab resources; verify that stopping a VM may not stop charges for attached storage and other resources.

### Chapter 1 recap

You should be able to: distinguish IaaS/PaaS/SaaS; explain regions/AZs; describe ECS, BMS, AS; and choose a basic compute platform for an application.

---

# Chapter 2 — Storage and Networking

## 2.1 Storage types: object, block, file

| Type | Huawei service | Interface | Typical usage | Important caution |
|---|---|---|---|---|
| Object | OBS | Object API/SDK, buckets/objects | PDFs, images, backups, logs, data lakes | Not a traditional OS disk |
| Block | EVS | Mounted disks/volumes | OS disks, database volumes | Filesystem and backup handling remain important |
| File | SFS | Shared filesystem mount | Shared application files | Choose protocol/performance carefully |

### Object Storage Service (OBS)

A **bucket** holds **objects**. Objects consist of content plus metadata and an object key. Common properties and practices:

- Configure bucket access policies and IAM permissions carefully; private by default is a sound starting posture.
- Prefer temporary scoped credentials or controlled service access rather than permanent embedded access keys.
- Consider server-side encryption, lifecycle policies, versioning, and replication when supported and needed.
- Understand storage classes and retrieval tradeoffs; archival tiers may delay access.
- Document retention ≠ backup ≠ legal hold. Each is a distinct control.

Example key convention: `applications/2026/10/case-123/document.pdf`. This is an object key, not necessarily a real directory.

**Banking example:** uploaded customer files go to a private OBS bucket; app metadata (application ID, owner, file reference, status) goes to the database.

### Elastic Volume Service (EVS)

Use for VM disk storage. Know **system disk vs data disk**, capacity, throughput/IOPS expectations, snapshots, and backups. A disk snapshot provides a point-in-time capture of disk state; do not equate it automatically with a consistent complete application backup.

### Scalable File Service (SFS)

Use when multiple compute resources need shared file access. Ensure mount access, network restrictions, and file locking/consistency fit the workload.

## 2.2 VPC fundamentals

A **Virtual Private Cloud (VPC)** provides logically isolated networking. Its building blocks include subnets, route tables, network interfaces, security groups, and network ACLs.

```text
                    VPC: 10.20.0.0/16
             +-----------------------------+
             | Public-facing tier          |
Internet -->| [Internet-facing ELB]*       |
             |             |               |
             | Private app subnet          |
             |     10.20.10.0/24           |
             |       ECS / CCE             |
             |             |               |
             | Private data subnet         |
             |     10.20.20.0/24           |
             |         RDS                 |
             +-----------------------------+
             *Only if external access is required
```

**CIDR refresher:** `/24` in IPv4 gives 256 addresses in the CIDR block mathematically; cloud providers reserve some addresses, so do not assume all 256 are usable.

- **Subnet:** a network segment within a VPC.
- **Route table:** decides where packets go based on destination networks.
- **Security group:** stateful virtual firewall rules associated with network interfaces/instances; allowed response traffic is tracked.
- **Network ACL (NACL):** subnet-level network filtering; often stateless, so both directions need careful consideration.
- **EIP:** Elastic IP, a publicly reachable IP address that can be associated with eligible resources.

**Security group vs NACL:** security groups are usually the primary instance-level filter; NACLs provide another subnet-layer control. Neither replaces application authentication, TLS, or vulnerability management.

## 2.3 Connectivity patterns

- **VPC peering:** privately connects appropriate VPC networks under defined routing constraints; overlapping CIDRs are a frequent obstacle.
- **Enterprise Router:** hub-style connectivity across multiple VPCs/networks; check attachment and routing design.
- **VPN:** encrypted tunnel over an underlying network, often the internet; suitable for hybrid connectivity depending on requirements.
- **Direct Connect:** dedicated network connectivity between enterprise premises and cloud; not automatically encryption by itself.
- **Cloud Connect:** inter-region and cross-network connectivity options; design depends on service scope.
- **NAT gateway:** allows translation/egress patterns; outbound access does not require making every instance directly public.
- **DNS:** resolves service names; internal versus public DNS names are part of architecture design.

### ELB (Elastic Load Balance)

Directs incoming traffic to healthy backend resources. Concepts include **listener**, **backend group**, **health check**, **protocol**, and **distribution policy**.

**Important:** an ELB improves traffic distribution, but high availability requires correctly designed backends, AZ spread, application state, and dependencies.

## 2.4 Example network request path

1. Client resolves the service's DNS name.
2. Traffic reaches the ingress endpoint (public or private, according to design).
3. ELB applies listener rules and sends traffic to a healthy application instance.
4. The application accesses a private RDS endpoint and/or a private OBS access path if supported and configured.
5. Responses return to the client; logging and monitoring capture permitted telemetry.

A private database does **not** need its own EIP for an application running in the same VPC to access it.

## 2.5 Troubleshooting: “My ECS cannot connect to the database”

Investigate systematically: VPC/subnet and routing → database endpoint/DNS → security group/NACL rules → port/listener → database user privileges/TLS → server process/health → logs. Do not fix access problems by opening all ports to `0.0.0.0/0`.

### Chapter 2 recap

You should be able to choose OBS/EVS/SFS, describe VPC and subnets, distinguish EIP/VPN/Direct Connect, and draw a private three-tier network.

---

# Chapter 3 — Security and Deployment

## 3.1 Shared responsibility

Huawei Cloud operates the physical cloud foundation and the security controls built into its managed services. Customers remain responsible for identity configuration, application vulnerabilities, data classification, permissions, networking policies, and many compliance duties. The boundary differs by service.

**A managed database is not the same as managed customer permissions or compliant data handling.**

## 3.2 Identity and Access Management (IAM)

Know these terms:

- **Identity/user:** principal that accesses cloud resources.
- **User group/role/policy:** constructs for bundling or assigning permissions; exact IAM mechanics depend on the particular Huawei Cloud IAM model.
- **Authentication:** confirming identity (e.g., MFA).
- **Authorization:** determining what actions the identity can perform.
- **Least privilege:** grant only the permissions necessary for the task.

**Good practices:** avoid daily root/primary account use; enable MFA where available; separate dev/test/prod access; use role or temporary access patterns when supported; rotate/revoke secrets; audit privilege changes.

### Example role separation

| Role | Example permission scope |
|---|---|
| App developer | Deploy to development resources, read dev logs |
| Operations engineer | Monitor, troubleshoot, restart approved production services |
| Database administrator | Manage DB schema, backup/recovery within authorized boundaries |
| Security auditor | Read audit logs and approved posture reports |
| Application workload | Read only designated OBS prefix and required DB endpoint |

## 3.3 Defense in depth

Think in multiple layers:

1. **Network:** VPC separation, subnet segmentation, security groups, protected connectivity.
2. **Identity:** IAM, MFA, workload identities, permission review.
3. **Host/container:** patches, hardening, malware controls, vulnerability scans.
4. **Application/API:** authentication, authorization, validation, rate limits, secure coding.
5. **Data:** encryption, retention, minimization, classified access.
6. **Detection/recovery:** monitoring, audit trails, backups, incident response.

### Services to recognize

- **WAF:** Web Application Firewall; filters many web-layer threats when properly configured.
- **DEW/KMS:** Data Encryption Workshop and key-management-related capabilities.
- **CTS:** Cloud Trace Service; audit trail for cloud resource operations.
- **Cloud Eye (CES):** metric monitoring and alarms.
- **CBR:** Cloud Backup and Recovery for supported resource types.
- **LTS:** Log Tank Service for collected logs.

Do not confuse **CTS control-plane audit events** with **application business audit logs**. A banking solution may need both.

## 3.4 Encryption and secrets

- **Encryption in transit:** prefer TLS for API and data connections.
- **Encryption at rest:** encrypt data stored on disks, objects, backups, or managed databases as required.
- **Key management:** determine who controls and rotates keys, and what happens when a key is revoked.
- **Secret management:** do not place passwords/API keys in code, images, tickets, or diagrams.
- **Data minimization:** store only what the workflow requires.

## 3.5 Deployment strategies

- **Manual deployment:** useful for first labs, error-prone for repeated production releases.
- **Scripted or Infrastructure-as-Code (IaC):** reproducible resource provisioning; review code and drift.
- **Rolling deployment:** gradually replace instances.
- **Blue-green deployment:** maintain two environments and switch traffic; may increase cost.
- **Canary deployment:** expose a small traffic portion first; requires monitoring and rollback.

**Rollback plan:** define previous artifact, compatibility with database migrations, traffic-switch mechanism, owners, and acceptance thresholds before deployment.

## 3.6 Enterprise and banking considerations

A banking solution may require customer-approved region, private connectivity, audit retention, maker-checker control, access approval, incident reporting, vendor risk assessment, recovery testing, and documented data flows. **Do not assume that selecting a cloud region or using encryption alone establishes regulatory compliance.** Exact requirements depend on bank policy, local regulation, contractual obligations, and workload risk.

### Chapter 3 recap

Be ready to explain least privilege, defense in depth, IAM versus network security, encryption, audit trails, and safe deployment/rollback.

---

# Chapter 4 — Database and Data Governance

## 4.1 Database fundamentals

A database provides durable structured or semi-structured data management. Selection depends on query patterns, consistency, schema, scaling, availability, operational complexity, and budget.

- **Relational / SQL:** tables, schemas, transactions, joins; suitable for application records and workflow state.
- **Document database:** flexible JSON-like documents; useful when access patterns fit document data.
- **Cache / key-value:** fast access to transient/hot data; usually not authoritative system of record.
- **Analytical warehouse:** optimized for reporting/analytics rather than individual OLTP transactions.

## 4.2 Huawei Cloud database-related services

| Service | Main purpose | Example |
|---|---|---|
| RDS | Managed relational databases | Loan application metadata and status |
| GaussDB | Huawei enterprise database offerings | Transactional enterprise workloads where appropriate |
| DDS | Managed document database | Document-like application records |
| DCS | Distributed Cache Service | Sessions, rate-limit counters, cached lookups |
| DRS | Data Replication Service | Migration/replication scenarios |
| DWS | Data Warehouse Service | Reporting/analytical workloads |

**Important:** available engines, editions, versions, deployment modes, and features vary by service and region. Always check current product documentation before committing to an architecture.

## 4.3 Transactions and integrity

ACID properties:

- **Atomicity:** all-or-nothing transaction.
- **Consistency:** transactions preserve database rules/constraints.
- **Isolation:** concurrently executing transactions do not break defined isolation guarantees.
- **Durability:** committed changes survive failures according to the system's guarantees.

**Banking example:** creating a loan decision and its audit linkage may require transactional consistency; a cache is not a substitute for durable workflow state.

## 4.4 Database availability, backups, and DR

Distinguish:

- **HA (high availability):** minimize disruption through redundancy/failover.
- **Backup:** restore data from saved copies after deletion/corruption/other failures.
- **Replication:** copy data to another location/instance; incorrect deletion can replicate too.
- **Disaster recovery (DR):** ability to restore operations after a serious site or region failure.

Define **RPO (Recovery Point Objective)** as the maximum acceptable data loss measured in time, and **RTO (Recovery Time Objective)** as the target maximum recovery time.

Example: an RPO of 15 minutes and RTO of 2 hours means designing/testing for at most ~15 minutes of lost data and restoring service in ~2 hours under the relevant failure scenario; these are objectives, not automatic guarantees.

## 4.5 Data governance lifecycle

```text
Capture -> Classify -> Validate -> Store -> Use/Share -> Retain -> Archive/Delete
                        |             |
                      Audit        Access control
```

Document:

- **Ownership and stewardship:** who is accountable for accuracy/access.
- **Data classification:** public, internal, confidential, restricted (labels vary by organization).
- **Lineage:** where data originated and how it was transformed.
- **Quality:** completeness, accuracy, timeliness, consistency.
- **Retention:** reason, lawful basis/policy, duration, archive and deletion controls.
- **Auditability:** who accessed or changed the records and when.

A 7-year retention requirement, if imposed by a particular organization or regulation, must be confirmed for the specific data type and jurisdiction; do not apply it globally by assumption.

## 4.6 Basic data architecture

```text
Dealer Portal / API
       |
       v
Application Service --------> RDS (case records, status)
       |
       +---------------------> OBS (uploaded documents)
       |
       +---------------------> DCS (optional temporary cache)
       |
       +---------------------> DWS (approved reporting pipeline)
```

Store object references in the database. Protect both the object and its reference metadata; signed links, if used, should have a bounded lifetime and scope.

### Chapter 4 recap

Understand relational vs document vs cache vs warehouse, transaction guarantees, RPO/RTO, backups/replication, and the data lifecycle.

---

# Chapter 5 — Distributed Deployment and Scalability

## 5.1 Why distributed applications?

One server may not provide adequate throughput, resilience, or deployment flexibility. Distributed architecture splits work across multiple nodes/services. Benefits bring costs: network latency, partial failures, operational complexity, consistency challenges, and observability needs.

## 5.2 Scale up versus scale out

- **Vertical scaling (scale up):** more CPU/RAM for one instance; constrained by max size and may need downtime.
- **Horizontal scaling (scale out):** add more service instances; requires load distribution and safe state management.

**Stateless application design** helps horizontal scaling: keep durable shared state in a database or another appropriate store, rather than only in one instance's memory.

## 5.3 Availability design

### Single point of failure (SPOF)

Any one resource whose loss breaks the entire application can be a SPOF. Common examples: one backend, one database instance, single network connection, single AZ placement, or unavailable authentication dependency.

### Multi-AZ pattern

```text
             Requests
                |
               ELB
             /     \
          AZ-A      AZ-B
          App-1     App-2
             \       /
          Managed DB HA*
                |
             Backup

   *If supported and configured for the service/region
```

**Failure walkthrough:** if App-1 fails, health checks should remove it from routing, App-2 should continue processing if capacity and state permit. If the DB fails, the application may still be unavailable without a viable DB failover path.

## 5.4 Auto Scaling concepts

- **Scaling group:** managed set of instances.
- **Scaling policy:** conditions that cause increase/decrease.
- **Metric-based:** e.g., average CPU crossing a threshold.
- **Scheduled:** increase before expected traffic spikes.
- **Cooldown/stabilization:** avoids rapid oscillation.

Set safe minimum and maximum instance counts, measure startup time, and test health checks before load-testing a scaling policy.

## 5.5 Caching and asynchronous processing

**Cache (DCS):** reduce repeated DB queries; plan TTL, invalidation, capacity, and failure behavior. Never assume stale cached decisions are safe for financial approvals.

**Message queues/event processing:** decouple ingestion from slow processing. For document processing:

```text
Upload document -> Persist job record -> Queue -> Worker/OCR
                                     -> Result store -> Status update
```

Important concepts: at-least-once delivery, duplicate processing, idempotency, retry policy, dead-letter queue, poison messages, ordering and backpressure. Do not assume every queue provides exactly-once business execution.

**Idempotency:** repeated requests with the same operation identifier do not create duplicate irreversible results.

## 5.6 Monitoring and reliability

Monitor four broad signal groups:

- **Latency:** how long requests take.
- **Traffic:** requests per second, tasks per minute.
- **Errors:** failed requests/jobs, percentage, reasons.
- **Saturation:** CPU, memory, storage, pool utilization, queue depth.

Useful metrics for an AI document service: pages processed, extraction error rate, model invocation time, queue age, API timeouts, duplicate job count, compute cost per document, and escalation volume.

## 5.7 Disaster recovery patterns

| Pattern | Tradeoff |
|---|---|
| Backups and restore | Lower standby cost, longer recovery time |
| Pilot light | Core recovery foundation kept ready, some resources started during failover |
| Warm standby | Smaller always-on secondary environment, faster recovery, higher cost |
| Active-active | Multiple serving locations, highest complexity and consistency concerns |

Architect to measured business RPO/RTO, then test recovery. Multi-AZ is not automatically multi-region disaster recovery.

### Chapter 5 recap

Explain scale up/out, stateless services, AS+ELB, SPOFs, caching, asynchronous workers, idempotency, and HA/DR tradeoffs.

---

# Chapter 6 — Cloud Native and Digital Transformation

## 6.1 What is cloud native?

Cloud-native design uses automation, containers, declarative orchestration, resilient services, and rapid delivery practices to exploit cloud platforms. It is an approach to building and operating systems, not merely running a Docker container in the cloud.

## 6.2 Containers and Docker

An **image** is a packaged application/runtime definition; a **container** is a running isolated process created from the image. Containers share the host kernel rather than each being a full virtual machine.

Example conceptual container pipeline:

```text
Developer -> Git repo -> CI checks -> Build image
                                 -> Registry
                                 -> Deploy -> ECS/CCE
```

**Security practice:** pin supported image versions; scan for vulnerabilities; run as non-root when possible; keep secrets outside images; set memory/CPU limits.

## 6.3 CCE and Kubernetes essentials

Huawei Cloud **Cloud Container Engine (CCE)** is a managed Kubernetes cluster service. It integrates with Huawei compute, network, and storage services.

Know these Kubernetes objects:

| Object | Role |
|---|---|
| Cluster | Kubernetes control plane plus worker capacity |
| Node | Machine providing compute resources |
| Pod | Smallest Kubernetes deployable unit; one or more containers |
| Deployment | Manages replicated Pods and rollout updates |
| Service | Stable endpoint to reach changing Pods |
| Ingress | HTTP(S) traffic-routing mechanism where configured |
| ConfigMap | Non-secret configuration |
| Secret | Secret-bearing configuration; still needs access controls/encryption handling |
| HPA | Horizontal Pod Autoscaler for supported metrics |

**ECS vs CCE:** start with ECS when one application needs simple deployment. CCE becomes attractive when there are many services, repeatable deployment requirements, orchestrated scaling, and a team able to operate the platform securely.

## 6.4 Serverless / FunctionGraph

Functions can respond to events without managing the full VM lifecycle. Good fit: event-driven document metadata processing, scheduled housekeeping, simple integrations. Consider execution time limits, cold starts, permissions, network access, and unpredictable load patterns.

## 6.5 Microservices versus monoliths

- **Monolith:** simpler deployment and consistency boundaries; good for many small-to-medium systems.
- **Microservices:** independently deployable components and team ownership; require robust API design, networking, distributed tracing, fault tolerance, and data boundaries.

**Not a rule:** every enterprise or AI system must use microservices. Start with modularity and operational needs.

## 6.6 CI/CD and infrastructure as code

A secure delivery pipeline might perform: code checkout → lint/tests → dependency scanning → build → image scan → approval → deploy to test → UAT → approved production release → health validation → rollback if needed.

IaC stores resource definitions in version control and enables review and reproducibility. Always protect state/secrets and constrain deployment permissions.

## 6.7 Digital transformation from an architecture viewpoint

Modernization is not only cloud migration; it includes process re-design, integrations, automation, governance, operational metrics, and user experience.

For a loan origination process:

- Paper submissions become secure digital uploads.
- OCR/IDP extracts information, but humans review exceptions.
- Workflow engines coordinate decision steps.
- APIs integrate approved credit and verification systems.
- Audit trails support traceability.
- Dashboards identify processing bottlenecks.

### Chapter 6 recap

Explain image/container/Pod, Kubernetes basics, CCE versus ECS, serverless, CI/CD, and when cloud-native patterns add value.

---

# Chapter 7 — Hands-on lab workbook

> **Safety and cost:** These are guided exercises, not commands to run in a production bank environment. Use an authorized sandbox, verify availability/pricing, and delete test resources afterward. Never upload real customer data to practice environments.

## Lab 1 — Explore cloud console and IAM (45–60 min)

**Goal:** identify regions, projects, IAM identities and resource inventory.

1. Sign in to a permitted Huawei Cloud account.
2. Identify the active region/project and where to change them.
3. Find ECS, VPC, OBS, IAM, RDS, and Cloud Eye in the console.
4. With authorization, create a narrowly scoped development identity/group or inspect a sample policy.
5. Record how MFA and audit logging are configured (do not publish account identifiers or secrets).

**Deliverable:** one-page diagram of account, region, VPC and IAM relationships. **Verification:** explain why application developers should not share one admin login.

## Lab 2 — Build a VPC and private subnet (60–90 min)

**Goal:** create an isolated network and understand routing/firewall rules.

1. Plan a non-overlapping CIDR, e.g., `10.20.0.0/16`.
2. Define app subnet `10.20.10.0/24` and data subnet `10.20.20.0/24`.
3. Create VPC/subnets if account privileges and quotas allow.
4. Create a security group permitting only explicitly necessary app traffic.
5. Examine route tables and network ACL behavior.
6. Record which component, if any, would legitimately require public internet access.

**Verification:** explain why RDS should not have broadly open inbound public access.

## Lab 3 — Host a simple API on ECS (90–120 min)

**Goal:** deploy a test web service on a Linux ECS.

1. Choose the lowest suitable lab instance flavor.
2. Create ECS in your designated app subnet and attach the needed disk.
3. Use approved secure administration (SSH keys, bastion, or equivalent).
4. Patch the OS and install Python in an isolated environment.
5. Create a simple FastAPI `/health` endpoint returning `{"status": "ok"}`.
6. Run it behind a production-appropriate server/reverse proxy for testing.
7. Apply restrictive security group rules; prefer private endpoint/VPN/bastion access.
8. Review Cloud Eye metrics and OS logs.

**Verification:** access the health endpoint from an approved client; document networking path and resource costs.

## Lab 4 — Upload a sample document to OBS (45–60 min)

1. Create a private lab bucket in the correct region.
2. Upload a **synthetic** PDF or text document.
3. Inspect object key, metadata, encryption options, and access policy.
4. Test access with only your authorized identity.
5. Configure and explain an appropriate lifecycle policy without prematurely deleting needed files.

**Verification:** document what happens if an anonymous client attempts to access the object.

## Lab 5 — Connect an app to managed data storage (90–120 min)

1. Select a small approved RDS instance if lab credits permit.
2. Put it in a private network with narrow access rules.
3. Create a sample `applications` table with non-sensitive records.
4. Configure the API's database connection through secure application configuration.
5. Test insert/query and a simulated transient connection failure.
6. Inspect backup settings and explain the difference between backup and replication.

**Verification:** no hard-coded production credentials; no public unrestricted DB ports.

## Lab 6 — Design for high availability (paper/lab; 60 min)

Given two app instances in different AZs, an ELB, and an HA database, draw dependencies and failure paths. Identify where health checks, backup recovery, monitoring, and failover tests are required.

**Verification:** explain what still fails if both application instances share a single unavailable DB endpoint with no viable failover.

## Lab 7 — Cloud-native container deployment (optional, 90–150 min)

Containerize the lab API; deploy on a test ECS or a CCE sandbox if available. Define health probes, resource limits, secrets handling, and rollback. CCE may incur substantial compute and cluster-related charges, so a single ECS container lab is an acceptable starter alternative.

## Lab cleanup checklist

Delete or shut down appropriately: compute instances, cluster resources, load balancers, EIPs/bandwidth, managed DB instances, snapshots/backups, volumes, test buckets and objects, gateways, and log-retention settings. **Check the billing console afterward**: deleting one parent resource does not always delete everything chargeable.

---

# Chapter 8 — Architecture case studies

## Case A — Secure AI Document Processing (conceptual, not customer-specific)

**Requirement:** enterprise users upload PDFs; the system extracts fields asynchronously, stores documents and case metadata, supports human verification, and records traceable decisions.

```text
  Enterprise Users
        |
  Approved Network Access (VPN / private link / protected ingress)
        |
  +---------------------- HUAWEI CLOUD VPC ------------------------+
  |                                                                |
  |  Ingress / ELB -> Application API (ECS or CCE)                  |
  |                         |                                      |
  |            +------------+-------------+                        |
  |            |            |             |                        |
  |            v            v             v                        |
  |       RDS Cases     Private OBS    Async Job Queue*             |
  |                                      |                          |
  |                                  Worker(s)                     |
  |                                      |                          |
  |                            OCR / AI Inference*                 |
  |                                      |                          |
  |                                RDS Results                     |
  |                                                                |
  +----------------------------------------------------------------+
  IAM + encryption + monitoring + logs + backups + approved access
  *Actual queue/inference service selected per availability and needs
```

**Architecture questions and model reasoning:**

- **Why OBS?** Object storage is appropriate for large PDFs; relational DB stores case references and transactional metadata.
- **Why asynchronous processing?** OCR/model execution is slower and can be retried without forcing the upload request to stay open.
- **Why private network components?** Minimize accidental exposure of sensitive data stores.
- **Where does human approval occur?** Application/workflow layer; AI extraction output alone is not an authorization decision.
- **What about recoverability?** Set explicit RPO/RTO, backups, queue recovery, and end-to-end restore tests.
- **What needs extra verification?** Current service coverage in the selected region, security assessment, model data handling, and applicable banking policy.

## Case B — Loan Origination System (LOS)

**Logical layers:**

1. **Channel:** dealer portal and internal reviewer portal.
2. **Application:** application state machine, validation, product configuration, maker-checker steps.
3. **Integration:** consent/identity, credit bureau, messaging, external finance services.
4. **Data:** case DB, document objects, immutable/controlled audit records as required.
5. **Operations/security:** identity, monitoring, backups, release approvals, incident response.

**Design lesson:** the architecture must support the **business process**; cloud services alone do not enforce maker-checker or a valid loan scorecard.

## Case C — RPA worker platform

An orchestrator/scheduler initiates automation jobs, dedicated worker instances run approved tasks, a database records job state, and monitoring detects failures. Sensitive credentials should be centrally controlled; screen-based automation may require session management, network access to target systems, and recovery logic. Not every legacy application supports clean API integration.

## Architecture decision record (ADR) template

```markdown
# ADR-001: Application compute selection
Status: Proposed
Context: Expected traffic, team capabilities, uptime, data classification
Options: ECS / CCE / Functions
Decision: <chosen option>
Reasons: <why it fits>
Tradeoffs: <complexity, cost, scaling, security>
Failure modes: <what could break>
Security requirements: <network, IAM, secrets, audit>
Cost assumptions: <instance sizes, usage, egress>
Review date / owner: <date and accountable person>
```

---

# Chapter 9 — Practice questions (original, not official exam questions)

**Instructions:** Attempt these without looking at Chapter 10. Some ask for explanations, not just definitions.

1. What is the main responsibility difference between ECS and a managed database service?
2. How are a region, an AZ, and a VPC different?
3. A Python API needs full VM control. Which Huawei Cloud compute service is the simplest initial fit?
4. Which Huawei service is best suited to store 500,000 scanned PDF documents?
5. Which Huawei service provides block volumes for ECS disks?
6. Why is putting a customer database directly on a public IP usually undesirable?
7. What does CIDR `10.20.10.0/24` describe?
8. What is the main difference between an ECS security group and a subnet network ACL?
9. What is an EIP used for?
10. How does a VPN differ from Direct Connect?
11. What is the purpose of ELB health checks?
12. A cloud user can view billing but must not delete resources. What security principle applies?
13. Name the two broad types of audit/monitoring information supported by CTS and Cloud Eye respectively.
14. Does encryption at rest prevent an over-privileged application account from reading data? Explain.
15. Why does automatic database replication not replace backups?
16. Compare an RPO of 10 minutes with an RTO of 1 hour.
17. What is the main purpose of a distributed cache such as DCS?
18. If a web app stores user sessions only in local process memory, what may happen after scaling out?
19. Why should queue workers be idempotent?
20. Explain vertical versus horizontal scaling.
21. Is running two application servers in a single AZ sufficient for AZ-level fault tolerance?
22. Identify three possible single points of failure in a simple three-tier app.
23. What is the relationship between a container image, a container, and a Kubernetes Pod?
24. When might CCE be a better fit than one ECS?
25. What is a suitable use case for FunctionGraph?
26. Why should secrets not be embedded in Docker images?
27. What does a blue-green deployment try to simplify?
28. A PDF upload triggers OCR that takes 90 seconds. Why might an asynchronous job architecture be better than one blocking HTTP request?
29. Name at least four items to confirm before hosting confidential bank documents in a cloud region.
30. After deleting your demo ECS, why might the bill still increase?

---

# Chapter 10 — Answers and explanations

1. **ECS:** customer manages guest OS/app and much of its security. **Managed DB:** provider handles more DB platform operations; customer still manages data, access, configurations, and workload controls.
2. **Region:** geography; **AZ:** infrastructure failure domain inside a region; **VPC:** logical network.
3. **ECS.** It provides a general-purpose VM.
4. **OBS.** Scales object storage for PDFs and similar files.
5. **EVS.** Block storage for attached VM volumes.
6. It expands attack surface and complicates access governance; prefer private endpoints and approved connectivity.
7. IPv4 block with a 24-bit network prefix (256 total addresses, fewer potentially usable in a cloud subnet).
8. Security groups are instance/interface-level and stateful; NACLs filter subnet traffic and commonly require explicit inbound/outbound allowance.
9. Assign a public IP for eligible cloud resources requiring internet-facing connectivity.
10. VPN builds an encrypted tunnel; Direct Connect provides dedicated connectivity but does not inherently guarantee encryption.
11. Detect unhealthy backends so ELB can avoid routing new requests to them according to configured policies.
12. **Least privilege.** Grant read-only billing access, not broad administrator rights.
13. **CTS:** cloud operation/audit trail. **Cloud Eye:** infrastructure/service metrics and alarms.
14. **No.** Encryption at rest does not compensate for excessive logical permissions; decrypted data can still be accessed by authorized-but-overprivileged principals.
15. Deletions/corruption may replicate; backups provide recoverable points in time.
16. RPO is acceptable time-based data loss (10 minutes); RTO is targeted time to restore service (1 hour).
17. Improve response times/reduce repeated calls to slower systems for suitable cacheable data.
18. Requests may reach different instances with no common session state; use a sound shared-session or stateless-token pattern.
19. Queue messages may be retried or delivered more than once; idempotency prevents duplicate business effects.
20. **Vertical:** bigger instance; **horizontal:** more instances.
21. **No.** One AZ still presents a shared AZ failure domain.
22. Examples: one ELB/ingress dependency, one app instance, single database instance, one network link, single identity provider dependency.
23. **Image:** package template. **Container:** running instance of that image. **Pod:** Kubernetes scheduling unit containing one or more containers.
24. When operating multiple containers/services with orchestration, rolling deployments, service discovery, and independent scaling is warranted and team skills support it.
25. Event-triggered or scheduled lightweight background processing.
26. Images can be copied, scanned, cached and shared; embedded secrets may persist long after deployment.
27. Switching between two environments and restoring the prior one if a release fails.
28. A queue decouples upload from slow processing; workers can retry and clients can check status without waiting on a long-lived request.
29. Examples: data residency and contract/regulatory approvals; IAM/access; private network design; encryption/key ownership; audit logs; retention/deletion; backups/DR; supported regional service availability; third-party/model data handling.
30. Other billable items might remain: EVS volumes, snapshots, EIPs/bandwidth, ELB, DB, log ingestion, storage, or backup.

---

# Chapter 11 — Quick-reference service map

| Need | Common Huawei service(s) | Remember |
|---|---|---|
| VM compute | ECS | Guest OS, security and lifecycle responsibilities |
| Physical-server-like compute | BMS | Specialized workload needs |
| Private networks | VPC | Subnets, CIDR, routes, ACLs, security groups |
| Public IP | EIP | Minimize exposed endpoints |
| Load balancing | ELB | Listeners, backend groups, health checks |
| Scale VM fleets | AS | Policies, capacity, cooldown |
| VM disks | EVS | Block storage and backups |
| Files/objects | OBS | Buckets, keys, lifecycle, access |
| Shared filesystem | SFS | Shared file mounts |
| Managed relational DB | RDS | Patching/HA capabilities depend on offering |
| Enterprise database | GaussDB | Check exact product variant |
| Document database | DDS | Document-oriented data |
| Cache | DCS | TTL/invalidation |
| Data warehouse | DWS | Analytics/BI |
| Replication/migration | DRS | Data movement and migrations |
| Access management | IAM | Least privilege and MFA |
| Audit of cloud operations | CTS | Control-plane event records |
| Metrics and alarms | Cloud Eye (CES) | Monitoring and alerting |
| Log collection | LTS | Application/platform logs |
| Backup/recovery | CBR | Restore planning and tests |
| Encryption/key controls | DEW/KMS | Key ownership and policies |
| Web protection | WAF | Web-layer defense |
| Kubernetes | CCE | Cluster, nodes, Pods, Services |
| Event-driven compute | FunctionGraph | Short-lived/event tasks |
| AI development/inference | ModelArts / appropriate AI APIs | Verify service and region fit |

## Common exam-style confusion pairs

| Do not confuse | Distinction |
|---|---|
| OBS vs EVS | Objects/API versus VM block volumes |
| EVS vs SFS | Block disk versus shared filesystem |
| VPC vs subnet | Whole logical network versus segment |
| Security group vs IAM | Packet-level rules versus cloud identity authorization |
| IAM vs application login | Cloud resource permissions versus business-app user authentication |
| CTS vs Cloud Eye | Cloud operation audit versus performance/health monitoring |
| HA vs backup | Continuous availability versus restore from copies |
| RPO vs RTO | Acceptable data loss window versus recovery time target |
| ELB vs AS | Traffic distribution versus capacity changes |
| ECS vs CCE | VM compute versus Kubernetes orchestration |
| CCE vs Docker | Orchestration platform versus container tooling |
| VPN vs Direct Connect | Encrypted tunnel versus dedicated network connection |
| Cache vs system of record | Fast transient copy versus authoritative durable business state |

---

# Chapter 12 — Exam and architecture readiness checklist

Tick when you can explain the item **without looking at the notes**:

- [ ] I can distinguish IaaS, PaaS, and SaaS.
- [ ] I can explain regions, AZs, VPCs, and subnets.
- [ ] I can choose ECS versus CCE versus event functions for a basic scenario.
- [ ] I can select OBS/EVS/SFS correctly.
- [ ] I can draw a secure 3-tier app and explain data flow.
- [ ] I understand ELB, AS, EIP, VPN, and Direct Connect.
- [ ] I can explain IAM, least privilege, security groups, and encryption.
- [ ] I can distinguish Cloud Eye, CTS, LTS, and CBR.
- [ ] I can explain relational DBs, document DBs, caches, and warehouses.
- [ ] I can explain transactions, backups, replication, RPO, and RTO.
- [ ] I can describe an HA deployment across multiple AZs.
- [ ] I can describe asynchronous processing and idempotency.
- [ ] I know image/container/Pod/Deployment/Service distinctions.
- [ ] I can reason about basic cloud cost and clean up lab resources.
- [ ] I can explain an AI document processing architecture end-to-end.

## 6-week study schedule

| Week | Focus | Required output |
|---|---|---|
| 1 | Chapter 1 and Lab 1 | Region/AZ/compute decision notes |
| 2 | Chapter 2 and Labs 2–3 | VPC diagram and simple ECS API |
| 3 | Chapter 3 and Lab 4 | IAM/security checklist and private OBS demo |
| 4 | Chapter 4 and Lab 5 | Simple application data model, backup/RPO/RTO notes |
| 5 | Chapter 5 and Lab 6 | HA/DR diagram and failure analysis |
| 6 | Chapter 6 and optional Lab 7 | Final reference architecture and 30-question practice test |

## Moving beyond HCCDA

After completing these foundations, begin Huawei Cloud **HCCDP–Solution Architectures**. At that stage, add architecture design review, disaster recovery calculations, cost models, deployment environments, cloud governance, observability, integration patterns, and regulatory controls.

---

# Official references and further reading

These links are provided so you can check current vendor guidance and course details. They do not imply that the entire guide reproduces Huawei's proprietary materials.

1. **Huawei Cloud Developer Certification Index:** https://edu.huaweicloud.com/intl/en-us/certificationindex/developercertificationindex.html
2. **Huawei Cloud HCCDA Learning Zone:** https://edu.huaweicloud.com/intl/en-us/dga-learningzone.html
3. **Huawei-published six-chapter Tech Essentials description (course event):** https://developer.huaweicloud.com/intl/en-us/activity/acecdaf2a4db400290f1ef7dfd80d36b
4. **Huawei Cloud Support / documentation hub:** https://support.huaweicloud.com/intl/en-us/
5. **ECS overview:** https://support.huaweicloud.com/intl/en-us/productdesc-ecs/ecs-productdesc-pdf.pdf
6. **VPC overview:** https://support.huaweicloud.com/intl/en-us/productdesc-vpc/vpc-productdesc-pdf.pdf
7. **CCE introduction:** https://support.huaweicloud.com/intl/en-us/productdesc-cce/cce_productdesc_0001.html
8. **Huawei Cloud solution architecture learning path:** https://edu.huaweicloud.com/intl/en-us/roadmap/solutionarchitectures.html

**Final note:** This is a self-contained foundation and revision guide aligned to publicly listed chapter themes, not a substitute for the current official course labs or a verified exam blueprint. Complete the vendor course in parallel and use current service documentation for deployment details.
