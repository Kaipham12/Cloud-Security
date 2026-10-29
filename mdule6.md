# Logging and Monitoring

## Module Objectives

By the end of this module you should be able to:

- Log and monitor access and control to help identify security threats.
- Read and interpret log reports to identify security threats.
- Monitor and report on your AWS resources and applications.
- Recognize when to use Amazon CloudWatch and when to use AWS CloudTrail.

## The Big Picture

- **Logging** = recording what happened.
- **Monitoring** = continuously checking security and performance.
- **CloudTrail** answers: who did what, when, and from where (actions and API calls).
- **CloudWatch** answers: how are my resources and applications performing (metrics, logs, alarms).
- Used together: CloudWatch alarms and notifications can be built on specific CloudTrail events.

> **Where this fits:** The shared responsibility model says AWS secures the cloud and you secure what you put in it (data, platform, applications, identity and access, OS, network and firewall configuration, encryption). Once those tiers are secured, logging and monitoring let you see and react to anomalous behavior in your own part of the model.

## 1. Importance of Logging and Monitoring

### 1.1 What Is Logging?

- Logging is the collection and recording of activity and event data.
- Logs can come from the service itself (e.g., Amazon VPC Flow Logs, Amazon S3 server access logs) or from a secondary service (e.g., AWS CloudTrail).
- The information logged varies by service, but some common elements appear in most logs:

| Common log element          | What it answers                                         |
| --------------------------- | ------------------------------------------------------- |
| Date and time of event      | When did it happen?                                     |
| Origin of event             | Where did the request come from (e.g., source IP address or location)? |
| Identity of resources accessed | What was touched, and by whom?                       |

### 1.2 Why Is Logging Important?

A log is a record of events, which is useful for:

- Troubleshooting
- Auditing
- Recordkeeping
- Incident response and remediation

Logs are a requirement for demonstrating compliance with regulations such as:

- **HIPAA** – Health Insurance Portability and Accountability Act (United States)
- **GDPR** – General Data Protection Regulation (European Union)
- **LGPD** – General Data Protection Law (Brazil)

Log files are central to incident response. They give investigators and responders detailed, time-stamped records of events captured at a specific point in time, so they help in every phase of your incident response strategy and show when an issue arose.

#### Common Log Types

| Log type             | What it typically records                                                                    |
| -------------------- | -------------------------------------------------------------------------------------------- |
| **Access logs**      | Requests made to a resource – who or what asked for access (e.g., S3 server access logs, ELB access logs) |
| **Application logs** | Messages, warnings, and errors written by the application itself                            |
| **System logs**      | Operating-system-level activity such as start-up, services, and sign-ins                     |
| **Event logs**       | Notable occurrences or state changes in a system or service                                  |
| **API call logs**    | Calls made to a service API – caller, action, time, origin (e.g., CloudTrail)                |
| **Database logs**    | Connections, queries, errors, and changes in a database                                      |

### 1.3 What Is Monitoring?

- Monitoring is the practice of continuously verifying the security and performance of your resources, applications, and data.
- AWS has several services that give you the visibility to spot issues before they impact operations:
  - AWS CloudTrail
  - Amazon CloudWatch
  - Amazon EventBridge
  - AWS X-Ray
- This module focuses on **CloudTrail** and **CloudWatch**.
- CloudTrail logs hold the action type, the identity of the user, and the time and date of the action, so you can see who is doing what and when.
- With CloudWatch you monitor resources and applications in real time, with system-wide visibility into resource utilization, application performance, and operational health.
- Monitoring tools plus a solid incident response plan help you reduce the effects of a system outage or a malicious actor. Often, monitoring lets you spot a problem before it has any operational impact.

### 1.4 Logging vs. Monitoring

