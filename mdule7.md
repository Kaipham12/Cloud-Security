# Responding to and Managing an Incident

## Module Objectives

By the end of this module you should be able to:

- **Identify** an incident.
- **Describe** the AWS services used for incident **recognition** and **remediation**.
- **Identify** best practices for incident response.

### Module Structure

| #   | Section                                 | Focus                                                             |
| --- | --------------------------------------- | ----------------------------------------------------------------- |
| 1   | Introduction                            | Module overview and objectives                                    |
| 2   | Identifying an incident                 | Definition, events vs. incidents, IR phases                       |
| 3   | AWS services: discovery and recognition | Trusted Advisor, CloudWatch, Inspector, GuardDuty, Shield, Config |
| 4   | AWS services: resolution and recovery   | Systems Manager, CloudFormation, SNS, Step Functions, Lambda      |
| 5   | Best practices for handling an incident | Industry practices                                                |
| 6   | Lab                                     | Remediating an incident using AWS Config and Lambda (90 min)      |
| 7   | Summary and wrap-up                     | Module summary, knowledge check, practice exam question           |

## 1. Core Concepts

### 1.1 Incident Response (IR)

**Definition:** Incident response is a **set of information security policies and procedures** that you can use to **identify, contain, and eliminate cyberattacks**.

Goals of incident response:

- **Quickly detect** and **halt** attacks.
- **Minimize damage** caused by the attack.
- **Prevent future attacks** of the same type.

### 1.2 Events vs. Incidents

> **Key idea:** Not all events are incidents. Some events are only abnormal and need monitoring. Others are incidents that need immediate analysis and remediation.

| Example event                                          | Why it may look suspicious                  | Incident needing immediate remedy?                                                                               |
| ------------------------------------------------------ | ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Logging in from a remote location**                  | Could be a compromised account              | **Not necessarily.** The user may be traveling or using an approved VPN. Verify and monitor.                     |
| **Failing hard drive that is still fully operational** | Hardware is degrading                       | **Not an incident yet.** Schedule a replacement cycle _before_ it fails.                                         |
| **Employee trying to access resources they shouldn't** | Possible insider threat or misconfiguration | **Not necessarily.** If access is denied, it may not be a breach, but it is a **behavioral insight to monitor**. |

Key terms from the example:

- **VPN (Virtual Private Network):** An encrypted tunnel for remote access.
- **Hot swapping:** Removing or plugging in hardware components while the system is powered on. Replacing a failing drive this way is safe only if it is done _before_ the drive has completely failed.

> **Question raised in the lecture:** How would an enterprise know that abuse is taking place? How does it tell abnormal events that need attention apart from incidents that need immediate analysis and remediation?

## 2. Incident Response Phases

Incident response happens in **two phases**.

### 2.1 Phase 1: Discovery and Recognition

| Step                                                     | What happens                                                                                                                                                                                                                        |
| -------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Incident identification, logging, and categorization** | An incident is identified through user reports, solution analyses (automated tools), or manual identification. It is then **logged** and **categorized**.                                                                           |
| **Incident notification and escalation**                 | **Notification:** alerts are sent to the user or to a specified channel (email, SMS, or push notification). **Escalation:** if an employee cannot resolve the incident, it is handed to a more experienced or specialized employee. |
| **Investigation and diagnosis**                          | Conduct a full incident investigation to gather answers and develop a strategy to resolve it.                                                                                                                                       |

### 2.2 Phase 2: Resolution and Recovery

| Step                   | What happens                                                                                                                                                                                                                                                                                              |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Forensic isolation** | Isolate the affected system and perform a deep dive to find the root cause. Forensics often requires capturing a **disk image** or the **as-is configuration** of the operating system. If the issue is a software bug, you must **reproduce the bug**. Without reproduction, you cannot reliably fix it. |
| **Stage a fix**        | Reproduce the issue, apply the fix, and test it, preferably in a **staging** environment.                                                                                                                                                                                                                 |
| **Deploy the fix**     | Push new infrastructure (as code) or new application code to **production**.                                                                                                                                                                                                                              |
| **Incident closure**   | Resolve the incident and close the record.                                                                                                                                                                                                                                                                |

