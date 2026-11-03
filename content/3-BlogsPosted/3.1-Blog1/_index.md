---
title: "Blog 1"
date: 2026-09-17
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# ☁️ I Tried to "Break" an AWS Architecture by Removing Each Service

## 📝 Introduction

When I first started learning AWS, I tended to memorize each service like this:

**EC2 = Compute**</br>

**RDS = Database**</br>

**ALB = Load Balancing**</br>

**CloudWatch = Monitoring**</br>

But the more I studied AWS, the more I realized that simply memorizing service definitions was not enough.

The question I started asking was:

> **If all of these services are part of the same system, why does each service need to exist?**

So I tried a different learning approach:

> **What happens if I remove each service from the architecture?**

## 🏗️ Architecture / Context

**Route 53 → CloudFront → AWS WAF → VPC → ALB → Auto Scaling EC2 → RDS Multi-AZ**

In addition, **IAM, CloudWatch, Internet Gateway, and NAT Gateway** are used to handle access control, observability, and network connectivity.

![Architecture Diagram](/fcj-workshop-nhat/images/archnew.png)

I use this as a **production-oriented architecture** for learning: it is designed to represent a system that can handle real traffic, improve availability, and apply multiple layers of security instead of simply placing as many AWS services as possible on one diagram.

Now, let's start removing each layer.

---

## 🔍 Experiment 1: 🌐 What Happens If I Remove the Internet Gateway?

Before discussing CloudFront or the Application Load Balancer, I need to understand the Internet Gateway (IGW), because it is one of the most fundamental networking components in a VPC architecture.

### What does an Internet Gateway do?

An Internet Gateway is a horizontally scaled, redundant, and highly available VPC component that provides a path between a VPC and the Internet.

A subnet is considered public when its route table contains a route to an Internet Gateway. However, an Internet Gateway alone does not make a subnet public: the resource also needs an appropriate public IPv4 or IPv6 address and routing configuration to communicate with the Internet.

In this architecture, the internet-facing ALB is deployed in public subnets. CloudFront can use that ALB as its origin.

A simplified application request path is:

```text
User
  ↓
CloudFront
  ↓
AWS WAF Web ACL
  ↓
Internet-facing ALB
  ↓
EC2 in Private Subnet
```

The outbound path for a private EC2 instance is different:

```text
EC2 in Private Subnet
  ↓
NAT Gateway
  ↓
Internet Gateway
  ↓
Internet
```

### What happens if I remove the IGW?

If the public subnets no longer have a working route to an Internet Gateway, the internet-facing ALB loses its Internet connectivity.

That means CloudFront cannot successfully reach the ALB origin through its public path.

At the same time, a public NAT Gateway also depends on Internet Gateway connectivity for Internet-bound traffic from private subnets.

The result is that both the public application entry path and private-subnet outbound Internet connectivity are affected.

### Verdict

The Internet Gateway is a fundamental networking component when the architecture requires Internet connectivity for public VPC resources.

> **"An Internet Gateway is simple, but without the correct routes and addressing, a public subnet is not actually public."**

---

## 🔍 Experiment 2: 🚪 What Happens If I Remove the NAT Gateway?

### What does a NAT Gateway do, and why is it in a Public Subnet?

EC2 instances in a Private Subnet normally do not have public IP addresses. This is a common security design because backend instances do not need to accept unsolicited inbound connections from the Internet.

However, private EC2 instances may still need to initiate outbound connections for tasks such as:

- Updating OS packages (`yum`, `dnf`, `apt-get`)
- Calling external APIs
- Pulling container images from public registries
- Accessing public service endpoints when a suitable VPC endpoint is not available

A public NAT Gateway is deployed in a **Public Subnet** and uses an Elastic IP address.

The private subnet route table sends Internet-bound traffic to the NAT Gateway, and the NAT Gateway sends that traffic toward the Internet Gateway.

The simplified path is:

```text
EC2 in Private Subnet
  ↓
NAT Gateway
  ↓
Internet Gateway
  ↓
Internet
```

### Why must the NAT Gateway be in a Public Subnet?

A public NAT Gateway needs a route to the Internet Gateway.

A typical public subnet route is:

```text
0.0.0.0/0 → Internet Gateway
```

The private subnet uses a different route:

```text
0.0.0.0/0 → NAT Gateway
```

This separation allows private workloads to initiate outbound connections without giving those workloads public IP addresses.

### What happens if I remove the NAT Gateway?

The application can still receive inbound traffic through the ALB, and EC2 instances can still communicate with resources that are reachable through the VPC's internal networking.

