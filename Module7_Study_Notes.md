# Module 7: Responding to and Managing an Incident
**Unit:** ACSF – AWS Academy Cloud Security Foundations (COMP3037 / ACSFv1)
**Sources:** Your lecture notes + Module 7 Student Guide (42 slides, pp. 4–45)

---

## Table of Contents
1. [Module at a Glance](#1-module-at-a-glance)
2. [Core Concepts](#2-core-concepts)
3. [Incident Response Phases](#3-incident-response-phases)
4. [AWS Services: Discovery & Recognition Phase](#4-aws-services-discovery--recognition-phase)
5. [AWS Services: Resolution & Recovery Phase](#5-aws-services-resolution--recovery-phase)
6. [Shared Responsibility Model](#6-shared-responsibility-model)
7. [Best Practices for Handling Incidents](#7-best-practices-for-handling-incidents)
8. [Case Study: Bank Business Scenario](#8-case-study-bank-business-scenario)
9. [Lab: Remediating an Incident Using AWS Config and Lambda](#9-lab-remediating-an-incident-using-aws-config-and-lambda)
10. [Master Cheat Sheet](#10-master-cheat-sheet)
11. [Self-Test Questions](#11-self-test-questions)
12. [Glossary](#12-glossary)

---

## 1. Module at a Glance

### Learning objectives
By the end of this module you should be able to:
- **Identify** an incident
- **Describe** the AWS services used for incident **recognition** and **remediation**
- **Identify** best practices for incident response

### Module structure
| # | Section | Focus |
|---|---------|-------|
| 1 | Introduction | Module overview and objectives |
| 2 | Identifying an incident | Definition, events vs. incidents, IR phases |
| 3 | AWS services: discovery & recognition | Trusted Advisor, CloudWatch, Inspector, GuardDuty, Shield, Config |
| 4 | AWS services: resolution & recovery | Systems Manager, CloudFormation, SNS, Step Functions, Lambda |
| 5 | Best practices for handling an incident | Industry practices |
| 6 | Lab | Remediating an incident using AWS Config and Lambda (90 min) |
| 7 | Summary and wrap-up | Module summary, knowledge check, practice exam question |

---

## 2. Core Concepts

### 2.1 Incident Response (IR)
**Definition:** Incident response is a **set of information security policies and procedures** that you can use to **identify, contain, and eliminate cyberattacks**.

**Goals of incident response:**
- **Quickly detect** and **halt** attacks
- **Minimize damage** caused by the attack
- **Prevent future attacks** of the same type

### 2.2 Events vs. Incidents
> **Key idea:** Not all events are incidents. Some events are only abnormal and need monitoring. Others are incidents that need immediate analysis and remediation.

| Example event | Why it may look suspicious | Incident needing immediate remedy? |
|---|---|---|
| **Logging in from a remote location** | Could be a compromised account | **Not necessarily**. The user may be travelling or using an approved VPN. Verify and monitor. |
| **Failing hard drive that is still fully operational** | Hardware is degrading | **Not an incident yet**. Schedule a replacement cycle *before* it fails. |
| **Employee trying to access resources they shouldn't** | Possible insider threat or misconfiguration | **Not necessarily**. If access is denied, it may not be a breach, but it is a **behavioural insight to monitor**. |

**Key terms from the example:**
- **VPN (Virtual Private Network):** An encrypted tunnel for remote access.
- **Hot swapping:** Removing or plugging in hardware components while the system is powered on. Replacing a failing drive this way is safe only if it is done *before* the drive has completely failed.

**Question the lecture raises:** How would an enterprise know that abuse is taking place? How does it tell abnormal events that need attention apart from incidents that need immediate analysis and remediation?

---

## 3. Incident Response Phases

Incident response happens in **two phases**.

### 3.1 Phase 1: Discovery and Recognition
| Step | What happens |
|---|---|
| **Incident identification, logging, and categorization** | An incident is identified through user reports, solution analyses (automated tools), or manual identification. It is then **logged** and **categorized**. |
| **Incident notification and escalation** | **Notification:** alerts are sent to the user, or to a specified channel (email, SMS, or push notification). **Escalation:** if an employee cannot resolve the incident, it is handed to a more experienced or specialised employee. |
| **Investigation and diagnosis** | Conduct a full incident investigation to gather answers and develop a strategy to resolve it. |

### 3.2 Phase 2: Resolution and Recovery
| Step | What happens |
|---|---|
| **Forensic isolation** | Isolate the affected system and perform a deep dive to find the root cause. Forensics often requires capturing a **disk image** or the **as-is configuration** of the operating system. If the issue is a software bug, you must **reproduce the bug**. Without reproduction, you cannot reliably fix it. |
| **Stage a fix** | Reproduce the issue, apply the fix, and test it, preferably in a **staging** environment. |
| **Deploy the fix** | Push new infrastructure (as code), or new application code, to **production**. |
| **Incident closure** | Resolve the incident and close the record. |

> **Tip:** Automate these processes wherever possible, and run **game days** in a staging environment to rehearse the response process.

### 3.3 Phase Summary
| | Phase 1: Discovery & Recognition | Phase 2: Resolution & Recovery |
|---|---|---|
| **Purpose** | Find and understand the incident | Fix it and restore normal operation |
| **Main activities** | Identify → log → categorise → notify → escalate → investigate | Isolate → reproduce → stage fix → deploy → close |
| **AWS services** | Trusted Advisor, CloudWatch, Inspector, GuardDuty, Shield, Config | Systems Manager, CloudFormation, SNS, Step Functions, Lambda |

---

## 4. AWS Services: Discovery & Recognition Phase

**Overview:** AWS offers several services that help you discover and recognise an attack. The student guide highlights six: Trusted Advisor, CloudWatch, Inspector, GuardDuty, Shield, and Config.

### 4.1 AWS Trusted Advisor
- Draws on **best practices learned from serving hundreds of thousands of AWS customers**.
- **Inspects your AWS environment**, then makes recommendations when opportunities exist to:
  - Improve **performance**
  - Improve **availability**
  - Save **cost**
  - **Close security gaps**
- **Support plans:**
  - *Basic / Developer:* core security checks and service quota checks
  - *Business / Enterprise On-Ramp / Enterprise:* full set of checks, via the console, Support API, and AWS CLI
- Can use **Amazon EventBridge** to monitor the status of Trusted Advisor checks.

### 4.2 Amazon CloudWatch
- A **reliable, scalable, and flexible monitoring** solution that you can start using **within minutes**.
- **Automatically displays metrics** about every AWS service you use.
- Lets you create **custom dashboards** and **custom metric collections**.
- Provides the ability to create **alarms** that watch metrics and:
  - send **notifications**, or
  - **automatically make changes** to the resources you are monitoring when a threshold is breached.
- Gives **system-wide visibility** into resource utilisation, application performance, and operational health.

### 4.3 Amazon Inspector
- A **vulnerability management service** that **continuously scans** your AWS workloads for vulnerabilities.
- **Automatically discovers and scans:**
  - **Amazon EC2 instances**
  - **Container images** stored in **Amazon ECR** (Elastic Container Registry)
- Checks for **software vulnerabilities** and **unintended network exposure**.
- When it discovers a vulnerability, it **creates a finding**. A finding describes the vulnerability, identifies the affected resource, rates its severity, and provides remediation guidance.

### 4.4 Amazon GuardDuty
- A **continuous security monitoring** service.
- **Identifies unexpected and potentially unauthorised or malicious activity** in your AWS environment.
- Uses **threat intelligence feeds** (such as lists of malicious IP addresses and domains) and **machine learning**.
- Can detect, for example:
  - **Compromised EC2 instances**: communication with malicious IPs or domains, or malware / bitcoin mining activity.
  - **Account compromise**: privilege escalation, exposed credentials, unusual API calls, or unauthorised infrastructure deployments.
  - Changes to account settings that reduce security, such as a weakened password policy.

### 4.5 AWS Shield
- **Automatically protects** against **DDoS** (Distributed Denial of Service) attacks.
- **Standard** protection is automatic for all AWS customers, and protects against common DDoS attacks.
- **AWS Shield Advanced** is a managed threat protection service that adds **additional DDoS detection, mitigation, and response capabilities**.

> **DDoS:** an attack that overwhelms a target with a flood of internet traffic from many sources, so legitimate traffic cannot reach it.

### 4.6 AWS Config
- A **continuous monitoring and assessment** service.
- Provides an **inventory of your AWS resources** and records changes to their configuration.
- Lets you:
  - View **current and historical** configurations of a resource
  - Use configuration history to troubleshoot outages and conduct security analysis
  - Receive **notifications when configurations change**
  - **Integrate with other AWS services** to **auto-remediate** issues
- **How evaluation works (from the slide diagram):**
  1. A configuration change occurs (example: an **S3 bucket** is made public).
  2. AWS Config records and normalises the change.
  3. AWS Config **evaluates** the change against your **rules**.
  4. Results appear in the console, and **alerts** can be sent.
  5. **Systems Manager** and **Amazon SNS** can be invoked to **remediate** or alert on the change.
  6. Change history and snapshots can be delivered to an **S3 bucket** for analysis.

### 4.7 Key Takeaways: Discovery & Recognition Services
- **Trusted Advisor:** inspects your environment and recommends improvements, including security gaps
- **CloudWatch:** monitors metrics and triggers alarms and actions
- **AWS Config:** tracks configuration changes and evaluates them against rules
- **Amazon Inspector:** finds vulnerabilities on EC2 instances and container images
- **Shield:** protects against DDoS attacks
- **GuardDuty:** detects unexpected or malicious activity using threat intelligence and machine learning

---

## 5. AWS Services: Resolution & Recovery Phase

**Overview:** AWS offers several services that help with **resolution and recovery**. Lambda, Amazon SNS, and Step Functions are **event-driven response services**.

> **Event-driven response:** a computer program that is written to respond to actions generated by the user or the system.

### 5.1 AWS Systems Manager
- Gives you **visibility and control** of your infrastructure on AWS.
- Provides a **unified user interface** to view **operational data from multiple AWS services**.
- Lets you **group resources by application** and view operational data for monitoring and troubleshooting.
- Helps you keep your instances in their **defined state**, and make **on-demand changes** such as updating applications or running **shell scripts**.

### 5.2 AWS CloudFormation
- Helps you **model and set up AWS resources** so you spend less time managing them and more time on your applications.
- Uses a **template** that describes all the AWS resources you want. CloudFormation then creates and configures those resources, and figures out dependencies between them.
- Can be used to **re-create a staging environment** inside an **isolated or forensic VPC**.
- Lets you create and provision infrastructure **predictably and repeatedly**.

### 5.3 Amazon Simple Notification Service (SNS)
- An **event-driven web service** that lets applications, end users, and devices **instantly send and receive notifications** from the cloud.
- Uses a **publish/subscribe** model: a message is published to a **topic**, and subscribed endpoints receive it.
- **Integrates with AWS Lambda**: a published message can invoke a Lambda function that is subscribed to the topic, with the message payload as input.
- An enterprise can use SNS to **receive notifications of potential exploits**.

### 5.4 AWS Step Functions
- A **low-code, visual workflow service** that developers use to build distributed applications and **automate IT and business processes**.
- Lets you build **event-driven workflows** that manage:
  - **Failures** and **retries**
  - **Parallelisation**
  - **Service integrations**
  - **Observability**
- Lets developers focus on **higher-value business logic**.
- Useful when an enterprise experiences an abnormality: Step Functions can help you create complex, multi-step business logic in event-driven workflows that connect services, systems, or people within minutes.

### 5.5 AWS Lambda
- A **serverless, event-driven compute** service.
- Runs code **on demand**, without provisioning or managing servers.
- You pay only for the compute time you consume. You are not charged when your code is not running.
- Scales automatically, with no affinity to the underlying infrastructure.
- **Lambda functions are stateless.** Each invocation should not rely on data stored in the function itself. Store state in an external service such as Amazon S3, DynamoDB, or Secrets Manager.
- **Ways Lambda is invoked:**
  - **Push model:** an event source publishes an event and directly invokes the Lambda function (e.g. SNS, EventBridge).
  - **Pull model:** the Lambda service polls the event source and invokes the function when records are available.
  - **Request-response (synchronous):** the calling application invokes the function and waits for the result.
- Secrets can be stored and retrieved securely with **AWS Key Management Service (KMS)**.

### 5.6 Worked Example: Lambda for Incident Response
**Scenario:** CloudTrail logging is disabled on an account.
1. **CloudTrail** detects that logging has been stopped (`StopLogging` request).
2. **EventBridge** receives the event and invokes a **Lambda function** to automatically remediate it.
3. The Lambda function **re-enables logging** (automatic remediation).
4. Information about the event (who disabled it, and which resource was affected) is sent through **SNS** to a **response analyst**.
5. The analyst can then investigate the user who disabled logging, and generate a notification or alert.

> **Takeaway:** Detective mechanisms (CloudTrail) detect the event. Responsive mechanisms (EventBridge, Lambda, SNS) react to it. Automating this reduces the time between detection and response.

### 5.7 Worked Example: Working Together for Incident Response
**Scenario:** A compromised EC2 instance is detected. The workflow remediates it using Step Functions, Lambda, CloudFormation, and SNS.
1. A **script or third-party tool** pushes an alert with the **instance ID** to an SNS topic.
2. A **Lambda function** verifies the instance ID. If it is compromised, the function starts a **Step Functions** workflow.
3. The instance is **removed from its Auto Scaling group**.
4. A **snapshot of every attached Amazon EBS volume** is created.
5. The instance is **isolated** by swapping its security groups for a new one with **no inbound or outbound rules**.
6. A **CloudFormation template** creates a **new VPC** with a **forensic instance**, which has forensic tools attached to **copies of the volumes** from the snapshots.
7. A basic **forensic investigation** is performed on the forensic instance.
8. A **report** is generated from the investigation results and sent to the team through **SNS**.

### 5.8 Key Takeaways: Resolution & Recovery Services
- **Systems Manager:** visibility and control of infrastructure
- **CloudFormation:** deploys infrastructure predictably and repeatedly, and can rebuild a staging or forensic environment
- **Lambda:** runs code in response to events without managing servers
- **Amazon SNS:** delivers messages from events to subscribers, such as endpoints and clients
- **Step Functions:** coordinates the steps of a distributed application as a visual workflow

---

## 6. Shared Responsibility Model

Shown on slide 11, as context for incident response. Responsibility is split between the customer and AWS.

| | **Customer** (security *in* the cloud) | **AWS** (security *of* the cloud) |
|---|---|---|
| **Data** | Customer data | Physical infrastructure |
| **Applications and identity** | Platform, applications, identity and access management | Foundation services (compute, storage, databases, networking) |
| **Operating system and network** | OS, network, and firewall configuration | Global infrastructure (Regions, Availability Zones, edge locations) |
| **Encryption and protection** | Client-side encryption, server-side encryption, network traffic protection | Security of the underlying cloud |

> **Why it matters for IR:** You are responsible for detecting and responding to incidents in *your* resources (configurations, data, identities). AWS provides the tools to do so.

---

## 7. Best Practices for Handling Incidents

Industry best practices, from slides 38 to 39 and your notes:

| # | Best practice | What it means |
|---|---|---|
| 1 | **Identify key personnel, external resources, and tooling** | Maintain a **contact list** of people who must be involved in an incident. Engage **external partners** (e.g. a specialist firm) if needed. Research and test the tools you will need. |
| 2 | **Automate containment capabilities** | Automation reduces the time between a **detective** mechanism and a **responsive** mechanism. Automating containment reduces damage. |
| 3 | **Develop an incident response plan** | Create **easy-to-follow runbooks** with steps for each type of incident. Include **escalation and communication plans** for internal and external parties. Include a process to find and document the **root cause**. Define **mitigation strategies** to limit or prevent damage. |
| 4 | **Pre-provision access and tools** | Security personnel should have their tools **pre-deployed** in AWS and the **correct access pre-provisioned**. This means they can respond immediately without waiting for access requests. |
| 5 | **Run incident response game days** | Simulate incident response events (involving key staff and management, for different threats). Use lessons learned to build a **feedback loop** that improves your processes. |

---

## 8. Case Study: Bank Business Scenario

A case study that runs through the module. It applies the concepts to a bank moving to the AWS cloud.

**Setup (Slides 8–10):**
- A bank, "AnyBank", is migrating its systems to the AWS cloud.
- **John** (manager) announces the migration to **Maria** (security lead). The board supports it.
- Maria is delighted, but she knows that **securing resources in the cloud is never fully complete**, so there is something more to address.

**Key point (Slide 10):**
- The migration means the bank's **incident response plan needs to be reviewed and updated**.
- The plan must include the **new tools and methods** that the security administrators will use.
- Updating the plan is **Maria's next project**.

**Lessons from the case:**
- Migration changes the attack surface, so the IR plan must change too.
- Shared responsibility: the bank must secure what it puts in the cloud.
- Use the module's services (GuardDuty, Config, CloudWatch, Systems Manager) and best practices (runbooks, pre-provisioned access, game days).

---

## 9. Lab: Remediating an Incident Using AWS Config and Lambda

**Duration:** 90 minutes
**Purpose:** Practise detecting and remediating a misconfiguration automatically.

### Lab tasks
1. Examine and update **IAM roles**
2. Set up **AWS Config** to monitor AWS resources
3. Modify a **security group** that AWS Config monitors
4. Create and run an **AWS Config rule** that calls a **Lambda function**
5. Revisit the **security group** configuration (check whether the remediation worked)
6. Use **CloudWatch Logs** to verify the results

### Lab debrief: key takeaways
After completing the lab, the educator may lead a discussion of the key takeaways. Points to prepare for:
- How Config rules detect a non-compliant change
- How Lambda remediates automatically in response to a Config evaluation
- How CloudWatch Logs give you evidence to verify the fix

---

## 10. Master Cheat Sheet

### AWS services at a glance
| Service | Phase | One-line purpose |
|---|---|---|
| **Trusted Advisor** | Discovery | Checks your environment against best practice, including security gaps |
| **CloudWatch** | Discovery | Monitors metrics, dashboards, and alarms that can trigger actions |
| **Inspector** | Discovery | Scans EC2 and ECR container images for vulnerabilities |
| **GuardDuty** | Discovery | Detects malicious or unexpected activity (threat intel + ML) |
| **Shield** | Discovery / defence | Protects against DDoS attacks (Advanced adds more) |
| **AWS Config** | Discovery | Records configuration changes and evaluates them against rules |
| **Systems Manager** | Resolution | Visibility and control of infrastructure; run commands and patches |
| **CloudFormation** | Resolution | Template-based infrastructure; rebuilds staging or forensic environments |
| **SNS** | Resolution | Publish/subscribe notifications to people and systems |
| **Step Functions** | Resolution | Visual workflows that coordinate multi-step response processes |
| **Lambda** | Resolution | Serverless code that runs in response to events |

### The 5 must-remember ideas
1. **IR** = policies and procedures to **identify, contain, and eliminate** cyberattacks.
2. **Not every event is an incident.** Triage first.
3. **Phase 1** = Discover and recognise. **Phase 2** = Resolve and recover.
4. **Automate** detection → response (Config/CloudWatch → EventBridge/Lambda → SNS).
5. **Prepare:** runbooks, pre-provisioned access and tools, and game days.

---

## 11. Self-Test Questions

Try answering these before checking the notes above.

1. **Q:** What is incident response?
   **A:** A set of information security policies and procedures used to identify, contain, and eliminate cyberattacks.

2. **Q:** Why is "logging in from a remote location" not automatically an incident?
   **A:** The user may be travelling or using an approved VPN. It is an event to verify and monitor, not an incident needing immediate remedy.

3. **Q:** What are the two phases of incident response?
   **A:** Phase 1 is discovery and recognition. Phase 2 is resolution and recovery.

4. **Q:** Which phase includes escalation, and what does escalation mean?
   **A:** Phase 1. Escalation hands an incident to a more experienced or specialised employee when the current person cannot resolve it.

5. **Q:** Why must a software bug be reproduced during forensic isolation?
   **A:** Without reproducing it, you cannot reliably fix it or confirm the fix works.

6. **Q:** Which service inspects EC2 and ECR container images for vulnerabilities?
   **A:** Amazon Inspector.

7. **Q:** Which service uses threat intelligence feeds and machine learning to detect malicious activity?
   **A:** Amazon GuardDuty.

8. **Q:** What is the difference between Shield Standard and Shield Advanced?
   **A:** Standard is automatic DDoS protection for all customers. Advanced adds extra detection, mitigation, and response capabilities.

9. **Q:** What does AWS Config do, and how does it help with remediation?
   **A:** It records configuration changes and evaluates them against rules. Integrated services (such as Systems Manager, SNS, or Lambda) can then auto-remediate or alert.

10. **Q:** Why are Lambda functions described as stateless?
    **A:** They keep no state between invocations. Any state must be stored in an external service.

11. **Q:** How can CloudFormation help in an incident?
    **A:** It can re-create a staging or forensic environment inside an isolated VPC, so you can analyse or test without touching production.

12. **Q:** Name the five industry best practices for handling incidents.
    **A:** Identify key personnel, external resources, and tooling. Automate containment. Develop an IR plan. Pre-provision access and tools. Run game days.

13. **Q:** In the Lambda worked example, what happens after CloudTrail logging is disabled?
    **A:** CloudTrail detects `StopLogging`, EventBridge invokes Lambda, Lambda re-enables logging, and SNS notifies a response analyst.

14. **Q:** Under the shared responsibility model, who is responsible for IAM configuration and customer data?
    **A:** The customer ("security in the cloud"). AWS is responsible for the security *of* the cloud (its infrastructure and foundation services).

---

## 12. Glossary

| Term | Meaning |
|---|---|
| **Incident** | An event that needs immediate analysis and remediation |
| **Incident response (IR)** | Policies and procedures to identify, contain, and eliminate cyberattacks |
| **Escalation** | Passing an incident to a more experienced or specialised person |
| **Forensic isolation** | Isolating a system and capturing its state (disk image, config) for investigation |
| **Staging environment** | A test copy of production used to verify fixes before deployment |
| **Game day** | A simulated incident used to rehearse and improve the response process |
| **Runbook** | A step-by-step, easy-to-follow procedure for handling an incident type |
| **Hot swapping** | Replacing hardware components while the system is powered on |
| **VPN** | Virtual Private Network: an encrypted connection for remote access |
| **DDoS** | Distributed Denial of Service: flooding a target with traffic to make it unavailable |
| **Threat intelligence feed** | Continuously updated lists of known malicious IPs, domains, and behaviours |
| **Vulnerability** | A weakness that an attacker could exploit |
| **Finding** | A record created by a security service (e.g. Inspector) describing a vulnerability or issue |
| **Event-driven** | A design where code runs in response to events (actions by users or systems) |
| **Stateless** | Keeps no data between invocations (Lambda) |
| **Publish/subscribe (pub/sub)** | Senders publish messages to a topic; subscribers receive them (SNS) |
| **Serverless** | You run code without provisioning or managing servers (Lambda) |
| **Automatic remediation** | A system fixes a problem without human action (e.g. Config rule + Lambda) |
| **Shared responsibility model** | Division of security duties between AWS (of the cloud) and the customer (in the cloud) |
| **Auto Scaling group** | A group of EC2 instances that scales in and out automatically |
| **EBS volume / snapshot** | Block storage for EC2 / point-in-time backup of a volume |
| **ECR** | Elastic Container Registry: AWS's container image store |
| **Security group** | A virtual firewall controlling inbound and outbound traffic for resources |