> **Tip:** Automate these processes wherever possible, and run **game days** in a staging environment to rehearse the response process.

### 2.3 Phase Summary

|                     | Phase 1: Discovery and Recognition                                | Phase 2: Resolution and Recovery                             |
| ------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------ |
| **Purpose**         | Find and understand the incident                                  | Fix it and restore normal operation                          |
| **Main activities** | Identify → log → categorize → notify → escalate → investigate     | Isolate → reproduce → stage fix → deploy → close             |
| **AWS services**    | Trusted Advisor, CloudWatch, Inspector, GuardDuty, Shield, Config | Systems Manager, CloudFormation, SNS, Step Functions, Lambda |

## 3. AWS Services: Discovery and Recognition Phase

AWS offers several services that help you discover and recognize an attack. The student guide highlights six: Trusted Advisor, CloudWatch, Inspector, GuardDuty, Shield, and Config.

### 3.1 AWS Trusted Advisor

- Draws on **best practices learned from serving hundreds of thousands of AWS customers**.
- **Inspects your AWS environment**, then makes recommendations when opportunities exist to:
  - Improve **performance**
  - Improve **availability**
  - Save **cost**
  - **Close security gaps**
- **Support plans:**
  - _Basic / Developer:_ core security checks and service quota checks.
  - _Business / Enterprise On-Ramp / Enterprise:_ full set of checks, via the console, Support API, and AWS CLI.
- Can use **Amazon EventBridge** to monitor the status of Trusted Advisor checks.

### 3.2 Amazon CloudWatch

- A **reliable, scalable, and flexible monitoring** solution that you can start using **within minutes**.
- **Automatically displays metrics** about every AWS service you use.
- Lets you create **custom dashboards** and **custom metric collections**.
- Lets you create **alarms** that watch metrics and:
  - send **notifications**, or
  - **automatically make changes** to the resources you are monitoring when a threshold is breached.
- Gives **system-wide visibility** into resource utilization, application performance, and operational health.

### 3.3 Amazon Inspector

- A **vulnerability management service** that **continuously scans** your AWS workloads for vulnerabilities.
- **Automatically discovers and scans:**
  - **Amazon EC2 instances**
  - **Container images** stored in **Amazon ECR** (Elastic Container Registry)
- Checks for **software vulnerabilities** and **unintended network exposure**.
- When it discovers a vulnerability, it **creates a finding**. A finding describes the vulnerability, identifies the affected resource, rates its severity, and provides remediation guidance.

### 3.4 Amazon GuardDuty

- A **continuous security monitoring** service.
- **Identifies unexpected and potentially unauthorized or malicious activity** in your AWS environment.
- Uses **threat intelligence feeds** (such as lists of malicious IP addresses and domains) and **machine learning**.
- Can detect, for example:
  - **Compromised EC2 instances:** communication with malicious IPs or domains, or malware / bitcoin mining activity.
  - **Account compromise:** privilege escalation, exposed credentials, unusual API calls, or unauthorized infrastructure deployments.
  - Changes to account settings that reduce security, such as a weakened password policy.

### 3.5 AWS Shield

- **Automatically protects** against **DDoS** (Distributed Denial of Service) attacks.
- **Standard** protection is automatic for all AWS customers and protects against common DDoS attacks.
- **AWS Shield Advanced** is a managed threat protection service that adds **additional DDoS detection, mitigation, and response capabilities**.

> **DDoS:** an attack that overwhelms a target with a flood of internet traffic from many sources, so legitimate traffic cannot reach it.

### 3.6 AWS Config

- A **continuous monitoring and assessment** service.
- Provides an **inventory of your AWS resources** and records changes to their configuration.
- Lets you:
  - View **current and historical** configurations of a resource.
  - Use configuration history to troubleshoot outages and conduct security analysis.
  - Receive **notifications when configurations change**.
  - **Integrate with other AWS services** to **auto-remediate** issues.