However, EC2 instances in the Private Subnet lose their normal outbound Internet path.

That can affect:

- OS package updates
- External API integrations
- Pulling public container images
- Accessing public endpoints without a VPC endpoint alternative

The important distinction is:

> **Inbound application traffic can continue to work while outbound Internet connectivity from private resources fails.**

### "Why not just put EC2 in a Public Subnet?"

Putting backend EC2 instances in a Public Subnet increases the attack surface because the instances may have public IP addresses and could potentially be reached directly from the Internet.

Keeping EC2 in a Private Subnet gives the architecture a clearer separation:

```text
Internet
   ↓
CloudFront + AWS WAF
   ↓
Public ALB
   ↓
Private EC2
```

### Correct architecture

In this design, the NAT Gateway is located in the Public Subnet while EC2 remains in the Private Subnet.

The NAT Gateway is used for **outbound connectivity**. It is not part of the inbound application request path.

For higher availability, a production design should normally consider a NAT Gateway in each active Availability Zone so that an AZ failure does not make the private-subnet egress path dependent on a different AZ.

### Verdict

The NAT Gateway is easy to overlook because users do not interact with it directly. You notice its importance when a private workload suddenly cannot reach the Internet.

> **"Private does not mean disconnected. It means outbound access is controlled through a separate network path."**

---

## 🔍 Experiment 3: 🌍 What Happens If I Remove CloudFront?

### What does CloudFront do in this architecture?

CloudFront is the edge distribution in front of the application.

AWS WAF can be associated with the CloudFront distribution so that requests can be inspected before CloudFront forwards them to the origin.

CloudFront provides several important capabilities.

**TLS termination at the edge.**

Users establish HTTPS connections to a CloudFront edge location. TLS can be terminated at the edge before CloudFront forwards the request to the origin.

**CDN caching.**

Static assets such as images, CSS, JavaScript, and fonts can be cached at edge locations. Cache hits reduce the number of requests that need to reach the origin.

**A convenient attachment point for WAF.**

When AWS WAF is associated with CloudFront, requests can be inspected at the edge before CloudFront forwards them to the origin.

**Origin abstraction.**

Users normally interact with the CloudFront distribution rather than directly with the origin hostname.

### What happens if I remove CloudFront?

The application can still run, but the architecture loses several edge-level capabilities.

**WAF placement changes.**

AWS WAF can also be associated with an Application Load Balancer, but the inspection point moves from the CloudFront edge to the ALB.

**Less edge caching.**

Static content that could have been served from CloudFront cache locations must instead be handled by the origin.

**Different latency characteristics.**

Users access the regional application entry point more directly instead of first reaching a nearby CloudFront edge location.

**The origin becomes more directly exposed.**

The ALB becomes the primary Internet-facing entry point for the application.

### Verdict

CloudFront is not only a CDN. In this architecture, it also provides an edge layer for content delivery, caching, TLS handling, and integration with AWS WAF.

> **"CloudFront is not just about speed. It also changes where performance and security controls are applied."**

---

## 🔍 Experiment 4: 🛡️ What Happens If I Remove AWS WAF?

### What does AWS WAF do, and why is it associated with CloudFront?

In this architecture, AWS WAF is associated with the CloudFront distribution.

WAF is not a standalone network proxy that should be drawn as a separate hop in the request path. Instead, AWS WAF uses a **Web ACL (Web Access Control List)** that is associated with a supported protected resource.

A simplified conceptual relationship is:

```text
CloudFront
   └── AWS WAF Web ACL
```

Requests can be evaluated by WAF before CloudFront forwards them to the origin.

AWS Managed Rules and custom rules can help address common Layer 7 threats and unwanted request patterns, such as:

- SQL injection attempts
- Cross-site scripting (XSS)
- Known malicious IP addresses or bot sources
- Excessive request rates through rate-based rules

### What happens if I remove WAF?

The application can continue to function normally.

However, the architecture loses an important Layer 7 request-filtering control.

That means unwanted or malicious requests can reach the application stack unless other security controls detect or block them.

The application still needs secure coding practices, authentication, authorization, input validation, and other defensive mechanisms. WAF is an additional security layer, not a replacement for application security.

### A common mistake: drawing WAF as a network hop

A misleading diagram would look like this:

```text
CloudFront → WAF → Internet Gateway → ALB
```

This makes WAF look like an independent proxy or router.

A better representation is:

```text
CloudFront
   - - - - - - - - → AWS WAF Web ACL
```

The dashed relationship communicates that WAF is a security control associated with the protected resource rather than a separate network hop.

