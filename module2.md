# Security in the AWS Cloud

## Benefits of the Cloud

1. **Trade fixed expense for variable expense**
   Pay only for what you use (pay as you go) instead of investing upfront in data centers and servers.

2. **Benefit from massive economies of scale**
   Higher economies of scale lead to lower pay-as-you-go prices.

3. **Stop guessing capacity needs**
   Scale up and down as required, with only a few minutes' notice, instead of guessing capacity in advance.

4. **Increase speed and agility**
   Reduce the time to make new IT resources available to developers from weeks to minutes.

5. **Stop spending money to run and maintain data centers**
   Focus on your business and customers instead of infrastructure.

6. **Go global in minutes**
   Easily deploy your application in multiple Regions around the world.

## Security is familiar

1. **Confidentiality**
   Limit information access and disclosure to authorized users, and prevent access by unauthorized people.

2. **Integrity**
   Maintain the consistency of data throughout its lifecycle, preserving data both at rest and in transit.

3. **Availability**
   Ensure information resources are accessible when needed.

## AWS Cloud Security Objectives

AWS strives to make security as familiar as traditional security while adding cloud capabilities:

1. **Controllability**
   Can I effectively manage users? How can I provide temporary credentials? Can I use my own keys?

2. **Auditability**
   Who has access? Who performed what action? When and from where? Where is the evidence?

3. **Visibility**
   What is in my environment? What impact did an action have? What has changed? Where is the evidence?

4. **Agility**
   Deploy security controls quickly and at scale to keep pace with business demands.

5. **Automation**
   Automate routine security tasks so security experts can focus on strategic measures.

## Security Design Principles

1. **Apply the principle of least privilege**
   Grant only the access that is needed.
   - Grant access only as needed, based on job role.
   - Start by denying access to everything, then grant incrementally.
   - Enforce separation of duties with appropriate authorization.
   - Reduce long-term credentials; use temporary credentials instead.
   - For workforce identities, use AWS IAM Identity Center (AWS Single Sign-On) or federation with IAM.
   - For machine identities, use IAM roles instead of long-term access keys.

   AWS IAM enables you to:
   - Define principals (accounts, users, roles, services).
   - Build granular policies aligned with principals.
   - Enforce strong password practices and MFA.
   - Use federation with existing directory services.

2. **Enable traceability**
   Monitor and audit actions and changes in real time.
   - Use logs and metrics to gain visibility.
   - Integrate with existing logging and monitoring solutions.

   Implement detective controls:
   - **AWS CloudTrail**: logs AWS API calls and actions.
   - **Amazon CloudWatch**: monitors metrics with alarming capability.
   - **AWS Config**: provides configuration history and change tracking.
   - **Amazon GuardDuty**: managed threat detection for malicious behavior.
   - **Service-level logs**: for example, Amazon S3 access request logging.

3. **Secure all layers**
   Implement defense in depth.
   - Apply security to all layers (network, application, data store).
   - Require strong authentication to applications.
   - Ensure users come from trusted network paths.
   - Require access to decryption keys to process encrypted data.
   - Use multiple AWS services together for comprehensive security.

   Harden instances:
   - Tailor the configuration of EC2, ECS, or Elastic Beanstalk instances.
   - Persist the configuration to an immutable Amazon Machine Image (AMI).
   - New instances launched from the hardened AMI receive the security configuration.

4. **Automate security**
   Free security experts to focus on strategic measures.
   - Automate routine tasks security experts normally handle.
   - Allow monitoring systems to respond to events automatically.

   Automation approaches:
   - Automate routine security tasks with APIs.
   - Implement infrastructure as code (IaC).
   - Use AWS CloudFormation for reproducible deployments.
   - Automate creation of forensics environments for investigations.
   - Reduce operator access to minimize the attack surface.

5. **Protect data in transit and at rest**
   Safeguard data with these principles:
   - Implement fine-grained access controls.
   - Create and control encryption keys.
   - Select appropriate encryption methods.
   - Implement integrity validation.
   - Use tagging to classify data sensitivity levels.

   AWS encryption services:
   - **Server-side encryption (SSE)**: Amazon S3 stores data in encrypted form.
   - **Elastic Load Balancing (ELB)**: handles HTTPS encryption and decryption (SSL termination).

   In-transit protection:
   - Virtual Private Network (VPN).
   - Transport Layer Security (TLS) connections.

6. **Prepare for security events**
   Put incident response processes in place.
   - Put processes in place to respond to and mitigate security incidents.
   - Design the architecture to support effective incident management.
   - Have tools and access in place before incidents occur.
   - Routinely practice incident response through game days.

   AWS incident response capabilities:
   - Detailed logs contain important content (file access, changes).
   - Events can be processed automatically with AWS APIs.
   - Pre-provision tooling and a "clean room" using CloudFormation.
   - Conduct forensics in safe, isolated environments.

7. **Minimize the attack surface**
   Reduce what an attacker can reach.
   - Harden operating systems and minimize components.
   - Remove unused components, packages, and applications.
   - Configure security groups and network ACLs in the VPC.
   - Use Auto Scaling and Amazon CloudFront to absorb attacks.

   Attack types and mitigation:
   - **UDP reflection attacks**: the attacker asks a target for information using a forged source address.
   - **SYN flood (DDoS)**: makes a server unavailable by consuming all its resources; automatic scaling helps absorb larger volumes of attacks.

## Shared Responsibility Model

AWS is responsible for the security **of** the cloud: the foundation services and the global infrastructure listed below.

1. **AWS foundation services**
   Compute, storage, database, and networking.
   - **Compute**: Amazon EC2, Amazon ECS, AWS Lambda, etc.
   - **Storage**: Amazon S3.
   - **Database**: Amazon RDS.
   - **Networking**: Amazon VPC, Elastic Load Balancing.

2. **AWS global infrastructure**
   Data centers, Availability Zones, Regions, and edge locations.
   - **Data center**: a physical facility that hosts servers, networking equipment, and storage systems.
   - **Availability Zone (AZ)**: one or more data centers.
   - **Region**: a physical location in the world where AWS has multiple Availability Zones (for example, us-east-1, us-east-2, ap-south-1, ap-southeast-2).
   - **Edge location**: part of the AWS content delivery network, delivering content and applications with low latency and high performance.

## Shared respondibility example