How evaluation works (from the slide diagram):

1. A configuration change occurs (example: an **S3 bucket** is made public).
2. AWS Config records and normalizes the change.
3. AWS Config **evaluates** the change against your **rules**.
4. Results appear in the console, and **alerts** can be sent.
5. **Systems Manager** and **Amazon SNS** can be invoked to **remediate** or alert on the change.
6. Change history and snapshots can be delivered to an **S3 bucket** for analysis.

### 3.7 Key Takeaways: Discovery and Recognition Services

- **Trusted Advisor:** inspects your environment and recommends improvements, including security gaps.
- **CloudWatch:** monitors metrics and triggers alarms and actions.
- **AWS Config:** tracks configuration changes and evaluates them against rules.
- **Amazon Inspector:** finds vulnerabilities on EC2 instances and container images.
- **Shield:** protects against DDoS attacks.
- **GuardDuty:** detects unexpected or malicious activity using threat intelligence and machine learning.

## 4. AWS Services: Resolution and Recovery Phase

AWS offers several services that help with **resolution and recovery**. Lambda, Amazon SNS, and Step Functions are **event-driven response services**.

> **Event-driven response:** a computer program that is written to respond to actions generated by the user or the system.

### 4.1 AWS Systems Manager

- Gives you **visibility and control** of your infrastructure on AWS.
- Provides a **unified user interface** to view **operational data from multiple AWS services**.
- Lets you **group resources by application** and view operational data for monitoring and troubleshooting.
- Helps you keep your instances in their **defined state**, and make **on-demand changes** such as updating applications or running **shell scripts**.

### 4.2 AWS CloudFormation

- Helps you **model and set up AWS resources** so you spend less time managing them and more time on your applications.
- Uses a **template** that describes all the AWS resources you want. CloudFormation then creates and configures those resources and figures out dependencies between them.
- Can be used to **re-create a staging environment** inside an **isolated or forensic VPC**.
- Lets you create and provision infrastructure **predictably and repeatedly**.

### 4.3 Amazon Simple Notification Service (SNS)

- An **event-driven web service** that lets applications, end users, and devices **instantly send and receive notifications** from the cloud.
- Uses a **publish/subscribe** model: a message is published to a **topic**, and subscribed endpoints receive it.
- **Integrates with AWS Lambda:** a published message can invoke a Lambda function that is subscribed to the topic, with the message payload as input.
- An enterprise can use SNS to **receive notifications of potential exploits**.

### 4.4 AWS Step Functions

- A **low-code, visual workflow service** that developers use to build distributed applications and **automate IT and business processes**.
- Lets you build **event-driven workflows** that manage:
  - **Failures** and **retries**
  - **Parallelization**
  - **Service integrations**
  - **Observability**
- Lets developers focus on **higher-value business logic**.
- Useful when an enterprise experiences an abnormality: Step Functions can help you create complex, multi-step business logic in event-driven workflows that connect services, systems, or people within minutes.

### 4.5 AWS Lambda

- A **serverless, event-driven compute** service.
- Runs code **on demand**, without provisioning or managing servers.
- You pay only for the compute time you consume. You are not charged when your code is not running.
- Scales automatically, with no affinity to the underlying infrastructure.
- **Lambda functions are stateless.** Each invocation should not rely on data stored in the function itself. Store state in an external service such as Amazon S3, DynamoDB, or Secrets Manager.
- Secrets can be stored and retrieved securely with **AWS Key Management Service (KMS)**.

Ways Lambda is invoked:

| Model                              | How it works                                                                                          |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **Push**                           | An event source publishes an event and directly invokes the Lambda function (e.g., SNS, EventBridge). |
| **Pull**                           | The Lambda service polls the event source and invokes the function when records are available.        |
| **Request-response (synchronous)** | The calling application invokes the function and waits for the result.                                |

### 4.6 Worked Example: Lambda for Incident Response

**Scenario:** CloudTrail logging is disabled on an account.