|                  | Logging                                                              | Monitoring                                                    |
| ---------------- | -------------------------------------------------------------------- | ------------------------------------------------------------- |
| **Core idea**    | Collecting and recording activity and event data                     | Continuously verifying security and performance               |
| **Think of it as** | The written record (evidence of what happened)                     | The live watch (spotting problems as they develop)            |
| **Main uses**    | Troubleshooting, auditing, recordkeeping, incident response, compliance | Early warning, alerts, dashboards, health and performance checks |
| **AWS examples** | CloudTrail, VPC Flow Logs, S3 server access logs, ELB access logs    | CloudWatch, EventBridge, X-Ray (CloudTrail data feeds monitoring too) |

### Key Takeaways

- Logging is the collection and recording of activity and event data.
- Monitoring is the continuous verification of the security and performance of your resources, applications, and data.
- AWS provides several services you can use to log and monitor your resources.

## 2. Capture and Collect

### 2.1 AWS CloudTrail

- Helps you enable governance, compliance, and operational and risk auditing of your AWS account.
- Records actions taken by a user, role, or AWS service as **events**.
- Events include actions taken in the AWS Management Console, AWS CLI, AWS SDKs, and APIs.
- Gives visibility of events in the CloudTrail console.
- You can view, search, download, archive, analyze, and respond to account activity across your AWS infrastructure.
- You can integrate CloudTrail into applications using its API, automate trail creation for your organization, check the status of trails, and control how users view CloudTrail events.

> **Extra context (not on the slides):** By default, the CloudTrail console **Event history** shows the last 90 days of management events for your account. To keep a longer record (for audits or investigations), create a **trail**, which delivers log files to an Amazon S3 bucket and can also send them to CloudWatch Logs (you do exactly this in the lab).

### 2.2 API Security-Relevant Information

For every API call, CloudTrail records six pieces of information. Together they let you track changes to AWS resources (creation, modification, and deletion of things like Amazon EC2 instances and VPC security groups), troubleshoot operational issues, and show compliance with internal policies and regulatory standards.

| CloudTrail records…          | What it tells you                                   | Example from the activity log   |
| ---------------------------- | --------------------------------------------------- | ------------------------------- |
| Name of the API              | What action was requested                           | `StopInstances`                 |
| Identity of the caller       | Who (or what) made the call                         | IAM user Jane                   |
| Time of the API call         | When – always recorded in UTC                       | 2021-07-06 at 21:01 UTC         |
| Originating location of request | Where the call came from                         | Source IP `203.0.113.176`       |
| Request parameters           | What the caller asked for                           | Instance `i-ebeaf9e2`, `force: false` |
| Response elements            | What the AWS service returned                       | State change running → stopping |

### 2.3 Activity: Reading a CloudTrail Log File

The activity uses a log record generated when an IAM user named Jane used the `ec2-stop-instances` command in the AWS CLI to call the Amazon EC2 `StopInstances` action. Work through the log in three parts.

#### Part 1 – Identity of the Caller (`userIdentity` block)

| Field         | Value in the log                          | Meaning                                                                                              |
| ------------- | ----------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `type`        | `IAMUser`                                 | The kind of identity that made the call (an IAM user, not a role or the root user)                   |
| `userName`    | `Jane`                                    | The friendly name of the user                                                                        |
| `arn`         | `arn:aws:iam::111122223333:user/Jane`     | Amazon Resource Name – uniquely identifies the entity: AWS partition, service (IAM), account ID, and resource (`user/Jane`) |
| `accountId`   | `111122223333`                            | The AWS account the user belongs to                                                                  |
| `accessKeyId` | `AKIAIOSFODNN7EXAMPLE`                    | The access key used to sign the request (typical of CLI/SDK calls)                                   |

| Slide question                                      | Answer                                                    |
| --------------------------------------------------- | --------------------------------------------------------- |
| What type of account did the log collect information about? | An IAM user (`type: IAMUser`).                    |
| What can you determine from the `arn` element?      | The caller is the IAM user Jane in AWS account 111122223333. |

#### Part 2 – Time and Origin of the Request

