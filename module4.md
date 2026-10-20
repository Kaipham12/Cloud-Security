# Module 4: Securing Your Infrastructure

## Shared Responsibility Model

This module focuses on the **customer side** of the model:

- OS, network and firewall configuration
- Network traffic protection (encryption, integrity, identity)

## Structure of a Three-Tier Web Application

| Tier | Role | AWS components in the example |
| --- | --- | --- |
| 1. Presentation | What users interact with (frontend) | Public subnets, Application Load Balancer, NAT gateway, bastion host |
| 2. Business logic (application) | Processes requests | Frontend and backend Auto Scaling groups in private subnets, Network Load Balancer between them |
| 3. Data storage | Stores data | Primary database in one AZ, read-only replica in another AZ |

### Why Use Three Tiers

- Each tier does one job and can be managed and scaled independently.
- It replaces the monolithic model, where frontend, backend and database all live in one place.
- **Security benefit:** only the presentation layer sits near the internet, while the logic and data tiers stay private.

## Using a VPC

A VPC is a logically isolated section of the AWS Cloud where you launch resources in a virtual network you define.

- You control:
  - the IP address range
  - subnets
  - route tables and network gateways
- Supports IPv4 and IPv6.
- Security layers inside a VPC include security groups and network ACLs.

### VPCs vs Subnets

| Attribute | VPC | Subnet |
| --- | --- | --- |
| Scope | One Region, can span multiple AZs | Part of a VPC, lives in one AZ |
| Ownership | Dedicated to your AWS account | A range of IP addresses that divides the VPC |
| Definition | Isolated virtual network | Segment of the VPC's IP range |
| Type | N/A | Public or private |

> **Tip:** VPC = Region-level, Subnet = AZ-level. For high availability, create subnets in different AZs.

## Public/Private Subnets and Internet Connectivity

### Internet Gateway (IGW)

1. Provides a target in route tables for internet-bound traffic.
2. Performs NAT for instances with public IPv4 addresses.

Other facts:

- Supports IPv4 and IPv6.
- Doesn't create availability risks or bandwidth limits.
- No extra charge.

**4 steps to give a subnet internet access:**

1. Create an IGW and attach it to the VPC.
2. Add a route in the subnet's route table pointing internet traffic (`0.0.0.0/0`) to the IGW.
3. Make sure each instance has a globally unique IP (public IPv4, Elastic IP, or IPv6).
4. Make sure the NACL and security group rules allow the traffic.

### NAT Gateway

- Lets instances in a private subnet reach the internet or other AWS services (for example, to download patches).
- Blocks the internet from starting connections to those instances.
- When you create one, you must specify:
  - a public subnet for the NAT gateway to live in
  - an Elastic IP address
- After creating it, update the private subnet's route table to send internet traffic to the NAT gateway.
- **NAT gateway vs NAT instance:** a NAT gateway is managed, more available, has higher bandwidth and needs less admin work. AWS recommends the NAT gateway.

### Private Subnet

- Every subnet is a contiguous range of IPs, and subnets must not overlap.
- Instances in a private subnet can't be reached from outside the VPC.
- They can still make outbound requests through a NAT gateway, and replies are allowed back in.
- Typically host databases and other resources that don't need public access.
- Traffic path:

  ```text
  EC2 (private) → private route table → NAT gateway (public subnet) → public route table → IGW
  ```

### Public Subnet

A subnet counts as public when **both** are true:

1. The instance has a public IP address.
2. The subnet's route table has a route to the internet gateway.

> **AWS best practice:** don't put web servers directly in the public subnet. Put a load balancer in the public subnet and keep the web servers in private subnets.

## IP Addressing

### CIDR

- When you create a VPC, you assign it a CIDR block, a range of private addresses.
- You can't change the CIDR range of an existing VPC or subnet, but you can add extra CIDR ranges to a VPC.
- A subnet's CIDR can be the same as the VPC's (one subnet only) or a subset of it (multiple subnets).
- Subnet ranges must not overlap, because duplicate IPs aren't allowed in a VPC.

> **Quick formula:** total addresses = 2^(32 − prefix). For example, `/24` gives 2^8 = 256 addresses.

### Reserved IP Addresses

AWS reserves 5 IPs in every subnet. For the subnet `10.0.0.0/24`:

| IP address | Purpose |
| --- | --- |
| `10.0.0.0` | Network address |
| `10.0.0.1` | VPC router (internal communication) |
| `10.0.0.2` | DNS resolution |
| `10.0.0.3` | Future use |
| `10.0.0.255` | Network broadcast address |