1. **CloudTrail** detects that logging has been stopped (`StopLogging` request).
2. **EventBridge** receives the event and invokes a **Lambda function** to automatically remediate it.
3. The Lambda function **re-enables logging** (automatic remediation).
4. Information about the event (who disabled it and which resource was affected) is sent through **SNS** to a **response analyst**.
5. The analyst can then investigate the user who disabled logging and generate a notification or alert.

> **Takeaway:** Detective mechanisms (CloudTrail) detect the event. Responsive mechanisms (EventBridge, Lambda, SNS) react to it. Automating this reduces the time between detection and response.

### 4.7 Worked Example: Working Together for Incident Response

**Scenario:** A compromised EC2 instance is detected. The workflow remediates it using Step Functions, Lambda, CloudFormation, and SNS.

1. A **script or third-party tool** pushes an alert with the **instance ID** to an SNS topic.
2. A **Lambda function** verifies the instance ID. If it is compromised, the function starts a **Step Functions** workflow.
3. The instance is **removed from its Auto Scaling group**.
4. A **snapshot of every attached Amazon EBS volume** is created.
5. The instance is **isolated** by swapping its security groups for a new one with **no inbound or outbound rules**.
6. A **CloudFormation template** creates a **new VPC** with a **forensic instance**, which has forensic tools attached to **copies of the volumes** from the snapshots.
7. A basic **forensic investigation** is performed on the forensic instance.
8. A **report** is generated from the investigation results and sent to the team through **SNS**.

### 4.8 Key Takeaways: Resolution and Recovery Services

- **Systems Manager:** visibility and control of infrastructure.
- **CloudFormation:** deploys infrastructure predictably and repeatedly, and can rebuild a staging or forensic environment.
- **Lambda:** runs code in response to events without managing servers.
- **Amazon SNS:** delivers messages from events to subscribers, such as endpoints and clients.
- **Step Functions:** coordinates the steps of a distributed application as a visual workflow.

## 5. Shared Responsibility Model

Shown on slide 11 as context for incident response. Responsibility is split between the customer and AWS.

|                                  | **Customer** (security _in_ the cloud)                                     | **AWS** (security _of_ the cloud)                                   |
| -------------------------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Data**                         | Customer data                                                              | Physical infrastructure                                             |
| **Applications and identity**    | Platform, applications, identity and access management                     | Foundation services (compute, storage, databases, networking)       |
| **Operating system and network** | OS, network, and firewall configuration                                    | Global infrastructure (Regions, Availability Zones, edge locations) |
| **Encryption and protection**    | Client-side encryption, server-side encryption, network traffic protection | Security of the underlying cloud                                    |

> **Why it matters for IR:** You are responsible for detecting and responding to incidents in _your_ resources (configurations, data, identities). AWS provides the tools to do so.

## 6. Best Practices for Handling Incidents

Industry best practices, from slides 38 to 39 and the lecture notes:

| #   | Best practice                                               | What it means                                                                                                                                                                                                                                                                      |
| --- | ----------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | **Identify key personnel, external resources, and tooling** | Maintain a **contact list** of people who must be involved in an incident. Engage **external partners** (e.g., a specialist firm) if needed. Research and test the tools you will need.                                                                                            |
| 2   | **Automate containment capabilities**                       | Automation reduces the time between a **detective** mechanism and a **responsive** mechanism. Automating containment reduces damage.                                                                                                                                               |
| 3   | **Develop an incident response plan**                       | Create **easy-to-follow runbooks** with steps for each type of incident. Include **escalation and communication plans** for internal and external parties. Include a process to find and document the **root cause**. Define **mitigation strategies** to limit or prevent damage. |
| 4   | **Pre-provision access and tools**                          | Security personnel should have their tools **pre-deployed** in AWS and the **correct access pre-provisioned**, so they can respond immediately without waiting for access requests.                                                                                                |
| 5   | **Run incident response game days**                         | Simulate incident response events (involving key staff and management, for different threats). Use lessons learned to build a **feedback loop** that improves your processes.                                                                                                      |