| Field             | Value in the log             | Meaning                                         |
| ----------------- | ---------------------------- | ----------------------------------------------- |
| `eventTime`       | `2021-07-06T21:01:59Z`       | When the call was made, in UTC                  |
| `eventSource`     | `ec2.amazonaws.com`          | The AWS service the request went to (Amazon EC2) |
| `eventName`       | `StopInstances`              | The API action that was requested               |
| `awsRegion`       | `us-east-2`                  | The AWS Region the request was made in          |
| `sourceIPAddress` | `203.0.113.176`              | The IP address the request came from            |
| `userAgent`       | `ec2-api-tools 1.6.12.2`     | The client or tool used to make the request     |

| Slide question                                   | Answer                                                                                                   |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------- |
| What does the `eventSource` field tell you?      | Which service the request was sent to – here Amazon EC2.                                                 |
| In `eventName`, what does `StopInstances` indicate? | The action requested: stop one or more EC2 instances.                                                 |
| What method was used – Console, AWS CLI, or other? | A command-line tool (the `userAgent` is an EC2 CLI tool, not a browser). The scenario says Jane used the AWS CLI. |

#### Part 3 – Request Parameters and Response Elements

| Field                          | Value in the log                       | Meaning                                                   |
| ------------------------------ | -------------------------------------- | --------------------------------------------------------- |
| `requestParameters`            | `instanceId: i-ebeaf9e2`, `force: false` | What the caller asked for – stop this instance, without forcing it |
| `responseElements` → `previousState` | code `16`, `running`             | State of the instance before the call                     |
| `responseElements` → `currentState`  | code `64`, `stopping`            | State of the instance after the call                      |

| Slide question                                   | Answer                                                         |
| ------------------------------------------------ | -------------------------------------------------------------- |
| In `instanceId`, what does `i-ebeaf9e2` indicate? | The ID of the specific EC2 instance the action was performed on. |
| What was the action performed?                   | The instance was stopped – it moved from `running` to `stopping`. |

#### What the Log Tells You

Reading the whole record as one sentence:

> IAM user Jane (account 111122223333), from IP 203.0.113.176, using an EC2 command-line tool in us-east-2 at about 21:01 UTC on 6 July 2021, called `StopInstances` on instance `i-ebeaf9e2`, which moved from `running` to `stopping`.

That is the **who, what, when, where, and how** that CloudTrail gives you. If you need to track changes to resources, answer questions about user activity, demonstrate compliance, troubleshoot, or run a security analysis, CloudTrail is the tool.

### 2.4 Key Takeaways: Capture and Collect

- CloudTrail helps you enable governance, compliance, and auditing of your AWS account.
- Actions taken by a user, role, or AWS service are recorded as events.
- CloudTrail records important information about each API call, including the identity of the caller, the time of the call in UTC, and the origin of the call.

## 3. AWS Services with Built-in Logs

Several AWS services have logging built in, so you can switch on detailed records for that specific service without needing a separate tool.

### 3.1 Amazon S3 – Server Access Logging

- Provides detailed access request records; you enable the feature to log requests made to an S3 bucket.
- Useful for security and access audits.
- Can give insight into your customer base.
- Can show your storage usage, which helps you understand your Amazon S3 bill.

### 3.2 Amazon VPC – VPC Flow Logs

- Capture information about inbound and outbound IP traffic from:
  - The VPC
  - Subnets
  - Individual network interfaces
- Flow log data can be published to Amazon CloudWatch Logs or Amazon S3.
- Helpful to troubleshoot, monitor, and analyze the flow of IP traffic within your VPC.
- Flow log data is collected outside the path of your network traffic, so it has no impact on throughput or latency.

> **Typical details in a flow log record:** source IP, destination IP, ports, protocol, packets, bytes, and whether the traffic was accepted or rejected.

### 3.3 Elastic Load Balancing (ELB) – Access Logs

- Capture detailed information about requests sent to your load balancer.
- Each log has the time the request was received, the client IP address, latencies, request paths, and server responses.
- Use them to analyze traffic patterns and troubleshoot issues.
- ELB captures, compresses, and stores the logs in a specified Amazon S3 bucket.