### Verdict

WAF is valuable because it enables request-level inspection and filtering before traffic reaches the application origin.

> **"WAF is a security control around the request path, not another router in the network path."**

---

## 🔍 Experiment 5: ⚖️ What Happens If I Remove the Application Load Balancer?

### What does the ALB do in this architecture?

The Application Load Balancer is the public entry point inside the VPC. It receives application traffic and distributes it to healthy EC2 instances in the Private Subnets.

It provides several important capabilities.

**Health-check-based routing.**

The ALB continuously performs health checks against registered targets. If a target becomes unhealthy, the ALB stops routing new requests to that target.

**TLS termination.**

HTTPS can terminate at the ALB, after which traffic can be forwarded to EC2 over HTTP or HTTPS depending on the security design.

**Connection draining / deregistration delay.**

When a target is removed from service, the ALB can stop sending new requests to it while allowing in-flight requests to complete, depending on the configured deregistration delay.

**A controlled entry point for private EC2.**

The EC2 instances do not need public IP addresses. Their Security Group can instead allow inbound application traffic from the ALB Security Group.

A simplified flow is:

```text
CloudFront + AWS WAF
        ↓
       ALB
        ↓
Private EC2
```

### What happens if I remove the ALB?

You lose centralized load balancing and health-based request routing.

For example, directly publishing multiple EC2 endpoints with DNS would introduce additional operational and security complexity.

Another option would be to make CloudFront use a single EC2 instance as the origin, but then there is no application load balancing across the compute tier.

Most importantly, removing the ALB weakens the architectural separation between the Internet-facing entry point and the private application instances.

### Verdict

The ALB is the application traffic distribution layer between the public entry point and the private compute tier.

> **"An ALB is not only a load balancer. It is also a controlled entry point to the private application tier."**

---

## 🔍 Experiment 6: 📈 What Happens If I Remove Auto Scaling?

### What does an Auto Scaling Group do?

In this architecture, the EC2 instances are managed by an **Auto Scaling Group (ASG)** spanning multiple Availability Zones.

The ASG can perform several functions:

- **Health management:** replace instances that become unhealthy
- **Scale out:** launch additional instances when scaling policies require more capacity
- **Scale in:** terminate instances when capacity is no longer required
- **Multi-AZ distribution:** maintain instances across multiple Availability Zones according to the ASG configuration

Scaling policies can use CloudWatch metrics such as CPU utilization or application-specific metrics.

### What happens if I remove Auto Scaling?

You can still run multiple EC2 instances, but capacity becomes largely static unless an operator manually changes it.

**Traffic spike.**

A sudden increase in traffic can overload the existing instances if there is not enough spare capacity.

**EC2 instance failure.**

If an instance becomes unhealthy and there is no Auto Scaling Group replacing it, the available capacity remains reduced.

**Operational overhead.**

Engineers have to monitor capacity and perform scaling or replacement actions manually.

### Practical scenario: Black Friday

Suppose an e-commerce application normally runs with two EC2 instances.

During a major sale event, traffic increases rapidly.

With an ASG, scaling policies can automatically add capacity according to the configured thresholds and policies.

Without an ASG, the system depends on the capacity that was provisioned in advance.

### Verdict

Auto Scaling is not just about adding servers when CPU is high. It is an **automated capacity and instance-lifecycle management mechanism**.

> **"Auto Scaling reduces the amount of manual operational work required to keep application capacity aligned with demand."**

---

## 🔍 Experiment 7: 🗄️ What Happens If I Replace RDS with a Database on EC2?

### RDS Multi-AZ in this architecture

Amazon RDS is placed in the private database tier and is not directly exposed to the Internet.

For a Multi-AZ DB instance deployment, RDS maintains a primary DB instance and a standby DB instance in another Availability Zone for failover support. The standby does not serve read traffic in this deployment model.

AWS manages the underlying high-availability and failover process.

### What happens if I install the database directly on EC2?

Running PostgreSQL or another database engine directly on EC2 gives you more control, but it also moves more operational responsibilities to you.

You now need to manage:

- Backup and restore
- Replication
- Failover
- Database patching
- Maintenance
- Monitoring
- Performance tuning
- Recovery procedures

### What do I lose by managing the database myself?

**Automated backup management.**

You need to design, schedule, monitor, and test your own backup process.

**Managed Multi-AZ failover.**

You are responsible for designing and operating database high availability.

**Managed maintenance.**

Database patching and maintenance become your responsibility.

**Operational isolation.**

Running the application and database on the same EC2 instance can create resource contention.

**Database administration work.**