So a `/24` subnet gives 256 − 5 = **251** usable IPs.

> **Exam tip:** usable IPs = 2^(32 − prefix) − 5. For example, `/28` gives 16 − 5 = 11 usable IPs.

### Public IP vs Elastic IP

| Attribute | Public IP | Elastic IP (EIP) |
| --- | --- | --- |
| Assigned by | Subnet's auto-assign setting | Allocated to your AWS account |
| Changes? | Dynamic: new IP after a stop/start | Static: never changes |
| Best for | Testing and short-term use | Production and long-term projects |
| Cost | N/A | Charged while allocated but unused, so release it when done |

Other Elastic IP facts:

- Every instance gets a private IP automatically.
- You can remap an EIP to another instance to hide a failure, or point a DNS record at it.
- EIPs come from Amazon's IPv4 pool or from your own IP pool (BYOIP).

### Elastic Network Interface (ENI)

- A virtual network card that you can attach to or detach from instances.
- Its attributes follow it when you move it, and traffic is redirected to the new instance.
- Each instance has a primary ENI, which can't be detached. You can add extra ENIs; the limit depends on the instance type.
- **Forensics use:** attach a second ENI so you can connect a forensic instance and track an exploit.

## Route Tables

- A route table is a set of rules called routes. Each route has a:
  - **Destination:** the CIDR block where traffic is going
  - **Target:** what the traffic is sent through (IGW, NAT gateway, and so on)
- Every route table has a **local route** for traffic inside the VPC, and you can't delete it.
- The **main route table** is created automatically. It applies to any subnet not explicitly associated with another table.
- A subnet can be associated with only one route table at a time, but one route table can serve many subnets.
- **Best practice:** give each subnet its own route table.

### Key Takeaways

- Public subnets are for resources that external traffic must reach.
- Private subnets are for things like databases that don't need internet access.
- Route tables decide where traffic goes.

## Security Groups

- A virtual firewall for an EC2 instance that controls inbound and outbound traffic.
- Works at the **instance (network interface) level**, not the subnet level.
- Different instances in the same subnet can have different security groups.
- Example: the web server security group allows traffic from the internet, and the database security group only allows traffic from the web server's security group.

### Rules and Default Behaviour

| Direction | Default behaviour |
| --- | --- |
| Inbound | No rules, so all inbound traffic is denied until you add rules |
| Outbound | Allows all outbound traffic (`0.0.0.0/0` and `::/0`) |

- Security groups support **allow rules only**. You can't write deny rules.
- If you remove every outbound rule, no outbound traffic is allowed.
- **Stateful:** if a request is allowed in, the response is automatically allowed out, and vice versa. No rules are needed for return traffic.
- All rules are evaluated before a decision is made. There's no rule order.
- A rule's source can be another security group (for example, `sg-xxxx`), which allows traffic from any instance in that group.

## Network ACLs (NACLs)

- An optional extra layer of security that acts as a firewall at the **subnet level**.
- Every subnet must be associated with a NACL. If you don't choose one, the default NACL is used.
- One NACL can cover many subnets, but each subnet has only one NACL at a time. Associating a new one replaces the old association.

### NACL Rules

- Separate inbound and outbound rules, and each rule can **ALLOW** or **DENY**.
- **Stateless:** return traffic must be explicitly allowed by rules in the other direction.
- Rules are evaluated in number order, lowest first, and the first match wins.
- The final rule is the asterisk (`*`) rule, which denies anything that didn't match. You can't modify or delete it.

| NACL type | Default behaviour |
| --- | --- |
| Default NACL | Allows all inbound and outbound traffic (rule 100 ALLOW, then `*` DENY) |
| Custom NACL | Denies all inbound and outbound traffic until you add rules |

### Security Groups vs NACLs

*High-yield for exams.*

| Attribute | Security Group | Network ACL |
| --- | --- | --- |
| Scope | Instance / interface | Subnet |
| Rules | Allow only | Allow and Deny |
| State | Stateful | Stateless |
| Evaluation | All rules evaluated | In number order |
| Default | Deny all in, allow all out | Default NACL allows all; custom NACL denies all |

> **Memory trick:** Security Group = **S**tateful, guards a sin**G**le instance. NACL = **N**umbered rules, **N**o memory (stateless), whole **N**eighbourhood (subnet).

### VPC Security Layers (from outside in)

1. Subnet routing (outer layer)
2. Network ACLs
3. Security groups
4. EC2 instance (centre)