### 3.4 Built-in Logs at a Glance

| Service                  | What is logged                                                                                   | Typical use                                              |
| ------------------------ | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------- |
| **Amazon S3**            | Requests made to a bucket (server access logs)                                                   | Security and access audits, customer insight, understanding the S3 bill |
| **Amazon VPC**           | Inbound and outbound IP traffic for a VPC, subnet, or network interface (flow logs) → CloudWatch Logs or S3 | Troubleshooting and monitoring network traffic |
| **Elastic Load Balancing** | Requests sent to the load balancer → compressed and stored in S3                               | Analyzing traffic patterns and troubleshooting           |

## 4. Monitor and Report

### 4.1 Amazon CloudWatch

- A monitoring and observability service.
- Gives a unified view of the operational health of your AWS resources, applications, and services.
- Collects metrics in the AWS Cloud and on premises.
- Can be used for infrastructure monitoring and troubleshooting.
- Lets you customize logs and events.
- Collects monitoring and operational data as **logs, metrics, and events**. With it you can detect anomalous behavior, set alarms, visualize logs and metrics side by side, take automated actions, troubleshoot, and discover insights. It breaks down data silos to give system-wide visibility.

#### Parts of CloudWatch Mentioned in the Guide

| Part                | What it does                                                                                                                                                                                                                                                                  |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CloudWatch Events** | Delivers a near-real-time stream of system events that describe changes in AWS resources and can trigger corrective action. Being replaced by Amazon EventBridge.                                                                                                           |
| **CloudWatch Logs** | Monitor, store, and access log files from EC2 instances, CloudTrail, Amazon Route 53, and other sources. Centralize logs in one scalable service, search for error codes or patterns, filter on fields, archive securely, query and sort, group, run custom computations with a query language, and visualize in dashboards. |

> **Extra context (not on the slides):** Other CloudWatch building blocks you will meet in the lab: **metrics** (numbers over time, e.g., CPU use), **alarms** (act when a metric crosses a threshold), **dashboards** (custom views), and **Logs Insights** (interactive queries over log data).

### 4.2 Comparing CloudTrail and CloudWatch

|                    | AWS CloudTrail                                                              | Amazon CloudWatch                                                         |
| ------------------ | --------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| **Watches**        | User activity and API calls – continuously monitors and logs user activities | Resource and application performance – continuously monitors it         |
| **Useful for**     | Compliance auditing, security analysis, and troubleshooting                 | Detecting anomalous service behavior, setting alarms, and discovering insights |
| **Answers**        | **WHO** performed **WHAT** unauthorized action and **WHEN** they did it     | Alerts you that an issue has occurred, e.g., due to an unauthorized action |

**Worked example from the guide:** an EC2 instance is deleted without permission.

- **CloudTrail** gives you a log with the identity of the user, group, or role that performed the unauthorized action.
- **CloudWatch** can detect and notify you of the anomalous behavior, such as the instance deletion – on its own or combined with CloudTrail.
- Together, you can create custom CloudWatch dashboards, alarms, and notifications for key metrics and specific CloudTrail events, so you can respond quickly to key operational issues.

> **Exam tip:** CloudTrail monitors **actions**. CloudWatch monitors **performance**. If a question asks for the identity of a user or who made a change, think CloudTrail. If it asks about metrics, alarms, dashboards, or performance, think CloudWatch.

### 4.3 Key Takeaways: Monitor and Report

- CloudWatch provides a unified view of the operational health of your AWS resources, applications, and services.
- CloudWatch collects monitoring and operational data as logs, metrics, and events.
- CloudTrail monitors actions, and CloudWatch monitors performance.
- Create custom dashboards, alarms, and notifications for key metrics.

## 5. Best Practices for Logging and Monitoring

### 5.1 Three Core Practices