You become responsible for tasks that a managed database service would otherwise handle.

### Practical scenario

If an EC2 instance hosting both the application and the database fails, both application availability and database availability can be affected at the same time.

With a managed RDS deployment, the database layer can use a separate high-availability design from the application compute tier.

### Verdict

The main value of RDS is not simply providing a database engine. It is reducing the amount of database infrastructure management that the application team must perform.

> **"RDS does not eliminate database responsibility, but it removes a large amount of undifferentiated operational work."**

---

## 🔍 Experiment 8: 📊 What Happens If I Remove CloudWatch?

### CloudWatch in this architecture

CloudWatch is an AWS monitoring and observability service used for metrics, logs, alarms, dashboards, and other telemetry.

It does not sit in the application's request path.

Instead, it provides visibility into components such as:

- EC2
- ALB
- RDS
- NAT Gateway
- Auto Scaling behavior

CloudWatch can also provide the metrics and alarms used by Auto Scaling policies.

### What happens if I remove CloudWatch?

The application can still run, but observability becomes significantly weaker.

**You lose visibility.**

You may not know that CPU utilization is high, error rates are increasing, or latency is degrading until users report a problem.

**Auto Scaling loses an important signal source.**

If a scaling policy relies on CloudWatch metrics, removing those metrics prevents the policy from using those signals as intended.

**Mean Time to Detect (MTTD) increases.**

Without alarms and dashboards, incidents may only become visible after customers are already affected.

**Troubleshooting becomes harder.**

Without logs and metrics, it becomes much more difficult to determine when a problem started and which change may have contributed to it.

### Cost consideration

CloudWatch pricing depends on the type and volume of metrics, logs, alarms, dashboards, and other features used.

The important lesson is not a fixed monthly number, but that observability should be treated as part of the operational design rather than as an optional afterthought.

### Verdict

CloudWatch is not part of the application request flow, but it is a critical part of operating the architecture.

> **"You cannot improve what you do not measure, and you cannot troubleshoot what you cannot observe."**

---

## 📌 Summary — Architecture Thinking Framework

### The common pattern

After going through each service, I noticed a repeating pattern:

When a service is removed, the problem it solves does not disappear. The responsibility simply moves somewhere else — usually to the operations team and often at the worst possible time.

### A three-question framework

When evaluating any service in an architecture, I can ask:

**1. "What happens if this service fails?"**

This helps identify the **failure domain**.

Does it affect one user, part of the traffic, or the entire system? How quickly can the system recover?

**2. "If I remove this service, who has to do the work instead?"**

This identifies the **operational ownership**.

Does the work move to the Ops team? Does it become a manual process? Or does the problem become invisible until it causes an incident?

**3. "Is the trade-off worth it?"**

This is the **value-versus-cost** question.

The right comparison is not only the AWS bill. It also includes engineering effort, operational risk, recovery time, security exposure, and the complexity introduced by the alternative design.

### Summary table

| Removed service | Does the system still run? | Direct impact | Who carries the burden? |
|---|---|---|---|
| Internet Gateway | Public Internet connectivity is lost | Public-facing access is affected | Operations must restore the VPC Internet path |
| NAT Gateway | Yes, for inbound application traffic | Private EC2 loses normal outbound Internet access | Application / Ops team must diagnose outbound failures |
| CloudFront | Yes, with a different edge architecture | No CloudFront edge caching and edge layer | Users may experience different latency; WAF can be associated with ALB instead |
| AWS WAF | Yes | Less Layer 7 request filtering | Application and security controls carry more of the burden |
| ALB | Yes, but with a more fragile design | No centralized load balancing or health-based routing | Operations and the application tier absorb the complexity |
| Auto Scaling | Yes | Capacity becomes largely static | Operations team handles scaling and replacement manually |
| RDS → self-managed DB on EC2 | Yes, but with more operational responsibility | More work for backup, HA, patching, and recovery | Application / Ops team becomes responsible for database operations |
| CloudWatch | Yes | Significantly weaker observability | Engineers discover incidents later and troubleshoot with less evidence |

### Closing thought

Architecture is not about using as many AWS services as possible.

It is also not about removing everything to reduce cost.

Architecture is about **understanding exactly what problem each service solves** — and making a conscious decision about whether that protection, capability, or automation is necessary for the current stage of the system.

A junior engineer may look at a diagram and ask:

> **"What does this service do?"**

A more experienced engineer may look at the same diagram and ask:

> **"If this component disappears at 2 a.m., what breaks — and who gets paged?"**

That second question is where architecture thinking begins.