Subnets also make traffic more efficient, because packets travel shorter paths through fewer routers.

### VPC Flow Logs

- Capture information about IP traffic going to and from network interfaces.
- Can be created at the VPC, subnet, or ENI level. A VPC or subnet flow log monitors every interface inside it.
- Can be published to CloudWatch Logs or Amazon S3.
- Each captured entry is a **flow log record**, describing one traffic flow.
- Useful for monitoring, troubleshooting and forensic investigation.

## Elastic Load Balancing (ELB)

### What ELB Does

- Distributes incoming traffic across targets such as EC2 instances, containers, IP addresses and Lambda functions.
- A **listener** checks for connection requests on a protocol and port.
- Scales automatically, improving availability and fault tolerance. Works in one AZ or across several.
- **Health checks** send traffic only to healthy targets. Traffic resumes once a target recovers.
- You can add or remove instances without disrupting the application.
- Integrates with EC2 Auto Scaling, ECS, CloudFormation and AWS Certificate Manager (ACM).

### Types of Load Balancer

| Type | Works at | Traffic | Key features |
| --- | --- | --- | --- |
| Application LB (ALB) | Request level (Layer 7) | HTTP / HTTPS | Content-based routing, good for microservices and containers, keeps SSL/TLS ciphers up to date |
| Network LB (NLB) | Connection level (Layer 4) | TCP / UDP | Millions of requests per second, ultra-low latency, one static IP per AZ, handles sudden traffic spikes |
| Classic LB (CLB) | Request and connection level | Basic | Legacy option for EC2-Classic applications |

### Data Protection in ELB

| Feature | What it means |
| --- | --- |
| Single point of contact | Clients connect only to the load balancer, which spreads traffic across AZs. An ALB can handle HTTPS certificates and terminate SSL so the application doesn't have to. |
| Encryption at rest | ELB access logs stored in S3 with SSE-S3 are encrypted automatically. Each file gets a unique key, which is itself encrypted with a regularly rotated key. |
| Encryption in transit | ELB terminates HTTPS/TLS and does the encryption and decryption work instead of each EC2 instance. |

### Load Balancers in a Multi-AZ Design

Traffic flows through the layers like this:

```text
Internet → IGW → ELB (public subnet) → web servers (private) → internal ELB → app servers (private) → primary DB (private, AZ A)
```

The primary DB replicates to a standby DB in AZ B. Every tier exists in both AZs, which gives high availability.

## Pulling It All Together

### Example Workflow

- The VPC is `10.0.0.0/16`, with an IPv6 block as well.
- **Subnet A** (public, `10.0.1.0/20`): instances → ELB → security group → NACL → route table → internet gateway. The NAT gateway lives here.
- **Subnet B** (private, `10.0.2.0/20`): instances → ELB → security group → NACL → route table → NAT gateway in Subnet A.

### Best Practices to Protect Your Network

1. **Control traffic at all layers.** Use security groups, NACLs and subnets for both inbound and outbound traffic. Separate application tiers into subnets across multiple AZs, and allow only the traffic that's needed.
2. **Inspect and filter traffic** at the application level.
3. **Automate network protection.** Use threat intelligence and anomaly detection to build a self-defending network.
4. **Limit exposure.** Allow only the minimum required access, both from the internet and internally (least privilege).

## Protecting Your Compute Resources

### Amazon Inspector

- An automated security assessment service for EC2 instances and applications.
- Finds vulnerabilities and deviations from best practices, both before deployment and in production. For example, it can flag unintended network access to an instance.
- You can define standards and check whether your applications follow them.
- Produces findings ranked by severity, viewable in the console, through the API, or in reports.

**Security benefits:**

| Benefit | Detail |
| --- | --- |
| Automated response | Combine with Amazon EventBridge to trigger actions based on findings |
| Regular monitoring | Catches issues before and after deployment |
| AWS expertise | Uses a knowledge base of rules that AWS keeps updated |
| DevOps integration | API-driven, with an optional agent so assessments can be built into your pipeline |

### AWS Systems Manager

- Inspector uses the **SSM Agent** to collect software inventory and configurations from EC2 instances, and uses that data to assess vulnerabilities.
- Systems Manager gives visibility and control of your infrastructure through one unified interface.
- Capabilities include:
  - collecting inventory
  - applying OS patches
  - keeping antivirus definitions up to date
  - configuring operating systems and applications at scale
- Helps keep systems compliant with your configuration policies.