| Step | Best practice                              | What it means                                                                                                                                                                                                                                                                                  |
| :--: | ------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | **Define your organizational requirements** | Define your requirements for logs, alerts, and metrics first. Identify the resources, applications, and services you need to keep logs for. Work out the organizational, legal, and compliance requirements for your workloads (they vary widely). Then evaluate which AWS resources can help. |
| 2    | **Configure service and application logging** | Configure logging throughout the workload: application logs, AWS service logs, and resource logs. Collect metrics and define baselines so you can gain insight into potential security threats. Define who receives alerts and what they should do with them.                              |
| 3    | **Analyze your logs centrally**            | Collect logs in one place. Use automation from services such as CloudWatch to analyze logs and detect anomalies or indicators of malicious activity or compromise.                                                                                                                              |

## 6. Additional AWS Services for Logging and Monitoring

### 6.1 AWS Trusted Advisor

- Provides recommendations based on five categories of AWS best practices:
  - Cost optimization
  - Security
  - Fault tolerance
  - Service limits (service quotas)
  - Performance improvement
- Evaluates your account and suggests improvements and optimizations for your resources, saving the time it would take to check manually.
- Accessible through the AWS Management Console and available to all support tiers:
  - **AWS Basic and Developer Support:** core security checks and service quota checks.
  - **AWS Business and Enterprise Support:** all checks – cost optimization, security, fault tolerance, performance, and service quotas.
- Trusted Advisor assesses posture but does **not** alert on anomalous behavior (this is why it is the wrong answer in the sample exam question).

### 6.2 Amazon EventBridge

- A serverless event bus used to connect your applications with data from a variety of sources.
- Provides a stream of real-time data from applications and services to targets such as AWS Lambda or other event buses.
- An **event** is a signal that a system's state has changed.
- No servers to provision, patch, or manage; it scales automatically and has built-in fault tolerance.
- Formerly Amazon CloudWatch Events. They use the same API, so existing code keeps working, but new features are only added to EventBridge.
- **Monitoring example:** when a resource is accessed by another account or publicly, an IAM Access Analyzer event can be sent via EventBridge to an AWS Lambda function that removes the unintended permissions.

### 6.3 AWS Security Hub

- Aggregates security alerts from various AWS services and third-party partner products in a standardized format.
- Runs automated, continuous security best-practice checks against your AWS resources and checks your cloud security posture.
- Gives a security score for each enabled standard and a total score across all accounts linked to the administrator account, so you can see your overall posture.
- Integrates with EventBridge to build automated response, remediation, and enrichment workflows.
- Helps central security teams and DevSecOps teams prioritize response and remediation by searching, correlating, and aggregating findings by account and resource.

### 6.4 AWS Config

- Helps you assess, audit, and evaluate the configurations of your AWS resources.
- Continuously monitors and records resource configurations.
- Provides automated evaluation of recorded configurations against desired configurations.
- Keeps historical configurations, so you can see which IAM policies were assigned to a user, group, or role at the time of recording – i.e., what permissions belonged to whom at that moment.
- Strong for change management: when resources are created, updated, or deleted, Config can stream the changes to Amazon SNS so you are notified. It also records relationships between resources, so you can see how one change may affect others.
- Supports compliance auditing, security analysis, change management, and troubleshooting.

### 6.5 Services Compared

| Service                 | Core job                                                | Keyword to look for                                              |
| ----------------------- | ------------------------------------------------------- | ---------------------------------------------------------------- |
| **AWS CloudTrail**      | Records API calls / actions (who, what, when, where)    | identity of user, audit, governance, who made the change         |
| **Amazon CloudWatch**   | Metrics, logs, alarms, dashboards for performance and health | performance, metrics, alarm, dashboard, anomalous behavior  |
| **Amazon EventBridge**  | Serverless event bus; routes events to targets          | event-driven, react to events, trigger Lambda                    |
| **AWS Config**          | Records and evaluates resource configurations over time | configuration history, compliance with desired config, what changed |
| **AWS Security Hub**    | Aggregates security findings and gives a security score | security posture, consolidated findings, security score          |
| **AWS Trusted Advisor** | Best-practice recommendations in five categories        | cost, security, fault tolerance, service limits, performance checks |
